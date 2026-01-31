---
title: Index
summary: "Top-level overview of OpenClaw, features, and purpose"
read_when:
  - Introducing OpenClaw to newcomers
---

<div id="openclaw">
  # OpenClaw 🦞
</div>

> _"EXFOLIATE! EXFOLIATE!"_ — A space lobster, probably

<p align="center">
    <picture>
        <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text-dark.png">
        <img src="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text.png" alt="OpenClaw" width="500">
    </picture>
</p>

<p align='center'>
  <strong>
    Any OS + WhatsApp/Telegram/Discord/iMessage gateway for AI agents (Pi).
  </strong>
  <br />
  Plugins add Mattermost and more. Send a message, get an agent response — from
  your pocket.
</p>

<p align='center'>
  <a href='https://github.com/openclaw/openclaw'>GitHub</a> ·
  <a href='https://github.com/openclaw/openclaw/releases'>Releases</a> ·
  <a href='/'>Docs</a> ·<a href='/start/openclaw'>OpenClaw assistant setup</a>
</p>

OpenClaw verbindet WhatsApp (über WhatsApp Web / Baileys), Telegram (Bot API / grammY), Discord (Bot API / channels.discord.js) und iMessage (imsg CLI) mit Coding-Agenten wie [Pi](https://github.com/badlogic/pi-mono). Plugins fügen Mattermost (Bot API + WebSocket) und mehr hinzu.
OpenClaw betreibt auch den OpenClaw-Assistenten.


<div id="start-here">
  ## Hier starten
</div>

- **Neuinstallation von Grund auf:** [Erste Schritte](/start/getting-started)
- **Geführte Einrichtung (empfohlen):** [Wizard](/start/wizard) (`openclaw onboard`)
- **Dashboard öffnen (lokales Gateway):** http://127.0.0.1:18789/ (oder http://localhost:18789/)

Wenn das Gateway auf demselben Computer läuft, öffnet dieser Link die Control UI
sofort im Browser. Falls das nicht klappt, starte zuerst das Gateway: `openclaw gateway`.



<div id="dashboard-browser-control-ui">
  ## Dashboard (browser Control UI)
</div>

Das Dashboard ist die browserbasierte Control UI für Chat, Konfiguration, Knoten, Sitzungen und mehr.
Lokale Standardadresse: http://127.0.0.1:18789/
Remotezugriff: [Web surfaces](/web) und [Tailscale](/gateway/tailscale)

<p align="center">
  <img src="whatsapp-openclaw.jpg" alt="OpenClaw" width="420" />
</p>



<div id="how-it-works">
  ## So funktioniert es
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

Die meisten Operationen laufen über den **Gateway** (`openclaw gateway`), einen einzelnen, dauerhaft laufenden Prozess, der die Channel-Verbindungen sowie die WebSocket-Control-Plane kapselt und steuert.


<div id="network-model">
  ## Netzwerkmodell
</div>

- **Ein Gateway pro Host (empfohlen)**: Es ist der einzige Prozess, der die WhatsApp-Web-Sitzung besitzen darf. Wenn du einen Rescue-Bot oder strikte Isolation benötigst, betreibe mehrere Gateways mit isolierten Profilen und Ports; siehe [Mehrere Gateways](/gateway/multiple-gateways).
- **Loopback-first**: Gateway-WS verwendet standardmäßig `ws://127.0.0.1:18789`.
  - Der Assistent erzeugt jetzt standardmäßig ein Gateway-Token (auch für Loopback).
  - Für Tailnet-Zugriff führe `openclaw gateway --bind tailnet --token ...` aus (ein Token ist für Bindungen außerhalb von Loopback erforderlich).
- **Knoten**: verbinden sich mit dem Gateway-WebSocket (LAN/Tailnet/SSH nach Bedarf); die Legacy-TCP-Bridge ist veraltet und wurde entfernt.
- **Canvas-Host**: HTTP-Dateiserver auf `canvasHost.port` (Standard `18793`), der `/__openclaw__/canvas/` für Knoten-WebViews bereitstellt; siehe [Gateway-Konfiguration](/gateway/configuration) (`canvasHost`).
- **Remote-Nutzung**: SSH-Tunnel oder Tailnet/VPN; siehe [Remote-Zugriff](/gateway/remote) und [Discovery](/gateway/discovery).



<div id="features-high-level">
  ## Features (high level)
</div>

- 📱 **WhatsApp-Integration** — Verwendet Baileys für das WhatsApp-Webprotokoll
- ✈️ **Telegram-Bot** — DMs + Gruppen über grammY
- 🎮 **Discord-Bot** — DMs + Guild-Channels über channels.discord.js
- 🧩 **Mattermost-Bot (Plugin)** — Bot-Token + WebSocket-Events
- 💬 **iMessage** — Lokale imsg CLI-Integration (macOS)
- 🤖 **Agent-Bridge** — Pi (RPC-Modus) mit Tool-Streaming
- ⏱️ **Streaming + Chunking** — Block-Streaming + Details zum Telegram-Draft-Streaming ([/concepts/streaming](/concepts/streaming))
- 🧠 **Multi-agent-Routing** — Leitet Anbieter-Konten/-Peers an isolierte Agenten weiter (Arbeitsbereich + Sitzungen pro Agent)
- 🔐 **Abonnement-Authentifizierung** — Anthropic (Claude Pro/Max) + OpenAI (ChatGPT/Codex) via OAuth
- 💬 **Sitzungen** — Direkte Chats werden in eine gemeinsame `main`-Sitzung (Standard) zusammengeführt; Gruppen sind isoliert
- 👥 **Gruppenchat-Unterstützung** — Standardmäßig erwähnungsbasiert; Owner kann `/activation always|mention` umschalten
- 📎 **Medienunterstützung** — Senden und Empfangen von Bildern, Audio, Dokumenten
- 🎤 **Sprachnachrichten** — Optionaler Transkriptions-Hook
- 🖥️ **WebChat + macOS-App** — Lokale UI + Menüleisten-Begleiter für Ops und Sprachaktivierung
- 📱 **iOS-Knoten** — Wird als Knoten gekoppelt und stellt eine Canvas-Oberfläche bereit
- 📱 **Android-Knoten** — Wird als Knoten gekoppelt und stellt Canvas + Chat + Kamera bereit

Hinweis: Veraltete Claude/Codex/Gemini/Opencode-Pfade wurden entfernt; Pi ist der einzige Coding-agent-Pfad.



<div id="quick-start">
  ## Schnellstart
</div>

Laufzeitvoraussetzung: **Node.js ≥ 22**.



```bash
# Empfohlen: Globale Installation (npm/pnpm)
npm install -g openclaw@latest
# oder: pnpm add -g openclaw@latest
```


# Onboarding + Installation des Dienstes (launchd/systemd-Benutzerdienst)
openclaw onboard --install-daemon



# WhatsApp Web koppeln (QR-Code anzeigen)
openclaw channels login



# Gateway wird nach dem Onboarding als Dienst ausgeführt; ein manueller Start ist weiterhin möglich:

openclaw gateway --port 18789

````

Der spätere Wechsel zwischen npm- und Git-Installationen ist einfach: Installieren Sie die andere Variante und führen Sie `openclaw doctor` aus, um den Einstiegspunkt des Gateway-Service zu aktualisieren.

Aus dem Quellcode (Entwicklung):

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # installiert UI-Abhängigkeiten beim ersten Ausführen automatisch
pnpm build
openclaw onboard --install-daemon
````

Wenn du noch keine globale Installation hast, führe den Onboarding-Schritt im Repo mit `pnpm openclaw ...` aus.

Multi-Instance-Schnellstart (optional):

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

Senden Sie eine Testnachricht (erfordert ein laufendes Gateway):

```bash
openclaw message send --target +15555550123 --message "Hallo von OpenClaw"
```


<div id="credits">
  ## Konfiguration (optional)
</div>

Die Konfiguration befindet sich unter `~/.openclaw/openclaw.json`.

* Wenn du **nichts machst**, verwendet OpenClaw das mitgelieferte Pi-Binary im RPC-Modus mit Sitzungen pro Absender.
* Wenn du es stärker absichern möchtest, fang mit `channels.whatsapp.allowFrom` an und (für Gruppen) mit Erwähnungsregeln.

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


<div id="core-contributors">
  ## Dokumentation
</div>

- Starte hier:
  - [Doku-Hubs (alle Seiten verlinkt)](/start/hubs)
  - [Hilfe](/help) ← *häufige Lösungen + Fehlerbehebung*
  - [Konfiguration](/gateway/configuration)
  - [Konfigurationsbeispiele](/gateway/configuration-examples)
  - [Slash-Befehle](/tools/slash-commands)
  - [Multi-Agent-Routing](/concepts/multi-agent)
  - [Aktualisierung / Rollback](/install/updating)
  - [Kopplung (DM + Knoten)](/start/pairing)
  - [Nix-Modus](/install/nix)
  - [Setup des OpenClaw Assistant](/start/openclaw)
  - [Fähigkeiten](/tools/skills)
  - [Fähigkeiten-Konfiguration](/tools/skills-config)
  - [Arbeitsbereichsvorlagen](/reference/templates/AGENTS)
  - [RPC-Adapter](/reference/rpc)
  - [Gateway-Runbook](/gateway)
  - [Knoten (iOS/Android)](/nodes)
  - [Web-Oberflächen (Control UI)](/web)
  - [Erkennung + Transportschichten](/gateway/discovery)
  - [Remotezugriff](/gateway/remote)
- Anbieter und UX:
  - [WebChat](/web/webchat)
  - [Control UI (Browser)](/web/control-ui)
  - [Telegram](/channels/telegram)
  - [Discord](/channels/discord)
  - [Mattermost (Plugin)](/channels/mattermost)
  - [iMessage](/channels/imessage)
  - [Gruppen](/concepts/groups)
  - [WhatsApp-Gruppennachrichten](/concepts/group-messages)
  - [Medien: Bilder](/nodes/images)
  - [Medien: Audio](/nodes/audio)
- Companion-Apps:
  - [macOS-App](/platforms/macos)
  - [iOS-App](/platforms/ios)
  - [Android-App](/platforms/android)
  - [Windows (WSL2)](/platforms/windows)
  - [Linux-App](/platforms/linux)
- Betrieb und Sicherheit:
  - [Sitzungen](/concepts/session)
  - [Cron-Jobs](/automation/cron-jobs)
  - [Webhooks](/automation/webhook)
  - [Gmail-Hooks (Pub/Sub)](/automation/gmail-pubsub)
  - [Sicherheit](/gateway/security)
  - [Fehlerbehebung](/gateway/troubleshooting)



<div id="license">
  ## Der Name
</div>

**OpenClaw = CLAW + TARDIS** — weil jeder Weltraum-Hummer eine Raum-und-Zeit-Maschine braucht.

---

*„Wir spielen doch alle nur mit unseren eigenen Prompts.“* — eine KI, vermutlich auf einem Token-High



## Danksagungen

- **Peter Steinberger** ([@steipete](https://twitter.com/steipete)) — Erfinder, Hummerflüsterer
- **Mario Zechner** ([@badlogicc](https://twitter.com/badlogicgames)) — Pi-Erfinder, Security-Penetrationstester
- **Clawd** — Der Weltraumhummer, der auf einen besseren Namen bestand



## Hauptmitwirkende

- **Maxim Vovshin** (@Hyaxia, 36747317+Hyaxia@users.noreply.github.com) — Blogwatcher-Skill
- **Nacho Iacovino** (@nachoiacovino, nacho.iacovino@gmail.com) — Standort-Parsing (Telegram + WhatsApp)



## Lizenz

MIT — Frei wie ein Hummer im Ozean 🦞

---

*„Wir spielen doch alle nur mit unseren eigenen Prompts herum.“* — Eine KI, wahrscheinlich auf einem Token-High
