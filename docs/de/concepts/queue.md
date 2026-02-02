---
title: Warteschlange
summary: "Design der Befehlswarteschlange zur Serialisierung eingehender Auto-Reply-Durchläufe"
read_when:
  - Beim Ändern der Auto-Reply-Ausführung oder -Parallelität
---

<div id="command-queue-2026-01-16">
  # Command Queue (2026-01-16)
</div>

Wir serialisieren eingehende Auto-Reply-Ausführungen (alle Kanäle) über eine kleine In-Process-Warteschlange, um zu verhindern, dass mehrere agent-Ausführungen miteinander kollidieren, während wir gleichzeitig eine sichere Parallelisierung zwischen Sitzungen ermöglichen.

<div id="why">
  ## Warum
</div>

- Auto-Reply-Durchläufe können kostspielig sein (LLM-Aufrufe) und in Konflikt geraten, wenn mehrere eingehende Nachrichten kurz nacheinander ankommen.
- Serialisierung vermeidet Konkurrenz um gemeinsame Ressourcen (Sitzungsdateien, Logs, CLI-stdin) und verringert die Wahrscheinlichkeit von Upstream-Rate-Limits.

<div id="how-it-works">
  ## Funktionsweise
</div>

- Eine lane-bewusste FIFO-Warteschlange leert jede Lane mit einer konfigurierbaren Parallelitätsbegrenzung (Standard 1 für nicht konfigurierte Lanes; `main` standardmäßig 4, `subagent` 8).
- `runEmbeddedPiAgent` reiht nach **Sitzungsschlüssel** in die Warteschlange ein (Lane `session:<key>`), um nur eine aktive Ausführung pro Sitzung zu garantieren.
- Jeder Sitzungslauf wird anschließend in eine **globale Lane** (`main` standardmäßig) eingereiht, sodass die Gesamtparallelität durch `agents.defaults.maxConcurrent` begrenzt ist.
- Wenn ausführliches Logging aktiviert ist, geben eingereihte Läufe einen kurzen Hinweis aus, falls sie vor dem Start mehr als ~2 s gewartet haben.
- Typing-Indikatoren werden beim Einreihen weiterhin sofort ausgelöst (sofern vom Kanal unterstützt), sodass die User Experience unverändert bleibt, während auf die Ausführung gewartet wird.

<div id="queue-modes-per-channel">
  ## Queue-Modi (pro Kanal)
</div>

Eingehende Nachrichten können den aktuellen Run steuern, auf einen nachfolgenden Turn warten oder beides:

* `steer`: sofort in den aktuellen Run einfügen (bricht ausstehende Tool-Aufrufe nach der nächsten Tool-Grenze ab). Wenn kein Streaming aktiv ist, Fallback auf `followup`.
* `followup`: für den nächsten agent-Turn nach Ende des aktuellen Runs in die Queue einreihen.
* `collect`: alle eingereihten Nachrichten zu einem **einzigen** Followup-Turn zusammenführen (Standard). Wenn Nachrichten auf verschiedene Kanäle/Threads zielen, werden sie einzeln abgearbeitet, um das Routing zu bewahren.
* `steer-backlog` (alias `steer+backlog`): jetzt steuern **und** die Nachricht für einen Followup-Turn aufbewahren.
* `interrupt` (veraltet): den aktiven Run für diese Sitzung abbrechen und dann die neueste Nachricht ausführen.
* `queue` (veraltetes Alias): dasselbe wie `steer`.

Steer-backlog bedeutet, dass du eine Followup-Antwort nach dem gesteuerten Run erhalten kannst, sodass
Streaming-Oberflächen wie Duplikate wirken können. Bevorzuge `collect`/`steer`, wenn du
eine Antwort pro eingehender Nachricht möchtest.
Sende `/queue collect` als eigenständigen Befehl (pro Sitzung) oder setze `messages.queue.byChannel.discord: "collect"`.

Standardwerte (wenn in der Konfiguration nicht gesetzt):

* Alle Oberflächen → `collect`

Konfiguriere global oder pro Kanal über `messages.queue`:

```json5
{
  messages: {
    queue: {
      mode: "collect",
      debounceMs: 1000,
      cap: 20,
      drop: "summarize",
      byChannel: { discord: "collect" }
    }
  }
}
```


<div id="queue-options">
  ## Warteschlangenoptionen
</div>

Diese Optionen gelten für `followup`, `collect` und `steer-backlog` (und für `steer`, wenn es auf `followup` zurückfällt):

- `debounceMs`: warte, bis es ruhig ist, bevor ein Follow-up-Schritt gestartet wird (verhindert „weiter, weiter“).
- `cap`: maximale Anzahl wartender Nachrichten pro Sitzung.
- `drop`: Überlaufstrategie (`old`, `new`, `summarize`).

`summarize` führt eine kurze Aufzählung der verworfenen Nachrichten und fügt sie als synthetischen Follow-up-Prompt ein.
Standardwerte: `debounceMs: 1000`, `cap: 20`, `drop: summarize`.

<div id="per-session-overrides">
  ## Sitzungsspezifische Überschreibungen
</div>

- Sende `/queue <mode>` als eigenständigen Befehl, um den Modus für die aktuelle Sitzung zu speichern.
- Optionen können kombiniert werden: `/queue collect debounce:2s cap:25 drop:summarize`
- `/queue default` oder `/queue reset` löscht die sitzungsspezifische Überschreibung.

<div id="scope-and-guarantees">
  ## Scope und Garantien
</div>

- Gilt für Auto-Reply-Agent-Ausführungen über alle eingehenden Kanäle hinweg, die die Gateway-Reply-Pipeline verwenden (WhatsApp Web, Telegram, Slack, Discord, Signal, iMessage, Webchat usw.).
- Die Standard-Lane (`main`) ist prozessweit für eingehende Nachrichten und Haupt-Herzschläge; setze `agents.defaults.maxConcurrent`, um mehrere Sitzungen parallel zuzulassen.
- Zusätzliche Lanes können existieren (z. B. `cron`, `subagent`), sodass Hintergrundjobs parallel laufen können, ohne eingehende Antworten zu blockieren.
- Sitzungsbezogene Lanes garantieren, dass jeweils nur eine Agent-Ausführung eine bestimmte Sitzung gleichzeitig verarbeitet.
- Keine externen Abhängigkeiten oder Hintergrund-Worker-Threads; reines TypeScript + Promises.

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

- Wenn Befehle zu hängen scheinen, aktiviere ausführliche Logs und suche nach Zeilen mit „queued for …ms“, um zu überprüfen, dass die Warteschlange abgebaut wird.
- Wenn du die Tiefe der Warteschlange prüfen musst, aktiviere ausführliche Logs und achte auf Zeilen mit Zeitangaben zur Warteschlange.