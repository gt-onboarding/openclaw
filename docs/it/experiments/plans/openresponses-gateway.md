---
title: OpenResponses Gateway
summary: "Piano: aggiungere l'endpoint /v1/responses di OpenResponses e dismettere in modo ordinato le chat completions"
owner: "openclaw"
status: "draft"
last_updated: "2026-01-19"
---

<div id="openresponses-gateway-integration-plan">
  # Piano di integrazione per OpenResponses Gateway
</div>

<div id="context">
  ## Contesto
</div>

OpenClaw Gateway attualmente espone un endpoint minimo compatibile con le Chat Completions di OpenAI in
`/v1/chat/completions` (vedi [OpenAI Chat Completions](/it/gateway/openai-http-api)).

Open Responses è uno standard di inferenza aperto basato sulla API OpenAI Responses. È progettato
per workflow basati su agenti e utilizza input basati su elementi e eventi di streaming semantici. La specifica OpenResponses
definisce `/v1/responses`, non `/v1/chat/completions`.

<div id="goals">
  ## Obiettivi
</div>

* Aggiungere un endpoint `/v1/responses` che aderisca alla semantica di OpenResponses.
* Mantenere Chat Completions come livello di compatibilità, facile da disabilitare e da rimuovere in seguito.
* Standardizzare la validazione e il parsing con schemi isolati e riutilizzabili.

<div id="non-goals">
  ## Non-obiettivi
</div>

* Piena parità di funzionalità con OpenResponses nella prima fase (immagini, file, strumenti ospitati).
* Sostituire la logica di esecuzione interna degli agenti o l&#39;orchestrazione degli strumenti.
* Modificare il comportamento esistente di `/v1/chat/completions` durante la prima fase.

<div id="research-summary">
  ## Sintesi della ricerca
</div>

Fonti: OpenResponses OpenAPI, sito delle specifiche OpenResponses e articolo del blog di Hugging Face.

Punti chiave estratti:

* `POST /v1/responses` accetta campi `CreateResponseBody` come `model`, `input` (stringa o
  `ItemParam[]`), `instructions`, `tools`, `tool_choice`, `stream`, `max_output_tokens` e
  `max_tool_calls`.
* `ItemParam` è un&#39;unione discriminata di:
  * elementi `message` con ruoli `system`, `developer`, `user`, `assistant`
  * `function_call` e `function_call_output`
  * `reasoning`
  * `item_reference`
* Le risposte andate a buon fine restituiscono una `ResponseResource` con `object: "response"`, `status` e
  elementi di `output`.
* Lo streaming utilizza eventi semantici come:
  * `response.created`, `response.in_progress`, `response.completed`, `response.failed`
  * `response.output_item.added`, `response.output_item.done`
  * `response.content_part.added`, `response.content_part.done`
  * `response.output_text.delta`, `response.output_text.done`
* Le specifiche richiedono:
  * `Content-Type: text/event-stream`
  * `event:` deve corrispondere al campo JSON `type`
  * l&#39;evento terminale deve essere letteralmente `[DONE]`
* Gli elementi `reasoning` possono esporre `content`, `encrypted_content` e `summary`.
* Gli esempi di Hugging Face includono `OpenResponses-Version: latest` nelle richieste (header opzionale).

<div id="proposed-architecture">
  ## Architettura proposta
</div>

* Aggiungi `src/gateway/open-responses.schema.ts` contenente solo gli schemi Zod (nessun import dal Gateway).
* Aggiungi `src/gateway/openresponses-http.ts` (o `open-responses-http.ts`) per `/v1/responses`.
* Mantieni `src/gateway/openai-http.ts` intatto come adapter di compatibilità legacy.
* Aggiungi la configurazione `gateway.http.endpoints.responses.enabled` (valore predefinito `false`).
* Mantieni `gateway.http.endpoints.chatCompletions.enabled` indipendente; consenti che entrambi gli endpoint
  possano essere attivati/disattivati separatamente.
* Emetti un avviso in fase di avvio quando Chat Completions è abilitato, per indicarne lo stato legacy.

<div id="deprecation-path-for-chat-completions">
  ## Percorso di deprecazione per Chat Completions
</div>

* Mantieni limiti rigorosi tra i moduli: nessun tipo di schema condiviso tra Responses e Chat Completions.
* Rendi Chat Completions una funzionalità attivabile esplicitamente tramite configurazione, in modo che possa essere disabilitata senza modifiche al codice.
* Aggiorna la documentazione per etichettare Chat Completions come legacy una volta che `/v1/responses` è stabile.
* Passaggio futuro opzionale: mappa le richieste di Chat Completions al gestore Responses per un percorso di rimozione più semplice.

<div id="phase-1-support-subset">
  ## Sottoinsieme di funzionalità supportate nella Fase 1
</div>

* Accetta `input` come stringa o `ItemParam[]` con i ruoli dei messaggi e `function_call_output`.
* Estrae i messaggi di sistema e dello sviluppatore in `extraSystemPrompt`.
* Usa il più recente `user` o `function_call_output` come messaggio corrente per le esecuzioni dell&#39;agente.
* Rifiuta parti di contenuto non supportate (immagine/file) con `invalid_request_error`.
* Restituisce un singolo messaggio assistant con contenuto `output_text`.
* Restituisce `usage` con valori azzerati finché il conteggio dei token non sarà implementato.

<div id="validation-strategy-no-sdk">
  ## Strategia di validazione (senza SDK)
</div>

* Implementa gli schemi Zod per il sottoinsieme supportato di:
  * `CreateResponseBody`
  * `ItemParam` + unioni delle parti del contenuto del messaggio
  * `ResponseResource`
  * Strutture degli eventi di streaming utilizzate dal Gateway
* Mantieni gli schemi in un singolo modulo isolato per evitare disallineamenti e consentire una futura generazione di codice.

<div id="streaming-implementation-phase-1">
  ## Implementazione dello streaming (Fase 1)
</div>

* Righe SSE contenenti sia `event:` che `data:`.
* Sequenza richiesta (minimo indispensabile):
  * `response.created`
  * `response.output_item.added`
  * `response.content_part.added`
  * `response.output_text.delta` (ripetere secondo necessità)
  * `response.output_text.done`
  * `response.content_part.done`
  * `response.completed`
  * `[DONE]`

<div id="tests-and-verification-plan">
  ## Piano di test e verifica
</div>

* Aggiungi copertura e2e per `/v1/responses`:
  * Autenticazione richiesta
  * Struttura della risposta non in streaming
  * Ordine degli eventi in streaming e `[DONE]`
  * Instradamento delle sessioni tramite header e `user`
* Lascia `src/gateway/openai-http.e2e.test.ts` invariato.
* Verifica manuale: esegui curl su `/v1/responses` con `stream: true` e controlla l&#39;ordine degli eventi e l&#39;evento `[DONE]` finale.

<div id="doc-updates-follow-up">
  ## Aggiornamenti alla documentazione (follow-up)
</div>

* Aggiungere una nuova pagina di documentazione per l&#39;uso di `/v1/responses` e relativi esempi.
* Aggiornare `/gateway/openai-http-api` con una nota di obsolescenza e un rimando a `/v1/responses`.