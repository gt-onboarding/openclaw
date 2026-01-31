---
title: Uso de tokens
summary: "Cómo OpenClaw construye el contexto del prompt e informa sobre el uso de tokens y los costos"
read_when:
  - Explicar el uso de tokens, los costos o las ventanas de contexto
  - Depurar el crecimiento del contexto o el comportamiento de compactación
---

<div id="token-use-costs">
  # Uso de tokens y costes
</div>

OpenClaw hace un seguimiento de los **tokens**, no de los caracteres. Los tokens son específicos de cada modelo, pero la mayoría de los modelos de tipo OpenAI promedian unos 4 caracteres por token para texto en inglés.

<div id="how-the-system-prompt-is-built">
  ## Cómo se construye el prompt del sistema
</div>

OpenClaw ensambla su propio prompt del sistema en cada ejecución. Incluye:

* Lista de herramientas + descripciones breves
* Lista de habilidades (solo metadatos; las instrucciones se cargan bajo demanda con `read`)
* Instrucciones de autoactualización
* Archivos de espacio de trabajo + bootstrap (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md` cuando son nuevos). Los archivos grandes se truncan mediante `agents.defaults.bootstrapMaxChars` (valor predeterminado: 20000).
* Hora (UTC + zona horaria del usuario)
* Etiquetas de respuesta + comportamiento del latido
* Metadatos de tiempo de ejecución (host/SO/modelo/thinking)

Consulta el desglose completo en [Prompt del sistema](/es/concepts/system-prompt).

<div id="what-counts-in-the-context-window">
  ## Qué cuenta en la ventana de contexto
</div>

Todo lo que recibe el modelo cuenta para el límite de contexto:

* System prompt (todas las secciones indicadas arriba)
* Historial de conversación (mensajes de usuario + asistente)
* Llamadas a herramientas y resultados de herramientas
* Archivos adjuntos/transcripciones (imágenes, audio, archivos)
* Resúmenes de compactación y artefactos de poda
* Wrappers del proveedor o cabeceras de seguridad (no visibles, pero igualmente contados)

Para un desglose práctico (por archivo inyectado, herramientas, habilidades y tamaño del system prompt), usa `/context list` o `/context detail`. Consulta [Contexto](/es/concepts/context).

<div id="how-to-see-current-token-usage">
  ## Cómo ver el uso actual de tokens
</div>

Usa estos comandos en el chat:

* `/status` → **tarjeta de estado rica en emojis** con el modelo de la sesión, uso de contexto,
  tokens de entrada/salida de la última respuesta y **costo estimado** (solo clave API).
* `/usage off|tokens|full` → agrega un **pie con información de uso por respuesta** a cada respuesta.
  * Persiste por sesión (almacenado como `responseUsage`).
  * La autenticación OAuth **oculta el costo** (solo tokens).
* `/usage cost` → muestra un resumen local de costos a partir de los registros de sesión de OpenClaw.

Otros lugares:

* **TUI/Web TUI:** se admiten `/status` y `/usage`.
* **CLI:** `openclaw status --usage` y `openclaw channels list` muestran
  ventanas de cuota del proveedor (no costos por respuesta).

<div id="cost-estimation-when-shown">
  ## Estimación de costes (cuando se muestren)
</div>

Los costes se estiman a partir de la configuración de precios de tu modelo:

```
models.providers.<provider>.models[].cost
```

Estos son **USD por cada 1M de tokens** para `input`, `output`, `cacheRead` y
`cacheWrite`. Si no hay información de precios, OpenClaw solo muestra los tokens. Los tokens de OAuth
nunca muestran el costo en dólares.

<div id="cache-ttl-and-pruning-impact">
  ## Impacto del TTL de caché y la poda
</div>

El almacenamiento en caché de prompts del proveedor solo se aplica dentro de la
ventana de TTL de la caché. OpenClaw puede ejecutar opcionalmente **poda por TTL de caché**:
poda la sesión una vez que el TTL de la caché ha expirado y luego restablece la
ventana de caché para que las solicitudes posteriores puedan reutilizar el
contexto recién almacenado en caché en lugar de volver a almacenar en caché todo
el historial. Esto mantiene más bajos los costos de escritura en caché cuando
una sesión queda inactiva más allá del TTL.

Configúralo en la [configuración del Gateway](/es/gateway/configuration) y consulta
los detalles del comportamiento en [Poda de sesiones](/es/concepts/session-pruning).

El latido puede mantener la caché **caliente** durante los periodos de inactividad.
Si el TTL de caché de tu modelo es `1h`, definir el intervalo de latido justo por
debajo de ese valor (por ejemplo, `55m`) puede evitar volver a almacenar en caché
el prompt completo, reduciendo los costos de escritura en caché.

En cuanto a los precios de la API de Anthropic, las lecturas de caché son
significativamente más baratas que los tokens de entrada, mientras que las
escrituras de caché se facturan con un multiplicador más alto. Consulta los
precios de almacenamiento en caché de prompts de Anthropic para conocer las
tarifas más recientes y los multiplicadores de TTL:
https://docs.anthropic.com/docs/build-with-claude/prompt-caching

<div id="example-keep-1h-cache-warm-with-heartbeat">
  ### Ejemplo: mantener en caliente la caché de 1 h con latido
</div>

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-5"
    models:
      "anthropic/claude-opus-4-5":
        params:
          cacheControlTtl: "1h"
    heartbeat:
      every: "55m"
```

<div id="tips-for-reducing-token-pressure">
  ## Consejos para reducir la presión de tokens
</div>

* Usa `/compact` para resumir sesiones largas.
* Recorta las salidas de herramientas muy grandes en tus flujos de trabajo.
* Mantén breves las descripciones de habilidades (la lista de habilidades se inyecta en el prompt).
* Da preferencia a modelos más pequeños para trabajo exploratorio y detallado.

Consulta [Habilidades](/es/tools/skills) para ver la fórmula exacta de la sobrecarga asociada a la lista de habilidades.