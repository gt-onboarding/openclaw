---
title: Hintergrundprozess
summary: "Ausführung von Background Exec und Prozessverwaltung"
read_when:
  - Hinzufügen oder Ändern des Background-Exec-Verhaltens
  - Debuggen von lange laufenden exec-Aufgaben
---

<div id="background-exec-process-tool">
  # Hintergrund-Exec- und -Prozess-Tool
</div>

OpenClaw führt Shell-Befehle über das `exec`-Tool aus und hält langlaufende Aufgaben im Speicher. Das `process`-Tool verwaltet diese Hintergrundsitzungen.

<div id="exec-tool">
  ## exec-Tool
</div>

Wichtige Parameter:

- `command` (erforderlich)
- `yieldMs` (Standard 10000): automatisches Auslagern in den Hintergrund nach dieser Verzögerung
- `background` (bool): sofort in den Hintergrund
- `timeout` (Sekunden, Standard 1800): Prozess nach diesem Timeout beenden
- `elevated` (bool): auf dem Host ausführen, wenn erhöhter Modus aktiviert/erlaubt ist
- Benötigst du ein echtes TTY? Setze `pty: true`.
- `workdir`, `env`

Verhalten:

- Ausführungen im Vordergrund geben die Ausgabe direkt zurück.
- Wenn in den Hintergrund verschoben (explizit oder durch Timeout), gibt das Tool `status: "running"` + `sessionId` und einen kurzen Tail der Ausgabe zurück.
- Die Ausgabe wird im Speicher gehalten, bis die Sitzung abgefragt oder geleert wird.
- Wenn das `process`-Tool nicht erlaubt ist, läuft `exec` synchron und ignoriert `yieldMs`/`background`.

<div id="child-process-bridging">
  ## Bridging von Child-Prozessen
</div>

Wenn du langlaufende Child-Prozesse außerhalb der exec/process-Tools spawnst (zum Beispiel CLI-Respawns oder Gateway-Helper), hänge den Child-Process-Bridge-Helper an, damit Beendigungssignale weitergeleitet und Listener bei Exit/Error wieder abgehängt werden. Das verhindert verwaiste Prozesse unter systemd und hält das Shutdown-Verhalten plattformübergreifend konsistent.

Environment-Overrides:

- `PI_BASH_YIELD_MS`: Standard-Yield (ms)
- `PI_BASH_MAX_OUTPUT_CHARS`: In‑Memory-Output-Limit (Zeichen)
- `OPENCLAW_BASH_PENDING_MAX_OUTPUT_CHARS`: Pending-stdout/stderr-Limit pro Stream (Zeichen)
- `PI_BASH_JOB_TTL_MS`: TTL für abgeschlossene Sitzungen (ms, begrenzt auf 1 min–3 h)

Config (bevorzugt):

- `tools.exec.backgroundMs` (Standard 10000)
- `tools.exec.timeoutSec` (Standard 1800)
- `tools.exec.cleanupMs` (Standard 1800000)
 - `tools.exec.notifyOnExit` (Standard true): reiht ein System-Event ein und fordert einen Herzschlag an, wenn ein im Hintergrund laufender exec beendet wird.

<div id="process-tool">
  ## Prozess-Tool
</div>

Aktionen:

- `list`: laufende + beendete Sitzungen
- `poll`: neue Ausgaben für eine Sitzung abrufen (meldet auch den Exit-Status)
- `log`: die aggregierte Ausgabe lesen (unterstützt `offset` + `limit`)
- `write`: stdin senden (`data`, optional `eof`)
- `kill`: eine Hintergrundsitzung beenden
- `clear`: eine beendete Sitzung aus dem Speicher entfernen
- `remove`: beenden, falls laufend, sonst löschen, falls bereits beendet

Hinweise:

- Nur im Hintergrund ausgeführte Sitzungen werden im Speicher aufgelistet und vorgehalten.
- Sitzungen gehen beim Neustart des Prozesses verloren (keine Persistenz auf der Festplatte).
- Sitzungslogs werden nur im Chatverlauf gespeichert, wenn du `process poll/log` ausführst und das Tool-Ergebnis aufgezeichnet wird.
- `process` ist pro Agent isoliert; es sieht nur Sitzungen, die von diesem Agent gestartet wurden.
- `process list` enthält einen abgeleiteten `name` (Befehlsverb + Ziel) für eine schnelle Übersicht.
- `process log` verwendet zeilenbasiertes `offset`/`limit` (lasse `offset` weg, um die letzten N Zeilen abzurufen).

<div id="examples">
  ## Beispiele
</div>

Eine lang laufende Aufgabe starten und später erneut abfragen:

```json
{"tool": "exec", "command": "sleep 5 && echo done", "yieldMs": 1000}
```

```json
{"tool": "process", "action": "poll", "sessionId": "<id>"}
```

Sofort im Hintergrund starten:

```json
{"tool": "exec", "command": "npm run build", "background": true}
```

Stdin senden:

```json
{"tool": "process", "action": "write", "sessionId": "<id>", "data": "y\n"}
```
