---
title: Moonshot
summary: "Moonshot K2 と Kimi Code の設定（プロバイダーとキーを分離）"
read_when:
  - Moonshot K2（Moonshot Open Platform）または Kimi Code のセットアップを行いたい
  - 別々のエンドポイント、キー、モデル参照について理解する必要がある
  - いずれのプロバイダー向けにも使えるコピペ用の設定が欲しい
---

<div id="moonshot-ai-kimi">
  # Moonshot AI (Kimi)
</div>

Moonshot は OpenAI 互換エンドポイントを備えた Kimi API を提供しています。プロバイダーを
設定し、デフォルトモデルを `moonshot/kimi-k2.5` に設定するか、
`kimi-code/kimi-for-coding` を指定して Kimi Code を使用します。

現在の Kimi K2 モデル ID は次のとおりです:

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

Kimiコード：

```bash
openclaw onboard --auth-choice kimi-code-api-key
```

注記: Moonshot と Kimi Code は別個のプロバイダーです。キーは相互に使い回しできず、エンドポイントやモデル参照も異なります (Moonshot は `moonshot/...` を使用し、Kimi Code は `kimi-code/...` を使用します)。


<div id="config-snippet-moonshot-api">
  ## 設定例（Moonshot API）
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
  ## 注意事項
</div>

- Moonshot のモデル参照には `moonshot/<modelId>` を使用します。Kimi Code のモデル参照には `kimi-code/<modelId>` を使用します。
- 必要に応じて、`models.providers` 内の料金およびコンテキスト関連のメタデータを上書きしてください。
- Moonshot があるモデルに対して異なるコンテキスト上限を公開した場合は、
  それに合わせて `contextWindow` を調整してください。
- 中国向けエンドポイントが必要な場合は `https://api.moonshot.cn/v1` を使用してください。