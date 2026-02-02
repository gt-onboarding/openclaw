---
title: Fuso orario
summary: "Gestione del fuso orario per agenti, envelope e prompt"
read_when:
  - Devi capire come vengono normalizzati i timestamp per il modello
  - Configurare il fuso orario dell'utente nei prompt di sistema
---

<div id="timezones">
  # Fusi orari
</div>

OpenClaw normalizza i timestamp in modo che il modello veda **un&#39;unica ora di riferimento**.

<div id="message-envelopes-local-by-default">
  ## Buste dei messaggi (in locale per impostazione predefinita)
</div>

I messaggi in ingresso sono racchiusi in una busta come:

```
[Provider ... 2026-01-05 16:26 PST] message text
```

Il timestamp dell&#39;envelope è **nel fuso orario dell&#39;host per impostazione predefinita**, con precisione al minuto.

Puoi sovrascrivere questo comportamento con:

```json5
{
  agents: {
    defaults: {
      envelopeTimezone: "local", // "utc" | "local" | "user" | fuso orario IANA
      envelopeTimestamp: "on", // "on" | "off"
      envelopeElapsed: "on" // "on" | "off"
    }
  }
}
```

* `envelopeTimezone: "utc"` usa UTC.
* `envelopeTimezone: "user"` usa `agents.defaults.userTimezone` (in caso contrario usa il fuso orario dell&#39;host).
* Usa un fuso orario IANA esplicito (ad esempio `"Europe/Vienna"`) per un offset fisso.
* `envelopeTimestamp: "off"` rimuove i timestamp assoluti dalle intestazioni degli envelope.
* `envelopeElapsed: "off"` rimuove i suffissi di tempo trascorso (in stile `+2m`).

<div id="examples">
  ### Esempi
</div>

**Ora locale (predefinita):**

```
[Signal Alice +1555 2026-01-18 00:19 PST] hello
```

**Fuso orario fisso:**

```
[Signal Alice +1555 2026-01-18 06:19 GMT+1] hello
```

**Tempo trascorso:**

```
[Signal Alice +1555 +2m 2026-01-18T05:19Z] follow-up
```

<div id="tool-payloads-raw-provider-data-normalized-fields">
  ## Payload degli strumenti (dati grezzi del provider + campi normalizzati)
</div>

Le chiamate agli strumenti (`channels.discord.readMessages`, `channels.slack.readMessages`, ecc.) restituiscono **timestamp grezzi del provider**.
Vengono inoltre aggiunti campi normalizzati per coerenza:

* `timestampMs` (millisecondi dall&#39;epoch UTC)
* `timestampUtc` (stringa UTC in formato ISO 8601)

I campi grezzi del provider vengono mantenuti.

<div id="user-timezone-for-the-system-prompt">
  ## Fuso orario utente per il system prompt
</div>

Imposta `agents.defaults.userTimezone` per indicare al modello il fuso orario locale dell&#39;utente. Se non è impostato, OpenClaw rileva **il fuso orario dell&#39;host in fase di esecuzione** (senza modificare la configurazione).

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } }
}
```

Il prompt di sistema include:

* una sezione `Current Date & Time` con ora locale e fuso orario
* `Time format: 12-hour` o `24-hour`

Puoi controllare il formato del prompt con `agents.defaults.timeFormat` (`auto` | `12` | `24`).

Consulta [Date &amp; Time](/it/date-time) per il comportamento dettagliato e gli esempi.
