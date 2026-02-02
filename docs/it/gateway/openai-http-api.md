---
title: OpenAI HTTP API
summary: "Esporre un endpoint HTTP /v1/chat/completions compatibile con OpenAI dal Gateway"
read_when:
  - Quando integri strumenti che richiedono OpenAI Chat Completions
---

<div id="openai-chat-completions-http">
  # OpenAI Chat Completions (HTTP)
</div>

Il Gateway di OpenClaw può servire un endpoint Chat Completions compatibile con OpenAI.

Questo endpoint è **disabilitato per impostazione predefinita**. Abilitalo prima nella configurazione.

- `POST /v1/chat/completions`
- Stessa porta del Gateway (multiplex WS + HTTP): `http://<gateway-host>:<port>/v1/chat/completions`

Internamente, le richieste vengono eseguite come una normale esecuzione di un agente del Gateway (stesso percorso di codice di `openclaw agent`), quindi instradamento/permessi/configurazione sono allineati a quelli del tuo Gateway.

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

Non sono necessarie intestazioni personalizzate: codifica l'ID dell'agente nel campo `model` di OpenAI:

- `model: "openclaw:&lt;agentId&gt;"` (esempio: `"openclaw:main"`, `"openclaw:beta"`)
- `model: "agent:&lt;agentId&gt;"` (alias)

Oppure indirizza uno specifico agente OpenClaw tramite intestazione:

- `x-openclaw-agent-id: &lt;agentId&gt;` (predefinito: `main`)

Avanzato:

- `x-openclaw-session-key: &lt;sessionKey&gt;` per avere il pieno controllo dell'instradamento della sessione.

<div id="enabling-the-endpoint">
  ## Abilitare l&#39;endpoint
</div>

Imposta `gateway.http.endpoints.chatCompletions.enabled` a `true`:

```json5
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: true }
      }
    }
  }
}
```


<div id="disabling-the-endpoint">
  ## Disabilitare l&#39;endpoint
</div>

Imposta `gateway.http.endpoints.chatCompletions.enabled` su `false`:

```json5
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: false }
      }
    }
  }
}
```


<div id="session-behavior">
  ## Comportamento della sessione
</div>

Per impostazione predefinita l'endpoint è **stateless per richiesta** (viene generata una nuova chiave di sessione a ogni chiamata).

Se la richiesta include una stringa `user` di OpenAI, il Gateway ne deriva una chiave di sessione stabile, così che chiamate ripetute possano condividere la stessa sessione dell'agente.

<div id="streaming-sse">
  ## Streaming (SSE)
</div>

Imposta `stream: true` per ricevere eventi SSE (Server-Sent Events):

- `Content-Type: text/event-stream`
- Ogni riga di evento ha la forma `data: <json>`
- Il flusso termina con `data: [DONE]`

<div id="examples">
  ## Esempi
</div>

Senza streaming:

```bash
curl -sS http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "messages": [{"role":"user","content":"hi"}]
  }'
```

Streaming:

```bash
curl -N http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "messages": [{"role":"user","content":"hi"}]
  }'
```
