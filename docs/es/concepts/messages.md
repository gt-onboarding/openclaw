---
title: Mensajes
summary: "Flujo de mensajes, sesiones, encolado y visibilidad del razonamiento"
read_when:
  - Explicando cómo los mensajes entrantes se convierten en respuestas
  - Aclarando sesiones, modos de encolado o el comportamiento de streaming
  - Documentando la visibilidad del razonamiento y las implicaciones de su uso
---

<div id="messages">
  # Mensajes
</div>

Esta página explica cómo OpenClaw gestiona los mensajes entrantes, las sesiones, el encolamiento,
el streaming y la visibilidad del razonamiento.

<div id="message-flow-high-level">
  ## Flujo de mensajes (visión de alto nivel)
</div>

```
Inbound message
  -> routing/bindings -> session key
  -> queue (if a run is active)
  -> agent run (streaming + tools)
  -> outbound replies (channel limits + chunking)
```

Los controles principales se encuentran en la configuración:

* `messages.*` para prefijos, encolado y comportamiento de grupos.
* `agents.defaults.*` para valores predeterminados de streaming por bloques y segmentación.
* Reglas específicas por canal (`channels.whatsapp.*`, `channels.telegram.*`, etc.) para límites máximos y opciones de activación/desactivación de streaming.

Consulta [Configuración](/es/gateway/configuration) para ver el esquema completo.

<div id="inbound-dedupe">
  ## Eliminación de duplicados de entrada
</div>

Los canales pueden volver a entregar el mismo mensaje después de una reconexión. OpenClaw mantiene
una caché de vida corta indexada por canal/cuenta/par/sesión/ID de mensaje para que las entregas
duplicadas no provoquen una nueva ejecución del agente.

<div id="inbound-debouncing">
  ## Debouncing de entrada
</div>

Los mensajes consecutivos rápidos del **mismo remitente** se pueden agrupar en un
único turno de agente mediante `messages.inbound`. El debouncing se aplica por canal + conversación
y usa el mensaje más reciente para los hilos de respuesta y los ID de réplica.

Configuración (valor predeterminado global + anulaciones por canal):

```json5
{
  messages: {
    inbound: {
      debounceMs: 2000,
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500
      }
    }
  }
}
```

Notas:

* El *debounce* se aplica únicamente a mensajes **de texto**; el contenido multimedia/adjuntos se envía al instante.
* Los comandos de control omiten el *debounce* para seguir siendo operaciones aisladas.

<div id="sessions-and-devices">
  ## Sesiones y dispositivos
</div>

Las sesiones pertenecen al Gateway, no a los clientes.

* Los chats directos se agrupan en la clave de sesión principal del agente.
* Los grupos/canales tienen sus propias claves de sesión.
* El almacén de sesiones y las transcripciones residen en el host del Gateway.

Varios dispositivos/canales pueden asociarse a la misma sesión, pero el historial no se
sincroniza completamente en todos los clientes. Recomendación: usa un dispositivo
principal para conversaciones largas para evitar que el contexto diverja. Control UI y la TUI siempre muestran la transcripción de la sesión respaldada por el Gateway, por lo que constituyen la fuente de verdad.

Detalles: [Gestión de sesiones](/es/concepts/session).

<div id="inbound-bodies-and-history-context">
  ## Cuerpos entrantes y contexto de historial
</div>

OpenClaw separa el **cuerpo del prompt** del **cuerpo del comando**:

* `Body`: texto del prompt enviado al agente. Esto puede incluir envolturas del canal y
  envolturas de historial opcionales.
* `CommandBody`: texto sin procesar del usuario para el análisis de directivas/comandos.
* `RawBody`: alias heredado de `CommandBody` (se mantiene por compatibilidad).

Cuando un canal proporciona historial, utiliza una envoltura compartida:

* `[Chat messages since your last reply - for context]`
* `[Current message - respond to this]`

Para **chats no directos** (grupos/canales/salas), al **cuerpo del mensaje actual** se le antepone la
etiqueta del remitente (el mismo estilo usado para las entradas del historial). Esto mantiene los mensajes
en tiempo real y los mensajes en cola/historial coherentes en el prompt del agente.

Los búferes de historial son **solo de pendientes**: incluyen mensajes de grupo que *no*
dispararon una ejecución (por ejemplo, mensajes condicionados a mención) y **excluyen** los mensajes
que ya están en la transcripción de la sesión.

La eliminación de directivas solo se aplica a la sección del **mensaje actual**, de modo que el historial
permanece intacto. Los canales que envuelvan historial deben establecer `CommandBody` (o
`RawBody`) al texto original del mensaje y mantener `Body` como el prompt combinado.
Los búferes de historial son configurables mediante `messages.groupChat.historyLimit` (valor
predeterminado global) y valores específicos por canal como `channels.slack.historyLimit` o
`channels.telegram.accounts.<id>.historyLimit` (establece `0` para desactivar).

<div id="queueing-and-followups">
  ## Encolado y seguimientos
</div>

Si ya hay una ejecución activa, los mensajes entrantes se pueden encolar, dirigir a la
ejecución actual o recopilar para un turno de seguimiento.

* Configura mediante `messages.queue` (y `messages.queue.byChannel`).
* Modos: `interrupt`, `steer`, `followup`, `collect`, además de variantes con backlog.

Detalles: [Encolado](/es/concepts/queue).

<div id="streaming-chunking-and-batching">
  ## Transmisión, fragmentación y procesamiento por lotes
</div>

La transmisión por bloques envía respuestas parciales a medida que el modelo genera bloques de texto.
La fragmentación respeta los límites de texto del canal y evita dividir bloques de código con cercas (fenced code).

Ajustes clave:

* `agents.defaults.blockStreamingDefault` (`on|off`, valor predeterminado: off)
* `agents.defaults.blockStreamingBreak` (`text_end|message_end`)
* `agents.defaults.blockStreamingChunk` (`minChars|maxChars|breakPreference`)
* `agents.defaults.blockStreamingCoalesce` (procesamiento por lotes basado en inactividad)
* `agents.defaults.humanDelay` (pausa similar a la humana entre respuestas por bloques)
* Anulaciones por canal: `*.blockStreaming` y `*.blockStreamingCoalesce` (los canales que no sean de Telegram requieren explícitamente `*.blockStreaming: true`)

Detalles: [Transmisión + fragmentación](/es/concepts/streaming).

<div id="reasoning-visibility-and-tokens">
  ## Visibilidad del razonamiento y tokens
</div>

OpenClaw puede exponer u ocultar el razonamiento del modelo:

* `/reasoning on|off|stream` controla la visibilidad.
* El contenido de razonamiento sigue contando para el consumo de tokens cuando lo genera el modelo.
* Telegram admite el flujo de razonamiento en la burbuja de borrador.

Detalles: [Directivas de pensamiento y razonamiento](/es/tools/thinking) y [Uso de tokens](/es/token-use).

<div id="prefixes-threading-and-replies">
  ## Prefijos, hilos y respuestas
</div>

El formato de los mensajes salientes está centralizado en `messages`:

* `messages.responsePrefix` (prefijo de salida) y `channels.whatsapp.messagePrefix` (prefijo de entrada de WhatsApp)
* Encadenamiento de respuestas mediante `replyToMode` y valores predeterminados por canal

Detalles: [Configuración](/es/gateway/configuration#messages) y documentación de canales.