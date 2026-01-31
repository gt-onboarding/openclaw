---
title: OpenRouter
summary: "Utilisez l'API unifiée d'OpenRouter pour accéder à de nombreux modèles dans OpenClaw"
read_when:
  - Vous souhaitez une seule clé API pour de nombreux LLMs
  - Vous souhaitez exécuter des modèles via OpenRouter dans OpenClaw
---

<div id="openrouter">
  # OpenRouter
</div>

OpenRouter fournit une **API unifiée** qui achemine les requêtes vers de nombreux modèles via un seul point de terminaison et une seule clé API. Il est compatible avec OpenAI, ce qui permet à la plupart des SDK OpenAI de fonctionner simplement en changeant l’URL de base.

<div id="cli-setup">
  ## Configuration de la CLI
</div>

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

<div id="config-snippet">
  ## Exemple de configuration
</div>

```json5
{
  env: { OPENROUTER_API_KEY: "sk-or-..." },
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" }
    }
  }
}
```

<div id="notes">
  ## Remarques
</div>

* Les références de modèles sont `openrouter/<provider>/<model>`.
* Pour davantage d’options de modèles et de fournisseurs, consultez [/concepts/model-providers](/fr/concepts/model-providers).
* OpenRouter utilise en interne un jeton d’authentification Bearer avec votre clé API.