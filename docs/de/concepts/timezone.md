---
title: Zeitzone
summary: "Zeitzonenbehandlung für Agenten, Envelopes und Prompts"
read_when:
  - Wenn du verstehen musst, wie Zeitstempel für das Modell normalisiert werden
  - Wenn du die Benutzerzeitzone für Systemprompts konfigurierst
---

<div id="timezones">
  # Zeitzonen
</div>

OpenClaw standardisiert Zeitstempel, damit das Modell eine **einheitliche Referenzzeit** verwendet.

<div id="message-envelopes-local-by-default">
  ## Nachrichtenumschläge (standardmäßig lokal)
</div>

Eingehende Nachrichten werden in einen Umschlag eingebettet, etwa so:

```
[Provider ... 2026-01-05 16:26 PST] message text
```

Der Zeitstempel im Envelope ist **standardmäßig host-lokal** mit einer Genauigkeit von Minuten.

Du kannst dies überschreiben mit:

```json5
{
  agents: {
    defaults: {
      envelopeTimezone: "local", // "utc" | "local" | "user" | IANA-Zeitzone
      envelopeTimestamp: "on", // "on" | "off"
      envelopeElapsed: "on" // "on" | "off"
    }
  }
}
```

* `envelopeTimezone: "utc"` verwendet UTC.
* `envelopeTimezone: "user"` verwendet `agents.defaults.userTimezone` (greift andernfalls auf die Host-Zeitzone zurück).
* Verwende eine explizite IANA-Zeitzone (z. B. `"Europe/Vienna"`) für einen festen Offset.
* `envelopeTimestamp: "off"` entfernt absolute Zeitstempel aus den Envelope-Headern.
* `envelopeElapsed: "off"` entfernt Suffixe für verstrichene Zeit (also Angaben wie `+2m`).

<div id="examples">
  ### Beispiele
</div>

**Lokal (Standard):**

```
[Signal Alice +1555 2026-01-18 00:19 PST] hello
```

**Feste Zeitzone:**

```
[Signal Alice +1555 2026-01-18 06:19 GMT+1] hello
```

**Vergangene Zeit:**

```
[Signal Alice +1555 +2m 2026-01-18T05:19Z] Nachverfolgung
```

<div id="tool-payloads-raw-provider-data-normalized-fields">
  ## Tool-Payloads (rohe Anbieterdaten + normalisierte Felder)
</div>

Toolaufrufe (`channels.discord.readMessages`, `channels.slack.readMessages`, etc.) liefern **rohe Anbieter-Zeitstempel**.
Zusätzlich fügen wir zur Vereinheitlichung normalisierte Felder hinzu:

* `timestampMs` (UTC-Epoch-Millisekunden)
* `timestampUtc` (ISO-8601-UTC-String)

Die rohen Anbieterfelder bleiben unverändert erhalten.

<div id="user-timezone-for-the-system-prompt">
  ## Benutzerzeitzone für den Systemprompt
</div>

Setze `agents.defaults.userTimezone`, damit das Modell die lokale Zeitzone des Benutzers kennt. Wenn dieser Wert
nicht gesetzt ist, bestimmt OpenClaw **die Host-Zeitzone zur Laufzeit** (ohne die Konfiguration zu verändern).

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } }
}
```

Der System-Prompt enthält:

* einen Abschnitt `Current Date & Time` mit lokaler Uhrzeit und Zeitzone
* `Time format: 12-hour` oder `24-hour`

Du kannst das Prompt-Format mit `agents.defaults.timeFormat` (`auto` | `12` | `24`) steuern.

Siehe [Date &amp; Time](/de/date-time) für eine vollständige Beschreibung des Verhaltens und Beispiele.
