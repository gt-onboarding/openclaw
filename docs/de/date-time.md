---
title: Datum und Uhrzeit
summary: "Handhabung von Datum und Uhrzeit in Envelopes, Prompts, Tools und Connectors"
read_when:
  - Du änderst, wie Zeitstempel für das Modell oder für Nutzer angezeigt werden
  - Du debuggst die Zeitformatierung in Nachrichten oder der Ausgabe von System-Prompts
---

<div id="date-time">
  # Datum &amp; Uhrzeit
</div>

OpenClaw verwendet standardmäßig **hostlokale Zeit für Transport-Zeitstempel** und **die Benutzerzeitzone nur im System-Prompt**.
Anbieter-Zeitstempel werden beibehalten, damit Tools ihre native Semantik behalten (die aktuelle Zeit ist über `session_status` verfügbar).

<div id="message-envelopes-local-by-default">
  ## Nachrichtenhüllen (standardmäßig lokale Zeit)
</div>

Eingehende Nachrichten werden mit einem Zeitstempel versehen (minutengenaue Auflösung):

```
[Provider ... 2026-01-05 16:26 PST] message text
```

Dieser Umschlag-Zeitstempel ist **standardmäßig host-lokal**, unabhängig von der Zeitzone des Anbieters.

Du kannst dieses Verhalten überschreiben:

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
* `envelopeTimezone: "local"` verwendet die Host-Zeitzone.
* `envelopeTimezone: "user"` verwendet `agents.defaults.userTimezone` (fällt auf die Host-Zeitzone zurück).
* Verwende eine explizite IANA-Zeitzone (z. B. `"America/Chicago"`) für eine feste Zeitzone.
* `envelopeTimestamp: "off"` entfernt absolute Zeitstempel aus den Envelope-Headern.
* `envelopeElapsed: "off"` entfernt Suffixe für verstrichene Zeit (im Stil `+2m`).

<div id="examples">
  ### Beispiele
</div>

**Lokal (Standard):**

```
[WhatsApp +1555 2026-01-18 00:19 PST] hello
```

**Benutzerzeitzone:**

```
[WhatsApp +1555 2026-01-18 00:19 CST] hello
```

**Anzeige der verstrichenen Zeit aktiviert:**

```
[WhatsApp +1555 +30s 2026-01-18T05:19Z] follow-up
```

<div id="system-prompt-current-date-time">
  ## System-Prompt: Aktuelles Datum &amp; Uhrzeit
</div>

Wenn die Zeitzone des Benutzers bekannt ist, enthält der System-Prompt einen eigenen
Abschnitt **Aktuelles Datum &amp; Uhrzeit** mit **nur der Zeitzone** (kein konkretes Uhrzeit-/Zeitformat),
um das Prompt-Caching stabil zu halten:

```
Time zone: America/Chicago
```

Wenn der agent die aktuelle Uhrzeit benötigt, verwende das `session_status`-Tool; die Statuskarte enthält eine Zeile mit Zeitstempel.

<div id="system-event-lines-local-by-default">
  ## Systemereigniszeilen (standardmäßig lokal)
</div>

In die agent-Kontexte eingestellte Systemereignisse werden mit einem Zeitstempel-Präfix versehen,
wobei dieselbe Zeitzonenauswahl wie für Nachrichtenumschläge verwendet wird (Standard: Host-lokal).

```
System: [2026-01-12 12:19:17 PST] Model switched.
```

<div id="configure-user-timezone-format">
  ### Benutzerzeitzone und Zeitformat konfigurieren
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

* `userTimezone` legt die **lokale Zeitzone des Benutzers** für den Prompt-Kontext fest.
* `timeFormat` steuert die **12h-/24h-Anzeige** im Prompt. `auto` folgt den Einstellungen des Betriebssystems.

<div id="time-format-detection-auto">
  ## Erkennung des Zeitformats (auto)
</div>

Wenn `timeFormat: "auto"` gesetzt ist, liest OpenClaw die Betriebssystemeinstellung (macOS/Windows) aus
und fällt sonst auf die lokale Formatierung zurück. Der ermittelte Wert wird **pro Prozess zwischengespeichert**, um wiederholte Systemaufrufe zu vermeiden.

<div id="tool-payloads-connectors-raw-provider-time-normalized-fields">
  ## Tool-Payloads + Connectors (rohe Anbieterzeit + normalisierte Felder)
</div>

Channel-Tools geben **anbieterspezifische Zeitstempel** zurück und fügen zur Konsistenz normalisierte Felder hinzu:

* `timestampMs`: Epoch-Millisekunden (UTC)
* `timestampUtc`: ISO-8601-UTC-String

Rohe Felder des Anbieters bleiben erhalten, damit nichts verloren geht.

* Slack: epochenähnliche Strings aus der API
* Discord: UTC-ISO-Zeitstempel
* Telegram/WhatsApp: anbieterspezifische numerische/ISO-Zeitstempel

Wenn du lokale Zeit benötigst, konvertiere sie nachgelagert mit der bekannten Zeitzone.

<div id="related-docs">
  ## Verwandte Dokumentation
</div>

* [System Prompt](/de/concepts/system-prompt)
* [Zeitzonen](/de/concepts/timezone)
* [Nachrichten](/de/concepts/messages)