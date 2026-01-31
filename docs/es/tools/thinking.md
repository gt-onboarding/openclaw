---
title: Pensamiento
summary: "Sintaxis de directivas para /think y /verbose y cómo afectan al razonamiento del modelo"
read_when:
  - Ajustar la interpretación o los valores predeterminados de las directivas /think o /verbose
---

<div id="thinking-levels-think-directives">
  # Niveles de pensamiento (directivas /think)
</div>

<div id="what-it-does">
  ## Qué hace
</div>

* Directiva en línea en cualquier cuerpo de mensaje entrante: `/t <level>`, `/think:<level>` o `/thinking <level>`.
* Niveles (alias): `off | minimal | low | medium | high | xhigh` (solo modelos GPT-5.2 + Codex)
  * minimal → “think”
  * low → “think hard”
  * medium → “think harder”
  * high → “ultrathink” (presupuesto máximo)
  * xhigh → “ultrathink+” (solo modelos GPT-5.2 + Codex)
  * `highest`, `max` se asignan a `high`.
* Notas del proveedor:
  * Z.AI (`zai/*`) solo admite pensamiento binario (`on`/`off`). Cualquier nivel distinto de `off` se trata como `on` (asignado a `low`).

<div id="resolution-order">
  ## Orden de resolución
</div>

1. Directiva en línea en el mensaje (se aplica solo a ese mensaje).
2. Anulación de la sesión (establecida enviando un mensaje solo con la directiva).
3. Valor predeterminado global (`agents.defaults.thinkingDefault` en la configuración).
4. Comportamiento de respaldo: bajo para modelos con capacidad de razonamiento; desactivado en caso contrario.

<div id="setting-a-session-default">
  ## Configurar un valor predeterminado de sesión
</div>

* Envía un mensaje que sea **solo** la directiva (se permiten espacios en blanco), por ejemplo `/think:medium` o `/t high`.
* Se mantiene para la sesión actual (por remitente, de forma predeterminada); se borra con `/think:off` o al restablecer la sesión por inactividad.
* Se envía una respuesta de confirmación (`Thinking level set to high.` / `Thinking disabled.`). Si el nivel no es válido (por ejemplo `/thinking big`), el comando se rechaza con una indicación y el estado de la sesión permanece sin cambios.
* Envía `/think` (o `/think:`) sin argumentos para ver el nivel de razonamiento actual.

<div id="application-by-agent">
  ## Aplicación por agente
</div>

* **Embedded Pi**: el nivel resuelto se pasa al runtime en proceso del agente Pi.

<div id="verbose-directives-verbose-or-v">
  ## Directivas de verbosidad (/verbose o /v)
</div>

* Niveles: `on` (mínimo) | `full` | `off` (predeterminado).
* Un mensaje que solo contenga la directiva activa o desactiva la verbosidad de la sesión y responde `Verbose logging enabled.` / `Verbose logging disabled.`; los niveles no válidos devuelven un aviso sin cambiar el estado.
* `/verbose off` almacena una anulación explícita de la configuración de la sesión; bórrala desde la UI de Sessions eligiendo `inherit`.
* Una directiva en línea solo afecta a ese mensaje; en caso contrario se aplican los valores predeterminados de sesión/globales.
* Envía `/verbose` (o `/verbose:`) sin argumento para ver el nivel de verbosidad actual.
* Cuando verbose está activado, los agentes que emiten resultados de herramientas estructurados (Pi, otros agentes JSON) envían cada llamada de herramienta como su propio mensaje solo de metadatos, con el prefijo `<emoji> <tool-name>: <arg>` cuando esté disponible (ruta/comando). Estos resúmenes de herramientas se envían tan pronto como se inicia cada herramienta (burbujas separadas), no como fragmentos en streaming.
* Cuando verbose está en `full`, las salidas de las herramientas también se reenvían tras completarse (burbuja separada, truncada a una longitud segura). Si cambias `/verbose on|full|off` mientras una ejecución está en curso, las burbujas de herramientas posteriores respetan la nueva configuración.

<div id="reasoning-visibility-reasoning">
  ## Visibilidad del razonamiento (/reasoning)
</div>

* Niveles: `on|off|stream`.
* Un mensaje que solo contenga la directiva alterna si se muestran los bloques de razonamiento en las respuestas.
* Cuando está habilitado, el razonamiento se envía como un **mensaje separado** con el prefijo `Reasoning:`.
* `stream` (solo Telegram): transmite el razonamiento en la burbuja de borrador de Telegram mientras se genera la respuesta y luego envía la respuesta final sin razonamiento.
* Alias: `/reason`.
* Envía `/reasoning` (o `/reasoning:`) sin argumentos para ver el nivel de razonamiento actual.

<div id="related">
  ## Relacionado
</div>

* La documentación del modo elevado está en [Modo elevado](/es/tools/elevated).

<div id="heartbeats">
  ## Latidos
</div>

* El cuerpo de la sonda de latido es el prompt de latido configurado (predeterminado: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`). Las directivas inline en un mensaje de latido se aplican como de costumbre (pero evita cambiar los valores predeterminados de la sesión desde los latidos).
* De forma predeterminada, la entrega del latido incluye solo el payload final. Para enviar también el mensaje separado `Reasoning:` (cuando esté disponible), establece `agents.defaults.heartbeat.includeReasoning: true` o, por agente, `agents.list[].heartbeat.includeReasoning: true`.

<div id="web-chat-ui">
  ## UI de chat web
</div>

* El selector de nivel de pensamiento del chat web refleja el nivel almacenado de la sesión desde el almacén/configuración de sesiones de entrada cuando se carga la página.
* Elegir otro nivel se aplica únicamente al siguiente mensaje (`thinkingOnce`); tras enviar, el selector vuelve al nivel de sesión almacenado.
* Para cambiar el valor predeterminado de la sesión, envía una directiva `/think:<level>` (como antes); el selector la reflejará después de la siguiente recarga de la página.