---
title: Memoria
summary: "Notas de investigación: sistema de memoria sin conexión para espacios de trabajo de Clawd (fuente de referencia en Markdown + índice derivado)"
read_when:
  - Al diseñar la memoria del espacio de trabajo (~/.openclaw/workspace) más allá de los registros diarios en Markdown
  - Al decidir entre una CLI independiente y una integración profunda con OpenClaw
  - Al añadir recuperación y reflexión sin conexión (retain/recall/reflect)
---

<div id="workspace-memory-v2-offline-research-notes">
  # Memoria del espacio de trabajo v2 (sin conexión): notas de investigación
</div>

Destino: espacio de trabajo al estilo Clawd (`agents.defaults.workspace`, valor predeterminado `~/.openclaw/workspace`) donde la “memoria” se almacena como un archivo Markdown por día (`memory/YYYY-MM-DD.md`), más un pequeño conjunto de archivos estables (p. ej., `memory.md`, `SOUL.md`).

Este documento propone una arquitectura de memoria **offline-first** que mantiene Markdown como la fuente de verdad canónica y revisable, pero añade **recuperación estructurada** (búsqueda, resúmenes de entidades, actualizaciones de confianza) mediante un índice derivado.

<div id="why-change">
  ## ¿Por qué cambiar?
</div>

La configuración actual (un archivo por día) es excelente para:

- llevar un registro tipo *append-only*
- edición manual
- durabilidad y auditabilidad con respaldo en git
- captura con poca fricción (“solo escríbelo”)

Es deficiente para:

- recuperación de alta cobertura (“¿qué decidimos sobre X?”, “¿qué pasó la última vez que probamos Y?”)
- respuestas centradas en entidades (“háblame de Alice / The Castle / warelay”) sin tener que releer muchos archivos
- estabilidad de opiniones/preferencias (y evidencia cuando cambia)
- restricciones temporales (“¿qué era cierto durante noviembre de 2025?”) y resolución de conflictos

<div id="design-goals">
  ## Objetivos de diseño
</div>

- **Offline**: funciona sin conexión; puede ejecutarse en un portátil/Castle; sin dependencia de la nube.
- **Explicable**: los elementos recuperados deben poder atribuirse (archivo + ubicación) y separarse de la inferencia.
- **Con poca ceremonia**: el registro diario sigue siendo Markdown, sin tener que diseñar esquemas complejos.
- **Incremental**: la v1 ya es útil solo con FTS; las capacidades semánticas/vectoriales y los grafos son mejoras opcionales.
- **Amigable para agentes**: facilita el “recuerdo dentro de los presupuestos de tokens” (devolver pequeños paquetes de hechos).

<div id="north-star-model-hindsight-letta">
  ## Modelo North Star (Hindsight × Letta)
</div>

Dos piezas a combinar:

1) **Bucle de control al estilo Letta/MemGPT**

- mantener un pequeño “núcleo” siempre en contexto (persona + datos clave del usuario)
- todo lo demás está fuera de contexto y se recupera mediante herramientas
- las escrituras de memoria son llamadas explícitas a herramientas (append/replace/insert), que se persisten y luego se reinyectan en el siguiente turno

2) **Sustrato de memoria al estilo Hindsight**

- separar lo observado de lo creído y de lo resumido
- admitir retener/recordar/reflexionar
- opiniones con un nivel de confianza que pueda evolucionar con la evidencia
- recuperación con reconocimiento de entidades + consultas temporales (incluso sin grafos de conocimiento completos)

<div id="proposed-architecture-markdown-source-of-truth-derived-index">
  ## Arquitectura propuesta (Markdown como fuente de la verdad + índice generado)
</div>

<div id="canonical-store-git-friendly">
  ### Almacén canónico (compatible con Git)
</div>

Mantén `~/.openclaw/workspace` como memoria canónica legible para humanos.

Estructura sugerida del espacio de trabajo:

```
~/.openclaw/workspace/
  memory.md                    # pequeño: hechos duraderos + preferencias (esencial)
  memory/
    YYYY-MM-DD.md              # registro diario (agregar; narrativa)
  bank/                        # páginas de memoria "tipadas" (estables, revisables)
    world.md                   # hechos objetivos sobre el mundo
    experience.md              # lo que hizo el agente (primera persona)
    opinions.md                # preferencias/juicios subjetivos + confianza + punteros a evidencia
    entities/
      Peter.md
      The-Castle.md
      warelay.md
      ...
```

Notas:

* **El registro diario sigue siendo el registro diario**. No hace falta convertirlo a JSON.
* Los archivos de `bank/` están **curados**, producidos por tareas de reflexión y aún se pueden editar a mano.
* `memory.md` sigue siendo “pequeño + bastante esencial”: las cosas que quieres que Clawd vea en cada sesión.


<div id="derived-store-machine-recall">
  ### Almacén derivado (recuperación automática)
</div>

Añade un índice derivado dentro del espacio de trabajo (no necesariamente con seguimiento en Git):

```
~/.openclaw/workspace/.memory/index.sqlite
```

Apóyalo con:

* Esquema SQLite para hechos + enlaces entre entidades + metadatos de opinión
* SQLite **FTS5** para recuperación léxica (rápida, ligera y sin conexión)
* tabla opcional de embeddings para recuperación semántica (también sin conexión)

El índice siempre es **reconstruible desde Markdown**.


<div id="retain-recall-reflect-operational-loop">
  ## Retener / Recordar / Reflexionar (ciclo operativo)
</div>

<div id="retain-normalize-daily-logs-into-facts">
  ### Retain: normalizar los registros diarios en “hechos”
</div>

La idea clave de Hindsight que importa aquí: almacena **hechos narrativos y autocontenidos**, no fragmentos diminutos.

Regla práctica para `memory/YYYY-MM-DD.md`:

* al final del día (o durante este), agrega una sección `## Retain` con 2–5 puntos que sean:
  * narrativos (se preserva el contexto entre turnos)
  * autocontenidos (tienen sentido por sí solos más adelante)
  * etiquetados con tipo + menciones de entidades

Ejemplo:

```
## Retener
- W @Peter: Actualmente en Marrakech (27 nov–1 dic, 2025) para el cumpleaños de Andy.
- B @warelay: Arreglé el fallo de WS de Baileys envolviendo los manejadores de connection.update en try/catch (ver memory/2025-11-27.md).
- O(c=0.95) @Peter: Prefiere respuestas concisas (&lt;1500 caracteres) en WhatsApp; el contenido largo va en archivos.
```

Análisis mínimo:

* Prefijo de tipo: `W` (mundo), `B` (experiencial/biográfica), `O` (opinión), `S` (observación/resumen; generalmente generado)
* Entidades: `@Peter`, `@warelay`, etc. (los slugs se corresponden con `bank/entities/*.md`)
* Confianza de la opinión: `O(c=0.0..1.0)` opcional

Si no quieres que los autores tengan que pensar en esto, el job de reflexión puede inferir estos puntos a partir del resto del log, pero tener una sección explícita `## Retain` es la “palanca de calidad” más sencilla.


<div id="recall-queries-over-the-derived-index">
  ### Recall: consultas sobre el índice derivado
</div>

Recall debería admitir:

- **léxica**: “encontrar términos / nombres / comandos exactos” (FTS5)
- **de entidad**: “háblame sobre X” (páginas de entidad + hechos vinculados a entidades)
- **temporal**: “qué pasó alrededor del 27 de nov” / “desde la semana pasada”
- **de opinión**: “qué prefiere Peter?” (con nivel de confianza + evidencia)

El formato de respuesta debería ser amigable para el agente y citar las fuentes:

- `kind` (`world|experience|opinion|observation`)
- `timestamp` (día de la fuente o intervalo de tiempo extraído, si está presente)
- `entities` (`["Peter","warelay"]`)
- `content` (el hecho en forma narrativa)
- `source` (`memory/2025-11-27.md#L12` etc)

<div id="reflect-produce-stable-pages-update-beliefs">
  ### Reflexión: producir páginas estables + actualizar creencias
</div>

La reflexión es una tarea programada (diaria o latido `ultrathink`) que:

- actualiza `bank/entities/*.md` a partir de hechos recientes (resúmenes de entidades)
- actualiza la confianza de `bank/opinions.md` en función de refuerzo/contradicción
- opcionalmente propone ediciones a `memory.md` (hechos duraderos tipo “núcleo”)

Evolución de opiniones (simple, explicable):

- cada opinión tiene:
  - enunciado
  - confianza `c ∈ [0,1]`
  - last_updated
  - enlaces de evidencia (ID de hechos que apoyan + contradicen)
- cuando llegan nuevos hechos:
  - se buscan opiniones candidatas por superposición de entidades + similitud (primero FTS, luego embeddings)
  - se actualiza la confianza con pequeños incrementos; los saltos grandes requieren contradicción fuerte + evidencia repetida

<div id="cli-integration-standalone-vs-deep-integration">
  ## Integración con la CLI: independiente vs integración profunda
</div>

Recomendación: **integración profunda en OpenClaw**, pero mantén una biblioteca principal separable.

<div id="why-integrate-into-openclaw">
  ### ¿Por qué integrar en OpenClaw?
</div>

- OpenClaw ya conoce:
  - la ruta del espacio de trabajo (`agents.defaults.workspace`)
  - el modelo de la sesión + latidos
  - los patrones de registro (`logging`) y diagnóstico de problemas
- Quieres que el propio agente invoque las herramientas:
  - `openclaw memory recall "…" --k 25 --since 30d`
  - `openclaw memory reflect --since 7d`

<div id="why-still-split-a-library">
  ### ¿Por qué seguir separando una biblioteca?
</div>

- mantener la lógica de memoria fácilmente testeable sin el Gateway ni el runtime
- reutilizarla en otros contextos (scripts locales, futura aplicación de escritorio, etc.)

Diseño:
Las herramientas de memoria están pensadas como una pequeña capa de CLI + biblioteca, pero por ahora esto es solo exploratorio.

<div id="s-collide-suco-when-to-use-it-research">
  ## “S-Collide” / SuCo: cuándo usarlo (investigación)
</div>

Si “S-Collide” se refiere a **SuCo (Subspace Collision)**: es un enfoque de recuperación ANN que busca buenos compromisos entre recall y latencia usando colisiones aprendidas/estructuradas en subespacios (paper: arXiv 2411.14754, 2024).

Enfoque pragmático para `~/.openclaw/workspace`:

- **no empieces** con SuCo.
- empieza con SQLite FTS + (opcional) embeddings simples; obtendrás la mayoría de las mejoras de UX de inmediato.
- considera soluciones tipo SuCo/HNSW/ScaNN solo cuando:
  - el corpus es grande (decenas/centenas de miles de fragmentos)
  - la búsqueda por embeddings por fuerza bruta se vuelve demasiado lenta
  - la calidad de recall queda significativamente limitada por la búsqueda léxica

Alternativas aptas para uso sin conexión (en orden creciente de complejidad):

- SQLite FTS5 + filtros de metadatos (cero ML)
- Embeddings + fuerza bruta (funciona sorprendentemente bien si el número de fragmentos es bajo)
- Índice HNSW (común, robusto; necesita un binding de biblioteca)
- SuCo (a nivel de investigación; atractivo si hay una implementación sólida que puedas integrar)

Pregunta abierta:

- ¿cuál es el **mejor** modelo de embeddings para uso sin conexión para la “memoria de asistente personal” en tus máquinas (portátil + escritorio)?
  - si ya tienes Ollama: genera embeddings con un modelo local; de lo contrario, incluye un modelo de embeddings pequeño en la cadena de herramientas.

<div id="smallest-useful-pilot">
  ## Piloto mínimo útil
</div>

Si quieres una versión mínima pero funcional:

- Añade páginas de entidad `bank/` y una sección `## Retain` en los registros diarios.
- Usa SQLite FTS para recuperación con citas (path + números de línea).
- Añade embeddings solo si la calidad o la escala de la recuperación lo requieren.

<div id="references">
  ## Referencias
</div>

- Conceptos de Letta / MemGPT: “core memory blocks” + “archival memory” + memoria autoeditada mediante herramientas.
- Hindsight Technical Report: “retain / recall / reflect”, memoria de cuatro redes, extracción narrativa de hechos, evolución de la confianza en las opiniones.
- SuCo: arXiv 2411.14754 (2024): recuperación aproximada de los vecinos más cercanos mediante “Subspace Collision”.