---
title: Anthropic
summary: "在 OpenClaw 中通过 API 密钥或 setup-token 使用 Anthropic Claude"
read_when:
  - 你希望在 OpenClaw 中使用 Anthropic 模型
  - 你希望使用 setup-token 而不是 API 密钥
---

<div id="anthropic-claude">
  # Anthropic (Claude)
</div>

Anthropic 构建了 **Claude** 模型系列，并提供基于 API 的访问。
在 OpenClaw 中，你可以使用 API 密钥（API key）或 **setup-token** 进行身份验证。

<div id="option-a-anthropic-api-key">
  ## 选项 A：Anthropic API 密钥
</div>

**最适合：** 标准 API 访问和按用量计费。
在 Anthropic Console 中创建 API 密钥。

<div id="cli-setup">
  ### CLI 配置
</div>

```bash
openclaw onboard
# 选择:Anthropic API 密钥

# 或者使用非交互式方式
openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
```

<div id="config-snippet">
  ### 配置示例
</div>

```json5
{
  env: { ANTHROPIC_API_KEY: "sk-ant-..." },
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="prompt-caching-anthropic-api">
  ## 提示缓存（Anthropic API）
</div>

除非你显式配置，否则 OpenClaw **不会** 覆盖 Anthropic 的默认缓存 TTL。
这仅适用于 **API 调用**；基于订阅的认证不会遵循 TTL 设置。

要为每个模型单独设置 TTL，请在模型的 `params` 中使用 `cacheControlTtl`：

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": {
          params: { cacheControlTtl: "5m" } // 或 "1h"
        }
      }
    }
  }
}
```

OpenClaw 为 Anthropic API 请求内置了 `extended-cache-ttl-2025-04-11` 测试版标记；如果你覆盖了提供方的请求头（参见 [/gateway/configuration](/zh/gateway/configuration)），请务必保留该标记。

<div id="option-b-claude-setup-token">
  ## 选项 B：Claude setup-token
</div>

**最适用于：使用你现有的 Claude 订阅。**

<div id="where-to-get-a-setup-token">
  ### 在哪里获取 setup-token
</div>

setup-token 是通过 **Claude Code CLI** 创建的，而不是通过 Anthropic 控制台创建。你可以在**任意一台机器**上运行它：

```bash
claude setup-token
```

将该 token 粘贴到 OpenClaw 中（向导：**Anthropic token（粘贴 setup-token）**），或者在 Gateway 主机上运行：

```bash
openclaw models auth setup-token --provider anthropic
```

如果你是在其他机器上生成的令牌，请在此粘贴：

```bash
openclaw models auth paste-token --provider anthropic
```

<div id="cli-setup">
  ### CLI 配置
</div>

```bash
# 在初始配置期间粘贴 setup-token
openclaw onboard --auth-choice setup-token
```

<div id="config-snippet">
  ### 配置示例
</div>

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="notes">
  ## 注意事项
</div>

* 使用 `claude setup-token` 生成 setup-token 并粘贴，或者在 Gateway 主机上运行 `openclaw models auth setup-token`。
* 如果你在 Claude 订阅中看到 “OAuth token refresh failed …” 错误，请使用 setup-token 重新进行认证。参见 [/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription](/zh/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription)。
* 身份验证详情和复用规则见 [/concepts/oauth](/zh/concepts/oauth)。

<div id="troubleshooting">
  ## 故障排查
</div>

**401 错误 / token 突然失效**

* Claude 订阅认证可能已过期或被撤销。重新运行 `claude setup-token`，
  然后将生成的 token 粘贴到 **gateway 主机** 上。
* 如果 Claude CLI 是在另一台机器上登录的，请在 gateway 主机上运行
  `openclaw models auth paste-token --provider anthropic`。

**No API key found for provider &quot;anthropic&quot;**

* 认证是**按智能体**隔离的。新的智能体不会继承主智能体的密钥。
* 为该智能体重新运行引导流程，或在 gateway 主机上粘贴 setup-token / API key，
  然后使用 `openclaw models status` 进行验证。

**No credentials found for profile `anthropic:default`**

* 运行 `openclaw models status` 查看当前激活的是哪个认证配置档（profile）。
* 重新运行引导流程，或为该配置档粘贴一个 setup-token / API key。

**No available auth profile (all in cooldown/unavailable)**

* 检查 `openclaw models status --json` 输出中的 `auth.unusableProfiles`。
* 新增一个 Anthropic 配置档，或者等待冷却时间结束。

更多内容：[/gateway/troubleshooting](/zh/gateway/troubleshooting) 和 [/help/faq](/zh/help/faq)。