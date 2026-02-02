---
title: Procedura guidata
summary: "Procedura guidata CLI per l'onboarding: configurazione assistita di Gateway, spazio di lavoro, canali e abilità"
read_when:
  - Durante l'esecuzione o la configurazione della procedura guidata di onboarding
  - Durante la configurazione di una nuova macchina
---

<div id="onboarding-wizard-cli">
  # Procedura guidata di onboarding (CLI)
</div>

La procedura guidata di onboarding è il metodo **consigliato** per configurare OpenClaw su macOS,
Linux o Windows (tramite WSL2; vivamente consigliato).
Configura un Gateway locale o una connessione a un Gateway remoto, oltre a canali, abilità
e impostazioni predefinite dello spazio di lavoro in un unico flusso guidato.

Punto di ingresso principale:

```bash
openclaw onboard
```

Chat più rapida in assoluto: apri la Control UI (non è necessaria alcuna configurazione di canali). Esegui
`openclaw dashboard` e chatta nel browser. Documentazione: [Dashboard](/it/web/dashboard).

Riconfigurazione successiva:

```bash
openclaw configure
```

Consigliato: configura una chiave API di Brave Search in modo che l&#39;agente possa usare `web_search`
(`web_fetch` funziona anche senza una chiave). Metodo più semplice: `openclaw configure --section web`,
che salva `tools.web.search.apiKey`. Documentazione: [Strumenti Web](/it/tools/web).

<div id="quickstart-vs-advanced">
  ## QuickStart vs Advanced
</div>

La procedura guidata si apre con **QuickStart** (impostazioni predefinite) oppure **Advanced** (controllo completo).

**QuickStart** mantiene i valori predefiniti:

* Gateway locale (loopback)
* Spazio di lavoro predefinito (o spazio di lavoro esistente)
* Porta del Gateway **18789**
* Autenticazione del Gateway tramite **Token** (generato automaticamente, anche su loopback)
* Esposizione tramite Tailscale **Off**
* I messaggi diretti (DM) di Telegram + WhatsApp usano per impostazione predefinita la **lista di autorizzati** (ti verrà richiesto il tuo numero di telefono)

**Advanced** espone ogni singolo passaggio (modalità, spazio di lavoro, Gateway, canali, demone, abilità).

<div id="what-the-wizard-does">
  ## Cosa fa la procedura guidata
</div>

**Modalità locale (predefinita)** ti guida nei passaggi seguenti:

* Modello/autenticazione (abbonamento OpenAI Code (Codex) OAuth, chiave API Anthropic (consigliata) o setup-token (incolla), oltre alle opzioni MiniMax/GLM/Moonshot/AI Gateway)
* Posizione dello spazio di lavoro + file di bootstrap
* Impostazioni del Gateway (port/bind/auth/Tailscale)
* Provider (Telegram, WhatsApp, Discord, Google Chat, Mattermost (plugin), Signal)
* Installazione del demone (LaunchAgent / unità utente systemd)
* Verifica dello stato (health check)
* Abilità (consigliate)

La **modalità remota** configura solo il client locale per connettersi a un Gateway remoto.
**Non** installa né modifica nulla sull&#39;host remoto.

Per aggiungere agenti più isolati tra loro (spazio di lavoro separato + sessioni + auth), usa:

```bash
openclaw agents add <name>
```

Suggerimento: `--json` **non** attiva automaticamente la modalità non interattiva. Per gli script usa `--non-interactive` (e `--workspace`).

<div id="flow-details-local">
  ## Dettagli del flusso (locale)
</div>

1. **Rilevamento configurazione esistente**
   * Se `~/.openclaw/openclaw.json` esiste, scegli **Keep / Modify / Reset**.
   * Rieseguire il wizard **non** cancella nulla a meno che tu non scelga esplicitamente **Reset**
     (o passi `--reset`).
   * Se la configurazione non è valida o contiene chiavi legacy, il wizard si ferma e ti chiede
     di eseguire `openclaw doctor` prima di continuare.
   * Reset usa `trash` (mai `rm`) e offre i seguenti ambiti:
     * Solo configurazione
     * Configurazione + credenziali + sessioni
     * Reset completo (rimuove anche lo spazio di lavoro)

2. **Modello/Auth**
   * **Chiave API Anthropic (consigliata)**: usa `ANTHROPIC_API_KEY` se presente oppure richiede una chiave, quindi la salva per l’uso da parte del demone.
   * **Anthropic OAuth (Claude Code CLI)**: su macOS il wizard controlla l’elemento del Portachiavi &quot;Claude Code-credentials&quot; (scegli &quot;Always Allow&quot; così gli avvii di launchd non vengono bloccati); su Linux/Windows riutilizza `~/.claude/.credentials.json` se presente.
   * **Token Anthropic (incolla setup-token)**: esegui `claude setup-token` su qualsiasi macchina, quindi incolla il token (puoi assegnargli un nome; vuoto = predefinito).
   * **Abbonamento OpenAI Code (Codex) (CLI Codex)**: se `~/.codex/auth.json` esiste, il wizard può riutilizzarlo.
   * **Abbonamento OpenAI Code (Codex) (OAuth)**: flusso tramite browser; incolla il `code#state`.
     * Imposta `agents.defaults.model` su `openai-codex/gpt-5.2` quando il modello non è impostato o è `openai/*`.
   * **Chiave API OpenAI**: usa `OPENAI_API_KEY` se presente oppure richiede una chiave, quindi la salva in `~/.openclaw/.env` così launchd può leggerla.
   * **OpenCode Zen (proxy multi‑modello)**: richiede `OPENCODE_API_KEY` (o `OPENCODE_ZEN_API_KEY`, ottienila su https://opencode.ai/auth).
   * **Chiave API**: memorizza la chiave per te.
   * **Vercel AI Gateway (proxy multi‑modello)**: richiede `AI_GATEWAY_API_KEY`.
   * Maggiori dettagli: [Vercel AI Gateway](/it/providers/vercel-ai-gateway)
   * **MiniMax M2.1**: la configurazione viene scritta automaticamente.
   * Maggiori dettagli: [MiniMax](/it/providers/minimax)
   * **Synthetic (compatibile Anthropic)**: richiede `SYNTHETIC_API_KEY`.
   * Maggiori dettagli: [Synthetic](/it/providers/synthetic)
   * **Moonshot (Kimi K2)**: la configurazione viene scritta automaticamente.
   * **Kimi Code**: la configurazione viene scritta automaticamente.
   * Maggiori dettagli: [Moonshot AI (Kimi + Kimi Code)](/it/providers/moonshot)
   * **Skip**: nessuna autenticazione configurata per ora.
   * Scegli un modello predefinito tra le opzioni rilevate (oppure inserisci manualmente provider/modello).
   * Il wizard esegue un controllo del modello e avvisa se il modello configurato è sconosciuto o se manca l’autenticazione.

* Le credenziali OAuth si trovano in `~/.openclaw/credentials/oauth.json`; i profili di autenticazione si trovano in `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` (chiavi API + OAuth).
  * Maggiori dettagli: [/concepts/oauth](/it/concepts/oauth)

3. **Spazio di lavoro**
   * Predefinito `~/.openclaw/workspace` (configurabile).
   * Inizializza i file dello spazio di lavoro necessari per la procedura di bootstrap dell&#39;agente.
   * Struttura completa dello spazio di lavoro + guida al backup: [Spazio di lavoro dell&#39;Agente](/it/concepts/agent-workspace)

4. **Gateway**
   * Porta, bind, modalità di autenticazione, esposizione Tailscale.
   * Raccomandazione per l&#39;autenticazione: mantieni **Token** anche per il loopback, così i client WS locali devono autenticarsi.
   * Disabilita l&#39;autenticazione solo se ti fidi completamente di ogni processo locale.
   * I bind non‑loopback richiedono comunque l&#39;autenticazione.

5. **Canali**
   * [WhatsApp](/it/channels/whatsapp): login tramite QR opzionale.
   * [Telegram](/it/channels/telegram): bot token.
   * [Discord](/it/channels/discord): bot token.
   * [Google Chat](/it/channels/googlechat): JSON dell&#39;account di servizio + audience del webhook.
   * [Mattermost](/it/channels/mattermost) (plugin): bot token + URL di base.
   * [Signal](/it/channels/signal): installazione opzionale di `signal-cli` + configurazione account.
   * [iMessage](/it/channels/imessage): percorso locale della CLI `imsg` + accesso al DB.
   * Sicurezza DM: l&#39;impostazione predefinita è l&#39;abbinamento. Il primo DM invia un codice; approva tramite `openclaw pairing approve <channel> <code>` oppure usa liste di autorizzati.

6. **Installazione del demone**
   * macOS: LaunchAgent
     * Richiede una sessione utente con login attivo; per ambienti headless, usa un LaunchDaemon personalizzato (non fornito).
   * Linux (e Windows via WSL2): unità utente systemd
     * Il wizard tenta di abilitare il lingering tramite `loginctl enable-linger <user>` così il Gateway resta attivo dopo il logout.
     * Può richiedere sudo (scrive in `/var/lib/systemd/linger`); prova prima senza sudo.
   * **Selezione runtime:** Node (consigliato; richiesto per WhatsApp/Telegram). Bun **non è consigliato**.

7. **Health check**
   * Avvia il Gateway (se necessario) ed esegue `openclaw health`.
   * Suggerimento: `openclaw status --deep` aggiunge sonde di stato del Gateway all&#39;output di stato (richiede un Gateway raggiungibile).

8. **Abilità (consigliato)**
   * Legge le abilità disponibili e verifica i requisiti.
   * Ti permette di scegliere un gestore dei pacchetti: **npm / pnpm** (bun non consigliato).
   * Installa dipendenze opzionali (alcune usano Homebrew su macOS).

9. **Fine**
   * Riepilogo + prossimi passi, incluse app iOS/Android/macOS per funzionalità aggiuntive.

* Se non viene rilevata alcuna GUI, il wizard stampa le istruzioni di port forwarding SSH per il Control UI invece di aprire un browser.
  * Se le risorse del Control UI mancano, il wizard tenta di eseguirne il build; il fallback è `pnpm ui:build` (installa automaticamente le dipendenze UI).

<div id="remote-mode">
  ## Modalità remota
</div>

La modalità remota configura un client locale per connettersi a un Gateway ospitato altrove.

Elementi da configurare:

* URL del Gateway remoto (`ws://...`)
* Token, se il Gateway remoto richiede autenticazione (consigliato)

Note:

* Non vengono eseguite installazioni remote né modifiche ai daemon.
* Se il Gateway accetta connessioni solo in loopback, usa il tunneling SSH o una tailnet.
* Suggerimenti per il rilevamento/discovery:
  * macOS: Bonjour (`dns-sd`)
  * Linux: Avahi (`avahi-browse`)

<div id="add-another-agent">
  ## Aggiungi un altro agente
</div>

Usa `openclaw agents add <name>` per creare un agente separato con il proprio spazio di lavoro,
le proprie sessioni e i propri profili di autenticazione. Eseguirlo senza `--workspace` avvia la procedura guidata.

Cosa configura:

* `agents.list[].name`
* `agents.list[].workspace`
* `agents.list[].agentDir`

Note:

* Gli spazi di lavoro predefiniti seguono `~/.openclaw/workspace-<agentId>`.
* Aggiungi `bindings` per instradare i messaggi in ingresso (la procedura guidata può farlo).
* Flag non interattivi: `--model`, `--agent-dir`, `--bind`, `--non-interactive`.

<div id="noninteractive-mode">
  ## Modalità non‑interattiva
</div>

Usa `--non-interactive` per automatizzare l’onboarding o integrarlo in script:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

Aggiungi `--json` per un riepilogo in formato leggibile dalla macchina.

Esempio con Gemini:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice gemini-api-key \
  --gemini-api-key "$GEMINI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Esempio di Z.AI:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice zai-api-key \
  --zai-api-key "$ZAI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Esempio di Vercel AI Gateway:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Esempio di Moonshot:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice moonshot-api-key \
  --moonshot-api-key "$MOONSHOT_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Esempio fittizio:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice synthetic-api-key \
  --synthetic-api-key "$SYNTHETIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Esempio OpenCode Zen:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice opencode-zen \
  --opencode-zen-api-key "$OPENCODE_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Esempio (non interattivo) di aggiunta di un agente:

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

<div id="gateway-wizard-rpc">
  ## RPC del wizard del Gateway
</div>

Il Gateway espone il flusso del wizard tramite RPC (`wizard.start`, `wizard.next`, `wizard.cancel`, `wizard.status`).
I client (app macOS, Control UI) possono visualizzare le diverse fasi senza dover implementare nuovamente la logica di onboarding.

<div id="signal-setup-signal-cli">
  ## Configurazione di Signal (signal-cli)
</div>

Il wizard può installare `signal-cli` dalle release di GitHub:

* Scarica il file di release appropriato.
* Lo salva in `~/.openclaw/tools/signal-cli/<version>/`.
* Scrive `channels.signal.cliPath` nella tua configurazione.

Note:

* Le build JVM richiedono **Java 21**.
* Le build native vengono usate quando disponibili.
* Su Windows viene usato WSL2; l&#39;installazione di signal-cli segue il flusso Linux all&#39;interno di WSL.

<div id="what-the-wizard-writes">
  ## Cosa scrive il wizard
</div>

Campi tipici in `~/.openclaw/openclaw.json`:

* `agents.defaults.workspace`
* `agents.defaults.model` / `models.providers` (se hai scelto Minimax)
* `gateway.*` (mode, bind, auth, tailscale)
* `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
* Liste di autorizzati dei canali (Slack/Discord/Matrix/Microsoft Teams) quando scegli di abilitarle durante i prompt (i nomi vengono risolti in ID quando possibile).
* `skills.install.nodeManager`
* `wizard.lastRunAt`
* `wizard.lastRunVersion`
* `wizard.lastRunCommit`
* `wizard.lastRunCommand`
* `wizard.lastRunMode`

`openclaw agents add` scrive `agents.list[]` e gli eventuali `bindings`.

Le credenziali WhatsApp vanno in `~/.openclaw/credentials/whatsapp/<accountId>/`.
Le sessioni sono archiviate in `~/.openclaw/agents/<agentId>/sessions/`.

Alcuni canali sono distribuiti come plugin. Quando ne scegli uno durante l&#39;onboarding, il wizard
ti chiederà di installarlo (npm o un percorso locale) prima che possa essere configurato.

<div id="related-docs">
  ## Documentazione correlata
</div>

* Onboarding dell&#39;app macOS: [Onboarding](/it/start/onboarding)
* Riferimenti di configurazione: [Configurazione del Gateway](/it/gateway/configuration)
* Provider: [WhatsApp](/it/channels/whatsapp), [Telegram](/it/channels/telegram), [Discord](/it/channels/discord), [Google Chat](/it/channels/googlechat), [Signal](/it/channels/signal), [iMessage](/it/channels/imessage)
* Abilità: [Abilità](/it/tools/skills), [Configurazione delle abilità](/it/tools/skills-config)