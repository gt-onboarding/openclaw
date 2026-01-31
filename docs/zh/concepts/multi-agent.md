---
summary: "多智能体路由：隔离的智能体、渠道账户与绑定关系"
title: 多智能体路由
read_when: "当你需要在单个 Gateway 进程中运行多个相互隔离的智能体（工作区 + 身份认证）时"
status: active
---

<div id="multi-agent-routing">
  # 多智能体路由
</div>

目标：在同一个运行中的 Gateway 中，支持多个*隔离*的智能体（各自独立的工作区 + `agentDir` + 会话），以及多个渠道账号（例如两个 WhatsApp 账号）。传入消息通过绑定被路由到对应的智能体。

<div id="what-is-one-agent">
  ## 什么是“单个智能体”？
</div>

一个**智能体**是一个拥有完整作用域的“大脑”，它具备自己独立的：

* **工作区**（文件、AGENTS.md/SOUL.md/USER.md、本地笔记、人格规则）。
* **状态目录**（`agentDir`），用于存放认证档案（auth profiles）、模型注册信息以及该智能体的专属配置。
* **会话存储**（聊天历史 + 路由状态），位于 `~/.openclaw/agents/<agentId>/sessions`。

认证档案（auth profiles）是**按智能体分别维护**的。每个智能体只会从自己的目录中读取：

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

主 Agent 代理的凭据**不会**自动共享。切勿在多个智能体之间共用 `agentDir`
（否则会导致认证/会话冲突）。如果你想共享凭据，请将 `auth-profiles.json`
复制到另一个智能体的 `agentDir` 中。

技能是按智能体划分的，位于各自工作区中的 `skills/` 文件夹；共享技能
可以从 `~/.openclaw/skills` 获取。参见 [技能：按智能体 vs 共享](/zh/tools/skills#per-agent-vs-shared-skills)。

Gateway 可以并行托管**一个智能体**（默认）或**多个智能体**。

**工作区说明：** 每个智能体的工作区是**默认当前工作目录（cwd）**，而不是严格意义上的
沙箱。相对路径会在工作区内解析，但如果未启用沙箱，绝对路径
可以访问宿主机上的其他位置。参见
[沙箱](/zh/gateway/sandboxing)。

<div id="paths-quick-map">
  ## 路径速览
</div>

* 配置：`~/.openclaw/openclaw.json`（或 `OPENCLAW_CONFIG_PATH`）
* 状态目录：`~/.openclaw`（或 `OPENCLAW_STATE_DIR`）
* 工作区：`~/.openclaw/workspace`（或 `~/.openclaw/workspace-<agentId>`）
* Agent 目录：`~/.openclaw/agents/<agentId>/agent`（或 `agents.list[].agentDir`）
* 会话：`~/.openclaw/agents/<agentId>/sessions`

<div id="single-agent-mode-default">
  ### 单智能体模式（默认）
</div>

如果你不做任何额外配置，OpenClaw 将以单个智能体运行：

* `agentId` 默认值为 **`main`**。
* 会话的键为 `agent:main:<mainKey>`。
* 工作区默认是 `~/.openclaw/workspace`（如果设置了 `OPENCLAW_PROFILE`，则为 `~/.openclaw/workspace-<profile>`）。
* 状态默认存储在 `~/.openclaw/agents/main/agent`。

<div id="agent-helper">
  ## Agent 助手
</div>

使用智能体向导添加一个新的独立智能体：

```bash
openclaw agents add work
```

然后添加 `bindings`（或者交给向导自动完成）来路由传入消息。

使用以下命令进行验证：

```bash
openclaw agents list --bindings
```

<div id="multiple-agents-multiple-people-multiple-personalities">
  ## 多个智能体 = 多个用户，多种人格
</div>

使用 **多个智能体** 时，每个 `agentId` 都会成为一个 **完全隔离的人格**：

* **不同的电话号码/账号**（按渠道的 `accountId` 区分）。
* **不同的人格**（每个智能体的工作区文件，如 `AGENTS.md` 和 `SOUL.md`）。
* **独立的认证与会话**（除非显式启用，否则不会相互串话）。

这样，**多个用户**就可以共享同一个 Gateway 服务器，同时保证各自的 AI「大脑」和数据相互隔离。

<div id="one-whatsapp-number-multiple-people-dm-split">
  ## 一个 WhatsApp 号码，多个人使用（DM 分流）
</div>

你可以在使用**一个 WhatsApp 账号**的前提下，将**不同的 WhatsApp 私聊（DM）**路由到不同的智能体。通过匹配发件人的 E.164（例如 `+15551234567`）并设置 `peer.kind: "dm"` 来实现。回复消息仍然从同一个 WhatsApp 号码发出（不支持按 Agent 代理区分的发件人身份）。

重要细节：直接私聊会折叠到该 Agent 代理的**主会话 key**，所以如果要做到真正隔离，必须是**每个人一个智能体**。

示例：

```json5
{
  agents: {
    list: [
      { id: "alex", workspace: "~/.openclaw/workspace-alex" },
      { id: "mia", workspace: "~/.openclaw/workspace-mia" }
    ]
  },
  bindings: [
    { agentId: "alex", match: { channel: "whatsapp", peer: { kind: "dm", id: "+15551230001" } } },
    { agentId: "mia",  match: { channel: "whatsapp", peer: { kind: "dm", id: "+15551230002" } } }
  ],
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551230001", "+15551230002"]
    }
  }
}
```

注意：

* DM 访问控制是**在每个 WhatsApp 账号层面全局生效**（配对/允许列表），而不是针对每个智能体。
* 对于共享群组，将该群组绑定到一个智能体，或者使用[广播群组](/zh/broadcast-groups)。

<div id="routing-rules-how-messages-pick-an-agent">
  ## 路由规则（消息如何选择智能体）
</div>

绑定是**确定性的**，并且遵循“**越具体越优先**”的原则：

1. `peer` 匹配（精确的私聊/群组/频道 ID）
2. `guildId`（Discord）
3. `teamId`（Slack）
4. 通道的 `accountId` 匹配
5. 通道级别匹配（`accountId: "*"`)
6. 回退到默认智能体（`agents.list[].default`；否则为列表中的第一个条目，默认：`main`）

<div id="multiple-accounts-phone-numbers">
  ## 多个账号 / 电话号码
</div>

支持**多个账号**的渠道（例如 WhatsApp）使用 `accountId` 来标识
每个登录账号。每个 `accountId` 都可以路由到不同的智能体，因此一台服务器可以托管
多个电话号码且不会混淆会话。

<div id="concepts">
  ## 概念
</div>

* `agentId`：一个“大脑”（工作区、每个智能体的认证、每个智能体的会话存储）。
* `accountId`：一个渠道账号实例（例如 WhatsApp 账号 `"personal"` 与 `"biz"`）。
* `binding`：按 `(channel, accountId, peer)`，以及可选的 guild/team ID，将入站消息路由到某个 `agentId`。
* 直接聊天会归并为 `agent:<agentId>:<mainKey>`（每个智能体的“main”；`session.mainKey`）。

<div id="example-two-whatsapps-two-agents">
  ## 示例：两个 WhatsApp 账号 → 两个智能体
</div>

`~/.openclaw/openclaw.json`（JSON5）：

```js
{
  agents: {
    list: [
      {
        id: "home",
        default: true,
        name: "Home",
        workspace: "~/.openclaw/workspace-home",
        agentDir: "~/.openclaw/agents/home/agent",
      },
      {
        id: "work",
        name: "Work",
        workspace: "~/.openclaw/workspace-work",
        agentDir: "~/.openclaw/agents/work/agent",
      },
    ],
  },

  // Deterministic routing: first match wins (most-specific first).
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },

    // Optional per-peer override (example: send a specific group to work agent).
    {
      agentId: "work",
      match: {
        channel: "whatsapp",
        accountId: "personal",
        peer: { kind: "group", id: "1203630...@g.us" },
      },
    },
  ],

  // Off by default: agent-to-agent messaging must be explicitly enabled + allowlisted.
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },

  channels: {
    whatsapp: {
      accounts: {
        personal: {
          // 可选覆盖。默认:~/.openclaw/credentials/whatsapp/personal
          // authDir: "~/.openclaw/credentials/whatsapp/personal",
        },
        biz: {
          // Optional override. Default: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

<div id="example-whatsapp-daily-chat-telegram-deep-work">
  ## 示例：WhatsApp 日常聊天 + Telegram 深度工作
</div>

按渠道拆分：将 WhatsApp 路由到一个快速的日常智能体，将 Telegram 路由到一个 Opus Agent 代理。

```json5
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5"
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-5"
      }
    ]
  },
  bindings: [
    { agentId: "chat", match: { channel: "whatsapp" } },
    { agentId: "opus", match: { channel: "telegram" } }
  ]
}
```

说明:

* 如果你在某个渠道下有多个账号，为该绑定添加 `accountId`（例如 `{ channel: "whatsapp", accountId: "personal" }`）。
* 如果你想把某个特定的私信/群聊路由到 Opus，同时让该渠道的其他会话继续走 chat，为该会话对应的 peer 添加一个 `match.peer` 绑定；peer 级匹配规则的优先级总是高于渠道级规则。

<div id="example-same-channel-one-peer-to-opus">
  ## 示例：同一通道，将单个联系人路由到 Opus
</div>

保持 WhatsApp 由快速智能体处理，但将与某个联系人的私聊路由到 Opus：

```json5
{
  agents: {
    list: [
      { id: "chat", name: "Everyday", workspace: "~/.openclaw/workspace-chat", model: "anthropic/claude-sonnet-4-5" },
      { id: "opus", name: "Deep Work", workspace: "~/.openclaw/workspace-opus", model: "anthropic/claude-opus-4-5" }
    ]
  },
  bindings: [
    { agentId: "opus", match: { channel: "whatsapp", peer: { kind: "dm", id: "+15551234567" } } },
    { agentId: "chat", match: { channel: "whatsapp" } }
  ]
}
```

对等绑定的优先级始终更高，因此请将它们放在频道级规则前面。

<div id="family-agent-bound-to-a-whatsapp-group">
  ## 绑定到 WhatsApp 群组的家庭 Agent 代理
</div>

将一个专用的家庭 Agent 代理绑定到单个 WhatsApp 群组，并通过基于 @ 提及的访问控制
和更严格的工具策略：

```json5
{
  agents: {
    list: [
      {
        id: "family",
        name: "Family",
        workspace: "~/.openclaw/workspace-family",
        identity: { name: "Family Bot" },
        groupChat: {
          mentionPatterns: ["@family", "@familybot", "@Family Bot"]
        },
        sandbox: {
          mode: "all",
          scope: "agent"
        },
        tools: {
          allow: ["exec", "read", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
          deny: ["write", "edit", "apply_patch", "browser", "canvas", "nodes", "cron"]
        }
      }
    ]
  },
  bindings: [
    {
      agentId: "family",
      match: {
        channel: "whatsapp",
        peer: { kind: "group", id: "120363999999999999@g.us" }
      }
    }
  ]
}
```

注意事项：

* 工具允许/拒绝列表针对的是 **tools**，而不是技能（skills）。如果某个技能需要运行
  二进制程序，请确保已允许 `exec`，并且该二进制文件存在于沙箱中。
* 如需更严格的控制，请设置 `agents.list[].groupChat.mentionPatterns`，并确保为该渠道启用了群组允许列表。

<div id="per-agent-sandbox-and-tool-configuration">
  ## 按智能体的沙箱与工具配置
</div>

从 v2026.1.6 开始，每个智能体都可以拥有各自的沙箱和工具限制：

```js
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: {
          mode: "off",  // No sandbox for personal agent
        },
        // No tool restrictions - all tools available
      },
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",     // Always sandboxed
          scope: "agent",  // One container per agent
          docker: {
            // Optional one-time setup after container creation
            setupCommand: "apt-get update && apt-get install -y git curl",
          },
        },
        tools: {
          allow: ["read"],                    // Only read tool
          deny: ["exec", "write", "edit", "apply_patch"],    // 拒绝其他工具
        },
      },
    ],
  },
}
```

注意：`setupCommand` 位于 `sandbox.docker` 下，并且只在容器创建时运行一次。
当最终解析到的 scope 为 `"shared"` 时，将忽略逐智能体的 `sandbox.docker.*` 覆盖配置。

**优势：**

* **安全隔离**：为不受信任的智能体限制可用工具
* **资源控制**：仅对特定智能体启用沙箱，同时让其他智能体直接运行在宿主机上
* **灵活策略**：为每个智能体配置不同的权限

注意：`tools.elevated` 是**全局的**且基于发送方；它无法按智能体单独配置。
如果你需要逐智能体的边界，请使用 `agents.list[].tools` 来拒绝 `exec`。
对于群组内的定向路由，使用 `agents.list[].groupChat.mentionPatterns`，以便将 @ 提及精确映射到目标智能体。

详见 [多智能体沙箱 &amp; 工具](/zh/multi-agent-sandbox-tools) 以获取详细示例。
