---
title: Webchat
summary: "Wie die macOS-App den Gateway-WebChat einbettet und wie du sie debuggen kannst"
read_when:
  - Debugging der WebChat-Ansicht der macOS-App oder des Loopback-Ports
---

<div id="webchat-macos-app">
  # WebChat (macOS-App)
</div>

Die macOS-Menüleisten-App bettet die WebChat-UI als native SwiftUI-Ansicht ein. Sie
verbindet sich mit dem Gateway und verwendet standardmäßig die **Hauptsitzung** für den ausgewählten
Agenten (mit einem Sitzungsumschalter für andere Sitzungen).

- **Lokaler Modus**: verbindet sich direkt mit dem lokalen Gateway-WebSocket.
- **Remote-Modus**: leitet den Gateway-Control-Port über SSH weiter und nutzt diesen
  Tunnel als Datenpfad.

<div id="launch-debugging">
  ## Starten & Debugging
</div>

- Manuell: Lobster-Menü → „Open Chat“.
- Automatisches Öffnen für Tests:
  ```bash
  dist/OpenClaw.app/Contents/MacOS/OpenClaw --webchat
  ```
- Logs: `./scripts/clawlog.sh` (Subsystem `bot.molt`, Kategorie `WebChatSwiftUI`).

<div id="how-its-wired">
  ## Wie es angebunden ist
</div>

- Datenebene: Gateway-WS-Methoden `chat.history`, `chat.send`, `chat.abort`,
  `chat.inject` und Events `chat`, `agent`, `presence`, `tick`, `health`.
- Sitzung: Standardmäßig wird die primäre Sitzung verwendet (`main` oder `global`,
  wenn Scope global ist). Die UI kann zwischen Sitzungen wechseln.
- Onboarding verwendet eine eigene Sitzung, um die Ersteinrichtung getrennt zu halten.

<div id="security-surface">
  ## Angriffsfläche
</div>

- Der Remote-Modus leitet nur den WebSocket-Steuerport des Gateways über SSH weiter.

<div id="known-limitations">
  ## Bekannte Einschränkungen
</div>

- Die UI ist auf Chat-Sitzungen optimiert (keine vollständige Browser-Sandbox).