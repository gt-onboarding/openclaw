---
title: Messaggi
summary: "Flusso dei messaggi, sessioni, messa in coda e visibilità del reasoning"
read_when:
  - Spiegare come i messaggi in ingresso vengono trasformati in risposte
  - Chiarire le sessioni, le modalità di messa in coda o il comportamento dello streaming
  - Documentare la visibilità del reasoning e le relative implicazioni di utilizzo
---

<div id="messages">
  # Messaggi
</div>

Questa pagina illustra in che modo OpenClaw gestisce i messaggi in ingresso, le sessioni, la messa in coda,
lo streaming e la visibilità del ragionamento.

<div id="message-flow-high-level">
  ## Flusso dei messaggi (a livello alto)
</div>

```
Inbound message
  -> routing/bindings -> session key
  -> queue (if a run is active)
  -> agent run (streaming + tools)
  -> outbound replies (channel limits + chunking)
```

Le impostazioni principali si trovano nella configurazione:

* `messages.*` per i prefissi, l’accodamento e il comportamento dei gruppi.
* `agents.defaults.*` per i valori predefiniti di block streaming e chunking.
* Override di canale (`channels.whatsapp.*`, `channels.telegram.*`, ecc.) per limiti e toggle dello streaming.

Vedi [Configurazione](/it/gateway/configuration) per lo schema completo.

<div id="inbound-dedupe">
  ## Deduplicazione in ingresso
</div>

I canali possono recapitare di nuovo lo stesso messaggio dopo una riconnessione. OpenClaw mantiene
una cache temporanea indicizzata da canale/account/peer/sessione/id messaggio, così che le consegne
duplicate non avviino un&#39;altra esecuzione dell&#39;agente.

<div id="inbound-debouncing">
  ## Debounce dei messaggi in ingresso
</div>

Messaggi consecutivi rapidi dallo **stesso mittente** possono essere raggruppati in un unico
turno dell&#39;agente tramite `messages.inbound`. Il debounce è applicato per canale + conversazione
e utilizza il messaggio più recente per il threading delle risposte/ID.

Configurazione (valore predefinito globale + sovrascritture per canale):

```json5
{
  messages: {
    inbound: {
      debounceMs: 2000,
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500
      }
    }
  }
}
```

Note:

* Il debounce si applica solo ai messaggi di **testo**; i contenuti multimediali/gli allegati vengono inviati immediatamente.
* I comandi di controllo ignorano il debounce, così da rimanere comandi separati.

<div id="sessions-and-devices">
  ## Sessioni e dispositivi
</div>

Le sessioni appartengono al Gateway, non ai client.

* Le chat dirette vengono ricondotte alla chiave di sessione principale dell&#39;agente.
* I gruppi/canali hanno le proprie chiavi di sessione.
* Il session store e le trascrizioni risiedono sull&#39;host del Gateway.

Più dispositivi/canali possono essere associati alla stessa sessione, ma la cronologia non è
sincronizzata completamente per ogni client. Si consiglia di usare un dispositivo primario
per le conversazioni lunghe, per evitare contesti divergenti. La Control UI e la TUI
mostrano sempre la trascrizione della sessione mantenuta dal Gateway, quindi sono la
fonte di verità.

Dettagli: [Gestione delle sessioni](/it/concepts/session).

<div id="inbound-bodies-and-history-context">
  ## Corpi in ingresso e contesto della cronologia
</div>

OpenClaw separa il **prompt body** dal **command body**:

* `Body`: testo del prompt inviato all&#39;agente. Può includere envelope del canale e
  wrapper di cronologia opzionali.
* `CommandBody`: testo grezzo dell&#39;utente per l&#39;analisi di direttive/comandi.
* `RawBody`: alias legacy di `CommandBody` (mantenuto per compatibilità).

Quando un canale fornisce la cronologia, usa un wrapper condiviso:

* `[Messaggi della chat dal tuo ultimo messaggio di risposta - per contesto]`
* `[Messaggio corrente - rispondi a questo]`

Per le **chat non dirette** (gruppi/canali/stanze), il **corpo del messaggio corrente** è prefissato
con l&#39;etichetta del mittente (stesso stile usato per le voci di cronologia). Questo mantiene coerenti i messaggi
in tempo reale e quelli in coda/di cronologia nel prompt dell&#39;agente.

I buffer di cronologia sono **solo pending**: includono i messaggi di gruppo che *non*
hanno attivato un&#39;esecuzione (ad esempio, messaggi che richiedono una menzione) ed **escludono** i messaggi
già presenti nella trascrizione della sessione.

La rimozione delle direttive si applica solo alla sezione del **messaggio corrente**, in modo che la cronologia
resti intatta. I canali che wrappano la cronologia dovrebbero impostare `CommandBody` (o
`RawBody`) al testo originale del messaggio e mantenere `Body` come prompt combinato.
I buffer di cronologia sono configurabili tramite `messages.groupChat.historyLimit` (valore predefinito
globale) e override per singolo canale come `channels.slack.historyLimit` o
`channels.telegram.accounts.<id>.historyLimit` (imposta `0` per disabilitare).

<div id="queueing-and-followups">
  ## Accodamento e follow-up
</div>

Se è già in corso un&#39;esecuzione, i messaggi in ingresso possono essere messi in coda, instradati nell&#39;esecuzione corrente oppure raccolti per un turno di follow-up.

* Configura tramite `messages.queue` (e `messages.queue.byChannel`).
* Modalità: `interrupt`, `steer`, `followup`, `collect`, più varianti per il backlog.

Dettagli: [Accodamento](/it/concepts/queue).

<div id="streaming-chunking-and-batching">
  ## Streaming, suddivisione in blocchi e batching
</div>

Lo streaming a blocchi invia risposte parziali man mano che il modello produce blocchi di testo.
La suddivisione in blocchi rispetta i limiti di testo del canale ed evita di spezzare il codice delimitato da fence.

Impostazioni principali:

* `agents.defaults.blockStreamingDefault` (`on|off`, valore predefinito off)
* `agents.defaults.blockStreamingBreak` (`text_end|message_end`)
* `agents.defaults.blockStreamingChunk` (`minChars|maxChars|breakPreference`)
* `agents.defaults.blockStreamingCoalesce` (batching basato sull&#39;inattività)
* `agents.defaults.humanDelay` (pausa simile a quella umana tra le risposte a blocchi)
* Override di canale: `*.blockStreaming` e `*.blockStreamingCoalesce` (i canali diversi da Telegram richiedono esplicitamente `*.blockStreaming: true`)

Dettagli: [Streaming + chunking](/it/concepts/streaming).

<div id="reasoning-visibility-and-tokens">
  ## Visibilità del ragionamento e token
</div>

OpenClaw può esporre o nascondere il ragionamento del modello:

* `/reasoning on|off|stream` ne controlla la visibilità.
* Il contenuto del ragionamento viene comunque conteggiato nel numero di token quando è prodotto dal modello.
* Telegram supporta lo streaming del ragionamento nella bolla del messaggio in bozza.

Dettagli: [Direttive per thinking e ragionamento](/it/tools/thinking) e [Uso dei token](/it/token-use).

<div id="prefixes-threading-and-replies">
  ## Prefissi, threading e risposte
</div>

La formattazione dei messaggi in uscita è gestita centralmente in `messages`:

* `messages.responsePrefix` (prefisso in uscita) e `channels.whatsapp.messagePrefix` (prefisso in ingresso per WhatsApp)
* Threading delle risposte tramite `replyToMode` e impostazioni predefinite specifiche per ciascun canale

Dettagli: [Configurazione](/it/gateway/configuration#messages) e documentazione dei canali.