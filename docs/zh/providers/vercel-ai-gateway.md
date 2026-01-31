---
title: "Vercel AI Gateway"
summary: "Vercel AI Gateway 配置（身份验证 + 模型选择）"
read_when:
  - 你想将 Vercel AI Gateway 与 OpenClaw 一起使用
  - 你需要配置 API 密钥环境变量，或通过 CLI 选择身份验证方式
---

<div id="vercel-ai-gateway">
  # Vercel AI Gateway
</div>

[Vercel AI Gateway](https://vercel.com/ai-gateway) 通过单个端点提供统一的 API，可访问数百个模型。

- 提供方：`vercel-ai-gateway`
- 鉴权：`AI_GATEWAY_API_KEY`
- API：兼容 Anthropic Messages

<div id="quick-start">
  ## 快速开始
</div>

1. 设置 API 密钥（建议：为 Gateway 持久化保存该密钥）：

```bash
openclaw onboard --auth-choice ai-gateway-api-key
```

2. 设置默认模型：

```json5
{
  agents: {
    defaults: {
      model: { primary: "vercel-ai-gateway/anthropic/claude-opus-4.5" }
    }
  }
}
```


<div id="non-interactive-example">
  ## 非交互式示例
</div>

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY"
```


<div id="environment-note">
  ## 环境注意事项
</div>

如果 Gateway 以守护进程（launchd/systemd）方式运行，确保该进程可以访问 `AI_GATEWAY_API_KEY`（例如将其写入 `~/.openclaw/.env`，或通过 `env.shellEnv` 提供）。