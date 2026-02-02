---
title: Moonshot
summary: "Configurer Moonshot K2 vs Kimi Code (fournisseurs et clés séparés)"
read_when:
  - Vous voulez configurer Moonshot K2 (Moonshot Open Platform) vs Kimi Code
  - Vous devez comprendre les endpoints, les clés et les références de modèles distincts
  - Vous voulez une configuration prête à copier/coller pour chaque fournisseur
---

<div id="moonshot-ai-kimi">
  # Moonshot AI (Kimi)
</div>

Moonshot fournit l’API Kimi avec des points de terminaison compatibles OpenAI. Configurez le
fournisseur et définissez le modèle par défaut sur `moonshot/kimi-k2.5`, ou utilisez
Kimi Code avec `kimi-code/kimi-for-coding`.

Identifiants de modèles Kimi K2 actuels :

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

Kimi Code :

```bash
openclaw onboard --auth-choice kimi-code-api-key
```

Remarque : Moonshot et Kimi Code sont des fournisseurs distincts. Les clés ne sont pas interchangeables, et les endpoints ainsi que les références de modèles ne sont pas les mêmes (Moonshot utilise `moonshot/...`, Kimi Code utilise `kimi-code/...`).


<div id="config-snippet-moonshot-api">
  ## Exemple de configuration (API Moonshot)
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
  ## Remarques
</div>

- Les références de modèles Moonshot utilisent `moonshot/<modelId>`. Les références de modèles Kimi Code utilisent `kimi-code/<modelId>`.
- Remplacez les métadonnées de tarification et de contexte dans `models.providers` si nécessaire.
- Si Moonshot publie des limites de contexte différentes pour un modèle, ajustez
  `contextWindow` en conséquence.
- Utilisez `https://api.moonshot.cn/v1` si vous avez besoin du point de terminaison pour la Chine.