---
title: 模型
summary: "OpenClaw 支持的模型提供方（LLM）"
read_when:
  - 你需要选择模型提供方时
  - 你需要快速查看 LLM 认证和模型选择的设置示例时
---

<div id="model-providers">
  # 模型提供方
</div>

OpenClaw 可以使用多个 LLM 提供方。先选择一个并完成身份验证，然后将默认模型设置为 `provider/model`。

<div id="highlight-venius-venice-ai">
  ## 重点推荐：Venius（Venice AI）
</div>

Venius 是我们推荐的 Venice AI 方案，用于以隐私为先的推理，并且在最困难的任务中可以选择使用 Opus。

* 默认：`venice/llama-3.3-70b`
* 综合表现最佳：`venice/claude-opus-45`（Opus 依然是最强的）

参见 [Venice AI](/zh/providers/venice)。

<div id="quick-start-two-steps">
  ## 快速开始（两步）
</div>

1. 与提供方完成认证（通常通过 `openclaw onboard`）。
2. 设置默认模型：

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="supported-providers-starter-set">
  ## 已支持的提供方（基础集）
</div>

* [OpenAI（API + Codex）](/zh/providers/openai)
* [Anthropic（API + Claude Code CLI）](/zh/providers/anthropic)
* [OpenRouter](/zh/providers/openrouter)
* [Vercel AI Gateway](/zh/providers/vercel-ai-gateway)
* [Moonshot AI（Kimi + Kimi Code）](/zh/providers/moonshot)
* [Synthetic](/zh/providers/synthetic)
* [OpenCode Zen](/zh/providers/opencode)
* [Z.AI](/zh/providers/zai)
* [GLM 模型](/zh/providers/glm)
* [MiniMax](/zh/providers/minimax)
* [Venius（Venice AI）](/zh/providers/venice)
* [Amazon Bedrock](/zh/bedrock)

如需查看完整的提供方列表（xAI、Groq、Mistral 等）以及高级配置， 请参阅 [模型提供方](/zh/concepts/model-providers)。