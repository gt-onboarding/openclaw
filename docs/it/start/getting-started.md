---
title: Primi passi
summary: "Guida introduttiva: da zero al primo messaggio (procedura guidata, autenticazione, canali, abbinamento)"
read_when:
  - Prima configurazione da zero
  - Vuoi il percorso più rapido da installazione → onboarding → primo messaggio
---

<div id="getting-started">
  # Introduzione
</div>

Obiettivo: passare da **zero** → **prima chat funzionante** (con impostazioni predefinite ragionevoli) il più rapidamente possibile.

Chat più veloce: apri la Control UI (nessuna configurazione di canale necessaria). Esegui `openclaw dashboard`
e chatta nel browser, oppure apri `http://127.0.0.1:18789/` sull&#39;host del Gateway.
Documentazione: [Dashboard](/it/web/dashboard) e [Control UI](/it/web/control-ui).

Percorso consigliato: usa il **wizard di onboarding della CLI** (`openclaw onboard`). Configura:

* modello/autenticazione (OAuth consigliato)
* impostazioni del Gateway
* canali (WhatsApp/Telegram/Discord/Mattermost (plugin)/...)
* impostazioni predefinite di abbinamento (DM sicuri)
* inizializzazione (bootstrap) dello spazio di lavoro + abilità
* servizio in background opzionale

Se vuoi le pagine di riferimento più approfondite, vai a: [Wizard](/it/start/wizard), [Setup](/it/start/setup), [Pairing](/it/start/pairing), [Security](/it/gateway/security).

Nota sul sandboxing: `agents.defaults.sandbox.mode: "non-main"` usa `session.mainKey` (predefinito `"main"`),
quindi le sessioni di gruppo/canale sono in sandbox. Se vuoi che l&#39;agente principale venga sempre
eseguito sull&#39;host, imposta un override esplicito per agente:

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
  ## 0) Prerequisiti
</div>

* Node `>=22`
* `pnpm` (opzionale; consigliato se compili da sorgente)
* **Consigliato:** chiave Brave Search API per la ricerca web. Il modo più semplice è:
  `openclaw configure --section web` (memorizza `tools.web.search.apiKey`).
  Vedi [Web tools](/it/tools/web).

macOS: se prevedi di creare le app, installa Xcode / CLT. Per la sola CLI e il Gateway è sufficiente Node.
Windows: usa **WSL2** (Ubuntu consigliato). WSL2 è fortemente consigliato; Windows nativo non è testato, è più problematico e ha una compatibilità degli strumenti inferiore. Installa prima WSL2, poi esegui i passaggi per Linux all&#39;interno di WSL. Vedi [Windows (WSL2)](/it/platforms/windows).

<div id="1-install-the-cli-recommended">
  ## 1) Installa la CLI (consigliato)
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Opzioni di installazione (metodo, modalità non interattiva, da GitHub): [Install](/it/install).

Windows (PowerShell):

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

In alternativa (installazione globale):

```bash
npm install -g openclaw@latest
```

```bash
pnpm add -g openclaw@latest
```

<div id="2-run-the-onboarding-wizard-and-install-the-service">
  ## 2) Esegui la procedura guidata di onboarding (e installa il servizio)
</div>

```bash
openclaw onboard --install-daemon
```

Cosa dovrai scegliere:

* **Gateway locale vs remoto**
* **Auth**: abbonamento OpenAI Code (Codex) (OAuth) o chiavi API. Per Anthropic consigliamo una chiave API; è supportato anche `claude setup-token`.
* **Providers**: login WhatsApp tramite QR, token bot Telegram/Discord, token plugin Mattermost, ecc.
* **Daemon**: installazione in background (launchd/systemd; WSL2 usa systemd)
  * **Runtime**: Node (consigliato; richiesto per WhatsApp/Telegram). Bun **non è consigliato**.
* **Gateway token**: il wizard ne genera uno per impostazione predefinita (anche su loopback) e lo salva in `gateway.auth.token`.

Documentazione del wizard: [Wizard](/it/start/wizard)

<div id="auth-where-it-lives-important">
  ### Auth: dove si trova (importante)
</div>

* **Percorso Anthropic consigliato:** imposta una chiave API (il wizard può salvarla per l&#39;uso da parte del servizio). `claude setup-token` è supportato anche se vuoi riutilizzare le credenziali di Claude Code.

* Credenziali OAuth (importazione legacy): `~/.openclaw/credentials/oauth.json`

* Profili di autenticazione (OAuth + chiavi API): `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

Suggerimento per modalità headless/server: esegui l&#39;OAuth prima su un computer normale, poi copia `oauth.json` sull&#39;host del Gateway.

<div id="3-start-the-gateway">
  ## 3) Avvia il Gateway
</div>

Se hai installato il servizio durante la procedura di onboarding, il Gateway dovrebbe essere già in esecuzione:

```bash
openclaw gateway status
```

Esecuzione manuale in primo piano:

```bash
openclaw gateway --port 18789 --verbose
```

Dashboard (loopback locale): `http://127.0.0.1:18789/`
Se hai configurato un token, incollalo nelle impostazioni del Control UI (viene memorizzato come `connect.params.auth.token`).

⚠️ **Avviso su Bun (WhatsApp + Telegram):** Bun presenta problemi noti con questi
canali. Se usi WhatsApp o Telegram, esegui il Gateway con **Node**.

<div id="35-quick-verify-2-min">
  ## 3.5) Verifica rapida (2 min)
</div>

```bash
openclaw status
openclaw health
openclaw security audit --deep
```

<div id="4-pair-connect-your-first-chat-surface">
  ## 4) Associa e collega il tuo primo canale di chat
</div>

<div id="whatsapp-qr-login">
  ### WhatsApp (accesso tramite QR)
</div>

```bash
openclaw channels login
```

Effettua la scansione da WhatsApp → Impostazioni → Dispositivi collegati.

Documentazione WhatsApp: [WhatsApp](/it/channels/whatsapp)

<div id="telegram-discord-others">
  ### Telegram / Discord / altri
</div>

La procedura guidata può scrivere per te i token/la configurazione. Se preferisci la configurazione manuale, parti da:

* Telegram: [Telegram](/it/channels/telegram)
* Discord: [Discord](/it/channels/discord)
* Mattermost (plugin): [Mattermost](/it/channels/mattermost)

**Suggerimento per i DM Telegram:** il tuo primo DM restituisce un codice di abbinamento. Approvalo (vedi il prossimo passaggio) oppure il bot non risponderà.

<div id="5-dm-safety-pairing-approvals">
  ## 5) Sicurezza DM (approvazioni di abbinamento)
</div>

Impostazione predefinita: i DM sconosciuti ricevono un codice corto e i messaggi non vengono elaborati finché non sono approvati.
Se al tuo primo DM non arriva risposta, approva l&#39;abbinamento:

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <code>
```

Documentazione sull&#39;abbinamento: [Abbinamento](/it/start/pairing)

<div id="from-source-development">
  ## Dal codice sorgente (sviluppo)
</div>

Se stai lavorando direttamente allo sviluppo di OpenClaw, eseguilo dal codice sorgente:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # installa automaticamente le dipendenze della UI alla prima esecuzione
pnpm build
openclaw onboard --install-daemon
```

Se non hai ancora un&#39;installazione globale, esegui la fase di onboarding tramite `pnpm openclaw ...` dal repository.
`pnpm build` include anche gli asset A2UI; se ti serve eseguire solo quella fase, usa `pnpm canvas:a2ui:bundle`.

Gateway (da questo repository):

```bash
node openclaw.mjs gateway --port 18789 --verbose
```

<div id="7-verify-end-to-end">
  ## 7) Verifica end-to-end
</div>

In un nuovo terminale, invia un messaggio di test:

```bash
openclaw message send --target +15555550123 --message "Ciao da OpenClaw"
```

Se `openclaw health` mostra “no auth configured”, torna alla procedura guidata e configura l’autenticazione OAuth/tramite chiave: l’agente non sarà in grado di rispondere senza di essa.

Suggerimento: `openclaw status --all` è il miglior report di debug, copiabile e in sola lettura.
Controlli di stato: `openclaw health` (oppure `openclaw status --deep`) chiede al Gateway in esecuzione un’istantanea dello stato di salute.

<div id="next-steps-optional-but-great">
  ## Prossimi passaggi (opzionali ma consigliati)
</div>

* App per la barra dei menu di macOS + attivazione vocale: [App macOS](/it/platforms/macos)
* Nodi iOS/Android (Canvas/fotocamera/voce): [Nodi](/it/nodes)
* Accesso remoto (tunnel SSH / Tailscale Serve): [Accesso remoto](/it/gateway/remote) e [Tailscale](/it/gateway/tailscale)
* Configurazioni sempre attive / VPN: [Accesso remoto](/it/gateway/remote), [exe.dev](/it/platforms/exe-dev), [Hetzner](/it/platforms/hetzner), [macOS remoto](/it/platforms/mac/remote)