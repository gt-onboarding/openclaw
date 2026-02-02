---
title: Igiene della trascrizione
summary: "Riferimento: regole di sanitizzazione e correzione della trascrizione specifiche per provider"
read_when:
  - Stai eseguendo il debug di rifiuti di richieste da parte dei provider legati alla struttura della trascrizione
  - Stai modificando la sanitizzazione della trascrizione o la logica di correzione delle chiamate agli strumenti
  - Stai indagando su disallineamenti degli ID delle chiamate agli strumenti tra diversi provider
---

<div id="transcript-hygiene-provider-fixups">
  # Igiene delle trascrizioni (Correzioni lato provider)
</div>

Questo documento descrive le **correzioni specifiche per provider** applicate alle trascrizioni prima di un&#39;esecuzione
(per costruire il contesto del modello). Si tratta di aggiustamenti **in memoria** utilizzati per soddisfare i requisiti
rigorosi dei provider. Queste correzioni **non** riscrivono il file di trascrizione JSONL archiviato su disco.

L&#39;ambito include:

* Sanitizzazione degli ID delle chiamate agli strumenti
* Ripristino dell&#39;abbinamento tra chiamate e risultati degli strumenti
* Validazione / ordinamento dei turni di conversazione
* Pulizia delle firme dei ragionamenti
* Sanitizzazione dei payload delle immagini

Per i dettagli sull&#39;archiviazione delle trascrizioni, vedi:

* [/reference/session-management-compaction](/it/reference/session-management-compaction)

***

<div id="where-this-runs">
  ## Dove viene eseguito
</div>

Tutta la pulizia dei transcript è centralizzata nel runner integrato:

* Selezione della policy: `src/agents/transcript-policy.ts`
* Applicazione di sanitizzazione/riparazione: `sanitizeSessionHistory` in `src/agents/pi-embedded-runner/google.ts`

La policy usa `provider`, `modelApi` e `modelId` per decidere cosa applicare.

***

<div id="global-rule-image-sanitization">
  ## Regola globale: sanitizzazione delle immagini
</div>

I payload delle immagini vengono sempre sanitizzati per evitare che il provider li rifiuti a causa dei limiti di dimensione
(ridimensionamento/ricompressione delle immagini base64 troppo grandi).

Implementazione:

* `sanitizeSessionMessagesImages` in `src/agents/pi-embedded-helpers/images.ts`
* `sanitizeContentBlocksImages` in `src/agents/tool-images.ts`

***

<div id="provider-matrix-current-behavior">
  ## Matrice dei provider (comportamento attuale)
</div>

**OpenAI / OpenAI Codex**

* Solo sanitizzazione delle immagini.
* Al cambio di modello verso OpenAI Responses/Codex, elimina le firme di ragionamento orfane (elementi di ragionamento isolati senza un blocco di contenuto successivo).
* Nessuna sanitizzazione degli ID di chiamata agli strumenti.
* Nessuna correzione dell&#39;abbinamento dei risultati degli strumenti.
* Nessuna validazione o riordinamento dei turni.
* Nessuna generazione di risultati sintetici degli strumenti.
* Nessuna rimozione delle firme di pensiero.

**Google (Generative AI / Gemini CLI / Antigravity)**

* Sanitizzazione degli ID di chiamata agli strumenti: alfanumerico rigoroso.
* Correzione dell&#39;abbinamento dei risultati degli strumenti e generazione di risultati sintetici degli strumenti.
* Validazione dei turni (alternanza dei turni in stile Gemini).
* Correzione dell&#39;ordinamento dei turni di Google (aggiunge un piccolo bootstrap utente all’inizio se la cronologia inizia con l&#39;assistente).
* Antigravity Claude: normalizza le firme di pensiero; elimina i blocchi di pensiero non firmati.

**Anthropic / Minimax (compatibile Anthropic)**

* Correzione dell&#39;abbinamento dei risultati degli strumenti e generazione di risultati sintetici degli strumenti.
* Validazione dei turni (unisce i turni utente consecutivi per soddisfare l&#39;alternanza rigorosa).

**Mistral (incluso il rilevamento basato su model-id)**

* Sanitizzazione degli ID di chiamata agli strumenti: strict9 (alfanumerico, lunghezza 9).

**OpenRouter Gemini**

* Pulizia delle firme di pensiero: rimuove i valori `thought_signature` non base64 (mantiene quelli in base64).

**Tutto il resto**

* Solo sanitizzazione delle immagini.

***

<div id="historical-behavior-pre-2026122">
  ## Comportamento storico (pre-2026.1.22)
</div>

Prima della release 2026.1.22, OpenClaw applicava più livelli di pulizia del transcript:

* Un&#39;estensione **transcript-sanitize** veniva eseguita a ogni build del contesto e poteva:
  * Riparare l&#39;abbinamento tra uso e risultato degli strumenti.
  * Sanitizzare gli ID delle chiamate agli strumenti (inclusa una modalità non rigorosa che preservava `_`/`-`).
* Il runner eseguiva anche una sanitizzazione specifica per provider, duplicando il lavoro.
* Ulteriori modifiche avvenivano al di fuori della policy del provider, tra cui:
  * Rimozione dei tag `<final>` dal testo dell&#39;assistente prima del salvataggio.
  * Eliminazione dei turni di errore vuoti dell&#39;assistente.
  * Troncamento del contenuto dell&#39;assistente dopo le chiamate agli strumenti.

Questa complessità causava regressioni tra provider (in particolare l&#39;abbinamento
`call_id|fc_id` per `openai-responses`). La pulizia del 2026.1.22 ha rimosso l&#39;estensione, ha centralizzato
la logica nel runner e ha reso OpenAI **no-touch** (nessuna modifica) oltre alla sanitizzazione delle immagini.