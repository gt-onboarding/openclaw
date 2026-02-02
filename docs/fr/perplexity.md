---
title: Perplexity
summary: "Configuration de Perplexity Sonar pour web_search"
read_when:
  - Vous souhaitez utiliser Perplexity Sonar pour la recherche web
  - Vous devez configurer PERPLEXITY_API_KEY ou OpenRouter
---

<div id="perplexity-sonar">
  # Perplexity Sonar
</div>

OpenClaw peut utiliser Perplexity Sonar pour l’outil `web_search`. Vous pouvez vous connecter
directement via l’API directe de Perplexity ou via OpenRouter.

<div id="api-options">
  ## Options de l’API
</div>

<div id="perplexity-direct">
  ### Perplexity (direct)
</div>

* URL de base : https://api.perplexity.ai
* Variable d&#39;environnement : `PERPLEXITY_API_KEY`

<div id="openrouter-alternative">
  ### OpenRouter (alternative)
</div>

* URL de base : https://openrouter.ai/api/v1
* Variable d&#39;environnement : `OPENROUTER_API_KEY`
* Prend en charge les crédits prépayés/cryptographiques.

<div id="config-example">
  ## Exemple de configuration
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
  ## Migrer depuis Brave
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

Si `PERPLEXITY_API_KEY` et `OPENROUTER_API_KEY` sont tous les deux définis, définissez
`tools.web.search.perplexity.baseUrl` (ou `tools.web.search.perplexity.apiKey`)
pour lever toute ambiguïté.

Si aucune URL de base n&#39;est définie, OpenClaw choisit une valeur par défaut en fonction de la source de la clé d&#39;API :

* `PERPLEXITY_API_KEY` ou `pplx-...` → Perplexity direct (`https://api.perplexity.ai`)
* `OPENROUTER_API_KEY` ou `sk-or-...` → OpenRouter (`https://openrouter.ai/api/v1`)
* Formats de clé inconnus → OpenRouter (solution de repli sûre)

<div id="models">
  ## Modèles
</div>

* `perplexity/sonar` — questions-réponses rapides avec recherche sur le web
* `perplexity/sonar-pro` (par défaut) — raisonnement en plusieurs étapes + recherche sur le web
* `perplexity/sonar-reasoning-pro` — recherche approfondie

Voir [Outils web](/fr/tools/web) pour la configuration complète de `web_search`.