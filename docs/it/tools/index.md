---
title: Strumenti
summary: "Set di strumenti dell'Agente in OpenClaw (browser, canvas, nodi, messaggi, cron) che sostituisce le abilità `openclaw-*` legacy"
read_when:
  - Aggiungere o modificare strumenti dell'agente
  - Disattivare o modificare le abilità `openclaw-*`
---

<div id="tools-openclaw">
  # Strumenti (OpenClaw)
</div>

OpenClaw espone **strumenti di primo livello per gli agenti** per browser, canvas, nodi e cron.
Questi sostituiscono le vecchie abilità `openclaw-*`: gli strumenti sono tipizzati, non richiedono l&#39;uso della shell e l&#39;agente dovrebbe usarli direttamente.

<div id="disabling-tools">
  ## Disabilitare gli strumenti
</div>

Puoi abilitare o disabilitare globalmente gli strumenti tramite `tools.allow` / `tools.deny` in `openclaw.json`
(con priorità a `deny`). Questo impedisce che gli strumenti non consentiti vengano inviati ai provider di modelli.

```json5
{
  tools: { deny: ["browser"] }
}
```

Note:

* La corrispondenza non distingue tra maiuscole e minuscole.
* I caratteri jolly `*` sono supportati (`"*"` indica tutti gli strumenti).
* Se `tools.allow` fa riferimento solo a nomi di strumenti di plugin sconosciuti o non caricati, OpenClaw registra un avviso nei log e ignora la lista di autorizzati, così gli strumenti core restano disponibili.

<div id="tool-profiles-base-allowlist">
  ## Profili degli strumenti (lista di autorizzati di base)
</div>

`tools.profile` imposta una **lista di base degli strumenti autorizzati** prima di `tools.allow`/`tools.deny`.
Override per agente: `agents.list[].tools.profile`.

Profili:

* `minimal`: solo `session_status`
* `coding`: `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
* `messaging`: `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
* `full`: nessuna restrizione (equivalente a non impostarlo)

Esempio (solo messaggistica per impostazione predefinita, consente anche gli strumenti Slack + Discord):

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"]
  }
}
```

Esempio (profilo di sviluppo, con exec/process negati ovunque):

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"]
  }
}
```

Esempio (profilo di codifica globale, agente di supporto solo via messaggistica):

```json5
{
  tools: { profile: "coding" },
  agents: {
    list: [
      {
        id: "support",
        tools: { profile: "messaging", allow: ["slack"] }
      }
    ]
  }
}
```

<div id="provider-specific-tool-policy">
  ## Criteri di utilizzo degli strumenti specifici per provider
</div>

Usa `tools.byProvider` per **limitare ulteriormente** gli strumenti per provider specifici
(o un singolo `provider/model`) senza modificare i tuoi default globali.
Override per agente: `agents.list[].tools.byProvider`.

Questo viene applicato **dopo** il profilo di base degli strumenti e **prima** delle allow/deny list,
quindi può solo restringere il set di strumenti.
Le chiavi dei provider accettano sia `provider` (ad es. `google-antigravity`) sia
`provider/model` (ad es. `openai/gpt-5.2`).

Esempio (mantieni il profilo globale di coding, ma strumenti minimi per Google Antigravity):

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" }
    }
  }
}
```

Esempio (lista di autorizzati specifica per provider/modello per un endpoint inaffidabile):

```json5
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] }
    }
  }
}
```

Esempio (override specifico dell&#39;agente per un singolo provider):

```json5
{
  agents: {
    list: [
      {
        id: "support",
        tools: {
          byProvider: {
            "google-antigravity": { allow: ["message", "sessions_list"] }
          }
        }
      }
    ]
  }
}
```

<div id="tool-groups-shorthands">
  ## Gruppi di strumenti (abbreviazioni)
</div>

Le policy degli strumenti (globali, per agente, sandbox) supportano voci `group:*` che si espandono in più strumenti.
Usale in `tools.allow` / `tools.deny`.

Gruppi disponibili:

* `group:runtime`: `exec`, `bash`, `process`
* `group:fs`: `read`, `write`, `edit`, `apply_patch`
* `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory`: `memory_search`, `memory_get`
* `group:web`: `web_search`, `web_fetch`
* `group:ui`: `browser`, `canvas`
* `group:automation`: `cron`, `gateway`
* `group:messaging`: `message`
* `group:nodes`: `nodes`
* `group:openclaw`: tutti gli strumenti OpenClaw integrati (esclude i plugin dei provider)

Esempio (consenti solo strumenti per file + browser):

```json5
{
  tools: {
    allow: ["group:fs", "browser"]
  }
}
```

<div id="plugins-tools">
  ## Plugin + strumenti
</div>

I plugin possono registrare **strumenti aggiuntivi** (e comandi CLI) oltre al set di base.
Consulta [Plugin](/it/plugin) per installazione + configurazione e [Abilità](/it/tools/skills) per il modo in cui
le linee guida sull&#39;uso degli strumenti vengono inserite nei prompt. Alcuni plugin includono le proprie abilità
insieme agli strumenti (per esempio, il plugin voice-call).

Strumenti opzionali forniti dai plugin:

* [Lobster](/it/tools/lobster): runtime di workflow tipizzato con approvazioni riprendibili (richiede la CLI di Lobster sull&#39;host del Gateway).
* [LLM Task](/it/tools/llm-task): step LLM basato solo su JSON per output di workflow strutturato (validazione opzionale dello schema).

<div id="tool-inventory">
  ## Elenco degli strumenti
</div>

<div id="apply_patch">
  ### `apply_patch`
</div>

Applica patch strutturate a uno o più file. Usalo per modifiche con più blocchi (multi-hunk).
Funzionalità sperimentale: attivala tramite `tools.exec.applyPatch.enabled` (solo modelli OpenAI).

<div id="exec">
  ### `exec`
</div>

Esegui comandi shell nello spazio di lavoro.

Parametri principali:

* `command` (obbligatorio)
* `yieldMs` (passa automaticamente in background dopo il timeout, predefinito 10000)
* `background` (passa immediatamente in background)
* `timeout` (secondi; termina il processo se superato, predefinito 1800)
* `elevated` (bool; esegue sull&#39;host se la modalità con privilegi elevati è abilitata/consentita; modifica il comportamento solo quando l&#39;agente è in sandbox)
* `host` (`sandbox | gateway | node`)
* `security` (`deny | allowlist | full`)
* `ask` (`off | on-miss | always`)
* `node` (id/nome del nodo per `host=node`)
* Hai bisogno di un vero TTY? Imposta `pty: true`.

Note:

* Restituisce `status: "running"` con un `sessionId` quando viene eseguito in background.
* Usa `process` per interrogare, registrare log, scrivere, terminare e ripulire le sessioni in background.
* Se `process` non è consentito, `exec` viene eseguito in modo sincrono e ignora `yieldMs`/`background`.
* `elevated` è controllato da `tools.elevated` più qualsiasi override `agents.list[].tools.elevated` (entrambi devono consentirlo) ed è un alias per `host=gateway` + `security=full`.
* `elevated` modifica il comportamento solo quando l&#39;agente è in sandbox (altrimenti è un no-op).
* `host=node` può indirizzare una app macOS companion o un host nodo headless (`openclaw node run`).
* Approvazioni e liste di autorizzati per gateway/node: [Exec approvals](/it/tools/exec-approvals).

<div id="process">
  ### `process`
</div>

Gestisci sessioni di esecuzione in background.

Azioni principali:

* `list`, `poll`, `log`, `write`, `kill`, `clear`, `remove`

Note:

* `poll` restituisce il nuovo output e il codice di uscita al termine.
* `log` supporta `offset`/`limit` a livello di righe (ometti `offset` per ottenere le ultime N righe).
* `process` è in scope per singolo agente; le sessioni appartenenti ad altri agenti non sono visibili.

<div id="web_search">
  ### `web_search`
</div>

Esegui ricerche sul web utilizzando la Brave Search API.

Parametri principali:

* `query` (obbligatorio)
* `count` (1–10; valore predefinito da `tools.web.search.maxResults`)

Note:

* Richiede una Brave API key (consigliato: `openclaw configure --section web`, oppure imposta `BRAVE_API_KEY`).
* Abilita tramite `tools.web.search.enabled`.
* Le risposte vengono messe in cache (15 min per impostazione predefinita).
* Consulta [Web tools](/it/tools/web) per la configurazione.

<div id="web_fetch">
  ### `web_fetch`
</div>

Recupera ed estrae contenuti leggibili da un URL (HTML → markdown/testo).

Parametri principali:

* `url` (obbligatorio)
* `extractMode` (`markdown` | `text`)
* `maxChars` (tronca le pagine molto lunghe)

Note:

* Abilita tramite `tools.web.fetch.enabled`.
* Le risposte vengono messe in cache (15 min per impostazione predefinita).
* Per i siti con uso intensivo di JS, preferisci lo strumento `browser`.
* Vedi [Web tools](/it/tools/web) per la configurazione.
* Vedi [Firecrawl](/it/tools/firecrawl) per il fallback anti-bot opzionale.

<div id="browser">
  ### `browser`
</div>

Controlla il browser dedicato gestito da OpenClaw.

Azioni principali:

* `status`, `start`, `stop`, `tabs`, `open`, `focus`, `close`
* `snapshot` (aria/ai)
* `screenshot` (restituisce un blocco immagine + `MEDIA:<path>`)
* `act` (azioni UI: click/type/press/hover/drag/select/fill/resize/wait/evaluate)
* `navigate`, `console`, `pdf`, `upload`, `dialog`

Gestione dei profili:

* `profiles` — elenca tutti i profili del browser con relativo stato
* `create-profile` — crea un nuovo profilo con porta assegnata automaticamente (o `cdpUrl`)
* `delete-profile` — arresta il browser, elimina i dati utente, lo rimuove dalla config (solo locale)
* `reset-profile` — termina il processo orfano sulla porta del profilo (solo locale)

Parametri comuni:

* `profile` (opzionale; valore predefinito `browser.defaultProfile`)
* `target` (`sandbox` | `host` | `node`)
* `node` (opzionale; seleziona uno specifico id/nome di nodo)

Note:

* Richiede `browser.enabled=true` (il valore predefinito è `true`; imposta `false` per disabilitare).
* Tutte le azioni accettano un parametro opzionale `profile` per il supporto multi‑istanza.
* Quando `profile` è omesso, viene usato `browser.defaultProfile` (predefinito &quot;chrome&quot;).
* Nomi dei profili: solo minuscole alfanumeriche + trattini (max 64 caratteri).
* Intervallo di porte: 18800-18899 (~100 profili max).
* I profili remoti sono solo collegabili (attach‑only; niente start/stop/reset).
* Se è connesso un nodo con capacità browser, lo strumento può instradare automaticamente verso di esso (a meno che tu non blocchi `target` su un valore specifico).
* `snapshot` usa `ai` come predefinito quando Playwright è installato; usa `aria` per l&#39;albero di accessibilità.
* `snapshot` supporta anche opzioni di snapshot per ruolo (`interactive`, `compact`, `depth`, `selector`) che restituiscono riferimenti come `e12`.
* `act` richiede un `ref` da `snapshot` (numero `12` dagli snapshot AI, oppure `e12` dagli snapshot per ruolo); usa `evaluate` per i rari casi che richiedono selettori CSS.
* Evita `act` → `wait` per impostazione predefinita; usalo solo in casi eccezionali (quando non esiste uno stato UI affidabile su cui attendere).
* `upload` può facoltativamente ricevere un `ref` per eseguire un click automatico dopo l’armamento.
* `upload` supporta anche `inputRef` (ref ARIA) o `element` (selettore CSS) per impostare direttamente `<input type="file">`.

<div id="canvas">
  ### `canvas`
</div>

Controlla il Canvas del nodo (present, eval, snapshot, A2UI).

Azioni principali:

* `present`, `hide`, `navigate`, `eval`
* `snapshot` (restituisce un blocco immagine + `MEDIA:<path>`)
* `a2ui_push`, `a2ui_reset`

Note:

* Usa internamente `node.invoke` del Gateway.
* Se non viene fornito alcun `node`, lo strumento seleziona un nodo predefinito (l’unico nodo connesso o il nodo Mac locale).
* A2UI è disponibile solo in v0.8 (nessun `createSurface`); la CLI rifiuta i JSONL v0.9 con errori di riga.
* Verifica rapida: `openclaw nodes canvas a2ui push --node <id> --text "Hello from A2UI"`.

<div id="nodes">
  ### `nodes`
</div>

Individua e indirizza i nodi abbinati; invia notifiche; acquisisci fotocamera/schermo.

Azioni principali:

* `status`, `describe`
* `pending`, `approve`, `reject` (abbinamento)
* `notify` (macOS `system.notify`)
* `run` (macOS `system.run`)
* `camera_snap`, `camera_clip`, `screen_record`
* `location_get`

Note:

* I comandi per fotocamera/schermo richiedono che l&#39;app del nodo sia in esecuzione in primo piano.
* Le immagini restituiscono blocchi immagine + `MEDIA:<path>`.
* I video restituiscono `FILE:<path>` (mp4).
* La posizione restituisce un payload JSON (lat/lon/accuracy/timestamp).
* Parametri di `run`: parametro `command` come array argv; opzionali `cwd`, `env` (`KEY=VAL`), `commandTimeoutMs`, `invokeTimeoutMs`, `needsScreenRecording`.

Esempio (`run`):

```json
{
  "action": "run",
  "node": "office-mac",
  "command": ["echo", "Hello"],
  "env": ["FOO=bar"],
  "commandTimeoutMs": 12000,
  "invokeTimeoutMs": 45000,
  "needsScreenRecording": false
}
```

<div id="image">
  ### `image`
</div>

Analizza un&#39;immagine usando il modello di immagini configurato.

Parametri principali:

* `image` (percorso o URL obbligatorio)
* `prompt` (opzionale; predefinito: &quot;Describe the image.&quot;)
* `model` (override opzionale)
* `maxBytesMb` (limite massimo di dimensione opzionale)

Note:

* Disponibile solo quando `agents.defaults.imageModel` è configurato (primario o di fallback), oppure quando è possibile inferire implicitamente un modello di immagini dal modello predefinito e dall&#39;autenticazione configurata (abbinamento best-effort).
* Usa direttamente il modello di immagini (indipendente dal modello principale di chat).

<div id="message">
  ### `message`
</div>

Invia messaggi e azioni di canale su Discord/Google Chat/Slack/Telegram/WhatsApp/Signal/iMessage/MS Teams.

Azioni principali:

* `send` (testo + media opzionali; MS Teams supporta anche `card` per le Adaptive Card)
* `poll` (sondaggi WhatsApp/Discord/MS Teams)
* `react` / `reactions` / `read` / `edit` / `delete`
* `pin` / `unpin` / `list-pins`
* `permissions`
* `thread-create` / `thread-list` / `thread-reply`
* `search`
* `sticker`
* `member-info` / `role-info`
* `emoji-list` / `emoji-upload` / `sticker-upload`
* `role-add` / `role-remove`
* `channel-info` / `channel-list`
* `voice-status`
* `event-list` / `event-create`
* `timeout` / `kick` / `ban`

Note:

* `send` instrada WhatsApp tramite il Gateway; gli altri canali vengono inviati direttamente.
* `poll` usa il Gateway per WhatsApp e MS Teams; i sondaggi Discord vengono inviati direttamente.
* Quando una chiamata allo strumento `message` è collegata a una sessione di chat attiva, le operazioni di invio sono limitate al destinatario di quella sessione per evitare perdite tra contesti.

<div id="cron">
  ### `cron`
</div>

Gestisci i cron job e i wakeup del Gateway.

Azioni principali:

* `status`, `list`
* `add`, `update`, `remove`, `run`, `runs`
* `wake` (accoda un evento di sistema + heartbeat immediato opzionale)

Note:

* `add` richiede un oggetto cron job completo (stesso schema dell&#39;RPC `cron.add`).
* `update` usa `{ id, patch }`.

<div id="gateway">
  ### `gateway`
</div>

Riavvia o applica aggiornamenti al processo Gateway in esecuzione (in-place).

Azioni principali:

* `restart` (autorizza + invia `SIGUSR1` per un riavvio in-process; riavvia `openclaw gateway` in-place)
* `config.get` / `config.schema`
* `config.apply` (valida + scrive la configurazione + riavvia + riattiva)
* `config.patch` (applica l’aggiornamento parziale unendolo + riavvia + riattiva)
* `update.run` (esegue l’aggiornamento + riavvia + riattiva)

Note:

* Usa `delayMs` (predefinito 2000) per evitare di interrompere una risposta in corso.
* `restart` è disabilitato per impostazione predefinita; abilitalo con `commands.restart: true`.

<div id="sessions_list-sessions_history-sessions_send-sessions_spawn-session_status">
  ### `sessions_list` / `sessions_history` / `sessions_send` / `sessions_spawn` / `session_status`
</div>

Elenca le sessioni, visualizza la cronologia della trascrizione o invia a un&#39;altra sessione.

Parametri principali:

* `sessions_list`: `kinds?`, `limit?`, `activeMinutes?`, `messageLimit?` (0 = nessun limite)
* `sessions_history`: `sessionKey` (o `sessionId`), `limit?`, `includeTools?`
* `sessions_send`: `sessionKey` (o `sessionId`), `message`, `timeoutSeconds?` (0 = fire-and-forget)
* `sessions_spawn`: `task`, `label?`, `agentId?`, `model?`, `runTimeoutSeconds?`, `cleanup?`
* `session_status`: `sessionKey?` (predefinito: sessione corrente; accetta `sessionId`), `model?` (`default` rimuove l&#39;override)

Note:

* `main` è la chiave canonica per la chat diretta; global/unknown sono nascoste.
* `messageLimit > 0` recupera gli ultimi N messaggi per sessione (messaggi degli strumenti filtrati).
* `sessions_send` attende il completamento finale quando `timeoutSeconds > 0`.
* La consegna/l&#39;annuncio avviene dopo il completamento ed è best-effort; `status: "ok"` conferma che l&#39;esecuzione dell&#39;agente è terminata, non che l&#39;annuncio sia stato consegnato.
* `sessions_spawn` avvia l&#39;esecuzione di un sotto-agente e pubblica una risposta di annuncio nella chat del richiedente.
* `sessions_spawn` è non bloccante e restituisce `status: "accepted"` immediatamente.
* `sessions_send` esegue un ping‑pong di risposta (rispondi con `REPLY_SKIP` per interrompere; numero massimo di turni tramite `session.agentToAgent.maxPingPongTurns`, 0–5).
* Dopo il ping‑pong, l&#39;agente di destinazione esegue una **fase di annuncio**; rispondi `ANNOUNCE_SKIP` per sopprimere l&#39;annuncio.

<div id="agents_list">
  ### `agents_list`
</div>

Elenca gli ID degli agenti che la sessione corrente può indirizzare con `sessions_spawn`.

Note:

* Il risultato è limitato dalle liste di autorizzati per singolo agente (`agents.list[].subagents.allowAgents`).
* Quando si configura `["*"]`, lo strumento include tutti gli agenti configurati e imposta `allowAny: true`.

<div id="parameters-common">
  ## Parametri (comuni)
</div>

Strumenti basati sul Gateway (`canvas`, `nodes`, `cron`):

* `gatewayUrl` (predefinito `ws://127.0.0.1:18789`)
* `gatewayToken` (se l&#39;autenticazione è abilitata)
* `timeoutMs`

Strumento browser:

* `profile` (opzionale; predefinito `browser.defaultProfile`)
* `target` (`sandbox` | `host` | `node`)
* `node` (opzionale; fissa un id/nome nodo specifico)

<div id="recommended-agent-flows">
  ## Flussi di lavoro dell&#39;agente consigliati
</div>

Automazione del browser:

1. `browser` → `status` / `start`
2. `snapshot` (ai o aria)
3. `act` (click/type/press)
4. `screenshot` se ti serve una conferma visiva

Rendering del canvas:

1. `canvas` → `present`
2. `a2ui_push` (opzionale)
3. `snapshot`

Selezione dei nodi di destinazione:

1. `nodes` → `status`
2. `describe` sul nodo scelto
3. `notify` / `run` / `camera_snap` / `screen_record`

<div id="safety">
  ## Sicurezza
</div>

* Evita di usare direttamente `system.run`; usa `nodes` → `run` solo con consenso esplicito dell&#39;utente.
* Rispetta il consenso dell&#39;utente per l&#39;acquisizione da fotocamera/schermo.
* Usa `status/describe` per verificare le autorizzazioni prima di invocare comandi multimediali.

<div id="how-tools-are-presented-to-the-agent">
  ## Come gli strumenti vengono presentati all&#39;agente
</div>

Gli strumenti sono esposti tramite due canali paralleli:

1. **Testo del prompt di sistema**: un elenco leggibile da esseri umani + indicazioni.
2. **Schema dello strumento**: le definizioni di funzione strutturate inviate all&#39;API del modello.

Questo significa che l&#39;agente vede sia &quot;quali strumenti esistono&quot; sia &quot;come chiamarli&quot;. Se uno strumento
non è presente nel prompt di sistema o nello schema, il modello non può chiamarlo.