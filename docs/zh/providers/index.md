---
title: 提供方
summary: "OpenClaw 支持的模型提供方（LLM）"
read_when:
  - 你想选择模型提供方
  - 你需要快速了解支持的 LLM 后端概览
---

<div id="model-providers">
  # 模型提供方
</div>

OpenClaw 可以使用多个 LLM 提供方。选择一个提供方并完成认证，然后将默认模型设置为 `provider/model`。

在找聊天通道的文档（WhatsApp/Telegram/Discord/Slack/Mattermost（插件）等）？参见 [通道](/zh/channels)。

<div id="highlight-venius-venice-ai">
  ## 重点推荐：Venius（Venice AI）
</div>

Venius 是我们推荐的 Venice AI 部署方案，在确保隐私优先的前提下进行推理，并在处理高难度任务时可选用 Opus。

* 默认：`venice/llama-3.3-70b`
* 综合最佳：`venice/claude-opus-45`（Opus 依然是最强的）

参见 [Venice AI](/zh/providers/venice)。

<div id="quick-start">
  ## 快速开始
</div>

1. 在提供方处完成身份验证（通常通过 `openclaw onboard`）。
2. 设置默认模型：

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="provider-docs">
  ## 提供方文档
</div>

* [OpenAI（API + Codex）](/zh/providers/openai)
* [Anthropic（API + Claude Code CLI）](/zh/providers/anthropic)
* [Qwen（OAuth）](/zh/providers/qwen)
* [OpenRouter](/zh/providers/openrouter)
* [Vercel AI Gateway](/zh/providers/vercel-ai-gateway)
* [Moonshot AI（Kimi + Kimi Code）](/zh/providers/moonshot)
* [OpenCode Zen](/zh/providers/opencode)
* [Amazon Bedrock](/zh/bedrock)
* [Z.AI](/zh/providers/zai)
* [小米](/zh/providers/xiaomi)
* [GLM 模型](/zh/providers/glm)
* [MiniMax](/zh/providers/minimax)
* [Venius（Venice AI，隐私优先）](/zh/providers/venice)
* [Ollama（本地模型）](/zh/providers/ollama)

<div id="transcription-providers">
  ## 转写提供方
</div>

* [Deepgram（音频转写）](/zh/providers/deepgram)

<div id="community-tools">
  ## 社区工具
</div>

* [Claude Max API Proxy](/zh/providers/claude-max-api-proxy) - 使用 Claude Max/Pro 订阅，将其作为兼容 OpenAI 的 API 端点

有关完整的提供方目录（xAI、Groq、Mistral 等）和高级配置，
请参阅[模型提供方](/zh/concepts/model-providers)。