---
title: Synthetic
summary: "Verwende die Anthropic-kompatible API von Synthetic in OpenClaw"
read_when:
  - Du möchtest Synthetic als Modellanbieter verwenden
  - Du musst einen Synthetic-API-Schlüssel oder eine Basis-URL konfigurieren
---

<div id="synthetic">
  # Synthetic
</div>

Synthetic bietet Anthropic-kompatible Endpunkte an. OpenClaw registriert es als
`synthetic` Anbieter und verwendet die Anthropic Messages-API.

<div id="quick-setup">
  ## Schnellstart
</div>

1. Richte `SYNTHETIC_API_KEY` ein (oder führe den Assistenten unten aus).
2. Starte das Onboarding:

```bash
openclaw onboard --auth-choice synthetic-api-key
```

Das Standardmodell ist eingestellt auf:

```
synthetic/hf:MiniMaxAI/MiniMax-M2.1
```

<div id="config-example">
  ## Beispielkonfiguration
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

Hinweis: Der Anthropic-Client von OpenClaw fügt der Basis-URL `/v1` hinzu; verwende daher
`https://api.synthetic.new/anthropic` (nicht `/anthropic/v1`). Wenn Synthetic seine
Basis-URL ändert, überschreibe `models.providers.synthetic.baseUrl`.

<div id="model-catalog">
  ## Modellkatalog
</div>

Alle unten aufgeführten Modelle verursachen Kosten von `0` (Input/Output/Cache).

| Modell-ID | Kontextfenster | Max. Tokens | Reasoning | Input |
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
  ## Hinweise
</div>

* Modell-Referenzen verwenden `synthetic/<modelId>`.
* Wenn du eine Modell-Allowlist (`agents.defaults.models`) aktivierst, füge jedes Modell hinzu,
  das du verwenden möchtest.
* Siehe [Modellanbieter](/de/concepts/model-providers) für Regeln zu Anbietern.