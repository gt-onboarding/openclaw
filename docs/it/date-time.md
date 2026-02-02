---
title: Data e ora
summary: "Gestione di data e ora all'interno di envelope, prompt, strumenti e connettori"
read_when:
  - Stai modificando il modo in cui i timestamp vengono mostrati al modello o agli utenti
  - Stai facendo debug della formattazione di data e ora nei messaggi o nell'output del prompt di sistema
---

<div id="date-time">
  # Data e ora
</div>

Per impostazione predefinita OpenClaw utilizza **l&#39;ora locale dell&#39;host per i timestamp di trasporto** e **il fuso orario dell&#39;utente solo nel system prompt**.
I timestamp del provider vengono mantenuti in modo che gli strumenti preservino la loro semantica nativa (l&#39;ora corrente è disponibile tramite `session_status`).

<div id="message-envelopes-local-by-default">
  ## Envelope dei messaggi (locale per impostazione predefinita)
</div>

I messaggi in ingresso vengono corredati da un timestamp (precisione al minuto):

```
[Provider ... 2026-01-05 16:26 PST] message text
```

Il timestamp di questo envelope è **in base all&#39;ora locale dell&#39;host per impostazione predefinita**, indipendentemente dal fuso orario del provider.

Puoi modificare questo comportamento:

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

* `envelopeTimezone: "utc"` usa l&#39;UTC.
* `envelopeTimezone: "local"` usa il fuso orario dell&#39;host.
* `envelopeTimezone: "user"` usa `agents.defaults.userTimezone` (in caso di assenza, usa il fuso orario dell&#39;host).
* Usa un fuso orario IANA esplicito (ad es. `"America/Chicago"`) per una zona fissa.
* `envelopeTimestamp: "off"` rimuove i timestamp assoluti dalle intestazioni dell&#39;envelope.
* `envelopeElapsed: "off"` rimuove i suffissi di tempo trascorso (stile `+2m`).

<div id="examples">
  ### Esempi
</div>

**Ora locale (impostazione predefinita):**

```
[WhatsApp +1555 2026-01-18 00:19 PST] hello
```

**Fuso orario utente:**

```
[WhatsApp +1555 2026-01-18 00:19 CST] hello
```

**Tempo trascorso attivo:**

```
[WhatsApp +1555 +30s 2026-01-18T05:19Z] follow-up
```

<div id="system-prompt-current-date-time">
  ## System prompt: Data e ora correnti
</div>

Se il fuso orario dell’utente è noto, il prompt di sistema include un’apposita
sezione **Data e ora correnti** con **solo il fuso orario** (senza indicare l’ora)
per mantenere stabile la cache dei prompt:

```
Time zone: America/Chicago
```

Quando l&#39;agente ha bisogno dell&#39;ora corrente, utilizza lo strumento `session_status`; la scheda di stato contiene una riga con il timestamp.

<div id="system-event-lines-local-by-default">
  ## Righe di eventi di sistema (locali per impostazione predefinita)
</div>

Gli eventi di sistema in coda, inseriti nel contesto dell’agente, sono preceduti da un timestamp che utilizza
la stessa impostazione del fuso orario delle envelope dei messaggi (impostazione predefinita: fuso orario locale dell’host).

```
System: [2026-01-12 12:19:17 PST] Model switched.
```

<div id="configure-user-timezone-format">
  ### Configura fuso orario e formato data/ora dell&#39;utente
</div>

```json5
{
  agents: {
    defaults: {
      userTimezone: "America/Chicago",
      timeFormat: "auto" // auto | 12 | 24
    }
  }
}
```

* `userTimezone` imposta il **fuso orario locale dell’utente** per il contesto del prompt.
* `timeFormat` controlla il **formato 12/24 ore** nel prompt. `auto` segue le preferenze del sistema operativo.

<div id="time-format-detection-auto">
  ## Rilevamento del formato dell&#39;ora (automatico)
</div>

Quando `timeFormat: "auto"`, OpenClaw legge l&#39;impostazione del sistema operativo (macOS/Windows)
e in caso contrario utilizza la formattazione locale. Il valore rilevato viene **messo in cache per processo**
per evitare chiamate ripetute al sistema.

<div id="tool-payloads-connectors-raw-provider-time-normalized-fields">
  ## Payload degli strumenti e connettori (timestamp grezzi del provider + campi normalizzati)
</div>

Gli strumenti dei canali restituiscono **timestamp nativi del provider** e aggiungono campi normalizzati per coerenza:

* `timestampMs`: millisecondi epoch (UTC)
* `timestampUtc`: stringa ISO 8601 in UTC

I campi grezzi del provider vengono preservati in modo che non si perda nulla.

* Slack: stringhe in stile epoch dall&#39;API
* Discord: timestamp ISO in UTC
* Telegram/WhatsApp: timestamp numerici/ISO specifici del provider

Se ti serve l&#39;orario locale, convertilo a valle usando il fuso orario noto.

<div id="related-docs">
  ## Documentazione correlata
</div>

* [System Prompt](/it/concepts/system-prompt)
* [Fusi orari](/it/concepts/timezone)
* [Messaggi](/it/concepts/messages)