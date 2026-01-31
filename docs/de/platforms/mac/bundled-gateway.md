---
title: Mitgelieferter Gateway
summary: "Gateway-Laufzeit unter macOS (externer launchd-Dienst)"
read_when:
  - OpenClaw.app paketieren
  - launchd-Dienst des macOS-Gateways debuggen
  - Gateway-CLI für macOS installieren
---

<div id="gateway-on-macos-external-launchd">
  # Gateway auf macOS (externer launchd-Dienst)
</div>

OpenClaw.app bündelt nicht länger Node/Bun oder die Gateway‑Laufzeitumgebung. Die macOS‑App
erwartet eine **externe** Installation der `openclaw`‑CLI, startet das Gateway nicht als
untergeordneten Prozess und verwaltet einen benutzerspezifischen launchd‑Dienst, um das Gateway
dauerhaft auszuführen (oder verbindet sich mit einem bereits lokal laufenden Gateway, falls schon eines aktiv ist).

<div id="install-the-cli-required-for-local-mode">
  ## Installiere die CLI (erforderlich für den lokalen Modus)
</div>

Stelle sicher, dass auf deinem Mac Node 22+ installiert ist, und installiere dann `openclaw` global:

```bash
npm install -g openclaw@<version>
```

Die Schaltfläche **Install CLI** in der macOS‑App führt denselben Prozess über npm/pnpm aus (bun wird für die Gateway‑Laufzeitumgebung nicht empfohlen).


<div id="launchd-gateway-as-launchagent">
  ## Launchd (Gateway als LaunchAgent)
</div>

Label:

- `bot.molt.gateway` (oder `bot.molt.<profile>`; veraltete `com.openclaw.*` können bestehen bleiben)

Plist-Speicherort (benutzerspezifisch):

- `~/Library/LaunchAgents/bot.molt.gateway.plist`
  (oder `~/Library/LaunchAgents/bot.molt.<profile>.plist`)

Verwaltung:

- Die macOS-App verwaltet Installation/Aktualisierung des LaunchAgent im lokalen Modus.
- Die CLI kann ihn ebenfalls installieren: `openclaw gateway install`.

Verhalten:

- „OpenClaw Active“ aktiviert/deaktiviert den LaunchAgent.
- Das Beenden der App stoppt das Gateway **nicht** (launchd hält es am Laufen).
- Wenn bereits ein Gateway auf dem konfigurierten Port läuft, verbindet sich die App
  damit, anstatt ein neues zu starten.

Logging:

- launchd stdout/err: `/tmp/openclaw/openclaw-gateway.log`

<div id="version-compatibility">
  ## Versionskompatibilität
</div>

Die macOS-App prüft die Gateway-Version gegen ihre eigene Version. Wenn sie
inkompatibel sind, aktualisiere die globale CLI, damit sie zur App-Version passt.

<div id="smoke-check">
  ## Smoke-Test
</div>

```bash
openclaw --version

OPENCLAW_SKIP_CHANNELS=1 \
OPENCLAW_SKIP_CANVAS_HOST=1 \
openclaw gateway --port 18999 --bind loopback
```

Als Nächstes:

```bash
openclaw gateway call health --url ws://127.0.0.1:18999 --timeout 3000
```
