---
title: Übersicht
summary: "Überblick über OpenClaw, seine Funktionen und seinen Zweck"
read_when:
  - OpenClaw neuen Einsteigerinnen und Einsteigern vorstellen
---

<div id="openclaw">
  # OpenClaw 🦞
</div>

> „EXFOLIATE! EXFOLIATE!“ — Ein Weltraum-Hummer, wahrscheinlich

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text-dark.png" />

    <img src="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text.png" alt="OpenClaw" width="500" />
  </picture>
</p>

<p align="center">
  <strong>Beliebiges Betriebssystem + WhatsApp/Telegram/Discord/iMessage-Gateway für KI-Agenten (Pi).</strong><br />
  Plugins ergänzen Mattermost und mehr.
  Sende eine Nachricht, erhalte die Antwort eines Agenten – direkt aus deiner Hosentasche.
</p>

<p align="center">
  <a href="https://github.com/openclaw/openclaw">GitHub</a> ·
  <a href="https://github.com/openclaw/openclaw/releases">Releases</a> ·
  <a href="/de/">Dokumentation</a> ·
  <a href="/de/start/openclaw">OpenClaw-Assistant-Einrichtung</a>
</p>

OpenClaw verbindet WhatsApp (über WhatsApp Web / Baileys), Telegram (Bot API / grammY), Discord (Bot API / channels.discord.js) und iMessage (imsg CLI) mit Coding-Agenten wie [Pi](https://github.com/badlogic/pi-mono). Plugins ergänzen Mattermost (Bot API + WebSocket) und mehr.
OpenClaw ist außerdem die Grundlage für den OpenClaw Assistant.

<div id="start-here">
  ## Hier starten
</div>

* **Neue Installation von Grund auf:** [Erste Schritte](/de/start/getting-started)
* **Geführte Einrichtung (empfohlen):** [Assistent](/de/start/wizard) (`openclaw onboard`)
* **Dashboard öffnen (lokales Gateway):** http://127.0.0.1:18789/ (oder http://localhost:18789/)

Wenn das Gateway auf demselben Computer läuft, öffnet dieser Link die browserbasierte Control UI
sofort. Wenn das nicht klappt, starte zuerst das Gateway: `openclaw gateway`.

<div id="dashboard-browser-control-ui">
  ## Dashboard (browserbasierte Control UI)
</div>

Das Dashboard ist die browserbasierte Control UI für Chat, Konfiguration, Knoten, Sitzungen und mehr.
Lokale Standard-URL: http://127.0.0.1:18789/
Remotezugriff: [Web-Oberflächen](/de/web) und [Tailscale](/de/gateway/tailscale)

<p align="center">
  <img src="/whatsapp-openclaw.jpg" alt="OpenClaw" width="420" />
</p>

<div id="how-it-works">
  ## Funktionsweise
</div>

```
WhatsApp / Telegram / Discord / iMessage (+ plugins)
        │
        ▼
  ┌───────────────────────────┐
  │          Gateway          │  ws://127.0.0.1:18789 (loopback-only)
  │     (einzelne Quelle)     │
  │                           │  http://<gateway-host>:18793
  │                           │    /__openclaw__/canvas/ (Canvas host)
  └───────────┬───────────────┘
              │
              ├─ Pi agent (RPC)
              ├─ CLI (openclaw …)
              ├─ Chat UI (SwiftUI)
              ├─ macOS app (OpenClaw.app)
              ├─ iOS node via Gateway WS + pairing
              └─ Android node via Gateway WS + pairing
```

Die meisten Abläufe laufen über das **Gateway** (`openclaw gateway`), einen dauerhaft laufenden Prozess, der die Kanalverbindungen und die WebSocket-Control-Plane verwaltet.

<div id="network-model">
  ## Netzwerkmodell
</div>

* **Ein Gateway pro Host (empfohlen)**: Es ist der einzige Prozess, der die WhatsApp-Web-Sitzung besitzen darf. Wenn du einen Rescue-Bot oder strikte Isolation brauchst, betreibe mehrere Gateways mit isolierten Profilen und Ports; siehe [Multiple gateways](/de/gateway/multiple-gateways).
* **Loopback-first**: Gateway-WS verwendet standardmäßig `ws://127.0.0.1:18789`.
  * Der Assistent erzeugt jetzt standardmäßig ein Gateway-Token (auch für Loopback).
  * Für Tailnet-Zugriff führe `openclaw gateway --bind tailnet --token ...` aus (ein Token ist für Nicht-Loopback-Binds erforderlich).
* **Knoten**: verbinden sich mit dem Gateway-WebSocket (LAN/Tailnet/SSH je nach Bedarf); die Legacy-TCP-Bridge wurde als veraltet eingestuft und entfernt.
* **Canvas-Host**: HTTP-Dateiserver auf `canvasHost.port` (Standard `18793`), der `/__openclaw__/canvas/` für Knoten-WebViews bereitstellt; siehe [Gateway configuration](/de/gateway/configuration) (`canvasHost`).
* **Remote-Nutzung**: SSH-Tunnel oder Tailnet/VPN; siehe [Remote access](/de/gateway/remote) und [Discovery](/de/gateway/discovery).

<div id="features-high-level">
  ## Features (High Level)
</div>

* 📱 **WhatsApp-Integration** — nutzt Baileys für das WhatsApp-Web-Protokoll
* ✈️ **Telegram-Bot** — Direktnachrichten + Gruppen über grammY
* 🎮 **Discord-Bot** — Direktnachrichten + Guild-Kanäle über channels.discord.js
* 🧩 **Mattermost-Bot (Plugin)** — Bot-Token + WebSocket-Ereignisse
* 💬 **iMessage** — lokale imsg-CLI-Integration (macOS)
* 🤖 **Agent-Bridge** — Pi (RPC-Modus) mit Tool-Streaming
* ⏱️ **Streaming + Chunking** — Block-Streaming + Details zum Telegram-Entwurfs-Streaming ([/concepts/streaming](/de/concepts/streaming))
* 🧠 **Multi-Agenten-Routing** — routet Anbieter-Konten/Peers zu isolierten Agenten (Arbeitsbereich + Sitzungen pro Agent)
* 🔐 **Abonnement-Authentifizierung** — Anthropic (Claude Pro/Max) + OpenAI (ChatGPT/Codex) via OAuth
* 💬 **Sitzungen** — direkte Chats werden in eine gemeinsame `main`-Sitzung (Standard) zusammengeführt; Gruppen sind isoliert
* 👥 **Gruppenchat-Unterstützung** — standardmäßig erwähnungsbasiert; der Besitzer kann `/activation always|mention` umschalten
* 📎 **Medienunterstützung** — Senden und Empfangen von Bildern, Audio und Dokumenten
* 🎤 **Sprachnachrichten** — optionaler Transkriptions-Hook
* 🖥️ **WebChat + macOS-App** — lokale UI + Menüleisten-Begleiter für Ops und Sprachaktivierung
* 📱 **iOS node** — wird als node gekoppelt und stellt eine Canvas-Oberfläche bereit
* 📱 **Android node** — wird als node gekoppelt und stellt Canvas + Chat + Kamera bereit

Hinweis: Veraltete Claude/Codex/Gemini/Opencode-Pfade wurden entfernt; Pi ist der einzige Coding-Agent-Pfad.

<div id="quick-start">
  ## Schnellstart
</div>

Laufzeitanforderung: **Node ≥ 22**.

```bash
# Recommended: global install (npm/pnpm)
npm install -g openclaw@latest
# or: pnpm add -g openclaw@latest

# Onboard + install the service (launchd/systemd user service)
openclaw onboard --install-daemon

# Pair WhatsApp Web (shows QR)
openclaw channels login

# Gateway läuft nach dem Onboarding über den Service; manueller Start ist weiterhin möglich:
openclaw gateway --port 18789
```

Der Wechsel zwischen npm- und git-Installationen ist später problemlos möglich: Installiere die andere Variante und führe `openclaw doctor` aus, um den Einstiegspunkt des Gateway-Dienstes zu aktualisieren.

Aus dem Quellcode (Entwicklung):

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # installiert UI-Abhängigkeiten beim ersten Ausführen automatisch
pnpm build
openclaw onboard --install-daemon
```

Wenn du noch keine globale Installation hast, führe den Onboarding-Schritt im Repository-Verzeichnis mit `pnpm openclaw ...` aus.

Schnellstart für mehrere Instanzen (optional):

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

Sende eine Testnachricht (setzt ein laufendes Gateway voraus):

```bash
openclaw message send --target +15555550123 --message "Hallo von OpenClaw"
```

<div id="configuration-optional">
  ## Konfiguration (optional)
</div>

Die Konfiguration liegt unter `~/.openclaw/openclaw.json`.

* Wenn du **nichts machst**, verwendet OpenClaw das mitgelieferte Pi-Binary im RPC-Modus mit Sitzungen pro Absender.
* Wenn du das System stärker einschränken möchtest, beginne mit `channels.whatsapp.allowFrom` und (für Gruppen) mit Erwähnungsregeln.

Beispiel:

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } }
    }
  },
  messages: { groupChat: { mentionPatterns: ["@openclaw"] } }
}
```

<div id="docs">
  ## Doku
</div>

* Starte hier:
  * [Doku-Hubs (alle Seiten verlinkt)](/de/start/hubs)
  * [Hilfe](/de/help) ← *häufige Lösungen + Fehlerbehebung*
  * [Konfiguration](/de/gateway/configuration)
  * [Konfigurationsbeispiele](/de/gateway/configuration-examples)
  * [Slash-Befehle](/de/tools/slash-commands)
  * [Routing mit mehreren Agenten](/de/concepts/multi-agent)
  * [Updates / Rollback](/de/install/updating)
  * [Kopplung (DM + Knoten)](/de/start/pairing)
  * [Nix-Modus](/de/install/nix)
  * [Einrichtung des OpenClaw-Assistants](/de/start/openclaw)
  * [Fähigkeiten](/de/tools/skills)
  * [Konfiguration der Fähigkeiten](/de/tools/skills-config)
  * [Arbeitsbereichs-Vorlagen](/de/reference/templates/AGENTS)
  * [RPC-Adapter](/de/reference/rpc)
  * [Gateway-Runbook](/de/gateway)
  * [Knoten (iOS/Android)](/de/nodes)
  * [Web-Oberflächen (Control UI)](/de/web)
  * [Discovery + Transports](/de/gateway/discovery)
  * [Remotezugriff](/de/gateway/remote)
* Anbieter und UX:
  * [WebChat](/de/web/webchat)
  * [Control UI (Browser)](/de/web/control-ui)
  * [Telegram](/de/channels/telegram)
  * [Discord](/de/channels/discord)
  * [Mattermost (Plugin)](/de/channels/mattermost)
  * [iMessage](/de/channels/imessage)
  * [Gruppen](/de/concepts/groups)
  * [WhatsApp-Gruppennachrichten](/de/concepts/group-messages)
  * [Medien: Bilder](/de/nodes/images)
  * [Medien: Audio](/de/nodes/audio)
* Begleit-Apps:
  * [macOS-App](/de/platforms/macos)
  * [iOS-App](/de/platforms/ios)
  * [Android-App](/de/platforms/android)
  * [Windows (WSL2)](/de/platforms/windows)
  * [Linux-App](/de/platforms/linux)
* Betrieb und Sicherheit:
  * [Sitzungen](/de/concepts/session)
  * [Cron-Jobs](/de/automation/cron-jobs)
  * [Webhooks](/de/automation/webhook)
  * [Gmail-Hooks (Pub/Sub)](/de/automation/gmail-pubsub)
  * [Sicherheit](/de/gateway/security)
  * [Fehlerbehebung](/de/gateway/troubleshooting)

<div id="the-name">
  ## Der Name
</div>

**OpenClaw = CLAW + TARDIS** — denn jeder Weltraum-Hummer braucht eine Raum-und-Zeit-Maschine.

***

*„Am Ende spielen wir doch alle nur mit unseren eigenen Prompts.“* — eine KI, vermutlich auf Tokens high

<div id="credits">
  ## Danksagungen
</div>

* **Peter Steinberger** ([@steipete](https://twitter.com/steipete)) — Erfinder, Hummer-Flüsterer
* **Mario Zechner** ([@badlogicc](https://twitter.com/badlogicgames)) — Pi-Erfinder, Security-Penetrationstester
* **Clawd** — der Weltraum-Hummer, der einen besseren Namen verlangte

<div id="core-contributors">
  ## Hauptmitwirkende
</div>

* **Maxim Vovshin** (@Hyaxia, 36747317+Hyaxia@users.noreply.github.com) — Blogwatcher-Skill
* **Nacho Iacovino** (@nachoiacovino, nacho.iacovino@gmail.com) — Standort-Parsing (Telegram und WhatsApp)

<div id="license">
  ## Lizenz
</div>

MIT — Frei wie ein Hummer im Ozean 🦞

***

*„Wir spielen doch alle nur mit unseren eigenen Prompts.“* — Eine KI, vermutlich auf Tokens high