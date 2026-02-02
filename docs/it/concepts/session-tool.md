---
title: Strumento di sessione
summary: "Strumenti di sessione dell'agente per elencare le sessioni, recuperare la cronologia e inviare messaggi tra sessioni"
read_when:
  - Aggiunta o modifica degli strumenti di sessione
---

<div id="session-tools">
  # Strumenti di sessione
</div>

Obiettivo: un set di strumenti essenziale e difficile da usare in modo errato, che permetta agli agenti di elencare le sessioni, recuperare la cronologia e inviare a un'altra sessione.

<div id="tool-names">
  ## Nomi degli strumenti
</div>

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

<div id="key-model">
  ## Modello di chiavi
</div>

- Il bucket principale per la chat diretta è sempre la chiave letterale `"main"` (risolta nella chiave principale dell’agente corrente).
- Le chat di gruppo usano `agent:<agentId>:<channel>:group:<id>` oppure `agent:<agentId>:<channel>:channel:<id>` (passa la chiave completa).
- I job cron usano `cron:<job.id>`.
- Gli hook usano `hook:<uuid>` a meno che non siano impostati esplicitamente.
- Le sessioni del nodo usano `node-<nodeId>` a meno che non siano impostate esplicitamente.

`global` e `unknown` sono valori riservati e non vengono mai elencati. Se `session.scope = "global"`, lo mappiamo su `main` per tutti gli strumenti, in modo che i chiamanti non vedano mai `global`.

<div id="sessions_list">
  ## sessions_list
</div>

Elenca le sessioni come un array di righe.

Parametri:

- `kinds?: string[]` filtro: uno qualsiasi tra `"main" | "group" | "cron" | "hook" | "node" | "other"`
- `limit?: number` numero massimo di righe (predefinito: valore predefinito del server, limitato ad es. a 200)
- `activeMinutes?: number` solo le sessioni aggiornate negli ultimi N minuti
- `messageLimit?: number` 0 = nessun messaggio (predefinito 0); >0 = includi gli ultimi N messaggi

Comportamento:

- `messageLimit > 0` recupera `chat.history` per sessione e include gli ultimi N messaggi.
- I risultati degli strumenti sono esclusi dall’output dell’elenco; usa `sessions_history` per i messaggi degli strumenti.
- Quando eseguito in una sessione di agente **in sandbox**, gli strumenti di sessione hanno per impostazione predefinita **visibilità solo per i processi generati** (vedi sotto).

Forma della riga (JSON):

- `key`: chiave della sessione (string)
- `kind`: `main | group | cron | hook | node | other`
- `channel`: `whatsapp | telegram | discord | signal | imessage | webchat | internal | unknown`
- `displayName` (etichetta di visualizzazione del gruppo, se disponibile)
- `updatedAt` (ms)
- `sessionId`
- `model`, `contextTokens`, `totalTokens`
- `thinkingLevel`, `verboseLevel`, `systemSent`, `abortedLastRun`
- `sendPolicy` (override di sessione, se impostato)
- `lastChannel`, `lastTo`
- `deliveryContext` (`{ channel, to, accountId }` normalizzato, quando disponibile)
- `transcriptPath` (percorso best-effort derivato da directory dello store + sessionId)
- `messages?` (solo quando `messageLimit > 0`)

<div id="sessions_history">
  ## sessions_history
</div>

Recupera il transcript di una singola sessione.

Parametri:

- `sessionKey` (obbligatorio; accetta la chiave di sessione oppure un `sessionId` da `sessions_list`)
- `limit?: number` numero massimo di messaggi (con limite imposto dal server)
- `includeTools?: boolean` (valore predefinito false)

Comportamento:

- Con `includeTools=false` vengono filtrati i messaggi con `role: "toolResult"`.
- Restituisce un array di messaggi nel formato grezzo del transcript.
- Quando viene fornito un `sessionId`, OpenClaw lo risolve nella corrispondente chiave di sessione (errore se l’ID è mancante/inesistente).

<div id="sessions_send">
  ## sessions_send
</div>

Invia un messaggio a un'altra sessione.

Parametri:

- `sessionKey` (obbligatorio; accetta la chiave di sessione o `sessionId` da `sessions_list`)
- `message` (obbligatorio)
- `timeoutSeconds?: number` (predefinito >0; 0 = modalità "fire-and-forget")

Comportamento:

- `timeoutSeconds = 0`: inserisce in coda e restituisce `{ runId, status: "accepted" }`.
- `timeoutSeconds > 0`: attende fino a N secondi il completamento, quindi restituisce `{ runId, status: "ok", reply }`.
- Se l'attesa va in timeout: `{ runId, status: "timeout", error }`. L'esecuzione continua; chiama `sessions_history` più tardi.
- Se l'esecuzione fallisce: `{ runId, status: "error", error }`.
- Le esecuzioni di annuncio della consegna vengono avviate dopo il completamento dell'esecuzione primaria e sono best-effort; `status: "ok"` non garantisce che l'annuncio sia stato effettivamente consegnato.
- Attende tramite `agent.wait` del Gateway (lato server) in modo che le riconnessioni non interrompano l'attesa.
- Il contesto del messaggio agente‑a‑agente viene iniettato per l'esecuzione primaria.
- Dopo che l'esecuzione primaria è completa, OpenClaw esegue un **reply-back loop**:
  - Dal secondo round in poi alterna tra agente richiedente e agente di destinazione.
  - Rispondi esattamente `REPLY_SKIP` per interrompere il ping‑pong.
  - Il numero massimo di turni è `session.agentToAgent.maxPingPongTurns` (0–5, predefinito 5).
- Una volta che il loop termina, OpenClaw esegue il **passaggio di annuncio agente‑a‑agente** (solo agente di destinazione):
  - Rispondi esattamente `ANNOUNCE_SKIP` per rimanere in silenzio.
  - Qualsiasi altra risposta viene inviata al canale di destinazione.
  - Il passaggio di annuncio include la richiesta originale + la risposta del round 1 + l'ultima risposta del ping‑pong.

<div id="channel-field">
  ## Campo Channel
</div>

- Per i gruppi, `channel` è il canale registrato nella voce di sessione.
- Per le chat dirette, `channel` viene ricavato da `lastChannel`.
- Per cron/hook/node, `channel` è `internal`.
- Se mancante, `channel` è `unknown`.

<div id="security-send-policy">
  ## Sicurezza / Criteri di invio
</div>

Blocco basato su criteri in base al tipo di canale/chat (non per ID di sessione).

```json
{
  "session": {
    "sendPolicy": {
      "rules": [
        {
          "match": { "channel": "discord", "chatType": "group" },
          "action": "deny"
        }
      ],
      "default": "allow"
    }
  }
}
```

Override a runtime (per voce di sessione):

* `sendPolicy: "allow" | "deny"` (non impostato = eredita la configurazione)
* Impostabile tramite `sessions.patch` o solo dal proprietario con `/send on|off|inherit` (messaggio autonomo).

Punti di applicazione:

* `chat.send` / `agent` (Gateway)
* logica di recapito delle risposte automatiche


<div id="sessions_spawn">
  ## sessions_spawn
</div>

Avvia l'esecuzione di un sotto-agente in una sessione isolata e annuncia il risultato nel canale chat del richiedente.

Parametri:

- `task` (obbligatorio)
- `label?` (facoltativo; usato per log/UI)
- `agentId?` (facoltativo; avvia sotto un altro id di agente, se consentito)
- `model?` (facoltativo; esegue l'override del modello del sotto-agente; i valori non validi generano un errore)
- `runTimeoutSeconds?` (valore predefinito 0; se impostato, interrompe l'esecuzione del sotto-agente dopo N secondi)
- `cleanup?` (`delete|keep`, valore predefinito `keep`)

Lista di autorizzati:

- `agents.list[].subagents.allowAgents`: elenco di id di agente autorizzati tramite `agentId` (`["*"]` per consentire qualsiasi). Predefinito: solo l'agente richiedente.

Discovery:

- Usa `agents_list` per scoprire quali id di agente sono autorizzati per `sessions_spawn`.

Comportamento:

- Avvia una nuova sessione `agent:<agentId>:subagent:<uuid>` con `deliver: false`.
- I sotto-agenti, per impostazione predefinita, hanno accesso all'intero set di strumenti **meno gli strumenti di sessione** (configurabile tramite `tools.subagents.tools`).
- I sotto-agenti non sono autorizzati a chiamare `sessions_spawn` (niente sotto-agente → sotto-agente annidato).
- Sempre non bloccante: restituisce immediatamente `{ status: "accepted", runId, childSessionKey }`.
- Al termine, OpenClaw esegue una **fase di annuncio** del sotto-agente e pubblica il risultato nel canale chat del richiedente.
- Rispondi esattamente `ANNOUNCE_SKIP` durante la fase di annuncio per non inviare alcun messaggio.
- Le risposte di annuncio sono normalizzate in `Status`/`Result`/`Notes`; `Status` deriva dall'esito di runtime (non dal testo del modello).
- Le sessioni dei sotto-agenti vengono archiviate automaticamente dopo `agents.defaults.subagents.archiveAfterMinutes` (valore predefinito: 60).
- Le risposte di annuncio includono una riga di statistiche (runtime, token, sessionKey/sessionId, percorso del transcript e costo facoltativo).

<div id="sandbox-session-visibility">
  ## Visibilità delle sessioni sandbox
</div>

Le sessioni in sandbox possono usare gli strumenti di sessione, ma per impostazione predefinita vedono solo le sessioni che hanno avviato tramite `sessions_spawn`.

Configurazione:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        // predefinito: "spawned"
        sessionToolsVisibility: "spawned" // oppure "all"
      }
    }
  }
}
```
