---
title: Perplexity
summary: "Configurazione di Perplexity Sonar per web_search"
read_when:
  - Vuoi utilizzare Perplexity Sonar per la ricerca sul web
  - Hai bisogno di PERPLEXITY_API_KEY o di una configurazione OpenRouter
---

<div id="perplexity-sonar">
  # Perplexity Sonar
</div>

OpenClaw può utilizzare Perplexity Sonar per lo strumento `web_search`. Puoi connetterti
all&#39;api diretta di Perplexity oppure tramite OpenRouter.

<div id="api-options">
  ## Opzioni API
</div>

<div id="perplexity-direct">
  ### Perplexity (diretto)
</div>

* URL base: https://api.perplexity.ai
* Variabile d&#39;ambiente: `PERPLEXITY_API_KEY`

<div id="openrouter-alternative">
  ### OpenRouter (alternativa)
</div>

* URL di base: https://openrouter.ai/api/v1
* Variabile d&#39;ambiente: `OPENROUTER_API_KEY`
* Supporta crediti prepagati e crypto.

<div id="config-example">
  ## Esempio di configurazione
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
  ## Migrare da Brave
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

Se sia `PERPLEXITY_API_KEY` sia `OPENROUTER_API_KEY` sono impostate, imposta
`tools.web.search.perplexity.baseUrl` (oppure `tools.web.search.perplexity.apiKey`)
per disambiguare.

Se non è impostato alcun base URL, OpenClaw sceglie un valore predefinito in base all&#39;origine della chiave API:

* `PERPLEXITY_API_KEY` o `pplx-...` → Perplexity diretto (`https://api.perplexity.ai`)
* `OPENROUTER_API_KEY` o `sk-or-...` → OpenRouter (`https://openrouter.ai/api/v1`)
* Formati di chiave sconosciuti → OpenRouter (fallback sicuro)

<div id="models">
  ## Modelli
</div>

* `perplexity/sonar` — domande e risposte veloci con ricerca sul web
* `perplexity/sonar-pro` (predefinito) — ragionamento a più passaggi + ricerca sul web
* `perplexity/sonar-reasoning-pro` — ricerca approfondita

Consulta [Web tools](/it/tools/web) per la configurazione completa di `web_search`.