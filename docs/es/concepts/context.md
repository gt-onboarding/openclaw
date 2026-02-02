---
title: Contexto
summary: "Contexto: lo que ve el modelo, cómo se construye y cómo inspeccionarlo"
read_when:
  - Quieres entender qué significa «contexto» en OpenClaw
  - Estás depurando por qué el modelo «sabe» algo (o lo ha olvidado)
  - Quieres reducir la sobrecarga de contexto (/context, /status, /compact)
---

<div id="context">
  # Contexto
</div>

El “contexto” es **todo lo que OpenClaw envía al modelo para una ejecución**. Está limitado por la **ventana de contexto** (límite de tokens) del modelo.

Modelo mental básico:

* **System prompt** (generado por OpenClaw): reglas, herramientas, lista de habilidades, hora/tiempo de ejecución y archivos del espacio de trabajo inyectados.
* **Historial de conversación**: tus mensajes + los mensajes del asistente de esta sesión.
* **Llamadas/resultados de herramientas + adjuntos**: salida de comandos, lectura de archivos, imágenes/audio, etc.

El contexto *no es lo mismo* que la “memoria”: la memoria puede almacenarse en disco y recargarse más tarde; el contexto es lo que está dentro de la ventana actual del modelo.

<div id="quick-start-inspect-context">
  ## Inicio rápido (inspeccionar contexto)
</div>

* `/status` → vista rápida de “¿qué tan llena está mi ventana?” + configuración de la sesión.
* `/context list` → qué se inyecta + tamaños aproximados (por archivo + totales).
* `/context detail` → desglose más detallado: tamaños por archivo, por esquema de herramienta, por entrada de habilidad y tamaño del mensaje del sistema (system prompt).
* `/usage tokens` → añade un pie con el uso por respuesta a las respuestas normales.
* `/compact` → resume el historial más antiguo en una entrada compacta para liberar espacio en la ventana.

Consulta también: [Comandos slash](/es/tools/slash-commands), [Uso de tokens y costes](/es/token-use), [Compactación](/es/concepts/compaction).

<div id="example-output">
  ## Ejemplo de salida
</div>

Los valores varían según el modelo, el proveedor, las políticas de herramientas y lo que haya en tu espacio de trabajo.

<div id="context-list">
  ### `/context list`
</div>

```
🧠 Desglose del contexto
Espacio de trabajo: <workspaceDir>
Bootstrap máx/archivo: 20,000 caracteres
Sandbox: mode=non-main sandboxed=false
Prompt del sistema (ejecución): 38,412 caracteres (~9,603 tok) (Contexto del Proyecto 23,901 caracteres (~5,976 tok))

Archivos del espacio de trabajo inyectados:
- AGENTS.md: OK | sin procesar 1,742 caracteres (~436 tok) | inyectado 1,742 caracteres (~436 tok)
- SOUL.md: OK | sin procesar 912 caracteres (~228 tok) | inyectado 912 caracteres (~228 tok)
- TOOLS.md: TRUNCADO | sin procesar 54,210 caracteres (~13,553 tok) | inyectado 20,962 caracteres (~5,241 tok)
- IDENTITY.md: OK | sin procesar 211 caracteres (~53 tok) | inyectado 211 caracteres (~53 tok)
- USER.md: OK | sin procesar 388 caracteres (~97 tok) | inyectado 388 caracteres (~97 tok)
- HEARTBEAT.md: FALTANTE | sin procesar 0 | inyectado 0
- BOOTSTRAP.md: OK | sin procesar 0 caracteres (~0 tok) | inyectado 0 caracteres (~0 tok)

Lista de habilidades (texto del prompt del sistema): 2,184 caracteres (~546 tok) (12 habilidades)
Herramientas: read, edit, write, exec, process, browser, message, sessions_send, …
Lista de herramientas (texto del prompt del sistema): 1,032 caracteres (~258 tok)
Esquemas de herramientas (JSON): 31,988 caracteres (~7,997 tok) (cuenta para el contexto; no se muestra como texto)
Herramientas: (igual que arriba)

Tokens de sesión (en caché): 14,250 total / ctx=32,000
```

<div id="context-detail">
  ### `/context detail`
</div>

```
🧠 Context breakdown (detailed)
…
Top skills (prompt entry size):
- frontend-design: 412 chars (~103 tok)
- oracle: 401 chars (~101 tok)
… (+10 more skills)

Top tools (schema size):
- browser: 9,812 chars (~2,453 tok)
- exec: 6,240 chars (~1,560 tok)
… (+N more tools)
```

<div id="what-counts-toward-the-context-window">
  ## Qué cuenta para la ventana de contexto
</div>

Todo lo que recibe el modelo cuenta, incluyendo:

* System prompt (todas las secciones).
* Historial de conversación.
* Llamadas a herramientas + resultados de herramientas.
* Archivos adjuntos/transcripciones (imágenes/audio/archivos).
* Resúmenes de compactación y artefactos de poda.
* “Wrappers” del proveedor o cabeceras ocultas (no visibles, pero igualmente cuentan).

<div id="how-openclaw-builds-the-system-prompt">
  ## Cómo construye OpenClaw el prompt del sistema
</div>

El prompt del sistema es **propiedad de OpenClaw** y se vuelve a generar en cada ejecución. Incluye:

* Lista de herramientas + descripciones breves.
* Lista de habilidades (solo metadatos; ver más abajo).
* Ubicación del espacio de trabajo.
* Hora (UTC + hora del usuario convertida, si está configurado).
* Metadatos de tiempo de ejecución (host/OS/model/thinking).
* Archivos de inicialización del espacio de trabajo inyectados en **Contexto del proyecto**.

Desglose completo: [Prompt del sistema](/es/concepts/system-prompt).

<div id="injected-workspace-files-project-context">
  ## Archivos del espacio de trabajo inyectados (Contexto del proyecto)
</div>

De forma predeterminada, OpenClaw inyecta un conjunto fijo de archivos del espacio de trabajo (si están presentes):

* `AGENTS.md`
* `SOUL.md`
* `TOOLS.md`
* `IDENTITY.md`
* `USER.md`
* `HEARTBEAT.md`
* `BOOTSTRAP.md` (solo en la primera ejecución)

Los archivos grandes se truncan de forma individual usando `agents.defaults.bootstrapMaxChars` (por defecto `20000` caracteres). `/context` muestra los tamaños **en bruto vs inyectados** y si se produjo truncamiento.

<div id="skills-whats-injected-vs-loaded-on-demand">
  ## Habilidades: qué se inyecta vs qué se carga bajo demanda
</div>

El mensaje del sistema incluye una **lista de habilidades** compacta (nombre + descripción + ubicación). Esta lista tiene un coste real.

Las instrucciones de la habilidad *no* se incluyen por defecto. Se espera que el modelo use `read` sobre el archivo `SKILL.md` de la habilidad **solo cuando sea necesario**.

<div id="tools-there-are-two-costs">
  ## Herramientas: hay dos costes
</div>

Las herramientas afectan al contexto de dos formas:

1. **Texto de la lista de herramientas** en el mensaje del sistema (“system prompt”, lo que ves como “Tooling”).
2. **Esquemas de herramientas** (JSON). Se envían al modelo para que pueda llamar a herramientas. Computan dentro del contexto aunque no los veas como texto plano.

`/context detail` desglosa los esquemas de herramientas más grandes para que puedas ver qué es lo que más pesa.

<div id="commands-directives-and-inline-shortcuts">
  ## Comandos, directivas y “atajos en línea”
</div>

Los comandos de barra inclinada son gestionados por el Gateway. Existen varios comportamientos:

* **Comandos independientes**: un mensaje que sea solo `/...` se ejecuta como un comando.
* **Directivas**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/model`, `/queue` se eliminan antes de que el modelo vea el mensaje.
  * Los mensajes que contienen solo directivas mantienen la configuración de la sesión.
  * Las directivas en línea en un mensaje normal actúan como sugerencias específicas para ese mensaje.
* **Atajos en línea** (solo remitentes en la lista de permitidos): ciertos tokens `/...` dentro de un mensaje normal pueden ejecutarse inmediatamente (ejemplo: “hey /status”) y se eliminan antes de que el modelo vea el texto restante.

Detalles: [Comandos de barra inclinada](/es/tools/slash-commands).

<div id="sessions-compaction-and-pruning-what-persists">
  ## Sesiones, compactación y poda (qué se conserva)
</div>

Lo que se conserva entre mensajes depende del mecanismo:

* El **historial normal** se mantiene en el registro de la sesión hasta que la política lo compacte o pode.
* La **compactación** guarda un resumen en el registro y mantiene intactos los mensajes recientes.
* La **poda** elimina resultados antiguos de herramientas del prompt *en memoria* de una ejecución, pero no modifica el registro.

Documentación: [Sesión](/es/concepts/session), [Compactación](/es/concepts/compaction), [Poda de sesión](/es/concepts/session-pruning).

<div id="what-context-actually-reports">
  ## Lo que realmente informa `/context`
</div>

`/context` prefiere el informe más reciente del prompt de sistema generado durante una ejecución (**run-built**) cuando está disponible:

* `System prompt (run)` = capturado desde la última ejecución incrustada (con capacidad de herramientas) y persistido en el almacén de sesión.
* `System prompt (estimate)` = calculado en tiempo real cuando no existe ningún informe de ejecución (o cuando se ejecuta a través de un backend de CLI que no genera el informe).

En cualquier caso, informa tamaños y componentes principales; **no** muestra el prompt de sistema completo ni los esquemas de herramientas.