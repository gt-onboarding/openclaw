---
title: Bucle de agente
summary: "Ciclo de vida del bucle de agente, flujos y semántica de espera"
read_when:
  - Necesitas una guía precisa del bucle de agente o de los eventos de su ciclo de vida
---

<div id="agent-loop-openclaw">
  # Bucle de Agente (OpenClaw)
</div>

Un bucle de agente es la ejecución “real” completa de un agente: recepción → ensamblaje de contexto → inferencia del modelo → ejecución de herramientas → respuestas en streaming → persistencia. Es la ruta canónica que convierte un mensaje en acciones y en una respuesta final, manteniendo coherente el estado de la sesión.

En OpenClaw, un bucle es una única ejecución serializada por sesión que emite eventos de ciclo de vida y de streaming a medida que el modelo “piensa”, invoca herramientas y transmite la salida. Este documento explica cómo está conectado de extremo a extremo ese bucle auténtico.

<div id="entry-points">
  ## Puntos de entrada
</div>

* RPC del Gateway: `agent` y `agent.wait`.
* CLI: comando `agent`.

<div id="how-it-works-high-level">
  ## Cómo funciona (a alto nivel)
</div>

1. La RPC `agent` valida los parámetros, resuelve la sesión (`sessionKey`/`sessionId`), persiste los metadatos de la sesión y devuelve `{ runId, acceptedAt }` de inmediato.
2. `agentCommand` ejecuta el agente:
   * resuelve el modelo y los valores predeterminados de thinking/verbose
   * carga el snapshot de habilidades
   * llama a `runEmbeddedPiAgent` (runtime de pi-agent-core)
   * emite **lifecycle end/error** si el bucle embebido no emite uno
3. `runEmbeddedPiAgent`:
   * serializa ejecuciones mediante colas por sesión y colas globales
   * resuelve el modelo y el perfil de autenticación y construye la sesión pi
   * se suscribe a eventos pi y transmite deltas de assistant/tool
   * aplica un timeout -&gt; aborta la ejecución si se excede
   * devuelve payloads y metadatos de uso
4. `subscribeEmbeddedPiSession` hace de puente entre eventos de pi-agent-core y el stream `agent` de OpenClaw:
   * eventos de herramienta =&gt; `stream: "tool"`
   * deltas de assistant =&gt; `stream: "assistant"`
   * eventos de ciclo de vida =&gt; `stream: "lifecycle"` (`phase: "start" | "end" | "error"`)
5. `agent.wait` usa `waitForAgentJob`:
   * espera **lifecycle end/error** para `runId`
   * devuelve `{ status: ok|error|timeout, startedAt, endedAt, error? }`

<div id="queueing-concurrency">
  ## Encolamiento + concurrencia
</div>

* Las ejecuciones se serializan por clave de sesión (session lane) y opcionalmente a través de una lane global.
* Esto evita condiciones de carrera entre herramientas y sesiones y mantiene coherente el historial de la sesión.
* Los canales de mensajería pueden elegir modos de encolamiento (collect/steer/followup) que alimentan este sistema de lanes.
  Consulta [Command Queue](/es/concepts/queue).

<div id="session-workspace-preparation">
  ## Preparación de sesión + espacio de trabajo
</div>

* Se resuelve y crea el espacio de trabajo; las ejecuciones en sandbox pueden redirigirse a una raíz de espacio de trabajo en la sandbox.
* Las habilidades se cargan (o se reutilizan desde una snapshot) y se inyectan en el entorno y en el prompt.
* Los archivos de arranque/contexto se resuelven y se inyectan en el informe del prompt del sistema.
* Se adquiere un bloqueo de escritura para la sesión; `SessionManager` se abre y se prepara antes de iniciar el streaming.

<div id="prompt-assembly-system-prompt">
  ## Ensamblaje del prompt + prompt del sistema
</div>

* El prompt del sistema se construye a partir del prompt base de OpenClaw, el prompt de habilidades, el contexto de arranque (bootstrap context) y las anulaciones por ejecución.
* Se aplican límites específicos del modelo y reservas de tokens para la compactación.
* Consulta [Prompt del sistema](/es/concepts/system-prompt) para ver lo que ve el modelo.

<div id="hook-points-where-you-can-intercept">
  ## Puntos de enganche (donde puedes interceptar)
</div>

OpenClaw dispone de dos sistemas de hooks:

* **Hooks internos** (hooks del Gateway): scripts basados en eventos para comandos y eventos de ciclo de vida.
* **Hooks de complementos**: puntos de extensión dentro del ciclo de vida del agente/herramienta y del pipeline del Gateway.

<div id="internal-hooks-gateway-hooks">
  ### Hooks internos (hooks del Gateway)
</div>

* **`agent:bootstrap`**: se ejecuta mientras se generan los archivos de bootstrap antes de que se finalice el mensaje de sistema.
  Úsalo para añadir o eliminar archivos de contexto de bootstrap.
* **Hooks de comandos**: eventos de comandos `/new`, `/reset`, `/stop` y otros (consulta la documentación de Hooks).

Consulta [Hooks](/es/hooks) para detalles de configuración y ejemplos.

<div id="plugin-hooks-agent-gateway-lifecycle">
  ### Ganchos de complementos (ciclo de vida de agente + Gateway)
</div>

Estos se ejecutan dentro del bucle del agente o del pipeline del Gateway:

* **`before_agent_start`**: inyecta contexto o reemplaza el prompt del sistema antes de que comience la ejecución.
* **`agent_end`**: inspecciona la lista final de mensajes y los metadatos de la ejecución una vez completada.
* **`before_compaction` / `after_compaction`**: observa o anota los ciclos de compactación.
* **`before_tool_call` / `after_tool_call`**: intercepta parámetros/resultados de herramientas.
* **`tool_result_persist`**: transforma de forma sincrónica los resultados de herramientas antes de que se escriban en la transcripción de la sesión.
* **`message_received` / `message_sending` / `message_sent`**: ganchos de mensajes entrantes y salientes.
* **`session_start` / `session_end`**: límites del ciclo de vida de la sesión.
* **`gateway_start` / `gateway_stop`**: eventos del ciclo de vida del Gateway.

Consulta [Complementos](/es/plugin#plugin-hooks) para obtener información sobre la API de ganchos y los detalles de registro.

<div id="streaming-partial-replies">
  ## Streaming + respuestas parciales
</div>

* Los deltas del asistente se transmiten desde pi-agent-core y se emiten como eventos `assistant`.
* El streaming por bloques puede emitir respuestas parciales ya sea en `text_end` o `message_end`.
* El streaming de razonamiento se puede emitir como un flujo separado o como respuestas por bloques.
* Consulta [Streaming](/es/concepts/streaming) para el comportamiento de fragmentación (chunking) y de respuestas por bloques.

<div id="tool-execution-messaging-tools">
  ## Ejecución de herramientas + herramientas de mensajería
</div>

* Los eventos de inicio/actualización/fin de herramienta se emiten en el stream `tool`.
* Los resultados de las herramientas se depuran en cuanto a tamaño y cargas útiles de imágenes antes de registrarlos/emitiros.
* Las operaciones `send` de las herramientas de mensajería se registran para suprimir confirmaciones duplicadas del asistente.

<div id="reply-shaping-suppression">
  ## Modelado y supresión de respuestas
</div>

* Las cargas útiles finales se ensamblan a partir de:
  * texto del asistente (y razonamiento opcional)
  * resúmenes en línea de herramientas (cuando está en modo detallado y permitido)
  * texto de error del asistente cuando el modelo produce un error
* `NO_REPLY` se trata como un token silencioso y se filtra de las cargas útiles salientes.
* Los duplicados de herramientas de mensajería se eliminan de la lista de cargas útiles finales.
* Si no quedan cargas útiles renderizables y una herramienta produjo un error, se emite una respuesta de error de herramienta de respaldo
  (a menos que una herramienta de mensajería ya haya enviado una respuesta visible para el usuario).

<div id="compaction-retries">
  ## Compactación + reintentos
</div>

* La compactación automática emite eventos de stream `compaction` y puede activar un reintento.
* En cada reintento, los búferes en memoria y los resúmenes de herramientas se restablecen para evitar duplicar la salida.
* Consulta [Compactación](/es/concepts/compaction) para ver el pipeline de compactación.

<div id="event-streams-today">
  ## Flujos de eventos (actualmente)
</div>

* `lifecycle`: generado por `subscribeEmbeddedPiSession` (y como mecanismo de respaldo por `agentCommand`)
* `assistant`: deltas en streaming desde pi-agent-core
* `tool`: eventos de herramientas en streaming desde pi-agent-core

<div id="chat-channel-handling">
  ## Gestión de canales de chat
</div>

* Los deltas del asistente se almacenan en búfer como mensajes de chat `delta`.
* Se emite un mensaje de chat `final` al **finalizar el ciclo de vida o producirse un error**.

<div id="timeouts">
  ## Tiempos de espera
</div>

* `agent.wait` predeterminado: 30s (solo la espera). El parámetro `timeoutMs` lo sobrescribe.
* Tiempo de ejecución del agente: `agents.defaults.timeoutSeconds` predeterminado 600s; se aplica en el temporizador de cancelación de `runEmbeddedPiAgent`.

<div id="where-things-can-end-early">
  ## Dónde las cosas pueden terminar antes de tiempo
</div>

* Tiempo de espera del agente (abortar)
* AbortSignal (cancelar)
* Desconexión del Gateway o tiempo de espera de RPC
* Tiempo de espera de `agent.wait` (solo espera, no detiene al agente)