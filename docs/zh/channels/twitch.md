---
title: Twitch
summary: "Twitch 聊天机器人配置与设置"
read_when:
  - 为 OpenClaw 配置 Twitch 聊天集成
---

<div id="twitch-plugin">
  # Twitch（插件）
</div>

通过 IRC 连接接入 Twitch 聊天。OpenClaw 以 Twitch 用户（机器人账户）的身份连接，用于在频道中收发消息。

<div id="plugin-required">
  ## 所需插件
</div>

Twitch 作为插件发布，不包含在核心安装中。

通过 CLI 从 npm 注册表安装：

```bash
openclaw plugins install @openclaw/twitch
```

本地代码检出（从 git 仓库运行时）：

```bash
openclaw plugins install ./extensions/twitch
```

详细信息：[插件](/zh/plugin)

<div id="quick-setup-beginner">
  ## 快速设置（入门）
</div>

1. 为机器人创建一个专用的 Twitch 账号（或使用现有账号）。
2. 生成凭证： [Twitch Token Generator](https://twitchtokengenerator.com/)
   * 选择 **Bot Token**
   * 确认已选择 `chat:read` 和 `chat:write` 这两个 scope
   * 复制 **Client ID** 和 **Access Token**
3. 查找你的 Twitch 用户 ID： https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/
4. 配置 token：
   * 环境变量：`OPENCLAW_TWITCH_ACCESS_TOKEN=...`（仅用于默认账号）
   * 或配置项：`channels.twitch.accessToken`
   * 如果两者都设置，则以配置项为准（环境变量仅作为默认账号的回退）。
5. 启动 Gateway。

**⚠️ 重要：** 添加访问控制（`allowFrom` 或 `allowedRoles`），以防止未授权用户触发机器人。`requireMention` 默认为 `true`。

最小配置示例：

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",              // Bot's Twitch account
      accessToken: "oauth:abc123...",    // OAuth Access Token (or use OPENCLAW_TWITCH_ACCESS_TOKEN env var)
      clientId: "xyz789...",             // Client ID from Token Generator
      channel: "vevisk",                 // Which Twitch channel's chat to join (required)
      allowFrom: ["123456789"]           // （推荐）仅限您的 Twitch 用户 ID——从 https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/ 获取
    }
  }
}
```

<div id="what-it-is">
  ## 功能说明
</div>

* 一个由 Gateway 管理的 Twitch 频道。
* 确定性路由：回复始终发送回 Twitch。
* 每个账号映射到一个隔离的会话键 `agent:<agentId>:twitch:<accountName>`。
* `username` 是机器人的账号（用于身份认证），`channel` 是要加入的聊天室。

<div id="setup-detailed">
  ## 详细配置
</div>

<div id="generate-credentials">
  ### 生成凭据
</div>

使用 [Twitch Token Generator](https://twitchtokengenerator.com/)：

* 选择 **Bot Token**
* 确认已选择 `chat:read` 和 `chat:write` 这两个 scope
* 复制 **Client ID** 和 **Access Token**

无需手动注册应用程序。令牌将在数小时后过期。

<div id="configure-the-bot">
  ### 配置机器人
</div>

**环境变量（仅适用于默认账户）：**

```bash
OPENCLAW_TWITCH_ACCESS_TOKEN=oauth:abc123...
```

**或通过配置：**

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk"
    }
  }
}
```

如果同时设置了环境变量和配置文件，则配置文件优先生效。

<div id="access-control-recommended">
  ### 访问控制（推荐）
</div>

```json5
{
  channels: {
    twitch: {
      allowFrom: ["123456789"],       // (推荐) 仅限您的 Twitch 用户 ID
      allowedRoles: ["moderator"]     // Or restrict to roles
    }
  }
}
```

**可用角色：** `"moderator"`、`"owner"`、`"vip"`、`"subscriber"`、`"all"`。

**为什么使用用户 ID？** 用户名可以更改，可能被他人用来冒充你。用户 ID 是永久不变的。

查找你的 Twitch 用户 ID：https://www.streamweasels.com/tools/convert-twitch-username-%20to-user-id/（将你的 Twitch 用户名转换为 ID）

<div id="token-refresh-optional">
  ## 令牌刷新（可选）
</div>

通过 [Twitch Token Generator](https://twitchtokengenerator.com/) 获取的令牌无法自动刷新——过期后需要重新生成。

如需自动刷新令牌，请在 [Twitch Developer Console](https://dev.twitch.tv/console) 创建自己的 Twitch 应用，并在配置中添加：

```json5
{
  channels: {
    twitch: {
      clientSecret: "your_client_secret",
      refreshToken: "your_refresh_token"
    }
  }
}
```

该机器人会在令牌过期前自动刷新令牌，并将刷新操作写入日志。

<div id="multi-account-support">
  ## 多账号支持
</div>

使用 `channels.twitch.accounts` 为每个账号配置各自的 token。通用的配置模式参见 [`gateway/configuration`](/zh/gateway/configuration)。

示例（一个机器人账号同时在两个频道中使用）：

```json5
{
  channels: {
    twitch: {
      accounts: {
        channel1: {
          username: "openclaw",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "vevisk"
        },
        channel2: {
          username: "openclaw",
          accessToken: "oauth:def456...",
          clientId: "uvw012...",
          channel: "secondchannel"
        }
      }
    }
  }
}
```

**注意：** 每个账户都需要独立的令牌（每个频道对应一个令牌）。

<div id="access-control">
  ## 访问控制
</div>

<div id="role-based-restrictions">
  ### 基于角色的访问限制
</div>

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowedRoles: ["moderator", "vip"]
        }
      }
    }
  }
}
```

<div id="allowlist-by-user-id-most-secure">
  ### 基于用户 ID 的允许列表（安全性最高）
</div>

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowFrom: ["123456789", "987654321"]
        }
      }
    }
  }
}
```

<div id="combined-allowlist-roles">
  ### 允许列表与角色组合使用
</div>

`allowFrom` 中的用户将跳过角色检查：

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowFrom: ["123456789"],
          allowedRoles: ["moderator"]
        }
      }
    }
  }
}
```

<div id="disable-mention-requirement">
  ### 关闭 @ 提及要求
</div>

默认情况下，`requireMention` 为 `true`。若要关闭该要求并响应所有消息：

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          requireMention: false
        }
      }
    }
  }
}
```

<div id="troubleshooting">
  ## 故障排查
</div>

首先运行以下诊断命令：

```bash
openclaw doctor
openclaw channels status --probe
```

<div id="bot-doesnt-respond-to-messages">
  ### 机器人未响应消息
</div>

**检查访问控制：** 临时将 `allowedRoles` 设置为 `["all"]` 进行测试。

**检查机器人是否在频道中：** 机器人必须加入在 `channel` 中指定的频道。

<div id="token-issues">
  ### Token 问题
</div>

**“Failed to connect（连接失败）”或身份验证错误：**

* 确认 `accessToken` 是 OAuth 访问令牌的值（通常以 `oauth:` 前缀开头）
* 检查令牌具有 `chat:read` 和 `chat:write` 这两个 scope
* 如果使用 Token 刷新，确认已设置 `clientSecret` 和 `refreshToken`

<div id="token-refresh-not-working">
  ### 令牌刷新无效
</div>

**检查日志中的刷新事件：**

```
Using env token source for mybot
已为用户 123456 刷新访问令牌(14400 秒后过期)
```

如果你看到“token refresh disabled (no refresh token)”：

* 确保已提供 `clientSecret`
* 确保已提供 `refreshToken`

<div id="config">
  ## 配置
</div>

**账号配置：**

* `username` - 机器人用户名
* `accessToken` - 具有 `chat:read` 和 `chat:write` 权限的 OAuth 访问令牌
* `clientId` - Twitch Client ID（来自 Token Generator 或你的应用）
* `channel` - 要加入的频道（必填）
* `enabled` - 是否启用此账号（默认：`true`）
* `clientSecret` - 可选：用于自动刷新令牌
* `refreshToken` - 可选：用于自动刷新令牌
* `expiresIn` - 令牌过期时间（秒）
* `obtainmentTimestamp` - 令牌获取的时间戳
* `allowFrom` - 用户 ID 允许列表
* `allowedRoles` - 基于角色的访问控制（`"moderator" | "owner" | "vip" | "subscriber" | "all"`）
* `requireMention` - 是否要求被 @ 提及（默认：`true`）

**提供方选项：**

* `channels.twitch.enabled` - 启用/禁用频道在启动时加载
* `channels.twitch.username` - 机器人用户名（简化的单账号配置）
* `channels.twitch.accessToken` - OAuth 访问令牌（简化的单账号配置）
* `channels.twitch.clientId` - Twitch Client ID（简化的单账号配置）
* `channels.twitch.channel` - 要加入的频道（简化的单账号配置）
* `channels.twitch.accounts.<accountName>` - 多账号配置（包含以上所有账号字段）

完整示例：

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
      clientSecret: "secret123...",
      refreshToken: "refresh456...",
      allowFrom: ["123456789"],
      allowedRoles: ["moderator", "vip"],
      accounts: {
        default: {
          username: "mybot",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "your_channel",
          enabled: true,
          clientSecret: "secret123...",
          refreshToken: "refresh456...",
          expiresIn: 14400,
          obtainmentTimestamp: 1706092800000,
          allowFrom: ["123456789", "987654321"],
          allowedRoles: ["moderator"]
        }
      }
    }
  }
}
```

<div id="tool-actions">
  ## 工具操作
</div>

智能体可以调用 `twitch` 执行以下动作：

* `send` - 向频道发送一条消息

示例：

```json5
{
  "action": "twitch",
  "params": {
    "message": "Hello Twitch!",
    "to": "#mychannel"
  }
}
```

<div id="safety-ops">
  ## 安全与运维
</div>

* **将令牌当作密码对待** - 切勿将令牌提交到 git
* **为长时间运行的机器人启用自动令牌刷新**
* **使用基于用户 ID 的允许列表** 而不是用户名来做访问控制
* **监控日志** 中的令牌刷新事件和连接状态
* **尽量缩小令牌权限范围** - 只请求 `chat:read` 和 `chat:write`
* **如果卡住**：在确认没有其他进程占用该会话后重启 Gateway

<div id="limits">
  ## 限制
</div>

* 每条消息 **500 个字符**（在单词边界处自动分片）
* 在分片前会去除 Markdown
* 无额外速率限制（依赖 Twitch 内置的速率限制）