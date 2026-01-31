---
title: Rpc
summary: "RPC-Adapter für externe CLIs (signal-cli, imsg) und Gateway-Pattern"
read_when:
  - Hinzufügen oder Anpassen externer CLI-Integrationen
  - Debuggen von RPC-Adaptern (signal-cli, imsg)
---

<div id="rpc-adapters">
  # RPC-Adapter
</div>

OpenClaw integriert externe CLIs über JSON-RPC. Derzeit werden zwei Muster verwendet.

<div id="pattern-a-http-daemon-signal-cli">
  ## Pattern A: HTTP-Dienst (signal-cli)
</div>

* `signal-cli` läuft als Dienst mit JSON-RPC über HTTP.
* Der Ereignisstream erfolgt per SSE (`/api/v1/events`).
* Health-Check: `/api/v1/check`.
* OpenClaw verwaltet den Lebenszyklus, wenn `channels.signal.autoStart=true`.

Siehe [Signal](/de/channels/signal) für Einrichtung und Endpunkte.

<div id="pattern-b-stdio-child-process-imsg">
  ## Pattern B: stdio-Kindprozess (imsg)
</div>

* OpenClaw startet `imsg rpc` als Kindprozess.
* JSON-RPC läuft zeilenbasiert über stdin/stdout (ein JSON-Objekt pro Zeile).
* Kein TCP-Port, kein Daemon erforderlich.

Verwendete Kernmethoden:

* `watch.subscribe` → Benachrichtigungen (`method: "message"`)
* `watch.unsubscribe`
* `send`
* `chats.list` (Probe/Diagnosezwecke)

Siehe [iMessage](/de/channels/imessage) für Einrichtung und Adressierung (`chat_id` bevorzugt).

<div id="adapter-guidelines">
  ## Adapter-Richtlinien
</div>

* Gateway verwaltet den Prozess (Start/Stopp ist an den Anbieter-Lebenszyklus gebunden).
* Stelle robuste RPC-Clients sicher: Timeouts, Neustart bei Beendigung.
* Bevorzuge stabile IDs (z. B. `chat_id`) gegenüber Anzeigenamen.