---
title: Moonshot
summary: "Configura Moonshot K2 vs Kimi Code (provider e chiavi separati)"
read_when:
  - Vuoi configurare Moonshot K2 (Moonshot Open Platform) o Kimi Code
  - Devi comprendere endpoint, chiavi e riferimenti ai modelli separati
  - Vuoi una configurazione copia/incolla per ciascun provider
---

<div id="moonshot-ai-kimi">
  # Moonshot AI (Kimi)
</div>

Moonshot fornisce l’API Kimi con endpoint compatibili con OpenAI. Configura il
provider e imposta il modello predefinito su `moonshot/kimi-k2.5`, oppure usa
Kimi Code con `kimi-code/kimi-for-coding`.

ID dei modelli Kimi K2 correnti:

{/* moonshot-kimi-k2-ids:start */}

* `kimi-k2.5`
* `kimi-k2-0905-preview`
* `kimi-k2-turbo-preview`
* `kimi-k2-thinking`
* `kimi-k2-thinking-turbo`

{/* moonshot-kimi-k2-ids:end */}

```bash
openclaw onboard --auth-choice moonshot-api-key
```

Codice Kimi:

```bash
openclaw onboard --auth-choice kimi-code-api-key
```

Nota: Moonshot e Kimi Code sono provider distinti. Le chiavi non sono intercambiabili, gli endpoint sono diversi e anche i riferimenti ai modelli (Moonshot usa `moonshot/...`, Kimi Code usa `kimi-code/...`).


<div id="config-snippet-moonshot-api">
  ## Esempio di configurazione (Moonshot API)
</div>

```json5
{
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: {
        // moonshot-kimi-k2-aliases:start
        "moonshot/kimi-k2.5": { alias: "Kimi K2.5" },
        "moonshot/kimi-k2-0905-preview": { alias: "Kimi K2" },
        "moonshot/kimi-k2-turbo-preview": { alias: "Kimi K2 Turbo" },
        "moonshot/kimi-k2-thinking": { alias: "Kimi K2 Thinking" },
        "moonshot/kimi-k2-thinking-turbo": { alias: "Kimi K2 Thinking Turbo" }
        // moonshot-kimi-k2-aliases:end
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          // moonshot-kimi-k2-models:start
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192
          },
          {
            id: "kimi-k2-0905-preview",
            name: "Kimi K2 0905 Preview",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192
          },
          {
            id: "kimi-k2-turbo-preview",
            name: "Kimi K2 Turbo",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192
          },
          {
            id: "kimi-k2-thinking",
            name: "Kimi K2 Thinking",
            reasoning: true,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192
          },
          {
            id: "kimi-k2-thinking-turbo",
            name: "Kimi K2 Thinking Turbo",
            reasoning: true,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192
          }
          // moonshot-kimi-k2-models:end
        ]
      }
    }
  }
}
```


<div id="kimi-code">
  ## Kimi Code
</div>

```json5
{
  env: { KIMICODE_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-code/kimi-for-coding" },
      models: {
        "kimi-code/kimi-for-coding": { alias: "Kimi Code" }
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      "kimi-code": {
        baseUrl: "https://api.kimi.com/coding/v1",
        apiKey: "${KIMICODE_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-for-coding",
            name: "Kimi For Coding",
            reasoning: true,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 262144,
            maxTokens: 32768,
            headers: { "User-Agent": "KimiCLI/0.77" },
            compat: { supportsDeveloperRole: false }
          }
        ]
      }
    }
  }
}
```


<div id="notes">
  ## Note
</div>

- I riferimenti ai modelli Moonshot usano `moonshot/<modelId>`. I riferimenti ai modelli Kimi Code usano `kimi-code/<modelId>`.
- Se necessario, sovrascrivi i metadati su tariffe e contesto in `models.providers`.
- Se Moonshot pubblica limiti di contesto diversi per un modello, aggiorna
  `contextWindow` di conseguenza.
- Usa `https://api.moonshot.cn/v1` se ti serve l'endpoint per la Cina.