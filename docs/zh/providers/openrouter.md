---
title: OpenRouter
summary: "在 OpenClaw 中通过 OpenRouter 的统一 api 访问多种模型"
read_when:
  - 你想要使用单个 API 密钥访问多个 LLM 模型
  - 你想要在 OpenClaw 中通过 OpenRouter 运行模型
---

<div id="openrouter">
  # OpenRouter
</div>

OpenRouter 提供一个**统一的 API**，通过单一的 endpoint 和 API key 将请求路由到多个模型。它与 OpenAI 接口兼容，因此大多数 OpenAI SDK 只需切换 base URL 即可直接使用。

<div id="cli-setup">
  ## CLI 配置
</div>

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

<div id="config-snippet">
  ## 配置示例
</div>

```json5
{
  env: { OPENROUTER_API_KEY: "sk-or-..." },
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" }
    }
  }
}
```

<div id="notes">
  ## 注意事项
</div>

* 模型引用格式为 `openrouter/<provider>/<model>`。
* 更多模型和提供方选项，请参见 [/concepts/model-providers](/zh/concepts/model-providers)。
* OpenRouter 在内部使用包含你的 api 密钥的 Bearer 令牌。