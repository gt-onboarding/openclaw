---
title: iMessage
summary: "通过 imsg（基于 stdio 的 JSON-RPC）实现 iMessage 支持、安装与 chat_id 路由配置"
read_when:
  - 设置 iMessage 支持
  - 调试 iMessage 收发
---

<div id="imessage-imsg">
  # iMessage (imsg)
</div>

状态：外部 CLI 集成。Gateway 会启动 `imsg rpc`（基于 stdio 的 JSON-RPC）。

<div id="quick-setup-beginner">
  ## 快速设置（入门）
</div>

1. 确保本机上的「信息」(Messages) 已登录账号。
2. 安装 `imsg`：
   * `brew install steipete/tap/imsg`
3. 使用 `channels.imessage.cliPath` 和 `channels.imessage.dbPath` 配置 OpenClaw。
4. 启动 Gateway，并在出现的任何 macOS 提示中授予权限（自动化 + 完全磁盘访问）。

最小配置：

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<你的用户名>/Library/Messages/chat.db"
    }
  }
}
```

<div id="what-it-is">
  ## 功能概述
</div>

* 由 macOS 上的 `imsg` 提供支持的 iMessage 通道。
* 确定性路由：回复始终发回 iMessage。
* 私聊共享该智能体的主会话；群聊则相互隔离（`agent:<agentId>:imessage:group:<chat_id>`）。
* 如果一个多参与者线程到达时标记为 `is_group=false`，你仍然可以通过 `channels.imessage.groups` 按 `chat_id` 将其隔离（参见下方 “Group-ish threads”）。

<div id="config-writes">
  ## 配置写入
</div>

默认情况下，允许 iMessage 写入由 `/config set|unset` 触发的配置更新（需要 `commands.config: true`）。

可通过以下方式禁用：

```json5
{
  channels: { imessage: { configWrites: false } }
}
```

<div id="requirements">
  ## 前置条件
</div>

* 已在 Messages 中登录的 macOS 系统。
* 为 OpenClaw 和 `imsg` 授予完全磁盘访问权限（用于访问 Messages 数据库）。
* 发送消息时所需的自动化权限。
* `channels.imessage.cliPath` 可以指向任意代理 stdin/stdout 的命令（例如，一个通过 SSH 连接到另一台 Mac 并运行 `imsg rpc` 的包装脚本）。

<div id="setup-fast-path">
  ## 快速设置（快速路径）
</div>

1. 确保本机的「信息」(Messages) 应用已登录。
2. 配置好 iMessage 并启动 Gateway。

<div id="dedicated-bot-macos-user-for-isolated-identity">
  ### 专用机器人 macOS 用户（用于隔离身份）
</div>

如果你希望机器人通过**单独的 iMessage 身份**发送消息（避免与你的个人 Messages 会话混在一起），请使用一个专用 Apple ID 加上一个专用的 macOS 用户。

1. 创建一个专用 Apple ID（示例：`my-cool-bot@icloud.com`）。
   * Apple 可能会要求手机号码用于验证 / 双重认证（2FA）。
2. 创建一个 macOS 用户（示例：`openclawhome`）并登录该用户。
3. 在该 macOS 用户下打开 Messages，并使用机器人 Apple ID 登录 iMessage。
4. 启用远程登录（系统设置 → 通用 → 共享 → 远程登录）。
5. 安装 `imsg`：
   * `brew install steipete/tap/imsg`
6. 配置 SSH，使 `ssh <bot-macos-user>@localhost true` 在无需输入密码的情况下可以正常执行。
7. 将 `channels.imessage.accounts.bot.cliPath` 指向一个通过 SSH 以机器人用户身份运行 `imsg` 的包装脚本。

首次运行说明：在*机器人 macOS 用户*中，收发消息可能需要 GUI 授权（自动化 + 完全磁盘访问）。如果 `imsg rpc` 看起来卡住或直接退出，请登录该用户（可以使用屏幕共享），运行一次 `imsg chats --limit 1` / `imsg send ...`，在弹窗中完成授权，然后重试。

包装脚本示例（记得先 `chmod +x`）。将 `<bot-macos-user>` 替换为你实际的 macOS 用户名：

```bash
#!/usr/bin/env bash
set -euo pipefail

# 首先运行一次交互式 SSH 以接受主机密钥:
#   ssh <bot-macos-user>@localhost true
exec /usr/bin/ssh -o BatchMode=yes -o ConnectTimeout=5 -T <bot-macos-user>@localhost \
  "/usr/local/bin/imsg" "$@"
```

配置示例：

```json5
{
  channels: {
    imessage: {
      enabled: true,
      accounts: {
        bot: {
          name: "Bot",
          enabled: true,
          cliPath: "/path/to/imsg-bot",
          dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db"
        }
      }
    }
  }
}
```

对于单账户场景，请使用扁平配置项（`channels.imessage.cliPath`、`channels.imessage.dbPath`），而不要使用 `accounts` 映射表。

<div id="remotessh-variant-optional">
  ### 远程 / SSH 方案（可选）
</div>

如果你想在另一台 Mac 上使用 iMessage，将 `channels.imessage.cliPath` 设置为一个包装脚本，通过 SSH 在远程 macOS 主机上运行 `imsg`。OpenClaw 只需要使用标准输入输出（stdio）。

包装脚本示例：

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

**远程附件：**当 `cliPath` 通过 SSH 指向远程主机时，Messages 数据库中的附件路径会引用远程机器上的文件。通过设置 `channels.imessage.remoteHost`，OpenClaw 可以自动通过 SCP 拉取这些文件：

```json5
{
  channels: {
    imessage: {
      cliPath: "~/imsg-ssh",                     // 到远程 Mac 的 SSH 包装器
      remoteHost: "user@gateway-host",           // for SCP file transfer
      includeAttachments: true
    }
  }
}
```

如果未设置 `remoteHost`，OpenClaw 会尝试通过解析包装脚本中的 SSH 命令来自动检测该值。为提高可靠性，建议显式配置。

<div id="remote-mac-via-tailscale-example">
  #### 通过 Tailscale 连接远程 Mac（示例）
</div>

如果 Gateway 运行在 Linux 主机或虚拟机上，而 iMessage 必须运行在 Mac 上，可以使用 Tailscale 作为最简单的桥接方式：Gateway 通过 tailnet 与 Mac 通信，通过 SSH 运行 `imsg`，并通过 SCP 将附件传回。

架构：

```
┌──────────────────────────────┐          SSH (imsg rpc)          ┌──────────────────────────┐
│ Gateway 主机 (Linux/VM)      │──────────────────────────────────▶│ Mac with Messages + imsg │
│ - openclaw gateway           │          SCP (附件)               │ - Messages 已登录        │
│ - channels.imessage.cliPath  │◀──────────────────────────────────│ - 已启用远程登录         │
└──────────────────────────────┘                                   └──────────────────────────┘
              ▲
              │ Tailscale tailnet (主机名或 100.x.y.z)
              ▼
        user@gateway-host
```

具体配置示例（Tailscale 主机名）：

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db"
    }
  }
}
```

示例封装脚本（`~/.openclaw/scripts/imsg-ssh`）：

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

注意：

* 确保这台 Mac 已登录 Messages，并已启用“远程登录”（Remote Login）。
* 使用 SSH 密钥，这样 `ssh bot@mac-mini.tailnet-1234.ts.net` 可以在无交互提示的情况下直接连接。
* `remoteHost` 应与 SSH 目标一致，以便 SCP 可以拉取附件。

多账号支持：使用 `channels.imessage.accounts`，为每个账号提供单独配置，以及可选的 `name`。参见 [`gateway/configuration`](/zh/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) 了解通用模式。不要提交 `~/.openclaw/openclaw.json`（其中通常包含 token）。

<div id="access-control-dms-groups">
  ## 访问控制（私信 + 群组）
</div>

私信（DMs）：

* 默认：`channels.imessage.dmPolicy = "pairing"`。
* 未知发送方会收到一个配对码；在批准之前，其消息会被忽略（配对码 1 小时后过期）。
* 通过以下方式批准：
  * `openclaw pairing list imessage`
  * `openclaw pairing approve imessage <CODE>`
* 配对是 iMessage 私信的默认令牌交换机制。详情见：[Pairing](/zh/start/pairing)

群组：

* `channels.imessage.groupPolicy = open | allowlist | disabled`。（其中 open 表示允许来自任意用户的不受限制的消息/触发）
* 当 `allowlist` 已设置时，`channels.imessage.groupAllowFrom` 控制谁可以在群组中触发。
* 提及门控使用 `agents.list[].groupChat.mentionPatterns`（或 `messages.groupChat.mentionPatterns`），因为 iMessage 没有原生的提及元数据。
* 多智能体覆写：在 `agents.list[].groupChat.mentionPatterns` 上为每个智能体分别设置匹配模式。

<div id="how-it-works-behavior">
  ## 工作原理（行为）
</div>

* `imsg` 会以流式方式输出消息事件；Gateway 会将其规范化为共享的通道封装结构。
* 回复始终会路由回同一个会话 id 或 handle。

<div id="group-ish-threads-is_groupfalse">
  ## 类群聊线程（`is_group=false`）
</div>

某些 iMessage 线程可能包含多个参与者，但由于信息 App 存储聊天标识符的方式不同，仍然会以 `is_group=false` 的形式出现。

如果你在 `channels.imessage.groups` 下显式配置了 `chat_id`，OpenClaw 会将该线程视为“群组”，用于：

* 会话隔离（单独的 `agent:<agentId>:imessage:group:<chat_id>` 会话键名）
* 群组允许列表 / 提及（@）门控行为

示例：

```json5
{
  channels: {
    imessage: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "42": { "requireMention": false }
      }
    }
  }
}
```

当你希望针对某个特定会话线程使用一个独立的人格/模型时，这会非常有用（参见 [Multi-agent routing](/zh/concepts/multi-agent)）。如需文件系统隔离，请参见 [Sandboxing](/zh/gateway/sandboxing)。

<div id="media-limits">
  ## 媒体及上限
</div>

* 可选接收附件，可通过 `channels.imessage.includeAttachments` 配置。
* 媒体大小上限可通过 `channels.imessage.mediaMaxMb` 配置。

<div id="limits">
  ## 限制
</div>

* 发出的文本会根据 `channels.imessage.textChunkLimit` 进行分块（默认 4000）。
* 可选的按换行分块：将 `channels.imessage.chunkMode` 设置为 `"newline"`，以在长度分块前按空行（段落边界）拆分。
* 媒体上传大小受 `channels.imessage.mediaMaxMb` 限制（默认 16）。

<div id="addressing-delivery-targets">
  ## 寻址 / 投递目标
</div>

稳定路由时优先使用 `chat_id`：

* `chat_id:123`（首选）
* `chat_guid:...`
* `chat_identifier:...`
* 直接 handle 标识：`imessage:+1555` / `sms:+1555` / `user@example.com`

列出聊天会话：

```
imsg chats --limit 20
```

<div id="configuration-reference-imessage">
  ## 配置参考（iMessage）
</div>

完整配置： [Configuration](/zh/gateway/configuration)

提供方选项：

* `channels.imessage.enabled`: 启用/禁用 Channel 启动。
* `channels.imessage.cliPath`: `imsg` 的路径。
* `channels.imessage.dbPath`: Messages 数据库路径。
* `channels.imessage.remoteHost`: 当 `cliPath` 指向远程 Mac（例如 `user@gateway-host`）时，用于通过 SCP 传输附件的 SSH 主机。如果未设置，则会从 SSH 包装脚本中自动检测。
* `channels.imessage.service`: `imessage | sms | auto`。
* `channels.imessage.region`: SMS 区域。
* `channels.imessage.dmPolicy`: `pairing | allowlist | open | disabled`（默认：`pairing`）。
* `channels.imessage.allowFrom`: 私信允许列表（handles、电子邮件、E.164 号码或 `chat_id:*`）。`open` 模式要求为 `"*"`。iMessage 没有用户名；请使用 handle 或聊天目标。
* `channels.imessage.groupPolicy`: `open | allowlist | disabled`（默认：`allowlist`）。
* `channels.imessage.groupAllowFrom`: 群组发送者允许列表。
* `channels.imessage.historyLimit` / `channels.imessage.accounts.*.historyLimit`: 作为上下文包含的群组消息最大数量（0 表示禁用）。
* `channels.imessage.dmHistoryLimit`: 私信历史上限，以用户轮次计。按用户进行覆盖配置：`channels.imessage.dms["<handle>"].historyLimit`。
* `channels.imessage.groups`: 每个群组的默认值和允许列表（使用 `"*"` 作为全局默认）。
* `channels.imessage.includeAttachments`: 将附件纳入上下文。
* `channels.imessage.mediaMaxMb`: 入站/出站媒体大小上限（MB）。
* `channels.imessage.textChunkLimit`: 出站分片大小（字符数）。
* `channels.imessage.chunkMode`: `length`（默认）或 `newline`，在长度分片前按空行（段落边界）切分。

相关全局选项：

* `agents.list[].groupChat.mentionPatterns`（或 `messages.groupChat.mentionPatterns`）。
* `messages.responsePrefix`。