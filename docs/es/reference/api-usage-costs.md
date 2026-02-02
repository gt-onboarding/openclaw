---
title: Costes de uso de la API
summary: "Audita qué puede generar gastos, qué claves se utilizan y cómo consultar el uso"
read_when:
  - Quieres entender qué funcionalidades pueden invocar APIs de pago
  - Necesitas auditar claves, costes y visibilidad del consumo
  - Estás explicando cómo funcionan los informes de costes de /status o /usage
---

<div id="api-usage-costs">
  # Uso de la API y costos
</div>

Este documento enumera **características que pueden usar claves de API** y dónde se reflejan sus costos. Se centra en
las características de OpenClaw que pueden generar uso de proveedores o llamadas de API de pago.

<div id="where-costs-show-up-chat-cli">
  ## Dónde aparecen los costos (chat + CLI)
</div>

**Instantánea de costo por sesión**

* `/status` muestra el modelo de la sesión actual, el uso de contexto y los últimos tokens de respuesta.
* Si el modelo usa **autenticación mediante clave de API**, `/status` también muestra el **costo estimado** de la última respuesta.

**Pie de costo por mensaje**

* `/usage full` agrega un pie de uso a cada respuesta, incluyendo el **costo estimado** (solo clave de API).
* `/usage tokens` muestra solo tokens; los flujos OAuth ocultan el costo en dólares.

**Ventanas de uso en la CLI (cuotas del proveedor)**

* `openclaw status --usage` y `openclaw channels list` muestran las **ventanas de uso** del proveedor
  (instantáneas de cuota, no costos por mensaje).

Consulta [Uso de tokens y costos](/es/token-use) para detalles y ejemplos.

<div id="how-keys-are-discovered">
  ## Cómo se detectan las claves
</div>

OpenClaw puede obtener credenciales de:

* **Perfiles de autenticación** (por agente, almacenados en `auth-profiles.json`).
* **Variables de entorno** (por ejemplo, `OPENAI_API_KEY`, `BRAVE_API_KEY`, `FIRECRAWL_API_KEY`).
* **Configuración** (`models.providers.*.apiKey`, `tools.web.search.*`, `tools.web.fetch.firecrawl.*`,
  `memorySearch.*`, `talk.apiKey`).
* **Habilidades** (`skills.entries.<name>.apiKey`) que pueden exportar claves al entorno del proceso de la habilidad.

<div id="features-that-can-spend-keys">
  ## Funcionalidades que pueden consumir claves
</div>

<div id="1-core-model-responses-chat-tools">
  ### 1) Respuestas del modelo central (chat + herramientas)
</div>

Cada respuesta o llamada de herramienta utiliza el **proveedor de modelo actual** (OpenAI, Anthropic, etc.). Esta es la
fuente principal de consumo y costes.

Consulta [Modelos](/es/providers/models) para la configuración de precios y [Uso de tokens y costes](/es/token-use) para verlos.

<div id="2-media-understanding-audioimagevideo">
  ### 2) Comprensión de medios (audio/imagen/video)
</div>

El contenido multimedia entrante puede resumirse/transcribirse antes de que se genere la respuesta. Esto utiliza APIs de modelos/proveedores.

* Audio: OpenAI / Groq / Deepgram (ahora **activado automáticamente** cuando hay claves configuradas).
* Imagen: OpenAI / Anthropic / Google.
* Video: Google.

Consulta [Comprensión de medios](/es/nodes/media-understanding).

<div id="3-memory-embeddings-semantic-search">
  ### 3) Embeddings de memoria + búsqueda semántica
</div>

La búsqueda semántica en memoria usa **APIs de embeddings** cuando se configura para proveedores remotos:

* `memorySearch.provider = "openai"` → embeddings de OpenAI
* `memorySearch.provider = "gemini"` → embeddings de Gemini
* Fallback opcional a OpenAI si fallan las embeddings locales

Puedes mantenerlo local con `memorySearch.provider = "local"` (sin uso de API).

Consulta [Memory](/es/concepts/memory).

<div id="4-web-search-tool-brave-perplexity-via-openrouter">
  ### 4) Herramienta de búsqueda web (Brave / Perplexity vía OpenRouter)
</div>

`web_search` usa claves de API y puede generar costos por uso:

* **Brave Search API**: `BRAVE_API_KEY` o `tools.web.search.apiKey`
* **Perplexity** (vía OpenRouter): `PERPLEXITY_API_KEY` o `OPENROUTER_API_KEY`

**Plan gratuito de Brave (bastante generoso):**

* **2.000 solicitudes/mes**
* **1 solicitud/segundo**
* **Tarjeta de crédito obligatoria** para la verificación (no se cobra nada a menos que actualices a un plan superior)

Consulta [Herramientas web](/es/tools/web).

<div id="5-web-fetch-tool-firecrawl">
  ### 5) Herramienta de obtención web (Firecrawl)
</div>

`web_fetch` puede llamar a **Firecrawl** cuando hay una clave de API configurada:

* `FIRECRAWL_API_KEY` o `tools.web.fetch.firecrawl.apiKey`

Si Firecrawl no está configurado, la herramienta recurre a `fetch` directo + legibilidad del contenido (sin API de pago).

Consulta [Herramientas web](/es/tools/web).

<div id="6-provider-usage-snapshots-statushealth">
  ### 6) Instantáneas de uso del proveedor (estado/salud)
</div>

Algunos comandos de estado llaman a **endpoints de uso del proveedor** para mostrar ventanas de cuota o el estado de la autenticación.
Suelen ser llamadas de bajo volumen, pero aun así realizan peticiones a las APIs del proveedor:

* `openclaw status --usage`
* `openclaw models status --json`

Consulta [Models CLI](/es/cli/models).

<div id="7-compaction-safeguard-summarization">
  ### 7) Resumen con protección de compactación
</div>

La protección de compactación puede resumir el historial de sesión usando el **modelo actual**, lo que
invoca las API del proveedor cuando se ejecuta.

Consulta [Gestión de sesiones + compactación](/es/reference/session-management-compaction).

<div id="8-model-scan-probe">
  ### 8) Escaneo / sondeo de modelos
</div>

`openclaw models scan` puede sondear modelos de OpenRouter y usa `OPENROUTER_API_KEY` cuando
el sondeo está habilitado.

Consulta [CLI de modelos](/es/cli/models).

<div id="9-talk-speech">
  ### 9) Talk (voz)
</div>

El modo Talk puede invocar **ElevenLabs** cuando está configurado:

* `ELEVENLABS_API_KEY` o `talk.apiKey`

Consulta el [modo Talk](/es/nodes/talk).

<div id="10-skills-third-party-apis">
  ### 10) Habilidades (APIs de terceros)
</div>

Las habilidades pueden almacenar `apiKey` en `skills.entries.<name>.apiKey`. Si una habilidad usa esa clave para APIs externas, puede generar costes según el proveedor de la habilidad.

Consulta [Habilidades](/es/tools/skills).