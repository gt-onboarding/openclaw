---
title: Webchat
summary: "Loopback-WebChat-Static-Hosting und Gateway-WS-Verwendung für die Chat-UI"
read_when:
  - Fehlerbehebung oder Konfiguration des WebChat-Zugriffs
---

<div id="webchat-gateway-websocket-ui">
  # WebChat (Gateway WebSocket UI)
</div>

Status: Die macOS-/iOS-SwiftUI-Chat-UI kommuniziert direkt mit dem Gateway-WebSocket.

<div id="what-it-is">
  ## Was es ist
</div>

* Eine native Chat-UI für das Gateway (kein eingebetteter Browser und kein lokaler statischer Server).
* Verwendet die gleichen Sitzungen und Routing-Regeln wie andere Kanäle.
* Deterministisches Routing: Antworten werden immer an WebChat zurückgesendet.

<div id="quick-start">
  ## Schnellstart
</div>

1. Starte das Gateway.
2. Öffne die WebChat UI (macOS/iOS-App) oder den Chat-Tab der Control UI.
3. Stelle sicher, dass die Gateway-Authentifizierung konfiguriert ist (standardmäßig erforderlich, auch auf der Loopback-Schnittstelle).

<div id="how-it-works-behavior">
  ## Funktionsweise (Verhalten)
</div>

* Die UI verbindet sich mit dem Gateway-WebSocket und verwendet `chat.history`, `chat.send` und `chat.inject`.
* `chat.inject` fügt eine Assistenten-Notiz direkt zum Transkript hinzu und überträgt sie an die UI (kein Agent-Durchlauf).
* Der Verlauf wird immer vom Gateway abgerufen (keine lokale Dateiüberwachung).
* Wenn das Gateway nicht erreichbar ist, ist WebChat schreibgeschützt.

<div id="remote-use">
  ## Remote-Betrieb
</div>

* Der Remote-Modus tunnelt den Gateway-WebSocket über SSH oder Tailscale.
* Du musst keinen eigenständigen WebChat-Server betreiben.

<div id="configuration-reference-webchat">
  ## Konfigurationsreferenz (WebChat)
</div>

Vollständige Konfiguration: [Konfiguration](/de/gateway/configuration)

Kanaloptionen:

* Kein eigener `webchat.*`-Block. WebChat verwendet den Gateway-Endpunkt und die untenstehenden Auth-Einstellungen.

Zugehörige globale Optionen:

* `gateway.port`, `gateway.bind`: WebSocket-Host/-Port.
* `gateway.auth.mode`, `gateway.auth.token`, `gateway.auth.password`: WebSocket-Authentifizierung.
* `gateway.remote.url`, `gateway.remote.token`, `gateway.remote.password`: Ziel des entfernten Gateways.
* `session.*`: Sitzungsspeicherung und Hauptschlüssel-Standardwerte.