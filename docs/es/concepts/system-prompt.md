---
title: Prompt del sistema
summary: "Qué contiene el prompt del sistema de OpenClaw y cómo se construye"
read_when:
  - Editar el texto del prompt del sistema, la lista de herramientas o las secciones de tiempo/latido
  - Cambiar el comportamiento de arranque del espacio de trabajo o la inyección de habilidades
---

<div id="system-prompt">
  # Prompt del sistema
</div>

OpenClaw construye un prompt de sistema personalizado para cada ejecución de un agente. El prompt es **propiedad de OpenClaw** y no utiliza el prompt predeterminado de p-coding-agent.

El prompt lo ensambla OpenClaw y se inyecta en cada ejecución de agente.

<div id="structure">
  ## Estructura
</div>

El prompt es deliberadamente compacto y utiliza secciones fijas:

* **Tooling**: lista actual de herramientas + descripciones breves.
* **Skills** (cuando estén disponibles): indica al modelo cómo cargar instrucciones de habilidades según se necesiten.
* **OpenClaw Self-Update**: cómo ejecutar `config.apply` y `update.run`.
* **Workspace**: directorio de trabajo (`agents.defaults.workspace`).
* **Documentation**: ruta local a la documentación de OpenClaw (repo o paquete npm) y cuándo leerla.
* **Workspace Files (injected)**: indica que los archivos de arranque (bootstrap) se incluyen a continuación.
* **Sandbox** (cuando está habilitado): indica el entorno de ejecución en sandbox, las rutas del sandbox y si la ejecución con privilegios elevados está disponible.
* **Current Date &amp; Time**: hora local del usuario, zona horaria y formato de hora.
* **Reply Tags**: sintaxis opcional de etiquetas de respuesta para proveedores compatibles.
* **Heartbeats**: prompt de latido y comportamiento de confirmación (ack).
* **Runtime**: host, SO, nodo, modelo, raíz del repo (cuando se detecta), nivel de razonamiento (una línea).
* **Reasoning**: nivel de visibilidad actual + indicación sobre el conmutador /reasoning.

<div id="prompt-modes">
  ## Modos de prompt
</div>

OpenClaw puede renderizar prompts de sistema más pequeños para subagentes. El tiempo de ejecución establece un
`promptMode` para cada ejecución (no es una configuración visible para el usuario):

* `full` (predeterminado): incluye todas las secciones anteriores.
* `minimal`: se usa para subagentes; omite **Habilidades**, **Recuperación de memoria**, **Autoactualización de OpenClaw**, **Alias de modelos**, **Identidad de usuario**, **Etiquetas de respuesta**, **Mensajería**, **Respuestas silenciosas** y **Latidos**. Las herramientas, el espacio de trabajo, la sandbox, la fecha y hora actuales (cuando se conocen), el tiempo de ejecución y el contexto inyectado permanecen disponibles.
* `none`: devuelve solo la línea de identidad base.

Cuando `promptMode=minimal`, los prompts adicionales inyectados se marcan como **Contexto de subagente**
en lugar de **Contexto de chat grupal**.

<div id="workspace-bootstrap-injection">
  ## Inyección de bootstrap en el espacio de trabajo
</div>

Los archivos de bootstrap se recortan y se agregan bajo **Contexto del proyecto** para que el modelo vea el contexto de identidad y perfil sin necesitar read explícitos:

* `AGENTS.md`
* `SOUL.md`
* `TOOLS.md`
* `IDENTITY.md`
* `USER.md`
* `HEARTBEAT.md`
* `BOOTSTRAP.md` (solo en espacios de trabajo recién creados)

Los archivos grandes se truncan con un marcador. El tamaño máximo por archivo se controla con
`agents.defaults.bootstrapMaxChars` (valor predeterminado: 20000). Si faltan archivos, se inyecta un
marcador corto indicando la ausencia del archivo.

Los hooks internos pueden interceptar este paso mediante `agent:bootstrap` para modificar o reemplazar
los archivos de bootstrap inyectados (por ejemplo, sustituyendo `SOUL.md` por una personalidad alternativa).

Para inspeccionar cuánto aporta cada archivo inyectado (contenido original frente al inyectado, truncamiento y sobrecarga del esquema de herramientas), usa `/context list` o `/context detail`. Consulta [Contexto](/es/concepts/context).

<div id="time-handling">
  ## Gestión de fecha y hora
</div>

El prompt del sistema incluye una sección dedicada de **Fecha y hora actuales** cuando se conoce la zona horaria del usuario. Para mantener el prompt estable en caché, ahora solo incluye la **zona horaria** (sin reloj dinámico ni formato horario).

Usa `session_status` cuando el agente necesite la hora actual; la tarjeta de estado incluye una línea con la marca de tiempo.

Configúralo con:

* `agents.defaults.userTimezone`
* `agents.defaults.timeFormat` (`auto` | `12` | `24`)

Consulta [Fecha y hora](/es/date-time) para obtener todos los detalles del comportamiento.

<div id="skills">
  ## Habilidades
</div>

Cuando hay habilidades elegibles, OpenClaw inyecta una **lista compacta de habilidades disponibles**
(`formatSkillsForPrompt`) que incluye la **ruta de archivo** de cada habilidad. El
prompt indica al modelo que use `read` para cargar el archivo SKILL.md en la ubicación
indicada (espacio de trabajo, gestionado o empaquetado). Si no hay habilidades elegibles, se
omite la sección Habilidades.

```
<available_skills>
  <skill>
    <name>...</name>
    <description>...</description>
    <location>...</location>
  </skill>
</available_skills>
```

Esto mantiene reducido el prompt base y, aun así, permite el uso dirigido de habilidades.

<div id="documentation">
  ## Documentación
</div>

Cuando está disponible, el prompt del sistema incluye una sección de **Documentación** que apunta al
directorio local de documentación de OpenClaw (ya sea `docs/` en el espacio de trabajo del repositorio o la
documentación del paquete npm incluido) y también menciona el espejo público, el repositorio de código fuente, el Discord de la comunidad y
ClawHub (https://clawhub.com) para el descubrimiento de habilidades. El prompt indica al modelo que consulte primero la documentación local
para el comportamiento, comandos, configuración o arquitectura de OpenClaw, y que ejecute
`openclaw status` por sí mismo cuando sea posible (pidiéndoselo al usuario solo cuando no tenga acceso).