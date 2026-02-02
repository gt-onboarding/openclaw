---
title: CLI
summary: "Riferimento della CLI di OpenClaw per i comandi, i sottocomandi e le opzioni di `openclaw`"
read_when:
  - Quando aggiungi o modifichi comandi o opzioni della CLI
  - Quando documenti nuove interfacce di comando
---

<div id="cli-reference">
  # Riferimento alla CLI
</div>

Questa pagina descrive il comportamento attuale della CLI. Se i comandi cambiano, aggiorna questo documento.

<div id="command-pages">
  ## Pagine dei comandi
</div>

* [`setup`](/it/cli/setup)
* [`onboard`](/it/cli/onboard)
* [`configure`](/it/cli/configure)
* [`config`](/it/cli/config)
* [`doctor`](/it/cli/doctor)
* [`dashboard`](/it/cli/dashboard)
* [`reset`](/it/cli/reset)
* [`uninstall`](/it/cli/uninstall)
* [`update`](/it/cli/update)
* [`message`](/it/cli/message)
* [`agent`](/it/cli/agent)
* [`agents`](/it/cli/agents)
* [`acp`](/it/cli/acp)
* [`status`](/it/cli/status)
* [`health`](/it/cli/health)
* [`sessions`](/it/cli/sessions)
* [`gateway`](/it/cli/gateway)
* [`logs`](/it/cli/logs)
* [`system`](/it/cli/system)
* [`models`](/it/cli/models)
* [`memory`](/it/cli/memory)
* [`nodes`](/it/cli/nodes)
* [`devices`](/it/cli/devices)
* [`node`](/it/cli/node)
* [`approvals`](/it/cli/approvals)
* [`sandbox`](/it/cli/sandbox)
* [`tui`](/it/cli/tui)
* [`browser`](/it/cli/browser)
* [`cron`](/it/cli/cron)
* [`dns`](/it/cli/dns)
* [`docs`](/it/cli/docs)
* [`hooks`](/it/cli/hooks)
* [`webhooks`](/it/cli/webhooks)
* [`pairing`](/it/cli/pairing)
* [`plugins`](/it/cli/plugins) (comandi plugin)
* [`channels`](/it/cli/channels)
* [`security`](/it/cli/security)
* [`skills`](/it/cli/skills)
* [`voicecall`](/it/cli/voicecall) (plugin; se installato)

<div id="global-flags">
  ## Flag globali
</div>

* `--dev`: isola lo stato all&#39;interno di `~/.openclaw-dev` e sposta le porte predefinite.
* `--profile <name>`: isola lo stato all&#39;interno di `~/.openclaw-<name>`.
* `--no-color`: disabilita i colori ANSI.
* `--update`: forma abbreviata di `openclaw update` (solo installazioni da sorgente).
* `-V`, `--version`, `-v`: stampa la versione ed esce.

<div id="output-styling">
  ## Stile dell&#39;output
</div>

* I colori ANSI e gli indicatori di avanzamento vengono visualizzati solo nelle sessioni TTY.
* I collegamenti ipertestuali OSC-8 vengono visualizzati come link cliccabili nei terminali supportati; in caso contrario si utilizzano semplici URL in chiaro.
* `--json` (e `--plain` dove supportato) disabilita la formattazione per un output pulito.
* `--no-color` disabilita la formattazione ANSI; viene rispettato anche `NO_COLOR=1`.
* I comandi di lunga esecuzione mostrano un indicatore di avanzamento (OSC 9;4 quando supportato).

<div id="color-palette">
  ## Tavolozza di colori
</div>

OpenClaw usa una tavolozza &quot;lobster&quot; per l&#39;output della CLI.

* `accent` (#FF5A2D): intestazioni, etichette, evidenziazioni primarie.
* `accentBright` (#FF7A3D): nomi di comandi, enfasi.
* `accentDim` (#D14A22): testo in evidenza secondaria.
* `info` (#FF8A5B): valori informativi.
* `success` (#2FBF71): stati di esito positivo.
* `warn` (#FFB020): avvisi, fallback, attenzione.
* `error` (#E23D2D): errori, guasti.
* `muted` (#8B7F77): elementi meno in evidenza, metadati.

Fonte di riferimento per la tavolozza: `src/terminal/palette.ts` (alias &quot;lobster seam&quot;).

<div id="command-tree">
  ## Struttura dei comandi
</div>

```
openclaw [--dev] [--profile <name>] <command>
  setup
  onboard
  configure
  config
    get
    set
    unset
  doctor
  security
    audit
  reset
  uninstall
  update
  channels
    list
    status
    logs
    add
    remove
    login
    logout
  skills
    list
    info
    check
  plugins
    list
    info
    install
    enable
    disable
    doctor
  memory
    status
    index
    search
  message
  agent
  agents
    list
    add
    delete
  acp
  status
  health
  sessions
  gateway
    call
    health
    status
    probe
    discover
    install
    uninstall
    start
    stop
    restart
    run
  logs
  system
    event
    heartbeat last|enable|disable
    presence
  models
    list
    status
    set
    set-image
    aliases list|add|remove
    fallbacks list|add|remove|clear
    image-fallbacks list|add|remove|clear
    scan
    auth add|setup-token|paste-token
    auth order get|set|clear
  sandbox
    list
    recreate
    explain
  cron
    status
    list
    add
    edit
    rm
    enable
    disable
    runs
    run
  nodes
  devices
  node
    run
    status
    install
    uninstall
    start
    stop
    restart
  approvals
    get
    set
    allowlist add|remove
  browser
    status
    start
    stop
    reset-profile
    tabs
    open
    focus
    close
    profiles
    create-profile
    delete-profile
    screenshot
    snapshot
    navigate
    resize
    click
    type
    press
    hover
    drag
    select
    upload
    fill
    dialog
    wait
    evaluate
    console
    pdf
  hooks
    list
    info
    check
    enable
    disable
    install
    update
  webhooks
    gmail setup|run
  pairing
    list
    approve
  docs
  dns
    setup
  tui
```

Nota: i plugin possono aggiungere nuovi comandi di primo livello (ad esempio `openclaw voicecall`).

<div id="security">
  ## Sicurezza
</div>

* `openclaw security audit` — analizza la configurazione e lo stato locale per individuare comuni errori di sicurezza.
* `openclaw security audit --deep` — sonda live il Gateway con approccio best-effort.
* `openclaw security audit --fix` — rafforza i valori sicuri predefiniti e applica chmod a stato/configurazione.

<div id="plugins">
  ## Plugin
</div>

Gestisci le estensioni e la loro configurazione:

* `openclaw plugins list` — individua i plugin (usa `--json` per un output in formato macchina).
* `openclaw plugins info <id>` — mostra i dettagli di un plugin.
* `openclaw plugins install <path|.tgz|npm-spec>` — installa un plugin (o aggiungi un percorso di plugin a `plugins.load.paths`).
* `openclaw plugins enable <id>` / `disable <id>` — attiva/disattiva `plugins.entries.<id>.enabled`.
* `openclaw plugins doctor` — segnala gli errori di caricamento dei plugin.

La maggior parte delle modifiche ai plugin richiede un riavvio del Gateway. Consulta [/plugin](/it/plugin).

<div id="memory">
  ## Memoria
</div>

Ricerca vettoriale in `MEMORY.md` + `memory/*.md`:

* `openclaw memory status` — mostra le statistiche dell&#39;indice.
* `openclaw memory index` — reindicizza i file di memoria.
* `openclaw memory search "<query>"` — esegue una ricerca semantica nella memoria.

<div id="chat-slash-commands">
  ## Comandi slash della chat
</div>

I messaggi in chat supportano comandi `/...` (testo e nativi). Consulta [/tools/slash-commands](/it/tools/slash-commands).

Punti principali:

* `/status` per una diagnostica rapida.
* `/config` per modifiche di configurazione persistenti.
* `/debug` per override di configurazione validi solo a runtime (in memoria, non su disco; richiede `commands.debug: true`).

<div id="setup-onboarding">
  ## Configurazione + onboarding
</div>

<div id="setup">
  ### `setup`
</div>

Inizializza la configurazione e lo spazio di lavoro.

Opzioni:

* `--workspace <dir>`: percorso dello spazio di lavoro dell&#39;agente (predefinito `~/.openclaw/workspace`).
* `--wizard`: esegui la procedura guidata di onboarding.
* `--non-interactive`: esegui la procedura guidata senza richieste di input.
* `--mode <local|remote>`: modalità della procedura guidata.
* `--remote-url <url>`: URL del Gateway remoto.
* `--remote-token <token>`: token del Gateway remoto.

La procedura guidata viene eseguita automaticamente quando è presente qualsiasi flag della procedura guidata (`--non-interactive`, `--mode`, `--remote-url`, `--remote-token`).

<div id="onboard">
  ### `onboard`
</div>

Procedura guidata interattiva per configurare il Gateway, lo spazio di lavoro e le abilità.

Opzioni:

* `--workspace <dir>`
* `--reset` (reimposta configurazione + credenziali + sessioni + spazio di lavoro prima della procedura guidata)
* `--non-interactive`
* `--mode <local|remote>`
* `--flow <quickstart|advanced|manual>` (manual è un alias di advanced)
* `--auth-choice <setup-token|token|chutes|openai-codex|openai-api-key|openrouter-api-key|ai-gateway-api-key|moonshot-api-key|kimi-code-api-key|synthetic-api-key|venice-api-key|gemini-api-key|zai-api-key|apiKey|minimax-api|minimax-api-lightning|opencode-zen|skip>`
* `--token-provider <id>` (non interattivo; usato con `--auth-choice token`)
* `--token <token>` (non interattivo; usato con `--auth-choice token`)
* `--token-profile-id <id>` (non interattivo; predefinito: `<provider>:manual`)
* `--token-expires-in <duration>` (non interattivo; ad es. `365d`, `12h`)
* `--anthropic-api-key <key>`
* `--openai-api-key <key>`
* `--openrouter-api-key <key>`
* `--ai-gateway-api-key <key>`
* `--moonshot-api-key <key>`
* `--kimi-code-api-key <key>`
* `--gemini-api-key <key>`
* `--zai-api-key <key>`
* `--minimax-api-key <key>`
* `--opencode-zen-api-key <key>`
* `--gateway-port <port>`
* `--gateway-bind <loopback|lan|tailnet|auto|custom>`
* `--gateway-auth <token|password>`
* `--gateway-token <token>`
* `--gateway-password <password>`
* `--remote-url <url>`
* `--remote-token <token>`
* `--tailscale <off|serve|funnel>`
* `--tailscale-reset-on-exit`
* `--install-daemon`
* `--no-install-daemon` (alias: `--skip-daemon`)
* `--daemon-runtime <node|bun>`
* `--skip-channels`
* `--skip-skills`
* `--skip-health`
* `--skip-ui`
* `--node-manager <npm|pnpm|bun>` (pnpm consigliato; bun non consigliato per il runtime del Gateway)
* `--json`

<div id="configure">
  ### `configure`
</div>

Configurazione guidata interattiva (modelli, canali, abilità, Gateway).

<div id="config">
  ### `config`
</div>

Utility di configurazione non interattive (get/set/unset). L&#39;esecuzione di `openclaw config` senza
sottocomando avvia la procedura guidata.

Sottocomandi:

* `config get <path>`: stampa un valore di configurazione (percorso con notazione a punti o con parentesi).
* `config set <path> <value>`: imposta un valore (JSON5 o stringa letterale).
* `config unset <path>`: rimuove un valore.

<div id="doctor">
  ### `doctor`
</div>

Verifiche di integrità + correzioni rapide (configurazione + Gateway + servizi legacy).

Opzioni:

* `--no-workspace-suggestions`: disabilita i suggerimenti di memoria dello spazio di lavoro.
* `--yes`: accetta i valori predefiniti senza chiedere conferma (headless).
* `--non-interactive`: salta le richieste di conferma; applica solo migrazioni sicure.
* `--deep`: analizza i servizi di sistema alla ricerca di installazioni aggiuntive del Gateway.

<div id="channel-helpers">
  ## Utility per i canali
</div>

<div id="channels">
  ### `channels`
</div>

Gestisci gli account dei canali chat (WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (plugin)/Signal/iMessage/MS Teams).

Sottocomandi:

* `channels list`: mostra i canali configurati e i profili di autenticazione.
* `channels status`: verifica la raggiungibilità del Gateway e lo stato di salute dei canali (`--probe` esegue controlli aggiuntivi; usa `openclaw health` o `openclaw status --deep` per i controlli di salute del Gateway).
* Suggerimento: `channels status` stampa avvisi con correzioni suggerite quando riesce a rilevare configurazioni errate comuni (e poi rimanda a `openclaw doctor`).
* `channels logs`: mostra i log recenti dei canali dal file di log del Gateway.
* `channels add`: procedura guidata in stile wizard quando non vengono passati flag; i flag abilitano la modalità non interattiva.
* `channels remove`: disabilita per impostazione predefinita; passa `--delete` per rimuovere le voci di configurazione senza richieste di conferma.
* `channels login`: login interattivo del canale (solo WhatsApp Web).
* `channels logout`: disconnette una sessione del canale (se supportato).

Opzioni comuni:

* `--channel <name>`: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams`
* `--account <id>`: ID dell&#39;account del canale (predefinito `default`)
* `--name <label>`: nome visualizzato per l&#39;account

Opzioni di `channels login`:

* `--channel <channel>` (predefinito `whatsapp`; supporta `whatsapp`/`web`)
* `--account <id>`
* `--verbose`

Opzioni di `channels logout`:

* `--channel <channel>` (predefinito `whatsapp`)
* `--account <id>`

Opzioni di `channels list`:

* `--no-usage`: salta gli snapshot di utilizzo/quote del provider di modelli (solo per provider basati su OAuth/API).
* `--json`: output JSON (include l&#39;utilizzo a meno che `--no-usage` non sia impostato).

Opzioni di `channels logs`:

* `--channel <name|all>` (predefinito `all`)
* `--lines <n>` (predefinito `200`)
* `--json`

Ulteriori dettagli: [/concepts/oauth](/it/concepts/oauth)

Esempi:

```bash
openclaw channels add --channel telegram --account alerts --name "Alerts Bot" --token $TELEGRAM_BOT_TOKEN
openclaw channels add --channel discord --account work --name "Work Bot" --token $DISCORD_BOT_TOKEN
openclaw channels remove --channel discord --account work --delete
openclaw channels status --probe
openclaw status --deep
```

<div id="skills">
  ### `skills`
</div>

Elenca e ispeziona le abilità disponibili e le informazioni sullo stato di disponibilità.

Sottocomandi:

* `skills list`: elenca le abilità (predefinito se non è specificato alcun sottocomando).
* `skills info <name>`: mostra i dettagli per una singola abilità.
* `skills check`: riepilogo dei requisiti soddisfatti e mancanti.

Opzioni:

* `--eligible`: mostra solo le abilità pronte.
* `--json`: produce un output JSON (senza formattazione).
* `-v`, `--verbose`: include i dettagli sui requisiti mancanti.

Suggerimento: usa il comando `npx clawhub` per cercare, installare e sincronizzare le abilità.

<div id="pairing">
  ### `pairing`
</div>

Approva le richieste di abbinamento per messaggi diretti (DM) tra i vari canali.

Subcomandi:

* `pairing list <channel> [--json]`
* `pairing approve <channel> <code> [--notify]`

<div id="webhooks-gmail">
  ### `webhooks gmail`
</div>

Configurazione dell&#39;hook Gmail Pub/Sub e runner. Consulta [/automation/gmail-pubsub](/it/automation/gmail-pubsub).

Sottocomandi:

* `webhooks gmail setup` (richiede `--account <email>`; supporta `--project`, `--topic`, `--subscription`, `--label`, `--hook-url`, `--hook-token`, `--push-token`, `--bind`, `--port`, `--path`, `--include-body`, `--max-bytes`, `--renew-minutes`, `--tailscale`, `--tailscale-path`, `--tailscale-target`, `--push-endpoint`, `--json`)
* `webhooks gmail run` (override in fase di esecuzione per gli stessi flag)

<div id="dns-setup">
  ### `dns setup`
</div>

Helper DNS per la discovery wide-area (CoreDNS + Tailscale). Consulta [/gateway/discovery](/it/gateway/discovery).

Opzioni:

* `--apply`: installa/aggiorna la configurazione di CoreDNS (richiede sudo; solo macOS).

<div id="messaging-agent">
  ## Messaggistica + agente
</div>

<div id="message">
  ### `message`
</div>

Messaggistica in uscita unificata e azioni sui canali.

Vedi: [/cli/message](/it/cli/message)

Sottocomandi:

* `message send|poll|react|reactions|read|edit|delete|pin|unpin|pins|permissions|search|timeout|kick|ban`
* `message thread <create|list|reply>`
* `message emoji <list|upload>`
* `message sticker <send|upload>`
* `message role <info|add|remove>`
* `message channel <info|list>`
* `message member info`
* `message voice status`
* `message event <list|create>`

Esempi:

* `openclaw message send --target +15555550123 --message "Hi"`
* `openclaw message poll --channel discord --target channel:123 --poll-question "Snack?" --poll-option Pizza --poll-option Sushi`

<div id="agent">
  ### `agent`
</div>

Esegui un singolo turno di un agente tramite il Gateway (o `--local` in locale/incorporato).

Obbligatorio:

* `--message <text>`

Opzioni:

* `--to <dest>` (per la chiave di sessione e l’eventuale consegna)
* `--session-id <id>`
* `--thinking <off|minimal|low|medium|high|xhigh>` (solo modelli GPT-5.2 + Codex)
* `--verbose <on|full|off>`
* `--channel <whatsapp|telegram|discord|slack|mattermost|signal|imessage|msteams>`
* `--local`
* `--deliver`
* `--json`
* `--timeout <seconds>`

<div id="agents">
  ### `agents`
</div>

Gestisci agenti isolati (spazi di lavoro + autenticazione + instradamento).

<div id="agents-list">
  #### `agents list`
</div>

Elenca gli agenti configurati.

Opzioni:

* `--json`
* `--bindings`

<div id="agents-add-name">
  #### `agents add [name]`
</div>

Aggiunge un nuovo agente isolato. Esegue la procedura guidata, a meno che non vengano passati flag (o `--non-interactive`); `--workspace` è obbligatorio in modalità non interattiva.

Opzioni:

* `--workspace <dir>`
* `--model <id>`
* `--agent-dir <dir>`
* `--bind <channel[:accountId]>` (ripetibile)
* `--non-interactive`
* `--json`

Le specifiche di binding usano `channel[:accountId]`. Quando `accountId` viene omesso per WhatsApp, viene utilizzato l’ID dell’account predefinito.

<div id="agents-delete-id">
  #### `agents delete <id>`
</div>

Elimina un agente e ripulisce il suo spazio di lavoro e lo stato associato.

Opzioni:

* `--force`
* `--json`

<div id="acp">
  ### `acp`
</div>

Avvia il bridge ACP che collega gli IDE al Gateway.

Consulta [`acp`](/it/cli/acp) per l’elenco completo di opzioni ed esempi.

<div id="status">
  ### `status`
</div>

Mostra lo stato di salute delle sessioni collegate e i destinatari recenti.

Opzioni:

* `--json`
* `--all` (diagnostica completa; in sola lettura, incollabile)
* `--deep` (analizza i canali)
* `--usage` (mostra utilizzo/quota del provider del modello)
* `--timeout <ms>`
* `--verbose`
* `--debug` (alias di `--verbose`)

Note:

* La panoramica include, quando disponibile, lo stato del servizio Gateway e del servizio nodo host.

<div id="usage-tracking">
  ### Monitoraggio dell&#39;utilizzo
</div>

OpenClaw può mostrare l&#39;utilizzo/la quota di un provider quando sono disponibili le credenziali OAuth/API.

Dove viene mostrato:

* `/status` (aggiunge una breve riga con l&#39;utilizzo del provider, quando disponibile)
* `openclaw status --usage` (stampa il dettaglio completo per provider)
* barra dei menu di macOS (sezione Usage sotto Context)

Note:

* I dati provengono direttamente dagli endpoint di utilizzo del provider (nessuna stima).
* Provider: Anthropic, GitHub Copilot, OpenAI Codex OAuth, oltre a Gemini CLI/Antigravity quando i relativi plugin del provider sono abilitati.
* Se non esistono credenziali corrispondenti, l&#39;utilizzo viene nascosto.
* Dettagli: vedi [Monitoraggio dell&#39;utilizzo](/it/concepts/usage-tracking).

<div id="health">
  ### `health`
</div>

Verifica lo stato di salute del Gateway in esecuzione.

Opzioni:

* `--json`
* `--timeout <ms>`
* `--verbose`

<div id="sessions">
  ### `sessions`
</div>

Elenca le sessioni di conversazione salvate.

Opzioni:

* `--json`
* `--verbose`
* `--store <path>`
* `--active <minutes>`

<div id="reset-uninstall">
  ## Reimpostazione / Disinstallazione
</div>

<div id="reset">
  ### `reset`
</div>

Reimposta la configurazione e lo stato locali (lascia la CLI installata).

Opzioni:

* `--scope <config|config+creds+sessions|full>`
* `--yes`
* `--non-interactive`
* `--dry-run`

Note:

* `--non-interactive` richiede `--scope` e `--yes`.

<div id="uninstall">
  ### `uninstall`
</div>

Disinstalla il servizio Gateway e i dati locali (la CLI rimane).

Opzioni:

* `--service`
* `--state`
* `--workspace`
* `--app`
* `--all`
* `--yes`
* `--non-interactive`
* `--dry-run`

Note:

* `--non-interactive` richiede `--yes` e scope espliciti (oppure `--all`).

## Gateway

<div id="gateway">
  ### `gateway`
</div>

Esegue il Gateway WebSocket.

Opzioni:

* `--port <port>`
* `--bind <loopback|tailnet|lan|auto|custom>`
* `--token <token>`
* `--auth <token|password>`
* `--password <password>`
* `--tailscale <off|serve|funnel>`
* `--tailscale-reset-on-exit`
* `--allow-unconfigured`
* `--dev`
* `--reset` (reimposta config di sviluppo + credenziali + sessioni + spazio di lavoro)
* `--force` (termina il processo in ascolto esistente sulla porta)
* `--verbose`
* `--claude-cli-logs`
* `--ws-log <auto|full|compact>`
* `--compact` (alias di `--ws-log compact`)
* `--raw-stream`
* `--raw-stream-path <path>`

<div id="gateway-service">
  ### `gateway service`
</div>

Gestisci il servizio Gateway (launchd/systemd/schtasks).

Sottocomandi:

* `gateway status` (interroga l&#39;RPC del Gateway per impostazione predefinita)
* `gateway install` (installazione del servizio)
* `gateway uninstall`
* `gateway start`
* `gateway stop`
* `gateway restart`

Note:

* `gateway status` interroga l&#39;RPC del Gateway per impostazione predefinita usando la porta/configurazione risolta del servizio (puoi eseguire l’override con `--url/--token/--password`).
* `gateway status` supporta `--no-probe`, `--deep` e `--json` per lo scripting.
* `gateway status` mostra anche servizi Gateway legacy o aggiuntivi quando riesce a rilevarli (`--deep` aggiunge scansioni a livello di sistema). I servizi OpenClaw con nome di profilo sono trattati come servizi di prima classe e non vengono contrassegnati come &quot;extra&quot;.
* `gateway status` stampa quale percorso di configurazione usa la CLI rispetto a quale configurazione probabilmente usa il servizio (ambiente del servizio), oltre all&#39;URL di destinazione del controllo risolto.
* `gateway install|uninstall|start|stop|restart` supporta `--json` per lo scripting (l&#39;output predefinito rimane leggibile per gli esseri umani).
* `gateway install` usa come valore predefinito il runtime Node.js; bun **non è raccomandato** (bug con WhatsApp/Telegram).
* Opzioni di `gateway install`: `--port`, `--runtime`, `--token`, `--force`, `--json`.

<div id="logs">
  ### `logs`
</div>

Mostra in tempo reale i file di log del Gateway tramite RPC.

Note:

* Le sessioni TTY mostrano una vista strutturata e a colori; le sessioni non-TTY passano al testo semplice.
* `--json` emette JSON delimitato per riga (un evento di log per riga).

Esempi:

```bash
openclaw logs --follow
openclaw logs --limit 200
openclaw logs --plain
openclaw logs --json
openclaw logs --no-color
```

<div id="gateway-subcommand">
  ### `gateway <subcommand>`
</div>

Helper della CLI del Gateway (usa `--url`, `--token`, `--password`, `--timeout`, `--expect-final` per i sottocomandi RPC).

Sottocomandi:

* `gateway call <method> [--params <json>]`
* `gateway health`
* `gateway status`
* `gateway probe`
* `gateway discover`
* `gateway install|uninstall|start|stop|restart`
* `gateway run`

RPC comuni:

* `config.apply` (valida + scrive la configurazione + riavvia + riattiva)
* `config.patch` (unisce un aggiornamento parziale + riavvia + riattiva)
* `update.run` (esegue l&#39;aggiornamento + riavvia + riattiva)

Suggerimento: quando chiami direttamente `config.set`/`config.apply`/`config.patch`, passa `baseHash` da
`config.get` se esiste già una configurazione.

<div id="models">
  ## Modelli
</div>

Consulta [/concepts/models](/it/concepts/models) per il comportamento di fallback e la strategia di scansione.

Autenticazione Anthropic consigliata (setup-token):

```bash
claude setup-token
openclaw models auth setup-token --provider anthropic
openclaw models status
```

<div id="models-root">
  ### `models` (root)
</div>

`openclaw models` è un alias per `models status`.

Opzioni root:

* `--status-json` (alias per `models status --json`)
* `--status-plain` (alias per `models status --plain`)

<div id="models-list">
  ### `models list`
</div>

Opzioni:

* `--all`
* `--local`
* `--provider <name>`
* `--json`
* `--plain`

<div id="models-status">
  ### `models status`
</div>

Opzioni:

* `--json`
* `--plain`
* `--check` (exit 1=scaduto/mancante, 2=in scadenza)
* `--probe` (verifica in tempo reale dei profili di autenticazione configurati)
* `--probe-provider <name>`
* `--probe-profile <id>` (ripetibile o elenco separato da virgole)
* `--probe-timeout <ms>`
* `--probe-concurrency <n>`
* `--probe-max-tokens <n>`

Include sempre la panoramica dell&#39;autenticazione e lo stato di scadenza OAuth per i profili nello store di autenticazione.
`--probe` esegue richieste in tempo reale (può consumare token e attivare limitazioni di frequenza / rate limit).

<div id="models-set-model">
  ### `models set <model>`
</div>

Imposta `agents.defaults.model.primary`.

<div id="models-set-image-model">
  ### `models set-image <model>`
</div>

Imposta `agents.defaults.imageModel.primary`.

<div id="models-aliases-listaddremove">
  ### `models aliases list|add|remove`
</div>

Opzioni:

* `list`: `--json`, `--plain`
* `add <alias> <model>`
* `remove <alias>`

<div id="models-fallbacks-listaddremoveclear">
  ### `models fallbacks list|add|remove|clear`
</div>

Opzioni:

* `list`: `--json`, `--plain`
* `add <model>`
* `remove <model>`
* `clear`

<div id="models-image-fallbacks-listaddremoveclear">
  ### `models image-fallbacks list|add|remove|clear`
</div>

Opzioni:

* `list`: `--json`, `--plain`
* `add <model>`
* `remove <model>`
* `clear`

<div id="models-scan">
  ### `models scan`
</div>

Opzioni:

* `--min-params <b>`
* `--max-age-days <days>`
* `--provider <name>`
* `--max-candidates <n>`
* `--timeout <ms>`
* `--concurrency <n>`
* `--no-probe`
* `--yes`
* `--no-input`
* `--set-default`
* `--set-image`
* `--json`

<div id="models-auth-addsetup-tokenpaste-token">
  ### `models auth add|setup-token|paste-token`
</div>

Opzioni:

* `add`: assistente interattivo per l&#39;autenticazione
* `setup-token`: `--provider <name>` (predefinito `anthropic`), `--yes`
* `paste-token`: `--provider <name>`, `--profile-id <id>`, `--expires-in <duration>`

<div id="models-auth-order-getsetclear">
  ### `models auth order get|set|clear`
</div>

Opzioni:

* `get`: `--provider <name>`, `--agent <id>`, `--json`
* `set`: `--provider <name>`, `--agent <id>`, `<profileIds...>`
* `clear`: `--provider <name>`, `--agent <id>`

<div id="system">
  ## Sistema
</div>

<div id="system-event">
  ### `system event`
</div>

Accoda un evento di sistema e opzionalmente attiva un heartbeat (RPC del Gateway).

Obbligatorio:

* `--text <text>`

Opzioni:

* `--mode <now|next-heartbeat>`
* `--json`
* `--url`, `--token`, `--timeout`, `--expect-final`

<div id="system-heartbeat-lastenabledisable">
  ### `system heartbeat last|enable|disable`
</div>

Controlli heartbeat (RPC del Gateway).

Opzioni:

* `--json`
* `--url`, `--token`, `--timeout`, `--expect-final`

<div id="system-presence">
  ### `system presence`
</div>

Elenca le voci di presenza del sistema (Gateway RPC).

Opzioni:

* `--json`
* `--url`, `--token`, `--timeout`, `--expect-final`

<div id="cron">
  ## Cron
</div>

Gestisci i job pianificati (Gateway RPC). Consulta [/automation/cron-jobs](/it/automation/cron-jobs).

Sottocomandi:

* `cron status [--json]`
* `cron list [--all] [--json]` (output tabellare per impostazione predefinita; usa `--json` per l’output grezzo)
* `cron add` (alias: `create`; richiede `--name` ed esattamente uno tra `--at` | `--every` | `--cron`, ed esattamente un payload tra `--system-event` | `--message`)
* `cron edit <id>` (modifica i campi)
* `cron rm <id>` (alias: `remove`, `delete`)
* `cron enable <id>`
* `cron disable <id>`
* `cron runs --id <id> [--limit <n>]`
* `cron run <id> [--force]`

Tutti i comandi `cron` accettano `--url`, `--token`, `--timeout`, `--expect-final`.

<div id="node-host">
  ## Host del nodo
</div>

`node` esegue un **nodo host headless** o lo gestisce come servizio in background. Consulta
[`openclaw node`](/it/cli/node).

Sottocomandi:

* `node run --host <gateway-host> --port 18789`
* `node status`
* `node install [--host <gateway-host>] [--port <port>] [--tls] [--tls-fingerprint <sha256>] [--node-id <id>] [--display-name <name>] [--runtime <node|bun>] [--force]`
* `node uninstall`
* `node stop`
* `node restart`

<div id="nodes">
  ## Nodi
</div>

`nodes` comunica con il Gateway e indirizza i comandi ai nodi associati. Consulta [/nodes](/it/nodes).

Opzioni comuni:

* `--url`, `--token`, `--timeout`, `--json`

Sottocomandi:

* `nodes status [--connected] [--last-connected <duration>]`
* `nodes describe --node <id|name|ip>`
* `nodes list [--connected] [--last-connected <duration>]`
* `nodes pending`
* `nodes approve <requestId>`
* `nodes reject <requestId>`
* `nodes rename --node <id|name|ip> --name <displayName>`
* `nodes invoke --node <id|name|ip> --command <command> [--params <json>] [--invoke-timeout <ms>] [--idempotency-key <key>]`
* `nodes run --node <id|name|ip> [--cwd <path>] [--env KEY=VAL] [--command-timeout <ms>] [--needs-screen-recording] [--invoke-timeout <ms>] <command...>` (nodo mac o host di nodo headless)
* `nodes notify --node <id|name|ip> [--title <text>] [--body <text>] [--sound <name>] [--priority <passive|active|timeSensitive>] [--delivery <system|overlay|auto>] [--invoke-timeout <ms>]` (solo macOS)

Fotocamera:

* `nodes camera list --node <id|name|ip>`
* `nodes camera snap --node <id|name|ip> [--facing front|back|both] [--device-id <id>] [--max-width <px>] [--quality <0-1>] [--delay-ms <ms>] [--invoke-timeout <ms>]`
* `nodes camera clip --node <id|name|ip> [--facing front|back] [--device-id <id>] [--duration <ms|10s|1m>] [--no-audio] [--invoke-timeout <ms>]`

Canvas + schermo:

* `nodes canvas snapshot --node <id|name|ip> [--format png|jpg|jpeg] [--max-width <px>] [--quality <0-1>] [--invoke-timeout <ms>]`
* `nodes canvas present --node <id|name|ip> [--target <urlOrPath>] [--x <px>] [--y <px>] [--width <px>] [--height <px>] [--invoke-timeout <ms>]`
* `nodes canvas hide --node <id|name|ip> [--invoke-timeout <ms>]`
* `nodes canvas navigate <url> --node <id|name|ip> [--invoke-timeout <ms>]`
* `nodes canvas eval [<js>] --node <id|name|ip> [--js <code>] [--invoke-timeout <ms>]`
* `nodes canvas a2ui push --node <id|name|ip> (--jsonl <path> | --text <text>) [--invoke-timeout <ms>]`
* `nodes canvas a2ui reset --node <id|name|ip> [--invoke-timeout <ms>]`
* `nodes screen record --node <id|name|ip> [--screen <index>] [--duration <ms|10s>] [--fps <n>] [--no-audio] [--out <path>] [--invoke-timeout <ms>]`

Posizione:

* `nodes location get --node <id|name|ip> [--max-age <ms>] [--accuracy <coarse|balanced|precise>] [--location-timeout <ms>] [--invoke-timeout <ms>]`

<div id="browser">
  ## Browser
</div>

CLI di controllo del browser (Chrome/Brave/Edge/Chromium dedicati). Consulta [`openclaw browser`](/it/cli/browser) e lo [strumento Browser](/it/tools/browser).

Opzioni comuni:

* `--url`, `--token`, `--timeout`, `--json`
* `--browser-profile <name>`

Gestione:

* `browser status`
* `browser start`
* `browser stop`
* `browser reset-profile`
* `browser tabs`
* `browser open <url>`
* `browser focus <targetId>`
* `browser close [targetId]`
* `browser profiles`
* `browser create-profile --name <name> [--color <hex>] [--cdp-url <url>]`
* `browser delete-profile --name <name>`

Ispezione:

* `browser screenshot [targetId] [--full-page] [--ref <ref>] [--element <selector>] [--type png|jpeg]`
* `browser snapshot [--format aria|ai] [--target-id <id>] [--limit <n>] [--interactive] [--compact] [--depth <n>] [--selector <sel>] [--out <path>]`

Azioni:

* `browser navigate <url> [--target-id <id>]`
* `browser resize <width> <height> [--target-id <id>]`
* `browser click <ref> [--double] [--button <left|right|middle>] [--modifiers <csv>] [--target-id <id>]`
* `browser type <ref> <text> [--submit] [--slowly] [--target-id <id>]`
* `browser press <key> [--target-id <id>]`
* `browser hover <ref> [--target-id <id>]`
* `browser drag <startRef> <endRef> [--target-id <id>]`
* `browser select <ref> <values...> [--target-id <id>]`
* `browser upload <paths...> [--ref <ref>] [--input-ref <ref>] [--element <selector>] [--target-id <id>] [--timeout-ms <ms>]`
* `browser fill [--fields <json>] [--fields-file <path>] [--target-id <id>]`
* `browser dialog --accept|--dismiss [--prompt <text>] [--target-id <id>] [--timeout-ms <ms>]`
* `browser wait [--time <ms>] [--text <value>] [--text-gone <value>] [--target-id <id>]`
* `browser evaluate --fn <code> [--ref <ref>] [--target-id <id>]`
* `browser console [--level <error|warn|info>] [--target-id <id>]`
* `browser pdf [--target-id <id>]`

<div id="docs-search">
  ## Ricerca nella documentazione
</div>

<div id="docs-query">
  ### `docs [query...]`
</div>

Cerca nell&#39;indice della documentazione live.

## TUI

<div id="tui">
  ### `tui`
</div>

Apri la UI del terminale collegata al Gateway.

Opzioni:

* `--url <url>`
* `--token <token>`
* `--password <password>`
* `--session <key>`
* `--deliver`
* `--thinking <level>`
* `--message <text>`
* `--timeout-ms <ms>` (predefinito: `agents.defaults.timeoutSeconds`)
* `--history-limit <n>`