---
title: Fournisseurs de modèles
summary: "Vue d’ensemble des fournisseurs de modèles avec exemples de configurations + workflows CLI"
read_when:
  - Vous avez besoin d’une référence de configuration des modèles par fournisseur
  - Vous voulez des exemples de configurations ou des commandes CLI de mise en route pour les fournisseurs de modèles
---

<div id="model-providers">
  # Fournisseurs de modèles
</div>

Cette page couvre les **fournisseurs de LLM/modèles** (et non les canaux de messagerie comme WhatsApp/Telegram).
Pour les règles de sélection des modèles, voir [/concepts/models](/fr/concepts/models).

<div id="quick-rules">
  ## Règles rapides
</div>

* Les références de modèles utilisent le format `provider/model` (exemple : `opencode/claude-opus-4-5`).
* Si vous définissez `agents.defaults.models`, cela devient la liste d’autorisation.
* Commandes CLI pratiques : `openclaw onboard`, `openclaw models list`, `openclaw models set <provider/model>`.

<div id="built-in-providers-pi-ai-catalog">
  ## Fournisseurs intégrés (catalogue pi-ai)
</div>

OpenClaw est livré avec le catalogue pi‑ai. Ces fournisseurs ne nécessitent **aucune**
configuration `models.providers` ; configurez simplement l’authentification et choisissez un modèle.

<div id="openai">
  ### OpenAI
</div>

* Fournisseur : `openai`
* Authentification : `OPENAI_API_KEY`
* Exemple de modèle : `openai/gpt-5.2`
* CLI : `openclaw onboard --auth-choice openai-api-key`

```json5
{
  agents: { defaults: { model: { primary: "openai/gpt-5.2" } } }
}
```

<div id="anthropic">
  ### Anthropic
</div>

* Fournisseur : `anthropic`
* Authentification : `ANTHROPIC_API_KEY` ou `claude setup-token`
* Modèle d’exemple : `anthropic/claude-opus-4-5`
* CLI : `openclaw onboard --auth-choice token` (collez le setup-token) ou `openclaw models auth paste-token --provider anthropic`

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="openai-code-codex">
  ### OpenAI Code (Codex)
</div>

* Fournisseur : `openai-codex`
* Authentification : OAuth (ChatGPT)
* Modèle d’exemple : `openai-codex/gpt-5.2`
* CLI : `openclaw onboard --auth-choice openai-codex` ou `openclaw models auth login --provider openai-codex`

```json5
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.2" } } }
}
```

<div id="opencode-zen">
  ### OpenCode Zen
</div>

* Fournisseur : `opencode`
* Authentification : `OPENCODE_API_KEY` (ou `OPENCODE_ZEN_API_KEY`)
* Exemple de modèle : `opencode/claude-opus-4-5`
* CLI : `openclaw onboard --auth-choice opencode-zen`

```json5
{
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-5" } } }
}
```

<div id="google-gemini-api-key">
  ### Google Gemini (clé API)
</div>

* Fournisseur : `google`
* Authentification : `GEMINI_API_KEY`
* Modèle d&#39;exemple : `google/gemini-3-pro-preview`
* CLI : `openclaw onboard --auth-choice gemini-api-key`

<div id="google-vertex-antigravity-gemini-cli">
  ### Google Vertex / Antigravity / Gemini CLI
</div>

* Fournisseurs : `google-vertex`, `google-antigravity`, `google-gemini-cli`
* Authentification : Vertex utilise gcloud ADC ; Antigravity/Gemini CLI utilisent chacun leur propre flux d’authentification
* L’OAuth Antigravity est fourni en tant que plugin intégré (`google-antigravity-auth`, désactivé par défaut).
  * Activer : `openclaw plugins enable google-antigravity-auth`
  * Connexion : `openclaw models auth login --provider google-antigravity --set-default`
* L’OAuth Gemini CLI est fourni en tant que plugin intégré (`google-gemini-cli-auth`, désactivé par défaut).
  * Activer : `openclaw plugins enable google-gemini-cli-auth`
  * Connexion : `openclaw models auth login --provider google-gemini-cli --set-default`
  * Remarque : vous ne devez **pas** coller d’identifiant client ni de secret dans `openclaw.json`. Le flux de connexion via la CLI stocke
    les jetons dans des profils d’authentification sur l’hôte du Gateway.

<div id="zai-glm">
  ### Z.AI (GLM)
</div>

* Fournisseur : `zai`
* Authentification : `ZAI_API_KEY`
* Modèle d&#39;exemple : `zai/glm-4.7`
* CLI : `openclaw onboard --auth-choice zai-api-key`
  * Alias : `z.ai/*` et `z-ai/*` sont normalisés vers `zai/*`

<div id="vercel-ai-gateway">
  ### Vercel AI Gateway
</div>

* Fournisseur : `vercel-ai-gateway`
* Authentification : `AI_GATEWAY_API_KEY`
* Exemple de modèle : `vercel-ai-gateway/anthropic/claude-opus-4.5`
* CLI : `openclaw onboard --auth-choice ai-gateway-api-key`

<div id="other-built-in-providers">
  ### Autres fournisseurs intégrés
</div>

* OpenRouter : `openrouter` (`OPENROUTER_API_KEY`)
* Modèle d’exemple : `openrouter/anthropic/claude-sonnet-4-5`
* xAI : `xai` (`XAI_API_KEY`)
* Groq : `groq` (`GROQ_API_KEY`)
* Cerebras : `cerebras` (`CEREBRAS_API_KEY`)
  * Les modèles GLM sur Cerebras utilisent les identifiants `zai-glm-4.7` et `zai-glm-4.6`.
  * URL de base compatible OpenAI : `https://api.cerebras.ai/v1`.
* Mistral : `mistral` (`MISTRAL_API_KEY`)
* GitHub Copilot : `github-copilot` (`COPILOT_GITHUB_TOKEN` / `GH_TOKEN` / `GITHUB_TOKEN`)

<div id="providers-via-modelsproviders-custombase-url">
  ## Fournisseurs via `models.providers` (URL personnalisée/base)
</div>

Utilisez `models.providers` (ou `models.json`) pour ajouter des fournisseurs **personnalisés** ou des proxys compatibles avec OpenAI/Anthropic.

<div id="moonshot-ai-kimi">
  ### Moonshot AI (Kimi)
</div>

Moonshot utilise des endpoints compatibles OpenAI ; configurez-le donc comme fournisseur personnalisé :

* Fournisseur : `moonshot`
* Auth : `MOONSHOT_API_KEY`
* Exemple de modèle : `moonshot/kimi-k2.5`
* Identifiants de modèles Kimi K2 :
  {/* moonshot-kimi-k2-model-refs:start */}
  * `moonshot/kimi-k2.5`
  * `moonshot/kimi-k2-0905-preview`
  * `moonshot/kimi-k2-turbo-preview`
  * `moonshot/kimi-k2-thinking`
  * `moonshot/kimi-k2-thinking-turbo`
  {/* moonshot-kimi-k2-model-refs:end */}

```json5
{
  agents: {
    defaults: { model: { primary: "moonshot/kimi-k2.5" } }
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [{ id: "kimi-k2.5", name: "Kimi K2.5" }]
      }
    }
  }
}
```

<div id="kimi-code">
  ### Kimi Code
</div>

Kimi Code utilise un point de terminaison et une clé d’API dédiés (distincts de Moonshot) :

* Fournisseur : `kimi-code`
* Authentification : `KIMICODE_API_KEY`
* Exemple de modèle : `kimi-code/kimi-for-coding`

```json5
{
  env: { KIMICODE_API_KEY: "sk-..." },
  agents: {
    defaults: { model: { primary: "kimi-code/kimi-for-coding" } }
  },
  models: {
    mode: "merge",
    providers: {
      "kimi-code": {
        baseUrl: "https://api.kimi.com/coding/v1",
        apiKey: "${KIMICODE_API_KEY}",
        api: "openai-completions",
        models: [{ id: "kimi-for-coding", name: "Kimi For Coding" }]
      }
    }
  }
}
```

<div id="qwen-oauth-free-tier">
  ### Qwen OAuth (offre gratuite)
</div>

Qwen fournit un accès OAuth à Qwen Coder + Vision via un flux de code d’appareil.
Activez le plugin intégré, puis connectez-vous :

```bash
openclaw plugins enable qwen-portal-auth
openclaw models auth login --provider qwen-portal --set-default
```

Références de modèles :

* `qwen-portal/coder-model`
* `qwen-portal/vision-model`

Voir [/providers/qwen](/fr/providers/qwen) pour les détails de configuration et des remarques.

<div id="synthetic">
  ### Synthetic
</div>

Synthetic fournit des modèles compatibles avec Anthropic via le fournisseur `synthetic` :

* Fournisseur : `synthetic`
* Authentification : `SYNTHETIC_API_KEY`
* Modèle d&#39;exemple : `synthetic/hf:MiniMaxAI/MiniMax-M2.1`
* CLI : `openclaw onboard --auth-choice synthetic-api-key`

```json5
{
  agents: {
    defaults: { model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" } }
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [{ id: "hf:MiniMaxAI/MiniMax-M2.1", name: "MiniMax M2.1" }]
      }
    }
  }
}
```

<div id="minimax">
  ### MiniMax
</div>

MiniMax est configuré via `models.providers` car il utilise des points de terminaison personnalisés :

* MiniMax (compatible Anthropic) : `--auth-choice minimax-api`
* Auth : `MINIMAX_API_KEY`

Voir [/providers/minimax](/fr/providers/minimax) pour la procédure de configuration, les options de modèles et des exemples de configuration.

<div id="ollama">
  ### Ollama
</div>

Ollama est un runtime LLM local qui fournit une API compatible OpenAI :

* Fournisseur : `ollama`
* Authentification : aucune (serveur local)
* Modèle d’exemple : `ollama/llama3.3`
* Installation : https://ollama.ai

```bash
# Installez Ollama, puis récupérez un modèle :
ollama pull llama3.3
```

```json5
{
  agents: {
    defaults: { model: { primary: "ollama/llama3.3" } }
  }
}
```

Ollama est détecté automatiquement lorsqu&#39;il est exécuté localement à `http://127.0.0.1:11434/v1`. Voir [/providers/ollama](/fr/providers/ollama) pour des recommandations de modèles et une configuration personnalisée.

<div id="local-proxies-lm-studio-vllm-litellm-etc">
  ### Proxies locaux (LM Studio, vLLM, LiteLLM, etc.)
</div>

Exemple (compatible OpenAI) :

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: { "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" } }
    }
  },
  models: {
    providers: {
      lmstudio: {
        baseUrl: "http://localhost:1234/v1",
        apiKey: "LMSTUDIO_KEY",
        api: "openai-completions",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 200000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

Remarques :

* Pour les fournisseurs personnalisés, `reasoning`, `input`, `cost`, `contextWindow` et `maxTokens` sont facultatifs.
  Lorsqu&#39;ils sont omis, OpenClaw applique les valeurs par défaut suivantes :
  * `reasoning: false`
  * `input: ["text"]`
  * `cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 }`
  * `contextWindow: 200000`
  * `maxTokens: 8192`
* Recommandé : définissez des valeurs explicites qui correspondent aux limites de votre proxy ou de votre modèle.

<div id="cli-examples">
  ## Exemples en CLI
</div>

```bash
openclaw onboard --auth-choice opencode-zen
openclaw models set opencode/claude-opus-4-5
openclaw models list
```

Voir aussi : [/gateway/configuration](/fr/gateway/configuration) pour des exemples complets de configuration.
