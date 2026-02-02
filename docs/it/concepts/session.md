---
title: Sessione
summary: "Regole, chiavi e persistenza delle sessioni di chat"
read_when:
  - Modifica della gestione o dell'archiviazione delle sessioni
---

<div id="session-management">
  # Gestione delle sessioni
</div>

OpenClaw considera **una sessione di chat diretta per agente** come primaria. Le chat dirette vengono ricondotte a `agent:<agentId>:<mainKey>` (predefinito `main`), mentre le chat di gruppo/canale ottengono le proprie chiavi. `session.mainKey` viene rispettata.

Usa `session.dmScope` per controllare come vengono raggruppati i **messaggi diretti**:

* `main` (predefinito): tutti i DM condividono la sessione principale per continuità.
* `per-peer`: isola per ID del mittente tra i canali.
* `per-channel-peer`: isola per canale + mittente (consigliato per caselle di posta multi‑utente).
* `per-account-channel-peer`: isola per account + canale + mittente (consigliato per caselle di posta multi‑account).
  Usa `session.identityLinks` per mappare gli ID dei peer con prefisso del provider a un&#39;identità canonica, in modo che la stessa persona condivida una sessione DM tra i canali quando usi `per-peer`, `per-channel-peer` o `per-account-channel-peer`.

<div id="gateway-is-the-source-of-truth">
  ## Il Gateway è la fonte di verità
</div>

Tutto lo stato della sessione è **di esclusiva competenza del Gateway** (l’OpenClaw “master”). I client UI (app macOS, WebChat, ecc.) devono interrogare il Gateway per gli elenchi di sessioni e i conteggi di token invece di leggere file locali.

* In **remote mode**, l’archivio delle sessioni che ti interessa risiede sull’host Gateway remoto, non sul tuo Mac.
* I conteggi di token mostrati nelle UI provengono dai campi dello store del Gateway (`inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`). I client non eseguono il parsing delle trascrizioni JSONL per “correggere” i totali.

<div id="where-state-lives">
  ## Dove risiede lo stato
</div>

* Sull’**host del Gateway**:
  * File dello store: `~/.openclaw/agents/<agentId>/sessions/sessions.json` (per agente).
* Trascrizioni: `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl` (le sessioni dei topic Telegram usano `.../<SessionId>-topic-<threadId>.jsonl`).
* Lo store è una mappa `sessionKey -> { sessionId, updatedAt, ... }`. Eliminare le voci è sicuro; vengono ricreate su richiesta.
* Le voci di gruppo possono includere `displayName`, `channel`, `subject`, `room` e `space` per etichettare le sessioni nelle UI.
* Le voci di sessione includono metadati `origin` (etichetta + indicazioni di instradamento) in modo che le UI possano spiegare da dove proviene una sessione.
* OpenClaw **non** legge le cartelle di sessione legacy Pi/Tau.

<div id="session-pruning">
  ## Pruning delle sessioni
</div>

Per impostazione predefinita, OpenClaw rimuove **risultati obsoleti degli strumenti** dal contesto in memoria subito prima delle chiamate all&#39;LLM.
Questo **non** riscrive la cronologia JSONL. Vedi [/concepts/session-pruning](/it/concepts/session-pruning).

<div id="pre-compaction-memory-flush">
  ## Svuotamento della memoria prima della compattazione
</div>

Quando una sessione si avvicina alla compattazione automatica, OpenClaw può eseguire un **svuotamento silenzioso della memoria**, un turno che ricorda al modello di scrivere note persistenti sul disco. Questo viene eseguito solo quando lo spazio di lavoro è scrivibile. Consulta [Memoria](/it/concepts/memory) e
[Compattazione](/it/concepts/compaction).

<div id="mapping-transports-session-keys">
  ## Mappatura dei trasporti → chiavi di sessione
</div>

* Le chat dirette seguono `session.dmScope` (valore predefinito `main`).
  * `main`: `agent:<agentId>:<mainKey>` (continuità tra dispositivi/canali).
    * Più numeri di telefono e canali possono essere associati alla stessa chiave principale dell&#39;agente; fungono da trasporti per un&#39;unica conversazione.
  * `per-peer`: `agent:<agentId>:dm:<peerId>`.
  * `per-channel-peer`: `agent:<agentId>:<channel>:dm:<peerId>`.
  * `per-account-channel-peer`: `agent:<agentId>:<channel>:<accountId>:dm:<peerId>` (accountId predefinito `default`).
  * Se `session.identityLinks` corrisponde a un peer ID con prefisso di provider (per esempio `telegram:123`), la chiave canonica sostituisce `<peerId>` così la stessa persona condivide una sessione tra canali.
* Le chat di gruppo isolano lo stato: `agent:<agentId>:<channel>:group:<id>` (stanze/canali usano `agent:<agentId>:<channel>:channel:<id>`).
  * I topic dei forum Telegram aggiungono `:topic:<threadId>` all&#39;ID del gruppo per l&#39;isolamento.
  * Le chiavi legacy `group:<id>` sono ancora riconosciute per la migrazione.
* I contesti in ingresso possono ancora usare `group:<id>`; il canale viene dedotto dal provider e normalizzato alla forma canonica `agent:<agentId>:<channel>:group:<id>`.
* Altre origini:
  * Job cron: `cron:<job.id>`
  * Webhook: `hook:<uuid>` (a meno che non sia impostata esplicitamente dall&#39;hook)
  * Esecuzioni del nodo: `node-<nodeId>`

<div id="lifecycle">
  ## Ciclo di vita
</div>

* Criterio di reset: le sessioni vengono riutilizzate fino alla loro scadenza, che viene valutata al successivo messaggio in ingresso.
* Reset giornaliero: per impostazione predefinita alle **4:00 del mattino ora locale sull&#39;host del Gateway**. Una sessione è considerata obsoleta quando il suo ultimo aggiornamento è precedente all&#39;orario più recente di reset giornaliero.
* Reset per inattività (opzionale): `idleMinutes` aggiunge una finestra di inattività scorrevole. Quando sono configurati sia il reset giornaliero sia quello per inattività, **quello che scade per primo** forza l’apertura di una nuova sessione.
* Solo inattività legacy: se imposti `session.idleMinutes` senza alcuna configurazione `session.reset`/`resetByType`, OpenClaw rimane in modalità solo inattività per compatibilità con le versioni precedenti.
* Override per tipo (opzionale): `resetByType` ti permette di sovrascrivere il criterio per le sessioni `dm`, `group` e `thread` (thread = thread Slack/Discord, topic Telegram, thread Matrix quando supportati dal connettore).
* Override per canale (opzionale): `resetByChannel` sovrascrive il criterio di reset per un canale (si applica a tutti i tipi di sessione per quel canale e ha precedenza su `reset`/`resetByType`).
* Trigger di reset: `/new` o `/reset` esatti (più eventuali extra in `resetTriggers`) avviano un nuovo id di sessione e fanno passare il resto del messaggio. `/new <model>` accetta un alias di modello, `provider/model` o il nome del provider (corrispondenza approssimativa/fuzzy match) per impostare il modello della nuova sessione. Se `/new` o `/reset` vengono inviati da soli, OpenClaw esegue un breve turno di saluto “hello” per confermare il reset.
* Reset manuale: elimina chiavi specifiche dall’archivio oppure rimuovi il transcript JSONL; il messaggio successivo le/lo ricrea.
* I job cron isolati generano sempre un nuovo `sessionId` per ogni esecuzione (nessun riutilizzo in base all’inattività).

<div id="send-policy-optional">
  ## Criterio di invio (opzionale)
</div>

Blocca l&#39;invio per tipi specifici di sessione senza elencare gli ID individuali.

```json5
{
  session: {
    sendPolicy: {
      rules: [
        { action: "deny", match: { channel: "discord", chatType: "group" } },
        { action: "deny", match: { keyPrefix: "cron:" } }
      ],
      default: "allow"
    }
  }
}
```

Override a runtime (solo proprietario):

* `/send on` → consenti per questa sessione
* `/send off` → nega per questa sessione
* `/send inherit` → cancella l&#39;override e applica le regole di configurazione
  Invia questi comandi come messaggi separati in modo che vengano registrati.

<div id="configuration-optional-rename-example">
  ## Configurazione (esempio di rinomina opzionale)
</div>

```json5
// ~/.openclaw/openclaw.json
{
  session: {
    scope: "per-sender",      // keep group keys separate
    dmScope: "main",          // DM continuity (set per-channel-peer/per-account-channel-peer for shared inboxes)
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"]
    },
    reset: {
      // Predefiniti: mode=daily, atHour=4 (ora locale dell'host del gateway).
      // Se imposti anche idleMinutes, prevale quello che scade per primo.
      mode: "daily",
      atHour: 4,
      idleMinutes: 120
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      dm: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 }
    },
    resetByChannel: {
      discord: { mode: "idle", idleMinutes: 10080 }
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    mainKey: "main",
  }
}
```

<div id="inspecting">
  ## Ispezione
</div>

* `openclaw status` — mostra il percorso dello store e le sessioni recenti.
* `openclaw sessions --json` — stampa tutte le voci (filtra con `--active <minutes>`).
* `openclaw gateway call sessions.list --params '{}'` — recupera le sessioni dal Gateway in esecuzione (usa `--url`/`--token` per l’accesso a un Gateway remoto).
* Invia `/status` come messaggio a sé stante in chat per verificare se l’agente è raggiungibile, quanta parte del contesto di sessione è utilizzata, gli attuali toggle di thinking/verbose e quando le tue credenziali WhatsApp Web sono state aggiornate l’ultima volta (aiuta a individuare quando è necessario ristabilire il collegamento).
* Invia `/context list` o `/context detail` per vedere cosa c’è nel system prompt e nei file di spazio di lavoro iniettati (e i principali contributori al contesto).
* Invia `/stop` come messaggio a sé stante per interrompere l’esecuzione corrente, cancellare i follow-up in coda per quella sessione e fermare eventuali esecuzioni di sub-agenti avviate da essa (la risposta include il conteggio delle esecuzioni fermate).
* Invia `/compact` (istruzioni opzionali) come messaggio a sé stante per riassumere il contesto più vecchio e liberare spazio nella finestra. Vedi [/concepts/compaction](/it/concepts/compaction).
* Le trascrizioni JSONL possono essere aperte direttamente per rivedere i turni completi.

<div id="tips">
  ## Suggerimenti
</div>

* Mantieni la chiave primaria dedicata al traffico 1:1; lascia che i gruppi utilizzino chiavi proprie.
* Quando automatizzi le operazioni di pulizia, elimina le singole chiavi invece dell’intero archivio per preservare il contesto altrove.

<div id="session-origin-metadata">
  ## Metadati di origine della sessione
</div>

Ogni voce di sessione registra, per quanto possibile, da dove proviene in `origin`:

* `label`: etichetta leggibile dall&#39;utente (derivata dall&#39;etichetta della conversazione + oggetto/canale del gruppo)
* `provider`: ID normalizzato del canale (incluse le estensioni)
* `from`/`to`: ID di instradamento grezzi dalla busta del messaggio in ingresso
* `accountId`: ID dell&#39;account del provider (in configurazioni multi-account)
* `threadId`: ID del thread/argomento quando il canale lo supporta

I campi di origine sono valorizzati per messaggi diretti, canali e gruppi. Se un
connettore aggiorna solo l&#39;instradamento di consegna (ad esempio, per mantenere
aggiornata la sessione principale di un DM), deve comunque fornire il contesto
in ingresso affinché la sessione mantenga i suoi metadati descrittivi. Le
estensioni possono farlo inviando `ConversationLabel`, `GroupSubject`,
`GroupChannel`, `GroupSpace` e `SenderName` nel contesto in ingresso e
chiamando `recordSessionMetaFromInbound` (o passando lo stesso contesto a
`updateLastRoute`).