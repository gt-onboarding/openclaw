---
title: Cron
summary: "CLI-Referenz für `openclaw cron` (Hintergrundjobs planen und ausführen)"
read_when:
  - Du geplante Jobs und Wakeups benötigst
  - Du die Ausführung und Logs von Cron-Jobs debuggen möchtest
---

<div id="openclaw-cron">
  # `openclaw cron`
</div>

Verwalte Cron-Jobs für den Gateway-Scheduler.

Verwandt:

* Cron-Jobs: [Cron jobs](/de/automation/cron-jobs)

Tipp: Führe `openclaw cron --help` aus, um alle verfügbaren Befehle und Optionen anzuzeigen.

<div id="common-edits">
  ## Häufige Änderungen
</div>

Aktualisiere die Zustellungseinstellungen, ohne die Nachricht selbst zu ändern:

```bash
openclaw cron edit <job-id> --deliver --channel telegram --to "123456789"
```

Zustellung für einen isolierten Job deaktivieren:

```bash
openclaw cron edit <job-id> --no-deliver
```
