---
title: Gateway
summary: "OpenClaw Gateway CLI (`openclaw gateway`) — esegui, interroga e individua i gateway"
read_when:
  - Esecuzione del Gateway dalla CLI (dev o server)
  - Debug dell'autenticazione del Gateway, delle modalità di bind e della connettività
  - Scoperta dei gateway tramite Bonjour (LAN + tailnet)
---

<div id="gateway-cli">
  # Gateway CLI
</div>

Il Gateway è il server WebSocket di OpenClaw (canali, nodi, sessioni, hook).

I sottocomandi in questa pagina sono disponibili sotto `openclaw gateway …`.

Documentazione correlata:

* [/gateway/bonjour](/it/gateway/bonjour)
* [/gateway/discovery](/it/gateway/discovery)
* [/gateway/configuration](/it/gateway/configuration)

<div id="run-the-gateway">
  ## Avvia il Gateway
</div>

Avvia un processo Gateway locale:

```bash
openclaw gateway
```

Alias per esecuzione in primo piano:

```bash
openclaw gateway run
```

Note:

* Per impostazione predefinita, il Gateway si rifiuta di avviarsi a meno che `gateway.mode=local` non sia impostato in `~/.openclaw/openclaw.json`. Usa `--allow-unconfigured` per esecuzioni ad‑hoc/dev.
* Il binding su interfacce diverse dal loopback senza autenticazione è bloccato (misura di sicurezza).
* `SIGUSR1` attiva un riavvio in‑process quando autorizzato (abilita `commands.restart` oppure usa il comando `gateway tool/config apply/update`).
* I gestori dei segnali `SIGINT`/`SIGTERM` arrestano il processo del gateway, ma non ripristinano alcuno stato personalizzato del terminale. Se esegui la CLI all&#39;interno di una TUI o con input in modalità raw, ripristina il terminale prima dell&#39;uscita.

<div id="options">
  ### Opzioni
</div>

* `--port <port>`: porta WS (valore predefinito da configurazione/ambiente; di solito `18789`).
* `--bind <loopback|lan|tailnet|auto|custom>`: modalità di binding del listener.
* `--auth <token|password>`: forza una modalità di autenticazione specifica.
* `--token <token>`: sovrascrive il token (imposta anche `OPENCLAW_GATEWAY_TOKEN` per il processo).
* `--password <password>`: sovrascrive la password (imposta anche `OPENCLAW_GATEWAY_PASSWORD` per il processo).
* `--tailscale <off|serve|funnel>`: espone il Gateway tramite Tailscale.
* `--tailscale-reset-on-exit`: ripristina la configurazione Tailscale serve/funnel all&#39;arresto.
* `--allow-unconfigured`: consente l&#39;avvio del Gateway senza `gateway.mode=local` nella configurazione.
* `--dev`: crea una configurazione di sviluppo + spazio di lavoro se mancanti (salta BOOTSTRAP.md).
* `--reset`: ripristina configurazione di sviluppo + credenziali + sessioni + spazio di lavoro (richiede `--dev`).
* `--force`: termina qualsiasi listener esistente sulla porta selezionata prima di avviare.
* `--verbose`: log dettagliati.
* `--claude-cli-logs`: mostra solo i log di claude-cli nella console (e abilita il relativo stdout/stderr).
* `--ws-log <auto|full|compact>`: stile di log WS (predefinito `auto`).
* `--compact`: alias per `--ws-log compact`.
* `--raw-stream`: registra gli eventi grezzi dello stream del modello in jsonl.
* `--raw-stream-path <path>`: percorso del file jsonl dello stream grezzo.

<div id="query-a-running-gateway">
  ## Eseguire query su un Gateway in esecuzione
</div>

Tutti i comandi di query usano RPC via WebSocket.

Modalità di output:

* Predefinita: leggibile da umani (colorata in TTY).
* `--json`: JSON per uso macchina (nessuno stile/spinner).
* `--no-color` (o `NO_COLOR=1`): disabilita ANSI mantenendo una formattazione leggibile.

Opzioni condivise (se supportate):

* `--url <url>`: URL WebSocket del Gateway.
* `--token <token>`: token del Gateway.
* `--password <password>`: password del Gateway.
* `--timeout <ms>`: timeout/budget (varia in base al comando).
* `--expect-final`: attende una risposta “finale” (chiamate dell’agente).

<div id="gateway-health">
  ### `gateway health`
</div>

```bash
openclaw gateway health --url ws://127.0.0.1:18789
```

<div id="gateway-status">
  ### `gateway status`
</div>

`gateway status` mostra il servizio Gateway (launchd/systemd/schtasks) e un controllo RPC opzionale.

```bash
openclaw gateway status
openclaw gateway status --json
```

Opzioni:

* `--url <url>`: sovrascrive l&#39;URL di probe.
* `--token <token>`: autenticazione tramite token per il probe.
* `--password <password>`: autenticazione tramite password per il probe.
* `--timeout <ms>`: timeout del probe (valore predefinito: `10000`).
* `--no-probe`: salta il probe RPC (vista solo servizi).
* `--deep`: esegue la scansione anche dei servizi a livello di sistema.

<div id="gateway-probe">
  ### `gateway probe`
</div>

`gateway probe` è il comando “diagnostica tutto”. Esegue sempre un controllo su:

* il gateway remoto configurato (se presente) e
* localhost (loopback) **anche se è configurato un remoto**.

Se sono raggiungibili più gateway, li elenca tutti. Sono supportati più gateway quando usi profili/porte isolati (ad esempio un bot di soccorso), ma nella maggior parte delle installazioni è comunque in esecuzione un singolo gateway.

```bash
openclaw gateway probe
openclaw gateway probe --json
```

<div id="remote-over-ssh-mac-app-parity">
  #### Remote over SSH (parità con l&#39;app macOS)
</div>

La modalità “Remote over SSH” dell&#39;app macOS usa un inoltro di porta locale, così il gateway remoto (che potrebbe essere vincolato solo all&#39;interfaccia di loopback) diventa raggiungibile su `ws://127.0.0.1:<port>`.

Equivalente via CLI:

```bash
openclaw gateway probe --ssh user@gateway-host
```

Options:

* `--ssh <target>`: `user@host` oppure `user@host:port` (la porta predefinita è `22`).
* `--ssh-identity <path>`: percorso del file di identità SSH.
* `--ssh-auto`: seleziona il primo host Gateway individuato come destinazione SSH (solo LAN/WAB).

Config (opzionale, usata come valori predefiniti):

* `gateway.remote.sshTarget`
* `gateway.remote.sshIdentity`

<div id="gateway-call-method">
  ### `gateway call <method>`
</div>

Utility RPC di basso livello.

```bash
openclaw gateway call status
openclaw gateway call logs.tail --params '{"sinceMs": 60000}'
```

<div id="manage-the-gateway-service">
  ## Gestisci il servizio Gateway
</div>

```bash
openclaw gateway install
openclaw gateway start
openclaw gateway stop
openclaw gateway restart
openclaw gateway uninstall
```

Note:

* `gateway install` supporta le opzioni `--port`, `--runtime`, `--token`, `--force`, `--json`.
* I comandi di ciclo di vita accettano l&#39;opzione `--json` per l&#39;uso negli script.

<div id="discover-gateways-bonjour">
  ## Individuare i Gateway (Bonjour)
</div>

`gateway discover` rileva i beacon dei Gateway (`_openclaw-gw._tcp`).

* Multicast DNS-SD: `local.`
* Unicast DNS-SD (Wide-Area Bonjour): scegli un dominio (esempio: `openclaw.internal.`) e configura split DNS + un server DNS; vedi [/gateway/bonjour](/it/gateway/bonjour)

Solo i Gateway con la discovery Bonjour abilitata (impostazione predefinita) annunciano il beacon.

I record di discovery Wide-Area includono (TXT):

* `role` (indicazione del ruolo del gateway)
* `transport` (indicazione del trasporto, ad es. `gateway`)
* `gatewayPort` (porta WebSocket, di solito `18789`)
* `sshPort` (porta SSH; valore predefinito `22` se non presente)
* `tailnetDns` (hostname MagicDNS, quando disponibile)
* `gatewayTls` / `gatewayTlsSha256` (TLS abilitato + impronta del certificato)
* `cliPath` (indicazione opzionale per installazioni remote)

<div id="gateway-discover">
  ### `gateway discover`
</div>

```bash
openclaw gateway discover
```

Opzioni:

* `--timeout <ms>`: timeout per comando (browse/resolve); predefinito `2000`.
* `--json`: output leggibile da macchine (disabilita anche formattazione/spinner).

Esempi:

```bash
openclaw gateway discover --timeout 4000
openclaw gateway discover --json | jq '.beacons[].wsUrl'
```
