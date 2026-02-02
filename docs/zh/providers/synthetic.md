---
title: Synthetic
summary: "在 OpenClaw 中使用 Synthetic 的 Anthropic 兼容 API"
read_when:
  - 你希望使用 Synthetic 作为模型提供方
  - 你需要配置 Synthetic API 密钥或基础 URL
---

<div id="synthetic">
  # Synthetic
</div>

Synthetic 暴露了与 Anthropic 兼容的接口端点。OpenClaw 将其注册为
`synthetic` 提供方，并使用 Anthropic Messages API。

<div id="quick-setup">
  ## 快速开始
</div>

1. 设置 `SYNTHETIC_API_KEY`（或运行下面的向导）。
2. 运行初始化流程：

```bash
openclaw onboard --auth-choice synthetic-api-key
```

默认模型为：

```
synthetic/hf:MiniMaxAI/MiniMax-M2.1
```

<div id="config-example">
  ## 配置示例
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

注意：OpenClaw 的 Anthropic 客户端会在基础 URL 后追加 `/v1`，因此请使用
`https://api.synthetic.new/anthropic`（而不是 `/anthropic/v1`）。如果 Synthetic 更改了
其基础 URL，请覆盖 `models.providers.synthetic.baseUrl`。

<div id="model-catalog">
  ## 模型目录
</div>

下列所有模型的成本均为 `0`（输入/输出/缓存）。

| 模型 ID | 上下文窗口 | 最大 Token 数 | 推理能力 | 输入类型 |
| --- | --- | --- | --- | --- |
| `hf:MiniMaxAI/MiniMax-M2.1` | 192000 | 65536 | false | 文本 |
| `hf:moonshotai/Kimi-K2-Thinking` | 256000 | 8192 | true | 文本 |
| `hf:zai-org/GLM-4.7` | 198000 | 128000 | false | 文本 |
| `hf:deepseek-ai/DeepSeek-R1-0528` | 128000 | 8192 | false | 文本 |
| `hf:deepseek-ai/DeepSeek-V3-0324` | 128000 | 8192 | false | 文本 |
| `hf:deepseek-ai/DeepSeek-V3.1` | 128000 | 8192 | false | 文本 |
| `hf:deepseek-ai/DeepSeek-V3.1-Terminus` | 128000 | 8192 | false | 文本 |
| `hf:deepseek-ai/DeepSeek-V3.2` | 159000 | 8192 | false | 文本 |
| `hf:meta-llama/Llama-3.3-70B-Instruct` | 128000 | 8192 | false | 文本 |
| `hf:meta-llama/Llama-4-Maverick-17B-128E-Instruct-FP8` | 524000 | 8192 | false | 文本 |
| `hf:moonshotai/Kimi-K2-Instruct-0905` | 256000 | 8192 | false | 文本 |
| `hf:openai/gpt-oss-120b` | 128000 | 8192 | false | 文本 |
| `hf:Qwen/Qwen3-235B-A22B-Instruct-2507` | 256000 | 8192 | false | 文本 |
| `hf:Qwen/Qwen3-Coder-480B-A35B-Instruct` | 256000 | 8192 | false | 文本 |
| `hf:Qwen/Qwen3-VL-235B-A22B-Instruct` | 250000 | 8192 | false | 文本 + 图像 |
| `hf:zai-org/GLM-4.5` | 128000 | 128000 | false | 文本 |
| `hf:zai-org/GLM-4.6` | 198000 | 128000 | false | 文本 |
| `hf:deepseek-ai/DeepSeek-V3` | 128000 | 8192 | false | 文本 |
| `hf:Qwen/Qwen3-235B-A22B-Thinking-2507` | 256000 | 8192 | true | 文本 |

<div id="notes">
  ## 备注
</div>

* 模型引用采用 `synthetic/&lt;modelId&gt;`。
* 如果你启用了模型允许列表（`agents.defaults.models`），请将你计划使用的每个模型都添加进去。
* 有关提供方相关规则，请参见[模型提供方](/zh/concepts/model-providers)。