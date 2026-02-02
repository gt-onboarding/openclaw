---
title: Herramienta de sesión
summary: "Herramientas de sesión de agente para listar sesiones, recuperar el historial y enviar mensajes entre sesiones"
read_when:
  - Añadir o modificar herramientas de sesión
---

<div id="session-tools">
  # Herramientas de sesión
</div>

Objetivo: un conjunto de herramientas pequeño y difícil de usar mal para que los agentes puedan listar sesiones, consultar el historial y enviar a otra sesión.

<div id="tool-names">
  ## Nombres de herramientas
</div>

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

<div id="key-model">
  ## Modelo de claves
</div>

- El contenedor principal de chat directo es siempre la clave literal `"main"` (resuelta a la clave principal del agente actual).
- Los chats de grupo usan `agent:<agentId>:<channel>:group:<id>` o `agent:<agentId>:<channel>:channel:<id>` (proporciona la clave completa).
- Los trabajos de cron usan `cron:<job.id>`.
- Los hooks usan `hook:<uuid>` a menos que se configure explícitamente.
- Las sesiones de nodo usan `node-<nodeId>` a menos que se configure explícitamente.

`global` y `unknown` son valores reservados y nunca se muestran en la lista. Si `session.scope = "global"`, lo mapeamos a `main` para todas las herramientas, de modo que los clientes nunca vean `global`.

<div id="sessions_list">
  ## sessions_list
</div>

Enumera las sesiones como un array de filas.

Parámetros:

- `kinds?: string[]` filtro: cualquiera de `"main" | "group" | "cron" | "hook" | "node" | "other"`
- `limit?: number` máximo de filas (predeterminado: valor predeterminado del servidor, limitado, p. ej., a 200)
- `activeMinutes?: number` solo las sesiones actualizadas dentro de N minutos
- `messageLimit?: number` 0 = sin mensajes (predeterminado 0); >0 = incluir los últimos N mensajes

Comportamiento:

- `messageLimit > 0` obtiene `chat.history` por sesión e incluye los últimos N mensajes.
- Los resultados de herramientas se excluyen de la salida de la lista; usa `sessions_history` para mensajes de herramientas.
- Cuando se ejecuta en una sesión de agente en **sandbox**, las herramientas de sesión usan por defecto **visibilidad solo para instancias generadas** (ver más abajo).

Estructura de la fila (JSON):

- `key`: clave de la sesión (string)
- `kind`: `main | group | cron | hook | node | other`
- `channel`: `whatsapp | telegram | discord | signal | imessage | webchat | internal | unknown`
- `displayName` (etiqueta de visualización del grupo, si está disponible)
- `updatedAt` (ms)
- `sessionId`
- `model`, `contextTokens`, `totalTokens`
- `thinkingLevel`, `verboseLevel`, `systemSent`, `abortedLastRun`
- `sendPolicy` (sobrescritura específica de la sesión, si está establecida)
- `lastChannel`, `lastTo`
- `deliveryContext` (`{ channel, to, accountId }` normalizado cuando está disponible)
- `transcriptPath` (ruta aproximada derivada del directorio de almacenamiento + sessionId)
- `messages?` (solo cuando `messageLimit > 0`)

<div id="sessions_history">
  ## sessions_history
</div>

Recupera la transcripción de una sesión.

Parámetros:

- `sessionKey` (obligatorio; acepta la clave de sesión o el `sessionId` de `sessions_list`)
- `limit?: number` máximo de mensajes (el servidor aplica un límite superior)
- `includeTools?: boolean` (false por defecto)

Comportamiento:

- `includeTools=false` filtra los mensajes con `role: "toolResult"`.
- Devuelve un array de mensajes en el formato de transcripción en bruto.
- Cuando se proporciona un `sessionId`, OpenClaw lo resuelve a la clave de sesión correspondiente (produce un error si falta el id).

<div id="sessions_send">
  ## sessions_send
</div>

Enviar un mensaje a otra sesión.

Parámetros:

- `sessionKey` (obligatorio; acepta la clave de sesión o `sessionId` de `sessions_list`)
- `message` (obligatorio)
- `timeoutSeconds?: number` (predeterminado >0; 0 = fire-and-forget, sin esperar respuesta)

Comportamiento:

- `timeoutSeconds = 0`: se pone en cola y se devuelve `{ runId, status: "accepted" }`.
- `timeoutSeconds > 0`: espera hasta N segundos a que se complete y luego devuelve `{ runId, status: "ok", reply }`.
- Si la espera expira: `{ runId, status: "timeout", error }`. La ejecución continúa; llama a `sessions_history` más tarde.
- Si la ejecución falla: `{ runId, status: "error", error }`.
- Las ejecuciones de anuncio de entrega se realizan después de que la ejecución primaria se haya completado y se ejecutan a mejor esfuerzo; `status: "ok"` no garantiza que el anuncio se haya entregado.
- Espera a través del Gateway con `agent.wait` (del lado del servidor), por lo que las reconexiones no cancelan la espera.
- El contexto de mensaje entre agentes se inyecta para la ejecución primaria.
- Después de que la ejecución primaria se completa, OpenClaw ejecuta un **bucle de respuesta de retorno**:
  - A partir de la ronda 2, se alterna entre el agente solicitante y el agente de destino.
  - Responde exactamente `REPLY_SKIP` para detener el ping‑pong.
  - El máximo de turnos es `session.agentToAgent.maxPingPongTurns` (0–5, predeterminado 5).
- Una vez que termina el bucle, OpenClaw ejecuta el **paso de anuncio de agente a agente** (solo el agente de destino):
  - Responde exactamente `ANNOUNCE_SKIP` para permanecer en silencio.
  - Cualquier otra respuesta se envía al canal de destino.
  - El paso de anuncio incluye la solicitud original + la respuesta de la ronda 1 + la última respuesta del ping‑pong.

<div id="channel-field">
  ## Campo `channel`
</div>

- Para grupos, `channel` es el canal registrado en el registro de la sesión.
- Para chats directos, `channel` se asigna a partir de `lastChannel`.
- Para cron/hook/nodo, `channel` es `internal`.
- Si no está presente, `channel` es `unknown`.

<div id="security-send-policy">
  ## Seguridad / Política de envío
</div>

Bloqueo basado en políticas según el tipo de canal/chat (no por ID de la sesión).

```json
{
  "session": {
    "sendPolicy": {
      "rules": [
        {
          "match": { "channel": "discord", "chatType": "group" },
          "action": "deny"
        }
      ],
      "default": "allow"
    }
  }
}
```

Sobrescritura en tiempo de ejecución (por entrada de sesión):

* `sendPolicy: "allow" | "deny"` (sin definir = hereda la configuración)
* Configurable mediante `sessions.patch` o solo por el propietario con `/send on|off|inherit` (mensaje independiente).

Puntos de aplicación:

* `chat.send` / `agente` (Gateway)
* lógica de entrega de respuestas automáticas


<div id="sessions_spawn">
  ## sessions_spawn
</div>

Inicia una ejecución de subagente en una sesión aislada y anuncia el resultado de vuelta en el canal de chat del solicitante.

Parámetros:

- `task` (obligatorio)
- `label?` (opcional; se usa para registros/UI)
- `agentId?` (opcional; inicia bajo otro id de agente si está permitido)
- `model?` (opcional; sobrescribe el modelo del subagente; los valores no válidos producen un error)
- `runTimeoutSeconds?` (por defecto 0; cuando se establece, aborta la ejecución del subagente después de N segundos)
- `cleanup?` (`delete|keep`, por defecto `keep`)

Lista de permitidos:

- `agents.list[].subagents.allowAgents`: lista de ids de agente permitidos vía `agentId` (`["*"]` para permitir cualquiera). Por defecto: solo el agente solicitante.

Descubrimiento:

- Usa `agents_list` para descubrir qué ids de agente están permitidos para `sessions_spawn`.

Comportamiento:

- Inicia una nueva sesión `agent:<agentId>:subagent:<uuid>` con `deliver: false`.
- Los subagentes usan por defecto el conjunto completo de herramientas **menos las herramientas de sesión** (configurable vía `tools.subagents.tools`).
- A los subagentes no se les permite llamar a `sessions_spawn` (no se permite subagente → subagente).
- Siempre no bloqueante: devuelve `{ status: "accepted", runId, childSessionKey }` inmediatamente.
- Al completarse, OpenClaw ejecuta un **paso de anuncio** del subagente y publica el resultado en el canal de chat del solicitante.
- Responde exactamente `ANNOUNCE_SKIP` durante el paso de anuncio para permanecer en silencio.
- Las respuestas de anuncio se normalizan a `Status`/`Result`/`Notes`; `Status` proviene del resultado de la ejecución en tiempo de ejecución (no del texto del modelo).
- Las sesiones de subagente se archivan automáticamente después de `agents.defaults.subagents.archiveAfterMinutes` (por defecto: 60).
- Las respuestas de anuncio incluyen una línea de estadísticas (tiempo de ejecución, tokens, sessionKey/sessionId, ruta de la transcripción y costo opcional).

<div id="sandbox-session-visibility">
  ## Visibilidad de sesiones en sandbox
</div>

Las sesiones en sandbox pueden usar herramientas de sesión, pero de forma predeterminada solo pueden ver las sesiones que generaron mediante `sessions_spawn`.

Configuración:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        // default: "spawned"
        sessionToolsVisibility: "spawned" // o "all"
      }
    }
  }
}
```
