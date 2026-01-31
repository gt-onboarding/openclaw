---
title: Macos
summary: "OpenClaw macOS-Companion-App (Menüleiste + Gateway-Broker)"
read_when:
  - Implementierung von macOS-App-Funktionen
  - Änderungen am Gateway-Lebenszyklus oder am Node-Bridging auf macOS
---

<div id="openclaw-macos-companion-menu-bar-gateway-broker">
  # OpenClaw macOS Companion (Menüleiste + Gateway-Broker)
</div>

Die macOS‑App ist die **Begleit‑App in der Menüleiste** für OpenClaw. Sie verfügt über die Berechtigungen,
bindet das Gateway lokal ein (über launchd oder manuell) und stellt dem Agent macOS‑Funktionen
als Knoten zur Verfügung.

<div id="what-it-does">
  ## Was es macht
</div>

* Zeigt native Mitteilungen (Benachrichtigungen) und Status in der Menüleiste an.
* Verwaltet TCC‑Prompts (Mitteilungen, Bedienungshilfen, Bildschirmaufnahme, Mikrofon,
  Spracherkennung, Automatisierung/AppleScript).
* Startet den Gateway oder verbindet sich mit ihm (lokal oder remote).
* Stellt macOS‑exklusive Tools bereit (Canvas, Kamera, Bildschirmaufnahme, `system.run`).
* Startet den lokalen Knoten‑Hostdienst im **remote**‑Modus (launchd) und stoppt ihn im **local**‑Modus.
* Kann optional **PeekabooBridge** für UI‑Automatisierung hosten.
* Installiert bei Bedarf die globale CLI (`openclaw`) via npm/pnpm (bun wird für die Gateway‑Laufzeit nicht empfohlen).

<div id="local-vs-remote-mode">
  ## Lokal vs Remote-Modus
</div>

* **Lokal** (Standard): Die App verbindet sich mit einem lokal laufenden Gateway, falls eines vorhanden ist;
  andernfalls aktiviert sie den launchd-Dienst über `openclaw gateway install`.
* **Remote**: Die App verbindet sich über SSH/Tailscale mit einem Gateway und startet dabei keinen
  lokalen Gateway-Prozess.
  Die App startet den lokalen **Node-Host-Dienst**, damit das entfernte Gateway diesen Mac erreichen kann.
  Die App startet das Gateway nicht als Unterprozess.

<div id="launchd-control">
  ## Launchd-Steuerung
</div>

Die App verwaltet einen LaunchAgent pro Benutzer mit dem Label `bot.molt.gateway`
(oder `bot.molt.<profile>` bei Verwendung von `--profile`/`OPENCLAW_PROFILE`; ältere `com.openclaw.*`-Einträge werden weiterhin entladen).

```bash
launchctl kickstart -k gui/$UID/bot.molt.gateway
launchctl bootout gui/$UID/bot.molt.gateway
```

Ersetze das Label durch `bot.molt.<profile>`, wenn du ein benanntes Profil verwendest.

Falls der LaunchAgent nicht installiert ist, aktiviere ihn über die App oder führe
`openclaw gateway install` aus.

<div id="node-capabilities-mac">
  ## Knotenfunktionen (macOS)
</div>

Die macOS-App meldet sich als Knoten an. Übliche Befehle:

* Canvas: `canvas.present`, `canvas.navigate`, `canvas.eval`, `canvas.snapshot`, `canvas.a2ui.*`
* Kamera: `camera.snap`, `camera.clip`
* Bildschirm: `screen.record`
* System: `system.run`, `system.notify`

Der Knoten stellt eine `permissions`-Map bereit, damit Agenten entscheiden können, was erlaubt ist.

Knoten-Dienst + App-IPC:

* Wenn der headless Knoten-Host-Dienst läuft (Remote-Modus), verbindet er sich als Knoten per WS mit dem Gateway.
* `system.run` wird in der macOS-App (UI/TCC-Kontext) über einen lokalen Unix-Socket ausgeführt; Prompts und Ausgaben bleiben in der App.

Diagramm (SCI):

```
Gateway -> Node Service (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac App (UI + TCC + system.run)
```

<div id="exec-approvals-systemrun">
  ## Exec-Bestätigungen (system.run)
</div>

`system.run` wird über **Exec-Bestätigungen** in der macOS-App gesteuert („Settings → Exec approvals“).
Security + ask + Allowlist werden lokal auf dem Mac unter gespeichert:

```
~/.openclaw/exec-approvals.json
```

Beispiel:

```json
{
  "version": 1,
  "defaults": {
    "security": "deny",
    "ask": "on-miss"
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [
        { "pattern": "/opt/homebrew/bin/rg" }
      ]
    }
  }
}
```

Hinweise:

* `allowlist`-Einträge sind Glob-Muster für aufgelöste Pfade zu Binärdateien.
* Wenn du im Prompt „Always Allow“ auswählst, wird dieser Befehl zur Allowlist hinzugefügt.
* `system.run`-Umgebungs-Overrides werden gefiltert (dabei werden `PATH`, `DYLD_*`, `LD_*`, `NODE_OPTIONS`, `PYTHON*`, `PERL*`, `RUBYOPT` entfernt) und anschließend mit der Umgebung der App zusammengeführt.

<div id="deep-links">
  ## Deep links
</div>

Die App registriert das URL-Schema `openclaw://` für lokale Aktionen.

<div id="openclawagent">
  ### `openclaw://agent`
</div>

Löst eine `agent`-Anfrage an das Gateway aus.

```bash
open 'openclaw://agent?message=Hello%20from%20deep%20link'
```

Abfrageparameter:

* `message` (erforderlich)
* `sessionKey` (optional)
* `thinking` (optional)
* `deliver` / `to` / `channel` (optional)
* `timeoutSeconds` (optional)
* `key` (optionaler Schlüssel für den unbeaufsichtigten Modus)

Sicherheit:

* Ohne `key` fordert die App eine Bestätigung an.
* Mit einem gültigen `key` wird der Lauf unbeaufsichtigt ausgeführt (vorgesehen für persönliche Automatisierungen).

<div id="onboarding-flow-typical">
  ## Onboarding-Flow (typisch)
</div>

1. Installiere und starte **OpenClaw.app**.
2. Schließe die Berechtigungs-Checkliste (TCC-Dialoge) ab.
3. Stelle sicher, dass der **Local-Modus** aktiv ist und das Gateway läuft.
4. Installiere die CLI, wenn du Terminalzugriff haben möchtest.

<div id="build-dev-workflow-native">
  ## Build- und Dev-Workflow (native)
</div>

* `cd apps/macos && swift build`
* `swift run OpenClaw` (oder Xcode)
* App paketieren: `scripts/package-mac-app.sh`

<div id="debug-gateway-connectivity-macos-cli">
  ## Gateway‑Konnektivität debuggen (macOS CLI)
</div>

Verwende die Debug-CLI, um denselben Gateway-WebSocket-Handshake und dieselbe Discovery-Logik zu testen, die auch die macOS-App verwendet – ohne die App zu starten.

```bash
cd apps/macos
swift run openclaw-mac connect --json
swift run openclaw-mac discover --timeout 3000 --json
```

Verbindungsoptionen:

* `--url <ws://host:port>`: Konfiguration überschreiben
* `--mode <local|remote>`: aus der Konfiguration ermitteln (Standard: config oder local)
* `--probe`: einen neuen Health-Check erzwingen
* `--timeout <ms>`: Anfrage-Timeout (Standard: `15000`)
* `--json`: strukturierte Ausgabe für Diffs/Vergleiche

Discovery-Optionen:

* `--include-local`: auch Gateways einbeziehen, die normalerweise als „local“ herausgefiltert würden
* `--timeout <ms>`: gesamtes Zeitfenster für Discovery (Standard: `2000`)
* `--json`: strukturierte Ausgabe für Diffs/Vergleiche

Tipp: Vergleiche mit `openclaw gateway discover --json`, um zu sehen, ob sich die
Discovery-Pipeline der macOS-App (NWBrowser + tailnet DNS‑SD Fallback) von der
`dns-sd`-basierten Discovery der Node-CLI unterscheidet.

<div id="remote-connection-plumbing-ssh-tunnels">
  ## Remote-Verbindungsinfrastruktur (SSH-Tunnel)
</div>

Wenn die macOS-App im **Remote**-Modus läuft, öffnet sie einen SSH-Tunnel, damit lokale UI-Komponenten mit einem entfernten Gateway so kommunizieren können, als liefe es auf `localhost`.

<div id="control-tunnel-gateway-websocket-port">
  ### Steuerungstunnel (Gateway-WebSocket-Port)
</div>

* **Zweck:** Health-Checks, Status, Web Chat, Konfiguration und andere Control-Plane-Aufrufe.
* **Lokaler Port:** der Gateway-Port (Standard: `18789`), immer stabil.
* **Entfernter Port:** derselbe Gateway-Port auf dem entfernten Host.
* **Verhalten:** kein zufälliger lokaler Port; die App nutzt einen vorhandenen, funktionsfähigen Tunnel erneut
  oder startet ihn bei Bedarf neu.
* **SSH-Form:** `ssh -N -L <local>:127.0.0.1:<remote>` mit BatchMode +
  ExitOnForwardFailure + Keepalive-Optionen.
* **IP-Erkennung:** der SSH-Tunnel verwendet Loopback, daher sieht das Gateway die Knoten-IP
  als `127.0.0.1`. Verwende den Transportmodus **Direct (ws/wss)**, wenn die echte Client-IP
  erscheinen soll (siehe [macOS remote access](/de/platforms/mac/remote)).

Für die Einrichtungsschritte siehe [macOS remote access](/de/platforms/mac/remote). Für Protokolldetails
siehe [Gateway protocol](/de/gateway/protocol).

<div id="related-docs">
  ## Verwandte Dokumente
</div>

* [Gateway-Runbook](/de/gateway)
* [Gateway (macOS)](/de/platforms/mac/bundled-gateway)
* [macOS-Berechtigungen](/de/platforms/mac/permissions)
* [Canvas](/de/platforms/mac/canvas)