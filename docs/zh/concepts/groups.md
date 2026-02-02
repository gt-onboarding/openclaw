---
title: 群组
summary: "跨各类平台（WhatsApp/Telegram/Discord/Slack/Signal/iMessage/Microsoft Teams）的群聊行为"
read_when:
  - 更改群聊行为或提及控制策略
---

<div id="groups">
  # 群组
</div>

OpenClaw 在各个渠道/平台上对群聊的处理方式保持一致：WhatsApp、Telegram、Discord、Slack、Signal、iMessage、Microsoft Teams。

<div id="beginner-intro-2-minutes">
  ## 入门简介（2 分钟）
</div>

OpenClaw “运行”在你自己的消息账号上。不会有单独的 WhatsApp 机器人用户。
如果**你**在某个群组里，OpenClaw 就能看到那个群组并在其中回复。

默认行为：

* 群组默认是受限的（`groupPolicy: "allowlist"`）。
* 只有在被提及时才会回复，除非你显式禁用提及门控机制。

换句话说：在允许列表里的发送方，可以通过提及 OpenClaw 来触发它。

> TL;DR
>
> * **私信访问** 由 `*.allowFrom` 控制。
> * **群组访问** 由 `*.groupPolicy` + 允许列表（`*.groups`、`*.groupAllowFrom`）控制。
> * **回复触发** 由提及门控机制（`requireMention`、`/activation`）控制。

快速流程（群组消息的处理过程）：

```
groupPolicy? disabled -> drop
groupPolicy? allowlist -> group allowed? no -> drop
requireMention? yes -> mentioned? no -> store for context only
otherwise -> reply
```

![群组消息流](/images/groups-flow.svg)

如果你想要这样配置……

| 目标                | 需要如何设置                                                     |
| ----------------- | ---------------------------------------------------------- |
| 允许所有群组，但只在被 @ 时回复 | `groups: { "*": { requireMention: true } }`                |
| 禁用所有群组回复          | `groupPolicy: "disabled"`                                  |
| 只允许特定群组           | `groups: { "<group-id>": { ... } }`（没有 `"*"` 键）            |
| 只有你可以在群组中触发       | `groupPolicy: "allowlist"`, `groupAllowFrom: ["+1555..."]` |

<div id="session-keys">
  ## 会话键
</div>

* 群组会话使用 `agent:<agentId>:<channel>:group:<id>` 作为会话键（房间/频道使用 `agent:<agentId>:<channel>:channel:<id>`）。
* Telegram 论坛话题会在群组 id 后追加 `:topic:<threadId>`，使每个话题拥有独立会话。
* 直接聊天使用主会话（或在相应配置启用时按发送者分别使用会话）。
* 群组会话不会触发心跳。

<div id="pattern-personal-dms-public-groups-single-agent">
  ## 模式：个人私信 + 公共群组（单个 Agent）
</div>

可以——如果你的「个人」流量是**私信（DMs）**，而你的「公共」流量是**群组**，这个模式非常适合。

原因是：在单智能体模式下，私信通常落在 **main** 会话键（`agent:main:main`）中，而群组始终使用 **non-main** 会话键（`agent:main:<channel>:group:<id>`）。如果你启用沙箱并设置 `mode: "non-main"`，这些群组会话会在 Docker 中运行，而你的主私信会话则保留在宿主机上。

这样你就拥有一个 Agent「大脑」（共享工作区 + 记忆），但有两种不同的执行模式：

* **私信（DMs）**：完整工具访问（宿主机）
* **群组**：沙箱 + 受限工具（Docker）

> 如果你需要真正完全隔离的工作区/人格（「个人」和「公共」绝不能混合），请再创建一个智能体并配置绑定。参见 [Multi-Agent Routing](/zh/concepts/multi-agent)。

示例（私信在宿主机上运行，群组在沙箱中运行 + 仅消息相关工具）：

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // groups/channels are non-main -> sandboxed
        scope: "session", // 最强隔离(每个群组/频道一个容器)
        workspaceAccess: "none"
      }
    }
  },
  tools: {
    sandbox: {
      tools: {
        // If allow is non-empty, everything else is blocked (deny still wins).
        allow: ["group:messaging", "group:sessions"],
        deny: ["group:runtime", "group:fs", "group:ui", "nodes", "cron", "gateway"]
      }
    }
  }
}
```

想要实现“各个组只能看到文件夹 X”，而不是“无宿主机访问权限”？保持 `workspaceAccess: "none"`，并且仅将允许列表中的路径挂载进沙箱：

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
        docker: {
          binds: [
            // 主机路径:容器路径:模式
            "~/FriendsShared:/data:ro"
          ]
        }
      }
    }
  }
}
```

相关内容：

* 配置键和默认值：[Gateway 配置](/zh/gateway/configuration#agentsdefaultssandbox)
* 排查工具被阻止的原因：[沙箱 vs 工具策略 vs 提权](/zh/gateway/sandbox-vs-tool-policy-vs-elevated)
* 绑定挂载详情：[沙箱](/zh/gateway/sandboxing#custom-bind-mounts)

<div id="display-labels">
  ## 显示标签
</div>

* UI 标签在可用时会使用 `displayName`，格式为 `<channel>:<token>`。
* `#room` 被保留用于房间/频道；群聊使用 `g-<slug>`（全部小写，空格 -&gt; `-`，保留 `#@+._-`）。

<div id="group-policy">
  ## 群组策略
</div>

控制每个渠道上群组/房间消息的处理方式：

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "disabled", // "open" | "disabled" | "allowlist" (open:允许所有群组消息 | disabled:禁用群组消息 | allowlist:仅允许列表中的群组)
      groupAllowFrom: ["+15551234567"]
    },
    telegram: {
      groupPolicy: "disabled",
      groupAllowFrom: ["123456789", "@username"]
    },
    signal: {
      groupPolicy: "disabled",
      groupAllowFrom: ["+15551234567"]
    },
    imessage: {
      groupPolicy: "disabled",
      groupAllowFrom: ["chat_id:123"]
    },
    msteams: {
      groupPolicy: "disabled",
      groupAllowFrom: ["user@org.com"]
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "GUILD_ID": { channels: { help: { allow: true } } }
      }
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } }
    },
    matrix: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["@owner:example.org"],
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true }
      }
    }
  }
}
```

| Policy        | Behavior             |
| ------------- | -------------------- |
| `"open"`      | 群组不受允许列表限制；提及门控仍然生效。 |
| `"disabled"`  | 完全阻止所有群组消息。          |
| `"allowlist"` | 仅允许与已配置允许列表匹配的群组/房间。 |

Notes:

* `groupPolicy` 独立于提及门控（后者需要 @ 提及）。
* WhatsApp/Telegram/Signal/iMessage/Microsoft Teams：使用 `groupAllowFrom`（否则回退到显式的 `allowFrom`）。
* Discord：允许列表使用 `channels.discord.guilds.<id>.channels`。
* Slack：允许列表使用 `channels.slack.channels`。
* Matrix：允许列表使用 `channels.matrix.groups`（房间 ID、别名或名称）。使用 `channels.matrix.groupAllowFrom` 限制发送方；也支持按房间配置 `users` 允许列表。
* 群组私信单独控制（`channels.discord.dm.*`，`channels.slack.dm.*`）。
* Telegram 允许列表可以匹配用户 ID（`"123456789"`、`"telegram:123456789"`、`"tg:123456789"`）或用户名（`"@alice"` 或 `"alice"`）；前缀不区分大小写。
* 默认是 `groupPolicy: "allowlist"`；如果你的群组允许列表为空，将阻止所有群组消息。

Quick mental model（群组消息的评估顺序）:

1. `groupPolicy`（open/disabled/allowlist）
2. 群组允许列表（`*.groups`、`*.groupAllowFrom`、特定渠道的允许列表）
3. 提及门控（`requireMention`、`/activation`）

<div id="mention-gating-default">
  ## 提及门控（默认）
</div>

群组消息必须包含提及，除非被该群组的单独配置覆盖。各子系统的默认配置位于 `*.groups."*"` 下。

在频道支持回复元数据时，回复机器人消息会被视为隐式提及。这适用于 Telegram、WhatsApp、Slack、Discord 和 Microsoft Teams。

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
        "123@g.us": { requireMention: false }
      }
    },
    telegram: {
      groups: {
        "*": { requireMention: true },
        "123456789": { requireMention: false }
      }
    },
    imessage: {
      groups: {
        "*": { requireMention: true },
        "123": { requireMention: false }
      }
    }
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@openclaw", "openclaw", "\\+15555550123"],
          historyLimit: 50
        }
      }
    ]
  }
}
```

注意：

* `mentionPatterns` 为不区分大小写的正则表达式。
* 对于支持显式 @ 提及的界面/入口，只要出现显式提及就会放行；模式只是后备方案。
* 按智能体级别的覆盖配置：`agents.list[].groupChat.mentionPatterns`（在多个智能体共享一个群组时很有用）。
* 只有在可以进行提及检测时（已配置原生提及或 `mentionPatterns`）才会启用并强制执行提及门控。
* Discord 默认值位于 `channels.discord.guilds."*"` 中（可按 guild/channel 覆盖）。
* 群组历史上下文在所有渠道上以统一方式封装，且仅适用于**待处理消息**（因提及门控而被跳过的消息）；使用 `messages.groupChat.historyLimit` 作为全局默认值，并使用 `channels.<channel>.historyLimit`（或 `channels.<channel>.accounts.*.historyLimit`）进行覆盖。设置为 `0` 可禁用。

<div id="groupchannel-tool-restrictions-optional">
  ## 群组/频道工具限制（可选）
</div>

某些频道配置支持限制**在特定群组/房间/频道内**可用的工具。

* `tools`：为整个群组设置允许/拒绝使用的工具。
* `toolsBySender`：在群组内按发送者定义覆盖规则（键为发送者 ID/用户名/邮箱/电话号码，具体取决于频道）。使用 `"*"` 作为通配符。

匹配顺序（越具体优先）：

1. 群组/频道级别的 `toolsBySender` 匹配
2. 群组/频道级别的 `tools`
3. 默认（`"*"`）的 `toolsBySender` 匹配
4. 默认（`"*"`）的 `tools`

示例（Telegram）：

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { tools: { deny: ["exec"] } },
        "-1001234567890": {
          tools: { deny: ["exec", "read", "write"] },
          toolsBySender: {
            "123456789": { alsoAllow: ["exec"] }
          }
        }
      }
    }
  }
}
```

说明：

* 群组/频道工具限制是在全局/Agent 代理工具策略基础上叠加生效的（如有冲突仍以 `deny` 为准）。
* 某些渠道在房间/频道的嵌套层级上有所不同（例如，Discord 使用 `guilds.*.channels.*`，Slack 使用 `channels.*`，MS Teams 使用 `teams.*.channels.*`）。

<div id="group-allowlists">
  ## 群组允许列表
</div>

当配置了 `channels.whatsapp.groups`、`channels.telegram.groups` 或 `channels.imessage.groups` 时，这些键就构成一个群组允许列表。使用 `"*"` 可以允许所有群组，同时仍然设置默认的提及行为。

常见配置意图（可复制粘贴）：

1. 禁用所有群组回复

```json5
{
  channels: { whatsapp: { groupPolicy: "disabled" } }
}
```

2. 仅允许指定的 WhatsApp 群组

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "123@g.us": { requireMention: true },
        "456@g.us": { requireMention: false }
      }
    }
  }
}
```

3. 允许所有群组，但要求显式提及

```json5
{
  channels: {
    whatsapp: {
      groups: { "*": { requireMention: true } }
    }
  }
}
```

4. 仅群组所有者可以在群组中触发（WhatsApp）

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
      groups: { "*": { requireMention: true } }
    }
  }
}
```

<div id="activation-owner-only">
  ## 激活（仅限所有者）
</div>

群组所有者可以切换每个群组的激活方式：

* `/activation mention`
* `/activation always`

所有者由 `channels.whatsapp.allowFrom` 决定（如果未设置，则为机器人的自身 E.164 号码）。将该命令作为单独的一条消息发送。其他界面当前会忽略 `/activation`。

<div id="context-fields">
  ## 上下文字段
</div>

群组入站负载会设置：

* `ChatType=group`
* `GroupSubject`（如果已知）
* `GroupMembers`（如果已知）
* `WasMentioned`（提及门控结果）
* Telegram 论坛主题还会包含 `MessageThreadId` 和 `IsForum`。

在新的群组会话的第一轮对话中，Agent 系统提示词会包含一个群组简介。它会提醒模型像人类一样回复、避免使用 Markdown 表格，并避免输入字面形式的 `\n` 序列。

<div id="imessage-specifics">
  ## iMessage 相关说明
</div>

* 在进行路由或配置允许列表时，优先使用 `chat_id:<id>`。
* 列出会话：`imsg chats --limit 20`。
* 群组回复始终会回到相同的 `chat_id`。

<div id="whatsapp-specifics">
  ## WhatsApp 特定说明
</div>

有关 WhatsApp 特定行为（历史注入、提及处理细节），请参阅[群组消息](/zh/concepts/group-messages)。