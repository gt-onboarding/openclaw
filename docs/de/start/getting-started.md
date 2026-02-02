---
title: Erste Schritte
summary: "Einstiegsleitfaden: von Null zur ersten Nachricht (Assistent, Authentifizierung, Kanäle, Kopplung)"
read_when:
  - Ersteinrichtung von Grund auf
  - Du möchtest den schnellsten Weg von Installation → Onboarding → zur ersten Nachricht
---

<div id="getting-started">
  # Erste Schritte
</div>

Ziel: so schnell wie möglich von **Null** → **erstem funktionierenden Chat** (mit sinnvollen Standardwerten) kommen.

Schnellster Weg zum Chat: Öffne die Control UI (kein Channel-Setup nötig). Führe `openclaw dashboard`
aus und chatte im Browser, oder öffne `http://127.0.0.1:18789/` auf dem Gateway-Host.
Dokumentation: [Dashboard](/de/web/dashboard) und [Control UI](/de/web/control-ui).

Empfohlener Weg: Verwende den **CLI-Onboarding-Assistenten** (`openclaw onboard`). Er richtet ein:

* Modell/Auth (OAuth wird empfohlen)
* Gateway-Einstellungen
* Channels (WhatsApp/Telegram/Discord/Mattermost (Plugin)/...)
* Pairing-Standardeinstellungen (sichere DMs)
* Initialisierung des Arbeitsbereichs + Fähigkeiten
* optionalen Hintergrunddienst

Wenn du die ausführlicheren Referenzseiten möchtest, wechsle zu: [Wizard](/de/start/wizard), [Setup](/de/start/setup), [Pairing](/de/start/pairing), [Security](/de/gateway/security).

Hinweis zur Sandbox: `agents.defaults.sandbox.mode: "non-main"` verwendet `session.mainKey` (Standard `"main"`),
damit werden Gruppen-/Channel-Sitzungen in der Sandbox ausgeführt. Wenn du möchtest, dass der Haupt-Agent immer
auf dem Host läuft, setze einen expliziten Override pro Agent:

```json
{
  "routing": {
    "agents": {
      "main": {
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      }
    }
  }
}
```

<div id="0-prereqs">
  ## 0) Voraussetzungen
</div>

* Node `>=22`
* `pnpm` (optional; empfohlen, wenn du aus dem Quellcode baust)
* **Empfohlen:** Brave Search-API-Schlüssel für die Websuche. Der einfachste Weg:
  `openclaw configure --section web` (speichert `tools.web.search.apiKey`).
  Siehe [Web tools](/de/tools/web).

macOS: Wenn du vorhast, die Apps zu bauen, installiere Xcode / CLT. Für nur CLI + Gateway reicht Node.
Windows: Verwende **WSL2** (Ubuntu empfohlen). WSL2 wird dringend empfohlen; natives Windows ist ungetestet, problematischer und hat schlechtere Kompatibilität mit Tools. Installiere zuerst WSL2 und führe dann die Linux-Schritte innerhalb von WSL aus. Siehe [Windows (WSL2)](/de/platforms/windows).

<div id="1-install-the-cli-recommended">
  ## 1) Installiere die CLI (empfohlen)
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Installationsoptionen (Installationsmethode, nicht interaktiv, von GitHub): [Installation](/de/install).

Windows (PowerShell):

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

Alternative (globale Installation):

```bash
npm install -g openclaw@latest
```

```bash
pnpm add -g openclaw@latest
```

<div id="2-run-the-onboarding-wizard-and-install-the-service">
  ## 2) Starte den Onboarding‑Assistenten (und installiere den Dienst)
</div>

```bash
openclaw onboard --install-daemon
```

Wofür du dich entscheidest:

* **Gateway**: lokal vs. remote
* **Auth**: OpenAI Code (Codex)-Abonnement (OAuth) oder API-Schlüssel. Für Anthropic empfehlen wir einen API-Schlüssel; `claude setup-token` wird ebenfalls unterstützt.
* **Anbieter**: WhatsApp-QR-Login, Telegram-/Discord-Bot-Tokens, Mattermost-Plugin-Tokens usw.
* **Daemon**: Hintergrundinstallation (launchd/systemd; WSL2 verwendet systemd)
  * **Runtime**: Node.js (empfohlen; erforderlich für WhatsApp/Telegram). Bun wird **nicht empfohlen**.
* **Gateway-Token**: Der Wizard erzeugt standardmäßig eines (auch auf Loopback) und speichert es in `gateway.auth.token`.

Wizard-Dokumentation: [Wizard](/de/start/wizard)

<div id="auth-where-it-lives-important">
  ### Auth: Speicherort (wichtig)
</div>

* **Empfohlener Anthropic-Pfad:** API-Schlüssel setzen (der Assistent kann ihn für den Dienst speichern). `claude setup-token` wird ebenfalls unterstützt, wenn du Claude-Code-Anmeldedaten wiederverwenden möchtest.

* OAuth-Zugangsdaten (Legacy-Import): `~/.openclaw/credentials/oauth.json`

* Auth-Profile (OAuth + API-Schlüssel): `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

Headless-/Server-Tipp: Führe OAuth zuerst auf einem normalen Rechner aus und kopiere dann `oauth.json` auf den Gateway-Host.

<div id="3-start-the-gateway">
  ## 3) Starte das Gateway
</div>

Wenn du den Dienst während des Onboardings installiert hast, sollte das Gateway bereits laufen:

```bash
openclaw gateway status
```

Manueller Start (Vordergrund):

```bash
openclaw gateway --port 18789 --verbose
```

Dashboard (lokaler Loopback): `http://127.0.0.1:18789/`
Wenn ein Token konfiguriert ist, trage es in den Control UI-Einstellungen ein (gespeichert als `connect.params.auth.token`).

⚠️ **Bun-Warnung (WhatsApp + Telegram):** Bun hat bekannte Probleme mit diesen
Kanälen. Wenn du WhatsApp oder Telegram verwendest, starte das Gateway mit **Node**.

<div id="35-quick-verify-2-min">
  ## 3.5) Kurzcheck (2 Min)
</div>

```bash
openclaw status
openclaw health
openclaw security audit --deep
```

<div id="4-pair-connect-your-first-chat-surface">
  ## 4) Kopple + verbinde deinen ersten Chat-Kanal
</div>

<div id="whatsapp-qr-login">
  ### WhatsApp (QR-Login)
</div>

```bash
openclaw channels login
```

Scanne in WhatsApp unter Einstellungen → Verknüpfte Geräte.

WhatsApp-Dokumentation: [WhatsApp](/de/channels/whatsapp)

<div id="telegram-discord-others">
  ### Telegram / Discord / andere
</div>

Der Wizard kann Tokens/Konfiguration für Sie erzeugen. Wenn Sie eine manuelle Konfiguration bevorzugen, starten Sie mit:

* Telegram: [Telegram](/de/channels/telegram)
* Discord: [Discord](/de/channels/discord)
* Mattermost (Plugin): [Mattermost](/de/channels/mattermost)

**Telegram-DM-Tipp:** Ihre erste DM (Direktnachricht) liefert einen Kopplungscode zurück. Genehmigen Sie ihn (siehe nächsten Schritt), sonst reagiert der Bot nicht.

<div id="5-dm-safety-pairing-approvals">
  ## 5) DM-Sicherheit (Kopplungsfreigaben)
</div>

Standardverhalten: Unbekannte DMs erhalten einen Kurzcode, und Nachrichten werden nicht verarbeitet, bis sie genehmigt wurden.
Wenn auf deine erste DM keine Antwort kommt, genehmige die Kopplung:

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <code>
```

Dokumentation zur Kopplung: [Kopplung](/de/start/pairing)

<div id="from-source-development">
  ## Aus dem Quellcode (Entwicklung)
</div>

Wenn du selbst an OpenClaw entwickelst, führe es direkt aus dem Quellcode aus:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # installiert UI-Abhängigkeiten beim ersten Ausführen automatisch
pnpm build
openclaw onboard --install-daemon
```

Falls Sie noch keine globale Installation haben, führen Sie den Onboarding-Schritt über `pnpm openclaw ...` aus dem Repository aus.
`pnpm build` bündelt außerdem die A2UI-Assets; wenn Sie nur diesen Schritt ausführen müssen, verwenden Sie `pnpm canvas:a2ui:bundle`.

Gateway (aus diesem Repo):

```bash
node openclaw.mjs gateway --port 18789 --verbose
```

<div id="7-verify-end-to-end">
  ## 7) End-to-End überprüfen
</div>

Sende in einem neuen Terminal eine Testnachricht:

```bash
openclaw message send --target +15555550123 --message "Hello from OpenClaw"
```

Wenn `openclaw health` „no auth configured“ anzeigt, geh zurück zum Wizard und richte OAuth-/Key-Authentifizierung ein — der Agent kann ohne sie nicht antworten.

Tipp: `openclaw status --all` ist der beste, zum Einfügen geeignete, schreibgeschützte Debug-Bericht.
Health-Checks: `openclaw health` (oder `openclaw status --deep`) fragt das laufende Gateway nach einem Health-Snapshot ab.

<div id="next-steps-optional-but-great">
  ## Nächste Schritte (optional, aber sehr sinnvoll)
</div>

* macOS-Menüleisten-App + Sprachaktivierung: [macOS-App](/de/platforms/macos)
* iOS/Android-Knoten (Canvas/Kamera/Sprachsteuerung): [Knoten](/de/nodes)
* Fernzugriff (SSH-Tunnel / Tailscale Serve): [Remote-Zugriff](/de/gateway/remote) und [Tailscale](/de/gateway/tailscale)
* Always-on-/VPN-Konfigurationen: [Remote-Zugriff](/de/gateway/remote), [exe.dev](/de/platforms/exe-dev), [Hetzner](/de/platforms/hetzner), [macOS-Remotezugriff](/de/platforms/mac/remote)