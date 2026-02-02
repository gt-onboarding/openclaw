---
title: Synthetic
summary: "Usa l'API compatibile con Anthropic di Synthetic in OpenClaw"
read_when:
  - Vuoi utilizzare Synthetic come provider di modelli
  - Hai bisogno di una chiave API di Synthetic o di configurare l'URL di base
---

<div id="synthetic">
  # Synthetic
</div>

Synthetic espone endpoint compatibili con Anthropic. OpenClaw lo registra come
provider `synthetic` e utilizza l&#39;API Anthropic Messages.

<div id="quick-setup">
  ## Configurazione rapida
</div>

1. Imposta `SYNTHETIC_API_KEY` (oppure esegui la procedura guidata di seguito).
2. Esegui l&#39;onboarding:

```bash
openclaw onboard --auth-choice synthetic-api-key
```

Il modello predefinito è:

```
synthetic/hf:MiniMaxAI/MiniMax-M2.1
```

<div id="config-example">
  ## Esempio di configurazione
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

Nota: il client Anthropic di OpenClaw aggiunge `/v1` all’URL di base, quindi devi usare
`https://api.synthetic.new/anthropic` (non `/anthropic/v1`). Se Synthetic modifica
il proprio URL di base, sovrascrivi `models.providers.synthetic.baseUrl`.

<div id="model-catalog">
  ## Catalogo dei modelli
</div>

Tutti i modelli elencati di seguito hanno costo `0` (input/output/cache).

| Model ID | Finestra di contesto | Token massimi | Ragionamento | Input |
| --- | --- | --- | --- | --- |
| `hf:MiniMaxAI/MiniMax-M2.1` | 192000 | 65536 | false | text |
| `hf:moonshotai/Kimi-K2-Thinking` | 256000 | 8192 | true | text |
| `hf:zai-org/GLM-4.7` | 198000 | 128000 | false | text |
| `hf:deepseek-ai/DeepSeek-R1-0528` | 128000 | 8192 | false | text |
| `hf:deepseek-ai/DeepSeek-V3-0324` | 128000 | 8192 | false | text |
| `hf:deepseek-ai/DeepSeek-V3.1` | 128000 | 8192 | false | text |
| `hf:deepseek-ai/DeepSeek-V3.1-Terminus` | 128000 | 8192 | false | text |
| `hf:deepseek-ai/DeepSeek-V3.2` | 159000 | 8192 | false | text |
| `hf:meta-llama/Llama-3.3-70B-Instruct` | 128000 | 8192 | false | text |
| `hf:meta-llama/Llama-4-Maverick-17B-128E-Instruct-FP8` | 524000 | 8192 | false | text |
| `hf:moonshotai/Kimi-K2-Instruct-0905` | 256000 | 8192 | false | text |
| `hf:openai/gpt-oss-120b` | 128000 | 8192 | false | text |
| `hf:Qwen/Qwen3-235B-A22B-Instruct-2507` | 256000 | 8192 | false | text |
| `hf:Qwen/Qwen3-Coder-480B-A35B-Instruct` | 256000 | 8192 | false | text |
| `hf:Qwen/Qwen3-VL-235B-A22B-Instruct` | 250000 | 8192 | false | text + image |
| `hf:zai-org/GLM-4.5` | 128000 | 128000 | false | text |
| `hf:zai-org/GLM-4.6` | 198000 | 128000 | false | text |
| `hf:deepseek-ai/DeepSeek-V3` | 128000 | 8192 | false | text |
| `hf:Qwen/Qwen3-235B-A22B-Thinking-2507` | 256000 | 8192 | true | text |

<div id="notes">
  ## Note
</div>

* I riferimenti ai modelli utilizzano `synthetic/<modelId>`.
* Se abiliti una lista di autorizzati per i modelli (`agents.defaults.models`), aggiungi tutti i modelli che prevedi di usare.
* Consulta [Model providers](/it/concepts/model-providers) per le regole sui provider.