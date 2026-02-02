---
title: Synthetic
summary: "Usa la API compatible con Anthropic de Synthetic en OpenClaw"
read_when:
  - Quieres usar Synthetic como proveedor de modelos
  - Necesitas configurar una clave de API o una URL base de Synthetic
---

<div id="synthetic">
  # Synthetic
</div>

Synthetic expone endpoints compatibles con Anthropic. OpenClaw lo registra como
proveedor `synthetic` y utiliza la API Anthropic Messages.

<div id="quick-setup">
  ## Configuración rápida
</div>

1. Configura `SYNTHETIC_API_KEY` (o ejecuta el asistente que se muestra más abajo).
2. Ejecuta el proceso de onboarding:

```bash
openclaw onboard --auth-choice synthetic-api-key
```

El modelo predeterminado está configurado como:

```
synthetic/hf:MiniMaxAI/MiniMax-M2.1
```

<div id="config-example">
  ## Ejemplo de configuración
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

Nota: el cliente Anthropic de OpenClaw añade `/v1` a la URL base, así que utiliza
`https://api.synthetic.new/anthropic` (no `/anthropic/v1`). Si Synthetic cambia
su URL base, sobrescribe `models.providers.synthetic.baseUrl`.

<div id="model-catalog">
  ## Catálogo de modelos
</div>

Todos los modelos a continuación usan costo `0` (entrada/salida/caché).

| ID de modelo | Ventana de contexto | Tokens máximos | Razonamiento | Entrada |
| --- | --- | --- | --- | --- |
| `hf:MiniMaxAI/MiniMax-M2.1` | 192000 | 65536 | false | texto |
| `hf:moonshotai/Kimi-K2-Thinking` | 256000 | 8192 | true | texto |
| `hf:zai-org/GLM-4.7` | 198000 | 128000 | false | texto |
| `hf:deepseek-ai/DeepSeek-R1-0528` | 128000 | 8192 | false | texto |
| `hf:deepseek-ai/DeepSeek-V3-0324` | 128000 | 8192 | false | texto |
| `hf:deepseek-ai/DeepSeek-V3.1` | 128000 | 8192 | false | texto |
| `hf:deepseek-ai/DeepSeek-V3.1-Terminus` | 128000 | 8192 | false | texto |
| `hf:deepseek-ai/DeepSeek-V3.2` | 159000 | 8192 | false | texto |
| `hf:meta-llama/Llama-3.3-70B-Instruct` | 128000 | 8192 | false | texto |
| `hf:meta-llama/Llama-4-Maverick-17B-128E-Instruct-FP8` | 524000 | 8192 | false | texto |
| `hf:moonshotai/Kimi-K2-Instruct-0905` | 256000 | 8192 | false | texto |
| `hf:openai/gpt-oss-120b` | 128000 | 8192 | false | texto |
| `hf:Qwen/Qwen3-235B-A22B-Instruct-2507` | 256000 | 8192 | false | texto |
| `hf:Qwen/Qwen3-Coder-480B-A35B-Instruct` | 256000 | 8192 | false | texto |
| `hf:Qwen/Qwen3-VL-235B-A22B-Instruct` | 250000 | 8192 | false | texto + imagen |
| `hf:zai-org/GLM-4.5` | 128000 | 128000 | false | texto |
| `hf:zai-org/GLM-4.6` | 198000 | 128000 | false | texto |
| `hf:deepseek-ai/DeepSeek-V3` | 128000 | 8192 | false | texto |
| `hf:Qwen/Qwen3-235B-A22B-Thinking-2507` | 256000 | 8192 | true | texto |

<div id="notes">
  ## Notas
</div>

* Las referencias a modelos usan `synthetic/<modelId>`.
* Si habilitas una lista de permitidos de modelos (`agents.defaults.models`), agrega cada modelo que planees usar.
* Consulta la sección [Proveedores de modelos](/es/concepts/model-providers) para conocer las reglas de los proveedores.