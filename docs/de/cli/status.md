---
title: Status
summary: "CLI-Referenz für `openclaw status` (Diagnose, Prüfungen, Nutzungs-Snapshots)"
read_when:
  - Du möchtest eine schnelle Diagnose des Kanalzustands und der Empfänger letzter Sitzungen
  - Du möchtest einen leicht einfügbaren „all“-Status für das Debugging
---

<div id="openclaw-status">
  # `openclaw status`
</div>

Diagnoseinformationen für Kanäle und Sitzungen.

```bash
openclaw status
openclaw status --all
openclaw status --deep
openclaw status --usage
```

Hinweise:

* `--deep` führt Live-Checks aus (WhatsApp Web + Telegram + Discord + Google Chat + Slack + Signal).
* Die Ausgabe enthält Sitzungsspeicher für jeden Agenten, wenn mehrere Agenten konfiguriert sind.
* Die Übersicht enthält, wenn verfügbar, den Installations- und Laufzeitstatus des Gateway- und Knoten-Hostdienstes.
* Die Übersicht enthält Update-Kanal und Git-SHA (für Source-Checkouts).
* Update-Informationen werden in der Übersicht angezeigt; wenn ein Update verfügbar ist, gibt `openclaw status` einen Hinweis aus, `openclaw update` auszuführen (siehe [Updating](/de/install/updating)).
