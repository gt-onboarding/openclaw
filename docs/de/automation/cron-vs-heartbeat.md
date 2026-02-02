---
title: Cron vs Herzschlag
summary: "Anleitung zur Auswahl zwischen Herzschlag und Cron-Jobs für die Automatisierung"
read_when:
  - Wenn du entscheidest, wie wiederkehrende Aufgaben geplant werden sollen
  - Beim Einrichten von Hintergrundüberwachung oder Benachrichtigungen
  - Beim Optimieren der Token-Nutzung für periodische Prüfungen
---

<div id="cron-vs-heartbeat-when-to-use-each">
  # Cron vs Herzschlag: Wann Sie welches verwenden sollten
</div>

Sowohl Herzschläge als auch Cron-Jobs ermöglichen Ihnen, Aufgaben nach Zeitplan auszuführen. Dieser Leitfaden hilft Ihnen, den richtigen Mechanismus für Ihren Anwendungsfall zu wählen.

<div id="quick-decision-guide">
  ## Schnelle Entscheidungsübersicht
</div>

| Anwendungsfall | Empfohlen | Warum |
|----------------|-----------|-------|
| Posteingang alle 30 Minuten prüfen | Herzschlag | Wird mit anderen Checks gebündelt, kontextbewusst |
| Täglichen Bericht punktgenau um 9 Uhr senden | Cron (isoliert) | Exakte Zeitsteuerung erforderlich |
| Kalender auf anstehende Termine überwachen | Herzschlag | Natürliche Wahl für periodische Aufmerksamkeit |
| Wöchentliche Tiefenanalyse ausführen | Cron (isoliert) | Eigenständige Aufgabe, kann anderes Modell verwenden |
| Mich in 20 Minuten erinnern | Cron (main, `--at`) | Einmalige Aktion mit präzisem Timing |
| Hintergrund-Gesundheitscheck für das Projekt | Herzschlag | Nutzt bestehenden Zyklus im Hintergrund |

<div id="heartbeat-periodic-awareness">
  ## Herzschlag: Periodische Wahrnehmung
</div>

Herzschläge laufen in der **Hauptsitzung** in regelmäßigen Abständen (Standard: alle 30 Min). Sie sind dafür gedacht, dass der agent Dinge prüft und alles Wichtige sichtbar macht.

<div id="when-to-use-heartbeat">
  ### Wann du Herzschlag einsetzen solltest
</div>

* **Mehrere periodische Prüfungen**: Anstatt 5 separater Cron-Jobs, die Posteingang, Kalender, Wetter, Benachrichtigungen und Projektstatus prüfen, kann ein einzelner Herzschlag all diese Vorgänge bündeln.
* **Kontextbewusste Entscheidungen**: Der Agent hat den vollständigen Kontext der Hauptsitzung und kann daher intelligente Entscheidungen darüber treffen, was dringend ist und was warten kann.
* **Gesprächskontinuität**: Herzschlag-Ausführungen verwenden dieselbe Sitzung, sodass der Agent sich an jüngste Unterhaltungen erinnert und nahtlos anknüpfen kann.
* **Monitoring mit geringem Overhead**: Ein Herzschlag ersetzt viele kleine Polling-Aufgaben.

<div id="heartbeat-advantages">
  ### Herzschlag-Vorteile
</div>

* **Bündelt mehrere Prüfungen**: Ein einzelner Agent-Durchlauf kann Posteingang, Kalender und Benachrichtigungen gemeinsam prüfen.
* **Reduziert API-Aufrufe**: Ein einzelner Herzschlag ist günstiger als 5 isolierte Cron-Jobs.
* **Kontextbewusst**: Der agent weiß, woran du gearbeitet hast, und kann entsprechend priorisieren.
* **Intelligente Unterdrückung**: Wenn nichts Aufmerksamkeit erfordert, antwortet der agent mit `HEARTBEAT_OK` und es wird keine Nachricht zugestellt.
* **Natürliches Timing**: Driftet leicht je nach Queue-Auslastung, was für die meisten Monitoring-Szenarien unproblematisch ist.

<div id="heartbeat-example-heartbeatmd-checklist">
  ### Herzschlag-Beispiel: Checkliste HEARTBEAT.md
</div>

```md
# Herzschlag-Checkliste

- E-Mails auf dringende Nachrichten überprüfen
- Kalender auf Termine in den nächsten 2 Stunden überprüfen
- Falls eine Hintergrundaufgabe abgeschlossen wurde, Ergebnisse zusammenfassen
- Falls seit 8+ Stunden inaktiv, kurze Statusmeldung senden
```

Der agent liest dies bei jedem Herzschlag und bearbeitet alle Elemente in einem Durchlauf.

<div id="configuring-heartbeat">
  ### Heartbeat konfigurieren
</div>

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",        // Intervall
        target: "last",      // Ziel für Benachrichtigungen
        activeHours: { start: "08:00", end: "22:00" }  // optional
      }
    }
  }
}
```

Siehe [Herzschlag](/de/gateway/heartbeat) für die vollständige Konfigurationsbeschreibung.

<div id="cron-precise-scheduling">
  ## Cron: Präzise Zeitsteuerung
</div>

Cron-Jobs laufen zu **exakten Zeitpunkten** und können in isolierten Sitzungen ausgeführt werden, ohne den Hauptkontext zu beeinflussen.

<div id="when-to-use-cron">
  ### Wann du Cron verwenden solltest
</div>

* **Exaktes Timing erforderlich**: „Sende das jeden Montag um 9:00 Uhr“ (nicht „irgendwann um 9“).
* **Eigenständige Aufgaben**: Aufgaben, die keinen Gesprächskontext benötigen.
* **Anderes Modell/andere Denkweise**: Aufwendige Analysen, die den Einsatz eines leistungsfähigeren Modells rechtfertigen.
* **Einmalige Erinnerungen**: „Erinnere mich in 20 Minuten“ mit `--at`.
* **Sehr häufige Aufgaben**: Aufgaben, die den Verlauf der Hauptsitzung zumüllen würden.
* **Externe Auslöser**: Aufgaben, die unabhängig davon laufen sollen, ob der agent ansonsten aktiv ist.

<div id="cron-advantages">
  ### Cron-Vorteile
</div>

* **Exakte Zeitplanung**: Cron-Ausdrücke mit fünf Feldern und Zeitzonenunterstützung.
* **Sitzungsisolation**: Läuft in `cron:<jobId>` ohne den Hauptverlauf zu beeinflussen.
* **Modell-Overrides**: Pro Job ein günstigeres oder leistungsstärkeres Modell verwenden.
* **Lieferkontrolle**: Kann direkt an einen Kanal zustellen; postet standardmäßig trotzdem eine Zusammenfassung in den Hauptkanal (konfigurierbar).
* **Kein Agent-Kontext nötig**: Läuft auch, wenn die Hauptsitzung inaktiv oder komprimiert ist.
* **One-Shot-Unterstützung**: `--at` für exakte zukünftige Zeitpunkte.

<div id="cron-example-daily-morning-briefing">
  ### Cron-Beispiel: Tägliches Morgenbriefing
</div>

```bash
openclaw cron add \
  --name "Morning briefing" \
  --cron "0 7 * * *" \
  --tz "America/New_York" \
  --session isolated \
  --message "Generate today's briefing: weather, calendar, top emails, news summary." \
  --model opus \
  --deliver \
  --channel whatsapp \
  --to "+15551234567"
```

Dies wird genau um 7:00 Uhr morgens New Yorker Zeit ausgeführt, verwendet Opus für hohe Qualität und wird direkt an WhatsApp zugestellt.

<div id="cron-example-one-shot-reminder">
  ### Cron-Beispiel: Einmalige Erinnerung
</div>

```bash
openclaw cron add \
  --name "Meeting reminder" \
  --at "20m" \
  --session main \
  --system-event "Reminder: standup meeting starts in 10 minutes." \
  --wake now \
  --delete-after-run
```

Siehe [Cron-Jobs](/de/automation/cron-jobs) für die vollständige CLI-Referenz.

<div id="decision-flowchart">
  ## Entscheidungsdiagramm
</div>

```
Does the task need to run at an EXACT time?
  YES -> Use cron
  NO  -> Continue...

Does the task need isolation from main session?
  YES -> Use cron (isolated)
  NO  -> Continue...

Can this task be batched with other periodic checks?
  YES -> Use heartbeat (add to HEARTBEAT.md)
  NO  -> Use cron

Is this a one-shot reminder?
  YES -> Use cron with --at
  NO  -> Continue...

Does it need a different model or thinking level?
  YES -> Use cron (isolated) with --model/--thinking
  NO  -> Use heartbeat
```

<div id="combining-both">
  ## Beides kombinieren
</div>

Die effizienteste Einrichtung verwendet **beides**:

1. **Herzschlag** übernimmt die routinemäßige Überwachung (Posteingang, Kalender, Benachrichtigungen) in einem gebündelten Durchlauf alle 30 Minuten.
2. **Cron** übernimmt präzise Zeitpläne (tägliche Berichte, wöchentliche Auswertungen) und einmalige Erinnerungen.

<div id="example-efficient-automation-setup">
  ### Beispiel: Effiziente Automatisierung einrichten
</div>

**HEARTBEAT.md** (wird alle 30 Minuten überprüft):

```md
# Herzschlag-Checkliste
- Posteingang nach dringenden E-Mails durchsuchen
- Kalender auf Termine in den nächsten 2 Stunden prüfen
- Ausstehende Aufgaben überprüfen
- Kurze Statusprüfung bei Inaktivität von über 8 Stunden
```

**Cronjobs** (exakte Zeitsteuerung):

```bash
# Daily morning briefing at 7am
openclaw cron add --name "Morning brief" --cron "0 7 * * *" --session isolated --message "..." --deliver

# Wöchentliche Projektbesprechung montags um 9 Uhr
openclaw cron add --name "Weekly review" --cron "0 9 * * 1" --session isolated --message "..." --model opus

# One-shot reminder
openclaw cron add --name "Call back" --at "2h" --session main --system-event "Call back the client" --wake now
```

<div id="lobster-deterministic-workflows-with-approvals">
  ## Lobster: Deterministische Workflows mit Genehmigungen
</div>

Lobster ist die Workflow-Runtime für **mehrstufige Tool-Pipelines**, die eine deterministische Ausführung und explizite Genehmigungen benötigen.
Verwende sie, wenn die Aufgabe mehr als einen einzelnen Turn eines agents umfasst und du einen wiederaufnehmbaren Workflow mit menschlichen Kontrollpunkten möchtest.

<div id="when-lobster-fits">
  ### Wann Lobster sinnvoll ist
</div>

* **Mehrstufige Automatisierung**: Du brauchst eine feste Pipeline von Tool-Aufrufen statt eines einmaligen Prompts.
* **Genehmigungs-Gates**: Seiteneffekte sollen pausieren, bis du sie genehmigst, und dann fortgesetzt werden.
* **Wiederaufnehmbare Ausführungen**: Einen pausierten Workflow fortsetzen, ohne frühere Schritte erneut auszuführen.

<div id="how-it-pairs-with-heartbeat-and-cron">
  ### Wie es mit Herzschlag und Cron zusammenarbeitet
</div>

* **Herzschlag/Cron** entscheiden, *wann* ein Durchlauf stattfindet.
* **Lobster** definiert, *welche Schritte* ausgeführt werden, sobald der Durchlauf beginnt.

Für geplante Workflows verwendest du Cron oder Herzschlag, um einen agent-Turn auszulösen, der Lobster aufruft.
Für Ad-hoc-Workflows rufst du Lobster direkt auf.

<div id="operational-notes-from-the-code">
  ### Hinweise zum Betrieb (aus dem Code)
</div>

* Lobster läuft als **lokaler Subprozess** (`lobster` CLI) im Tool-Modus und gibt einen **JSON-Envelope** zurück.
* Wenn das Tool `needs_approval` zurückgibt, setzt du mit einem `resumeToken` und dem `approve`-Flag fort.
* Das Tool ist ein **optionales Plugin**; aktiviere es zusätzlich über `tools.alsoAllow: ["lobster"]` (empfohlen).
* Wenn du `lobsterPath` übergibst, muss es ein **absoluter Pfad** sein.

Siehe [Lobster](/de/tools/lobster) für vollständige Verwendung und Beispiele.

<div id="main-session-vs-isolated-session">
  ## Hauptsitzung vs isolierte Sitzung
</div>

Sowohl Herzschlag als auch Cron können mit der Hauptsitzung interagieren, aber auf unterschiedliche Weise:

| | Herzschlag | Cron (Haupt) | Cron (isoliert) |
|---|---|---|---|
| Sitzung | Haupt | Haupt (per Systemereignis) | `cron:<jobId>` |
| Verlauf | Gemeinsam | Gemeinsam | Neu bei jedem Lauf |
| Kontext | Vollständig | Vollständig | Kein Kontext (startet leer) |
| Modell | Modell der Hauptsitzung | Modell der Hauptsitzung | Kann überschrieben werden |
| Ausgabe | Zugestellt, sofern nicht `HEARTBEAT_OK` | Herzschlag-Prompt + Ereignis | Zusammenfassung in der Hauptsitzung gepostet |

<div id="when-to-use-main-session-cron">
  ### Wann du Cron mit der Hauptsitzung verwenden solltest
</div>

Verwende `--session main` mit `--system-event`, wenn du möchtest:

* Die Erinnerung/das Ereignis im Kontext der Hauptsitzung haben
* Dass der agent es beim nächsten Herzschlag mit vollem Kontext verarbeitet
* Keinen separaten, isolierten Lauf

```bash
openclaw cron add \
  --name "Check project" \
  --every "4h" \
  --session main \
  --system-event "Time for a project health check" \
  --wake now
```

<div id="when-to-use-isolated-cron">
  ### Wann du isolierten Cron verwenden solltest
</div>

Verwende `--session isolated`, wenn du Folgendes willst:

* Eine saubere Ausgangsbasis ohne vorherigen Kontext
* Andere Modell- oder Thinking-Einstellungen
* Ausgabe, die direkt an einen Kanal gesendet wird (die Zusammenfassung wird standardmäßig weiterhin im Hauptkanal veröffentlicht)
* Verlauf, der die Hauptsitzung nicht überfrachtet

```bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 0" \
  --session isolated \
  --message "Weekly codebase analysis..." \
  --model opus \
  --thinking high \
  --deliver
```

<div id="cost-considerations">
  ## Kostenüberlegungen
</div>

| Mechanismus | Kostenprofil |
|-----------|--------------|
| Herzschlag | Eine Interaktion alle N Minuten; skaliert mit der Größe von HEARTBEAT.md |
| Cron (main) | Fügt ein Ereignis zum nächsten Herzschlag hinzu (keine isolierte Interaktion) |
| Cron (isolated) | Voller agent-Durchlauf pro Job; kann ein günstigeres Modell verwenden |

**Tipps**:

* Halte `HEARTBEAT.md` klein, um den Token-Overhead zu minimieren.
* Fasse ähnliche Prüfungen im Herzschlag zusammen, anstatt mehrere Cron-Jobs zu verwenden.
* Verwende `target: "none"` beim Herzschlag, wenn du nur interne Verarbeitung möchtest.
* Verwende isolierte Cron-Jobs mit einem günstigeren Modell für Routineaufgaben.

<div id="related">
  ## Verwandt
</div>

* [Heartbeat](/de/gateway/heartbeat) - vollständige Herzschlag-Konfiguration
* [Cron jobs](/de/automation/cron-jobs) - vollständige Referenz zur Cron-CLI und API
* [System](/de/cli/system) - Systemereignisse und Herzschlag-Steuerung