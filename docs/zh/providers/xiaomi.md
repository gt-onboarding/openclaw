---
title: 小米
summary: "在 OpenClaw 中使用 Xiaomi MiMo（mimo-v2-flash）"
read_when:
  - 你想在 OpenClaw 中使用 Xiaomi MiMo 模型
  - 你需要完成 XIAOMI_API_KEY 的设置
---

<div id="xiaomi-mimo">
  # 小米 MiMo
</div>

小米 MiMo 是面向 **MiMo** 模型的 API 平台。它提供与 OpenAI 和 Anthropic 格式兼容的 REST API，并使用 API 密钥进行认证。你可以在 [Xiaomi MiMo 控制台](https://platform.xiaomimimo.com/#/console/api-keys) 中创建你的 API 密钥。OpenClaw 使用 `xiaomi` 提供方，并搭配 Xiaomi MiMo 的 API 密钥进行访问。

<div id="model-overview">
  ## 模型概览
</div>

* **mimo-v2-flash**：262144 个 token 的上下文窗口，兼容 Anthropic Messages API。
* 基本 URL：`https://api.xiaomimimo.com/anthropic`
* 授权：`Bearer $XIAOMI_API_KEY`

<div id="cli-setup">
  ## CLI 配置
</div>

```bash
openclaw onboard --auth-choice xiaomi-api-key
# 或以非交互方式
openclaw onboard --auth-choice xiaomi-api-key --xiaomi-api-key "$XIAOMI_API_KEY"
```

<div id="config-snippet">
  ## 配置示例
</div>

```json5
{
  env: { XIAOMI_API_KEY: "your-key" },
  agents: { defaults: { model: { primary: "xiaomi/mimo-v2-flash" } } },
  models: {
    mode: "merge",
    providers: {
      xiaomi: {
        baseUrl: "https://api.xiaomimimo.com/anthropic",
        api: "anthropic-messages",
        apiKey: "XIAOMI_API_KEY",
        models: [
          {
            id: "mimo-v2-flash",
            name: "Xiaomi MiMo V2 Flash",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 262144,
            maxTokens: 8192
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

* 模型标识：`xiaomi/mimo-v2-flash`。
* 当设置了 `XIAOMI_API_KEY`（或存在认证配置文件）时，会自动注入提供方。
* 有关提供方规则，请参阅 [/concepts/model-providers](/zh/concepts/model-providers)。