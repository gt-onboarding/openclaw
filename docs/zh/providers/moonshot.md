---
title: Moonshot
summary: "配置 Moonshot K2 与 Kimi Code（分别作为提供方并使用独立密钥）"
read_when:
  - 你想要分别配置 Moonshot K2（Moonshot Open Platform）和 Kimi Code
  - 你需要理解各自独立的 endpoint、密钥和模型引用
  - 你想要获取任一提供方可直接复制粘贴的配置
---

<div id="moonshot-ai-kimi">
  # Moonshot AI（Kimi）
</div>

Moonshot 提供与 OpenAI 兼容端点的 Kimi API。配置该提供方并将默认模型设置为 `moonshot/kimi-k2.5`，或者使用 `kimi-code/kimi-for-coding` 来启用 Kimi Code。

当前 Kimi K2 模型 ID：

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

Kimi 代码示例：

```bash
openclaw onboard --auth-choice kimi-code-api-key
```

注意：Moonshot 和 Kimi Code 是彼此独立的提供方。密钥不可互用，端点不同，模型引用也不同（Moonshot 使用 `moonshot/...`，Kimi Code 使用 `kimi-code/...`）。


<div id="config-snippet-moonshot-api">
  ## 配置示例（Moonshot API）
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
  ## 说明
</div>

- Moonshot 模型引用使用 `moonshot/&lt;modelId&gt;`，Kimi Code 模型引用使用 `kimi-code/&lt;modelId&gt;`。
- 如有需要，在 `models.providers` 中覆盖定价和上下文元数据。
- 如果 Moonshot 为某个模型发布了不同的上下文限制，请相应调整 `contextWindow`。
- 如果你需要使用中国区的 endpoint，请使用 `https://api.moonshot.cn/v1`。