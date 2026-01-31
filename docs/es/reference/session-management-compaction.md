---
title: Compactación en la gestión de sesiones
summary: "Análisis en profundidad: almacén de sesiones y transcripciones, ciclo de vida y detalles internos de la (auto)compactación"
read_when:
  - Necesitas depurar IDs de sesión, JSONL de transcripción o campos de sessions.json
  - Estás cambiando el comportamiento de la compactación automática o añadiendo tareas de mantenimiento previas a la compactación
  - Quieres implementar vaciados de memoria o turnos silenciosos del sistema
---

<div id="session-management-compaction-deep-dive">
  # Gestión y compactación de sesiones (análisis en profundidad)
</div>

Este documento explica cómo OpenClaw gestiona las sesiones de extremo a extremo:

* **Enrutamiento de sesiones** (cómo los mensajes entrantes se asignan a una `sessionKey`)
* **Almacén de sesiones** (`sessions.json`) y qué información registra
* **Persistencia de transcripciones** (`*.jsonl`) y su estructura
* **Higiene de transcripciones** (correcciones específicas del proveedor antes de las ejecuciones)
* **Límites de contexto** (ventana de contexto vs tokens contabilizados)
* **Compactación** (compactación manual + automática) y dónde acoplar trabajo previo a la compactación
* **Mantenimiento silencioso** (por ejemplo, escrituras de memoria que no deberían producir salida visible para el usuario)

Si quieres primero una visión de más alto nivel, comienza con:

* [/concepts/session](/es/concepts/session)
* [/concepts/compaction](/es/concepts/compaction)
* [/concepts/session-pruning](/es/concepts/session-pruning)
* [/reference/transcript-hygiene](/es/reference/transcript-hygiene)

***

<div id="source-of-truth-the-gateway">
  ## Fuente de la verdad: el Gateway
</div>

OpenClaw está diseñado en torno a un único **proceso de Gateway** que mantiene el estado de las sesiones.

* Las UIs (app de macOS, Control UI web, TUI) deben consultar al Gateway para obtener las listas de sesiones y los recuentos de tokens.
* En modo remoto, los archivos de sesión están en el host remoto; «comprobar los archivos locales de tu Mac» no reflejará lo que está utilizando el Gateway.

***

<div id="two-persistence-layers">
  ## Dos capas de persistencia
</div>

OpenClaw guarda las sesiones de forma persistente en dos capas:

1. **Almacén de sesiones (`sessions.json`)**
   * Mapa clave/valor: `sessionKey -> SessionEntry`
   * Pequeño, mutable y seguro de editar (o eliminar entradas)
   * Hace un seguimiento de los metadatos de la sesión (ID de sesión actual, última actividad, interruptores, contadores de tokens, etc.)

2. **Transcripción (`<sessionId>.jsonl`)**
   * Transcripción de solo anexión con estructura de árbol (las entradas tienen `id` + `parentId`)
   * Almacena la conversación real + llamadas a herramientas + resúmenes de compactación
   * Se usa para reconstruir el contexto del modelo en turnos futuros

***

<div id="on-disk-locations">
  ## Ubicaciones en disco
</div>

Por agente, en el host del Gateway:

* Almacenamiento: `~/.openclaw/agents/<agentId>/sessions/sessions.json`
* Transcripciones: `~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl`
  * Sesiones por tema en Telegram: `.../<sessionId>-topic-<threadId>.jsonl`

OpenClaw resuelve estas ubicaciones mediante `src/config/sessions.ts`.

***

<div id="session-keys-sessionkey">
  ## Claves de sesión (`sessionKey`)
</div>

Un `sessionKey` identifica *en qué contenedor de conversación* estás (enrutamiento + aislamiento).

Patrones comunes:

* Chat principal/directo (por agente): `agent:<agentId>:<mainKey>` (por defecto `main`)
* Grupo: `agent:<agentId>:<channel>:group:<id>`
* Sala/canal (Discord/Slack): `agent:<agentId>:<channel>:channel:<id>` o `...:room:<id>`
* Cron: `cron:<job.id>`
* Webhook: `hook:<uuid>` (a menos que se haya sobrescrito)

Las reglas canónicas están documentadas en [/concepts/session](/es/concepts/session).

***

<div id="session-ids-sessionid">
  ## IDs de sesión (`sessionId`)
</div>

Cada `sessionKey` apunta a un `sessionId` actual (el archivo de transcripción que continúa la conversación).

Reglas generales:

* **Reset** (`/new`, `/reset`) crea un nuevo `sessionId` para ese `sessionKey`.
* **Daily reset** (por defecto a las 4:00 AM hora local en el host del Gateway) crea un nuevo `sessionId` en el siguiente mensaje después del límite de reinicio.
* **Idle expiry** (`session.reset.idleMinutes` o `session.idleMinutes` heredado) crea un nuevo `sessionId` cuando llega un mensaje después de la ventana de inactividad. Cuando el reinicio diario y por inactividad están ambos configurados, gana el que expire primero.

Detalle de implementación: la decisión se toma en `initSessionState()` en `src/auto-reply/reply/session.ts`.

***

<div id="session-store-schema-sessionsjson">
  ## Esquema del almacén de sesiones (`sessions.json`)
</div>

El tipo de valor del almacén es `SessionEntry` en `src/config/sessions.ts`.

Campos clave (lista no exhaustiva):

* `sessionId`: ID de la transcripción actual (el nombre de archivo se deriva de este valor a menos que se defina `sessionFile`)
* `updatedAt`: marca de tiempo de la última actividad
* `sessionFile`: ruta explícita y opcional de la transcripción que anula la predeterminada
* `chatType`: `direct | group | room` (ayuda a las UIs y a la política de send)
* `provider`, `subject`, `room`, `space`, `displayName`: metadatos para el etiquetado de grupos/canales
* Conmutadores:
  * `thinkingLevel`, `verboseLevel`, `reasoningLevel`, `elevatedLevel`
  * `sendPolicy` (anulación por sesión)
* Selección de modelo:
  * `providerOverride`, `modelOverride`, `authProfileOverride`
* Contadores de tokens (de mejor esfuerzo / dependientes del proveedor):
  * `inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`
* `compactionCount`: frecuencia con la que se completó la compactación automática para esta clave de sesión
* `memoryFlushAt`: marca de tiempo del último vaciado de memoria previo a la compactación
* `memoryFlushCompactionCount`: conteo de compactaciones cuando se ejecutó el último vaciado

Es seguro editar el almacén, pero el Gateway tiene la última palabra: puede reescribir o rehidratar entradas a medida que se ejecutan las sesiones.

***

<div id="transcript-structure-jsonl">
  ## Estructura de transcripción (`*.jsonl`)
</div>

Las transcripciones son gestionadas por el `SessionManager` de `@mariozechner/pi-coding-agent`.

El archivo está en formato JSONL:

* Primera línea: encabezado de sesión (`type: "session"`, incluye `id`, `cwd`, `timestamp`, `parentSession` opcional)
* Luego: entradas de sesión con `id` + `parentId` (árbol)

Tipos de entrada principales:

* `message`: mensajes de usuario/asistente/toolResult
* `custom_message`: mensajes inyectados por extensiones que *sí* entran en el contexto del modelo (pueden ocultarse de la UI)
* `custom`: estado de extensión que *no* entra en el contexto del modelo
* `compaction`: resumen de compactación persistente con `firstKeptEntryId` y `tokensBefore`
* `branch_summary`: resumen persistente al navegar por una rama del árbol

<div id="context-windows-vs-tracked-tokens">
  ## OpenClaw intencionalmente **no** “arregla” las transcripciones; el Gateway usa `SessionManager` para leerlas y escribirlas.
</div>

<div id="compaction-what-it-is">
  ## Ventanas de contexto vs tokens rastreados
</div>

Hay dos conceptos diferentes relevantes:

1. **Ventana de contexto del modelo**: límite estricto por modelo (tokens visibles para el modelo)
2. **Contadores del almacén de sesiones**: estadísticas acumulativas continuas escritas en `sessions.json` (usadas para /status y paneles)

Si estás ajustando límites:

* La ventana de contexto viene del catálogo de modelos (y se puede sobrescribir mediante la configuración).
* `contextTokens` en el almacén es un valor estimado/de reporte en tiempo de ejecución; no lo consideres una garantía estricta.

Para más información, consulta [/token-use](/es/token-use).

***

<div id="when-auto-compaction-happens-pi-runtime">
  ## Compactación: en qué consiste
</div>

La compactación resume las partes más antiguas de la conversación en una entrada `compaction` persistente en la transcripción y mantiene intactos los mensajes recientes.

Después de la compactación, las interacciones futuras verán:

* El resumen de compactación
* Los mensajes posteriores a `firstKeptEntryId`

La compactación es **persistente** (a diferencia de la poda de sesión). Consulta [/concepts/session-pruning](/es/concepts/session-pruning).

***

<div id="compaction-settings-reservetokens-keeprecenttokens">
  ## Cuándo ocurre la compactación automática (runtime de Pi)
</div>

En el agente Pi integrado, la compactación automática se desencadena en dos casos:

1. **Recuperación por desbordamiento**: el modelo devuelve un error de desbordamiento de contexto → se compacta → se reintenta.
2. **Mantenimiento del umbral**: después de un turno exitoso, cuando:

`contextTokens > contextWindow - reserveTokens`

Donde:

* `contextWindow` es la ventana de contexto del modelo
* `reserveTokens` es el margen reservado para los prompts + la siguiente salida del modelo

Estas son las semánticas del runtime de Pi (OpenClaw consume los eventos, pero Pi decide cuándo compactar).

***

<div id="user-visible-surfaces">
  ## Configuración de compactación (`reserveTokens`, `keepRecentTokens`)
</div>

Los parámetros de compactación de Pi se encuentran en la configuración de Pi:

```json5
{
  compaction: {
    enabled: true,
    reserveTokens: 16384,
    keepRecentTokens: 20000
  }
}
```

OpenClaw también aplica un umbral mínimo de seguridad para ejecuciones embebidas:

* Si `compaction.reserveTokens < reserveTokensFloor`, OpenClaw lo ajusta.
* El umbral mínimo predeterminado es de `20000` tokens.
* Establece `agents.defaults.compaction.reserveTokensFloor: 0` para desactivar el umbral.
* Si ya es más alto, OpenClaw lo deja como está.

Motivo: dejar suficiente margen para las tareas de mantenimiento multiturno (como escrituras en memoria) antes de que la compactación sea inevitable.

Implementación: `ensurePiCompactionReserveTokens()` en `src/agents/pi-settings.ts`
(se llama desde `src/agents/pi-embedded-runner.ts`).

***

<div id="silent-housekeeping-no_reply">
  ## Elementos visibles para el usuario
</div>

Puedes observar la compactación y el estado de la sesión mediante:

* `/status` (en cualquier sesión de chat)
* `openclaw status` (CLI)
* `openclaw sessions` / `sessions --json`
* Modo detallado (verbose): `🧹 Auto-compaction complete` + número de compactaciones

***

<div id="pre-compaction-memory-flush-implemented">
  ## Mantenimiento silencioso (`NO_REPLY`)
</div>

OpenClaw permite turnos “silenciosos” para tareas en segundo plano en las que el usuario no debe ver resultados intermedios.

Convención:

* El asistente comienza su salida con `NO_REPLY` para indicar “no entregar una respuesta al usuario”.
* OpenClaw elimina o suprime esto en la capa de entrega.

A partir de `2026.1.10`, OpenClaw también suprime el **streaming de borrador/escritura** cuando un fragmento parcial comienza con `NO_REPLY`, de modo que las operaciones silenciosas no expongan salida parcial a mitad de turno.

***

<div id="troubleshooting-checklist">
  ## “Vaciado de memoria” previo a la compactación (implementado)
</div>

Objetivo: antes de que ocurra la compactación automática, ejecutar un turno silencioso del agente que escriba estado duradero en disco (p. ej., `memory/YYYY-MM-DD.md` en el espacio de trabajo del agente) para que la compactación no pueda borrar contexto crítico.

OpenClaw usa el enfoque de **vaciado previo al umbral**:

1. Monitorizar el uso de contexto de la sesión.
2. Cuando cruza un “umbral suave” (por debajo del umbral de compactación de Pi), ejecutar una directiva silenciosa
   de “escribir memoria ahora” para el agente.
3. Usar `NO_REPLY` para que el usuario no vea nada.

Config (`agents.defaults.compaction.memoryFlush`):

* `enabled` (valor predeterminado: `true`)
* `softThresholdTokens` (valor predeterminado: `4000`)
* `prompt` (mensaje de usuario para el turno de vaciado)
* `systemPrompt` (prompt de sistema adicional añadido para el turno de vaciado)

Notas:

* El prompt y el prompt de sistema predeterminados incluyen una indicación `NO_REPLY` para suprimir la entrega.
* El vaciado se ejecuta una vez por ciclo de compactación (registrado en `sessions.json`).
* El vaciado se ejecuta solo para sesiones Pi embebidas (los backends de la CLI lo omiten).
* El vaciado se omite cuando el espacio de trabajo de la sesión es de solo lectura (`workspaceAccess: "ro"` o `"none"`).
* Consulta [Memory](/es/concepts/memory) para la estructura de archivos del espacio de trabajo y los patrones de escritura.

Pi también expone un hook `session_before_compact` en la API de extensiones, pero la lógica de vaciado de OpenClaw reside actualmente en el lado del Gateway.

***

## Lista de comprobación para la resolución de problemas

* ¿Clave de sesión incorrecta? Empieza con [/concepts/session](/es/concepts/session) y confirma la `sessionKey` en `/status`.
* ¿Desajuste entre store y transcripción? Confirma el host del Gateway y la ruta del store a partir de `openclaw status`.
* ¿Compaction excesiva? Revisa:
  * la ventana de contexto del modelo (demasiado pequeña)
  * los ajustes de compaction (`reserveTokens` demasiado alto para la ventana del modelo puede provocar compactación anticipada)
  * sobrecarga de resultados de herramientas: habilita/ajusta la poda de sesiones
* ¿Se están filtrando turnos silenciosos? Confirma que la respuesta comience con `NO_REPLY` (token exacto) y que estés en una versión que incluya la corrección para la supresión de streaming.