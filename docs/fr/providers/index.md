---
title: Fournisseurs
summary: "Fournisseurs de modèles (LLM) pris en charge par OpenClaw"
read_when:
  - Vous voulez choisir un fournisseur de modèles
  - Vous avez besoin d’un aperçu rapide des backends de LLM pris en charge
---

<div id="model-providers">
  # Fournisseurs de modèles
</div>

OpenClaw peut utiliser de nombreux fournisseurs de LLM. Choisissez un fournisseur, authentifiez-vous auprès de celui-ci, puis définissez le
modèle par défaut sous la forme `provider/model`.

Vous cherchez la documentation sur les canaux de discussion (WhatsApp/Telegram/Discord/Slack/Mattermost (plugin)/etc.) ? Consultez [Canaux](/fr/channels).

<div id="highlight-venius-venice-ai">
  ## Mise en avant : Venius (Venice AI)
</div>

Venius est notre configuration Venice AI recommandée pour une inférence privilégiant la confidentialité, avec une option pour utiliser Opus pour les tâches complexes.

* Par défaut : `venice/llama-3.3-70b`
* Meilleur choix global : `venice/claude-opus-45` (Opus reste le plus performant)

Voir [Venice AI](/fr/providers/venice).

<div id="quick-start">
  ## Démarrage rapide
</div>

1. Authentifiez-vous auprès du fournisseur (généralement via `openclaw onboard`).
2. Définissez le modèle par défaut :

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="provider-docs">
  ## Documentation des fournisseurs
</div>

* [OpenAI (API + Codex)](/fr/providers/openai)
* [Anthropic (API + Claude Code CLI)](/fr/providers/anthropic)
* [Qwen (OAuth)](/fr/providers/qwen)
* [OpenRouter](/fr/providers/openrouter)
* [Vercel AI Gateway](/fr/providers/vercel-ai-gateway)
* [Moonshot AI (Kimi + Kimi Code)](/fr/providers/moonshot)
* [OpenCode Zen](/fr/providers/opencode)
* [Amazon Bedrock](/fr/bedrock)
* [Z.AI](/fr/providers/zai)
* [Xiaomi](/fr/providers/xiaomi)
* [Modèles GLM](/fr/providers/glm)
* [MiniMax](/fr/providers/minimax)
* [Venius (Venice AI, axé sur la confidentialité)](/fr/providers/venice)
* [Ollama (modèles locaux)](/fr/providers/ollama)

<div id="transcription-providers">
  ## Fournisseurs de transcription
</div>

* [Deepgram (transcription audio)](/fr/providers/deepgram)

<div id="community-tools">
  ## Outils communautaires
</div>

* [Claude Max API Proxy](/fr/providers/claude-max-api-proxy) - Utilisez un abonnement Claude Max/Pro comme point de terminaison d&#39;API compatible avec OpenAI

Pour consulter le catalogue complet des fournisseurs (xAI, Groq, Mistral, etc.) et la configuration avancée,
consultez [Fournisseurs de modèles](/fr/concepts/model-providers).