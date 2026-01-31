---
title: Moonshot
summary: "Konfiguration von Moonshot K2 vs Kimi Code (separate Anbieter + Keys)"
read_when:
  - Du möchtest ein Setup mit Moonshot K2 (Moonshot Open Platform) im Vergleich zu Kimi Code
  - Du musst separate Endpunkte, Keys und Modellreferenzen verstehen
  - Du möchtest eine Copy/Paste-Konfiguration für einen der beiden Anbieter
---

<div id="moonshot-ai-kimi">
  # Moonshot AI (Kimi)
</div>

Moonshot stellt die Kimi-API mit OpenAI-kompatiblen Endpunkten bereit. Konfiguriere den
Anbieter und setze das Standardmodell auf `moonshot/kimi-k2.5`, oder verwende
Kimi Code mit `kimi-code/kimi-for-coding`.

Aktuelle Kimi-K2-Modell-IDs:

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

Kimi-Code:

```bash
openclaw onboard --auth-choice kimi-code-api-key
```

Hinweis: Moonshot und Kimi Code sind separate Anbieter. Die Schlüssel sind nicht austauschbar, und sowohl die Endpunkte als auch die Modell-Referenzen unterscheiden sich (Moonshot verwendet `moonshot/...`, Kimi Code verwendet `kimi-code/...`).


<div id="config-snippet-moonshot-api">
  ## Konfigurationsbeispiel (Moonshot API)
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
  ## Hinweise
</div>

- Moonshot-Modellreferenzen verwenden `moonshot/<modelId>`. Kimi-Code-Modellreferenzen verwenden `kimi-code/<modelId>`.
- Überschreibe bei Bedarf Preis- und Kontextmetadaten in `models.providers`.
- Wenn Moonshot andere Kontextlimits für ein Modell veröffentlicht, passe
  `contextWindow` entsprechend an.
- Verwende `https://api.moonshot.cn/v1`, wenn du den China-Endpunkt benötigst.