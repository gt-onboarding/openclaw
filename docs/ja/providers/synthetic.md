---
title: Synthetic
summary: "OpenClaw で Synthetic の Anthropic 互換 API を使用する"
read_when:
  - Synthetic をモデルプロバイダーとして利用したい
  - Synthetic の API キーまたはベース URL を設定する必要がある
---

<div id="synthetic">
  # Synthetic
</div>

Synthetic は Anthropic 互換のエンドポイントを提供します。OpenClaw はこれを
`synthetic` プロバイダーとして登録し、Anthropic Messages API を使用します。

<div id="quick-setup">
  ## クイックセットアップ
</div>

1. `SYNTHETIC_API_KEY` を設定します（または以下のウィザードを実行します）。
2. オンボーディングを実行します：

```bash
openclaw onboard --auth-choice synthetic-api-key
```

デフォルトのモデルは次のとおりに設定されています：

```
synthetic/hf:MiniMaxAI/MiniMax-M2.1
```

<div id="config-example">
  ## 設定例
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

注意: OpenClaw の Anthropic クライアントはベース URL に自動的に `/v1` を付与するため、
`https://api.synthetic.new/anthropic`（`/anthropic/v1` ではない）を使用してください。
Synthetic がベース URL を変更した場合は、`models.providers.synthetic.baseUrl` をオーバーライドしてください。

<div id="model-catalog">
  ## モデルカタログ
</div>

以下のモデルはすべて、コスト（入力／出力／キャッシュ）が `0` です。

| Model ID | コンテキストウィンドウ | 最大トークン数 | Reasoning 対応 | 入力 |
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
  ## 注意事項
</div>

* モデル参照には `synthetic/<modelId>` を使用します。
* モデルの許可リスト（`agents.defaults.models`）を有効にする場合は、利用する予定のすべてのモデルを追加してください。
* プロバイダーに関するルールについては、[Model providers](/ja/concepts/model-providers) を参照してください。