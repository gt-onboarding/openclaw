---
title: Knoten
summary: "CLI-Referenz für `openclaw nodes` (list/status/approve/invoke, camera/canvas/screen)"
read_when:
  - Du verwaltest verbundene Knoten (Kameras, Bildschirm, Canvas)
  - Du musst Anfragen genehmigen oder Knotenbefehle ausführen
---

<div id="openclaw-nodes">
  # `openclaw nodes`
</div>

Verwalte gekoppelte Knoten (Geräte) und rufe Knoten-Fähigkeiten auf.

Verwandte Themen:

* Knoten-Übersicht: [Nodes](/de/nodes)
* Kamera: [Kamera-Knoten](/de/nodes/camera)
* Bilder: [Bild-Knoten](/de/nodes/images)

Allgemeine Optionen:

* `--url`, `--token`, `--timeout`, `--json`

<div id="common-commands">
  ## Häufig genutzte Befehle
</div>

```bash
openclaw nodes list
openclaw nodes list --connected
openclaw nodes list --last-connected 24h
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes status
openclaw nodes status --connected
openclaw nodes status --last-connected 24h
```

`nodes list` gibt Tabellen mit ausstehenden/gepaarten Knoten aus. Gepaarte Zeilen enthalten die Zeit seit der letzten Verbindung (Last Connect).
Verwende `--connected`, um nur aktuell verbundene Knoten anzuzeigen. Verwende `--last-connected <duration>`, um
auf Knoten zu filtern, die innerhalb eines bestimmten Zeitraums eine Verbindung hergestellt haben (z. B. `24h`, `7d`).

<div id="invoke-run">
  ## Aufruf / Ausführung
</div>

```bash
openclaw nodes invoke --node <id|name|ip> --command <command> --params <json>
openclaw nodes run --node <id|name|ip> <command...>
openclaw nodes run --raw "git status"
openclaw nodes run --agent main --node <id|name|ip> --raw "git status"
```

Invoke-Flags:

* `--params <json>`: JSON-Objekt-String (Standardwert `{}`).
* `--invoke-timeout <ms>`: Knoten-Invoke-Timeout (Standardwert `15000`).
* `--idempotency-key <key>`: optionaler Idempotenzschlüssel.

<div id="exec-style-defaults">
  ### Exec-Standardwerte
</div>

`nodes run` spiegelt das Exec-Verhalten des Modells (Standardwerte + Genehmigungen) wider:

* Liest `tools.exec.*` (plus `agents.list[].tools.exec.*`-Overrides).
* Verwendet Exec-Genehmigungen (`exec.approval.request`), bevor `system.run` aufgerufen wird.
* `--node` kann weggelassen werden, wenn `tools.exec.node` gesetzt ist.
* Erfordert einen Knoten, der `system.run` bereitstellt (macOS-Companion-App oder Headless-Knoten-Host).

Flags:

* `--cwd <path>`: Arbeitsverzeichnis.
* `--env <key=val>`: Überschreibung von Umgebungsvariablen (wiederholbar).
* `--command-timeout <ms>`: Befehls-Timeout.
* `--invoke-timeout <ms>`: Knoten-Aufruf-Timeout (Standard `30000`).
* `--needs-screen-recording`: Bildschirmaufnahmeberechtigung erforderlich.
* `--raw <command>`: Führt einen Shell-String aus (`/bin/sh -lc` oder `cmd.exe /c`).
* `--agent <id>`: agent-bezogene Genehmigungen/Allowlist-Einträge (Standard ist der konfigurierte agent).
* `--ask <off|on-miss|always>`, `--security <deny|allowlist|full>`: Überschreibungen.