---
title: API HTTP de OpenAI
summary: "Exponer un endpoint HTTP /v1/chat/completions compatible con OpenAI desde el Gateway"
read_when:
  - Integrar herramientas que requieren compatibilidad con OpenAI Chat Completions
---

<div id="openai-chat-completions-http">
  # Finalizaciones de chat de OpenAI (HTTP)
</div>

El Gateway de OpenClaw puede ofrecer un endpoint de Finalizaciones de chat compatible con OpenAI.

Este endpoint está **desactivado de forma predeterminada**. Primero debes activarlo en la configuración.

- `POST /v1/chat/completions`
- Mismo puerto que el Gateway (multiplexación WS + HTTP): `http://<gateway-host>:<port>/v1/chat/completions`

Internamente, las solicitudes se ejecutan como una ejecución normal de un agente del Gateway (el mismo flujo de código que `openclaw agent`), por lo que el enrutamiento, los permisos y la configuración coinciden con los de tu Gateway.

<div id="authentication">
  ## Autenticación
</div>

Utiliza la configuración de autenticación del Gateway. Envía un token de tipo Bearer:

- `Authorization: Bearer <token>`

Notas:

- Cuando `gateway.auth.mode="token"`, usa `gateway.auth.token` (o `OPENCLAW_GATEWAY_TOKEN`).
- Cuando `gateway.auth.mode="password"`, usa `gateway.auth.password` (o `OPENCLAW_GATEWAY_PASSWORD`).

<div id="choosing-an-agent">
  ## Seleccionar un agente
</div>

No se requieren encabezados personalizados: incluye el ID del agente en el campo `model` de OpenAI:

- `model: "openclaw:<agentId>"` (ejemplo: `"openclaw:main"`, `"openclaw:beta"`)
- `model: "agent:<agentId>"` (alias)

O bien dirige la petición a un agente específico de OpenClaw mediante un encabezado:

- `x-openclaw-agent-id: <agentId>` (predeterminado: `main`)

Avanzado:

- `x-openclaw-session-key: <sessionKey>` para controlar completamente el enrutamiento de la sesión.

<div id="enabling-the-endpoint">
  ## Activar el endpoint
</div>

Configura `gateway.http.endpoints.chatCompletions.enabled` a `true`:

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
  ## Desactivar el endpoint
</div>

Configura `gateway.http.endpoints.chatCompletions.enabled` en `false`:

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
  ## Comportamiento de la sesión
</div>

De forma predeterminada, el endpoint es **sin estado por cada solicitud** (se genera una nueva clave de sesión en cada llamada).

Si la solicitud incluye una cadena `user` de OpenAI, el Gateway calcula una clave de sesión estable a partir de ella, de modo que las llamadas repetidas puedan compartir la misma sesión de agente.

<div id="streaming-sse">
  ## Transmisión (SSE)
</div>

Establece `stream: true` para recibir Server-Sent Events (SSE):

- `Content-Type: text/event-stream`
- Cada línea de evento tiene el formato `data: <json>`
- El flujo termina con `data: [DONE]`

<div id="examples">
  ## Ejemplos
</div>

Sin streaming:

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

Transmisión en streaming:

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
