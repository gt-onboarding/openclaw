---
title: Sesión
summary: "Reglas para la gestión de sesiones, claves y persistencia de chats"
read_when:
  - Modificar el manejo o el almacenamiento de sesiones
---

<div id="session-management">
  # Gestión de sesiones
</div>

OpenClaw considera **una sesión de chat directo por agente** como primaria. Los chats directos se colapsan a `agent:<agentId>:<mainKey>` (por defecto `main`), mientras que los chats de grupo/canal obtienen sus propias claves. Se respeta `session.mainKey`.

Usa `session.dmScope` para controlar cómo se agrupan los **mensajes directos**:

* `main` (predeterminado): todos los mensajes directos comparten la sesión principal para mantener la continuidad.
* `per-peer`: aísla por ID de remitente a través de canales.
* `per-channel-peer`: aísla por canal + remitente (recomendado para bandejas de entrada multiusuario).
* `per-account-channel-peer`: aísla por cuenta + canal + remitente (recomendado para bandejas de entrada multicuenta).
  Usa `session.identityLinks` para asignar IDs de usuarios con prefijo de proveedor a una identidad canónica, de modo que la misma persona comparta una sesión de mensajes directos a través de canales cuando se use `per-peer`, `per-channel-peer` o `per-account-channel-peer`.

<div id="gateway-is-the-source-of-truth">
  ## El Gateway es la fuente de la verdad
</div>

Todo el estado de la sesión **es propiedad del Gateway** (el OpenClaw “maestro”). Los clientes de la UI (app de macOS, WebChat, etc.) deben consultar al Gateway para obtener listas de sesiones y conteos de tokens en lugar de leer archivos locales.

* En **modo remoto**, el almacén de sesiones que importa reside en el host remoto del Gateway, no en tu Mac.
* Los conteos de tokens mostrados en las UIs provienen de los campos del almacén del Gateway (`inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`). Los clientes no analizan transcripciones JSONL para corregir los totales.

<div id="where-state-lives">
  ## Dónde se almacena el estado
</div>

* En el **host del Gateway**:
  * Archivo de almacenamiento: `~/.openclaw/agents/<agentId>/sessions/sessions.json` (por agente).
* Transcripciones: `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl` (las sesiones de topic en Telegram usan `.../<SessionId>-topic-<threadId>.jsonl`).
* El almacén es un mapa `sessionKey -> { sessionId, updatedAt, ... }`. Eliminar entradas es seguro; se regeneran bajo demanda.
* Las entradas de grupo pueden incluir `displayName`, `channel`, `subject`, `room` y `space` para etiquetar sesiones en UIs.
* Las entradas de sesión incluyen metadatos de `origin` (etiqueta + sugerencias de enrutamiento) para que las UIs puedan explicar de dónde procede una sesión.
* OpenClaw **no** lee carpetas de sesiones heredadas de Pi/Tau.

<div id="session-pruning">
  ## Purgado de sesiones
</div>

De forma predeterminada, OpenClaw elimina **resultados antiguos de herramientas** del contexto en memoria justo antes de las llamadas al LLM.
Esto **no** modifica el historial JSONL. Consulta [/concepts/session-pruning](/es/concepts/session-pruning).

<div id="pre-compaction-memory-flush">
  ## Vaciado de memoria previo a la compactación
</div>

Cuando una sesión se aproxima a la compactación automática, OpenClaw puede ejecutar
un **vaciado silencioso de memoria** que le recuerda al modelo escribir notas
persistentes en disco. Esto solo se ejecuta cuando el espacio de trabajo tiene
permisos de escritura. Consulta [Memoria](/es/concepts/memory) y
[Compactación](/es/concepts/compaction).

<div id="mapping-transports-session-keys">
  ## Asignación de transportes → claves de sesión
</div>

* Los chats directos siguen `session.dmScope` (por defecto `main`).
  * `main`: `agent:<agentId>:<mainKey>` (continuidad entre dispositivos/canales).
    * Varios números de teléfono y canales pueden asignarse a la misma clave principal de agente; actúan como transportes hacia una misma conversación.
  * `per-peer`: `agent:<agentId>:dm:<peerId>`.
  * `per-channel-peer`: `agent:<agentId>:<channel>:dm:<peerId>`.
  * `per-account-channel-peer`: `agent:<agentId>:<channel>:<accountId>:dm:<peerId>` (`accountId` es `default` de forma predeterminada).
  * Si `session.identityLinks` coincide con un ID de peer con prefijo de proveedor (por ejemplo `telegram:123`), la clave canónica reemplaza `<peerId>` para que la misma persona comparta una sesión entre canales.
* Los chats de grupo aíslan el estado: `agent:<agentId>:<channel>:group:<id>` (las salas/canales usan `agent:<agentId>:<channel>:channel:<id>`).
  * Los temas de foros de Telegram agregan `:topic:<threadId>` al ID de grupo para el aislamiento.
  * Las claves heredadas `group:<id>` siguen siendo reconocidas para la migración.
* Los contextos entrantes aún pueden usar `group:<id>`; el canal se infiere a partir de `Provider` y se normaliza a la forma canónica `agent:<agentId>:<channel>:group:<id>`.
* Otras fuentes:
  * Tareas cron: `cron:<job.id>`
  * Webhooks: `hook:<uuid>` (a menos que el hook la establezca explícitamente)
  * Ejecuciones de nodo: `node-<nodeId>`

<div id="lifecycle">
  ## Ciclo de vida
</div>

* Política de reinicio: las sesiones se reutilizan hasta que expiran, y la expiración se evalúa en el siguiente mensaje entrante.
* Reinicio diario: el valor predeterminado es **4:00 AM hora local en el host donde se ejecuta el Gateway**. Una sesión se considera obsoleta cuando su última actualización es anterior a la hora de reinicio diario más reciente.
* Reinicio por inactividad (opcional): `idleMinutes` agrega una ventana deslizante de inactividad. Cuando se configuran reinicios diarios y por inactividad, **el que expire primero** fuerza una nueva sesión.
* Solo inactividad heredado: si configuras `session.idleMinutes` sin ninguna configuración `session.reset`/`resetByType`, OpenClaw permanece en modo solo inactividad para mantener la compatibilidad hacia atrás.
* Anulaciones por tipo (opcional): `resetByType` te permite anular la política para sesiones `dm`, `group` y `thread` (thread = hilos de Slack/Discord, temas de Telegram, hilos de Matrix cuando los proporciona el conector).
* Anulaciones por canal (opcional): `resetByChannel` anula la política de reinicio para un canal (se aplica a todos los tipos de sesión de ese canal y tiene prioridad sobre `reset`/`resetByType`).
* Disparadores de reinicio: `/new` o `/reset` exactos (más cualquier extra en `resetTriggers`) inician un nuevo id de sesión y pasan el resto del mensaje. `/new <model>` acepta un alias de modelo, `provider/model` o un nombre de proveedor (búsqueda aproximada) para establecer el modelo de la nueva sesión. Si se envía `/new` o `/reset` solo, OpenClaw ejecuta un breve turno de saludo de “hola” para confirmar el reinicio.
* Reinicio manual: elimina claves específicas del almacén o quita la transcripción JSONL; el siguiente mensaje las recrea.
* Los trabajos de cron aislados siempre generan un `sessionId` nuevo por ejecución (sin reutilización por inactividad).

<div id="send-policy-optional">
  ## Política de send (opcional)
</div>

Bloquea la entrega de tipos específicos de sesión sin necesidad de enumerar identificadores individuales.

```json5
{
  session: {
    sendPolicy: {
      rules: [
        { action: "deny", match: { channel: "discord", chatType: "group" } },
        { action: "deny", match: { keyPrefix: "cron:" } }
      ],
      default: "allow"
    }
  }
}
```

Anulación en tiempo de ejecución (solo propietario):

* `/send on` → permitir para esta sesión
* `/send off` → denegar para esta sesión
* `/send inherit` → eliminar la anulación y usar las reglas de configuración
  Envía estos comandos como mensajes independientes para que se registren.

<div id="configuration-optional-rename-example">
  ## Configuración (ejemplo opcional de cambio de nombre)
</div>

```json5
// ~/.openclaw/openclaw.json
{
  session: {
    scope: "per-sender",      // keep group keys separate
    dmScope: "main",          // DM continuity (set per-channel-peer/per-account-channel-peer for shared inboxes)
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"]
    },
    reset: {
      // Valores predeterminados: mode=daily, atHour=4 (hora local del host del gateway).
      // Si también estableces idleMinutes, el que expire primero tiene prioridad.
      mode: "daily",
      atHour: 4,
      idleMinutes: 120
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      dm: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 }
    },
    resetByChannel: {
      discord: { mode: "idle", idleMinutes: 10080 }
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    mainKey: "main",
  }
}
```

<div id="inspecting">
  ## Inspección
</div>

* `openclaw status` — muestra la ruta del almacén y las sesiones recientes.
* `openclaw sessions --json` — vuelca todas las entradas (filtra usando `--active <minutes>`).
* `openclaw gateway call sessions.list --params '{}'` — obtiene sesiones del Gateway en ejecución (usa `--url`/`--token` para acceder a un Gateway remoto).
* Envía `/status` como mensaje independiente en el chat para ver si el agente está disponible, cuánto del contexto de la sesión está en uso, los modos actuales thinking/verbose y cuándo se actualizaron por última vez tus credenciales de WhatsApp Web (ayuda a detectar cuándo hay que volver a vincular).
* Envía `/context list` o `/context detail` para ver qué hay en el system prompt y en los archivos del espacio de trabajo inyectados (y cuáles son los mayores contribuyentes al contexto).
* Envía `/stop` como mensaje independiente para abortar la ejecución actual, limpiar los mensajes de seguimiento en cola para esa sesión y detener cualquier ejecución de subagentes que haya generado (la respuesta incluye el número de ejecuciones detenidas).
* Envía `/compact` (instrucciones opcionales) como mensaje independiente para resumir el contexto más antiguo y liberar espacio en la ventana de contexto. Consulta [/concepts/compaction](/es/concepts/compaction).
* Las transcripciones JSONL pueden abrirse directamente para revisar los turnos completos.

<div id="tips">
  ## Consejos
</div>

* Mantén la clave primaria dedicada exclusivamente al tráfico 1:1; permite que los grupos mantengan sus propias claves.
* Al automatizar la limpieza, elimina claves individuales en lugar de borrar todo el almacén para preservar el contexto en otros lugares.

<div id="session-origin-metadata">
  ## Metadatos de origen de la sesión
</div>

Cada entrada de sesión registra de dónde proviene (en la medida de lo posible) en `origin`:

* `label`: etiqueta legible por humanos (resuelta a partir de la etiqueta de la conversación + asunto del grupo/canal)
* `provider`: identificador de canal normalizado (incluyendo extensiones)
* `from`/`to`: identificadores de enrutamiento sin procesar del sobre de entrada
* `accountId`: identificador de cuenta del proveedor (cuando hay varias cuentas)
* `threadId`: identificador de hilo/tema cuando el canal lo admite
  Los campos de origen se completan para mensajes directos, canales y grupos. Si un
  conector solo actualiza el enrutamiento de entrega (por ejemplo, para mantener
  actualizada una sesión principal de DM), aun así debe proporcionar contexto
  entrante para que la sesión conserve sus metadatos descriptivos. Las
  extensiones pueden hacer esto enviando `ConversationLabel`, `GroupSubject`,
  `GroupChannel`, `GroupSpace` y `SenderName` en el contexto de entrada y
  llamando a `recordSessionMetaFromInbound` (o pasando el mismo contexto a
  `updateLastRoute`).