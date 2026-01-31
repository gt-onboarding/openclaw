---
title: Web
summary: "Herramientas de búsqueda y recuperación web (Brave Search API, Perplexity direct/OpenRouter)"
read_when:
  - Deseas habilitar web_search o web_fetch
  - Necesitas configurar la clave de Brave Search API
  - Deseas usar Perplexity Sonar para búsquedas web
---

<div id="web-tools">
  # Herramientas web
</div>

OpenClaw incluye dos herramientas web ligeras:

* `web_search` — Busca en la web mediante la Brave Search API (por defecto) o Perplexity Sonar (directamente o a través de OpenRouter).
* `web_fetch` — Realiza una petición HTTP y extrae contenido legible (HTML → markdown/texto).

**No** son automatización del navegador. Para sitios muy cargados de JS o que requieren inicio de sesión, usa la
[Herramienta de navegador](/es/tools/browser).

<div id="how-it-works">
  ## Cómo funciona
</div>

* `web_search` llama a tu proveedor configurado y devuelve resultados.
  * **Brave** (predeterminado): devuelve resultados estructurados (título, URL, fragmento).
  * **Perplexity**: devuelve respuestas generadas por IA con citas de búsquedas web en tiempo real.
* Los resultados se almacenan en caché por consulta durante 15 minutos (configurable).
* `web_fetch` hace una petición HTTP GET simple y extrae contenido legible
  (HTML → markdown/texto). **No** ejecuta JavaScript.
* `web_fetch` está habilitado de forma predeterminada (a menos que se deshabilite explícitamente).

<div id="choosing-a-search-provider">
  ## Elegir un proveedor de búsqueda
</div>

| Proveedor                  | Ventajas                                                          | Desventajas                               | Clave de API                                |
| -------------------------- | ----------------------------------------------------------------- | ----------------------------------------- | ------------------------------------------- |
| **Brave** (predeterminado) | Resultados rápidos y estructurados, plan gratuito                 | Resultados de búsqueda tradicionales      | `BRAVE_API_KEY`                             |
| **Perplexity**             | Respuestas sintetizadas por IA, citas, información en tiempo real | Requiere acceso a Perplexity o OpenRouter | `OPENROUTER_API_KEY` o `PERPLEXITY_API_KEY` |

Consulta [configuración de Brave Search](/es/brave-search) y [Perplexity Sonar](/es/perplexity) para obtener detalles específicos de cada proveedor.

Configura el proveedor en la configuración:

```json5
{
  tools: {
    web: {
      search: {
        provider: "brave"  // o "perplexity"
      }
    }
  }
}
```

Ejemplo: cambiar a Perplexity Sonar (API directa):

```json5
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai",
          model: "perplexity/sonar-pro"
        }
      }
    }
  }
}
```

<div id="getting-a-brave-api-key">
  ## Obtener una clave de API de Brave
</div>

1. Crea una cuenta de Brave Search API en https://brave.com/search/api/
2. En el panel de control, elige el plan **Data for Search** (no “Data for AI”) y genera una clave de API.
3. Ejecuta `openclaw configure --section web` para guardar la clave en la configuración (recomendado), o define `BRAVE_API_KEY` en tu entorno.

Brave ofrece un plan gratuito además de planes de pago; consulta el portal de la API de Brave para ver los límites y precios actuales.

<div id="where-to-set-the-key-recommended">
  ### Dónde configurar la clave (recomendado)
</div>

**Recomendado:** ejecuta `openclaw configure --section web`. Esto almacena la clave en
`~/.openclaw/openclaw.json` bajo `tools.web.search.apiKey`.

**Alternativa mediante variables de entorno:** establece `BRAVE_API_KEY` en el entorno
del proceso de Gateway. Para una instalación de Gateway, colócalo en `~/.openclaw/.env`
(o en el entorno de tu servicio). Consulta [Env vars](/es/help/faq#how-does-openclaw-load-environment-variables).

<div id="using-perplexity-direct-or-via-openrouter">
  ## Uso de Perplexity (directamente o mediante OpenRouter)
</div>

Los modelos Perplexity Sonar tienen capacidades de búsqueda en la web integradas y devuelven respuestas sintetizadas por IA con citas. Puedes usarlos a través de OpenRouter (no se requiere tarjeta de crédito; admite cripto/prepago).

<div id="getting-an-openrouter-api-key">
  ### Obtener una clave de API de OpenRouter
</div>

1. Crea una cuenta en https://openrouter.ai/
2. Añade saldo (se admiten criptomonedas, prepago o tarjeta de crédito)
3. Genera una clave de API desde la configuración de tu cuenta

<div id="setting-up-perplexity-search">
  ### Configuración de la búsqueda con Perplexity
</div>

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "perplexity",
        perplexity: {
          // Clave API (opcional si OPENROUTER_API_KEY o PERPLEXITY_API_KEY está configurada)
          apiKey: "sk-or-v1-...",
          // Base URL (key-aware default if omitted)
          baseUrl: "https://openrouter.ai/api/v1",
          // Model (defaults to perplexity/sonar-pro)
          model: "perplexity/sonar-pro"
        }
      }
    }
  }
}
```

**Alternativa de entorno:** configura `OPENROUTER_API_KEY` o `PERPLEXITY_API_KEY` en el entorno del Gateway.
Para una instalación del Gateway, colócala en `~/.openclaw/.env`.

Si no se configura una URL base, OpenClaw elige un valor predeterminado según el origen de la clave de API:

* `PERPLEXITY_API_KEY` o `pplx-...` → `https://api.perplexity.ai`
* `OPENROUTER_API_KEY` o `sk-or-...` → `https://openrouter.ai/api/v1`
* Formatos de clave desconocidos → OpenRouter (opción de reserva segura)

<div id="available-perplexity-models">
  ### Modelos Perplexity disponibles
</div>

| Modelo | Descripción | Mejor para |
|-------|-------------|----------|
| `perplexity/sonar` | Preguntas y respuestas rápidas con búsqueda web | Consultas rápidas |
| `perplexity/sonar-pro` (predeterminado) | Razonamiento de varios pasos con búsqueda web | Preguntas complejas |
| `perplexity/sonar-reasoning-pro` | Análisis de cadena de razonamiento | Investigación profunda |

<div id="web_search">
  ## web_search
</div>

Busca en la web usando el proveedor que tengas configurado.

<div id="requirements">
  ### Requisitos
</div>

* `tools.web.search.enabled` no debe ser `false` (valor predeterminado: habilitado)
* Clave de API para el proveedor que elijas:
  * **Brave**: `BRAVE_API_KEY` o `tools.web.search.apiKey`
  * **Perplexity**: `OPENROUTER_API_KEY`, `PERPLEXITY_API_KEY` o `tools.web.search.perplexity.apiKey`

<div id="config">
  ### Configuración
</div>

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE", // opcional si BRAVE_API_KEY está definida
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15
      }
    }
  }
}
```

<div id="tool-parameters">
  ### Parámetros de la herramienta
</div>

* `query` (obligatorio)
* `count` (1–10; valor predeterminado de la configuración)
* `country` (opcional): código de país de 2 letras para resultados específicos por región (por ejemplo, &quot;DE&quot;, &quot;US&quot;, &quot;ALL&quot;). Si se omite, Brave elige su región predeterminada.
* `search_lang` (opcional): código de idioma ISO para los resultados de búsqueda (por ejemplo, &quot;de&quot;, &quot;en&quot;, &quot;fr&quot;)
* `ui_lang` (opcional): código de idioma ISO para los elementos de la UI
* `freshness` (opcional, solo Brave): filtrar por momento de descubrimiento (`pd`, `pw`, `pm`, `py`, o `YYYY-MM-DD a YYYY-MM-DD`)

**Ejemplos:**

```javascript
// German-specific search
await web_search({
  query: "TV online schauen",
  count: 10,
  country: "DE",
  search_lang: "de"
});

// Búsqueda en francés con UI en francés
await web_search({
  query: "actualités",
  country: "FR",
  search_lang: "fr",
  ui_lang: "fr"
});

// Recent results (past week)
await web_search({
  query: "TMBG interview",
  freshness: "pw"
});
```

<div id="web_fetch">
  ## web_fetch
</div>

Recupera una URL y extrae el contenido legible.

<div id="requirements">
  ### Requisitos
</div>

* `tools.web.fetch.enabled` no debe ser `false` (predeterminado: habilitado)
* Fallback opcional con Firecrawl: configura `tools.web.fetch.firecrawl.apiKey` o `FIRECRAWL_API_KEY`.

<div id="config">
  ### Configuración
</div>

```json5
{
  tools: {
    web: {
      fetch: {
        enabled: true,
        maxChars: 50000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        maxRedirects: 3,
        userAgent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 14_7_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36",
        readability: true,
        firecrawl: {
          enabled: true,
          apiKey: "FIRECRAWL_API_KEY_HERE", // opcional si FIRECRAWL_API_KEY está configurado
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 86400000, // ms (1 day)
          timeoutSeconds: 60
        }
      }
    }
  }
}
```

<div id="tool-parameters">
  ### Parámetros de la herramienta
</div>

* `url` (obligatorio, solo http/https)
* `extractMode` (`markdown` | `text`)
* `maxChars` (trunca páginas largas)

Notas:

* `web_fetch` usa primero Readability (extracción del contenido principal) y luego Firecrawl (si está configurado). Si ambos fallan, la herramienta devuelve un error.
* Las solicitudes a Firecrawl usan un modo de evasión de bots y almacenan en caché los resultados de forma predeterminada.
* `web_fetch` envía por defecto un User-Agent similar a Chrome y `Accept-Language`; puedes sobrescribir `userAgent` si es necesario.
* `web_fetch` bloquea nombres de host privados/internos y vuelve a comprobar las redirecciones (limita con `maxRedirects`).
* `web_fetch` realiza una extracción de mejor esfuerzo; algunos sitios necesitarán la herramienta de navegador.
* Consulta [Firecrawl](/es/tools/firecrawl) para la configuración de claves y detalles del servicio.
* Las respuestas se almacenan en caché (15 minutos por defecto) para reducir las solicitudes repetidas.
* Si usas perfiles/listas de permitidos de herramientas, añade `web_search`/`web_fetch` o `group:web`.
* Si falta la clave de Brave, `web_search` devuelve una breve sugerencia de configuración con un enlace a la documentación.