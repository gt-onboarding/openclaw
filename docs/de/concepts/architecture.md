---
title: Architektur
summary: "WebSocket-Gateway-Architektur, Komponenten und Client-Flows"
read_when:
  - Wenn du am Gateway-Protokoll, an Clients oder Transporten arbeitest
---

<div id="gateway-architecture">
  # Gateway-Architektur
</div>

Zuletzt aktualisiert: 2026-01-22

<div id="overview">
  ## Überblick
</div>

* Ein einzelnes, langlebiges **Gateway** verwaltet alle Messaging-Kanäle (WhatsApp über
  Baileys, Telegram über grammY, Slack, Discord, Signal, iMessage, WebChat).
* Control-Plane-Clients (macOS-App, CLI, Web-UI, Automatisierungen) verbinden sich mit dem
  Gateway über **WebSocket** über den konfigurierten Bind-Host (Standardwert
  `127.0.0.1:18789`).
* **Nodes** (macOS/iOS/Android/headless) verbinden sich ebenfalls über **WebSocket**, deklarieren
  aber `role: node` mit expliziten Fähigkeiten/Befehlen.
* Ein Gateway pro Host; es ist die einzige Instanz, die eine WhatsApp-Sitzung herstellt.
* Ein **canvas host** (Standardwert `18793`) stellt von Agents bearbeitbares HTML und A2UI bereit.

<div id="components-and-flows">
  ## Komponenten und Datenflüsse
</div>

<div id="gateway-daemon">
  ### Gateway (Daemon)
</div>

* Hält Verbindungen zu Anbietern aufrecht.
* Stellt eine typisierte WS‑API bereit (mit Requests, Responses und Server‑Push‑Events).
* Validiert eingehende Frames gegen JSON‑Schema.
* Löst Events wie `agent`, `chat`, `presence`, `health`, `heartbeat`, `cron` aus.

<div id="clients-mac-app-cli-web-admin">
  ### Clients (Mac-App / CLI / Web-Admin)
</div>

* Eine WS-Verbindung pro Client.
* Anfragen senden (`health`, `status`, `send`, `agent`, `system-presence`).
* Events abonnieren (`tick`, `agent`, `presence`, `shutdown`).

<div id="nodes-macos-ios-android-headless">
  ### Knoten (macOS / iOS / Android / headless)
</div>

* Verbinden sich mit demselben **WS-Server** mit `role: node`.
* Geben in `connect` eine Geräteidentität an; die Kopplung ist **gerätebasiert** (Rolle `node`) und
  die Genehmigung wird im Gerätekopplungs-Store verwaltet.
* Stellen Befehle wie `canvas.*`, `camera.*`, `screen.record`, `location.get` bereit.

Protokolldetails:

* [Gateway protocol](/de/gateway/protocol)

<div id="webchat">
  ### WebChat
</div>

* Statische UI, die die Gateway-WS-API für Chatverlauf und Sendevorgänge verwendet.
* In Remote-Setups stellt sie die Verbindung über denselben SSH-/Tailscale-Tunnel wie andere
  Clients her.

<div id="connection-lifecycle-single-client">
  ## Lebenszyklus der Verbindung (einzelner Client)
</div>

```
Client                    Gateway
  |                          |
  |---- req:connect -------->|
  |<------ res (ok) ---------|   (or res error + close)
  |   (payload=hello-ok carries snapshot: presence + health)
  |                          |
  |<------ event:presence ---|
  |<------ event:tick -------|
  |                          |
  |------- req:agent ------->|
  |<------ res:agent --------|   (ack: {runId,status:"accepted"})
  |<------ event:agent ------|   (streaming)
  |<------ res:agent --------|   (final: {runId,status,summary})
  |                          |
```

<div id="wire-protocol-summary">
  ## Wire-Protokoll (Zusammenfassung)
</div>

* Transport: WebSocket, Textframes mit JSON-Nutzlasten.
* Der erste Frame **muss** `connect` sein.
* Nach dem Handshake:
  * Requests: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
  * Events: `{type:"event", event, payload, seq?, stateVersion?}`
* Wenn `OPENCLAW_GATEWAY_TOKEN` (oder `--token`) gesetzt ist, muss
  `connect.params.auth.token` übereinstimmen, sonst wird der Socket geschlossen.
* Idempotenzschlüssel sind für Methoden mit Seiteneffekten (`send`, `agent`)
  erforderlich, um sicheres Wiederholen zu ermöglichen; der Server hält einen kurzlebigen Deduplizierungs‑Cache vor.
* Knoten müssen `role: "node"` plus Fähigkeiten/Befehle/Berechtigungen in `connect` angeben.

<div id="pairing-local-trust">
  ## Kopplung + lokales Vertrauen
</div>

* Alle WS-Clients (Operatoren + Knoten) übermitteln beim `connect` eine **Geräteidentität**.
* Neue Geräte-IDs erfordern eine Kopplungsbestätigung; das Gateway stellt ein **Gerätetoken**
  für nachfolgende Verbindungen aus.
* **Lokale** Verbindungen (Loopback oder die eigene Tailnet-Adresse des Gateway-Hosts) können
  automatisch freigegeben werden, um die UX auf demselben Host reibungslos zu halten.
* **Nicht-lokale** Verbindungen müssen die `connect.challenge`-Nonce signieren und erfordern
  eine explizite Freigabe.
* Gateway-Auth (`gateway.auth.*`) gilt weiterhin für **alle** Verbindungen, lokale wie
  entfernte.

Details: [Gateway protocol](/de/gateway/protocol), [Pairing](/de/start/pairing),
[Security](/de/gateway/security).

<div id="protocol-typing-and-codegen">
  ## Protokoll-Typisierung und Codegenerierung
</div>

* TypeBox-Schemata definieren das Protokoll.
* Das JSON-Schema wird aus diesen Schemata generiert.
* Swift-Modelle werden aus dem JSON-Schema generiert.

<div id="remote-access">
  ## Remotezugriff
</div>

* Bevorzugt: Tailscale oder VPN.
* Alternative: SSH-Tunnel
  ```bash
  ssh -N -L 18789:127.0.0.1:18789 user@host
  ```
* Der gleiche Handshake und dasselbe Auth-Token gelten auch über den Tunnel.
* TLS und optionales Pinning können für WS in Remote-Setups aktiviert werden.

<div id="operations-snapshot">
  ## Operations-Überblick
</div>

* Start: `openclaw gateway` (im Vordergrund, gibt Logs auf stdout aus).
* Health: `health` über WS (auch in `hello-ok` enthalten).
* Überwachung: launchd/systemd für automatischen Neustart.

<div id="invariants">
  ## Invarianten
</div>

* Genau ein Gateway steuert eine einzelne Baileys-Sitzung pro Host.
* Handshake ist verpflichtend; jedes erste Frame, das weder JSON noch ein connect-Frame ist, führt zu einem harten Verbindungsabbruch.
* Events werden nicht erneut gesendet; Clients müssen bei Lücken ein Refresh durchführen.