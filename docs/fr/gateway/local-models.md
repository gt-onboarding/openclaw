---
title: Modèles locaux
summary: "Exécuter OpenClaw avec des LLM locaux (LM Studio, vLLM, LiteLLM, endpoints OpenAI personnalisés)"
read_when:
  - Vous voulez servir des modèles depuis votre propre machine équipée d’un GPU
  - Vous intégrez LM Studio ou un proxy compatible OpenAI
  - Vous avez besoin des consignes de sécurité les plus strictes pour les modèles locaux
---

<div id="local-models">
  # Modèles locaux
</div>

Les modèles locaux sont possibles, mais OpenClaw suppose un large contexte et des défenses solides contre l’injection de prompt. Des cartes GPU trop limitées tronquent le contexte et affaiblissent la sécurité. Visez haut : **≥2 Mac Studio au maximum de leur configuration ou une station GPU équivalente (~30 000 $+)**. Un seul GPU de **24 Go** ne convient qu’à des prompts plus légers, avec une latence plus élevée. Utilisez la **plus grande variante de modèle / variante pleine taille que vous pouvez faire tourner** ; des checkpoints agressivement quantifiés ou « petits » augmentent le risque d’injection de prompt (voir [Sécurité](/fr/gateway/security)).

<div id="recommended-lm-studio-minimax-m21-responses-api-full-size">
  ## Recommandé : LM Studio + MiniMax M2.1 (Responses API, taille complète)
</div>

Meilleure pile locale actuelle. Charge MiniMax M2.1 dans LM Studio, active le serveur local (par défaut `http://127.0.0.1:1234`) et utilise Responses API pour garder le raisonnement séparé du texte final.

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" }
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

**Liste de vérification d’installation**

* Installe LM Studio : https://lmstudio.ai
* Dans LM Studio, télécharge **la plus grande version MiniMax M2.1 disponible** (évite d’utiliser les variantes « small »/très quantisées), démarre le serveur, et vérifie que `http://127.0.0.1:1234/v1/models` la liste bien.
* Garde le modèle chargé ; un chargement à froid ajoute de la latence au démarrage.
* Ajuste `contextWindow`/`maxTokens` si ta build LM Studio diffère.
* Pour WhatsApp, reste sur Responses API afin que seul le texte final soit envoyé.

Laisse les modèles hébergés configurés même lorsque tu exécutes des modèles en local ; utilise `models.mode: "merge"` pour que les mécanismes de repli restent disponibles.

<div id="hybrid-config-hosted-primary-local-fallback">
  ### Configuration hybride : primaire hébergé, secours local
</div>

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["lmstudio/minimax-m2.1-gs32", "anthropic/claude-opus-4-5"]
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "lmstudio/minimax-m2.1-gs32": { alias: "MiniMax Local" },
        "anthropic/claude-opus-4-5": { alias: "Opus" }
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

<div id="local-first-with-hosted-safety-net">
  ### Local-first avec filet de sécurité hébergé
</div>

Inverse l’ordre principal et de secours ; conserve le même bloc de fournisseurs et `models.mode: "merge"` afin de pouvoir basculer vers Sonnet ou Opus lorsque la machine locale est indisponible.

<div id="regional-hosting-data-routing">
  ### Hébergement régional / routage des données
</div>

* Des variantes MiniMax/Kimi/GLM hébergées existent également sur OpenRouter avec des endpoints associés à une région donnée (par exemple, hébergés aux États-Unis). Choisissez la variante régionale correspondante pour garder le trafic dans la juridiction de votre choix tout en continuant à utiliser `models.mode: "merge"` pour les mécanismes de repli Anthropic/OpenAI.
* Le mode strictement local reste l’option la plus robuste en matière de confidentialité ; le routage régional hébergé constitue un compromis lorsque vous avez besoin des fonctionnalités du fournisseur mais que vous voulez garder le contrôle sur les flux de données.

<div id="other-openai-compatible-local-proxies">
  ## Autres proxys locaux compatibles avec OpenAI
</div>

vLLM, LiteLLM, OAI-proxy ou des passerelles personnalisées fonctionnent s’ils exposent un point de terminaison `/v1` au format OpenAI. Remplace le bloc de fournisseur ci-dessus par ton point de terminaison et l’ID de modèle :

```json5
{
  models: {
    mode: "merge",
    providers: {
      local: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "sk-local",
        api: "openai-responses",
        models: [
          {
            id: "my-local-model",
            name: "Local Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 120000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

Laissez `models.mode: "merge"` afin que les modèles hébergés restent disponibles en solution de repli.

<div id="troubleshooting">
  ## Dépannage
</div>

* Gateway peut-il atteindre le proxy ? `curl http://127.0.0.1:1234/v1/models`.
* Modèle LM Studio déchargé ? Rechargez-le ; un démarrage à froid est une cause fréquente de blocages.
* Erreurs de contexte ? Réduisez `contextWindow` ou augmentez la limite de votre serveur.
* Sécurité : les modèles locaux contournent les filtres côté fournisseur ; gardez les agents très ciblés et laissez la compaction activée pour limiter le rayon d’explosion des attaques par injection de prompt.