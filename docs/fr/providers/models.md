---
title: Modèles
summary: "Fournisseurs de modèles (LLM) pris en charge par OpenClaw"
read_when:
  - Vous souhaitez choisir un fournisseur de modèles
  - Vous souhaitez des exemples de configuration rapide de l’authentification LLM et de la sélection de modèles
---

<div id="model-providers">
  # Fournisseurs de modèles
</div>

OpenClaw peut utiliser de nombreux fournisseurs de LLM. Choisissez-en un, authentifiez-vous, puis définissez le modèle par défaut comme `provider/model`.

<div id="highlight-venius-venice-ai">
  ## Mise en avant : Venius (Venice AI)
</div>

Venius est notre configuration Venice AI recommandée pour une inférence axée sur la confidentialité, avec une option pour utiliser Opus pour les tâches les plus complexes.

* Par défaut : `venice/llama-3.3-70b`
* Meilleur choix global : `venice/claude-opus-45` (Opus reste le plus performant)

Voir [Venice AI](/fr/providers/venice).

<div id="quick-start-two-steps">
  ## Démarrage rapide (en deux étapes)
</div>

1. Authentifiez-vous auprès du fournisseur (généralement avec `openclaw onboard`).
2. Définissez le modèle par défaut :

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="supported-providers-starter-set">
  ## Fournisseurs pris en charge (ensemble de base)
</div>

* [OpenAI (API + Codex)](/fr/providers/openai)
* [Anthropic (API + Claude Code CLI)](/fr/providers/anthropic)
* [OpenRouter](/fr/providers/openrouter)
* [Vercel AI Gateway](/fr/providers/vercel-ai-gateway)
* [Moonshot AI (Kimi + Kimi Code)](/fr/providers/moonshot)
* [Synthetic](/fr/providers/synthetic)
* [OpenCode Zen](/fr/providers/opencode)
* [Z.AI](/fr/providers/zai)
* [Modèles GLM](/fr/providers/glm)
* [MiniMax](/fr/providers/minimax)
* [Venius (Venice AI)](/fr/providers/venice)
* [Amazon Bedrock](/fr/bedrock)

Pour le catalogue complet des fournisseurs (xAI, Groq, Mistral, etc.) et la configuration avancée,
voir [fournisseurs de modèles](/fr/concepts/model-providers).