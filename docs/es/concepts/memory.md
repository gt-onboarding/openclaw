---
title: Memoria
summary: "Cómo funciona la memoria en OpenClaw (archivos del espacio de trabajo + vaciado automático de memoria)"
read_when:
  - Quieres conocer la estructura de los archivos de memoria y el flujo de trabajo
  - Quieres ajustar el vaciado automático de memoria previo a la compactación
---

<div id="memory">
  # Memoria
</div>

La memoria de OpenClaw es **Markdown simple en el espacio de trabajo del agente**. Los archivos son la fuente de la verdad; el modelo solo &quot;recuerda&quot; lo que se escribe en disco.

Las herramientas de búsqueda en la memoria las proporciona el complemento de memoria activo (predeterminado:
`memory-core`). Desactiva los complementos de memoria con `plugins.slots.memory = "none"`.

<div id="memory-files-markdown">
  ## Archivos de memoria (Markdown)
</div>

El diseño predeterminado del espacio de trabajo usa dos capas de memoria:

* `memory/YYYY-MM-DD.md`
  * Registro diario (solo para agregar entradas).
  * Leer el de hoy y el de ayer al inicio de la sesión.
* `MEMORY.md` (opcional)
  * Memoria a largo plazo seleccionada.
  * **Cargar solo en la sesión principal y privada** (nunca en contextos de grupo).

Estos archivos se encuentran en el espacio de trabajo (`agents.defaults.workspace`, valor predeterminado
`~/.openclaw/workspace`). Consulta [espacio de trabajo del agente](/es/concepts/agent-workspace) para ver la estructura completa.

<div id="when-to-write-memory">
  ## Cuándo escribir en la memoria
</div>

* Las decisiones, preferencias y hechos persistentes van a `MEMORY.md`.
* Las notas del día a día y el contexto activo van a `memory/YYYY-MM-DD.md`.
* Si alguien dice «acuérdate de esto», escríbelo (no lo mantengas solo en RAM).
* Esta área sigue evolucionando. Ayuda recordarle al modelo que guarde cosas en la memoria; sabrá qué hacer.
* Si quieres que algo se conserve, **pídele al bot que lo escriba** en la memoria.

<div id="automatic-memory-flush-pre-compaction-ping">
  ## Vaciado automático de memoria (ping previo a la compactación)
</div>

Cuando una sesión está **cerca de la compactación automática**, OpenClaw ejecuta un **turno silencioso del agente** que le recuerda al modelo que registre memoria persistente **antes** de que el contexto se compacte. Los prompts predeterminados dicen explícitamente que el modelo *puede responder*, pero normalmente `NO_REPLY` es la respuesta correcta, de forma que el usuario nunca vea este turno.

Esto se controla mediante `agents.defaults.compaction.memoryFlush`:

```json5
{
  agents: {
    defaults: {
      compaction: {
        reserveTokensFloor: 20000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 4000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Escribe cualquier nota duradera en memory/YYYY-MM-DD.md; responde con NO_REPLY si no hay nada que almacenar."
        }
      }
    }
  }
}
```

Detalles:

* **Umbral suave**: el vaciado se activa cuando la estimación de tokens de la sesión cruza
  `contextWindow - reserveTokensFloor - softThresholdTokens`.
* **Silencioso por defecto**: los prompts incluyen `NO_REPLY` para que no se entregue nada.
* **Dos prompts**: un prompt de usuario más un prompt de sistema añaden el recordatorio.
* **Un vaciado por ciclo de compactación** (registrado en `sessions.json`).
* **El espacio de trabajo debe permitir escritura**: si la sesión se ejecuta en sandbox con
  `workspaceAccess: "ro"` o `"none"`, se omite el vaciado.

Para conocer todo el ciclo de vida de la compactación, consulta
[Gestión de sesiones + compactación](/es/reference/session-management-compaction).

<div id="vector-memory-search">
  ## Búsqueda de memoria vectorial
</div>

OpenClaw puede construir un pequeño índice vectorial a partir de `MEMORY.md` y `memory/*.md` (más
cualquier directorio o archivo adicional que incluyas explícitamente), de modo que las consultas
semánticas puedan encontrar notas relacionadas incluso cuando la redacción difiera.

Valores predeterminados:

* Activada de forma predeterminada.
* Supervisa los archivos de memoria para detectar cambios (con *debounce*).
* Utiliza embeddings remotos de forma predeterminada. Si `memorySearch.provider` no está configurado, OpenClaw selecciona automáticamente:
  1. `local` si se ha configurado `memorySearch.local.modelPath` y el archivo existe.
  2. `openai` si se puede resolver una clave de OpenAI.
  3. `gemini` si se puede resolver una clave de Gemini.
  4. En caso contrario, la búsqueda de memoria permanece desactivada hasta que se configure.
* El modo local usa node-llama-cpp y puede requerir `pnpm approve-builds`.
* Usa sqlite-vec (cuando está disponible) para acelerar la búsqueda vectorial dentro de SQLite.

Los embeddings remotos **requieren** una clave de API para el proveedor de embeddings. OpenClaw
resuelve claves a partir de perfiles de autenticación, `models.providers.*.apiKey` o variables
de entorno. Codex OAuth solo cubre chat/completions y **no** sirve para
embeddings en la búsqueda de memoria. Para Gemini, usa `GEMINI_API_KEY` o
`models.providers.google.apiKey`. Cuando uses un endpoint personalizado compatible con OpenAI,
configura `memorySearch.remote.apiKey` (y opcionalmente `memorySearch.remote.headers`).

<div id="additional-memory-paths">
  ### Rutas de memoria adicionales
</div>

Si quieres indexar archivos Markdown fuera de la configuración predeterminada del espacio de trabajo, añade rutas explícitas:

```json5
agents: {
  defaults: {
    memorySearch: {
      extraPaths: ["../team-docs", "/srv/shared-notes/overview.md"]
    }
  }
}
```

Notas:

* Las rutas pueden ser absolutas o relativas al espacio de trabajo.
* Los directorios se recorren de forma recursiva en busca de archivos `.md`.
* Solo se indexan archivos Markdown.
* Los enlaces simbólicos (de archivos o directorios) se omiten.

<div id="gemini-embeddings-native">
  ### Embeddings nativos de Gemini
</div>

Configura el proveedor como `gemini` para usar directamente la API de embeddings de Gemini:

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "gemini",
      model: "gemini-embedding-001",
      remote: {
        apiKey: "YOUR_GEMINI_API_KEY"
      }
    }
  }
}
```

Notas:

* `remote.baseUrl` es opcional (de forma predeterminada usa la URL base de la API de Gemini).
* `remote.headers` te permite añadir cabeceras HTTP adicionales si es necesario.
* Modelo predeterminado: `gemini-embedding-001`.

Si deseas usar un **endpoint personalizado compatible con OpenAI** (OpenRouter, vLLM o un proxy),
puedes usar la configuración `remote` con el proveedor OpenAI:

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      remote: {
        baseUrl: "https://api.example.com/v1/",
        apiKey: "YOUR_OPENAI_COMPAT_API_KEY",
        headers: { "X-Custom-Header": "value" }
      }
    }
  }
}
```

Si no quieres configurar una clave de API, usa `memorySearch.provider = "local"` o configura
`memorySearch.fallback = "none"`.

Mecanismos de respaldo (fallbacks):

* `memorySearch.fallback` puede ser `openai`, `gemini`, `local` o `none`.
* El proveedor de respaldo solo se usa cuando falla el proveedor de embeddings principal.

Indexación por lotes (OpenAI + Gemini):

* Habilitada de forma predeterminada para embeddings de OpenAI y Gemini. Configura `agents.defaults.memorySearch.remote.batch.enabled = false` para deshabilitarla.
* El comportamiento predeterminado espera a que termine el lote; ajusta `remote.batch.wait`, `remote.batch.pollIntervalMs` y `remote.batch.timeoutMinutes` si es necesario.
* Configura `remote.batch.concurrency` para controlar cuántos trabajos por lotes enviamos en paralelo (valor predeterminado: 2).
* El modo por lotes se aplica cuando `memorySearch.provider = "openai"` o `"gemini"` y usa la clave de API correspondiente.
* Los trabajos por lotes de Gemini usan el endpoint asíncrono de embeddings batch y requieren que la Gemini Batch API esté disponible.

Por qué el procesamiento batch de OpenAI es rápido y barato:

* Para grandes backfills (reprocesos históricos de gran tamaño), OpenAI suele ser la opción más rápida que admitimos porque podemos enviar muchas solicitudes de embeddings en un único trabajo por lotes y dejar que OpenAI las procese de forma asíncrona.
* OpenAI ofrece precios con descuento para cargas de trabajo de Batch API, por lo que las ejecuciones de indexación grandes suelen ser más baratas que enviar las mismas solicitudes de forma síncrona.
* Consulta la documentación y los precios de OpenAI Batch API para más detalles:
  * https://platform.openai.com/docs/api-reference/batch
  * https://platform.openai.com/pricing

Ejemplo de configuración:

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      fallback: "openai",
      remote: {
        batch: { enabled: true, concurrency: 2 }
      },
      sync: { watch: true }
    }
  }
}
```

Herramientas:

* `memory_search` — devuelve fragmentos con archivo y rangos de líneas.
* `memory_get` — lee el contenido de un archivo de memoria por ruta.

Modo local:

* Establece `agents.defaults.memorySearch.provider = "local"`.
* Proporciona `agents.defaults.memorySearch.local.modelPath` (GGUF o URI `hf:`).
* Opcional: establece `agents.defaults.memorySearch.fallback = "none"` para evitar el fallback remoto.

<div id="how-the-memory-tools-work">
  ### Cómo funcionan las herramientas de memoria
</div>

* `memory_search` realiza una búsqueda semántica sobre fragmentos de Markdown (objetivo de ~400 tokens, solapamiento de 80 tokens) de `MEMORY.md` + `memory/**/*.md`. Devuelve el texto del fragmento (limitado a ~700 caracteres), la ruta del archivo, el rango de líneas, el score, el proveedor/modelo y si hubo una conmutación (fallback) de embeddings locales → remotos. No se devuelve el contenido completo del archivo.
* `memory_get` lee un archivo Markdown de memoria específico (relativo al espacio de trabajo), opcionalmente desde una línea inicial y durante N líneas. Las rutas fuera de `MEMORY.md` / `memory/` solo se permiten cuando están explícitamente listadas en `memorySearch.extraPaths`.
* Ambas herramientas solo se habilitan cuando `memorySearch.enabled` se evalúa como true para el agente.

<div id="what-gets-indexed-and-when">
  ### Qué se indexa (y cuándo)
</div>

* Tipo de archivo: solo Markdown (`MEMORY.md`, `memory/**/*.md`, más cualquier archivo `.md` bajo `memorySearch.extraPaths`).
* Almacenamiento del índice: SQLite por agente en `~/.openclaw/memory/<agentId>.sqlite` (configurable vía `agents.defaults.memorySearch.store.path`, admite el token `{agentId}`).
* Actualización: un watcher sobre `MEMORY.md`, `memory/` y `memorySearch.extraPaths` marca el índice como desactualizado (debounce de 1,5 s). La sincronización se programa al iniciar una sesión, al realizar una búsqueda o de forma periódica, y se ejecuta de manera asíncrona. Las transcripciones de sesión usan umbrales delta para activar la sincronización en segundo plano.
* Disparadores de reindexación: el índice almacena el **proveedor/modelo de embedding + huella del endpoint + parámetros de fragmentación (chunking)**. Si cualquiera de esos cambia, OpenClaw reinicia y reindexa automáticamente todo el almacén.

<div id="hybrid-search-bm25-vector">
  ### Búsqueda híbrida (BM25 + vector)
</div>

Cuando está activada, OpenClaw combina:

* **Similitud vectorial** (coincidencia semántica, la redacción puede diferir)
* **Relevancia de palabras clave con BM25** (tokens exactos como IDs, variables de entorno, símbolos de código)

Si la búsqueda de texto completo no está disponible en tu plataforma, OpenClaw recurre a la búsqueda basada solo en vectores.

<div id="why-hybrid">
  #### ¿Por qué búsqueda híbrida?
</div>

La búsqueda vectorial es excelente para “esto significa lo mismo”:

* “Mac Studio gateway host” frente a “la máquina que ejecuta el Gateway”
* “debounce file updates” frente a “evitar indexar en cada escritura”

Pero puede ser débil con tokens exactos y de alta relevancia:

* IDs (`a828e60`, `b3b9895a…`)
* símbolos de código (`memorySearch.query.hybrid`)
* cadenas de error (“sqlite-vec unavailable”)

BM25 (búsqueda de texto completo) es lo contrario: fuerte con tokens exactos, más débil con paráfrasis.
La búsqueda híbrida es el término medio pragmático: **usa ambas señales de recuperación** para obtener
buenos resultados tanto para consultas en “lenguaje natural” como para consultas de “aguja en un pajar”.

<div id="how-we-merge-results-the-current-design">
  #### Cómo combinamos resultados (el diseño actual)
</div>

Esquema de implementación:

1. Obtener un conjunto de candidatos de ambos lados:

* **Vector**: los primeros `maxResults * candidateMultiplier` por similitud de coseno.
* **BM25**: los primeros `maxResults * candidateMultiplier` por ranking BM25 de FTS5 (más bajo es mejor).

2. Convertir el ranking BM25 en una puntuación aproximadamente en el rango 0..1:

* `textScore = 1 / (1 + max(0, bm25Rank))`

3. Unir los candidatos por ID de fragmento (chunk) y calcular una puntuación ponderada:

* `finalScore = vectorWeight * vectorScore + textWeight * textScore`

Notas:

* `vectorWeight` + `textWeight` se normalizan a 1.0 en la resolución de configuración, así que los pesos se comportan como porcentajes.
* Si las embeddings no están disponibles (o el proveedor devuelve un vector nulo/zero-vector), igualmente ejecutamos BM25 y devolvemos coincidencias por palabra clave.
* Si no se puede crear FTS5, mantenemos la búsqueda solo vectorial (sin fallo crítico).

Esto no es “teoría de RI perfecta”, pero es simple, rápido y suele mejorar el recall/la precisión en notas reales.
Si más adelante queremos hacer algo más sofisticado, los siguientes pasos habituales son Reciprocal Rank Fusion (RRF) o normalización de puntuaciones
(min/max o z-score) antes de combinarlas.

Config:

```json5
agents: {
  defaults: {
    memorySearch: {
      query: {
        hybrid: {
          enabled: true,
          vectorWeight: 0.7,
          textWeight: 0.3,
          candidateMultiplier: 4
        }
      }
    }
  }
}
```

<div id="embedding-cache">
  ### Caché de embeddings
</div>

OpenClaw puede almacenar en caché **embeddings de fragmentos** en SQLite, de modo que la reindexación y las actualizaciones frecuentes (especialmente las transcripciones de sesiones) no vuelvan a generar embeddings para texto que no ha cambiado.

Configuración:

```json5
agents: {
  defaults: {
    memorySearch: {
      cache: {
        enabled: true,
        maxEntries: 50000
      }
    }
  }
}
```

<div id="session-memory-search-experimental">
  ### Búsqueda de memoria de sesión (experimental)
</div>

Opcionalmente puedes indexar **las transcripciones de la sesión** y consultarlas mediante `memory_search`.
Esta función está protegida por un flag experimental.

```json5
agents: {
  defaults: {
    memorySearch: {
      experimental: { sessionMemory: true },
      sources: ["memory", "sessions"]
    }
  }
}
```

Notas:

* La indexación de sesiones es **optativa** (desactivada por defecto).
* Las actualizaciones de la sesión se someten a *debounce* y se **indexan de forma asíncrona** cuando superan umbrales de delta (en régimen de mejor esfuerzo).
* `memory_search` nunca se bloquea esperando a la indexación; los resultados pueden estar ligeramente desactualizados hasta que termine la sincronización en segundo plano.
* Los resultados siguen incluyendo solo fragmentos; `memory_get` continúa limitado a archivos de memoria.
* La indexación de sesiones está aislada por agente (solo se indexan los registros de sesión de ese agente).
* Los registros de sesión residen en disco (`~/.openclaw/agents/<agentId>/sessions/*.jsonl`). Cualquier proceso/usuario con acceso al sistema de archivos puede leerlos, así que considera el acceso al disco como el límite de confianza. Para un aislamiento más estricto, ejecuta los agentes bajo distintos usuarios del sistema operativo o en hosts separados.

Umbrales de delta (valores predeterminados mostrados):

```json5
agents: {
  defaults: {
    memorySearch: {
      sync: {
        sessions: {
          deltaBytes: 100000,   // ~100 KB
          deltaMessages: 50     // líneas JSONL
        }
      }
    }
  }
}
```

<div id="sqlite-vector-acceleration-sqlite-vec">
  ### Aceleración vectorial con SQLite (sqlite-vec)
</div>

Cuando la extensión sqlite-vec está disponible, OpenClaw almacena las *embeddings* en una
tabla virtual de SQLite (`vec0`) y realiza consultas de distancia entre vectores en la
base de datos. Esto mantiene la búsqueda rápida sin cargar cada *embedding* en JS.

Configuración (opcional):

```json5
agents: {
  defaults: {
    memorySearch: {
      store: {
        vector: {
          enabled: true,
          extensionPath: "/path/to/sqlite-vec"
        }
      }
    }
  }
}
```

Notas:

* `enabled` es `true` de forma predeterminada; cuando está deshabilitado, la búsqueda recurre a similitud de coseno dentro del proceso sobre embeddings almacenados.
* Si la extensión sqlite-vec no está presente o falla al cargarse, OpenClaw registra el
  error y continúa con la alternativa en JS (sin tabla vectorial).
* `extensionPath` sobrescribe la ruta del sqlite-vec empaquetado (útil para compilaciones personalizadas
  o ubicaciones de instalación no estándar).

<div id="local-embedding-auto-download">
  ### Descarga automática de embeddings locales
</div>

* Modelo de embeddings local predeterminado: `hf:ggml-org/embeddinggemma-300M-GGUF/embeddinggemma-300M-Q8_0.gguf` (~0,6 GB).
* Cuando `memorySearch.provider = "local"`, `node-llama-cpp` resuelve `modelPath`; si falta el archivo GGUF, lo **descarga automáticamente** en la caché (o en `local.modelCacheDir` si está configurado) y luego lo carga. Las descargas se reanudan al reintentar.
* Requisito de compilación nativa: ejecuta `pnpm approve-builds`, selecciona `node-llama-cpp` y luego `pnpm rebuild node-llama-cpp`.
* Mecanismo de reserva: si la configuración local falla y `memorySearch.fallback = "openai"`, cambiamos automáticamente a embeddings remotos (`openai/text-embedding-3-small`, salvo que se haya sobrescrito) y registramos el motivo.

<div id="custom-openai-compatible-endpoint-example">
  ### Ejemplo de endpoint personalizado compatible con OpenAI
</div>

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      remote: {
        baseUrl: "https://api.example.com/v1/",
        apiKey: "YOUR_REMOTE_API_KEY",
        headers: {
          "X-Organization": "org-id",
          "X-Project": "project-id"
        }
      }
    }
  }
}
```

Notas:

* `remote.*` tiene prioridad sobre `models.providers.openai.*`.
* `remote.headers` se combina con los encabezados de OpenAI; `remote` tiene prioridad en caso de conflicto de claves. Omite `remote.headers` para usar los valores predeterminados de OpenAI.
