---
title: Mattermost
summary: "Mattermost 机器人设置与 OpenClaw 配置"
read_when:
  - 配置 Mattermost
  - 调试 Mattermost 路由
---

<div id="mattermost-plugin">
  # Mattermost（插件）
</div>

状态：通过插件支持（机器人令牌 + WebSocket 事件）。支持频道、群组和私信。
Mattermost 是一款可自托管的团队消息平台；产品详情和下载请参见官方网站
[mattermost.com](https://mattermost.com)。

<div id="plugin-required">
  ## 需要安装插件
</div>

Mattermost 以插件形式提供，不包含在核心安装中。

通过 CLI（npm registry）安装：

```bash
openclaw plugins install @openclaw/mattermost
```

本地检出（从 Git 仓库运行时）：

```bash
openclaw plugins install ./extensions/mattermost
```

如果你在配置/初始设置过程中选择了 Mattermost，并且检测到通过 Git 检出的代码，
OpenClaw 会自动填充本地安装路径。

详情：[插件](/zh/plugin)

<div id="quick-setup">
  ## 快速配置
</div>

1. 安装 Mattermost 插件。
2. 创建一个 Mattermost 机器人账户，并复制 **bot token**。
3. 复制 Mattermost 的 **base URL**（例如：`https://chat.example.com`）。
4. 配置 OpenClaw 并启动 Gateway。

最小配置：

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing"
    }
  }
}
```

<div id="environment-variables-default-account">
  ## 环境变量（默认账号）
</div>

如果你更倾向于使用环境变量，可以在 Gateway 主机上设置：

* `MATTERMOST_BOT_TOKEN=...`
* `MATTERMOST_URL=https://chat.example.com`

环境变量只适用于**默认**账号（`default`）。其他账号必须使用配置值。

<div id="chat-modes">
  ## 聊天模式
</div>

Mattermost 会自动回复私信（DM）。在频道中的行为由 `chatmode` 控制：

* `oncall`（默认）：仅在频道中被 @ 提及时回复。
* `onmessage`：对每一条频道消息都进行回复。
* `onchar`：当消息以触发前缀开头时回复。

配置示例：

```json5
{
  channels: {
    mattermost: {
      chatmode: "onchar",
      oncharPrefixes: [">", "!"]
    }
  }
}
```

Notes:

* `onchar` 仍然会响应显式的 @ 提及。
* 旧版配置中仍然会遵守 `channels.mattermost.requireMention`，但更推荐使用 `chatmode`。

<div id="access-control-dms">
  ## 访问控制（私信）
</div>

* 默认值：`channels.mattermost.dmPolicy = "pairing"`（未知发送者会收到配对码）。
* 通过以下命令进行批准：
  * `openclaw pairing list mattermost`
  * `openclaw pairing approve mattermost <CODE>`
* 公开私信：`channels.mattermost.dmPolicy="open"` 加上 `channels.mattermost.allowFrom=["*"]`（`open` 表示允许从任意用户无条件接收私信）。

<div id="channels-groups">
  ## 频道（群组）
</div>

* 默认：`channels.mattermost.groupPolicy = "allowlist"`（需 @ 提及，基于允许列表）。
* 使用 `channels.mattermost.groupAllowFrom` 配置允许列表发送方（用户 ID 或 `@username`）。
* 开放频道：`channels.mattermost.groupPolicy="open"`（需 @ 提及；`open` 表示允许任何用户不受限制地发送消息）。

<div id="targets-for-outbound-delivery">
  ## 出站发送目标
</div>

在使用 `openclaw message send` 或 cron/webhook 时，可以使用以下目标格式：

* `channel:<id>` 表示一个频道
* `user:<id>` 表示一条私信
* `@username` 表示一条私信（通过 Mattermost API 解析）

不带前缀的 ID 会被视为频道。

<div id="multi-account">
  ## 多账号
</div>

Mattermost 支持在 `channels.mattermost.accounts` 下配置多个账号：

```json5
{
  channels: {
    mattermost: {
      accounts: {
        default: { name: "Primary", botToken: "mm-token", baseUrl: "https://chat.example.com" },
        alerts: { name: "Alerts", botToken: "mm-token-2", baseUrl: "https://alerts.example.com" }
      }
    }
  }
}
```

<div id="troubleshooting">
  ## 故障排查
</div>

* 频道中没有回复：确认机器人已在频道中，并 @ 它（oncall），使用触发前缀（onchar），或设置 `chatmode: "onmessage"`。
* 认证错误：检查机器人令牌、基础 URL，以及账号是否已启用。
* 多账号问题：环境变量只适用于 `default` 账号。