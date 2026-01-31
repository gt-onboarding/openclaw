---
title: Openclaw
summary: "关于将 OpenClaw 作为个人助理运行的端到端指南（含安全注意事项）"
read_when:
  - 为新的助手实例执行初始配置时
  - 需要审查安全性和权限影响时
---

<div id="building-a-personal-assistant-with-openclaw">
  # 使用 OpenClaw 构建个人助理
</div>

OpenClaw 是一个面向 **Pi** 智能体的 WhatsApp、Telegram、Discord 和 iMessage Gateway。插件可以添加对 Mattermost 的支持。本指南介绍“个人助理”配置：使用一个专用的 WhatsApp 号码，把它配置成你始终在线的智能体。

<div id="safety-first">
  ## ⚠️ 安全第一
</div>

你即将让一个智能体具备以下能力：

* 在你的机器上运行命令（取决于你的 Pi 工具设置）
* 在你的工作区中读/写文件
* 通过 WhatsApp/Telegram/Discord/Mattermost（插件）向外发送消息

请从保守配置开始：

* 一定要设置 `channels.whatsapp.allowFrom`（绝不要在你的个人 Mac 上以向全世界开放的方式运行）。
* 为助手使用一个专用的 WhatsApp 号码。
* 心跳现在默认每 30 分钟一次。在你尚未完全信任当前配置之前，通过设置 `agents.defaults.heartbeat.every: "0m"` 将其禁用。

<div id="prerequisites">
  ## 先决条件
</div>

* Node **22+**
* 已添加到 PATH 的 OpenClaw（建议全局安装）
* 为助手准备的第二个电话号码（SIM/eSIM/预付费）

```bash
npm install -g openclaw@latest
# 或者：pnpm add -g openclaw@latest
```

从源代码构建（开发环境）：

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # 首次运行时自动安装 UI 依赖项
pnpm build
pnpm link --global
```

<div id="the-two-phone-setup-recommended">
  ## 双手机方案（推荐）
</div>

你想要的是这样的：

```
Your Phone (personal)          Second Phone (assistant)
┌─────────────────┐           ┌─────────────────┐
│  Your WhatsApp  │  ──────▶  │  Assistant WA   │
│  +1-555-YOU     │  message  │  +1-555-ASSIST  │
└─────────────────┘           └────────┬────────┘
                                       │ linked via QR
                                       ▼
                              ┌─────────────────┐
                              │  Your Mac       │
                              │  (openclaw)      │
                              │    Pi agent     │
                              └─────────────────┘
```

如果你把自己的个人 WhatsApp 账号直接绑定到 OpenClaw，那么发给你的每一条消息都会变成“智能体输入（agent input）”。而这几乎从来都不是你真正想要的效果。

<div id="5-minute-quick-start">
  ## 5 分钟快速上手
</div>

1. 配对 WhatsApp Web（会显示二维码；用助手用的手机扫码）：

```bash
openclaw channels login
```

2. 启动 Gateway（并保持运行）：

```bash
openclaw gateway --port 18789
```

3. 在 `~/.openclaw/openclaw.json` 中写入一个最小配置：

```json5
{
  channels: { whatsapp: { allowFrom: ["+15555550123"] } }
}
```

现在，用你在允许列表中的手机号向助手的号码发送消息。

引导流程完成后，我们会自动打开已带上你 Gateway 令牌的仪表盘，并输出带令牌的链接。之后要重新打开：`openclaw dashboard`。

<div id="give-the-agent-a-workspace-agents">
  ## 为智能体提供一个工作区（AGENTS）
</div>

OpenClaw 会从其工作区目录中读取操作指令和「记忆」。

默认情况下，OpenClaw 使用 `~/.openclaw/workspace` 作为智能体工作区，并会在安装/首次运行智能体时自动创建该目录（以及初始的 `AGENTS.md`、`SOUL.md`、`TOOLS.md`、`IDENTITY.md`、`USER.md`）。只有在工作区全新创建时才会生成 `BOOTSTRAP.md`（删除后不应再次出现）。

提示：把这个文件夹当作 OpenClaw 的「记忆」，并将其初始化为一个 Git 仓库（最好是私有的），以便备份你的 `AGENTS.md` 和记忆文件。如果已安装 Git，全新创建的工作区会自动初始化为仓库。

```bash
openclaw setup
```

完整工作区布局和备份指南：[Agent 工作区](/zh/concepts/agent-workspace)
记忆工作流：[Memory](/zh/concepts/memory)

可选：通过 `agents.defaults.workspace` 选择不同的工作区（支持 `~`）。

```json5
{
  agent: {
    workspace: "~/.openclaw/workspace"
  }
}
```

如果你已经在代码仓库中自带自己的工作区文件，就可以完全禁用引导文件的创建：

```json5
{
  agent: {
    skipBootstrap: true
  }
}
```

<div id="the-config-that-turns-it-into-an-assistant">
  ## 让它真正成为“助理”的配置
</div>

OpenClaw 默认已经是一个不错的助理配置，但你通常会想要调整：

* `SOUL.md` 中的人设/指令
* 思考相关的默认配置（如有需要）
* 心跳（在你开始信任它之后）

示例：

```json5
{
  logging: { level: "info" },
  agent: {
    model: "anthropic/claude-opus-4-5",
    workspace: "~/.openclaw/workspace",
    thinkingDefault: "high",
    timeoutSeconds: 1800,
    // 初始设为 0;后续再启用。
    heartbeat: { every: "0m" }
  },
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true }
      }
    }
  },
  routing: {
    groupChat: {
      mentionPatterns: ["@openclaw", "openclaw"]
    }
  },
  session: {
    scope: "per-sender",
    resetTriggers: ["/new", "/reset"],
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 10080
    }
  }
}
```

<div id="sessions-and-memory">
  ## 会话与记忆
</div>

* 会话文件：`~/.openclaw/agents/<agentId>/sessions/{{SessionId}}.jsonl`
* 会话元数据（token 使用量、最近一次路由等）：`~/.openclaw/agents/<agentId>/sessions/sessions.json`（旧路径：`~/.openclaw/sessions/sessions.json`）
* `/new` 或 `/reset` 会为当前对话启动一个全新的会话（可通过 `resetTriggers` 配置）。如果单独发送该指令，智能体会回复一条简短问候，以确认已重置。
* `/compact [instructions]` 会压缩会话上下文，并报告剩余的上下文预算。

<div id="heartbeats-proactive-mode">
  ## 心跳（主动模式）
</div>

默认情况下，OpenClaw 每 30 分钟运行一次心跳，使用的提示为：
`Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`
将 `agents.defaults.heartbeat.every: "0m"` 设为 `"0m"` 可禁用心跳。

* 如果存在 `HEARTBEAT.md`，但实际上是空的（只有空行和像 `# Heading` 这样的 markdown 标题），OpenClaw 会跳过这次心跳，以节省 api 调用。
* 如果文件缺失，心跳依然会运行，由模型自行决定如何处理。
* 如果智能体回复 `HEARTBEAT_OK`（可以带少量填充文本；参见 `agents.defaults.heartbeat.ackMaxChars`），OpenClaw 会抑制该次心跳消息的对外发送。
* 心跳会执行完整的一次智能体轮次——间隔越短，消耗的 token 越多。

```json5
{
  agent: {
    heartbeat: { every: "30m" }
  }
}
```

<div id="media-in-and-out">
  ## 媒体输入与输出
</div>

传入附件（图片/音频/文档）可以通过模板传递给你的命令：

* `{{MediaPath}}`（本地临时文件路径）
* `{{MediaUrl}}`（伪 URL）
* `{{Transcript}}`（如果启用了音频转录）

智能体发送的出站附件：让 `MEDIA:<path-or-url>` 单独占一行（中间不要有空格）。示例：

```
这是截图。
MEDIA:/tmp/screenshot.png
```

OpenClaw 会提取这些内容，并将其作为媒体内容连同文本一起发送。

<div id="operations-checklist">
  ## 运维检查清单
</div>

```bash
openclaw status          # 本地状态（凭证、会话、排队事件）
openclaw status --all    # 完整诊断（只读、可粘贴）
openclaw status --deep   # 添加 Gateway 健康探测（Telegram + Discord）
openclaw health --json   # Gateway 健康快照（WS）
```

日志位于 `/tmp/openclaw/` 目录下（默认文件名格式：`openclaw-YYYY-MM-DD.log`）。

<div id="next-steps">
  ## 后续步骤
</div>

* WebChat：[WebChat](/zh/web/webchat)
* Gateway 运维：[Gateway 运行手册](/zh/gateway)
* Cron 与唤醒任务：[Cron 任务](/zh/automation/cron-jobs)
* macOS 菜单栏助手：[OpenClaw macOS 应用](/zh/platforms/macos)
* iOS 节点应用：[iOS 应用](/zh/platforms/ios)
* Android 节点应用：[Android 应用](/zh/platforms/android)
* Windows 支持情况：[Windows（WSL2）](/zh/platforms/windows)
* Linux 支持情况：[Linux 应用](/zh/platforms/linux)
* 安全：[Security](/zh/gateway/security)