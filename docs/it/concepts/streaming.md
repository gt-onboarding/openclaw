---
title: Streaming
summary: "Comportamento di streaming e chunking (risposte a blocchi, streaming delle bozze, limiti)"
read_when:
  - Spiegare il funzionamento dello streaming e del chunking sui canali
  - Modificare il comportamento dello streaming a blocchi o del chunking dei canali
  - Diagnosticare problemi di risposte a blocchi duplicate/anticipate o di streaming delle bozze
---

<div id="streaming-chunking">
  # Streaming + chunking
</div>

OpenClaw ha due livelli separati di “streaming”:

- **Block streaming (canali):** emette **blocchi** completi man mano che l'assistente scrive. Questi sono normali messaggi di canale (non delta di token).
- **Token-ish streaming (solo Telegram):** aggiorna una **bolla di messaggio in bozza** con testo parziale durante la generazione; il messaggio finale viene inviato al termine.

Al momento **non esiste un vero streaming a livello di token** verso i messaggi dei canali esterni. Lo streaming delle bozze in Telegram è l'unico punto in cui è disponibile lo streaming parziale.

<div id="block-streaming-channel-messages">
  ## Streaming a blocchi (messaggi del canale)
</div>

Lo streaming a blocchi invia l&#39;output dell&#39;assistente in blocchi relativamente grandi man mano che diventa disponibile.

```
Model output
  └─ text_delta/events
       ├─ (blockStreamingBreak=text_end)
       │    └─ chunker emits blocks as buffer grows
       └─ (blockStreamingBreak=message_end)
            └─ chunker flushes at message_end
                   └─ channel send (block replies)
```

Legenda:

* `text_delta/events`: eventi di streaming del modello (possono essere sporadici per i modelli non in streaming).
* `chunker`: `EmbeddedBlockChunker` che applica limiti min/max + preferenza di interruzione.
* `channel send`: messaggi effettivamente in uscita (risposte a blocchi).

**Controlli:**

* `agents.defaults.blockStreamingDefault`: `"on"`/`"off"` (predefinito off).
* Override di canale: `*.blockStreaming` (e varianti per account) per forzare `"on"`/`"off"` per canale.
* `agents.defaults.blockStreamingBreak`: `"text_end"` o `"message_end"`.
* `agents.defaults.blockStreamingChunk`: `{ minChars, maxChars, breakPreference? }`.
* `agents.defaults.blockStreamingCoalesce`: `{ minChars?, maxChars?, idleMs? }` (unisce i blocchi in streaming prima dell&#39;invio).
* Limite rigido di canale: `*.textChunkLimit` (ad es. `channels.whatsapp.textChunkLimit`).
* Modalità di chunk del canale: `*.chunkMode` (`length` predefinito, `newline` divide sulle righe vuote (confini di paragrafo) prima del chunking per lunghezza).
* Limite soft di Discord: `channels.discord.maxLinesPerMessage` (predefinito 17) divide le risposte molto lunghe per evitare il clipping della UI.

**Semantica dei confini:**

* `text_end`: invia in streaming i blocchi non appena il chunker li emette; svuota a ogni `text_end`.
* `message_end`: aspetta che il messaggio dell&#39;assistente sia terminato, poi svuota l&#39;output in buffer.

`message_end` utilizza comunque il chunker se il testo nel buffer supera `maxChars`, quindi può emettere più chunk alla fine.


<div id="chunking-algorithm-lowhigh-bounds">
  ## Algoritmo di chunking (limiti inferiore/superiore)
</div>

Il chunking dei blocchi è implementato da `EmbeddedBlockChunker`:

- **Limite inferiore:** non emettere finché il buffer non raggiunge `minChars` (a meno che non venga forzato).
- **Limite superiore:** preferisci le suddivisioni prima di `maxChars`; se forzato, suddividi a `maxChars`.
- **Preferenza di interruzione:** `paragraph` → `newline` → `sentence` → `whitespace` → interruzione forzata.
- **Code fences:** non suddividere mai all'interno dei fence; quando devi forzare a `maxChars`, chiudi e riapri il fence per mantenere valido il Markdown.

`maxChars` è limitato al valore di `textChunkLimit` del canale, quindi non puoi superare i limiti per canale.

<div id="coalescing-merge-streamed-blocks">
  ## Coalescing (unione dei blocchi in streaming)
</div>

Quando il block streaming è abilitato, OpenClaw può **unire blocchi consecutivi**
prima di inviarli. Questo riduce lo “spam di righe singole” continuando comunque a
fornire output progressivo.

- Il coalescing attende **intervalli di inattività** (`idleMs`) prima di eseguire il flush.
- I buffer sono limitati da `maxChars` ed eseguono il flush se lo superano.
- `minChars` impedisce l'invio di frammenti troppo piccoli finché non si accumula
  testo sufficiente (il flush finale invia sempre il testo rimanente).
- Il joiner è derivato da `blockStreamingChunk.breakPreference`
  (`paragraph` → `\n\n`, `newline` → `\n`, `sentence` → spazio).
- Le sovrascritture specifiche per canale sono disponibili tramite `*.blockStreamingCoalesce` (incluse le configurazioni per singolo account).
- Il valore predefinito di `minChars` per il coalescing viene aumentato a 1500 per Signal/Slack/Discord, a meno che non venga sovrascritto.

<div id="human-like-pacing-between-blocks">
  ## Ritmo simile a quello umano tra i blocchi
</div>

Quando lo streaming a blocchi è abilitato, puoi aggiungere una **pausa casuale**
tra i blocchi di risposta (dopo il primo blocco). Questo rende le risposte con più bolle più naturali.

- Configurazione: `agents.defaults.humanDelay` (puoi sovrascrivere per singolo agente tramite `agents.list[].humanDelay`).
- Modalità: `off` (predefinita), `natural` (800–2500ms), `custom` (`minMs`/`maxMs`).
- Si applica solo alle **risposte a blocchi**, non alle risposte finali o ai riepiloghi degli strumenti.

<div id="stream-chunks-or-everything">
  ## "Esegui lo streaming dei chunk o di tutto"
</div>

Questo corrisponde a:

- **Esegui lo streaming dei chunk:** `blockStreamingDefault: "on"` + `blockStreamingBreak: "text_end"` (emetti man mano). I canali diversi da Telegram richiedono anche `*.blockStreaming: true`.
- **Esegui lo streaming di tutto alla fine:** `blockStreamingBreak: "message_end"` (svuota il buffer una volta sola, eventualmente in più chunk se molto lungo).
- **Nessuno streaming a blocchi:** `blockStreamingDefault: "off"` (solo risposta finale).

**Nota sul canale:** Per i canali diversi da Telegram, lo streaming a blocchi è **disattivato a meno che**
`*.blockStreaming` non sia impostato esplicitamente su `true`. Telegram può eseguire lo streaming delle bozze
(`channels.telegram.streamMode`) senza risposte a blocchi.

Promemoria sulla posizione della configurazione: i valori predefiniti `blockStreaming*` si trovano sotto
`agents.defaults`, non nella configurazione radice.

<div id="telegram-draft-streaming-token-ish">
  ## Streaming delle bozze Telegram (a livello di token)
</div>

Telegram è l&#39;unico canale con streaming delle bozze:

* Usa la Bot API `sendMessageDraft` **nelle chat private con argomenti**.
* `channels.telegram.streamMode: "partial" | "block" | "off"`.
  * `partial`: aggiornamenti della bozza con il testo più recente dello stream.
  * `block`: aggiornamenti della bozza in blocchi segmentati (stesse regole del chunker).
  * `off`: nessuno streaming delle bozze.
* Configurazione dei chunk di bozza (solo per `streamMode: "block"`): `channels.telegram.draftChunk` (valori predefiniti: `minChars: 200`, `maxChars: 800`).
* Lo streaming delle bozze è separato dallo streaming a blocchi; le risposte a blocchi sono disattivate per impostazione predefinita e abilitate solo con `*.blockStreaming: true` sui canali non-Telegram.
* La risposta finale è comunque un messaggio normale.
* `/reasoning stream` scrive il ragionamento nella bolla di bozza (solo Telegram).

Quando lo streaming delle bozze è attivo, OpenClaw disabilita lo streaming a blocchi per quella risposta per evitare una duplicazione dello streaming.

```
Telegram (privato + argomenti)
  └─ sendMessageDraft (bolla bozza)
       ├─ streamMode=partial → aggiorna l'ultimo testo
       └─ streamMode=block   → il chunker aggiorna la bozza
  └─ risposta finale → messaggio normale
```

Legend:

* `sendMessageDraft`: fumetto di bozza su Telegram (non un vero messaggio).
* `final reply`: normale invio di un messaggio Telegram.
