---
title: Indice
summary: "Panoramica di alto livello su OpenClaw, le sue funzionalità e il suo scopo"
read_when:
  - Presentare OpenClaw a chi lo utilizza per la prima volta
---

<div id="openclaw">
  # OpenClaw 🦞
</div>

> *&quot;ESFOLIA! ESFOLIA!&quot;* — Una aragosta spaziale, probabilmente

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text-dark.png" />

    <img src="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text.png" alt="OpenClaw" width="500" />
  </picture>
</p>

<p align="center">
  <strong>Qualsiasi sistema operativo + gateway WhatsApp/Telegram/Discord/iMessage per agenti IA (Pi).</strong><br />
  I plugin aggiungono Mattermost e altro.
  Invia un messaggio, ottieni la risposta di un agente — direttamente dalla tua tasca.
</p>

<p align="center">
  <a href="https://github.com/openclaw/openclaw">GitHub</a> ·
  <a href="https://github.com/openclaw/openclaw/releases">Releases</a> ·
  <a href="/it/">Documentazione</a> ·
  <a href="/it/start/openclaw">Configurazione dell&#39;assistente OpenClaw</a>
</p>

OpenClaw collega WhatsApp (tramite WhatsApp Web / Baileys), Telegram (Bot API / grammY), Discord (Bot API / channels.discord.js) e iMessage (imsg CLI) ad agenti per il coding come [Pi](https://github.com/badlogic/pi-mono). I plugin aggiungono Mattermost (Bot API + WebSocket) e altro.
OpenClaw è anche il motore dell&#39;assistente OpenClaw.

<div id="start-here">
  ## Inizia qui
</div>

* **Nuova installazione da zero:** [Guida introduttiva](/it/start/getting-started)
* **Configurazione guidata (consigliata):** [Wizard](/it/start/wizard) (`openclaw onboard`)
* **Apri la dashboard (Gateway locale):** http://127.0.0.1:18789/ (oppure http://localhost:18789/)

Se il Gateway è in esecuzione sullo stesso computer, quel link apre immediatamente la Control UI nel browser. Se non funziona, avvia prima il Gateway: `openclaw gateway`.

<div id="dashboard-browser-control-ui">
  ## Dashboard (Control UI nel browser)
</div>

La dashboard è la Control UI nel browser per chat, configurazione, nodi, sessioni e altro.
Indirizzo locale predefinito: http://127.0.0.1:18789/
Accesso remoto: [Interfacce web](/it/web) e [Tailscale](/it/gateway/tailscale)

<p align="center">
  <img src="/whatsapp-openclaw.jpg" alt="OpenClaw" width="420" />
</p>

<div id="how-it-works">
  ## Come funziona
</div>

```
WhatsApp / Telegram / Discord / iMessage (+ plugins)
        │
        ▼
  ┌───────────────────────────┐
  │          Gateway          │  ws://127.0.0.1:18789 (loopback-only)
  │     (sorgente singola)    │
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

La maggior parte delle operazioni passa attraverso il **Gateway** (`openclaw gateway`), un unico processo a lunga esecuzione che mantiene le connessioni ai canali e il piano di controllo WebSocket.

<div id="network-model">
  ## Modello di rete
</div>

* **Un solo Gateway per host (raccomandato)**: è l&#39;unico processo autorizzato a mantenere la sessione WhatsApp Web. Se ti serve un bot di emergenza o un isolamento rigoroso, esegui più Gateway con profili e porte isolati; vedi [Multiple gateways](/it/gateway/multiple-gateways).
* **Loopback-first**: il WS del Gateway usa per default `ws://127.0.0.1:18789`.
  * Il wizard ora genera un token del Gateway per impostazione predefinita (anche per il loopback).
  * Per l&#39;accesso tramite Tailnet, esegui `openclaw gateway --bind tailnet --token ...` (il token è obbligatorio per i bind non loopback).
* **Nodi**: si connettono al Gateway tramite WebSocket (LAN/tailnet/SSH secondo necessità); il bridge TCP legacy è stato deprecato/rimosso.
* **Canvas host**: server di file HTTP su `canvasHost.port` (default `18793`), che espone `/__openclaw__/canvas/` per le WebView dei nodi; vedi [Gateway configuration](/it/gateway/configuration) (`canvasHost`).
* **Utilizzo remoto**: tunnel SSH o tailnet/VPN; vedi [Remote access](/it/gateway/remote) e [Discovery](/it/gateway/discovery).

<div id="features-high-level">
  ## Funzionalità (di alto livello)
</div>

* 📱 **Integrazione WhatsApp** — Usa Baileys per il protocollo WhatsApp Web
* ✈️ **Bot Telegram** — DM + gruppi tramite grammY
* 🎮 **Bot Discord** — DM + canali delle guild tramite channels.discord.js
* 🧩 **Bot Mattermost (plugin)** — Token bot + eventi WebSocket
* 💬 **iMessage** — Integrazione locale con CLI imsg (macOS)
* 🤖 **Bridge agente** — Pi (modalità RPC) con streaming degli strumenti
* ⏱️ **Streaming + chunking** — Streaming a blocchi + dettagli sullo streaming delle bozze Telegram ([/concepts/streaming](/it/concepts/streaming))
* 🧠 **Instradamento multi‑agente** — Instrada account/peer dei provider verso agenti isolati (spazio di lavoro + sessioni per agente)
* 🔐 **Autenticazione tramite abbonamento** — Anthropic (Claude Pro/Max) + OpenAI (ChatGPT/Codex) via OAuth
* 💬 **Sessioni** — Le chat dirette confluiscono nella `main` condivisa (predefinita); i gruppi sono isolati
* 👥 **Supporto per le chat di gruppo** — Basato sulle menzioni per impostazione predefinita; il proprietario può attivare/disattivare `/activation always|mention`
* 📎 **Supporto contenuti multimediali** — Invia e riceve immagini, audio, documenti
* 🎤 **Note vocali** — Hook di trascrizione opzionale
* 🖥️ **WebChat + app macOS** — UI locale + companion nella barra dei menu per operazioni e attivazione vocale
* 📱 **Nodo iOS** — Effettua il pairing come nodo ed espone una superficie Canvas
* 📱 **Nodo Android** — Effettua il pairing come nodo ed espone Canvas + Chat + Camera

Nota: i percorsi legacy Claude/Codex/Gemini/Opencode sono stati rimossi; Pi è l&#39;unico percorso di agente dedicato al coding.

<div id="quick-start">
  ## Avvio rapido
</div>

Requisito di runtime: **Node ≥ 22**.

```bash
# Recommended: global install (npm/pnpm)
npm install -g openclaw@latest
# or: pnpm add -g openclaw@latest

# Onboard + install the service (launchd/systemd user service)
openclaw onboard --install-daemon

# Pair WhatsApp Web (shows QR)
openclaw channels login

# Il Gateway viene eseguito tramite il servizio dopo l'onboarding; l'esecuzione manuale è comunque possibile:
openclaw gateway --port 18789
```

Passare in un secondo momento da un&#39;installazione via npm a una da git (e viceversa) è semplice: installa l&#39;altra variante ed esegui `openclaw doctor` per aggiornare l&#39;entrypoint del servizio Gateway.

Da sorgente (sviluppo):

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # installa automaticamente le dipendenze della UI alla prima esecuzione
pnpm build
openclaw onboard --install-daemon
```

Se non hai ancora un&#39;installazione globale, esegui la procedura di onboarding tramite `pnpm openclaw ...` dal repository.

Avvio rapido multiistanza (opzionale):

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

Invia un messaggio di prova (richiede che il Gateway sia in esecuzione):

```bash
openclaw message send --target +15555550123 --message "Ciao da OpenClaw"
```

<div id="configuration-optional">
  ## Configurazione (opzionale)
</div>

La configurazione si trova in `~/.openclaw/openclaw.json`.

* Se **non fai nulla**, OpenClaw utilizza il binario Pi incluso in modalità RPC con sessioni separate per ciascun mittente.
* Se vuoi limitarne l&#39;accesso, parti da `channels.whatsapp.allowFrom` e (per i gruppi) imposta le regole di menzione.

Esempio:

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
  ## Documentazione
</div>

* Inizia da qui:
  * [Hub della documentazione (tutte le pagine collegate)](/it/start/hubs)
  * [Guida](/it/help) ← *correzioni comuni + risoluzione dei problemi*
  * [Configurazione](/it/gateway/configuration)
  * [Esempi di configurazione](/it/gateway/configuration-examples)
  * [Comandi slash](/it/tools/slash-commands)
  * [Instradamento multi‑agente](/it/concepts/multi-agent)
  * [Aggiornamento / rollback](/it/install/updating)
  * [Abbinamento (DM + nodi)](/it/start/pairing)
  * [Modalità Nix](/it/install/nix)
  * [Configurazione dell&#39;assistente OpenClaw](/it/start/openclaw)
  * [Abilità](/it/tools/skills)
  * [Configurazione delle abilità](/it/tools/skills-config)
  * [Modelli di spazio di lavoro](/it/reference/templates/AGENTS)
  * [Adattatori RPC](/it/reference/rpc)
  * [Runbook del Gateway](/it/gateway)
  * [Nodi (iOS/Android)](/it/nodes)
  * [Interfacce web (Control UI)](/it/web)
  * [Discovery + trasporti](/it/gateway/discovery)
  * [Accesso remoto](/it/gateway/remote)
* Provider e UX:
  * [WebChat](/it/web/webchat)
  * [Control UI (browser)](/it/web/control-ui)
  * [Telegram](/it/channels/telegram)
  * [Discord](/it/channels/discord)
  * [Mattermost (plugin)](/it/channels/mattermost)
  * [iMessage](/it/channels/imessage)
  * [Gruppi](/it/concepts/groups)
  * [Messaggi di gruppo WhatsApp](/it/concepts/group-messages)
  * [Media: immagini](/it/nodes/images)
  * [Media: audio](/it/nodes/audio)
* App companion:
  * [App macOS](/it/platforms/macos)
  * [App iOS](/it/platforms/ios)
  * [App Android](/it/platforms/android)
  * [Windows (WSL2)](/it/platforms/windows)
  * [App Linux](/it/platforms/linux)
* Operazioni e sicurezza:
  * [Sessioni](/it/concepts/session)
  * [Processi cron](/it/automation/cron-jobs)
  * [Webhook](/it/automation/webhook)
  * [Hook Gmail (Pub/Sub)](/it/automation/gmail-pubsub)
  * [Sicurezza](/it/gateway/security)
  * [Risoluzione dei problemi](/it/gateway/troubleshooting)

<div id="the-name">
  ## Il nome
</div>

**OpenClaw = CLAW + TARDIS** — perché ogni aragosta spaziale ha bisogno di una macchina spazio-temporale.

***

*&quot;In fondo stiamo tutti solo giocando con i nostri prompt.&quot;* — un&#39;IA, probabilmente su di giri per i token

<div id="credits">
  ## Crediti
</div>

* **Peter Steinberger** ([@steipete](https://twitter.com/steipete)) — Creatore, sussurratore di aragoste
* **Mario Zechner** ([@badlogicc](https://twitter.com/badlogicgames)) — Creatore di Pi, esperto di sicurezza e penetration testing
* **Clawd** — L&#39;aragosta spaziale che ha preteso un nome migliore

<div id="core-contributors">
  ## Principali contributori
</div>

* **Maxim Vovshin** (@Hyaxia, 36747317+Hyaxia@users.noreply.github.com) — skill Blogwatcher
* **Nacho Iacovino** (@nachoiacovino, nacho.iacovino@gmail.com) — analisi delle posizioni (Telegram e WhatsApp)

<div id="license">
  ## Licenza
</div>

MIT — Libero come un&#39;aragosta in mare aperto 🦞

***

*&quot;In fondo stiamo solo giocando con i nostri prompt.&quot;* — Un&#39;IA, probabilmente fatta di troppi token