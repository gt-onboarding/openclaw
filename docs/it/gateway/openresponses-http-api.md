---
title: API HTTP OpenResponses
summary: "Esporre dal Gateway un endpoint HTTP /v1/responses compatibile con OpenResponses"
read_when:
  - Stai integrando client che utilizzano l'API OpenResponses
  - Vuoi input basati su item, chiamate di tool dal client o eventi SSE
---

<div id="openresponses-api-http">
  # API OpenResponses (HTTP)
</div>

Il Gateway di OpenClaw può servire un endpoint `POST /v1/responses` compatibile con OpenResponses.

Questo endpoint è **disabilitato per impostazione predefinita**. Abilitalo prima nella configurazione.

- `POST /v1/responses`
- Stessa porta del Gateway (multiplex WS + HTTP): `http://<gateway-host>:<port>/v1/responses`

Dietro le quinte, le richieste vengono eseguite come una normale esecuzione di un agente del Gateway (stesso percorso di codice di
`openclaw agent`), quindi instradamento/autorizzazioni/configurazione sono allineati al tuo Gateway.

<div id="authentication">
  ## Autenticazione
</div>

Usa la configurazione di autenticazione del Gateway. Invia un token Bearer:

- `Authorization: Bearer <token>`

Note:

- Quando `gateway.auth.mode="token"`, usa `gateway.auth.token` (o `OPENCLAW_GATEWAY_TOKEN`).
- Quando `gateway.auth.mode="password"`, usa `gateway.auth.password` (o `OPENCLAW_GATEWAY_PASSWORD`).

<div id="choosing-an-agent">
  ## Scelta di un agente
</div>

Nessuna intestazione personalizzata necessaria: includi l'id dell'agente nel campo `model` di OpenResponses:

- `model: "openclaw:<agentId>"` (esempio: `"openclaw:main"`, `"openclaw:beta"`)
- `model: "agent:<agentId>"` (alias)

Oppure seleziona uno specifico agente OpenClaw tramite intestazione:

- `x-openclaw-agent-id: <agentId>` (predefinito: `main`)

Avanzato:

- `x-openclaw-session-key: <sessionKey>` per avere il pieno controllo dell'instradamento della sessione.

<div id="enabling-the-endpoint">
  ## Abilitare l&#39;endpoint
</div>

Imposta `gateway.http.endpoints.responses.enabled` su `true`:

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: true }
      }
    }
  }
}
```


<div id="disabling-the-endpoint">
  ## Disattivare l&#39;endpoint
</div>

Imposta `gateway.http.endpoints.responses.enabled` su `false`:

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: false }
      }
    }
  }
}
```


<div id="session-behavior">
  ## Comportamento della sessione
</div>

Per impostazione predefinita, l'endpoint è **stateless per richiesta** (a ogni chiamata viene generata una nuova chiave di sessione).

Se la richiesta include una stringa `user` OpenResponses, il Gateway ne ricava una chiave di sessione stabile, così le chiamate ripetute possono condividere una sessione dell'agente.

<div id="request-shape-supported">
  ## Formato della richiesta (supportato)
</div>

La richiesta segue l'API OpenResponses con input basato su elementi (item). Supporto attuale:

- `input`: stringa o array di oggetti item.
- `instructions`: combinate nel prompt di sistema.
- `tools`: definizioni degli strumenti client (function tools).
- `tool_choice`: filtra o richiede strumenti client.
- `stream`: abilita lo streaming SSE.
- `max_output_tokens`: limite di output best-effort (dipende dal provider).
- `user`: instradamento stabile della sessione.

Accettati ma **attualmente ignorati**:

- `max_tool_calls`
- `reasoning`
- `metadata`
- `store`
- `previous_response_id`
- `truncation`

<div id="items-input">
  ## Elementi (input)
</div>

<div id="message">
  ### `message`
</div>

Ruoli: `system`, `developer`, `user`, `assistant`.

- `system` e `developer` vengono aggiunti al prompt di sistema.
- L'elemento `user` o `function_call_output` più recente diventa il “messaggio corrente”.
- I messaggi precedenti di `user`/`assistant` sono inclusi come cronologia della conversazione per contesto.

<div id="function_call_output-turn-based-tools">
  ### `function_call_output` (strumenti basati su turni)
</div>

Invia i risultati degli strumenti al modello:

```json
{
  "type": "function_call_output",
  "call_id": "call_123",
  "output": "{\"temperature\": \"72F\"}"
}
```


<div id="reasoning-and-item_reference">
  ### `reasoning` e `item_reference`
</div>

Sono accettati per compatibilità con lo schema, ma ignorati durante la generazione del prompt.

<div id="tools-client-side-function-tools">
  ## Strumenti (strumenti funzione lato client)
</div>

Fornisci gli strumenti con `tools: [{ type: "function", function: { name, description?, parameters? } }]`.

Se l'agente decide di chiamare uno strumento, la risposta restituisce un elemento di output `function_call`.
Quindi invia una richiesta successiva con `function_call_output` per continuare il turno.

<div id="images-input_image">
  ## Immagini (`input_image`)
</div>

Supporta fonti in base64 o URL:

```json
{
  "type": "input_image",
  "source": { "type": "url", "url": "https://example.com/image.png" }
}
```

Tipi MIME consentiti (correnti): `image/jpeg`, `image/png`, `image/gif`, `image/webp`.
Dimensione massima del file (corrente): 10MB.


<div id="files-input_file">
  ## File (`input_file`)
</div>

Supporta sorgenti codificate in base64 o URL:

```json
{
  "type": "input_file",
  "source": {
    "type": "base64",
    "media_type": "text/plain",
    "data": "SGVsbG8gV29ybGQh",
    "filename": "hello.txt"
  }
}
```

Tipi MIME consentiti (attuali): `text/plain`, `text/markdown`, `text/html`, `text/csv`,
`application/json`, `application/pdf`.

Dimensione massima (attuale): 5MB.

Comportamento attuale:

* Il contenuto del file viene decodificato e aggiunto al **system prompt**, non al messaggio utente,
  quindi rimane effimero (non viene mantenuto nella cronologia della sessione).
* I PDF vengono analizzati per estrarne il testo. Se viene trovato poco testo, le prime pagine
  vengono rasterizzate in immagini e passate al modello.

L&#39;analisi dei PDF utilizza la build legacy `pdfjs-dist` compatibile con Node (senza worker). La build
moderna di PDF.js si aspetta worker del browser e globali DOM, quindi non viene usata nel Gateway.

Valori predefiniti per il fetch via URL:

* `files.allowUrl`: `true`
* `images.allowUrl`: `true`
* Le richieste sono protette (risoluzione DNS, blocco degli indirizzi IP privati, limiti sui redirect, timeout).


<div id="file-image-limits-config">
  ## Limiti di file e immagini (config)
</div>

Le impostazioni predefinite possono essere modificate in `gateway.http.endpoints.responses`:

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: {
          enabled: true,
          maxBodyBytes: 20000000,
          files: {
            allowUrl: true,
            allowedMimes: ["text/plain", "text/markdown", "text/html", "text/csv", "application/json", "application/pdf"],
            maxBytes: 5242880,
            maxChars: 200000,
            maxRedirects: 3,
            timeoutMs: 10000,
            pdf: {
              maxPages: 4,
              maxPixels: 4000000,
              minTextChars: 200
            }
          },
          images: {
            allowUrl: true,
            allowedMimes: ["image/jpeg", "image/png", "image/gif", "image/webp"],
            maxBytes: 10485760,
            maxRedirects: 3,
            timeoutMs: 10000
          }
        }
      }
    }
  }
}
```

Valori predefiniti se omessi:

* `maxBodyBytes`: 20MB
* `files.maxBytes`: 5MB
* `files.maxChars`: 200k
* `files.maxRedirects`: 3
* `files.timeoutMs`: 10s
* `files.pdf.maxPages`: 4
* `files.pdf.maxPixels`: 4,000,000
* `files.pdf.minTextChars`: 200
* `images.maxBytes`: 10MB
* `images.maxRedirects`: 3
* `images.timeoutMs`: 10s


<div id="streaming-sse">
  ## Streaming (SSE)
</div>

Imposta `stream: true` per ricevere Server-Sent Events (SSE):

- `Content-Type: text/event-stream`
- Ogni riga dell'evento ha la forma `event: <type>` e `data: <json>`
- Lo stream termina con `data: [DONE]`

Tipi di eventi attualmente emessi:

- `response.created`
- `response.in_progress`
- `response.output_item.added`
- `response.content_part.added`
- `response.output_text.delta`
- `response.output_text.done`
- `response.content_part.done`
- `response.output_item.done`
- `response.completed`
- `response.failed` (in caso di errore)

<div id="usage">
  ## Utilizzo
</div>

`usage` viene compilato quando il provider sottostante riporta i conteggi dei token.

<div id="errors">
  ## Errori
</div>

Gli errori sono rappresentati tramite un oggetto JSON come:

```json
{ "error": { "message": "...", "type": "invalid_request_error" } }
```

Casi comuni:

* `401` autenticazione mancante o non valida
* `400` body della richiesta non valido
* `405` metodo HTTP non consentito


<div id="examples">
  ## Esempi
</div>

Senza streaming:

```bash
curl -sS http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "input": "hi"
  }'
```

Streaming:

```bash
curl -N http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "input": "hi"
  }'
```
