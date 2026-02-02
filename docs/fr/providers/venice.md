---
title: Venice
summary: "Utiliser les modèles Venice AI axés sur le respect de la vie privée dans OpenClaw"
read_when:
  - Vous souhaitez effectuer des inférences respectueuses de la vie privée dans OpenClaw
  - Vous avez besoin d’un guide de configuration pour Venice AI
---

<div id="venice-ai-venice-highlight">
  # Venice AI (Venice en vedette)
</div>

**Venice** est notre configuration Venice phare pour une inférence axée sur la confidentialité, avec accès optionnel et anonymisé à des modèles propriétaires.

Venice AI fournit une inférence IA axée sur la confidentialité, avec prise en charge de modèles non censurés et accès aux principaux modèles propriétaires via leur proxy anonymisé. Toute l’inférence est privée par défaut : aucun entraînement sur vos données, aucune journalisation.

<div id="why-venice-in-openclaw">
  ## Pourquoi Venice avec OpenClaw
</div>

- **Inférence privée** pour les modèles open source (aucune journalisation).
- **Modèles non censurés** lorsque vous en avez besoin.
- **Accès anonymisé** aux modèles propriétaires (Opus/GPT/Gemini) lorsque la qualité est cruciale.
- Points de terminaison `/v1` compatibles avec OpenAI.

<div id="privacy-modes">
  ## Modes de confidentialité
</div>

Venice propose deux niveaux de confidentialité — comprendre ces niveaux est essentiel pour choisir votre modèle :

| Mode | Description | Modèles |
|------|-------------|--------|
| **Privé** | Entièrement privé. Les prompts/réponses ne sont **jamais stockés ni consignés**. Éphémère. | Llama, Qwen, DeepSeek, Venice Uncensored, etc. |
| **Anonymisé** | Requêtes transitant via Venice, avec les métadonnées supprimées. Le fournisseur sous-jacent (OpenAI, Anthropic) ne voit que des requêtes anonymisées. | Claude, GPT, Gemini, Grok, Kimi, MiniMax |

<div id="features">
  ## Fonctionnalités
</div>

- **Axé sur la confidentialité** : choisissez entre les modes « private » (entièrement privé) et « anonymized » (via proxy)
- **Modèles non censurés** : accès à des modèles sans restrictions de contenu
- **Accès aux principaux modèles** : utilisez Claude, GPT-5.2, Gemini, Grok via le proxy anonymisé de Venice
- **API compatible OpenAI** : points de terminaison `/v1` standard pour une intégration facile
- **Streaming** : ✅ pris en charge pour tous les modèles
- **Function calling** : ✅ pris en charge pour certains modèles (vérifiez les capacités du modèle)
- **Vision** : ✅ pris en charge pour les modèles avec fonctionnalité de vision
- **Pas de limites de débit strictes** : une limitation basée sur un usage équitable peut s’appliquer en cas d’utilisation extrême

<div id="setup">
  ## Configuration
</div>

<div id="1-get-api-key">
  ### 1. Obtenir la clé API
</div>

1. Créez un compte sur [venice.ai](https://venice.ai)
2. Accédez à **Settings → API Keys → Create new key**
3. Copiez votre clé API (format : `vapi_xxxxxxxxxxxx`)

<div id="2-configure-openclaw">
  ### 2. Configurer OpenClaw
</div>

**Option A : Variable d’environnement**

```bash
export VENICE_API_KEY="vapi_xxxxxxxxxxxx"
```

**Option B : Configuration interactive (recommandée)**

```bash
openclaw onboard --auth-choice venice-api-key
```

Cette commande va :

1. Vous demander votre clé API (ou utiliser la variable d’environnement `VENICE_API_KEY` existante)
2. Afficher tous les modèles Venice disponibles
3. Vous permettre de choisir votre modèle par défaut
4. Configurer automatiquement le fournisseur

**Option C : mode non interactif**

```bash
openclaw onboard --non-interactive \
  --auth-choice venice-api-key \
  --venice-api-key "vapi_xxxxxxxxxxxx"
```


<div id="3-verify-setup">
  ### 3. Vérifier la configuration
</div>

```bash
openclaw chat --model venice/llama-3.3-70b "Hello, are you working?"
```


<div id="model-selection">
  ## Sélection du modèle
</div>

Après la configuration, OpenClaw affiche tous les modèles Venice disponibles. Choisissez en fonction de vos besoins :

* **Par défaut (notre choix)** : `venice/llama-3.3-70b` pour des performances équilibrées tout en préservant la confidentialité.
* **Meilleure qualité globale** : `venice/claude-opus-45` pour les tâches difficiles (Opus reste le plus performant).
* **Confidentialité** : choisissez des modèles « private » pour une inférence entièrement privée.
* **Capacités** : choisissez des modèles « anonymized » pour accéder à Claude, GPT et Gemini via le proxy de Venice.

Modifiez votre modèle par défaut à tout moment :

```bash
openclaw models set venice/claude-opus-45
openclaw models set venice/llama-3.3-70b
```

Afficher tous les modèles disponibles :

```bash
openclaw models list | grep venice
```


<div id="configure-via-openclaw-configure">
  ## Configurer via `openclaw configure`
</div>

1. Exécutez `openclaw configure`
2. Sélectionnez **Model/auth**
3. Choisissez **Venice AI**

<div id="which-model-should-i-use">
  ## Quel modèle dois-je utiliser ?
</div>

| Cas d’utilisation | Modèle recommandé | Pourquoi |
|-------------------|-------------------|----------|
| **Discussion générale** | `llama-3.3-70b` | Modèle polyvalent, entièrement privé |
| **Meilleure qualité globale** | `claude-opus-45` | Opus reste le plus performant sur les tâches difficiles |
| **Confidentialité + qualité Claude** | `claude-opus-45` | Meilleur raisonnement via proxy anonymisé |
| **Code** | `qwen3-coder-480b-a35b-instruct` | Optimisé pour le code, contexte 262k |
| **Tâches de vision** | `qwen3-vl-235b-a22b` | Meilleur modèle de vision privé |
| **Non censuré** | `venice-uncensored` | Aucune restriction de contenu |
| **Rapide + économique** | `qwen3-4b` | Léger mais toujours performant |
| **Raisonnement complexe** | `deepseek-v3.2` | Raisonnement puissant, privé |

<div id="available-models-25-total">
  ## Modèles disponibles (25 au total)
</div>

<div id="private-models-15-fully-private-no-logging">
  ### Modèles privés (15) — Entièrement privés, sans journalisation
</div>

| Model ID | Nom | Contexte (jetons) | Fonctionnalités |
|----------|------|------------------|----------|
| `llama-3.3-70b` | Llama 3.3 70B | 131k | Général |
| `llama-3.2-3b` | Llama 3.2 3B | 131k | Rapide, léger |
| `hermes-3-llama-3.1-405b` | Hermes 3 Llama 3.1 405B | 131k | Tâches complexes |
| `qwen3-235b-a22b-thinking-2507` | Qwen3 235B Thinking | 131k | Raisonnement |
| `qwen3-235b-a22b-instruct-2507` | Qwen3 235B Instruct | 131k | Général |
| `qwen3-coder-480b-a35b-instruct` | Qwen3 Coder 480B | 262k | Code |
| `qwen3-next-80b` | Qwen3 Next 80B | 262k | Général |
| `qwen3-vl-235b-a22b` | Qwen3 VL 235B | 262k | Vision |
| `qwen3-4b` | Venice Small (Qwen3 4B) | 32k | Rapide, raisonnement |
| `deepseek-v3.2` | DeepSeek V3.2 | 163k | Raisonnement |
| `venice-uncensored` | Venice Uncensored | 32k | Non censuré |
| `mistral-31-24b` | Venice Medium (Mistral) | 131k | Vision |
| `google-gemma-3-27b-it` | Gemma 3 27B Instruct | 202k | Vision |
| `openai-gpt-oss-120b` | OpenAI GPT OSS 120B | 131k | Général |
| `zai-org-glm-4.7` | GLM 4.7 | 202k | Raisonnement, multilingue |

<div id="anonymized-models-10-via-venice-proxy">
  ### Modèles anonymisés (10) — via le proxy Venice
</div>

| ID du modèle | Original | Contexte (jetons) | Capacités |
|----------|----------|------------------|----------|
| `claude-opus-45` | Claude Opus 4.5 | 202k | Raisonnement, vision |
| `claude-sonnet-45` | Claude Sonnet 4.5 | 202k | Raisonnement, vision |
| `openai-gpt-52` | GPT-5.2 | 262k | Raisonnement |
| `openai-gpt-52-codex` | GPT-5.2 Codex | 262k | Raisonnement, vision |
| `gemini-3-pro-preview` | Gemini 3 Pro | 202k | Raisonnement, vision |
| `gemini-3-flash-preview` | Gemini 3 Flash | 262k | Raisonnement, vision |
| `grok-41-fast` | Grok 4.1 Fast | 262k | Raisonnement, vision |
| `grok-code-fast-1` | Grok Code Fast 1 | 262k | Raisonnement, code |
| `kimi-k2-thinking` | Kimi K2 Thinking | 262k | Raisonnement |
| `minimax-m21` | MiniMax M2.1 | 202k | Raisonnement |

<div id="model-discovery">
  ## Découverte de modèles
</div>

OpenClaw découvre automatiquement les modèles à partir de l'API Venice lorsque `VENICE_API_KEY` est défini. Si l'API est inaccessible, il bascule sur un catalogue statique.

L'endpoint `/models` est public (aucune authentification requise pour lister les modèles), mais l'inférence nécessite une clé d'API valide.

<div id="streaming-tool-support">
  ## Prise en charge du streaming et des outils
</div>

| Fonctionnalité | Prise en charge |
|----------------|-----------------|
| **Streaming** | ✅ Tous les modèles |
| **Appel de fonctions** | ✅ La plupart des modèles (vérifiez `supportsFunctionCalling` dans l’api) |
| **Vision/Images** | ✅ Modèles marqués avec la fonctionnalité « Vision » |
| **Mode JSON** | ✅ Pris en charge via `response_format` |

<div id="pricing">
  ## Tarification
</div>

Venice utilise un système de crédits. Consultez [venice.ai/pricing](https://venice.ai/pricing) pour les tarifs actuels :

- **Modèles privés** : Coût généralement inférieur
- **Modèles anonymisés** : Tarifs similaires à ceux de l’API directe, plus une petite commission Venice

<div id="comparison-venice-vs-direct-api">
  ## Comparaison : Venice vs API directe
</div>

| Aspect | Venice (anonymisé) | API directe |
|--------|---------------------|------------|
| **Confidentialité** | Métadonnées supprimées, anonymisées | Votre compte associé |
| **Latence** | +10-50 ms (proxy) | Connexion directe |
| **Fonctionnalités** | Prise en charge de la plupart des fonctionnalités | Toutes les fonctionnalités disponibles |
| **Facturation** | Crédits Venice | Facturation par le fournisseur |

<div id="usage-examples">
  ## Exemples d’utilisation
</div>

```bash
# Use default private model
openclaw chat --model venice/llama-3.3-70b

# Utiliser Claude via Venice (anonymisé)
openclaw chat --model venice/claude-opus-45

# Use uncensored model
openclaw chat --model venice/venice-uncensored

# Use vision model with image
openclaw chat --model venice/qwen3-vl-235b-a22b

# Use coding model
openclaw chat --model venice/qwen3-coder-480b-a35b-instruct
```


<div id="troubleshooting">
  ## Dépannage
</div>

<div id="api-key-not-recognized">
  ### Clé d’API non reconnue
</div>

```bash
echo $VENICE_API_KEY
openclaw models list | grep venice
```

Assurez-vous que la clé commence par `vapi_`.


<div id="model-not-available">
  ### Modèle non disponible
</div>

Le catalogue de modèles Venice est mis à jour dynamiquement. Exécutez la commande `openclaw models list` pour afficher les modèles actuellement disponibles. Certains modèles peuvent être temporairement indisponibles.

<div id="connection-issues">
  ### Problèmes de connexion
</div>

L’API Venice est accessible à `https://api.venice.ai/api/v1`. Assurez-vous que votre réseau autorise les connexions HTTPS.

<div id="config-file-example">
  ## Exemple de fichier de configuration
</div>

```json5
{
  env: { VENICE_API_KEY: "vapi_..." },
  agents: { defaults: { model: { primary: "venice/llama-3.3-70b" } } },
  models: {
    mode: "merge",
    providers: {
      venice: {
        baseUrl: "https://api.venice.ai/api/v1",
        apiKey: "${VENICE_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "llama-3.3-70b",
            name: "Llama 3.3 70B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 131072,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```


<div id="links">
  ## Liens
</div>

- [Venice AI](https://venice.ai)
- [Documentation de l'API](https://docs.venice.ai)
- [Tarifs](https://venice.ai/pricing)
- [État du service](https://status.venice.ai)