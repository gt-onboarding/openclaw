---
title: Cron-Jobs
summary: "Cron-Jobs und Wakeups für den Gateway-Scheduler"
read_when:
  - Hintergrundjobs oder Wakeups einplanen
  - Automatisierung anbinden, die zusammen mit dem Herzschlag laufen soll
  - Sich zwischen Herzschlag und Cron für geplante Aufgaben entscheiden
---

<div id="cron-jobs-gateway-scheduler">
  # Cron-Jobs (Gateway-Scheduler)
</div>

> **Cron vs Herzschlag?** Siehe [Cron vs Heartbeat](/de/automation/cron-vs-heartbeat) für Hinweise, wann Sie welches verwenden sollten.

Cron ist der eingebaute Scheduler des Gateways. Er speichert Jobs dauerhaft, weckt den agent zur richtigen Zeit auf und kann optional die Ausgabe zurück in einen Chat liefern.

Wenn Sie etwas wie *„führe das jeden Morgen aus“* oder *„stupse den agent in 20 Minuten an“* möchten, ist Cron der richtige Mechanismus.

<div id="tldr">
  ## TL;DR
</div>

* Cron läuft **im Gateway** (nicht im Modell).
* Jobs werden unter `~/.openclaw/cron/` gespeichert, damit Zeitpläne Neustarts überstehen.
* Zwei Ausführungsmodi:
  * **Hauptsitzung**: ein Systemereignis einreihen, dann beim nächsten Herzschlag ausführen.
  * **Isoliert**: einen dedizierten Agent-Turn in `cron:&lt;jobId&gt;` ausführen und die Ausgabe bei Bedarf zustellen.
* Wakeups sind Objekte erster Klasse: Ein Job kann „jetzt wecken“ statt „beim nächsten Herzschlag“ anfordern.

<div id="beginner-friendly-overview">
  ## Einsteigerfreundliche Übersicht
</div>

Stell dir einen Cron-Job so vor: **wann** er läuft + **was** er tun soll.

1. **Zeitplan wählen**
   * Einmalige Erinnerung → `schedule.kind = "at"` (CLI: `--at`)
   * Wiederholter Job → `schedule.kind = "every"` oder `schedule.kind = "cron"`
   * Wenn dein ISO-Zeitstempel keine Zeitzone enthält, wird er als **UTC** interpretiert.

2. **Ausführungsort wählen**
   * `sessionTarget: "main"` → läuft beim nächsten Herzschlag mit Hauptkontext.
   * `sessionTarget: "isolated"` → führt einen dedizierten Agent-Turn in `cron:<jobId>` aus.

3. **Payload wählen**
   * Hauptsitzung → `payload.kind = "systemEvent"`
   * Isolierte Sitzung → `payload.kind = "agentTurn"`

Optional: `deleteAfterRun: true` entfernt erfolgreiche einmalige Jobs aus dem Speicher.

<div id="concepts">
  ## Konzepte
</div>

<div id="jobs">
  ### Jobs
</div>

Ein Cron-Job ist ein gespeicherter Eintrag mit:

* einem **schedule** (wann er ausgeführt werden soll),
* einem **payload** (was er tun soll),
* optionaler **delivery** (wohin die Ausgabe gesendet werden soll).
* optionaler **Agent-Bindung** (`agentId`): führt den Job unter einem bestimmten Agent aus; falls
  fehlend oder unbekannt, fällt das Gateway auf den Standard-Agent zurück.

Jobs werden durch eine stabile `jobId` identifiziert (verwendet von CLI-/Gateway-APIs).
In Agent-Toolaufrufen ist `jobId` kanonisch; das veraltete `id` wird zur Kompatibilität akzeptiert.
Jobs können optional nach einer erfolgreichen One-Shot-Ausführung automatisch gelöscht werden, mittels `deleteAfterRun: true`.

<div id="schedules">
  ### Zeitpläne
</div>

Cron unterstützt drei Arten von Zeitplänen:

* `at`: einmaliger Zeitstempel (ms seit der Unix-Epoche). Gateway akzeptiert ISO 8601 und wandelt nach UTC um.
* `every`: festes Intervall (ms).
* `cron`: Cron-Ausdruck mit 5 Feldern und optionaler IANA-Zeitzone.

Cron-Ausdrücke werden mit `croner` verarbeitet. Wenn eine Zeitzone weggelassen wird, wird die lokale Zeitzone des Gateway-Hosts verwendet.

<div id="main-vs-isolated-execution">
  ### Ausführung im Hauptkontext vs. isolierte Ausführung
</div>

<div id="main-session-jobs-system-events">
  #### Jobs der Hauptsitzung (Systemereignisse)
</div>

Main-Jobs stellen ein Systemereignis in die Warteschlange und wecken optional den Herzschlag-Runner.
Sie müssen `payload.kind = "systemEvent"` verwenden.

* `wakeMode: "next-heartbeat"` (Standard): Ereignis wartet auf den nächsten geplanten Herzschlag.
* `wakeMode: "now"`: Ereignis löst eine sofortige Herzschlag-Ausführung aus.

Dies ist die beste Option, wenn du den normalen Herzschlag-Prompt plus den Kontext der Hauptsitzung nutzen willst.
Siehe [Herzschlag](/de/gateway/heartbeat).

<div id="isolated-jobs-dedicated-cron-sessions">
  #### Isolierte Jobs (dedizierte Cron-Sitzungen)
</div>

Isolierte Jobs führen einen dedizierten Agent-Turn in der Sitzung `cron:&lt;jobId&gt;` aus.

Wesentliche Eigenschaften:

* Der Prompt wird zur Nachverfolgbarkeit mit `[cron:&lt;jobId&gt; &lt;job name&gt;]` präfixiert.
* Jeder Lauf startet eine **frische Sitzungs-ID** (keine Übernahme vorheriger Unterhaltung).
* Eine Zusammenfassung wird in der Hauptsitzung gepostet (Präfix `Cron`, konfigurierbar).
* `wakeMode: "now"` löst einen sofortigen Herzschlag aus, nachdem die Zusammenfassung gepostet wurde.
* Wenn `payload.deliver: true`, wird die Ausgabe an einen Kanal zugestellt; andernfalls bleibt sie intern.

Verwende isolierte Jobs für laute, häufige oder „Hintergrundaufgaben“, die deinen
Haupt-Chat-Verlauf nicht zuspammen sollen.

<div id="payload-shapes-what-runs">
  ### Payload-Formate (was ausgeführt wird)
</div>

Es werden zwei Payload-Arten unterstützt:

* `systemEvent`: nur Hauptsitzung, über den Herzschlag-Prompt geroutet.
* `agentTurn`: nur isolierte Sitzung, führt einen eigenen Agenten-Turn aus.

Gemeinsame `agentTurn`-Felder:

* `message`: erforderlicher Text-Prompt.
* `model` / `thinking`: optionale Overrides (siehe unten).
* `timeoutSeconds`: optionaler Timeout-Override.
* `deliver`: `true`, um die Ausgabe an ein Kanalziel zu senden.
* `channel`: `last` oder ein bestimmter Kanal.
* `to`: kanal­spezifisches Ziel (Telefon-/Chat-/Kanal-ID).
* `bestEffortDeliver`: verhindert nach Möglichkeit, dass der Job fehlschlägt, wenn die Zustellung fehlschlägt.

Isolationsoptionen (nur für `session=isolated`):

* `postToMainPrefix` (CLI: `--post-prefix`): Präfix für das Systemereignis in der Hauptsitzung.
* `postToMainMode`: `summary` (Standard) oder `full`.
* `postToMainMaxChars`: maximale Zeichenanzahl, wenn `postToMainMode=full` (Standard 8000).

<div id="model-and-thinking-overrides">
  ### Modell- und Denk-Overrides
</div>

Isolierte Jobs (`agentTurn`) können Modell und Denkstufe überschreiben:

* `model`: Anbieter-/Modell-String (z. B. `anthropic/claude-sonnet-4-20250514`) oder Alias (z. B. `opus`)
* `thinking`: Denkstufe (`off`, `minimal`, `low`, `medium`, `high`, `xhigh`; nur GPT-5.2- + Codex-Modelle)

Hinweis: Du kannst `model` auch bei Jobs der Hauptsitzung setzen, aber das ändert das gemeinsame Modell der Hauptsitzung. Wir empfehlen Modell-Overrides nur für isolierte Jobs, um unerwartete Kontextwechsel zu vermeiden.

Priorität bei der Auflösung:

1. Override im Job-Payload (höchste Priorität)
2. Hook-spezifische Standardwerte (z. B. `hooks.gmail.model`)
3. Standard in der Agent-Konfiguration

<div id="delivery-channel-target">
  ### Zustellung (Kanal + Ziel)
</div>

Isolierte Jobs können Ausgaben an einen Kanal zustellen. Die Job-Payload kann Folgendes angeben:

* `channel`: `whatsapp` / `telegram` / `discord` / `slack` / `mattermost` (Plugin) / `signal` / `imessage` / `last`
* `to`: kanal­spezifischen Empfänger

Wenn `channel` oder `to` weggelassen wird, kann cron auf die „last route“ der Hauptsitzung zurückfallen
(den letzten Ort, an dem der agent geantwortet hat).

Hinweise zur Zustellung:

* Wenn `to` gesetzt ist, stellt cron die endgültige Ausgabe des agents automatisch zu, selbst wenn `deliver` weggelassen wird.
* Verwende `deliver: true`, wenn du eine Zustellung über die „last route“ ohne ein explizites `to` möchtest.
* Verwende `deliver: false`, um Ausgaben intern zu behalten, selbst wenn ein `to` vorhanden ist.

Hinweise zum Zielformat:

* Slack-/Discord-/Mattermost-Plugin-Ziele sollten explizite Präfixe verwenden (z. B. `channel:<id>`, `user:<id>`), um Mehrdeutigkeiten zu vermeiden.
* Telegram-Themen sollten die Form `:topic:` verwenden (siehe unten).

<div id="telegram-delivery-targets-topics-forum-threads">
  #### Telegram-Zustellziele (Topics / Foren-Threads)
</div>

Telegram unterstützt Foren-Themen (Topics) über `message_thread_id`. Für die Cron-Auslieferung kannst du
das Topic bzw. den Thread im `to`-Feld kodieren:

* `-1001234567890` (nur Chat-ID)
* `-1001234567890:topic:123` (bevorzugt: explizite Topic-Markierung)
* `-1001234567890:123` (Kurzform: numerische Endung)

Ziele mit Präfix wie `telegram:...` / `telegram:group:...` werden ebenfalls akzeptiert:

* `telegram:group:-1001234567890:topic:123`

<div id="storage-history">
  ## Speicher &amp; Verlauf
</div>

* Job-Speicher: `~/.openclaw/cron/jobs.json` (JSON, vom Gateway verwaltet).
* Ausführungsverlauf: `~/.openclaw/cron/runs/<jobId>.jsonl` (JSONL, wird automatisch bereinigt).
* Speicherpfad überschreiben: `cron.store` in der Konfiguration.

<div id="configuration">
  ## Konfiguration
</div>

```json5
{
  cron: {
    enabled: true, // default true
    store: "~/.openclaw/cron/jobs.json",
    maxConcurrentRuns: 1 // Standardwert: 1
  }
}
```

Cron vollständig deaktivieren:

* `cron.enabled: false` (Konfiguration)
* `OPENCLAW_SKIP_CRON=1` (Umgebungsvariable)

<div id="cli-quickstart">
  ## CLI-Schnellstart
</div>

Einmalige Erinnerung (UTC ISO, automatisches Löschen bei Erfolg):

```bash
openclaw cron add \
  --name "Send reminder" \
  --at "2026-01-12T18:00:00Z" \
  --session main \
  --system-event "Reminder: submit expense report." \
  --wake now \
  --delete-after-run
```

Einmalige Erinnerung (Hauptsitzung, sofort aufwecken):

```bash
openclaw cron add \
  --name "Calendar check" \
  --at "20m" \
  --session main \
  --system-event "Next heartbeat: check calendar." \
  --wake now
```

Wiederkehrender isolierter Job (Zustellung an WhatsApp):

```bash
openclaw cron add \
  --name "Morning status" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize inbox + calendar for today." \
  --deliver \
  --channel whatsapp \
  --to "+15551234567"
```

Wiederkehrender, isolierter Job (an ein Telegram-Topic senden):

```bash
openclaw cron add \
  --name "Nightly summary (topic)" \
  --cron "0 22 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize today; send to the nightly topic." \
  --deliver \
  --channel telegram \
  --to "-1001234567890:topic:123"
```

Isolierter Job mit Override für Modell und Denkprozess:

````bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 1" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Wöchentliche Tiefenanalyse des Projektfortschritts." \
  --model "opus" \
  --thinking high \
  --deliver \
  --channel whatsapp \
  --to "+15551234567"

Agentenauswahl (Multi-Agenten-Setups):
```bash
# Job an Agent „ops" binden (fällt auf Standard zurück, falls dieser Agent fehlt)
openclaw cron add --name "Ops sweep" --cron "0 6 * * *" --session isolated --message "Ops-Warteschlange prüfen" --agent ops

# Agent bei einem bestehenden Job wechseln oder entfernen
openclaw cron edit <jobId> --agent ops
openclaw cron edit <jobId> --clear-agent
````

````

Manueller Durchlauf (Debug):
```bash
openclaw cron run <jobId> --force
````

Bestehenden Job bearbeiten (Felder per Patch ändern):

```bash
openclaw cron edit <jobId> \
  --message "Updated prompt" \
  --model "opus" \
  --thinking low
```

Ausführungsverlauf:

```bash
openclaw cron runs --id <jobId> --limit 50
```

Direktes Systemereignis ohne Job-Erstellung:

```bash
openclaw system event --mode now --text "Next heartbeat: check battery."
```

<div id="gateway-api-surface">
  ## Gateway-API-Oberfläche
</div>

* `cron.list`, `cron.status`, `cron.add`, `cron.update`, `cron.remove`
* `cron.run` (erzwungen oder wenn fällig), `cron.runs`
  Für sofortige Systemereignisse ohne Job verwenden Sie [`openclaw system event`](/de/cli/system).

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

<div id="nothing-runs">
  ### „Nichts läuft“
</div>

* Überprüfe, ob cron aktiviert ist: `cron.enabled` und `OPENCLAW_SKIP_CRON`.
* Überprüfe, ob das Gateway durchgängig läuft (cron läuft im Gateway-Prozess).
* Bei `cron`-Zeitplänen: Überprüfe die Zeitzone (`--tz`) im Vergleich zur Host-Zeitzone.

<div id="telegram-delivers-to-the-wrong-place">
  ### Telegram liefert an den falschen Ort
</div>

* Für Foren-Topics verwende `-100…:topic:<id>`, damit es explizit und eindeutig ist.
* Wenn du `telegram:...`-Präfixe in Logs oder gespeicherten „last route“-Zielen siehst, ist das normal;
  die Cron-Auslieferung akzeptiert sie und parst Topic-IDs trotzdem korrekt.