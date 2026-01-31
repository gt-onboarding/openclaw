---
title: Agente
summary: "Tiempo de ejecución del Agente (pi-mono integrado), contrato del espacio de trabajo e inicialización de la sesión"
read_when:
  - Al cambiar el tiempo de ejecución del agente, la inicialización del espacio de trabajo o el comportamiento de la sesión
---

<div id="agent-runtime">
  # Tiempo de ejecución del agente 🤖
</div>

OpenClaw ejecuta una única instancia de tiempo de ejecución de agente integrada basada en **pi-mono**.

<div id="workspace-required">
  ## Espacio de trabajo (obligatorio)
</div>

OpenClaw usa un único directorio de espacio de trabajo del agente (`agents.defaults.workspace`) como el **único** directorio de trabajo (`cwd`) del agente para herramientas y contexto.

Recomendado: usa `openclaw setup` para crear `~/.openclaw/openclaw.json` si no existe e inicializar los archivos del espacio de trabajo.

Guía completa de la estructura del espacio de trabajo + copia de seguridad: [Espacio de trabajo del agente](/es/concepts/agent-workspace)

Si `agents.defaults.sandbox` está habilitado, las sesiones no principales pueden anular esto con
espacios de trabajo por sesión bajo `agents.defaults.sandbox.workspaceRoot` (consulta
[Configuración del Gateway](/es/gateway/configuration)).

<div id="bootstrap-files-injected">
  ## Archivos de arranque (inyectados)
</div>

Dentro de `agents.defaults.workspace`, OpenClaw espera estos archivos editables por el usuario:

* `AGENTS.md` — instrucciones de funcionamiento + “memoria”
* `SOUL.md` — personalidad, límites, tono
* `TOOLS.md` — notas de herramientas mantenidas por el usuario (por ejemplo, `imsg`, `sag`, convenciones)
* `BOOTSTRAP.md` — ritual único de primera ejecución (se elimina tras completarse)
* `IDENTITY.md` — nombre del agente / estilo / emoji
* `USER.md` — perfil de usuario + forma de tratamiento preferida

En el primer turno de una nueva sesión, OpenClaw inyecta el contenido de estos archivos directamente en el contexto del agente.

Los archivos en blanco se omiten. Los archivos grandes se recortan y se truncan con un marcador para que los prompts se mantengan ligeros (lee el archivo para ver el contenido completo).

Si falta un archivo, OpenClaw inyecta una única línea de marcador de “archivo ausente” (y `openclaw setup` creará una plantilla segura predeterminada).

`BOOTSTRAP.md` solo se crea para un **espacio de trabajo completamente nuevo** (sin otros archivos de arranque presentes). Si lo borras después de completar el ritual, no debería recrearse en reinicios posteriores.

Para desactivar por completo la creación de archivos de arranque (para espacios de trabajo ya preconfigurados), configura:

```json5
{ agent: { skipBootstrap: true } }
```

<div id="built-in-tools">
  ## Herramientas integradas
</div>

Las herramientas principales (read/exec/edit/write y las herramientas del sistema relacionadas) siempre están disponibles,
sujetas a la política de herramientas. `apply_patch` es opcional y se controla mediante
`tools.exec.applyPatch`. `TOOLS.md` **no** determina qué herramientas existen; es
una guía sobre cómo quieres que se utilicen.

<div id="skills">
  ## Habilidades
</div>

OpenClaw carga habilidades desde tres ubicaciones (el espacio de trabajo prevalece en caso de conflicto de nombres):

* Incluidas (se entregan con la instalación)
* Gestionadas/locales: `~/.openclaw/skills`
* Espacio de trabajo: `<workspace>/skills`

Las habilidades pueden estar condicionadas por la configuración o las variables de entorno (consulta `skills` en [Configuración del Gateway](/es/gateway/configuration)).

<div id="pi-mono-integration">
  ## integración con pi-mono
</div>

OpenClaw reutiliza partes del código base de pi-mono (modelos/herramientas), pero **la gestión de sesiones, el descubrimiento y la conexión de herramientas son responsabilidad de OpenClaw**.

* No hay tiempo de ejecución de agente de pi-coding.
* No se tienen en cuenta las configuraciones `~/.pi/agent` ni `<workspace>/.pi`.

<div id="sessions">
  ## Sesiones
</div>

Las transcripciones de la sesión se almacenan como JSONL en:

* `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`

El ID de la sesión es estable y lo elige OpenClaw.
Las carpetas de sesión heredadas de Pi/Tau **no** se leen.

<div id="steering-while-streaming">
  ## Control durante el streaming
</div>

Cuando el modo de cola es `steer`, los mensajes entrantes se inyectan en la ejecución actual.
La cola se comprueba **después de cada llamada a herramienta**; si hay un mensaje en cola,
se omiten las llamadas de herramienta restantes del mensaje actual del asistente (resultados
de herramienta de error con el mensaje &quot;Skipped due to queued user message.&quot;), y luego el mensaje de
usuario en cola se inyecta antes de la siguiente respuesta del asistente.

Cuando el modo de cola es `followup` o `collect`, los mensajes entrantes se mantienen hasta que
termine el turno actual y, a continuación, comienza un nuevo turno del agente con los payloads en cola.
Consulta [Queue](/es/concepts/queue) para el modo y el comportamiento de debounce/cap.

El streaming por bloques envía bloques completos del asistente tan pronto como terminan; está
**desactivado de forma predeterminada** (`agents.defaults.blockStreamingDefault: "off"`).
Ajusta el límite mediante `agents.defaults.blockStreamingBreak` (`text_end` vs `message_end`; el valor predeterminado es text&#95;end).
Controla la fragmentación suave de bloques con `agents.defaults.blockStreamingChunk` (predeterminado:
800–1200 caracteres; prefiere saltos de párrafo, luego saltos de línea y, en último lugar, oraciones).
Fusiona fragmentos transmitidos con `agents.defaults.blockStreamingCoalesce` para reducir
el spam de una sola línea (fusión basada en inactividad antes de send). Los canales que no son Telegram requieren
`*.blockStreaming: true` explícito para habilitar las respuestas por bloques.
Los resúmenes detallados de las herramientas se emiten al inicio de la herramienta (sin debounce); Control UI
transmite la salida de la herramienta a través de eventos del agente cuando está disponible.
Más detalles: [Streaming + fragmentación](/es/concepts/streaming).

<div id="model-refs">
  ## Referencias de modelos
</div>

Las referencias de modelos en la configuración (por ejemplo, `agents.defaults.model` y `agents.defaults.models`) se analizan separándolas por la **primera** `/`.

* Usa `provider/model` al configurar los modelos.
* Si el ID del modelo en sí contiene `/` (estilo OpenRouter), incluye el prefijo del proveedor (ejemplo: `openrouter/moonshotai/kimi-k2`).
* Si omites el proveedor, OpenClaw trata el valor como un alias o como un modelo del **proveedor predeterminado** (solo funciona cuando no hay `/` en el ID del modelo).

<div id="configuration-minimal">
  ## Configuración mínima
</div>

Como mínimo, establece:

* `agents.defaults.workspace`
* `channels.whatsapp.allowFrom` (altamente recomendado)

***

*Siguiente: [Chats grupales](/es/concepts/group-messages)* 🦞