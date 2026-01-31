---
title: Setup
summary: "Setup-Anleitung: Halte dein OpenClaw-Setup personalisiert und trotzdem auf dem neuesten Stand"
read_when:
  - Beim Einrichten eines neuen Systems
  - Du möchtest „latest + greatest“, ohne dein persönliches Setup zu beeinträchtigen
---

<div id="setup">
  # Einrichtung
</div>

Zuletzt aktualisiert: 2026-01-01

<div id="tldr">
  ## TL;DR
</div>

* **Tailoring findet außerhalb des Repositories statt:** `~/.openclaw/workspace` (Arbeitsbereich) + `~/.openclaw/openclaw.json` (Konfiguration).
* **Stabiler Workflow:** Installiere die macOS‑App; lass sie das mitgelieferte Gateway ausführen.
* **Bleeding-Edge-Workflow:** Führe das Gateway selbst über `pnpm gateway:watch` aus und lass anschließend die macOS‑App im lokalen Modus verbinden.

<div id="prereqs-from-source">
  ## Voraussetzungen (aus dem Quellcode)
</div>

* Node.js `>=22`
* `pnpm`
* Docker (optional; nur für containerisiertes Setup/E2E – siehe [Docker](/de/install/docker))

<div id="tailoring-strategy-so-updates-dont-hurt">
  ## Strategie für Anpassungen (damit Updates nicht wehtun)
</div>

Wenn Sie „100 % auf mich zugeschnitten“ *und* einfache Updates wollen, sollten Ihre Anpassungen hier liegen:

* **Config:** `~/.openclaw/openclaw.json` (JSON/JSON5-ähnlich)
* **Arbeitsbereich:** `~/.openclaw/workspace` (Fähigkeiten, Prompts, Erinnerungen; machen Sie daraus ein privates Git-Repository)

Einmalig bootstrappen:

```bash
openclaw setup
```

Verwende in diesem Repo den lokalen CLI-Einstiegspunkt:

```bash
openclaw setup
```

Wenn du openclaw noch nicht global installiert hast, führe den Befehl `pnpm openclaw setup` aus.

<div id="stable-workflow-macos-app-first">
  ## Stabiler Workflow (macOS‑App zuerst)
</div>

1. **OpenClaw.app** installieren und starten (Menüleiste).
2. Die Onboarding-/Berechtigungs-Checkliste durchlaufen (TCC‑Aufforderungen).
3. Sicherstellen, dass der Gateway **lokal** ist und läuft (die App verwaltet ihn).
4. Kanäle verknüpfen (Beispiel: WhatsApp):

```bash
openclaw channels login
```

5. Funktionsprüfung:

```bash
openclaw health
```

Wenn das Onboarding in deinem Build nicht zur Verfügung steht:

* Führe zunächst `openclaw setup` aus, dann `openclaw channels login`, und starte anschließend das Gateway manuell (`openclaw gateway`).

<div id="bleeding-edge-workflow-gateway-in-a-terminal">
  ## Bleeding-Edge-Workflow (Gateway in einem Terminal)
</div>

Ziel: Am TypeScript-Gateway entwickeln, Hot Reload nutzen und die macOS-App-UI verbunden lassen.

<div id="0-optional-run-the-macos-app-from-source-too">
  ### 0) (Optional) Die macOS-App ebenfalls aus dem Quellcode starten
</div>

Wenn du die macOS-App ebenfalls auf dem neuesten Stand (Bleeding Edge) haben möchtest:

```bash
./scripts/restart-mac.sh
```

<div id="1-start-the-dev-gateway">
  ### 1) Dev-Gateway starten
</div>

```bash
pnpm install
pnpm gateway:watch
```

`gateway:watch` startet den Gateway im Watch-Modus und lädt ihn bei TypeScript-Änderungen neu.

<div id="2-point-the-macos-app-at-your-running-gateway">
  ### 2) Verbinde die macOS-App mit deinem laufenden Gateway
</div>

In **OpenClaw.app**:

* Verbindungsmodus: **Lokal**
  Die App stellt über den konfigurierten Port eine Verbindung zum laufenden Gateway her.

<div id="3-verify">
  ### 3) Überprüfen
</div>

* Der Gateway-Status in der App sollte **„Using existing gateway …“** anzeigen
* Oder über die CLI:

```bash
openclaw health
```

<div id="common-footguns">
  ### Häufige Stolperfallen
</div>

* **Falscher Port:** Gateway WS verwendet standardmäßig `ws://127.0.0.1:18789`; verwende für App und CLI denselben Port.
* **Speicherorte:**
  * Anmeldedaten: `~/.openclaw/credentials/`
  * Sitzungen: `~/.openclaw/agents/<agentId>/sessions/`
  * Logs: `/tmp/openclaw/`

<div id="credential-storage-map">
  ## Speicherorte für Zugangsdaten
</div>

Verwende das beim Debuggen von Auth-Problemen oder um zu entscheiden, was du sichern musst:

* **WhatsApp**: `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
* **Telegram-Bot-Token**: Konfiguration/Umgebung oder `channels.telegram.tokenFile`
* **Discord-Bot-Token**: Konfiguration/Umgebung (Token-Datei wird noch nicht unterstützt)
* **Slack-Tokens**: Konfiguration/Umgebung (`channels.slack.*`)
* **Kopplungs-Allowlists**: `~/.openclaw/credentials/<channel>-allowFrom.json`
* **Modell-Auth-Profile**: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
* **Legacy-OAuth-Import**: `~/.openclaw/credentials/oauth.json`
  Weitere Details: [Security](/de/gateway/security#credential-storage-map).

<div id="updating-without-wrecking-your-setup">
  ## Aktualisieren (ohne dein Setup zu zerschießen)
</div>

* Behalte `~/.openclaw/workspace` und `~/.openclaw/` als „deine Sachen“; lege keine persönlichen Prompts oder Konfigurationen ins `openclaw`-Repo.
* Quellcode aktualisieren: `git pull` + `pnpm install` (wenn sich der Lockfile geändert hat) + weiterhin `pnpm gateway:watch` verwenden.

<div id="linux-systemd-user-service">
  ## Linux (systemd-User-Service)
</div>

Linux-Installationen verwenden einen systemd-**User**-Service. Standardmäßig
beendet systemd User-Services bei Abmeldung oder Inaktivität, wodurch der Gateway beendet wird.
Der Onboarding-Prozess versucht, Lingering für dich zu aktivieren (kann nach sudo fragen).
Wenn es trotzdem deaktiviert ist, führe Folgendes aus:

```bash
sudo loginctl enable-linger $USER
```

Für Always-on- oder Multi-User-Server solltest du einen **System**-Service statt eines User-Services verwenden (kein Lingering erforderlich). Siehe [Gateway runbook](/de/gateway) für Hinweise zu systemd.

<div id="related-docs">
  ## Verwandte Dokumentation
</div>

* [Gateway-Runbook](/de/gateway) (Flags, Überwachung, Ports)
* [Gateway-Konfiguration](/de/gateway/configuration) (Konfigurationsschema + Beispiele)
* [Discord](/de/channels/discord) und [Telegram](/de/channels/telegram) (Reply-Tags + `replyToMode`-Einstellungen)
* [Einrichtung des OpenClaw-Assistenten](/de/start/openclaw)
* [macOS-App](/de/platforms/macos) (Lebenszyklus des Gateways)