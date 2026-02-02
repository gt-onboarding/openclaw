---
title: Poda de sesiones
summary: "Poda de sesiones: recorte de resultados de herramientas para reducir el crecimiento excesivo del contexto"
read_when:
  - Quieres limitar el crecimiento del contexto del LLM causado por los resultados de las herramientas
  - Estás ajustando agents.defaults.contextPruning
---

<div id="session-pruning">
  # Poda de sesiones
</div>

La poda de sesiones elimina **resultados antiguos de herramientas** del contexto en memoria justo antes de cada llamada al LLM. **No** modifica el historial de la sesión almacenado en disco (`*.jsonl`).

<div id="when-it-runs">
  ## Cuándo se ejecuta
</div>

* Cuando `mode: "cache-ttl"` está habilitado y la última llamada de Anthropic para la sesión es anterior a `ttl`.
* Solo afecta a los mensajes enviados al modelo para esa solicitud.
* Solo está activo para llamadas a la API de Anthropic (y modelos de Anthropic en OpenRouter).
* Para obtener mejores resultados, ajusta `ttl` para que coincida con el `cacheControlTtl` de tu modelo.
* Después de una poda, la ventana de TTL se reinicia, por lo que las solicitudes posteriores seguirán usando la caché hasta que `ttl` vuelva a expirar.

<div id="smart-defaults-anthropic">
  ## Valores predeterminados inteligentes (Anthropic)
</div>

* Perfiles de **OAuth o setup-token**: habilitan la poda de `cache-ttl` y configuran el heartbeat en `1h`.
* Perfiles de **clave de API**: habilitan la poda de `cache-ttl`, configuran el heartbeat en `30m` y establecen `cacheControlTtl` por defecto en `1h` en los modelos de Anthropic.
* Si estableces explícitamente cualquiera de estos valores, OpenClaw **no** los sobrescribe.

<div id="what-this-improves-cost-cache-behavior">
  ## Qué mejora esto (coste + comportamiento de caché)
</div>

* **Por qué podar:** el caché de prompts de Anthropic solo se aplica dentro del TTL. Si una sesión permanece inactiva más allá del TTL, la siguiente solicitud vuelve a almacenar en caché el prompt completo, a menos que lo recortes primero.
* **Qué resulta más barato:** la poda reduce el tamaño de **cacheWrite** para esa primera solicitud después de que expire el TTL.
* **Por qué importa el reinicio del TTL:** una vez que se ejecuta la poda, se reinicia la ventana de caché, por lo que las solicitudes posteriores pueden reutilizar el prompt recién almacenado en caché en lugar de volver a almacenar en caché todo el historial.
* **Lo que no hace:** la poda no añade tokens ni “duplica” costes; solo cambia qué se almacena en caché en esa primera solicitud posterior al TTL.

<div id="what-can-be-pruned">
  ## Qué se puede podar
</div>

* Solo mensajes `toolResult`.
* Los mensajes de usuario y de asistente **nunca** se modifican.
* Los últimos mensajes de asistente indicados por `keepLastAssistants` están protegidos; los resultados de herramientas posteriores a ese umbral no se podan.
* Si no hay suficientes mensajes de asistente para establecer el umbral, se omite la poda.
* Los resultados de herramientas que contienen **bloques de imagen** se omiten (nunca se recortan/borran).

<div id="context-window-estimation">
  ## Estimación de la ventana de contexto
</div>

La poda utiliza una ventana de contexto estimada (caracteres ≈ tokens × 4). El tamaño de la ventana se determina en este orden:

1. Definición de modelo `contextWindow` (del registro de modelos).
2. Override `models.providers.*.models[].contextWindow`.
3. `agents.defaults.contextTokens`.
4. Valor por defecto de `200000` tokens.

<div id="mode">
  ## Modo
</div>

<div id="cache-ttl">
  ### cache-ttl
</div>

* La poda solo se ejecuta si la última llamada a Anthropic tiene más de `ttl` de antigüedad (valor predeterminado: `5m`).
* Cuando se ejecuta: mismo comportamiento de recorte suave + limpieza total que antes.

<div id="soft-vs-hard-pruning">
  ## Poda suave vs rígida
</div>

* **Soft-trim**: solo para resultados de herramientas demasiado grandes.
  * Conserva el principio y el final, inserta `...` y añade una nota con el tamaño original.
  * Omite resultados con bloques de imagen.
* **Hard-clear**: sustituye todo el resultado de la herramienta por `hardClear.placeholder`.

<div id="tool-selection">
  ## Selección de herramientas
</div>

* `tools.allow` / `tools.deny` admiten comodines `*`.
* La denegación prevalece.
* La coincidencia no distingue entre mayúsculas y minúsculas.
* Lista `tools.allow` vacía =&gt; todas las herramientas permitidas.

<div id="interaction-with-other-limits">
  ## Interacción con otros límites
</div>

* Las herramientas integradas ya truncan su propia salida; la poda de sesiones es una capa adicional que evita que los chats de larga duración acumulen demasiada salida de herramientas en el contexto del modelo.
* La compactación es independiente: la compactación resume y persiste, mientras que la poda es transitoria por cada solicitud. Consulta [/concepts/compaction](/es/concepts/compaction).

<div id="defaults-when-enabled">
  ## Valores predeterminados (cuando está habilitado)
</div>

* `ttl`: `"5m"`
* `keepLastAssistants`: `3`
* `softTrimRatio`: `0.3`
* `hardClearRatio`: `0.5`
* `minPrunableToolChars`: `50000`
* `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }`
* `hardClear`: `{ enabled: true, placeholder: "[Contenido anterior del resultado de la herramienta eliminado]" }`

<div id="examples">
  ## Ejemplos
</div>

Valor predeterminado (desactivado):

```json5
{
  agent: {
    contextPruning: { mode: "off" }
  }
}
```

Habilita la poda basada en TTL:

```json5
{
  agent: {
    contextPruning: { mode: "cache-ttl", ttl: "5m" }
  }
}
```

Restringir la limpieza a herramientas específicas:

```json5
{
  agent: {
    contextPruning: {
      modo: "cache-ttl",
      tools: { allow: ["exec", "read"], deny: ["*image*"] }
    }
  }
}
```

Consulta la referencia de configuración: [Configuración de Gateway](/es/gateway/configuration)
