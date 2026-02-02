---
title: Perplexity
summary: "Configuración de Perplexity Sonar para web_search"
read_when:
  - Quieres usar Perplexity Sonar para búsqueda en la web
  - Necesitas PERPLEXITY_API_KEY o la configuración de OpenRouter
---

<div id="perplexity-sonar">
  # Perplexity Sonar
</div>

OpenClaw puede usar Perplexity Sonar para la herramienta `web_search`. Puedes conectarte
a través de la API directa de Perplexity o mediante OpenRouter.

<div id="api-options">
  ## Opciones de la API
</div>

<div id="perplexity-direct">
  ### Perplexity (directo)
</div>

* URL base: https://api.perplexity.ai
* Variable de entorno: `PERPLEXITY_API_KEY`

<div id="openrouter-alternative">
  ### OpenRouter (alternativa)
</div>

* URL base: https://openrouter.ai/api/v1
* Variable de entorno: `OPENROUTER_API_KEY`
* Admite créditos de prepago/en criptomonedas.

<div id="config-example">
  ## Ejemplo de configuración
</div>

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

<div id="switching-from-brave">
  ## Cambiar desde Brave
</div>

```json5
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai"
        }
      }
    }
  }
}
```

Si tanto `PERPLEXITY_API_KEY` como `OPENROUTER_API_KEY` están definidas, establece
`tools.web.search.perplexity.baseUrl` (o `tools.web.search.perplexity.apiKey`)
para desambiguar.

Si no se establece ninguna URL base, OpenClaw elige un valor predeterminado en función del origen de la clave de API:

* `PERPLEXITY_API_KEY` o `pplx-...` → Perplexity directa (`https://api.perplexity.ai`)
* `OPENROUTER_API_KEY` o `sk-or-...` → OpenRouter (`https://openrouter.ai/api/v1`)
* Formatos de clave desconocidos → OpenRouter (opción de reserva segura)

<div id="models">
  ## Modelos
</div>

* `perplexity/sonar` — preguntas y respuestas rápidas con búsqueda web
* `perplexity/sonar-pro` (por defecto) — razonamiento paso a paso + búsqueda web
* `perplexity/sonar-reasoning-pro` — investigación en profundidad

Consulta [Herramientas web](/es/tools/web) para ver la configuración completa de web&#95;search.