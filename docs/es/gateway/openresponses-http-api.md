---
title: API HTTP de OpenResponses
summary: "Exponer desde el Gateway un endpoint HTTP /v1/responses compatible con OpenResponses"
read_when:
  - Integrar clientes que usan la API OpenResponses
  - Necesitas entradas basadas en elementos, llamadas de herramientas del cliente o eventos SSE
---

<div id="openresponses-api-http">
  # API de OpenResponses (HTTP)
</div>

El Gateway de OpenClaw puede exponer un endpoint `POST /v1/responses` compatible con OpenResponses.

Este endpoint está **deshabilitado de forma predeterminada**. Habilítalo primero en la configuración.

- `POST /v1/responses`
- Mismo puerto que el Gateway (multiplexación WS + HTTP): `http://<gateway-host>:<port>/v1/responses`

Internamente, las solicitudes se ejecutan como una ejecución normal de un agente del Gateway (el mismo flujo de código que
`openclaw agent`), por lo que el enrutamiento/permisos/configuración coinciden con tu Gateway.

<div id="authentication">
  ## Autenticación
</div>

Utiliza la configuración de autenticación del Gateway. Envía un token de tipo Bearer:

- `Authorization: Bearer <token>`

Notas:

- Cuando `gateway.auth.mode="token"`, usa `gateway.auth.token` (o `OPENCLAW_GATEWAY_TOKEN`).
- Cuando `gateway.auth.mode="password"`, usa `gateway.auth.password` (o `OPENCLAW_GATEWAY_PASSWORD`).

<div id="choosing-an-agent">
  ## Elegir un agente
</div>

No necesitas encabezados personalizados: codifica el ID del agente en el campo `model` de OpenResponses:

- `model: "openclaw:<agentId>"` (ejemplo: `"openclaw:main"`, `"openclaw:beta"`)
- `model: "agent:<agentId>"` (alias)

O dirige la solicitud a un agente específico de OpenClaw mediante un encabezado:

- `x-openclaw-agent-id: <agentId>` (valor predeterminado: `main`)

Avanzado:

- `x-openclaw-session-key: <sessionKey>` para tener control total sobre el enrutamiento de la sesión.

<div id="enabling-the-endpoint">
  ## Habilitar el endpoint
</div>

Configura `gateway.http.endpoints.responses.enabled` en `true`:

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
  ## Desactivar el endpoint
</div>

Establece `gateway.http.endpoints.responses.enabled` en `false`:

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
  ## Comportamiento de la sesión
</div>

De forma predeterminada, el endpoint es **sin estado por solicitud** (se genera una nueva clave de sesión en cada llamada).

Si la solicitud incluye una cadena `user` de OpenResponses, el Gateway deriva una clave de sesión estable
a partir de ella, de modo que las llamadas repetidas puedan compartir la misma sesión de agente.

<div id="request-shape-supported">
  ## Estructura de la solicitud (admitida)
</div>

La solicitud sigue la API de OpenResponses con entrada basada en ítems. Compatibilidad actual:

- `input`: cadena o array de objetos ítem.
- `instructions`: se integran en el mensaje de sistema.
- `tools`: definiciones de herramientas del cliente (function tools).
- `tool_choice`: filtra o exige herramientas del cliente.
- `stream`: habilita la transmisión por SSE.
- `max_output_tokens`: límite de salida de mejor esfuerzo (dependiente del proveedor).
- `user`: enrutamiento estable de la sesión.

Aceptados pero **actualmente ignorados**:

- `max_tool_calls`
- `reasoning`
- `metadata`
- `store`
- `previous_response_id`
- `truncation`

<div id="items-input">
  ## Elementos de entrada
</div>

<div id="message">
  ### `message`
</div>

Roles: `system`, `developer`, `user`, `assistant`.

- `system` y `developer` se añaden al prompt del sistema.
- El elemento `user` o `function_call_output` más reciente se convierte en el “mensaje actual”.
- Los mensajes anteriores de usuario/asistente se incluyen como historial de contexto.

<div id="function_call_output-turn-based-tools">
  ### `function_call_output` (herramientas basadas en turnos)
</div>

Envía los resultados de las herramientas de vuelta al modelo:

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

Se aceptan por compatibilidad con el esquema, pero se ignoran al construir el prompt.

<div id="tools-client-side-function-tools">
  ## Herramientas (herramientas de tipo función del lado del cliente)
</div>

Proporciona las herramientas con `tools: [{ type: "function", function: { name, description?, parameters? } }]`.

Si el agente decide invocar una herramienta, la respuesta devuelve un elemento de salida `function_call`.
Luego envía una solicitud de seguimiento con `function_call_output` para continuar el turno.

<div id="images-input_image">
  ## Imágenes (`input_image`)
</div>

Admite orígenes base64 o URL:

```json
{
  "type": "input_image",
  "source": { "type": "url", "url": "https://example.com/image.png" }
}
```

Tipos MIME permitidos (actualmente): `image/jpeg`, `image/png`, `image/gif`, `image/webp`.
Tamaño máximo (actualmente): 10 MB.


<div id="files-input_file">
  ## Archivos (`input_file`)
</div>

Permite orígenes en base64 o mediante URL:

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

Tipos MIME permitidos (actuales): `text/plain`, `text/markdown`, `text/html`, `text/csv`,
`application/json`, `application/pdf`.

Tamaño máximo (actual): 5 MB.

Comportamiento actual:

* El contenido del archivo se decodifica y se agrega al **system prompt**, no al mensaje del usuario,
  por lo que permanece efímero (no se persiste en el historial de la sesión).
* Los PDF se analizan para extraer texto. Si se encuentra poco texto, las primeras páginas se rasterizan
  como imágenes y se pasan al modelo.

El análisis de PDF usa la compilación legacy de `pdfjs-dist` compatible con Node (sin worker). La compilación moderna
de PDF.js espera workers de navegador y globales DOM, por lo que no se utiliza en el Gateway.

Valores predeterminados para la obtención por URL:

* `files.allowUrl`: `true`
* `images.allowUrl`: `true`
* Las solicitudes están protegidas (resolución de DNS, bloqueo de IP privadas, límites de redirecciones, tiempos de espera).


<div id="file-image-limits-config">
  ## Límites de archivos e imágenes (configuración)
</div>

Los valores predeterminados se pueden ajustar en `gateway.http.endpoints.responses`:

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

Valores predeterminados si se omiten:

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
  ## Transmisión (SSE)
</div>

Establece `stream: true` para recibir Server-Sent Events (SSE):

- `Content-Type: text/event-stream`
- Cada línea de evento tiene el formato `event: <type>` y `data: <json>`
- El flujo termina con `data: [DONE]`

Tipos de eventos que se generan actualmente:

- `response.created`
- `response.in_progress`
- `response.output_item.added`
- `response.content_part.added`
- `response.output_text.delta`
- `response.output_text.done`
- `response.content_part.done`
- `response.output_item.done`
- `response.completed`
- `response.failed` (en caso de error)

<div id="usage">
  ## Uso
</div>

`usage` se rellena cuando el proveedor subyacente informa los recuentos de tokens.

<div id="errors">
  ## Errores
</div>

Los errores se representan mediante un objeto JSON como:

```json
{ "error": { "message": "...", "type": "invalid_request_error" } }
```

Casos frecuentes:

* `401` autenticación ausente o no válida
* `400` cuerpo de la solicitud no válido
* `405` método incorrecto


<div id="examples">
  ## Ejemplos
</div>

Sin streaming:

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
