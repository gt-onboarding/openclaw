---
title: Risoluzione dei problemi
summary: "Guida rapida alla risoluzione dei problemi per gli errori più comuni di OpenClaw"
read_when:
  - Quando indaghi su problemi o errori in fase di esecuzione
---

<div id="troubleshooting">
  # Risoluzione dei problemi 🔧
</div>

Quando OpenClaw non funziona correttamente, ecco come intervenire.

Inizia con la sezione [Primi 60 secondi](/it/help/faq#first-60-seconds-if-somethings-broken) delle FAQ se vuoi solo una procedura rapida di valutazione. Questa pagina entra più nel dettaglio su errori in fase di esecuzione e diagnostica.

Scorciatoie specifiche per provider: [/channels/troubleshooting](/it/channels/troubleshooting)

<div id="status-diagnostics">
  ## Stato &amp; diagnostica
</div>

Comandi rapidi di triage (in quest&#39;ordine):

| Command | Cosa ti dice | Quando usarlo |
|---|---|---|
| `openclaw status` | Riepilogo locale: OS + aggiornamenti, raggiungibilità/modalità del Gateway, servizio, agenti/sessioni, stato della configurazione dei provider | Primo controllo, panoramica veloce |
| `openclaw status --all` | Diagnosi locale completa (sola lettura, copiabile, relativamente sicura) incl. parte finale dei log | Quando devi condividere un report di debug |
| `openclaw status --deep` | Esegue gli health check del Gateway (incluse sonde verso i provider; richiede Gateway raggiungibile) | Quando “configurato” non significa “funzionante” |
| `openclaw gateway probe` | Scoperta del Gateway + raggiungibilità (target locali + remoti) | Quando sospetti di stare interrogando il Gateway sbagliato |
| `openclaw channels status --probe` | Chiede al Gateway in esecuzione lo stato dei canali (e opzionalmente esegue sonde) | Quando il Gateway è raggiungibile ma i canali si comportano in modo anomalo |
| `openclaw gateway status` | Stato del supervisor (launchd/systemd/schtasks), PID/uscita del runtime, ultimo errore del Gateway | Quando il servizio “sembra avviato” ma in realtà non gira nulla |
| `openclaw logs --follow` | Log in tempo reale (il segnale migliore per problemi di runtime) | Quando ti serve il motivo esatto dell’errore |

**Condivisione dell&#39;output:** preferisci usare `openclaw status --all` (oscura automaticamente i token). Se incolli `openclaw status`, valuta di impostare prima `OPENCLAW_SHOW_SECRETS=0` (disabilita le anteprime dei token).

Vedi anche: [Health checks](/it/gateway/health) e [Logging](/it/logging).

<div id="common-issues">
  ## Problemi comuni
</div>

<div id="no-api-key-found-for-provider-anthropic">
  ### Nessuna chiave API trovata per il provider &quot;anthropic&quot;
</div>

Questo significa che **lo store di autenticazione dell&#39;agente è vuoto** o che mancano le credenziali Anthropic.
L&#39;autenticazione è **per singolo agente**, quindi un nuovo agente non eredita le chiavi dell&#39;agente principale.

Opzioni per risolvere:

* Riesegui l&#39;onboarding e scegli **Anthropic** per quell&#39;agente.
* Oppure incolla un setup-token sull&#39;**host del Gateway**:
  ```bash
  openclaw models auth setup-token --provider anthropic
  ```
* Oppure copia `auth-profiles.json` dalla directory dell&#39;agente principale alla nuova directory dell&#39;agente.

Verifica:

```bash
openclaw models status
```

<div id="oauth-token-refresh-failed-anthropic-claude-subscription">
  ### Aggiornamento del token OAuth non riuscito (abbonamento Anthropic Claude)
</div>

Questo significa che il token OAuth Anthropic memorizzato è scaduto e il tentativo di aggiornamento non è andato a buon fine.
Se utilizzi un abbonamento a Claude (senza chiave API), la soluzione più affidabile è
passare a un **Claude Code setup-token** e incollarlo sull’**host del Gateway**.

**Consigliato (setup-token):**

```bash
# Esegui sull'host del Gateway (incolla il setup-token)
openclaw models auth setup-token --provider anthropic
openclaw models status
```

Se hai generato il token altrove:

```bash
openclaw models auth paste-token --provider anthropic
openclaw models status
```

Per maggiori dettagli: [Anthropic](/it/providers/anthropic) e [OAuth](/it/concepts/oauth).

<div id="control-ui-fails-on-http-device-identity-required-connect-failed">
  ### Control UI non funziona su HTTP (&quot;device identity required&quot; / &quot;connect failed&quot;)
</div>

Se apri la dashboard in chiaro via HTTP (ad es. `http://<lan-ip>:18789/` oppure
`http://<tailscale-ip>:18789/`), il browser viene eseguito in un **contesto non sicuro** e
blocca WebCrypto, quindi l&#39;identità del dispositivo non può essere generata.

**Soluzione:**

* Preferisci HTTPS tramite [Tailscale Serve](/it/gateway/tailscale).
* Oppure apri localmente sull&#39;host del Gateway: `http://127.0.0.1:18789/`.
* Se devi rimanere su HTTP, abilita `gateway.controlUi.allowInsecureAuth: true` e
  usa un gateway token (solo token; nessuna identità del dispositivo/abbinamento). Vedi
  [Control UI](/it/web/control-ui#insecure-http).

<div id="ci-secrets-scan-failed">
  ### Scansione dei secret CI non riuscita
</div>

Questo significa che `detect-secrets` ha trovato nuovi candidati che non sono ancora presenti nella baseline.
Segui la sezione [Secret scanning](/it/gateway/security#secret-scanning-detect-secrets).

<div id="service-installed-but-nothing-is-running">
  ### Servizio installato ma nessun processo in esecuzione
</div>

Se il servizio Gateway è installato ma il processo termina immediatamente, il servizio
può apparire come “caricato” pur non avendo alcun processo in esecuzione.

**Controlla:**

```bash
openclaw gateway status
openclaw doctor
```

Doctor/service mostrerà lo stato di esecuzione (PID/ultimo codice di uscita) e indicazioni dai log.

**Log:**

* Preferito: `openclaw logs --follow`
* Log su file (sempre attivi): `/tmp/openclaw/openclaw-YYYY-MM-DD.log` (oppure il tuo `logging.file` configurato)
* macOS LaunchAgent (se installato): `$OPENCLAW_STATE_DIR/logs/gateway.log` e `gateway.err.log`
* Linux systemd (se installato): `journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`
* Windows: `schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST`

**Abilitare una registrazione più dettagliata:**

* Aumenta il dettaglio dei log su file (JSONL persistente):
  ```json
  { "logging": { "level": "debug" } }
  ```
* Aumenta la verbosità sulla console (solo output TTY):
  ```json
  { "logging": { "consoleLevel": "debug", "consoleStyle": "pretty" } }
  ```
* Suggerimento rapido: `--verbose` influisce solo sull&#39;output sulla **console**. I log su file restano controllati da `logging.level`.

Consulta [/logging](/it/logging) per una panoramica completa di formati, configurazione e modalità di accesso.

<div id="gateway-start-blocked-set-gatewaymodelocal">
  ### &quot;Gateway start blocked: set gateway.mode=local&quot;
</div>

Questo significa che la configurazione esiste ma `gateway.mode` non è impostato (o non è `local`), quindi il Gateway non viene avviato.

**Correzione (consigliata):**

* Esegui la procedura guidata e imposta la modalità di esecuzione del Gateway su **Local**:
  ```bash
  openclaw configure
  ```
* Oppure impostala direttamente:
  ```bash
  openclaw config set gateway.mode local
  ```

**Se invece vuoi eseguire un Gateway remoto:**

* Imposta un URL remoto e mantieni `gateway.mode=remote`:
  ```bash
  openclaw config set gateway.mode remote
  openclaw config set gateway.remote.url "wss://gateway.example.com"
  ```

**Solo per uso ad-hoc/dev:** passa `--allow-unconfigured` per avviare il Gateway senza
`gateway.mode=local`.

**Non hai ancora un file di configurazione?** Esegui `openclaw setup` per creare una configurazione iniziale, quindi riavvia
il Gateway.

<div id="service-environment-path-runtime">
  ### Ambiente del servizio (PATH + runtime)
</div>

Il servizio Gateway viene eseguito con un **PATH minimo** per evitare i residui dei vari shell/manager:

* macOS: `/opt/homebrew/bin`, `/usr/local/bin`, `/usr/bin`, `/bin`
* Linux: `/usr/local/bin`, `/usr/bin`, `/bin`

Questo esclude intenzionalmente i version manager (nvm/fnm/volta/asdf) e i package
manager (pnpm/npm) perché il servizio non carica l&#39;init della tua shell. Variabili
di runtime come `DISPLAY` dovrebbero risiedere in `~/.openclaw/.env` (caricato
in anticipo dal Gateway).
Le esecuzioni con `host=gateway` fondono il `PATH` della tua login shell nell&#39;ambiente
di esecuzione, quindi gli strumenti mancanti di solito significano che l&#39;init della tua shell non li sta esportando (oppure imposta
`tools.exec.pathPrepend`). Vedi [/tools/exec](/it/tools/exec).

I canali WhatsApp + Telegram richiedono **Node**; Bun non è supportato. Se il tuo
servizio è stato installato con Bun o con un percorso Node gestito da un version manager, esegui `openclaw doctor`
per migrare a un&#39;installazione di Node di sistema.

<div id="skill-missing-api-key-in-sandbox">
  ### Skill con chiave API mancante nella sandbox
</div>

**Sintomo:** la skill funziona sull&#39;host ma fallisce nella sandbox con chiave API mancante.

**Perché:** l&#39;esecuzione nella sandbox avviene all&#39;interno di Docker e **non** eredita il `process.env` dell&#39;host.

**Soluzione:**

* imposta `agents.defaults.sandbox.docker.env` (o, per singolo agente, `agents.list[].sandbox.docker.env`)
* oppure incorpora la chiave nella tua immagine sandbox personalizzata
* quindi esegui `openclaw sandbox recreate --agent <id>` (oppure `--all`)

<div id="service-running-but-port-not-listening">
  ### Servizio in esecuzione ma porta non in ascolto
</div>

Se il servizio risulta **in esecuzione** ma niente è in ascolto sulla porta del gateway,
probabilmente il Gateway ha rifiutato il bind.

**Cosa significa &quot;running&quot; qui**

* `Runtime: running` significa che il supervisore (launchd/systemd/schtasks) ritiene che il processo sia vivo.
* `RPC probe` significa che la CLI è effettivamente riuscita a connettersi al WebSocket del gateway e a chiamare `status`.
* Fai sempre affidamento su `Probe target:` + `Config (service):` come le righe che indicano “che cosa abbiamo effettivamente provato?”.

**Verifica:**

* `gateway.mode` deve essere `local` per `openclaw gateway` e per il servizio.
* Se imposti `gateway.mode=remote`, la **CLI per impostazione predefinita** usa un URL remoto. Il servizio può comunque essere in esecuzione in locale, ma la tua CLI potrebbe eseguire il probe nel posto sbagliato. Usa `openclaw gateway status` per vedere la porta risolta dal servizio e il relativo target del probe (oppure passa `--url`).
* `openclaw gateway status` e `openclaw doctor` espongono **l’ultimo errore del gateway** dai log quando il servizio sembra in esecuzione ma la porta è chiusa.
* I bind non-loopback (`lan`/`tailnet`/`custom`, oppure `auto` quando il loopback non è disponibile) richiedono autenticazione:
  `gateway.auth.token` (oppure `OPENCLAW_GATEWAY_TOKEN`).
* `gateway.remote.token` serve solo per le chiamate CLI remote; **non** abilita l’autenticazione locale.
* `gateway.token` viene ignorato; usa `gateway.auth.token`.

**Se `openclaw gateway status` mostra una discrepanza di configurazione**

* `Config (cli): ...` e `Config (service): ...` normalmente dovrebbero coincidere.
* Se non coincidono, quasi certamente stai modificando una configurazione mentre il servizio ne usa un’altra.
* Soluzione: esegui di nuovo `openclaw gateway install --force` dallo stesso `--profile` / `OPENCLAW_STATE_DIR` che vuoi che il servizio utilizzi.

**Se `openclaw gateway status` segnala problemi di configurazione del servizio**

* La configurazione del supervisore (launchd/systemd/schtasks) non include le impostazioni predefinite correnti.
* Soluzione: esegui `openclaw doctor` per aggiornarla (oppure `openclaw gateway install --force` per una riscrittura completa).

**Se `Last gateway error:` menziona “refusing to bind … without auth”**

* Hai impostato `gateway.bind` su una modalità non-loopback (`lan`/`tailnet`/`custom`, oppure `auto` quando il loopback non è disponibile) ma non hai configurato l’autenticazione.
* Soluzione: imposta `gateway.auth.mode` + `gateway.auth.token` (oppure esporta `OPENCLAW_GATEWAY_TOKEN`) e riavvia il servizio.

**Se `openclaw gateway status` indica `bind=tailnet` ma non è stata trovata alcuna interfaccia tailnet**

* Il gateway ha provato a eseguire il bind a un IP Tailscale (100.64.0.0/10) ma non ne è stato rilevato nessuno sull’host.
* Soluzione: attiva Tailscale su quella macchina (oppure cambia `gateway.bind` in `loopback`/`lan`).

**Se `Probe note:` indica che il probe usa loopback**

* È previsto per `bind=lan`: il gateway è in ascolto su `0.0.0.0` (tutte le interfacce), e il loopback dovrebbe comunque connettersi in locale.
* Per i client remoti, usa un vero IP LAN (non `0.0.0.0`) più la porta e assicurati che l’autenticazione sia configurata.

<div id="address-already-in-use-port-18789">
  ### Indirizzo già in uso (porta 18789)
</div>

Questo significa che un altro processo è già in ascolto sulla porta del Gateway.

**Verifica:**

```bash
openclaw gateway status
```

Mostrerà i listener e le cause più probabili (Gateway già in esecuzione, tunnel SSH).
Se necessario, termina il servizio oppure usa una porta diversa.

<div id="extra-workspace-folders-detected">
  ### Cartelle aggiuntive dello spazio di lavoro rilevate
</div>

Se hai effettuato l&#39;upgrade da installazioni precedenti, potresti avere ancora `~/openclaw` sul disco.
Più directory di spazio di lavoro possono causare problemi di autenticazione o disallineamenti dello stato,
perché è attivo un solo spazio di lavoro alla volta.

**Soluzione:** mantieni un solo spazio di lavoro attivo e archivia/rimuovi gli altri. Vedi
[Spazio di lavoro dell&#39;Agente](/it/concepts/agent-workspace#extra-workspace-folders).

<div id="main-chat-running-in-a-sandbox-workspace">
  ### Chat principale in esecuzione in uno spazio di lavoro sandbox
</div>

Sintomi: `pwd` o gli strumenti per i file mostrano `~/.openclaw/sandboxes/...` anche se
ti aspettavi lo spazio di lavoro dell&#39;host.

**Perché:** `agents.defaults.sandbox.mode: "non-main"` si basa su `session.mainKey` (predefinito `"main"`).
Le sessioni di gruppo/canale usano le proprie chiavi, quindi vengono trattate come non-main e
utilizzano spazi di lavoro sandbox.

**Possibili soluzioni:**

* Se vuoi spazi di lavoro dell&#39;host per un agente: imposta `agents.list[].sandbox.mode: "off"`.
* Se vuoi accesso allo spazio di lavoro dell&#39;host all&#39;interno della sandbox: imposta `workspaceAccess: "rw"` per quell&#39;agente.

<div id="agent-was-aborted">
  ### &quot;Agente interrotto&quot;
</div>

L&#39;agente è stato interrotto durante la risposta.

**Cause:**

* L&#39;utente ha inviato `stop`, `abort`, `esc`, `wait` o `exit`
* Timeout superato
* Processo arrestato in modo anomalo

**Soluzione:** Invia semplicemente un altro messaggio. La sessione prosegue.

<div id="agent-failed-before-reply-unknown-model-anthropicclaude-haiku-3-5">
  ### &quot;Agente interrotto prima della risposta: modello sconosciuto: anthropic/claude-haiku-3-5&quot;
</div>

OpenClaw rifiuta intenzionalmente i **modelli più vecchi/insicuri** (soprattutto
quelli più vulnerabili ad attacchi di prompt injection). Se vedi questo errore, il nome
del modello non è più supportato.

**Soluzione:**

* Scegli un modello **più recente** per il provider e aggiorna la tua configurazione o l&#39;alias del modello.
* Se non sei sicuro di quali modelli siano disponibili, esegui `openclaw models list` o
  `openclaw models scan` e scegline uno supportato.
* Controlla i log del Gateway per il motivo dettagliato dell&#39;errore.

Vedi anche: [Models CLI](/it/cli/models) e [Model providers](/it/concepts/model-providers).

<div id="messages-not-triggering">
  ### Messaggi che non attivano l&#39;agente
</div>

**Verifica 1:** Il mittente è presente nella lista di autorizzati?

```bash
openclaw status
```

Cerca `AllowFrom: ...` nell&#39;output.

**Verifica 2:** Nelle chat di gruppo è necessario essere menzionati?

```bash
# Il messaggio deve corrispondere a mentionPatterns o a menzioni esplicite; i valori predefiniti si trovano nei gruppi/guild dei canali.
# Multi-agente: `agents.list[].groupChat.mentionPatterns` sovrascrive i pattern globali.
grep -n "agents\\|groupChat\\|mentionPatterns\\|channels\\.whatsapp\\.groups\\|channels\\.telegram\\.groups\\|channels\\.imessage\\.groups\\|channels\\.discord\\.guilds" \
  "${OPENCLAW_CONFIG_PATH:-$HOME/.openclaw/openclaw.json}"
```

**Verifica 3:** controlla i log

```bash
openclaw logs --follow
# oppure, se vuoi filtri rapidi:
tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)" | grep "blocked\\|skip\\|unauthorized"
```

<div id="pairing-code-not-arriving">
  ### Codice di abbinamento non ricevuto
</div>

Se `dmPolicy` è `pairing`, i mittenti sconosciuti dovrebbero ricevere un codice e il loro messaggio viene ignorato finché non viene approvato.

**Verifica 1:** C&#39;è già una richiesta in attesa?

```bash
openclaw pairing list <channel>
```

Le richieste di abbinamento DM in sospeso sono limitate, per impostazione predefinita, a **3 per canale**. Se l&#39;elenco è pieno, le nuove richieste non genereranno un codice finché una di esse non viene approvata o non scade.

**Controllo 2:** La richiesta è stata creata ma non è stata inviata alcuna risposta?

```bash
openclaw logs --follow | grep "pairing request"
```

**Verifica 3:** Controlla che `dmPolicy` non sia impostato su `open` (accetta messaggi da chiunque) o `allowlist` (lista di autorizzati) per quel canale.

<div id="image-mention-not-working">
  ### Immagine + menzione non funziona
</div>

Problema noto: quando invii un&#39;immagine con SOLO una menzione (senza altro testo), WhatsApp a volte non include i metadati della menzione.

**Soluzione alternativa:** aggiungi del testo insieme alla menzione:

* ❌ `@openclaw` + immagine
* ✅ `@openclaw check this` + immagine

<div id="session-not-resuming">
  ### La sessione non viene ripristinata
</div>

**Verifica 1:** Il file di sessione è presente?

```bash
ls -la ~/.openclaw/agents/<agentId>/sessions/
```

**Verifica 2:** L&#39;intervallo di reset è troppo breve?

```json
{
  "session": {
    "reset": {
      "mode": "daily",
      "atHour": 4,
      "idleMinutes": 10080  // 7 giorni
    }
  }
}
```

**Verifica 3:** Qualcuno ha inviato `/new`, `/reset` o ha attivato un reset?

<div id="agent-timing-out">
  ### Timeout dell&#39;Agente
</div>

Il timeout predefinito è di 30 minuti. Per attività di lunga durata:

```json
{
  "reply": {
    "timeoutSeconds": 3600  // 1 ora
  }
}
```

Oppure usa il tool `process` per eseguire in background i comandi lunghi.

<div id="whatsapp-disconnected">
  ### WhatsApp disconnesso
</div>

```bash
# Controlla lo stato locale (credenziali, sessioni, eventi in coda)
openclaw status
# Verifica il gateway in esecuzione + canali (connessione WA + API Telegram + Discord)
openclaw status --deep

# Visualizza gli eventi di connessione recenti
openclaw logs --limit 200 | grep "connection\\|disconnect\\|logout"
```

**Soluzione:** Di solito si riconnette automaticamente una volta che il Gateway è in esecuzione. Se resta bloccato, riavvia il processo del Gateway (a seconda di come lo gestisci) oppure eseguilo manualmente con output dettagliato:

```bash
openclaw gateway --verbose
```

Se sei disconnesso o l&#39;account non è collegato:

```bash
openclaw channels logout
trash "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/credentials" # se logout non riesce a rimuovere tutto completamente
openclaw channels login --verbose       # re-scan QR
```

<div id="media-send-failing">
  ### Invio di contenuti multimediali non riuscito
</div>

**Verifica 1:** Il percorso del file è valido?

```bash
ls -la /path/to/your/image.jpg
```

**Verifica 2:** È troppo grande?

* Immagini: massimo 6 MB
* Audio/Video: massimo 16 MB
* Documenti: massimo 100 MB

**Verifica 3:** Controlla i log dei contenuti multimediali

```bash
grep "media\\|fetch\\|download" "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)" | tail -20
```

<div id="high-memory-usage">
  ### Utilizzo elevato della memoria
</div>

OpenClaw mantiene la cronologia delle conversazioni in memoria.

**Soluzione:** Riavvia OpenClaw periodicamente oppure imposta limiti per le sessioni:

```json
{
  "session": {
    "historyLimit": 100  // Numero massimo di messaggi da mantenere
  }
}
```

<div id="common-troubleshooting">
  ## Procedure comuni di risoluzione dei problemi
</div>

<div id="gateway-wont-start-configuration-invalid">
  ### “Gateway non si avvia — configurazione non valida”
</div>

OpenClaw ora rifiuta di avviarsi quando la configurazione contiene chiavi sconosciute, valori malformati o tipi non validi.
È una scelta intenzionale per motivi di sicurezza.

Risolvi il problema con Doctor:

```bash
openclaw doctor
openclaw doctor --fix
```

Note:

* `openclaw doctor` segnala ogni voce non valida.
* `openclaw doctor --fix` applica migrazioni/riparazioni e riscrive la configurazione.
* I comandi diagnostici come `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw gateway status` e `openclaw gateway probe` vengono comunque eseguiti anche se la configurazione non è valida.

<div id="all-models-failed-what-should-i-check-first">
  ### &quot;All models failed&quot; — cosa devo controllare per prima?
</div>

* **Credenziali** configurate per i provider utilizzati (profili di autenticazione + variabili d&#39;ambiente).
* **Instradamento del modello**: verifica che `agents.defaults.model.primary` e i fallback siano modelli a cui hai accesso.
* **Log del Gateway** in `/tmp/openclaw/…` per l&#39;errore esatto del provider.
* **Stato del modello**: usa `/model status` (chat) oppure `openclaw models status` (CLI).

<div id="im-running-on-my-personal-whatsapp-number-why-is-self-chat-weird">
  ### Sto usando il mio numero WhatsApp personale — perché la chat con me stesso si comporta in modo strano?
</div>

Abilita la modalità di chat con te stesso e aggiungi il tuo numero alla lista di autorizzati:

```json5
{
  channels: {
    whatsapp: {
      selfChatMode: true,
      dmPolicy: "allowlist",
      allowFrom: ["+15555550123"]
    }
  }
}
```

Consulta la sezione [Configurazione di WhatsApp](/it/channels/whatsapp).

<div id="whatsapp-logged-me-out-how-do-i-reauth">
  ### WhatsApp mi ha disconnesso. Come faccio a effettuare di nuovo l’accesso?
</div>

Esegui di nuovo il comando di login e scansiona il codice QR:

```bash
openclaw channels login
```

<div id="build-errors-on-main-whats-the-standard-fix-path">
  ### Errori di build su `main` — qual è la procedura standard per risolverli?
</div>

1. `git pull origin main && pnpm install`
2. `openclaw doctor`
3. Controlla le issue su GitHub o su Discord
4. Soluzione temporanea: esegui il checkout di un commit precedente

<div id="npm-install-fails-allow-build-scripts-missing-tar-or-yargs-what-now">
  ### npm install non funziona (allow-build-scripts / tar o yargs mancanti). Che fare?
</div>

Se stai eseguendo il progetto dal sorgente, usa il package manager del repo: **pnpm** (consigliato).
Il repo dichiara `packageManager: "pnpm@…"`.

Procedura di recupero tipica:

```bash
git status   # assicurati di trovarti nella root del repository
pnpm install
pnpm build
openclaw doctor
openclaw gateway restart
```

Perché: pnpm è il gestore di pacchetti configurato per questo repository.

<div id="how-do-i-switch-between-git-installs-and-npm-installs">
  ### Come faccio a passare dalle installazioni tramite git a quelle tramite npm?
</div>

Usa il **programma di installazione dal sito web** e seleziona il metodo di installazione con l’apposito flag. Esegue un aggiornamento in-place e riscrive il servizio Gateway per puntare alla nuova installazione.

Passa **a un’installazione tramite git**:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git --no-onboard
```

Passa **all&#39;installazione globale con npm**:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Note:

* Il flusso Git esegue il rebase solo se la repository è pulita. Esegui prima un commit o effettua uno stash delle modifiche.
* Dopo lo switch, esegui:
  ```bash
  openclaw doctor
  openclaw gateway restart
  ```

<div id="telegram-block-streaming-isnt-splitting-text-between-tool-calls-why">
  ### Lo streaming a blocchi di Telegram non suddivide il testo tra le chiamate agli strumenti. Perché?
</div>

Lo streaming a blocchi invia solo **blocchi di testo completi**. Motivi comuni per cui ricevi un solo messaggio:

* `agents.defaults.blockStreamingDefault` è ancora `"off"`.
* `channels.telegram.blockStreaming` è impostato su `false`.
* `channels.telegram.streamMode` è `partial` o `block` **e lo streaming delle bozze è attivo**
  (chat privata + topic). In quel caso, lo streaming delle bozze disabilita lo streaming a blocchi.
* Le impostazioni `minChars` / coalesce sono troppo alte, quindi i chunk vengono uniti.
* Il modello emette un unico grande blocco di testo (nessun punto di flush intermedio nella risposta).

Lista di controllo per la risoluzione:

1. Metti le impostazioni di streaming a blocchi sotto `agents.defaults`, non nel root.
2. Imposta `channels.telegram.streamMode: "off"` se vuoi risposte a blocchi con veri messaggi multipli.
3. Usa soglie chunk/coalesce più basse durante il debugging.

Vedi [Streaming](/it/concepts/streaming).

<div id="discord-doesnt-reply-in-my-server-even-with-requiremention-false-why">
  ### Discord non risponde nel mio server anche con `requireMention: false`. Perché?
</div>

`requireMention` controlla solo la necessità di menzione **dopo** che il canale ha superato la lista di autorizzati.
Per impostazione predefinita, `channels.discord.groupPolicy` è **allowlist**, quindi i server (guild) devono essere abilitati esplicitamente.
Se imposti `channels.discord.guilds.<guildId>.channels`, vengono consentiti solo i canali elencati; omettilo per consentire tutti i canali nel server.

Checklist di correzione:

1. Imposta `channels.discord.groupPolicy: "open"` **oppure** aggiungi una voce nella lista di autorizzati per il server (e facoltativamente una lista di autorizzati per i canali).
2. Usa **ID numerici dei canali** in `channels.discord.guilds.<guildId>.channels`.
3. Metti `requireMention: false` **sotto** `channels.discord.guilds` (a livello globale o per singolo canale).
   La chiave di primo livello `channels.discord.requireMention` non è supportata.
4. Assicurati che il bot abbia il **Message Content Intent** e i permessi sul canale.
5. Esegui `openclaw channels status --probe` per ottenere indicazioni utili alla verifica.

Documentazione: [Discord](/it/channels/discord), [Risoluzione problemi dei canali](/it/channels/troubleshooting).

<div id="cloud-code-assist-api-error-invalid-tool-schema-400-what-now">
  ### Errore Cloud Code Assist API: invalid tool schema (400). E adesso?
</div>

Quasi sempre si tratta di un problema di **compatibilità dello schema degli strumenti**. L&#39;endpoint Cloud Code Assist accetta un sottoinsieme ristretto di JSON Schema. OpenClaw ripulisce/normalizza gli schemi degli strumenti nell&#39;attuale `main`, ma la correzione non è ancora presente nell&#39;ultima release (al 13 gennaio 2026).

Checklist per la correzione:

1. **Aggiorna OpenClaw**:
   * Se puoi eseguirlo dal sorgente, fai il pull di `main` e riavvia il Gateway.
   * In caso contrario, attendi la prossima release che includa lo schema scrubber.
2. Evita parole chiave non supportate come `anyOf/oneOf/allOf`, `patternProperties`,
   `additionalProperties`, `minLength`, `maxLength`, `format`, ecc.
3. Se definisci strumenti personalizzati, mantieni lo schema di primo livello come `type: "object"` con
   `properties` ed enum semplici.

Consulta [Tools](/it/tools) e [TypeBox schemas](/it/concepts/typebox).

<div id="macos-specific-issues">
  ## Problemi specifici su macOS
</div>

<div id="app-crashes-when-granting-permissions-speechmic">
  ### L&#39;app va in crash quando concedi le autorizzazioni (voce/microfono)
</div>

Se l&#39;app scompare o mostra &quot;Abort trap 6&quot; quando fai clic su &quot;Allow&quot; in un avviso sulla privacy:

**Soluzione 1: reimposta la cache TCC**

```bash
tccutil reset All bot.molt.mac.debug
```

**Soluzione 2: Forzare un nuovo Bundle ID**
Se il ripristino non funziona, modifica il `BUNDLE_ID` in [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) (ad esempio, aggiungi il suffisso `.test`) e ricompila. In questo modo obblighi macOS a considerarla come una nuova app.

<div id="gateway-stuck-on-starting">
  ### Gateway bloccato su &quot;Starting...&quot;
</div>

L&#39;app si connette a un Gateway locale sulla porta `18789`. Se rimane bloccata:

**Soluzione 1: arresta il supervisore (consigliato)**
Se il Gateway è gestito da launchd, limitarsi a terminare il PID lo farà semplicemente riavviare. Arresta prima il supervisore:

```bash
openclaw gateway status
openclaw gateway stop
# Oppure: launchctl bootout gui/$UID/bot.molt.gateway (sostituire con bot.molt.<profile>; il legacy com.openclaw.* funziona ancora)
```

**Correzione 2: porta occupata (trova il processo in ascolto)**

```bash
lsof -nP -iTCP:18789 -sTCP:LISTEN
```

Se si tratta di un processo non supervisionato, prova prima un arresto pulito, poi passa a metodi più drastici:

```bash
kill -TERM <PID>
sleep 1
kill -9 <PID> # ultima risorsa
```

**Correzione 3: Verifica l&#39;installazione della CLI**
Assicurati che la CLI globale `openclaw` sia installata e che la sua versione corrisponda a quella dell&#39;app:

```bash
openclaw --version
npm install -g openclaw@<version>
```

<div id="debug-mode">
  ## Modalità di debug
</div>

Abilita il logging dettagliato:

```bash
# Attiva il logging di trace nella configurazione:
#   ${OPENCLAW_CONFIG_PATH:-$HOME/.openclaw/openclaw.json} -> { logging: { level: "trace" } }
#
# Quindi esegui i comandi in modalità verbose per replicare l'output di debug su stdout:
openclaw gateway --verbose
openclaw channels login --verbose
```

<div id="log-locations">
  ## Posizioni dei log
</div>

| Log | Posizione |
|-----|----------|
| Log su file del Gateway (strutturati) | `/tmp/openclaw/openclaw-YYYY-MM-DD.log` (oppure `logging.file`) |
| Log del servizio del Gateway (supervisor) | macOS: `$OPENCLAW_STATE_DIR/logs/gateway.log` + `gateway.err.log` (predefinito: `~/.openclaw/logs/...`; i profili usano `~/.openclaw-<profile>/logs/...`)<br />Linux: `journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`<br />Windows: `schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST` |
| File di sessione | `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/` |
| Cache multimediale | `$OPENCLAW_STATE_DIR/media/` |
| Credenziali | `$OPENCLAW_STATE_DIR/credentials/` |

<div id="health-check">
  ## Verifica dello stato
</div>

```bash
# Supervisor + probe target + config paths
openclaw gateway status
# Include scansioni a livello di sistema (servizi legacy/extra, listener di porta)
openclaw gateway status --deep

# Is the gateway reachable?
openclaw health --json
# If it fails, rerun with connection details:
openclaw health --verbose

# Is something listening on the default port?
lsof -nP -iTCP:18789 -sTCP:LISTEN

# Recent activity (RPC log tail)
openclaw logs --follow
# Fallback if RPC is down
tail -20 /tmp/openclaw/openclaw-*.log
```

<div id="reset-everything">
  ## Reimposta tutto
</div>

L&#39;opzione nucleare:

```bash
openclaw gateway stop
# Se hai installato un servizio e desideri un'installazione pulita:
# openclaw gateway uninstall

trash "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
openclaw channels login         # ri-associa WhatsApp
openclaw gateway restart           # oppure: openclaw gateway
```

⚠️ Questo comporta la perdita di tutte le sessioni e richiede di ripetere l’abbinamento di WhatsApp.

<div id="getting-help">
  ## Come ottenere assistenza
</div>

1. Controlla prima i log: `/tmp/openclaw/` (per impostazione predefinita: `openclaw-YYYY-MM-DD.log`, oppure il `logging.file` che hai configurato)
2. Cerca le issue esistenti su GitHub
3. Apri una nuova issue con:
   * Versione di OpenClaw
   * Estratti di log rilevanti
   * Passaggi per riprodurre il problema
   * La tua configurazione (occulta i segreti!)

***

*&quot;Hai provato a spegnerlo e riaccenderlo?&quot;* — Ogni tecnico IT, sempre

🦞🔧

<div id="browser-not-starting-linux">
  ### Il browser non si avvia (Linux)
</div>

Se vedi `"Failed to start Chrome CDP on port 18800"`:

**Causa più probabile:** versione Snap di Chromium su Ubuntu.

**Soluzione rapida:** installa invece Google Chrome:

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
```

Poi imposta in config:

```json
{
  "browser": {
    "executablePath": "/usr/bin/google-chrome-stable"
  }
}
```

**Guida completa:** consulta [browser-linux-troubleshooting](/it/tools/browser-linux-troubleshooting)
