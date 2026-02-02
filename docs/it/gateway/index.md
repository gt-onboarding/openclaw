---
title: Gateway
summary: "Runbook per il servizio Gateway, il suo ciclo di vita e le operazioni"
read_when:
  - Esecuzione o debug del processo Gateway
---

<div id="gateway-service-runbook">
  # Runbook del servizio Gateway
</div>

Ultimo aggiornamento: 2025-12-09

<div id="what-it-is">
  ## Che cos&#39;è
</div>

* Il processo sempre attivo che detiene l&#39;unica connessione Baileys/Telegram e il piano di controllo/eventi.
* Sostituisce il comando legacy `gateway`. Punto di ingresso della CLI: `openclaw gateway`.
* Rimane in esecuzione finché non viene arrestato; in caso di errori fatali termina con un codice di uscita diverso da zero, così che il supervisor lo riavvii.

<div id="how-to-run-local">
  ## Come eseguire in locale
</div>

```bash
openclaw gateway --port 18789
# per log completi di debug/trace su stdio:
openclaw gateway --port 18789 --verbose
# se la porta è occupata, termina i listener e avvia:
openclaw gateway --force
# ciclo di sviluppo (ricaricamento automatico su modifiche TS):
pnpm gateway:watch
```

* Il riavvio a caldo della configurazione monitora `~/.openclaw/openclaw.json` (o `OPENCLAW_CONFIG_PATH`).
  * Modalità predefinita: `gateway.reload.mode="hybrid"` (applica a caldo le modifiche sicure, riavvia in caso di modifiche critiche).
  * Il riavvio a caldo esegue un riavvio in-process tramite **SIGUSR1** quando necessario.
  * Disabilita con `gateway.reload.mode="off"`.
* Collega il control plane WebSocket a `127.0.0.1:<port>` (predefinito 18789).
* La stessa porta espone anche HTTP (Control UI, hooks, A2UI). Multiplexing su singola porta.
  * OpenAI Chat Completions (HTTP): [`/v1/chat/completions`](/it/gateway/openai-http-api).
  * OpenResponses (HTTP): [`/v1/responses`](/it/gateway/openresponses-http-api).
  * Tools Invoke (HTTP): [`/tools/invoke`](/it/gateway/tools-invoke-http-api).
* Avvia un file server Canvas per impostazione predefinita su `canvasHost.port` (predefinito `18793`), servendo `http://<gateway-host>:18793/__openclaw__/canvas/` da `~/.openclaw/workspace/canvas`. Disabilita con `canvasHost.enabled=false` o `OPENCLAW_SKIP_CANVAS_HOST=1`.
* Scrive i log su stdout; usa launchd/systemd per mantenerlo in esecuzione e ruotare i log.
* Passa `--verbose` per duplicare il logging di debug (handshake, req/res, eventi) dal file di log allo stdio durante la risoluzione dei problemi.
* `--force` usa `lsof` per trovare i listener sulla porta scelta, invia SIGTERM, registra cosa ha terminato, quindi avvia il gateway (fallisce immediatamente se `lsof` non è disponibile).
* Se lo esegui sotto un supervisor (launchd/systemd/modalità processo figlio dell&#39;app mac), uno stop/restart in genere invia **SIGTERM**; build meno recenti possono esporre questo come codice di uscita `pnpm` `ELIFECYCLE` **143** (SIGTERM), che è un arresto normale, non un crash.
* **SIGUSR1** attiva un riavvio in-process quando autorizzato (applicazione/aggiornamento di strumenti/configurazione del gateway, oppure abilita `commands.restart` per i riavvii manuali).
* L&#39;autenticazione del Gateway è richiesta per impostazione predefinita: imposta `gateway.auth.token` (o `OPENCLAW_GATEWAY_TOKEN`) oppure `gateway.auth.password`. I client devono inviare `connect.params.auth.token/password` a meno che non usino l&#39;identità Tailscale Serve.
* Il wizard ora genera un token per impostazione predefinita, anche su loopback.
* Precedenza porte: `--port` &gt; `OPENCLAW_GATEWAY_PORT` &gt; `gateway.port` &gt; predefinito `18789`.

<div id="remote-access">
  ## Accesso remoto
</div>

* Usa preferibilmente Tailscale/VPN; in alternativa, un tunnel SSH:
  ```bash
  ssh -N -L 18789:127.0.0.1:18789 user@host
  ```
* I client si connettono quindi a `ws://127.0.0.1:18789` tramite il tunnel.
* Se è configurato un token, i client devono includerlo in `connect.params.auth.token` anche tramite il tunnel.

<div id="multiple-gateways-same-host">
  ## Multiple gateways (same host)
</div>

Di solito non è necessario: un singolo Gateway può servire più canali di messaggistica e agenti. Usa più Gateway solo per ridondanza o isolamento rigoroso (es: bot di emergenza).

È supportato se isoli stato + configurazione e utilizzi porte univoche. Guida completa: [Multiple gateways](/it/gateway/multiple-gateways).

I nomi dei servizi sono sensibili al profilo:

* macOS: `bot.molt.<profile>` (i servizi legacy `com.openclaw.*` possono esistere ancora)
* Linux: `openclaw-gateway-<profile>.service`
* Windows: `OpenClaw Gateway (<profile>)`

I metadati di installazione sono incorporati nella configurazione del servizio:

* `OPENCLAW_SERVICE_MARKER=openclaw`
* `OPENCLAW_SERVICE_KIND=gateway`
* `OPENCLAW_SERVICE_VERSION=<version>`

Rescue-Bot Pattern: mantieni un secondo Gateway isolato con il proprio profilo, directory di stato, spazio di lavoro e intervallo di porte di base. Guida completa: [Rescue-bot guide](/it/gateway/multiple-gateways#rescue-bot-guide).

<div id="dev-profile-dev">
  ### Profilo di sviluppo (`--dev`)
</div>

Procedura rapida: esegui un&#39;istanza di sviluppo completamente isolata (config/state/workspace) senza modificare la configurazione principale.

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
# poi punta all'istanza di sviluppo:
openclaw --dev status
openclaw --dev health
```

Valori predefiniti (possono essere sovrascritti tramite env/flag/config):

* `OPENCLAW_STATE_DIR=~/.openclaw-dev`
* `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
* `OPENCLAW_GATEWAY_PORT=19001` (Gateway WS + HTTP)
* porta del servizio di controllo browser = `19003` (derivata: `gateway.port+2`, solo loopback)
* `canvasHost.port=19005` (derivata: `gateway.port+4`)
* il valore predefinito di `agents.defaults.workspace` diventa `~/.openclaw/workspace-dev` quando esegui `setup`/`onboard` con `--dev`.

Porte derivate (regole pratiche):

* porta base = `gateway.port` (o `OPENCLAW_GATEWAY_PORT` / `--port`)
* porta del servizio di controllo browser = base + 2 (solo loopback)
* `canvasHost.port = base + 4` (o `OPENCLAW_CANVAS_HOST_PORT` / override tramite config)
* le porte CDP del profilo browser vengono allocate automaticamente da `browser.controlPort + 9 .. + 108` (persistenti per profilo).

Checklist per istanza:

* `gateway.port` univoca
* `OPENCLAW_CONFIG_PATH` univoco
* `OPENCLAW_STATE_DIR` univoco
* `agents.defaults.workspace` univoco
* numeri WhatsApp separati (se usi WA)

Installazione del servizio per profilo:

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

Esempio:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

<div id="protocol-operator-view">
  ## Protocollo (prospettiva operatore)
</div>

* Documentazione completa: [Gateway protocol](/it/gateway/protocol) e [Bridge protocol (legacy)](/it/gateway/bridge-protocol).
* Primo frame obbligatorio dal client: `req {type:"req", id, method:"connect", params:{minProtocol,maxProtocol,client:{id,displayName?,version,platform,deviceFamily?,modelIdentifier?,mode,instanceId?}, caps, auth?, locale?, userAgent? } }`.
* Il Gateway risponde con `res {type:"res", id, ok:true, payload:hello-ok }` (oppure `ok:false` con un errore, quindi chiude).
* Dopo l&#39;handshake:
  * Richieste: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
  * Eventi: `{type:"event", event, payload, seq?, stateVersion?}`
* Voci di presenza strutturate: `{host, ip, version, platform?, deviceFamily?, modelIdentifier?, mode, lastInputSeconds?, ts, reason?, tags?[], instanceId? }` (per i client WS, `instanceId` proviene da `connect.client.instanceId`).
* Le risposte `agent` sono a due fasi: prima l&#39;ack `res` `{runId,status:"accepted"}`, poi una `res` finale `{runId,status:"ok"|"error",summary}` al termine dell&#39;esecuzione; l&#39;output in streaming arriva come `event:"agent"`.

<div id="methods-initial-set">
  ## Metodi (insieme iniziale)
</div>

* `health` — istantanea completa dello stato di salute (stessa struttura di `openclaw health --json`).
* `status` — breve riepilogo.
* `system-presence` — elenco delle presenze correnti.
* `system-event` — pubblica una nota di presenza/sistema (strutturata).
* `send` — invia un messaggio tramite i canali attivi.
* `agent` — esegue un turno di agente (rimanda gli eventi in streaming sulla stessa connessione).
* `node.list` — elenca i nodi abbinati e quelli attualmente connessi (include `caps`, `deviceFamily`, `modelIdentifier`, `paired`, `connected` e i `commands` annunciati).
* `node.describe` — descrive un nodo (capacità + comandi `node.invoke` supportati; funziona per i nodi abbinati e per i nodi non abbinati attualmente connessi).
* `node.invoke` — invoca un comando su un nodo (ad es. `canvas.*`, `camera.*`).
* `node.pair.*` — ciclo di vita dell&#39;abbinamento (`request`, `list`, `approve`, `reject`, `verify`).

Vedi anche: [Presence](/it/concepts/presence) per come viene generata/deduplicata la presenza e perché è importante un `client.instanceId` stabile.

<div id="events">
  ## Eventi
</div>

* `agent` — eventi di tool/output in streaming generati dall&#39;esecuzione dell&#39;agente (marcati con tag di sequenza).
* `presence` — aggiornamenti di presenza (delta con stateVersion) inviati a tutti i client connessi.
* `tick` — keepalive/no-op periodico per confermare che la connessione sia attiva.
* `shutdown` — il Gateway sta terminando; il payload include `reason` e facoltativamente `restartExpectedMs`. I client devono riconnettersi.

<div id="webchat-integration">
  ## Integrazione WebChat
</div>

* WebChat è una UI SwiftUI nativa che comunica direttamente con il WebSocket del Gateway per cronologia, invio, annullamento ed eventi.
* L&#39;uso remoto passa attraverso lo stesso tunnel SSH/Tailscale; se è configurato un token del Gateway, il client lo include durante `connect`.
* L&#39;app macOS si connette tramite un singolo WS (connessione condivisa); ricostruisce lo stato di presenza dallo snapshot iniziale e ascolta gli eventi `presence` per aggiornare la UI.

<div id="typing-and-validation">
  ## Tipizzazione e validazione
</div>

* Il server valida ogni frame in ingresso con AJV rispetto allo schema JSON generato dalle definizioni del protocollo.
* I client (TS/Swift) utilizzano i tipi generati (TS direttamente; Swift tramite il generatore del repository).
* Le definizioni del protocollo sono la fonte di verità; rigenera schemi e modelli con:
  * `pnpm protocol:gen`
  * `pnpm protocol:gen:swift`

<div id="connection-snapshot">
  ## Snapshot di connessione
</div>

* `hello-ok` include uno `snapshot` con `presence`, `health`, `stateVersion` e `uptimeMs` più `policy {maxPayload,maxBufferedBytes,tickIntervalMs}` in modo che i client possano renderizzare immediatamente senza richieste aggiuntive.
* `health`/`system-presence` rimangono disponibili per un aggiornamento manuale, ma non sono necessari al momento della connessione.

<div id="error-codes-reserror-shape">
  ## Codici di errore (struttura di res.error)
</div>

* Gli errori hanno la forma `{ code, message, details?, retryable?, retryAfterMs? }`.
* Codici standard:
  * `NOT_LINKED` — WhatsApp non autenticato.
  * `AGENT_TIMEOUT` — l&#39;agente non ha risposto entro la scadenza configurata.
  * `INVALID_REQUEST` — convalida di schema/parametri non riuscita.
  * `UNAVAILABLE` — il Gateway si sta arrestando oppure una dipendenza non è disponibile.

<div id="keepalive-behavior">
  ## Comportamento di keepalive
</div>

* Gli eventi `tick` (o ping/pong WS) vengono emessi periodicamente in modo che i client sappiano che il Gateway è attivo anche quando non c’è traffico.
* Le conferme di Send/agente restano risposte separate; non usare i tick come conferme dei Send.

<div id="replay-gaps">
  ## Replay / gap
</div>

* Gli eventi non vengono riprodotti. I client rilevano i gap di `seq` e devono eseguire un aggiornamento (`health` + `system-presence`) prima di proseguire. I client WebChat e macOS ora eseguono automaticamente l&#39;aggiornamento in caso di gap.

<div id="supervision-macos-example">
  ## Supervisione (esempio macOS)
</div>

* Usa launchd per mantenere il servizio attivo:
  * Program: percorso a `openclaw`
  * Arguments: `gateway`
  * KeepAlive: true
  * StandardOut/Err: percorsi di file oppure `syslog`
* In caso di errore, launchd riavvia; una misconfigurazione fatale dovrebbe continuare a terminare in errore, così che l&#39;operatore se ne accorga.
* I LaunchAgents sono per singolo utente e richiedono una sessione con un utente connesso; per configurazioni headless usa un LaunchDaemon personalizzato (non fornito).
  * `openclaw gateway install` scrive `~/Library/LaunchAgents/bot.molt.gateway.plist`
    (oppure `bot.molt.<profile>.plist`; i vecchi `com.openclaw.*` vengono rimossi).
  * `openclaw doctor` verifica la configurazione del LaunchAgent e può aggiornarla ai valori predefiniti correnti.

<div id="gateway-service-management-cli">
  ## Gestione del servizio Gateway (CLI)
</div>

Usa la CLI del Gateway per installare, avviare, arrestare, riavviare e verificarne lo stato:

```bash
openclaw gateway status
openclaw gateway install
openclaw gateway stop
openclaw gateway restart
openclaw logs --follow
```

Note:

* `gateway status` sonda l&#39;RPC del Gateway per impostazione predefinita usando la porta/configurazione risolta del servizio (puoi sovrascriverlo con `--url`).
* `gateway status --deep` aggiunge scansioni a livello di sistema (LaunchDaemons/unità di sistema).
* `gateway status --no-probe` omette il probe RPC (utile quando la rete non è disponibile).
* `gateway status --json` è stabile per gli script.
* `gateway status` riporta il **runtime del supervisore** (launchd/systemd in esecuzione) separatamente dalla **raggiungibilità RPC** (connessione WS + status RPC).
* `gateway status` stampa il percorso della config e il target della sonda per evitare confusione tra “localhost vs bind LAN” e discrepanze di profilo.
* `gateway status` include l&#39;ultima riga di errore del gateway quando il servizio sembra in esecuzione ma la porta è chiusa.
* `logs` segue in streaming il file di log del Gateway via RPC (nessun bisogno di `tail`/`grep` manuali).
* Se vengono rilevati altri servizi simili a gateway, la CLI emette un avviso a meno che non siano servizi di profilo OpenClaw.
  Si raccomanda comunque **un gateway per macchina** per la maggior parte delle configurazioni; usa profili/porte isolati per ridondanza o per un bot di soccorso. Vedi [Multiple gateways](/it/gateway/multiple-gateways).
  * Pulizia: `openclaw gateway uninstall` (servizio corrente) e `openclaw doctor` (migrazioni legacy).
* `gateway install` è un no-op quando è già installato; usa `openclaw gateway install --force` per reinstallare (modifiche di profilo/ambiente/percorso).

App macOS integrata:

* OpenClaw.app può includere un relay del gateway basato su Node e installare un LaunchAgent per utente etichettato
  `bot.molt.gateway` (o `bot.molt.<profile>`; le etichette legacy `com.openclaw.*` vengono comunque scaricate correttamente).
* Per arrestarlo correttamente, usa `openclaw gateway stop` (o `launchctl bootout gui/$UID/bot.molt.gateway`).
* Per riavviarlo, usa `openclaw gateway restart` (o `launchctl kickstart -k gui/$UID/bot.molt.gateway`).
  * `launchctl` funziona solo se il LaunchAgent è installato; in caso contrario usa prima `openclaw gateway install`.
  * Sostituisci l&#39;etichetta con `bot.molt.<profile>` quando esegui un profilo con nome.

<div id="supervision-systemd-user-unit">
  ## Supervisione (unità utente systemd)
</div>

OpenClaw installa per impostazione predefinita un **servizio utente systemd** su Linux/WSL2. Consigliamo i servizi utente per macchine a utente singolo (ambiente più semplice, configurazione per utente). Usa un **servizio di sistema** per server multiutente o sempre attivi (non richiede il lingering dell’utente, supervisione condivisa).

`openclaw gateway install` scrive l’unità utente. `openclaw doctor` verifica l’unità e può aggiornarla per allinearla ai valori predefiniti attualmente raccomandati.

Crea `~/.config/systemd/user/openclaw-gateway[-<profile>].service`:

```
[Unit]
Description=OpenClaw Gateway (profile: <profile>, v<version>)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5
Environment=OPENCLAW_GATEWAY_TOKEN=
WorkingDirectory=/home/youruser

[Install]
WantedBy=default.target
```

Abilita il lingering (necessario perché il servizio utente resti attivo dopo il logout/inattività):

```
sudo loginctl enable-linger youruser
```

La procedura di onboarding esegue questa operazione su Linux/WSL2 (potrebbe richiedere sudo; scrive in `/var/lib/systemd/linger`).
Quindi abilita il servizio:

```
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

**Alternativa (servizio di sistema)** - per server sempre attivi o multiutente, puoi
installare un&#39;unità **system** di systemd invece di un&#39;unità utente (non è necessario abilitare il lingering).
Crea `/etc/systemd/system/openclaw-gateway[-<profile>].service` (copia l&#39;unit file sopra,
sostituisci `WantedBy=multi-user.target`, imposta `User=` + `WorkingDirectory=`), quindi:

```
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

<div id="windows-wsl2">
  ## Windows (WSL2)
</div>

Le installazioni Windows devono usare **WSL2** e seguire la sezione relativa a Linux con systemd riportata sopra.

<div id="operational-checks">
  ## Verifiche operative
</div>

* Liveness: apri WS e invia `req:connect` → dovresti ricevere una `res` con `payload.type="hello-ok"` (con snapshot).
* Readiness: chiama `health` → dovresti ricevere `ok: true` e un canale collegato in `linkChannel` (quando applicabile).
* Debug: iscriviti agli eventi `tick` e `presence`; verifica che `status` mostri l&#39;anzianità del collegamento/autenticazione e che le voci di presenza mostrino l&#39;host del Gateway e i client connessi.

<div id="safety-guarantees">
  ## Garanzie di sicurezza
</div>

* Assumi per impostazione predefinita un solo Gateway per host; se esegui più profili, isola porte/stato e indirizza l&#39;istanza corretta.
* Nessun fallback a connessioni Baileys dirette; se il Gateway è inattivo, gli invii falliscono immediatamente.
* I frame iniziali non di tipo `connect` o il JSON malformato vengono rifiutati e il socket viene chiuso.
* Arresto controllato: emette l&#39;evento `shutdown` prima di chiudere; i client devono gestire la chiusura e riconnettersi.

<div id="cli-helpers">
  ## Utility CLI
</div>

* `openclaw gateway health|status` — richiede lo stato/health tramite il WS del Gateway.
* `openclaw message send --target <num> --message "hi" [--media ...]` — invia tramite Gateway (idempotente per WhatsApp).
* `openclaw agent --message "hi" --to <num>` — esegue un turno dell&#39;agente (attende il risultato finale per impostazione predefinita).
* `openclaw gateway call <method> --params '{"k":"v"}'` — invocatore di metodi a basso livello per il debugging.
* `openclaw gateway stop|restart` — arresta/riavvia il servizio Gateway supervisionato (launchd/systemd).
* I sottocomandi di supporto del Gateway presuppongono un Gateway in esecuzione su `--url`; non ne avviano più automaticamente uno.

<div id="migration-guidance">
  ## Indicazioni per la migrazione
</div>

* Abbandona l&#39;uso di `openclaw gateway` e della porta di controllo TCP legacy.
* Aggiorna i client per utilizzare il protocollo WS con messaggio `connect` obbligatorio e presenza strutturata.