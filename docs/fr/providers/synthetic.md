---
title: Synthetic
summary: "Utiliser l'API compatible Anthropic de Synthetic dans OpenClaw"
read_when:
  - Vous souhaitez utiliser Synthetic comme fournisseur de modèles
  - Vous devez configurer une clé API Synthetic ou une URL de base
---

<div id="synthetic">
  # Synthetic
</div>

Synthetic fournit des endpoints compatibles avec Anthropic. OpenClaw l’enregistre comme le fournisseur `synthetic` et utilise l’API Messages d’Anthropic.

<div id="quick-setup">
  ## Configuration rapide
</div>

1. Définissez `SYNTHETIC_API_KEY` (ou exécutez l’assistant ci-dessous).
2. Démarrez l’onboarding :

```bash
openclaw onboard --auth-choice synthetic-api-key
```

Le modèle par défaut est :

```
synthetic/hf:MiniMaxAI/MiniMax-M2.1
```

<div id="config-example">
  ## Exemple de configuration
</div>

```json5
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.1": { alias: "MiniMax M2.1" } }
    }
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536
          }
        ]
      }
    }
  }
}
```

Remarque : le client Anthropic d&#39;OpenClaw ajoute `/v1` à l&#39;URL de base, utilisez donc
`https://api.synthetic.new/anthropic` (et non `/anthropic/v1`). Si Synthetic modifie
son URL de base, remplacez la valeur de `models.providers.synthetic.baseUrl`.

<div id="model-catalog">
  ## Catalogue de modèles
</div>

Tous les modèles ci-dessous ont un coût de `0` (entrée/sortie/cache).

| ID du modèle | Fenêtre de contexte | Nombre maximal de tokens | Raisonnement | Entrée |
| --- | --- | --- | --- | --- |
| `hf:MiniMaxAI/MiniMax-M2.1` | 192000 | 65536 | false | texte |
| `hf:moonshotai/Kimi-K2-Thinking` | 256000 | 8192 | true | texte |
| `hf:zai-org/GLM-4.7` | 198000 | 128000 | false | texte |
| `hf:deepseek-ai/DeepSeek-R1-0528` | 128000 | 8192 | false | texte |
| `hf:deepseek-ai/DeepSeek-V3-0324` | 128000 | 8192 | false | texte |
| `hf:deepseek-ai/DeepSeek-V3.1` | 128000 | 8192 | false | texte |
| `hf:deepseek-ai/DeepSeek-V3.1-Terminus` | 128000 | 8192 | false | texte |
| `hf:deepseek-ai/DeepSeek-V3.2` | 159000 | 8192 | false | texte |
| `hf:meta-llama/Llama-3.3-70B-Instruct` | 128000 | 8192 | false | texte |
| `hf:meta-llama/Llama-4-Maverick-17B-128E-Instruct-FP8` | 524000 | 8192 | false | texte |
| `hf:moonshotai/Kimi-K2-Instruct-0905` | 256000 | 8192 | false | texte |
| `hf:openai/gpt-oss-120b` | 128000 | 8192 | false | texte |
| `hf:Qwen/Qwen3-235B-A22B-Instruct-2507` | 256000 | 8192 | false | texte |
| `hf:Qwen/Qwen3-Coder-480B-A35B-Instruct` | 256000 | 8192 | false | texte |
| `hf:Qwen/Qwen3-VL-235B-A22B-Instruct` | 250000 | 8192 | false | texte + image |
| `hf:zai-org/GLM-4.5` | 128000 | 128000 | false | texte |
| `hf:zai-org/GLM-4.6` | 198000 | 128000 | false | texte |
| `hf:deepseek-ai/DeepSeek-V3` | 128000 | 8192 | false | texte |
| `hf:Qwen/Qwen3-235B-A22B-Thinking-2507` | 256000 | 8192 | true | texte |

<div id="notes">
  ## Remarques
</div>

* Les références de modèle utilisent `synthetic/<modelId>`.
* Si vous activez une liste d’autorisation de modèles (`agents.defaults.models`), ajoutez chaque modèle
  que vous prévoyez d’utiliser.
* Voir [Fournisseurs de modèles](/fr/concepts/model-providers) pour les règles applicables aux fournisseurs.