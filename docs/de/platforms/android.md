---
title: Android
summary: "Android-App (Knoten): Runbook zur Verbindung + Canvas/Chat/Kamera"
read_when:
  - Kopplung oder erneute Verbindung des Android-Knotens
  - Fehlerbehebung bei der Android-Gateway-Erkennung oder -Authentifizierung
  - Überprüfen, ob der Chatverlauf auf allen Clients übereinstimmt
---

<div id="android-app-node">
  # Android-App (Knoten)
</div>

<div id="support-snapshot">
  ## Support-Snapshot
</div>

* Rolle: Companion-Knoten-App (Android hostet das Gateway nicht).
* Gateway erforderlich: ja (auf macOS, Linux oder unter Windows via WSL2 ausführen).
* Installation: [Erste Schritte](/de/start/getting-started) + [Kopplung](/de/gateway/pairing).
* Gateway: [Runbook](/de/gateway) + [Konfiguration](/de/gateway/configuration).
  * Protokolle: [Gateway-Protokoll](/de/gateway/protocol) (Knoten + Control Plane).

<div id="system-control">
  ## Systemsteuerung
</div>

Die Systemsteuerung (`launchd`/`systemd`) befindet sich auf dem Gateway-Host. Siehe [Gateway](/de/gateway).

<div id="connection-runbook">
  ## Verbindungs-Runbook
</div>

Android-Knoten-App ⇄ (mDNS/NSD + WebSocket) ⇄ **Gateway**

Android verbindet sich direkt mit dem Gateway-WebSocket (Standard `ws://<host>:18789`) und verwendet die vom Gateway verwaltete Kopplung.

<div id="prerequisites">
  ### Voraussetzungen
</div>

* Du kannst das Gateway auf der „Master“-Maschine ausführen.
* Das Android-Gerät bzw. der Emulator kann den WebSocket des Gateways erreichen:
  * Im selben LAN mit mDNS/NSD, **oder**
  * Im selben Tailscale-Tailnet mithilfe von Wide-Area Bonjour / Unicast DNS-SD (siehe unten), **oder**
  * Manuell konfigurierter Gateway-Host/-Port (Fallback)
* Du kannst die CLI (`openclaw`) auf der Gateway-Maschine (oder per SSH) ausführen.

<div id="1-start-the-gateway">
  ### 1) Starte das Gateway
</div>

```bash
openclaw gateway --port 18789 --verbose
```

Prüfe in den Logs, dass du eine Zeile siehst wie:

* `listening on ws://0.0.0.0:18789`

Für reine Tailnet-Setups (empfohlen für Wien ⇄ London) binde das Gateway an die Tailnet-IP:

* Setze `gateway.bind: "tailnet"` in `~/.openclaw/openclaw.json` auf dem Gateway-Host.
* Starte das Gateway bzw. die macOS-Menüleisten-App neu.

<div id="2-verify-discovery-optional">
  ### 2) Discovery überprüfen (optional)
</div>

Vom Gateway-Host aus:

```bash
dns-sd -B _openclaw-gw._tcp local.
```

Weitere Hinweise zum Debugging: [Bonjour](/de/gateway/bonjour).

<div id="tailnet-vienna-london-discovery-via-unicast-dns-sd">
  #### Tailnet-Discovery (Wien ⇄ London) über Unicast-DNS-SD
</div>

Die Android-NSD/mDNS-Discovery funktioniert nicht über Netzwerkgrenzen hinweg. Wenn sich dein Android-Knoten und das Gateway in unterschiedlichen Netzwerken befinden, aber über Tailscale verbunden sind, verwende stattdessen Wide-Area Bonjour / Unicast-DNS-SD:

1. Richte auf dem Gateway-Host eine DNS-SD-Zone (Beispiel `openclaw.internal.`) ein und veröffentliche `_openclaw-gw._tcp`-Records.
2. Konfiguriere Tailscale Split-DNS für deine gewählte Domain so, dass sie auf diesen DNS-Server verweist.

Details und Beispielkonfiguration für CoreDNS: [Bonjour](/de/gateway/bonjour).

<div id="3-connect-from-android">
  ### 3) Von Android aus verbinden
</div>

In der Android-App:

* Die App hält ihre Gateway-Verbindung über einen **Foreground-Service** (dauerhafte Benachrichtigung) aktiv.
* Öffne **Settings**.
* Unter **Discovered Gateways** wählst du dein Gateway aus und tippst auf **Connect**.
* Wenn mDNS blockiert ist, verwende **Advanced → Manual Gateway** (Host + Port) und **Connect (Manual)**.

Nach der ersten erfolgreichen Kopplung stellt Android die Verbindung beim Start automatisch wieder her:

* Manuellen Endpoint (falls aktiviert), andernfalls
* Das zuletzt entdeckte Gateway (Best-Effort).

<div id="4-approve-pairing-cli">
  ### 4) Kopplung genehmigen (CLI)
</div>

Auf dem Gateway-Host:

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

Details zur Kopplung: [Gateway-Kopplung](/de/gateway/pairing).

<div id="5-verify-the-node-is-connected">
  ### 5) Überprüfe, ob der Knoten verbunden ist
</div>

* Über `nodes status`:
  ```bash
  openclaw nodes status
  ```
* Über das Gateway:
  ```bash
  openclaw gateway call node.list --params "{}"
  ```

<div id="6-chat-history">
  ### 6) Chat + Verlauf
</div>

Das Chat-Panel des Android-Knotens verwendet den **primären Sitzungsschlüssel** (`main`) des Gateways, sodass Verlauf und Antworten mit WebChat und anderen Clients geteilt werden:

* Verlauf: `chat.history`
* Senden: `chat.send`
* Push-Updates (Best-Effort): `chat.subscribe` → `event:"chat"`

<div id="7-canvas-camera">
  ### 7) Canvas + Kamera
</div>

<div id="gateway-canvas-host-recommended-for-web-content">
  #### Gateway Canvas-Host (empfohlen für Webinhalte)
</div>

Wenn du möchtest, dass der Knoten echtes HTML/CSS/JS anzeigt, das der Agent auf dem Dateisystem bearbeiten kann, weise den Knoten auf den Gateway Canvas-Host.

Hinweis: Knoten nutzen den eigenständigen Canvas-Host auf `canvasHost.port` (Standard `18793`).

1. Erstelle `~/.openclaw/workspace/canvas/index.html` auf dem Gateway-Host.

2. Navigiere vom Knoten (im LAN) dorthin:

```bash
openclaw nodes invoke --node "<Android Node>" --command canvas.navigate --params '{"url":"http://<gateway-hostname>.local:18793/__openclaw__/canvas/"}'
```

Tailnet (optional): Wenn beide Geräte über Tailscale verbunden sind, verwende einen MagicDNS-Namen oder eine Tailnet-IP anstelle von `.local`, z. B. `http://<gateway-magicdns>:18793/__openclaw__/canvas/`.

Dieser Server fügt einen Live-Reload-Client in HTML ein und lädt bei Dateiänderungen neu.
Der A2UI-Host ist unter `http://<gateway-host>:18793/__openclaw__/a2ui/` erreichbar.

Canvas-Befehle (nur im Vordergrund):

* `canvas.eval`, `canvas.snapshot`, `canvas.navigate` (verwende `{"url":""}` oder `{"url":"/"}`, um zum Standard-Scaffold zurückzukehren). `canvas.snapshot` gibt `{ format, base64 }` zurück (Standard: `format="jpeg"`).
* A2UI: `canvas.a2ui.push`, `canvas.a2ui.reset` (`canvas.a2ui.pushJSONL` Legacy-Alias)

Kamera-Befehle (nur im Vordergrund, genehmigungspflichtig):

* `camera.snap` (jpg)
* `camera.clip` (mp4)

Siehe [Camera-Knoten](/de/nodes/camera) für Parameter und CLI-Helfer.
