---
title: Gestione e compattazione delle sessioni
summary: "Approfondimento: session store e trascritti, ciclo di vita e meccanismi interni di (auto)compattazione"
read_when:
  - Devi eseguire il debug degli ID di sessione, del transcript JSONL o dei campi di sessions.json
  - Stai cambiando il comportamento di auto-compattazione o aggiungendo operazioni di manutenzione “pre-compattazione”
  - Vuoi implementare svuotamenti di memoria o turni di sistema silenziosi
---

<div id="session-management-compaction-deep-dive">
  # Gestione delle sessioni e compattazione (Approfondimento)
</div>

Questo documento spiega come OpenClaw gestisce le sessioni end-to-end:

* **Instradamento delle sessioni** (come i messaggi in ingresso vengono mappati a un `sessionKey`)
* **Archivio delle sessioni** (`sessions.json`) e che cosa registra
* **Persistenza dei transcript** (`*.jsonl`) e la loro struttura
* **Igiene dei transcript** (correzioni specifiche per provider prima delle esecuzioni)
* **Limiti di contesto** (finestra di contesto vs token tracciati)
* **Compattazione** (compattazione manuale + automatica) e dove agganciare le attività di pre-compattazione
* **Manutenzione silenziosa** (ad esempio scritture di memoria che non dovrebbero produrre output visibile all’utente)

Se vuoi prima una panoramica di livello più alto, inizia da:

* [/concepts/session](/it/concepts/session)
* [/concepts/compaction](/it/concepts/compaction)
* [/concepts/session-pruning](/it/concepts/session-pruning)
* [/reference/transcript-hygiene](/it/reference/transcript-hygiene)

***

<div id="source-of-truth-the-gateway">
  ## Fonte di verità: il Gateway
</div>

OpenClaw è progettato attorno a un singolo **processo Gateway** che gestisce lo stato delle sessioni.

* Le UI (app macOS, Control UI web, TUI) devono interrogare il Gateway per ottenere gli elenchi delle sessioni e il conteggio dei token.
* In modalità remota, i file di sessione risiedono sull&#39;host remoto; “controllare i file locali del tuo Mac” non rifletterà ciò che il Gateway sta effettivamente usando.

***

<div id="two-persistence-layers">
  ## Due livelli di persistenza
</div>

OpenClaw mantiene le sessioni su due livelli:

1. **Session store (`sessions.json`)**
   * Mappa chiave/valore: `sessionKey -> SessionEntry`
   * Piccolo, mutabile, sicuro da modificare (o eliminare elementi)
   * Tiene traccia dei metadati della sessione (ID della sessione corrente, ultima attività, flag/impostazioni, contatori di token, ecc.)

2. **Transcript (`<sessionId>.jsonl`)**
   * Transcript in sola aggiunta (append-only) con struttura ad albero (gli elementi hanno `id` + `parentId`)
   * Archivia la conversazione effettiva + le chiamate agli strumenti + i sommari di compattazione
   * Usato per ricostruire il contesto del modello per i turni futuri

***

<div id="on-disk-locations">
  ## Percorsi su disco
</div>

Per ogni agente, sull&#39;host del Gateway:

* Archivio: `~/.openclaw/agents/<agentId>/sessions/sessions.json`
* Trascrizioni: `~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl`
  * Sessioni per i topic di Telegram: `.../<sessionId>-topic-<threadId>.jsonl`

OpenClaw risolve questi percorsi tramite `src/config/sessions.ts`.

***

<div id="session-keys-sessionkey">
  ## Chiavi di sessione (`sessionKey`)
</div>

Una `sessionKey` identifica *in quale contenitore logico di conversazione* ti trovi (routing + isolamento).

Pattern comuni:

* Chat principale/diretta (per agente): `agent:<agentId>:<mainKey>` (predefinito `main`)
* Gruppo: `agent:<agentId>:<channel>:group:<id>`
* Stanza/canale (Discord/Slack): `agent:<agentId>:<channel>:channel:<id>` oppure `...:room:<id>`
* Cron: `cron:<job.id>`
* Webhook: `hook:<uuid>` (a meno che non venga ridefinito)

Le regole canoniche sono documentate in [/concepts/session](/it/concepts/session).

***

<div id="session-ids-sessionid">
  ## ID di sessione (`sessionId`)
</div>

Ogni `sessionKey` punta a un `sessionId` corrente (il file di trascrizione che continua la conversazione).

Regole pratiche:

* **Reset** (`/new`, `/reset`) crea un nuovo `sessionId` per quel `sessionKey`.
* **Reset giornaliero** (per impostazione predefinita alle 4:00 del mattino ora locale sull&#39;host del Gateway) crea un nuovo `sessionId` al messaggio successivo dopo il limite di reset.
* **Scadenza per inattività** (`session.reset.idleMinutes` o `session.idleMinutes` legacy) crea un nuovo `sessionId` quando arriva un messaggio dopo la finestra di inattività. Quando reset giornaliero e scadenza per inattività sono entrambi configurati, prevale quello che scade per primo.

Dettaglio di implementazione: la decisione avviene in `initSessionState()` in `src/auto-reply/reply/session.ts`.

***

<div id="session-store-schema-sessionsjson">
  ## Schema dell’archivio delle sessioni (`sessions.json`)
</div>

Il tipo di valore dell’archivio è `SessionEntry` in `src/config/sessions.ts`.

Campi principali (non esaustivo):

* `sessionId`: ID della trascrizione corrente (il nome file è derivato da questo, a meno che `sessionFile` non sia impostato)
* `updatedAt`: timestamp dell’ultima attività
* `sessionFile`: percorso esplicito opzionale che sostituisce quello predefinito della trascrizione
* `chatType`: `direct | group | room` (aiuta le UI e i criteri di invio)
* `provider`, `subject`, `room`, `space`, `displayName`: metadati per l’etichettatura di gruppi/canali
* Interruttori:
  * `thinkingLevel`, `verboseLevel`, `reasoningLevel`, `elevatedLevel`
  * `sendPolicy` (override per sessione)
* Selezione del modello:
  * `providerOverride`, `modelOverride`, `authProfileOverride`
* Contatori di token (best-effort / dipendenti dal provider):
  * `inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`
* `compactionCount`: quante volte l’auto-compattazione è stata completata per questa chiave di sessione
* `memoryFlushAt`: timestamp dell’ultimo svuotamento della memoria prima della compattazione
* `memoryFlushCompactionCount`: valore del contatore di compattazione quando è stato eseguito l’ultimo svuotamento

L’archivio può essere modificato in sicurezza, ma il Gateway fa fede: può riscrivere o reidratare le voci mentre le sessioni sono in esecuzione.

***

<div id="transcript-structure-jsonl">
  ## Struttura della trascrizione (`*.jsonl`)
</div>

Le trascrizioni sono gestite dal `SessionManager` di `@mariozechner/pi-coding-agent`.

Il file è in formato JSONL:

* Prima riga: intestazione di sessione (`type: "session"`, include `id`, `cwd`, `timestamp`, opzionale `parentSession`)
* Poi: voci di sessione con `id` + `parentId` (albero)

Tipi di voce principali:

* `message`: messaggi user/assistant/toolResult
* `custom_message`: messaggi iniettati da estensioni che *entrano* nel contesto del modello (possono essere nascosti dalla UI)
* `custom`: stato dell&#39;estensione che *non* entra nel contesto del modello
* `compaction`: riepilogo di compattazione memorizzato in modo persistente con `firstKeptEntryId` e `tokensBefore`
* `branch_summary`: riepilogo memorizzato in modo persistente durante la navigazione di un ramo dell&#39;albero

OpenClaw intenzionalmente **non** “corregge” le trascrizioni; il Gateway usa `SessionManager` per leggerle e scriverle.

<div id="context-windows-vs-tracked-tokens">
  ## Finestra di contesto vs token tracciati
</div>

Entrano in gioco due concetti diversi:

1. **Finestra di contesto del modello**: limite rigido per modello (token visibili al modello)
2. **Contatori dello store delle sessioni**: statistiche a scorrimento scritte in `sessions.json` (usate per /status e dashboard)

Se stai regolando i limiti:

* La finestra di contesto proviene dal catalogo dei modelli (e può essere sovrascritta tramite la configurazione).
* `contextTokens` nello store è una stima/un valore di report a runtime; non considerarlo una garanzia rigida.

Per maggiori dettagli, vedi [/token-use](/it/token-use).

***

<div id="compaction-what-it-is">
  ## Compattazione: che cos&#39;è
</div>

La compattazione riassume le conversazioni meno recenti in una voce `compaction` memorizzata in modo persistente nella trascrizione e mantiene intatti i messaggi recenti.

Dopo la compattazione, i turni successivi vedono:

* Il riepilogo di compattazione
* I messaggi dopo `firstKeptEntryId`

La compattazione è **persistente** (a differenza del pruning della sessione). Consulta [/concepts/session-pruning](/it/concepts/session-pruning).

***

<div id="when-auto-compaction-happens-pi-runtime">
  ## Quando avviene l’auto-compattazione (runtime Pi)
</div>

Nell’agente Pi integrato, l’auto-compattazione si attiva in due casi:

1. **Recupero da overflow**: il modello restituisce un errore di overflow del contesto → compattazione → nuovo tentativo.
2. **Mantenimento della soglia**: dopo un turno riuscito, quando:

`contextTokens > contextWindow - reserveTokens`

Dove:

* `contextWindow` è la finestra di contesto del modello
* `reserveTokens` è il margine riservato per i prompt + il successivo output del modello

Questa è la semantica del runtime Pi (OpenClaw consuma gli eventi, ma è Pi a decidere quando compattare).

***

<div id="compaction-settings-reservetokens-keeprecenttokens">
  ## Impostazioni di compattazione (`reserveTokens`, `keepRecentTokens`)
</div>

Le impostazioni di compattazione di Pi si configurano nelle impostazioni di Pi:

```json5
{
  compaction: {
    enabled: true,
    reserveTokens: 16384,
    keepRecentTokens: 20000
  }
}
```

OpenClaw applica anche una soglia minima di sicurezza per le esecuzioni embedded:

* Se `compaction.reserveTokens < reserveTokensFloor`, OpenClaw lo incrementa.
* La soglia minima predefinita è di `20000` token.
* Imposta `agents.defaults.compaction.reserveTokensFloor: 0` per disabilitare la soglia minima.
* Se è già superiore, OpenClaw lo lascia invariato.

Motivo: lasciare sufficiente margine per le attività di “manutenzione” a più turni (come le scritture di memoria) prima che la compattazione diventi inevitabile.

Implementazione: `ensurePiCompactionReserveTokens()` in `src/agents/pi-settings.ts`
(chiamata da `src/agents/pi-embedded-runner.ts`).

***

<div id="user-visible-surfaces">
  ## Interfacce visibili all&#39;utente
</div>

Puoi monitorare la compattazione e lo stato della sessione tramite:

* `/status` (in qualsiasi sessione di chat)
* `openclaw status` (CLI)
* `openclaw sessions` / `sessions --json`
* Modalità dettagliata: `🧹 Auto-compaction complete` + numero di compattazioni

***

<div id="silent-housekeeping-no_reply">
  ## Manutenzione silenziosa (`NO_REPLY`)
</div>

OpenClaw supporta turni “silenziosi” per attività in background in cui l&#39;utente non deve vedere output intermedi.

Convenzione:

* L&#39;assistente inizia il proprio output con `NO_REPLY` per indicare “non recapitare una risposta all&#39;utente”.
* OpenClaw rimuove/sopprime questo indicatore nel livello di consegna.

A partire dalla versione `2026.1.10`, OpenClaw sopprime anche lo **streaming di bozze/digitazione** quando un blocco (chunk) parziale inizia con `NO_REPLY`, in modo che le operazioni silenziose non facciano trapelare output parziali a metà turno.

***

<div id="pre-compaction-memory-flush-implemented">
  ## “Flush di memoria” pre-compattazione (implementato)
</div>

Obiettivo: prima che avvenga la compattazione automatica, eseguire un turno dell’agente silenzioso che scrive lo stato durevole su disco (ad esempio `memory/YYYY-MM-DD.md` nello spazio di lavoro dell’agente) in modo che la compattazione non possa cancellare contesto critico.

OpenClaw usa l’approccio di **flush pre-soglia**:

1. Monitora l’uso del contesto di sessione.
2. Quando supera una “soglia soft” (al di sotto della soglia di compattazione di Pi), esegui una direttiva silenziosa
   “write memory now” all’agente.
3. Usa `NO_REPLY` così l’utente non vede nulla.

Config (`agents.defaults.compaction.memoryFlush`):

* `enabled` (default: `true`)
* `softThresholdTokens` (default: `4000`)
* `prompt` (messaggio utente per il turno di flush)
* `systemPrompt` (prompt di sistema aggiuntivo aggiunto per il turno di flush)

Note:

* Il prompt/system prompt predefinito includono un’indicazione `NO_REPLY` per sopprimere l’invio della risposta.
* Il flush viene eseguito una volta per ciclo di compattazione (tracciato in `sessions.json`).
* Il flush viene eseguito solo per le sessioni Pi integrate (i backend CLI lo saltano).
* Il flush viene saltato quando lo spazio di lavoro della sessione è in sola lettura (`workspaceAccess: "ro"` o `"none"`).
* Vedi [Memory](/it/concepts/memory) per la struttura dei file dello spazio di lavoro e i pattern di scrittura.

Pi espone anche un hook `session_before_compact` nella extension API, ma la logica di flush di OpenClaw attualmente risiede lato Gateway.

<div id="troubleshooting-checklist">
  ## Checklist di troubleshooting
</div>

* Chiave di sessione errata? Inizia da [/concepts/session](/it/concepts/session) e conferma il `sessionKey` in `/status`.
* Incongruenza tra store e transcript? Conferma l&#39;host del Gateway e il percorso dello store da `openclaw status`.
* Spam di compattazione? Verifica:
  * finestra di contesto del modello (troppo piccola)
  * impostazioni di compattazione (`reserveTokens` troppo alto rispetto alla finestra del modello può causare una compattazione anticipata)
  * crescita dei risultati degli strumenti: abilita/regola il pruning della sessione
* Turni silenziosi che trapelano? Conferma che la risposta inizi con `NO_REPLY` (token esatto) e che stai usando una build che include la correzione per la soppressione dello streaming.