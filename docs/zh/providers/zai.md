---
title: Zai
summary: "在 OpenClaw 中使用 Z.AI（GLM 模型）"
read_when:
  - 你想在 OpenClaw 中使用 Z.AI / GLM 模型
  - 你需要一个简单的 ZAI_API_KEY 设置
---

<div id="zai">
  # Z.AI
</div>

Z.AI 是面向 **GLM** 模型的 API 平台。它为 GLM 提供 REST API，并使用 API 密钥
进行身份验证。请在 Z.AI 控制台中创建你的 API 密钥。OpenClaw 使用 `zai` 提供方，
并通过 Z.AI 的 API 密钥进行访问。

<div id="cli-setup">
  ## CLI 配置
</div>

```bash
openclaw onboard --auth-choice zai-api-key
# 或者使用非交互模式
openclaw onboard --zai-api-key "$ZAI_API_KEY"
```

<div id="config-snippet">
  ## 配置示例
</div>

```json5
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-4.7" } } }
}
```

<div id="notes">
  ## 注意事项
</div>

* GLM 模型可通过 `zai/&lt;model&gt;` 使用（例如：`zai/glm-4.7`）。
* 模型系列概览请参见 [/providers/glm](/zh/providers/glm)。
* Z.AI 使用 Bearer 身份验证并配合你的 API 密钥。