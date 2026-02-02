---
title: 配置
summary: "~/.openclaw/openclaw.json 的所有配置项及示例"
read_when:
  - 添加或修改配置字段时
---

<div id="configuration">
  # 配置 🔧
</div>

OpenClaw 会从 `~/.openclaw/openclaw.json` 读取一个可选的 **JSON5** 配置文件（允许注释和尾随逗号）。

如果该文件不存在，OpenClaw 会使用相对安全的默认配置（内置 Pi Agent 代理 + 按发送者区分的会话 + 工作区 `~/.openclaw/workspace`）。通常你只需要配置文件来：

* 限制谁可以触发机器人（`channels.whatsapp.allowFrom`、`channels.telegram.allowFrom` 等）
* 控制群组允许列表和 @ 提及行为（`channels.whatsapp.groups`、`channels.telegram.groups`、`channels.discord.guilds`、`agents.list[].groupChat`）
* 自定义消息前缀（`messages`）
* 设置智能体的工作区（`agents.defaults.workspace` 或 `agents.list[].workspace`）
* 调整内置智能体默认值（`agents.defaults`）和会话行为（`session`）
* 设置每个智能体的身份信息（`agents.list[].identity`）

> **第一次接触配置？** 请查看 [配置示例](/zh/gateway/configuration-examples) 指南，其中包含完整示例和详细说明！

<div id="strict-config-validation">
  ## 严格的配置校验
</div>

OpenClaw 只接受与 schema 完全匹配的配置。
未知键、类型不正确或无效值都会导致 Gateway 出于安全原因**拒绝启动**。

当校验失败时：

* Gateway 不会启动。
* 只允许运行诊断类命令（例如：`openclaw doctor`、`openclaw logs`、`openclaw health`、`openclaw status`、`openclaw service`、`openclaw help`）。
* 运行 `openclaw doctor` 以查看具体问题。
* 运行 `openclaw doctor --fix`（或 `--yes`）以应用迁移/修复。

`openclaw doctor` 在你没有明确使用 `--fix`/`--yes` 之前，绝不会对配置或状态进行任何修改。

<div id="schema-ui-hints">
  ## Schema + UI 提示
</div>

Gateway 通过 `config.schema` 向 UI 编辑器暴露配置的 JSON Schema 表达形式。
Control UI 会基于该 schema 渲染表单，并提供一个 **Raw JSON** 编辑器作为备用通道。

Channel 插件和扩展可以为其配置注册 schema + UI 提示，这样在不同应用中，
频道设置都能保持基于 schema，而不依赖硬编码表单。

这些提示（标签、分组、敏感字段）会与 schema 一并下发，这样客户端在无需硬编码配置细节的情况下，
也能渲染出更合理的表单。

<div id="apply-restart-rpc">
  ## 应用并重启（RPC）
</div>

使用 `config.apply` 一次性完成完整配置的校验、写入并重启 Gateway。
它会写入一个重启哨兵标记，并在 Gateway 重启完成后 ping 最后一个活动会话。

警告：`config.apply` 会替换**整个配置**。如果你只想修改少数几个键，
请使用 `config.patch` 或 `openclaw config set`。务必备份 `~/.openclaw/openclaw.json`。

参数：

* `raw` (string) — 整个配置的 JSON5 载荷
* `baseHash` (optional) — 来自 `config.get` 的配置哈希（当已有配置时必需）
* `sessionKey` (optional) — 最后一个活动会话的 key，用于唤醒 ping
* `note` (optional) — 要包含在重启哨兵标记中的备注
* `restartDelayMs` (optional) — 重启前的延迟毫秒数（默认为 2000）

示例（通过 `gateway call`）：

```bash
openclaw gateway call config.get --params '{}' # 捕获 payload.hash
openclaw gateway call config.apply --params '{
  "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

<div id="partial-updates-rpc">
  ## 部分更新（RPC）
</div>

使用 `config.patch` 将部分更新合并到现有配置中，而不会覆盖无关的键。
它采用 JSON merge patch 语义：

* 对象递归合并
* `null` 会删除一个键
* 数组则整体替换

  类似 `config.apply`，它会进行校验、写入配置、存储重启哨兵标记，并调度
  Gateway 重启（如果提供了 `sessionKey`，则会在唤醒时使用）。

参数：

* `raw`（string）— 仅包含需要变更键的 JSON5 负载
* `baseHash`（必填）— 来自 `config.get` 的配置哈希
* `sessionKey`（可选）— 用于唤醒 ping 的最近一次活动会话键
* `note`（可选）— 要包含在重启哨兵标记中的备注
* `restartDelayMs`（可选）— 重启前的延迟（默认 2000）

示例：

```bash
openclaw gateway call config.get --params '{}' # 捕获 payload.hash
openclaw gateway call config.patch --params '{
  "raw": "{\\n  channels: { telegram: { groups: { \\"*\\": { requireMention: false } } } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

<div id="minimal-config-recommended-starting-point">
  ## 最小配置（推荐起点）
</div>

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } }
}
```

使用以下命令构建一次默认镜像：

```bash
scripts/sandbox-setup.sh
```

<div id="self-chat-mode-recommended-for-group-control">
  ## 自聊模式（推荐用于群聊控制）
</div>

为防止机器人在 WhatsApp 群聊中对 @ 提及做出响应（仅响应特定的文本触发词）：

```json5
{
  agents: {
    defaults: { workspace: "~/.openclaw/workspace" },
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["@openclaw", "reisponde"] }
      }
    ]
  },
  channels: {
    whatsapp: {
      // 允许列表仅限私信;包含您自己的号码将启用自聊模式。
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } }
    }
  }
}
```

<div id="config-includes-include">
  ## 配置包含（`$include`）
</div>

使用 `$include` 指令将你的配置拆分为多个文件。这样做在以下场景中很有用：

* 组织大型配置（例如按客户端划分的智能体定义）
* 在不同环境之间共享通用设置
* 将敏感配置拆分为独立文件单独管理

<div id="basic-usage">
  ### 基本用法
</div>

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  
  // 包含单个文件(替换键的值)
  agents: { "$include": "./agents.json5" },
  
  // Include multiple files (deep-merged in order)
  broadcast: { 
    "$include": [
      "./clients/mueller.json5",
      "./clients/schmidt.json5"
    ]
  }
}
```

```json5
// ~/.openclaw/agents.json5
{
  defaults: { sandbox: { mode: "all", scope: "session" } },
  list: [
    { id: "main", workspace: "~/.openclaw/workspace" }
  ]
}
```

<div id="merge-behavior">
  ### 合并行为
</div>

* **单个文件**：替换包含 `$include` 的对象本身
* **文件数组**：按顺序对文件进行深度合并（后面的文件覆盖前面的文件）
* **存在同级键时**：同级键在 include 之后再合并（会覆盖已 include 的值）
* **同级键 + 数组/原始值**：不支持（include 的内容必须是对象）

```json5
// 同级键覆盖包含的值
{
  "$include": "./base.json5",   // { a: 1, b: 2 }
  b: 99                          // Result: { a: 1, b: 99 }
}
```

<div id="nested-includes">
  ### 嵌套包含
</div>

被包含的文件本身也可以包含 `$include` 指令（最多可嵌套 10 层）：

```json5
// clients/mueller.json5
{
  agents: { "$include": "./mueller/agents.json5" },
  broadcast: { "$include": "./mueller/broadcast.json5" }
}
```

<div id="path-resolution">
  ### 路径解析
</div>

* **相对路径**：相对于包含该内容的文件进行解析
* **绝对路径**：按原样使用
* **父目录**：`../` 引用可按预期使用

```json5
{ "$include": "./sub/config.json5" }      // relative
{ "$include": "/etc/openclaw/base.json5" } // absolute
{ "$include": "../shared/common.json5" }   // 父目录
```

<div id="error-handling">
  ### 错误处理
</div>

* **缺少文件**：会使用解析后的路径给出清晰的错误信息
* **解析错误**：会指出是哪一个被包含的文件解析失败
* **循环包含**：会检测到并连同包含链一并报告

<div id="example-multi-client-legal-setup">
  ### 示例：多客户法律配置
</div>

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789, auth: { token: "secret" } },
  
  // Common agent defaults
  agents: {
    defaults: {
      sandbox: { mode: "all", scope: "session" }
    },
    // 合并所有客户端的智能体列表
    list: { "$include": [
      "./clients/mueller/agents.json5",
      "./clients/schmidt/agents.json5"
    ]}
  },
  
  // Merge broadcast configs
  broadcast: { "$include": [
    "./clients/mueller/broadcast.json5",
    "./clients/schmidt/broadcast.json5"
  ]},
  
  channels: { whatsapp: { groupPolicy: "allowlist" } }
}
```

```json5
// ~/.openclaw/clients/mueller/agents.json5
[
  { id: "mueller-transcribe", workspace: "~/clients/mueller/transcribe" },
  { id: "mueller-docs", workspace: "~/clients/mueller/docs" }
]
```

```json5
// ~/.openclaw/clients/mueller/broadcast.json5
{
  "120363403215116621@g.us": ["mueller-transcribe", "mueller-docs"]
}
```

<div id="common-options">
  ## 通用配置项
</div>

<div id="env-vars-env">
  ### 环境变量 + `.env`
</div>

OpenClaw 会从父进程（shell、launchd/systemd、CI 等）读取环境变量。

此外，它还会加载：

* 当前工作目录下的 `.env` 文件（如果存在）
* 全局备用 `.env` 文件，路径为 `~/.openclaw/.env`（即 `$OPENCLAW_STATE_DIR/.env`）

这两个 `.env` 文件都不会覆盖已存在的环境变量值。

你也可以在配置中内联提供环境变量。只有当进程环境中缺少该键时才会生效（同样遵循“不覆盖”规则）：

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-..."
    }
  }
}
```

有关完整的优先级顺序和配置来源，请参见 [/environment](/zh/environment)。

<div id="envshellenv-optional">
  ### `env.shellEnv`（可选）
</div>

选择性启用的便捷功能：如果启用，并且预期的键目前都还未设置，OpenClaw 会运行你的登录 shell，并只导入缺失的这些预期键（绝不会覆盖已存在的值）。
这在效果上等同于对你的 shell 启动配置文件执行一次 `source`。

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000
    }
  }
}
```

对应的环境变量：

* `OPENCLAW_LOAD_SHELL_ENV=1`
* `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

<div id="env-var-substitution-in-config">
  ### 配置中的环境变量替换
</div>

你可以在任何配置项的字符串值中直接引用环境变量，使用
`${VAR_NAME}` 语法。变量会在加载配置时、验证之前完成替换。

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}"
      }
    }
  },
  gateway: {
    auth: {
      token: "${OPENCLAW_GATEWAY_TOKEN}"
    }
  }
}
```

**规则：**

* 仅匹配全大写的环境变量名：`[A-Z_][A-Z0-9_]*`
* 缺失或为空的环境变量会在加载配置时抛出错误
* 使用 `$${VAR}` 转义以输出字面量 `${VAR}`
* 可与 `$include` 一起使用（被包含的文件同样会进行替换）

**内联替换：**

```json5
{
  models: {
    providers: {
      custom: {
        baseUrl: "${CUSTOM_API_BASE}/v1"  // → "https://api.example.com/v1"
      }
    }
  }
}
```

<div id="auth-storage-oauth-api-keys">
  ### 认证存储（OAuth + API keys）
</div>

OpenClaw 将**每个智能体**的认证配置（OAuth + API keys）存储在：

* `<agentDir>/auth-profiles.json`（默认：`~/.openclaw/agents/<agentId>/agent/auth-profiles.json`）

另见：[/concepts/oauth](/zh/concepts/oauth)

旧版 OAuth 导入位置：

* `~/.openclaw/credentials/oauth.json`（或 `$OPENCLAW_STATE_DIR/credentials/oauth.json`）

嵌入式 Pi 智能体会在运行时维护一个缓存：

* `<agentDir>/auth.json`（自动管理；请勿手动编辑）

旧版智能体目录（多智能体之前）：

* `~/.openclaw/agent/*`（由 `openclaw doctor` 迁移至 `~/.openclaw/agents/<defaultAgentId>/agent/*`）

覆盖项：

* OAuth 目录（仅用于旧版导入）：`OPENCLAW_OAUTH_DIR`
* 智能体目录（用于覆盖默认智能体根目录）：`OPENCLAW_AGENT_DIR`（推荐），`PI_CODING_AGENT_DIR`（旧版）

首次使用时，OpenClaw 会将 `oauth.json` 中的条目导入到 `auth-profiles.json` 中。

<div id="auth">
  ### `auth`
</div>

用于认证配置档的可选元数据。这里**不会**存储任何机密信息；它只会将配置档 ID 映射到提供方 + 模式（以及可选的电子邮件地址），并定义在故障切换时要使用的提供方轮换顺序。

```json5
{
  auth: {
    profiles: {
      "anthropic:me@example.com": { provider: "anthropic", mode: "oauth", email: "me@example.com" },
      "anthropic:work": { provider: "anthropic", mode: "api_key" }
    },
    order: {
      anthropic: ["anthropic:me@example.com", "anthropic:work"]
    }
  }
}
```

<div id="agentslistidentity">
  ### `agents.list[].identity`
</div>

可选的每个智能体的身份配置，用于默认值和用户体验。该字段由 macOS 引导助手写入。

如果设置了它，OpenClaw 会推导出一些默认值（仅在你没有显式设置时）：

* `messages.ackReaction` 来自**当前激活的智能体**的 `identity.emoji`（如果没有则回退为 👀）
* `agents.list[].groupChat.mentionPatterns` 来自该智能体的 `identity.name`/`identity.emoji`（这样“@Samantha”在 Telegram/Slack/Discord/Google Chat/iMessage/WhatsApp 群聊中都能正常工作）
* `identity.avatar` 接受一个相对于工作区的图片路径或一个远程 URL/data URL。本地文件必须位于该智能体工作区内部。

`identity.avatar` 接受：

* 相对于工作区的路径（必须保持在该智能体工作区内）
* `http(s)` URL
* `data:` URI

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png"
        }
      }
    ]
  }
}
```

<div id="wizard">
  ### `wizard`
</div>

由 CLI 向导（`onboard`、`configure`、`doctor`）生成的元数据。

```json5
{
  wizard: {
    lastRunAt: "2026-01-01T00:00:00.000Z",
    lastRunVersion: "2026.1.4",
    lastRunCommit: "abc1234",
    lastRunCommand: "configure",
    lastRunMode: "local"
  }
}
```

<div id="logging">
  ### `logging`
</div>

* 默认日志文件：`/tmp/openclaw/openclaw-YYYY-MM-DD.log`
* 如果你需要固定文件路径，将 `logging.file` 设置为 `/tmp/openclaw/openclaw.log`。
* 控制台输出可以单独配置：
  * `logging.consoleLevel`（默认为 `info`，使用 `--verbose` 时提升为 `debug`）
  * `logging.consoleStyle`（`pretty` | `compact` | `json`）
* 可以对工具摘要进行脱敏，以避免泄露敏感信息：
  * `logging.redactSensitive`（`off` | `tools`，默认：`tools`）
  * `logging.redactPatterns`（正则表达式字符串数组；会覆盖默认值）

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty",
    redactSensitive: "tools",
    redactPatterns: [
      // 示例:用您自己的规则覆盖默认值。
      "\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1",
      "/\\bsk-[A-Za-z0-9_-]{8,}\\b/gi"
    ]
  }
}
```

<div id="channelswhatsappdmpolicy">
  ### `channels.whatsapp.dmPolicy`
</div>

控制 WhatsApp 私聊（DM）的处理方式：

* `"pairing"`（默认）：未知发送方会收到一个配对码；需要所有者批准
* `"allowlist"`：只允许 `channels.whatsapp.allowFrom`（或已配对的允许列表存储）中的发送方
* `"open"`：允许所有入站 DM（**需要** `channels.whatsapp.allowFrom` 包含 `"*"`；此设置表示可以从任意用户无限制接收消息）
* `"disabled"`：忽略所有入站 DM

配对码在 1 小时后过期；机器人只会在创建新请求时发送配对码。待处理 DM 配对请求默认上限为 **每个通道 3 个**。

配对审批：

* `openclaw pairing list whatsapp`
* `openclaw pairing approve whatsapp <code>`

<div id="channelswhatsappallowfrom">
  ### `channels.whatsapp.allowFrom`
</div>

E.164 电话号码的允许列表，可用于触发 WhatsApp 自动回复（**仅限私信**）。
如果为空且 `channels.whatsapp.dmPolicy="pairing"`，未知发件人将会收到配对码。
对于群组，请使用 `channels.whatsapp.groupPolicy` + `channels.whatsapp.groupAllowFrom`。

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000, // optional outbound chunk size (chars)
      chunkMode: "length", // 可选:分块模式(length | newline)
      mediaMaxMb: 50 // optional inbound media cap (MB)
    }
  }
}
```

<div id="channelswhatsappsendreadreceipts">
  ### `channels.whatsapp.sendReadReceipts`
</div>

控制是否将收到的 WhatsApp 消息标记为已读（蓝色对勾）。默认值：`true`。

在自聊模式下始终跳过已读回执，即使已启用也是如此。

按账号单独覆盖：`channels.whatsapp.accounts.<id>.sendReadReceipts`。

```json5
{
  channels: {
    whatsapp: { sendReadReceipts: false }
  }
}
```

<div id="channelswhatsappaccounts-multi-account">
  ### `channels.whatsapp.accounts`（多账户）
</div>

在单个 Gateway 中运行多个 WhatsApp 账户：

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {}, // optional; keeps the default id stable
        personal: {},
        biz: {
          // 可选覆盖。默认:~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        }
      }
    }
  }
}
```

注意：

* 出站命令在存在名为 `default` 的账户时，会默认使用该账户；否则会使用按排序后第一个已配置的账户 ID。
* 旧版的单账户 Baileys 身份验证目录会由 `openclaw doctor` 迁移到 `whatsapp/default`。

<div id="channelstelegramaccounts-channelsdiscordaccounts-channelsgooglechataccounts-channelsslackaccounts-channelsmattermostaccounts-channelssignalaccounts-channelsimessageaccounts">
  ### `channels.telegram.accounts` / `channels.discord.accounts` / `channels.googlechat.accounts` / `channels.slack.accounts` / `channels.mattermost.accounts` / `channels.signal.accounts` / `channels.imessage.accounts`
</div>

在每个通道下运行多个账户（每个账户都有自己的 `accountId` 和可选的 `name`）：

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC..."
        },
        alerts: {
          name: "Alerts bot",
          botToken: "987654:XYZ..."
        }
      }
    }
  }
}
```

Notes:

* 当省略 `accountId` 时会使用 `default`（CLI + 路由）。
* 环境变量令牌只适用于 **default** 账户。
* 基础通道设置（群组策略、@ 提及门控等）适用于所有账户，除非被单个账户级别的配置覆盖。
* 使用 `bindings[].match.accountId` 将每个账户路由到不同的 agents.defaults。

<div id="group-chat-mention-gating-agentslistgroupchat-messagesgroupchat">
  ### 群聊提及门控（`agents.list[].groupChat` + `messages.groupChat`）
</div>

群聊消息默认**要求包含提及**（通过元数据提及或正则模式）。适用于 WhatsApp、Telegram、Discord、Google Chat 和 iMessage 群聊。

**提及类型：**

* **元数据提及**：平台原生 @ 提及（例如 WhatsApp 的点按提及）。在 WhatsApp 自聊模式中会被忽略（参见 `channels.whatsapp.allowFrom`）。
* **文本匹配模式**：在 `agents.list[].groupChat.mentionPatterns` 中定义的正则表达式模式。无论是否为自聊模式，都会始终进行检查。
* 仅当能够进行提及检测（存在原生提及或至少一个 `mentionPattern`）时，才会强制启用提及门控。

```json5
{
  messages: {
    groupChat: { historyLimit: 50 }
  },
  agents: {
    list: [
      { id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }
    ]
  }
}
```

`messages.groupChat.historyLimit` 用于设置群聊历史上下文上限的全局默认值。各个 channel 可以通过 `channels.<channel>.historyLimit`（多账号场景则使用 `channels.<channel>.accounts.*.historyLimit`）进行覆盖。将其设为 `0` 可禁用历史封装（wrapping）功能。

<div id="dm-history-limits">
  #### 私信历史记录限制
</div>

私信（DM）会话使用由智能体管理的基于会话的历史记录。你可以限制每个私信会话中保留的用户对话轮数：

```json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30,  // limit DM sessions to 30 user turns
      dms: {
        "123456789": { historyLimit: 50 }  // 按用户覆盖(用户 ID)
      }
    }
  }
}
```

生效顺序：

1. 按私聊级别覆盖：`channels.<provider>.dms[userId].historyLimit`
2. 提供方默认值：`channels.<provider>.dmHistoryLimit`
3. 无限制（保留全部历史）

支持的提供方：`telegram`、`whatsapp`、`discord`、`slack`、`signal`、`imessage`、`msteams`。

按智能体级别覆盖（当设置时优先生效，即使值为 `[]`）：

```json5
{
  agents: {
    list: [
      { id: "work", groupChat: { mentionPatterns: ["@workbot", "\\+15555550123"] } },
      { id: "personal", groupChat: { mentionPatterns: ["@homebot", "\\+15555550999"] } }
    ]
  }
}
```

提及门控默认值是按频道分别生效的（`channels.whatsapp.groups`、`channels.telegram.groups`、`channels.imessage.groups`、`channels.discord.guilds`）。当设置了 `*.groups` 时，它同时也充当群组允许列表；包含 `"*"` 以允许所有群组。

若要**仅**响应特定文本触发词（忽略平台原生的 @ 提及）：

```json5
{
  channels: {
    whatsapp: {
      // 包含您自己的号码以启用自聊模式(忽略原生 @-提及)。
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } }
    }
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          // Only these text patterns will trigger responses
          mentionPatterns: ["reisponde", "@openclaw"]
        }
      }
    ]
  }
}
```

<div id="group-policy-per-channel">
  ### 群组策略（按通道）
</div>

使用 `channels.*.groupPolicy` 来控制是否允许接收群组/房间消息：

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"]
    },
    telegram: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["tg:123456789", "@alice"]
    },
    signal: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"]
    },
    imessage: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["chat_id:123"]
    },
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"]
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "GUILD_ID": {
          channels: { help: { allow: true } }
        }
      }
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } }
    }
  }
}
```

Notes:

* `"open"`：群组绕过允许列表；基于提及的门控规则仍然生效。
* `"disabled"`：阻止所有群组/房间消息。
* `"allowlist"`：只允许匹配已配置允许列表的群组/房间。
* 当某个提供方的 `groupPolicy` 未设置时，`channels.defaults.groupPolicy` 用作默认值。
* WhatsApp/Telegram/Signal/iMessage/Microsoft Teams 使用 `groupAllowFrom`（否则回退到显式的 `allowFrom`）。
* Discord/Slack 使用频道允许列表（`channels.discord.guilds.*.channels`、`channels.slack.channels`）。
* 群组私信（Discord/Slack）仍由 `dm.groupEnabled` + `dm.groupChannels` 控制。
* 默认值为 `groupPolicy: "allowlist"`（除非被 `channels.defaults.groupPolicy` 覆盖）；如果未配置任何允许列表，则会阻止群组消息。

<div id="multi-agent-routing-agentslist-bindings">
  ### 多智能体路由（`agents.list` + `bindings`）
</div>

在一个 Gateway 内运行多个相互隔离的智能体（独立工作区、`agentDir`、会话）。
入站消息通过 bindings 路由到某个智能体。

* `agents.list[]`: 针对每个智能体的覆盖配置。
  * `id`: 稳定的智能体 id（必填）。
  * `default`: 可选；如果设置了多个，按顺序第一个生效，并记录一条警告日志。
    如果都未设置，则列表中的**第一个条目**是默认智能体。
  * `name`: 智能体的显示名称。
  * `workspace`: 默认为 `~/.openclaw/workspace-<agentId>`（对于 `main`，回退到 `agents.defaults.workspace`）。
  * `agentDir`: 默认为 `~/.openclaw/agents/<agentId>/agent`。
  * `model`: 智能体级默认模型，会覆盖该智能体的 `agents.defaults.model`。
    * 字符串形式：`"provider/model"`，仅覆盖 `agents.defaults.model.primary`
    * 对象形式：`{ primary, fallbacks }`（fallbacks 覆盖 `agents.defaults.model.fallbacks`；`[]` 会为该智能体禁用全局 fallbacks）
  * `identity`: 针对智能体的名称/主题/emoji（用于提及匹配模式和确认反应）。
  * `groupChat`: 针对智能体的群聊提及门控配置（`mentionPatterns`）。
  * `sandbox`: 针对智能体的沙箱配置（覆盖 `agents.defaults.sandbox`）。
    * `mode`: `"off"` | `"non-main"` | `"all"`
    * `workspaceAccess`: `"none"` | `"ro"` | `"rw"`
    * `scope`: `"session"` | `"agent"` | `"shared"`
    * `workspaceRoot`: 自定义沙箱工作区根目录
    * `docker`: 针对智能体的 docker 覆盖配置（例如 `image`、`network`、`env`、`setupCommand`、资源限制；当 `scope: "shared"` 时会被忽略）
    * `browser`: 针对智能体的沙箱浏览器覆盖配置（当 `scope: "shared"` 时会被忽略）
    * `prune`: 针对智能体的沙箱清理策略覆盖配置（当 `scope: "shared"` 时会被忽略）
  * `subagents`: 针对智能体的子智能体默认配置。
    * `allowAgents`: 用于 `sessions_spawn` 从该智能体派生的智能体 id 允许列表（`["*"]` = 允许任意；默认：仅允许同一智能体）
  * `tools`: 针对智能体的工具限制（在沙箱工具策略之前应用）。
    * `profile`: 基础工具配置（在 allow/deny 之前应用）
    * `allow`: 允许的工具名称数组
    * `deny`: 禁止的工具名称数组（禁止优先生效）
* `agents.defaults`: 共享的智能体默认配置（model、workspace、sandbox 等）。
* `bindings[]`: 将入站消息路由到某个 `agentId`。
  * `match.channel`（必填）
  * `match.accountId`（可选；`*` = 任意账号；省略 = 默认账号）
  * `match.peer`（可选；`{ kind: dm|group|channel, id }`）
  * `match.guildId` / `match.teamId`（可选；与具体通道相关）

确定性匹配顺序：

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId`（精确匹配，不包含 peer/guild/team）
5. `match.accountId: "*"`（通道范围，不包含 peer/guild/team）
6. 默认智能体（`agents.list[].default`，否则列表第一个条目，否则 `"main"`）

在每一层匹配级别中，`bindings` 中第一个匹配的条目具有优先权。

<div id="per-agent-access-profiles-multi-agent">
  #### 每个智能体的访问配置（多智能体）
</div>

每个智能体都可以拥有自己的沙箱和工具策略。利用这一点，你可以在同一个 Gateway 中配置不同的访问级别：

* **完全访问**（个人智能体）
* **只读** 工具和工作区
* **禁止访问文件系统**（仅限消息/会话类工具）

参见 [Multi-Agent Sandbox &amp; Tools](/zh/multi-agent-sandbox-tools) 了解优先级和更多示例。

完全访问（无沙箱）：

```json5
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: { mode: "off" }
      }
    ]
  }
}
```

只读工具 + 只读工作区：

```json5
{
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "ro"
        },
        tools: {
          allow: ["read", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
          deny: ["write", "edit", "apply_patch", "exec", "process", "browser"]
        }
      }
    ]
  }
}
```

不提供文件系统访问（启用消息/会话工具）：

```json5
{
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "none"
        },
        tools: {
          allow: ["sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status", "whatsapp", "telegram", "slack", "discord", "gateway"],
          deny: ["read", "write", "edit", "apply_patch", "exec", "process", "browser", "canvas", "nodes", "cron", "gateway", "image"]
        }
      }
    ]
  }
}
```

示例：两个 WhatsApp 账号 → 两个智能体：

```json5
{
  agents: {
    list: [
      { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
      { id: "work", workspace: "~/.openclaw/workspace-work" }
    ]
  },
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } }
  ],
  channels: {
    whatsapp: {
      accounts: {
        personal: {},
        biz: {},
      }
    }
  }
}
```

<div id="toolsagenttoagent-optional">
  ### `tools.agentToAgent`（可选）
</div>

智能体之间的消息传递需要主动启用：

```json5
{
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"]
    }
  }
}
```

<div id="messagesqueue">
  ### `messages.queue`
</div>

控制在已有智能体运行时如何处理入站消息的行为。

```json5
{
  messages: {
    queue: {
      mode: "collect", // steer | followup | collect | steer-backlog (steer+backlog ok) | interrupt (queue=steer 遗留)
      debounceMs: 1000,
      cap: 20,
      drop: "summarize", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
        discord: "collect",
        imessage: "collect",
        webchat: "collect"
      }
    }
  }
}
```

<div id="messagesinbound">
  ### `messages.inbound`
</div>

对来自**同一发送方**的快速连续入站消息进行防抖处理，将多条连续
消息合并为一个智能体轮次。防抖以通道 + 会话为粒度，并使用最新消息
作为回复线程/ID 的依据。

```json5
{
  messages: {
    inbound: {
      debounceMs: 2000, // 0 禁用
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500
      }
    }
  }
}
```

说明：

* 对**纯文本**消息进行防抖分批处理；媒体/附件会立即发送。
* 控制命令（例如 `/queue`、`/new`）会绕过防抖处理，以保持为独立消息。

<div id="commands-chat-command-handling">
  ### `commands`（聊天命令处理）
</div>

控制聊天命令在各个连接器上的启用方式。

```json5
{
  commands: {
    native: "auto",         // register native commands when supported (auto)
    text: true,             // parse slash commands in chat messages
    bash: false,            // 允许 !（别名：/bash）（仅限主机；需要 tools.elevated 允许列表）
    bashForegroundMs: 2000, // bash foreground window (0 backgrounds immediately)
    config: false,          // allow /config (writes to disk)
    debug: false,           // allow /debug (runtime-only overrides)
    restart: false,         // allow /restart + gateway restart tool
    useAccessGroups: true   // enforce access-group allowlists/policies for commands
  }
}
```

Notes:

* 文本指令必须作为**独立**消息发送，并且使用前缀 `/`（不支持纯文本别名）。
* `commands.text: false` 会禁用对聊天消息中的指令解析。
* `commands.native: "auto"`（默认）会为 Discord/Telegram 启用原生指令，并保持 Slack 关闭；不支持的渠道将仅使用文本指令。
* 通过设置 `commands.native: true|false` 可对所有渠道统一强制启用/关闭，或使用 `channels.discord.commands.native`、`channels.telegram.commands.native`、`channels.slack.commands.native`（布尔值或 `"auto"`）按渠道单独覆盖。`false` 会在启动时清除 Discord/Telegram 上之前注册的指令；Slack 指令在 Slack 应用内管理。
* `channels.telegram.customCommands` 会为 Telegram 机器人菜单添加额外条目。名称会被规范化；与原生指令冲突的条目会被忽略。
* `commands.bash: true` 允许通过 `! <cmd>` 运行宿主 shell 命令（`/bash <cmd>` 也可用作别名）。需要启用 `tools.elevated.enabled`，并在 `tools.elevated.allowFrom.<channel>` 中将发送者加入允许列表。
* `commands.bashForegroundMs` 控制 bash 在切换到后台前等待的时间。在某个 bash 任务运行期间，新的 `! <cmd>` 请求会被拒绝（一次只允许一个任务）。
* `commands.config: true` 启用 `/config`（读写 `openclaw.json`）。
* `channels.<provider>.configWrites` 用于控制由该渠道发起的配置变更（默认：true）。适用于 `/config set|unset` 以及提供方特定的自动迁移（如 Telegram 超级群组 ID 变更、Slack 频道 ID 变更）。
* `commands.debug: true` 启用 `/debug`（仅运行时覆盖）。
* `commands.restart: true` 启用 `/restart` 和 Gateway 工具的重启操作。
* `commands.useAccessGroups: false` 允许指令绕过访问组的允许列表/策略。
* 斜杠指令和指令式指示仅对**已授权发送者**生效。授权来自于渠道允许列表/配对以及 `commands.useAccessGroups` 的组合。

<div id="web-whatsapp-web-channel-runtime">
  ### `web`（WhatsApp Web 渠道运行时）
</div>

WhatsApp 通过 Gateway 的 Web 渠道（Baileys Web）运行。当存在关联的会话时会自动启动。
将 `web.enabled` 设置为 `false`，以在默认情况下将其禁用。

```json5
{
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0
    }
  }
}
```

<div id="channelstelegram-bot-transport">
  ### `channels.telegram`（机器人传输）
</div>

只有在存在 `channels.telegram` 配置节时，OpenClaw 才会启动 Telegram。机器人 token 从 `channels.telegram.botToken`（或 `channels.telegram.tokenFile`）解析，对默认账号则回退为使用 `TELEGRAM_BOT_TOKEN`。
将 `channels.telegram.enabled: false` 设为 false 以禁用自动启动。
多账号支持位于 `channels.telegram.accounts` 下（参见上面的多账号部分）。环境变量中的 token 只适用于默认账号。
将 `channels.telegram.configWrites: false` 设为 false 以阻止由 Telegram 发起的配置写入（包括超级群组 ID 迁移以及 `/config set|unset`）。

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing",                 // pairing | allowlist | open | disabled
      allowFrom: ["tg:123456789"],         // optional; "open" requires ["*"]
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Keep answers brief.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Stay on topic."
            }
          }
        }
      },
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" }
      ],
      historyLimit: 50,                     // include last N group messages as context (0 disables)
      replyToMode: "first",                 // off | first | all
      linkPreview: true,                   // toggle outbound link previews
      streamMode: "partial",               // off | partial | block (草稿流式传输;与块流式传输分开)
      draftChunk: {                        // optional; only for streamMode=block
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph"       // paragraph | newline | sentence
      },
      actions: { reactions: true, sendMessage: true }, // tool action gates (false disables)
      reactionNotifications: "own",   // off | own | all
      mediaMaxMb: 5,
      retry: {                             // outbound retry policy
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1
      },
      network: {                           // transport overrides
        autoSelectFamily: false
      },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook"
    }
  }
}
```

草稿流式传输说明：

* 使用 Telegram `sendMessageDraft`（草稿气泡，而非真实消息）。
* 需要**私聊话题**（私信中的 `message_thread_id`；bot 已启用话题功能）。
* `/reasoning stream` 会将推理内容以流式方式写入草稿，然后发送最终答案。
  重试策略的默认值和行为详见[重试策略](/zh/concepts/retry)。

<div id="channelsdiscord-bot-transport">
  ### `channels.discord`（机器人传输通道）
</div>

通过设置机器人 token 和可选的门控策略来配置 Discord 机器人：
多账号支持位于 `channels.discord.accounts` 下（参见上面的多账号章节）。环境变量中的 token 仅适用于默认账号。

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8,                          // clamp inbound media size
      allowBots: false,                       // allow bot-authored messages
      actions: {                              // tool action gates (false disables)
        reactions: true,
        stickers: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        voiceStatus: true,
        events: true,
        moderation: false
      },
      replyToMode: "off",                     // off | first | all
      dm: {
        enabled: true,                        // disable all DMs when false
        policy: "pairing",                    // pairing | allowlist | open | disabled
        allowFrom: ["1234567890", "steipete"], // optional DM allowlist ("open" requires ["*"])
        groupEnabled: false,                 // enable group DMs
        groupChannels: ["openclaw-dm"]          // optional group DM allowlist
      },
      guilds: {
        "123456789012345678": {               // guild id (preferred) or slug
          slug: "friends-of-openclaw",
          requireMention: false,              // per-guild default
          reactionNotifications: "own",       // off | own | all | allowlist
          users: ["987654321098765432"],      // optional per-guild user allowlist
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Short answers only."
            }
          }
        }
      },
      historyLimit: 20,                       // include last N guild messages as context
      textChunkLimit: 2000,                   // optional outbound text chunk size (chars)
      chunkMode: "length",                    // optional chunking mode (length | newline)
      maxLinesPerMessage: 17,                 // 每条消息的软性最大行数(Discord UI 裁剪)
      retry: {                                // outbound retry policy
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1
      }
    }
  }
}
```

只有在存在 `channels.discord` 配置节时，OpenClaw 才会启动 Discord。token 从 `channels.discord.token` 解析，默认账号会回退使用 `DISCORD_BOT_TOKEN`（除非 `channels.discord.enabled` 为 `false`）。在为 cron/CLI 命令指定投递目标时，使用 `user:<id>`（私信 DM）或 `channel:<id>`（服务器频道）；裸数字 ID 含义不明确，会被拒绝。
Guild slug 为小写，空格替换为 `-`；channel 键使用 slug 化后的频道名称（不带前缀 `#`）。优先使用 guild id 作为键，以避免重命名带来的歧义。
机器人发送的消息默认会被忽略。通过 `channels.discord.allowBots` 启用（自身消息仍会被过滤，以防止自回复循环）。
Reaction 通知模式：

* `off`：不处理任何 reaction 事件。
* `own`：仅处理针对机器人自身消息的 reaction（默认）。
* `all`：处理所有消息上的所有 reaction。
* `allowlist`：处理来自 `guilds.<id>.users` 的 reaction，作用于所有消息（列表为空则禁用）。
  出站文本会按 `channels.discord.textChunkLimit`（默认 2000）进行分片。将 `channels.discord.chunkMode="newline"` 设置为按空行（段落边界）优先分割，然后再按长度分片。Discord 客户端可能会截断行数很多的超长消息，因此 `channels.discord.maxLinesPerMessage`（默认 17）会对多行的长回复进行拆分，即便其长度未达到 2000 字符。
  重试策略的默认值和行为记录在 [Retry policy](/zh/concepts/retry) 中。

<div id="channelsgooglechat-chat-api-webhook">
  ### `channels.googlechat`（Chat API webhook）
</div>

Google Chat 通过带有应用级身份验证（service account）的 HTTP webhook 工作。
多账户支持位于 `channels.googlechat.accounts`（参见上面的多账户章节）。环境变量仅适用于默认账户。

```json5
{
  channels: {
    "googlechat": {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url",             // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890",        // optional; improves mention detection
      dm: {
        enabled: true,
        policy: "pairing",                // pairing（配对）| allowlist（允许列表）| open（开放，允许所有用户）| disabled（禁用）
        allowFrom: ["users/1234567890"]   // optional; "open" requires ["*"]
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true }
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20
    }
  }
}
```

说明：

* 服务账号 JSON 可以以内联方式提供（`serviceAccount`），也可以通过文件提供（`serviceAccountFile`）。
* 默认账号的环境变量兜底：`GOOGLE_CHAT_SERVICE_ACCOUNT` 或 `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`。
* `audienceType` + `audience` 必须与 Chat 应用的 webhook 身份验证配置一致。
* 设置投递目标时，使用 `spaces/&lt;spaceId&gt;` 或 `users/&lt;userId|email&gt;`。

<div id="channelsslack-socket-mode">
  ### `channels.slack`（Socket 模式）
</div>

Slack 以 Socket 模式运行，需要同时提供 bot token 和 app token：

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["U123", "U456", "*"], // optional; "open" requires ["*"]
        groupEnabled: false,
        groupChannels: ["G123"]
      },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "Short answers only."
        }
      },
      historyLimit: 50,          // 将最后 N 条频道/群组消息作为上下文包含(0 禁用)
      allowBots: false,
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["U123"],
      replyToMode: "off",           // off | first | all
      thread: {
        historyScope: "thread",     // thread | channel
        inheritParent: false
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true
      },
      textChunkLimit: 4000,
      chunkMode: "length",
      mediaMaxMb: 20
    }
  }
}
```

多账号支持位于 `channels.slack.accounts` 下（参见上面的多账号部分）。环境变量中的 token 只适用于默认账号。

当提供方已启用且两个 token 都已设置（通过配置或 `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN`）时，OpenClaw 会启动 Slack。为 cron/CLI 命令指定投递目标时，使用 `user:<id>`（私信 DM）或 `channel:<id>`。
将 `channels.slack.configWrites` 设为 `false` 可阻止由 Slack 发起的配置写入（包括频道 ID 迁移和 `/config set|unset`）。

默认会忽略由机器人发送的消息。可通过 `channels.slack.allowBots` 或 `channels.slack.channels.<id>.allowBots` 启用。

表情回应（reaction）通知模式：

* `off`：不处理任何表情回应事件。
* `own`：仅处理机器人自己消息上的表情回应（默认）。
* `all`：处理所有消息上的所有表情回应。
* `allowlist`：仅处理 `channels.slack.reactionAllowlist` 中实体对所有消息的表情回应（列表为空则禁用）。

线程会话隔离：

* `channels.slack.thread.historyScope` 控制线程历史是按线程隔离（`thread`，默认）还是在整个频道中共享（`channel`）。
* `channels.slack.thread.inheritParent` 控制新线程会话是否继承父频道的对话记录（默认：false）。

Slack 动作分组（为 `slack` 工具动作加访问控制）：

| Action group | Default | Notes          |
| ------------ | ------- | -------------- |
| reactions    | enabled | 添加/列出表情回应      |
| messages     | enabled | 读取/发送/编辑/删除消息  |
| pins         | enabled | 固定/取消固定/列出置顶消息 |
| memberInfo   | enabled | 成员信息           |
| emojiList    | enabled | 自定义表情列表        |

<div id="channelsmattermost-bot-token">
  ### `channels.mattermost`（机器人 Token）
</div>

Mattermost 以插件形式提供，不随核心安装一起分发。
请先安装它：`openclaw plugins install @openclaw/mattermost`（或者在 git checkout 环境中使用 `./extensions/mattermost`）。

Mattermost 需要一个机器人 Token，以及你的服务器的基础 URL（base URL）：

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      textChunkLimit: 4000,
      chunkMode: "length"
    }
  }
}
```

当账户已配置好（bot token + base URL）并启用时，OpenClaw 会启动 Mattermost。token + base URL 会从 `channels.mattermost.botToken` + `channels.mattermost.baseUrl` 或默认账户的 `MATTERMOST_BOT_TOKEN` + `MATTERMOST_URL` 解析得到（除非 `channels.mattermost.enabled` 为 `false`）。

聊天模式：

* `oncall`（默认）：仅在被 @ 提及时才响应频道消息。
* `onmessage`：对每一条频道消息都进行响应。
* `onchar`：当消息以触发前缀开头时响应（`channels.mattermost.oncharPrefixes`，默认值为 `[">", "!"]`）。

访问控制：

* 默认私信：`channels.mattermost.dmPolicy="pairing"`（未知发送者会收到一个配对码）。
* 公开私信：`channels.mattermost.dmPolicy="open"` 加上 `channels.mattermost.allowFrom=["*"]`（`open` 表示允许任意用户发消息，不做限制）。
* 群组：`channels.mattermost.groupPolicy="allowlist"` 为默认值（基于 @ 提及进行控制）。使用 `channels.mattermost.groupAllowFrom` 来限定允许的发送者。

多账户支持位于 `channels.mattermost.accounts` 下（参见上面的多账户章节）。环境变量只适用于默认账户。
在指定投递目标时，使用 `channel:<id>` 或 `user:<id>`（或 `@username`）；裸 id 会被视为频道 id。

<div id="channelssignal-signal-cli">
  ### `channels.signal` (signal-cli)
</div>

Signal 消息回应可以触发系统事件（复用通用 reaction 工具链）：

```json5
{
  channels: {
    signal: {
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50 // 将最后 N 条群组消息包含为上下文(0 表示禁用)
    }
  }
}
```

反应通知模式：

* `off`：不发送任何反应事件。
* `own`：仅针对机器人自己消息上的反应（默认）。
* `all`：所有消息上的所有反应。
* `allowlist`：仅转发 `channels.signal.reactionAllowlist` 中允许来源对所有消息所做的反应（列表为空则禁用）。

<div id="channelsimessage-imsg-cli">
  ### `channels.imessage`（imsg CLI）
</div>

OpenClaw 会启动 `imsg rpc`（经由 stdio 的 JSON-RPC）。无需守护进程或端口。

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host", // 使用 SSH 包装器时,通过 SCP 传输远程附件
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50,    // 将最后 N 条群组消息包含为上下文(0 表示禁用)
      includeAttachments: false,
      mediaMaxMb: 16,
      service: "auto",
      region: "US"
    }
  }
}
```

多账户支持位于 `channels.imessage.accounts` 下（参见上方的多账户配置部分）。

注意：

* 需要对 Messages 数据库的完全磁盘访问权限（Full Disk Access）。
* 第一次发送时会弹出 Messages 自动化权限请求。
* 优先使用 `chat_id:<id>` 作为目标。使用 `imsg chats --limit 20` 列出会话。
* `channels.imessage.cliPath` 可以指向一个封装脚本（例如通过 `ssh` 到另一台运行 `imsg rpc` 的 Mac）；使用 SSH 密钥以避免密码提示。
* 对于远程 SSH 封装脚本，在启用 `includeAttachments` 时，将 `channels.imessage.remoteHost` 设置为通过 SCP 获取附件。

封装脚本示例：

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

<div id="agentsdefaultsworkspace">
  ### `agents.defaults.workspace`
</div>

配置智能体执行文件操作时使用的**唯一全局工作区目录**。

默认值：`~/.openclaw/workspace`。

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } }
}
```

如果启用了 `agents.defaults.sandbox`，非主会话可以在 `agents.defaults.sandbox.workspaceRoot` 下，为各自的 scope 使用专属工作区来覆盖该设置。

<div id="agentsdefaultsreporoot">
  ### `agents.defaults.repoRoot`
</div>

可选的仓库根目录，用于在系统提示中的 Runtime 行显示。若未设置，OpenClaw
会从工作区（以及当前工作目录）向上遍历父目录，尝试检测 `.git` 目录。该路径必须实际存在才会被使用。

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } }
}
```

<div id="agentsdefaultsskipbootstrap">
  ### `agents.defaults.skipBootstrap`
</div>

禁用自动创建工作区引导文件（`AGENTS.md`、`SOUL.md`、`TOOLS.md`、`IDENTITY.md`、`USER.md` 和 `BOOTSTRAP.md`）。

适用于已预置内容的部署场景，当你的工作区文件来自代码仓库时使用此选项。

```json5
{
  agents: { defaults: { skipBootstrap: true } }
}
```

<div id="agentsdefaultsbootstrapmaxchars">
  ### `agents.defaults.bootstrapMaxChars`
</div>

在截断之前，注入到系统提示中的每个工作区启动引导文件的最大字符数。默认值：`20000`。

当某个文件超过该上限时，OpenClaw 会记录一条警告日志，并仅注入带有标记的截断头部和尾部内容。

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } }
}
```

<div id="agentsdefaultsusertimezone">
  ### `agents.defaults.userTimezone`
</div>

为**系统提示词上下文**设置用户的时区（不影响消息封套中的时间戳）。如果未设置，OpenClaw 会在运行时使用宿主机的时区。

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } }
}
```

<div id="agentsdefaultstimeformat">
  ### `agents.defaults.timeFormat`
</div>

控制系统提示中“当前日期与时间”部分所显示的**时间格式**。
默认值：`auto`（遵循操作系统偏好设置）。

```json5
{
  agents: { defaults: { timeFormat: "auto" } } // auto | 12 | 24
}
```

<div id="messages">
  ### `messages`
</div>

控制入站/出站前缀以及可选的确认（ack）响应。
有关消息排队、会话和流式处理上下文，参见 [消息](/zh/concepts/messages)。

```json5
{
  messages: {
    responsePrefix: "🦞", // 或者 "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions",
    removeAckAfterReply: false
  }
}
```

`responsePrefix` 会应用到跨所有渠道的**所有外发回复**（工具摘要、分块
流式输出、最终回复），除非回复中已经包含该前缀。

如果未设置 `messages.responsePrefix`，默认不会应用任何前缀。WhatsApp 自聊
回复是个例外：当已设置 identity 时，默认使用 `[{identity.name}]`，否则使用
`[openclaw]`，以便同一手机上的对话保持清晰可读。
将其设置为 `"auto"` 时，会为被路由的智能体（在已设置时）自动推导并使用 `[{identity.name}]`。

<div id="template-variables">
  #### 模板变量
</div>

`responsePrefix` 字符串可以包含会被动态解析的模板变量：

| Variable          | Description | Example                     |
| ----------------- | ----------- | --------------------------- |
| `{model}`         | 模型短名称       | `claude-opus-4-5`, `gpt-4o` |
| `{modelFull}`     | 模型完整标识符     | `anthropic/claude-opus-4-5` |
| `{provider}`      | 提供方名称       | `anthropic`, `openai`       |
| `{thinkingLevel}` | 当前思考级别      | `high`, `low`, `off`        |
| `{identity.name}` | Agent 身份名称  | (same as `"auto"` 模式)       |

变量不区分大小写（`{MODEL}` = `{model}`）。`{think}` 是 `{thinkingLevel}` 的别名。
未解析的变量会保留为原样的字面文本。

```json5
{
  messages: {
    responsePrefix: "[{model} | think:{thinkingLevel}]"
  }
}
```

示例输出：`[claude-opus-4-5 | think:high] Here's my response...`

WhatsApp 入站消息前缀通过 `channels.whatsapp.messagePrefix` 配置（已弃用：
`messages.messagePrefix`）。默认值保持**不变**：当 `channels.whatsapp.allowFrom`
为空时为 `"[openclaw]"`，否则为 `""`（无前缀）。当使用 `"[openclaw]"` 时，如果路由到的
智能体设置了 `identity.name`，OpenClaw 会改用 `[{identity.name}]`。

`ackReaction` 会在支持表情回应的渠道（Slack/Discord/Telegram/Google Chat）上尽最大努力发送一个表情回应，用于确认接收到入站消息。默认值为当前活动
Agent 代理的 `identity.emoji`（如果已设置），否则为 `"👀"`。将其设为 `""` 可禁用该功能。

`ackReactionScope` 控制何时触发回应：

* `group-mentions`（默认）：仅在群组/房间要求提及 **且** 机器人被提及时
* `group-all`：所有群组/房间消息
* `direct`：仅私聊消息
* `all`：所有消息

`removeAckAfterReply` 会在发送回复后移除机器人的确认表情回应
（仅限 Slack/Discord/Telegram/Google Chat）。默认值：`false`。

<div id="messagestts">
  #### `messages.tts`
</div>

为发出的回复启用文本转语音功能。启用后，OpenClaw 会使用 ElevenLabs 或 OpenAI
生成音频，并将其附加到回复中。Telegram 使用 Opus 语音消息；其他渠道则发送 MP3 音频。

```json5
{
  messages: {
    tts: {
      auto: "always", // off | always | inbound | tagged
      mode: "final", // final | all (include tool/block replies)
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true
      },
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0
        }
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy"
      }
    }
  }
}
```

Notes:

* `messages.tts.auto` 控制自动 TTS（`off`、`always`、`inbound`、`tagged`）。
* `/tts off|always|inbound|tagged` 设置每个会话的自动模式（优先于配置中的设置）。
* `messages.tts.enabled` 是旧字段；`doctor` 会将其迁移到 `messages.tts.auto`。
* `prefsPath` 存储本地覆写设置（提供方/限制/摘要）。
* `maxTextLength` 是 TTS 输入的硬性上限；摘要会被截断以适配该限制。
* `summaryModel` 在自动摘要时覆写 `agents.defaults.model.primary`。
  * 接受 `provider/model` 或来自 `agents.defaults.models` 的别名。
* `modelOverrides` 启用基于模型的覆写，例如 `[[tts:...]]` 标签（默认开启）。
* `/tts limit` 和 `/tts summary` 控制每个用户的摘要设置。
* `apiKey` 的值会回退为使用 `ELEVENLABS_API_KEY`/`XI_API_KEY` 和 `OPENAI_API_KEY`。
* `elevenlabs.baseUrl` 覆写 ElevenLabs API 的基础 URL。
* `elevenlabs.voiceSettings` 支持 `stability`/`similarityBoost`/`style`（0..1），
  `useSpeakerBoost`，以及 `speed`（0.5..2.0）。

<div id="talk">
  ### `talk`
</div>

Talk 模式的默认设置（macOS/iOS/Android）。语音 ID 在未设置时会回退到 `ELEVENLABS_VOICE_ID` 或 `SAG_VOICE_ID`。
`apiKey` 在未设置时会回退到 `ELEVENLABS_API_KEY`（或 Gateway 的 shell 配置环境）。
`voiceAliases` 允许在 Talk 指令中使用更友好的名称（例如 `"voice":"Clawd"`）。

```json5
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    voiceAliases: {
      Clawd: "EXAVITQu4vr4xnSDxMaL",
      Roger: "CwhRBWXzGAHq8TQ4Fs17"
    },
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true
  }
}
```

<div id="agentsdefaults">
  ### `agents.defaults`
</div>

控制嵌入式智能体运行时（模型/思考过程/详细程度/超时）。
`agents.defaults.models` 定义已配置的模型目录（并且充当 `/model` 的允许列表）。
`agents.defaults.model.primary` 设置默认模型；`agents.defaults.model.fallbacks` 是全局回退模型。
`agents.defaults.imageModel` 是可选项，**仅当 primary 模型不支持图像输入时才会使用**。
每个 `agents.defaults.models` 条目可以包含：

* `alias`（可选的模型快捷命令，例如 `/opus`）。
* `params`（可选的、传递给模型请求的提供方特定 API 参数）。

`params` 同样会应用于流式执行（嵌入式智能体 + 压缩）。当前支持的键：`temperature`、`maxTokens`。这些会与调用时的选项合并；以调用方提供的值为准。`temperature` 是一个高级调节项——除非你了解模型的默认值并确实需要调整，否则请保持未设置。

示例：

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-sonnet-4-5-20250929": {
          params: { temperature: 0.6 }
        },
        "openai/gpt-5.2": {
          params: { maxTokens: 8192 }
        }
      }
    }
  }
}
```

Z.AI GLM-4.x 模型会自动启用 thinking 模式，除非你：

* 设置 `--thinking off`，或者
* 自行定义 `agents.defaults.models["zai/<model>"].params.thinking`。

OpenClaw 也内置了一些别名简写。只有当该模型已经存在于 `agents.defaults.models` 中时，才会应用默认值：

* `opus` -&gt; `anthropic/claude-opus-4-5`
* `sonnet` -&gt; `anthropic/claude-sonnet-4-5`
* `gpt` -&gt; `openai/gpt-5.2`
* `gpt-mini` -&gt; `openai/gpt-5-mini`
* `gemini` -&gt; `google/gemini-3-pro-preview`
* `gemini-flash` -&gt; `google/gemini-3-flash-preview`

如果你自行配置了相同的别名名称（不区分大小写），则以你的配置为准（默认值绝不会覆盖你显式的设置）。

示例：以 Opus 4.5 为主模型，MiniMax M2.1 为回退模型（使用 MiniMax 官方托管服务）：

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" }
      },
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: ["minimax/MiniMax-M2.1"]
      }
    }
  }
}
```

MiniMax 认证：设置环境变量 `MINIMAX_API_KEY`，或配置 `models.providers.minimax`。

<div id="agentsdefaultsclibackends-cli-fallback">
  #### `agents.defaults.cliBackends` (CLI 回退)
</div>

用于纯文本回退执行（无工具调用）的可选 CLI 后端。当 API 提供方失败时，它们可作为
备用路径使用。配置一个接受文件路径的 `imageArg` 后，支持图像透传。

注意：

* CLI 后端是**文本优先**的；工具始终被禁用。
* 当设置了 `sessionArg` 时，会话受支持；会话 ID 会按后端分别持久化。
* 对于 `claude-cli`，默认值已内置。如果 PATH 很精简/受限（launchd/systemd），可覆盖命令路径。

示例：

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude"
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          modelArg: "--model",
          sessionArg: "--session",
          sessionMode: "existing",
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat"
        }
      }
    }
  }
}
```

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "anthropic/claude-sonnet-4-1": { alias: "Sonnet" },
        "openrouter/deepseek/deepseek-r1:free": {},
        "zai/glm-4.7": {
          alias: "GLM",
          params: {
            thinking: {
              type: "enabled",
              clear_thinking: false
            }
          }
        }
      },
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: [
          "openrouter/deepseek/deepseek-r1:free",
          "openrouter/meta-llama/llama-3.3-70b-instruct:free"
        ]
      },
      imageModel: {
        primary: "openrouter/qwen/qwen-2.5-vl-72b-instruct:free",
        fallbacks: [
          "openrouter/google/gemini-2.0-flash-vision:free"
        ]
      },
      thinkingDefault: "low",
      verboseDefault: "off",
      elevatedDefault: "on",
      timeoutSeconds: 600,
      mediaMaxMb: 5,
      heartbeat: {
        every: "30m",
        target: "last"
      },
      maxConcurrent: 3,
      subagents: {
        model: "minimax/MiniMax-M2.1",
        maxConcurrent: 1,
        archiveAfterMinutes: 60
      },
      exec: {
        backgroundMs: 10000,
        timeoutSec: 1800,
        cleanupMs: 1800000
      },
      contextTokens: 200000
    }
  }
}
```

<div id="agentsdefaultscontextpruning-tool-result-pruning">
  #### `agents.defaults.contextPruning`（工具结果裁剪）
</div>

`agents.defaults.contextPruning` 会在请求发送给 LLM 之前，从内存上下文中裁剪**旧的工具结果**。
它**不会**修改磁盘上的会话历史（`*.jsonl` 仍然是完整的）。

此选项用于减少健谈型智能体在长时间累积大量工具输出时的 token 使用量。

整体行为概览：

* 绝不会修改 user/assistant 消息。
* 保护最近的 `keepLastAssistants` 条 assistant 消息（该位置之后的工具结果不会被裁剪）。
* 保护引导前缀（第一条 user 消息之前的内容不会被裁剪）。
* 模式：
  * `adaptive`：当估算的上下文占比超过 `softTrimRatio` 时，对超大的工具结果做“软裁剪”（保留头/尾）。
    然后当估算的上下文占比超过 `hardClearRatio` **且** 可裁剪工具结果的总体量达到 `minPrunableToolChars` 时，
    对最旧的可裁剪工具结果执行“硬清除”。
  * `aggressive`：始终把截断点之前的可裁剪工具结果替换为 `hardClear.placeholder`（不做占比检查）。

软裁剪 vs 硬清除（对发送给 LLM 的上下文有哪些变化）：

* **软裁剪（soft-trim）**：仅针对*超大*工具结果。保留开头与结尾，在中间插入 `...`。
  * 之前：`toolResult("…very long output…")`
  * 之后：`toolResult("HEAD…\n...\n…TAIL\n\n[Tool result trimmed: …]")`
* **硬清除（hard-clear）**：用占位符替换整个工具结果。
  * 之前：`toolResult("…very long output…")`
  * 之后：`toolResult("[Old tool result content cleared]")`

注意 / 当前限制：

* 目前包含**图像块的工具结果会被跳过**（当前阶段永不裁剪/清除）。
* 估算的“上下文占比”基于**字符数**（近似值），而不是精确 token。
* 如果会话中尚不足 `keepLastAssistants` 条 assistant 消息，将跳过裁剪。
* 在 `aggressive` 模式下，会忽略 `hardClear.enabled`（符合条件的工具结果始终会被替换成 `hardClear.placeholder`）。

默认（adaptive）模式：

```json5
{
  agents: { defaults: { contextPruning: { mode: "adaptive" } } }
}
```

若要禁用：

```json5
{
  agents: { defaults: { contextPruning: { mode: "off" } } }
}
```

默认值（当将 `mode` 设置为 `"adaptive"` 或 `"aggressive"` 时）：

* `keepLastAssistants`: `3`
* `softTrimRatio`: `0.3`（仅在 `"adaptive"` 模式下）
* `hardClearRatio`: `0.5`（仅在 `"adaptive"` 模式下）
* `minPrunableToolChars`: `50000`（仅在 `"adaptive"` 模式下）
* `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }`（仅在 `"adaptive"` 模式下）
* `hardClear`: `{ enabled: true, placeholder: "[Old tool result content cleared]" }`

示例（`"aggressive"` 模式，精简配置）：

```json5
{
  agents: { defaults: { contextPruning: { mode: "aggressive" } } }
}
```

示例（自适应调优）：

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "adaptive",
        keepLastAssistants: 3,
        softTrimRatio: 0.3,
        hardClearRatio: 0.5,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { enabled: true, placeholder: "[Old tool result content cleared]" },
        // 可选:限制对特定工具的修剪(deny 优先;支持 "*" 通配符)
        tools: { deny: ["browser", "canvas"] },
      }
    }
  }
}
```

有关具体行为说明，请参阅 [/concepts/session-pruning](/zh/concepts/session-pruning)。

<div id="agentsdefaultscompaction-reserve-headroom-memory-flush">
  #### `agents.defaults.compaction`（预留余量 + 记忆刷新）
</div>

`agents.defaults.compaction.mode` 用于选择压缩摘要策略。默认值为 `default`；将其设置为 `safeguard` 可对超长历史启用分块摘要。参见 [/concepts/compaction](/zh/concepts/compaction)。

`agents.defaults.compaction.reserveTokensFloor` 为 Pi 压缩强制设定 `reserveTokens`
的最小值（默认：`20000`）。将其设置为 `0` 可关闭该下限。

`agents.defaults.compaction.memoryFlush` 会在自动压缩前运行一次**静默**的智能体轮次，
指示模型将持久记忆写入磁盘（例如 `memory/YYYY-MM-DD.md`）。当会话的 token 估算值
跨过位于压缩上限之下的软阈值时触发。

历史默认值：

* `memoryFlush.enabled`：`true`
* `memoryFlush.softThresholdTokens`：`4000`
* `memoryFlush.prompt` / `memoryFlush.systemPrompt`：带有 `NO_REPLY` 的内置默认值
* 注意：当会话工作区为只读时，会跳过 memory flush
  （`agents.defaults.sandbox.workspaceAccess: "ro"` 或 `"none"`）。

示例（已调优）：

```json5
{
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard",
        reserveTokensFloor: 24000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 6000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "将任何持久笔记写入 memory/YYYY-MM-DD.md;如果没有内容需要存储,请回复 NO_REPLY。"
        }
      }
    }
  }
}
```

块式流式输出（Block streaming）：

* `agents.defaults.blockStreamingDefault`: `"on"`/`"off"`（默认 off）。
* 渠道级覆盖：通过 `*.blockStreaming`（以及按账号的变体）强制开启/关闭块式流式输出。
  非 Telegram 渠道必须显式设置 `*.blockStreaming: true` 才会启用块式回复。
* `agents.defaults.blockStreamingBreak`: `"text_end"` 或 `"message_end"`（默认：text&#95;end）。
* `agents.defaults.blockStreamingChunk`: 块式流式输出的软分块配置。默认
  800–1200 个字符，优先在段落分隔符（`\n\n`）处切分，其次是换行，再其次是句子边界。
  示例：
  ```json5
  {
    agents: { defaults: { blockStreamingChunk: { minChars: 800, maxChars: 1200 } } }
  }
  ```
* `agents.defaults.blockStreamingCoalesce`: 在发送前合并已流式输出的块。
  默认值为 `{ idleMs: 1000 }`，并从 `blockStreamingChunk` 继承 `minChars`，
  同时将 `maxChars` 限制在渠道文本长度上限以内。Signal/Slack/Discord/Google Chat 默认
  使用 `minChars: 1500`，除非显式覆盖。
  渠道级覆盖：`channels.whatsapp.blockStreamingCoalesce`, `channels.telegram.blockStreamingCoalesce`,
  `channels.discord.blockStreamingCoalesce`, `channels.slack.blockStreamingCoalesce`, `channels.mattermost.blockStreamingCoalesce`,
  `channels.signal.blockStreamingCoalesce`, `channels.imessage.blockStreamingCoalesce`, `channels.msteams.blockStreamingCoalesce`,
  `channels.googlechat.blockStreamingCoalesce`
  （以及按账号的变体）。
* `agents.defaults.humanDelay`: 第一条块式回复之后，各块之间的随机停顿。
  模式：`off`（默认）、`natural`（800–2500ms）、`custom`（使用 `minMs`/`maxMs`）。
  按智能体覆盖配置：`agents.list[].humanDelay`。
  示例：
  ```json5
  {
    agents: { defaults: { humanDelay: { mode: "natural" } } }
  }
  ```

有关行为与分块的详细信息，参见 [/concepts/streaming](/zh/concepts/streaming)。

正在输入指示（Typing indicators）：

* `agents.defaults.typingMode`: `"never" | "instant" | "thinking" | "message"`。默认：
  直接对话 / 被提及时为 `instant`，未被提及的群聊为 `message`。
* `session.typingMode`: 会话级模式覆盖。
* `agents.defaults.typingIntervalSeconds`: 输入指示信号刷新频率（默认：6 秒）。
* `session.typingIntervalSeconds`: 会话级刷新间隔覆盖。
  有关行为细节，参见 [/concepts/typing-indicators](/zh/concepts/typing-indicators)。

`agents.defaults.model.primary` 应设置为 `provider/model`（例如 `anthropic/claude-opus-4-5`）。
别名来自 `agents.defaults.models.*.alias`（例如 `Opus`）。
如果你省略 provider，OpenClaw 当前会暂时假定为 `anthropic`，作为弃用过渡期的回退行为。
Z.AI 模型可通过 `zai/<model>` 使用（例如 `zai/glm-4.7`），并且需要在环境中设置
`ZAI_API_KEY`（或旧版的 `Z_AI_API_KEY`）。

`agents.defaults.heartbeat` 用于配置周期性心跳运行：

* `every`：持续时间字符串（`ms`、`s`、`m`、`h`）；默认单位为分钟。默认值：
  `30m`。将其设为 `0m` 可禁用。
* `model`：用于心跳运行的可选模型覆盖值（`provider/model`）。
* `includeReasoning`：当为 `true` 时，心跳在可用时还会发送单独的 `Reasoning:` 消息（结构与 `/reasoning on` 相同）。默认值：`false`。
* `session`：可选的会话键，用于控制心跳在哪个会话中运行。默认值：`main`。
* `to`：可选的接收方覆盖（按渠道的 id，例如 WhatsApp 的 E.164、Telegram 的 chat id）。
* `target`：可选的投递渠道（`last`、`whatsapp`、`telegram`、`discord`、`slack`、`msteams`、`signal`、`imessage`、`none`）。默认值：`last`。
* `prompt`：用于心跳正文的可选覆盖（默认值：`Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`）。覆盖内容会被原样发送；如果你仍希望读取该文件，请在其中包含一行 `Read HEARTBEAT.md`。
* `ackMaxChars`：投递前在 `HEARTBEAT_OK` 之后允许的最大字符数（默认：300）。

按智能体粒度的心跳配置：

* 设置 `agents.list[].heartbeat`，即可为特定智能体启用或覆盖心跳设置。
* 如果任一智能体条目定义了 `heartbeat`，则**只有这些智能体**会运行心跳；默认值
  会成为这些智能体共享的基线配置。

心跳会运行完整的智能体轮次。更短的间隔会消耗更多 token；请注意
`every` 设置，尽量保持 `HEARTBEAT.md` 体积很小，和/或选择更便宜的 `model`。

`tools.exec` 用于配置后台执行的默认配置：

* `backgroundMs`：自动切入后台前的时间（毫秒，默认 10000）
* `timeoutSec`：运行超过该时间后自动终止（秒，默认 1800）
* `cleanupMs`：在内存中保留已完成会话的时长（毫秒，默认 1800000）
* `notifyOnExit`：当后台执行退出时，入队一个系统事件并请求一次心跳（默认 true）
* `applyPatch.enabled`：启用实验性的 `apply_patch`（仅限 OpenAI/OpenAI Codex；默认 false）
* `applyPatch.allowModels`：可选的模型 id 允许列表（例如 `gpt-5.2` 或 `openai/gpt-5.2`）
  注意：`applyPatch` 仅位于 `tools.exec` 下。

`tools.web` 用于配置网页搜索和抓取工具：

* `tools.web.search.enabled`（默认：当该键存在时为 true）
* `tools.web.search.apiKey`（推荐：通过 `openclaw configure --section web` 设置，或使用环境变量 `BRAVE_API_KEY`）
* `tools.web.search.maxResults`（1–10，默认 5）
* `tools.web.search.timeoutSeconds`（默认 30）
* `tools.web.search.cacheTtlMinutes`（默认 15）
* `tools.web.fetch.enabled`（默认 true）
* `tools.web.fetch.maxChars`（默认 50000）
* `tools.web.fetch.timeoutSeconds`（默认 30）
* `tools.web.fetch.cacheTtlMinutes`（默认 15）
* `tools.web.fetch.userAgent`（可选覆盖值）
* `tools.web.fetch.readability`（默认 true；禁用后仅使用基础 HTML 清理）
* `tools.web.fetch.firecrawl.enabled`（当设置了 API key 时默认 true）
* `tools.web.fetch.firecrawl.apiKey`（可选；默认使用 `FIRECRAWL_API_KEY`）
* `tools.web.fetch.firecrawl.baseUrl`（默认 https://api.firecrawl.dev）
* `tools.web.fetch.firecrawl.onlyMainContent`（默认 true）
* `tools.web.fetch.firecrawl.maxAgeMs`（可选）
* `tools.web.fetch.firecrawl.timeoutSeconds`（可选）

`tools.media` 用于配置入站媒体理解（图像/音频/视频）：

* `tools.media.models`：共享模型列表（带能力标签；在每能力专用列表之后使用）。
* `tools.media.concurrency`：最大并发能力执行数（默认 2）。
* `tools.media.image` / `tools.media.audio` / `tools.media.video`：
  * `enabled`：选择退出开关（当已配置模型时默认为 true）。
  * `prompt`：可选的 prompt 覆盖（image/video 会自动追加 `maxChars` 提示）。
  * `maxChars`：最大输出字符数（image/video 默认 500；audio 未设置上限）。
  * `maxBytes`：待发送媒体的最大大小（默认：image 10MB、audio 20MB、video 50MB）。
  * `timeoutSeconds`：请求超时时间（默认：image 60s、audio 60s、video 120s）。
  * `language`：可选的音频语言提示。
  * `attachments`：附件策略（`mode`、`maxAttachments`、`prefer`）。
  * `scope`：可选的门控（首个匹配生效），支持 `match.channel`、`match.chatType` 或 `match.keyPrefix`。
  * `models`：模型条目的有序列表；失败或媒体超限会回退到下一个条目。
* 每个 `models[]` 条目：
  * 提供方条目（`type: "provider"` 或省略）：
    * `provider`：API 提供方 id（`openai`、`anthropic`、`google`/`gemini`、`groq` 等）。
    * `model`：模型 id 覆写（image 必填；audio 提供方默认 `gpt-4o-mini-transcribe`/`whisper-large-v3-turbo`，video 默认 `gemini-3-flash-preview`）。
    * `profile` / `preferredProfile`：认证 profile 选择。
  * CLI 条目（`type: "cli"`）：
    * `command`：要运行的可执行文件。
    * `args`：模板化参数（支持 `{{MediaPath}}`、`{{Prompt}}`、`{{MaxChars}}` 等）。
  * `capabilities`：可选列表（`image`、`audio`、`video`），用于给共享条目加门控。省略时的默认值：`openai`/`anthropic`/`minimax` → image，`google` → image+audio+video，`groq` → audio。
  * `prompt`、`maxChars`、`maxBytes`、`timeoutSeconds`、`language` 可以按条目单独覆盖。

如果未配置任何模型（或 `enabled: false`），将跳过理解过程；模型仍然会收到原始附件。

提供方认证遵循标准模型认证顺序（认证 profile 列表、环境变量如 `OPENAI_API_KEY`/`GROQ_API_KEY`/`GEMINI_API_KEY`，或 `models.providers.*.apiKey`）。

示例：

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        scope: {
          default: "deny",
          rules: [{ action: "allow", match: { chatType: "direct" } }]
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          { type: "cli", command: "whisper", args: ["--model", "base", "{{MediaPath}}"] }
        ]
      },
      video: {
        enabled: true,
        maxBytes: 52428800,
        models: [{ provider: "google", model: "gemini-3-flash-preview" }]
      }
    }
  }
}
```

`agents.defaults.subagents` 用于配置子智能体的默认设置：

* `model`：新建子智能体的默认模型（字符串或 `{ primary, fallbacks }`）。如果省略，子智能体会继承调用方的模型，除非在具体智能体或单次调用上另行覆盖。
* `maxConcurrent`：子智能体最大并发运行数（默认 1）
* `archiveAfterMinutes`：在 N 分钟后自动归档子智能体会话（默认 60；设置为 `0` 可禁用）
* 每个子智能体的工具策略：`tools.subagents.tools.allow` / `tools.subagents.tools.deny`（`deny` 优先生效）

`tools.profile` 在 `tools.allow`/`tools.deny` 之前设置一个**基础工具允许列表**：

* `minimal`：仅 `session_status`
* `coding`：`group:fs`、`group:runtime`、`group:sessions`、`group:memory`、`image`
* `messaging`：`group:messaging`、`sessions_list`、`sessions_history`、`sessions_send`、`session_status`
* `full`：无限制（与未设置相同）

按智能体覆写：`agents.list[].tools.profile`。

示例（默认仅允许消息类工具，同时额外允许 Slack 与 Discord 工具）：

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"]
  }
}
```

示例（用于编程的配置方案，但在所有场景禁用 exec/process）：

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"]
  }
}
```

`tools.byProvider` 允许你为特定提供方（或单个 `provider/model`）**进一步限制**可用工具。
按智能体级别覆盖：`agents.list[].tools.byProvider`。

生效顺序：基础配置 → 提供方配置 → 允许/拒绝策略。
提供方键既可以是 `provider`（例如 `google-antigravity`），也可以是 `provider/model`
（例如 `openai/gpt-5.2`）。

示例（保持全局 coding 配置，但对 Google Antigravity 仅启用最少量的工具）：

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" }
    }
  }
}
```

示例（针对特定提供方/模型的允许列表）：

```json5
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] }
    }
  }
}
```

`tools.allow` / `tools.deny` 用于配置全局工具的允许/拒绝策略（拒绝优先）。
匹配不区分大小写，并支持 `*` 通配符（`"*"` 表示所有工具）。
即使 Docker 沙箱处于 **off** 状态时，该策略也会生效。

示例（全局禁用 browser/canvas）：

```json5
{
  tools: { deny: ["browser", "canvas"] }
}
```

工具分组（简写）在 **全局** 和 **每个智能体** 的工具策略中都适用：

* `group:runtime`: `exec`, `bash`, `process`
* `group:fs`: `read`, `write`, `edit`, `apply_patch`
* `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory`: `memory_search`, `memory_get`
* `group:web`: `web_search`, `web_fetch`
* `group:ui`: `browser`, `canvas`
* `group:automation`: `cron`, `gateway`
* `group:messaging`: `message`
* `group:nodes`: `nodes`
* `group:openclaw`: 所有内置的 OpenClaw 工具（不包括提供方插件）

`tools.elevated` 控制提升（宿主机）执行权限的访问：

* `enabled`: 允许提权模式（默认 true）
* `allowFrom`: 按通道配置的允许列表（为空 = 禁用）
  * `whatsapp`: E.164 号码
  * `telegram`: chat ID 或用户名
  * `discord`: user ID 或用户名（如果省略，则回退到 `channels.discord.dm.allowFrom`）
  * `signal`: E.164 号码
  * `imessage`: 句柄或聊天 ID
  * `webchat`: 会话 ID 或用户名

示例：

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15555550123"],
        discord: ["steipete", "1234567890123"]
      }
    }
  }
}
```

按智能体覆写（进一步收紧）：

```json5
{
  agents: {
    list: [
      {
        id: "family",
        tools: {
          elevated: { enabled: false }
        }
      }
    ]
  }
}
```

注意：

* `tools.elevated` 是全局基准配置。`agents.list[].tools.elevated` 只能在此基础上进一步收紧（两者都允许时才视为允许）。
* `/elevated on|off|ask|full` 会按会话键存储状态；行内指令仅对单条消息生效。
* 提升权限的 `exec` 在宿主机上运行，并绕过沙箱。
* 工具策略依然生效；如果 `exec` 被拒绝，则无法以提升权限方式使用。

`agents.defaults.maxConcurrent` 用于设置可在多个会话间并行执行的嵌入式智能体运行实例的最大数量。每个会话本身仍然是串行的（同一会话键一次仅允许一个运行）。默认值：1。

<div id="agentsdefaultssandbox">
  ### `agents.defaults.sandbox`
</div>

为嵌入式智能体提供可选的 **Docker 沙箱**。主要面向非主会话使用，
以防止它们访问你的宿主系统。

详情参见：[Sandboxing](/zh/gateway/sandboxing)

默认值（如果启用）：

* scope: `"agent"`（每个智能体一个容器 + 一个工作区）
* 基于 Debian bookworm-slim 的镜像
* 智能体工作区访问级别：`workspaceAccess: "none"`（默认）
  * `"none"`：在 `~/.openclaw/sandboxes` 下，为每个 scope 使用一个独立的沙箱工作区
* `"ro"`：将沙箱工作区保留在 `/workspace`，并以只读方式把智能体工作区挂载到 `/agent`（禁用 `write`/`edit`/`apply_patch`）
  * `"rw"`：以读写方式将智能体工作区挂载到 `/workspace`
* 自动清理：空闲时间 &gt; 24h 或 存在时间 &gt; 7d
* 工具策略：仅允许 `exec`, `process`, `read`, `write`, `edit`, `apply_patch`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`（拒绝优先）
  * 通过 `tools.sandbox.tools` 配置，可在每个智能体下通过 `agents.list[].tools.sandbox.tools` 覆盖
  * 沙箱策略中支持工具分组简写：`group:runtime`, `group:fs`, `group:sessions`, `group:memory`（参见 [Sandbox vs Tool Policy vs Elevated](/zh/gateway/sandbox-vs-tool-policy-vs-elevated#tool-groups-shorthands)）
* 可选沙箱化浏览器（Chromium + CDP，noVNC 观察器）
* 加固选项：`network`, `user`, `pidsLimit`, `memory`, `cpus`, `ulimits`, `seccompProfile`, `apparmorProfile`

警告：`scope: "shared"` 表示共享容器和共享工作区。没有跨会话隔离。
使用 `scope: "session"` 以获得按会话隔离。

历史用法：仍然支持 `perSession`（`true` → `scope: "session"`，
`false` → `scope: "shared"`）。

`setupCommand` 在容器创建后（在容器内通过 `sh -lc`）**仅运行一次**。
对于软件包安装，请确保允许网络出站访问、根文件系统可写，并且使用 root 用户。

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent is default)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          containerPrefix: "openclaw-sbx-",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          // 每个智能体覆盖配置(多智能体): agents.list[].sandbox.docker.*
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
          binds: ["/var/run/docker.sock:/var/run/docker.sock", "/home/user/source:/source:rw"]
        },
        browser: {
          enabled: false,
          image: "openclaw-sandbox-browser:bookworm-slim",
          containerPrefix: "openclaw-sbx-browser-",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          allowedControlUrls: ["http://10.0.0.42:18791"],
          allowedControlHosts: ["browser.lab.local", "10.0.0.42"],
          allowedControlPorts: [18791],
          autoStart: true,
          autoStartTimeoutMs: 12000
        },
        prune: {
          idleHours: 24,  // 0 disables idle pruning
          maxAgeDays: 7   // 0 disables max-age pruning
        }
      }
    }
  },
  tools: {
    sandbox: {
      tools: {
        allow: ["exec", "process", "read", "write", "edit", "apply_patch", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"]
      }
    }
  }
}
```

使用以下命令构建一次默认的沙箱镜像：

```bash
scripts/sandbox-setup.sh
```

注意：沙箱容器默认使用 `network: "none"`；如果智能体需要出站访问，请将 `agents.defaults.sandbox.docker.network` 设置为 `"bridge"`（或你的自定义网络）。

注意：入站附件会被暂存到当前活动工作区的 `media/inbound/*` 目录中。在 `workspaceAccess: "rw"` 的情况下，这意味着文件会被写入智能体工作区。

注意：`docker.binds` 会挂载额外的宿主机目录；全局和每个智能体级别的绑定会被合并。

使用以下命令构建可选的浏览器镜像：

```bash
scripts/sandbox-browser-setup.sh
```

当 `agents.defaults.sandbox.browser.enabled=true` 时，浏览器工具会使用沙箱化的
Chromium 实例（CDP）。如果启用了 noVNC（在 headless=false 时为默认值），
noVNC URL 会被注入到系统提示词中，方便智能体引用。
这不需要在主配置中启用 `browser.enabled`；沙箱控制
URL 会在每个会话中注入。

`agents.defaults.sandbox.browser.allowHostControl`（默认值：false）允许
沙箱会话通过浏览器工具（`target: "host"`）显式地访问**主机**浏览器控制服务器。
如果你希望严格的沙箱隔离，请保持该选项关闭。

远程控制的允许列表：

* `allowedControlUrls`：允许用于 `target: "custom"` 的精确控制 URL。
* `allowedControlHosts`：允许的主机名（仅主机名，不含端口）。
* `allowedControlPorts`：允许的端口（默认值：http=80，https=443）。
  默认情况：所有允许列表均未设置（不做限制）。`allowHostControl` 默认为 false。

<div id="models-custom-providers-base-urls">
  ### `models`（自定义提供方 + 基础 URL）
</div>

OpenClaw 使用 **pi-coding-agent** 模型目录。你可以通过编写
`~/.openclaw/agents/<agentId>/agent/models.json`，或在你的 OpenClaw 配置中的
`models.providers` 下定义相同的 schema，来添加自定义提供方
（LiteLLM、本地 OpenAI 兼容服务器、Anthropic 代理等）。
按提供方划分的概览与示例：[/concepts/model-providers](/zh/concepts/model-providers)。

当存在 `models.providers` 时，OpenClaw 会在启动时将一个 `models.json` 写入/合并到
`~/.openclaw/agents/<agentId>/agent/` 中：

* 默认行为：**merge**（保留已有提供方，按名称覆盖）
* 将 `models.mode: "replace"` 设置为覆盖文件内容

通过 `agents.defaults.model.primary`（provider/model）选择模型。

```json5
{
  agents: {
    defaults: {
      model: { primary: "custom-proxy/llama-3.1-8b" },
      models: {
        "custom-proxy/llama-3.1-8b": {}
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions",
        models: [
          {
            id: "llama-3.1-8b",
            name: "Llama 3.1 8B", // 模型显示名称
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 32000
          }
        ]
      }
    }
  }
}
```

<div id="opencode-zen-multi-model-proxy">
  ### OpenCode Zen（多模型代理）
</div>

OpenCode Zen 是一个为每个模型提供独立端点的多模型 Gateway。OpenClaw 使用
来自 pi-ai 的内置 `opencode` 提供方；在 https://opencode.ai/auth
获取并设置 `OPENCODE_API_KEY`（或 `OPENCODE_ZEN_API_KEY`）。

注意：

* 模型引用使用 `opencode/<modelId>`（示例：`opencode/claude-opus-4-5`）。
* 如果你通过 `agents.defaults.models` 启用了允许列表，需要把计划使用的每个模型都添加进去。
* 快捷方式：`openclaw onboard --auth-choice opencode-zen`。

```json5
{
  agents: {
    defaults: {
      model: { primary: "opencode/claude-opus-4-5" },
      models: { "opencode/claude-opus-4-5": { alias: "Opus" } }
    }
  }
}
```

<div id="zai-glm-47-provider-alias-support">
  ### Z.AI (GLM-4.7) — 提供方别名支持
</div>

Z.AI 模型可通过内置的 `zai` 提供方访问。请在运行环境中设置 `ZAI_API_KEY`，
并以 提供方/模型 的形式引用该模型。

快捷命令：`openclaw onboard --auth-choice zai-api-key`。

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} }
    }
  }
}
```

备注：

* `z.ai/*` 和 `z-ai/*` 是可接受的别名，并会被规范化为 `zai/*`。
* 如果缺少 `ZAI_API_KEY`，对 `zai/*` 的请求将在运行时因认证错误而失败。
* 示例错误：`No API key found for provider "zai".`
* Z.AI 的通用 API 端点为 `https://api.z.ai/api/paas/v4`。GLM 编码
  请求使用专用的 Coding 端点 `https://api.z.ai/api/coding/paas/v4`。
  内置的 `zai` 提供方使用的是该 Coding 端点。如果你需要通用
  端点，请在 `models.providers` 中定义一个自定义提供方，并通过覆盖 base URL
  来实现（参见上面的自定义提供方部分）。
* 在文档/配置中使用伪造的占位符值；不要提交任何真实的 API key。

<div id="moonshot-ai-kimi">
  ### Moonshot AI（Kimi）
</div>

使用 Moonshot 的兼容 OpenAI 的接口：

```json5
{
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: { "moonshot/kimi-k2.5": { alias: "Kimi K2.5" } }
    }
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

注意事项：

* 在环境变量中设置 `MOONSHOT_API_KEY`，或使用 `openclaw onboard --auth-choice moonshot-api-key`。
* 模型参考：`moonshot/kimi-k2.5`。
* 如果你需要中国区的接口地址，请使用 `https://api.moonshot.cn/v1`。

<div id="kimi-code">
  ### Kimi Code
</div>

使用 Kimi Code 专用的、兼容 OpenAI 的端点（与 Moonshot 分开）：

```json5
{
  env: { KIMICODE_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-code/kimi-for-coding" },
      models: { "kimi-code/kimi-for-coding": { alias: "Kimi Code" } }
    }
  },
  models: {
    mode: "merge",
    providers: {
      "kimi-code": {
        baseUrl: "https://api.kimi.com/coding/v1",
        apiKey: "${KIMICODE_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-for-coding",
            name: "Kimi For Coding",
            reasoning: true,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 262144,
            maxTokens: 32768,
            headers: { "User-Agent": "KimiCLI/0.77" },
            compat: { supportsDeveloperRole: false }
          }
        ]
      }
    }
  }
}
```

注意：

* 在环境变量中设置 `KIMICODE_API_KEY`，或使用 `openclaw onboard --auth-choice kimi-code-api-key`。
* 模型标识：`kimi-code/kimi-for-coding`。

<div id="synthetic-anthropic-compatible">
  ### Synthetic（Anthropic 兼容）
</div>

使用 Synthetic 的 Anthropic 兼容端点：

```json5
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.1": { alias: "MiniMax M2.1" } }
    }
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536
          }
        ]
      }
    }
  }
}
```

注意：

* 设置 `SYNTHETIC_API_KEY`，或使用 `openclaw onboard --auth-choice synthetic-api-key`。
* 模型引用：`synthetic/hf:MiniMaxAI/MiniMax-M2.1`。
* 基础 URL 中应省略 `/v1`，因为 Anthropic 客户端会自动追加该路径。

<div id="local-models-lm-studio-recommended-setup">
  ### 本地模型（LM Studio）— 推荐配置
</div>

请参阅 [/gateway/local-models](/zh/gateway/local-models) 获取当前本地模型使用指南。简而言之：在性能足够强的硬件上通过 LM Studio Responses API 运行 MiniMax M2.1；同时保留托管模型的合并配置，用作回退方案。

<div id="minimax-m21">
  ### MiniMax M2.1
</div>

无需借助 LM Studio，直接使用 MiniMax M2.1：

```json5
{
  agent: {
    model: { primary: "minimax/MiniMax-M2.1" },
    models: {
      "anthropic/claude-opus-4-5": { alias: "Opus" },
      "minimax/MiniMax-M2.1": { alias: "Minimax" }
    }
  },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            // 定价:如需精确的成本跟踪,请在 models.json 中更新。
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

Notes:

* 设置 `MINIMAX_API_KEY` 环境变量，或使用 `openclaw onboard --auth-choice minimax-api`。
* 可用模型：`MiniMax-M2.1`（默认）。
* 如果需要精确的成本统计，请在 `models.json` 中更新定价。

<div id="cerebras-glm-46-47">
  ### Cerebras（GLM 4.6 / 4.7）
</div>

通过其 OpenAI 兼容端点使用 Cerebras：

```json5
{
  env: { CEREBRAS_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: {
        primary: "cerebras/zai-glm-4.7",
        fallbacks: ["cerebras/zai-glm-4.6"]
      },
      models: {
        "cerebras/zai-glm-4.7": { alias: "GLM 4.7 (Cerebras)" },
        "cerebras/zai-glm-4.6": { alias: "GLM 4.6 (Cerebras)" }
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      cerebras: {
        baseUrl: "https://api.cerebras.ai/v1",
        apiKey: "${CEREBRAS_API_KEY}",
        api: "openai-completions",
        models: [
          { id: "zai-glm-4.7", name: "GLM 4.7 (Cerebras)" },
          { id: "zai-glm-4.6", name: "GLM 4.6 (Cerebras)" }
        ]
      }
    }
  }
}
```

备注：

* 对于 Cerebras 使用 `cerebras/zai-glm-4.7`；对于 Z.AI 直连使用 `zai/glm-4.7`。
* 在环境变量或配置中设置 `CEREBRAS_API_KEY`。

备注：

* 支持的 API：`openai-completions`、`openai-responses`、`anthropic-messages`、`google-generative-ai`
* 对于自定义鉴权需求，使用 `authHeader: true` + `headers`。
* 如果希望将 `models.json` 存储到其他位置，可以通过设置 `OPENCLAW_AGENT_DIR`（或 `PI_CODING_AGENT_DIR`）来覆盖智能体配置根目录（默认：`~/.openclaw/agents/main/agent`）。

<div id="session">
  ### `session`
</div>

控制会话的作用域、重置策略、重置触发条件，以及会话存储的写入位置。

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main",
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"]
    },
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 60
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      dm: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 }
    },
    resetTriggers: ["/new", "/reset"],
    // 默认已按智能体独立存储在 ~/.openclaw/agents/<agentId>/sessions/sessions.json
    // 可以使用 {agentId} 模板覆盖:
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    // Direct chats collapse to agent:<agentId>:<mainKey> (default: "main").
    mainKey: "main",
    agentToAgent: {
      // Max ping-pong reply turns between requester/target (0–5).
      maxPingPongTurns: 5
    },
    sendPolicy: {
      rules: [
        { action: "deny", match: { channel: "discord", chatType: "group" } }
      ],
      default: "allow"
    }
  }
}
```

Fields:

* `mainKey`: 直接聊天（direct-chat）bucket 键（默认：`"main"`）。当你想在不更改 `agentId` 的情况下“重命名”主私信线程时很有用。
  * 沙箱说明：`agents.defaults.sandbox.mode: "non-main"` 使用该键来检测主会话。任何与 `mainKey` 不匹配的会话键（群组/频道）都会在沙箱中运行。
* `dmScope`: 私信会话如何分组（默认：`"main"`）。
  * `main`: 所有私信共享同一个主会话以保持连续性。
  * `per-peer`: 按发送方 id 在各个频道间隔离私信。
  * `per-channel-peer`: 按「频道 + 发送方」隔离私信（推荐用于多用户收件箱）。
  * `per-account-channel-peer`: 按「账号 + 频道 + 发送方」隔离私信（推荐用于多账号收件箱）。
* `identityLinks`: 将规范 id 映射到带提供方前缀的对端，使同一人在使用 `per-peer`、`per-channel-peer` 或 `per-account-channel-peer` 时可以在多个频道间共享同一个私信会话。
  * 示例：`alice: ["telegram:123456789", "discord:987654321012345678"]`。
* `reset`: 主重置策略。默认为每天凌晨 4:00（Gateway 主机本地时间）重置。
  * `mode`: `daily` 或 `idle`（当存在 `reset` 时默认是 `daily`）。
  * `atHour`: 每日重置边界的本地小时（0–23）。
  * `idleMinutes`: 滑动空闲时间窗口（分钟）。当同时配置了 daily 和 idle 时，先到期的优先生效。
* `resetByType`: 针对 `dm`、`group` 和 `thread` 的按会话类型的重置策略覆盖配置。
  * 如果你只设置了传统的 `session.idleMinutes` 而没有任何 `reset`/`resetByType`，OpenClaw 会保持仅按空闲时间重置的模式以兼容旧行为。
* `heartbeatIdleMinutes`: 用于心跳检查的可选空闲超时覆盖（启用时，每日重置仍然生效）。
* `agentToAgent.maxPingPongTurns`: 请求方/目标方之间最多来回回复的轮数（0–5，默认 5）。
* `sendPolicy.default`: 当没有规则匹配时的 `allow` 或 `deny` 回退策略。
* `sendPolicy.rules[]`: 按 `channel`、`chatType`（`direct|group|room`）或 `keyPrefix`（例如 `cron:`）进行匹配。遇到的第一个 deny 生效；否则允许。

<div id="skills-skills-config">
  ### `skills`（技能配置）
</div>

控制内置技能的允许列表、安装偏好、额外技能目录以及按技能的覆盖配置。作用于**内置**技能和 `~/.openclaw/skills`（在名称冲突时仍以工作区技能为准）。

字段：

* `allowBundled`：仅针对**内置**技能的可选允许列表。如果设置，只有这些
  内置技能会被视为可用（托管/工作区技能不受影响）。
* `load.extraDirs`：要扫描的额外技能目录（优先级最低）。
* `install.preferBrew`：在可用时优先使用 brew 安装工具（默认：true）。
* `install.nodeManager`：Node.js 包管理器偏好（`npm` | `pnpm` | `yarn`，默认：npm）。
* `entries.<skillKey>`：按技能的配置覆盖项。

每个技能可用字段：

* `enabled`：设为 `false` 可禁用某个技能，即使它已被内置/安装。
* `env`：为智能体运行注入的环境变量（仅在尚未设置时注入）。
* `apiKey`：为声明了主环境变量的技能提供的可选便捷字段（例如 `nano-banana-pro` → `GEMINI_API_KEY`）。

示例：

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: [
        "~/Projects/agent-scripts/skills",
        "~/Projects/oss/some-skill-pack/skills"
      ]
    },
    install: {
      preferBrew: true,
      nodeManager: "npm"
    },
    entries: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE"
        }
      },
      peekaboo: { enabled: true },
      sag: { enabled: false }
    }
  }
}
```

<div id="plugins-extensions">
  ### `plugins`（扩展）
</div>

控制插件的发现、允许/拒绝策略以及每个插件的配置。插件会从
`~/.openclaw/extensions`、`<workspace>/.openclaw/extensions`，以及任何
`plugins.load.paths` 条目中加载。**更改配置后需要重启 Gateway。**
完整用法参见 [/plugin](/zh/plugin)。

字段：

* `enabled`：插件加载的总开关（默认：true）。
* `allow`：可选的插件 ID 允许列表；如果设置，则只加载列出的插件。
* `deny`：可选的插件 ID 拒绝列表（拒绝优先生效）。
* `load.paths`：要额外加载的插件文件或目录（绝对路径或 `~`）。
* `entries.<pluginId>`：针对单个插件的覆盖配置项。
  * `enabled`：设为 `false` 以禁用。
  * `config`：插件特定的配置对象（如果提供，由插件自行校验）。

示例：

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    load: {
      paths: ["~/Projects/oss/voice-call-extension"]
    },
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio"
        }
      }
    }
  }
}
```

<div id="browser-openclaw-managed-browser">
  ### `browser`（openclaw 管理的浏览器）
</div>

OpenClaw 可以为 openclaw 启动一个**专用且隔离**的 Chrome/Brave/Edge/Chromium 实例，并暴露一个小型环回控制服务。
通过 `profiles.<name>.cdpUrl`，配置文件可以指向一个**远程**的基于 Chromium 的浏览器。远程
配置文件是仅附加模式（attach-only），不支持启动/停止/重置。

`browser.cdpUrl` 仍然保留，用于旧版的单配置文件（single-profile）配置，以及作为仅设置了 `cdpPort` 的配置文件的基础
scheme/host。

默认值：

* enabled：`true`
* evaluateEnabled：`true`（设为 `false` 以禁用 `act:evaluate` 和 `wait --fn`）
* 控制服务：仅限环回（端口基于 `gateway.port` 推导，默认 `18791`）
* CDP URL：`http://127.0.0.1:18792`（控制服务端口 + 1，旧版单配置文件）
* 配置文件颜色：`#FF4500`（龙虾橙）
* 注意：控制服务器由正在运行的 Gateway 启动（通过 OpenClaw.app 菜单栏，或 `openclaw gateway`）。
* 自动检测顺序：如果默认浏览器是基于 Chromium，则优先使用默认浏览器；否则按 Chrome → Brave → Edge → Chromium → Chrome Canary 的顺序依次尝试。

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: true,
    // cdpUrl: "http://127.0.0.1:18792", // legacy single-profile override
    defaultProfile: "chrome",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" }
    },
    color: "#FF4500",
    // Advanced:
    // headless: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    // attachOnly: false, // 通过隧道将远程 CDP 连接到 localhost 时设为 true
  }
}
```

<div id="ui-appearance">
  ### `ui` (外观)
</div>

原生应用在 UI 装饰区域中使用的可选强调色（例如 Talk Mode 气泡的色调）。

如果未设置，客户端将回退为一种柔和的浅蓝色。

```json5
{
  ui: {
    seamColor: "#FF4500", // hex (RRGGBB or #RRGGBB)
    // 可选:Control UI 助手身份覆盖。
    // 如果未设置,Control UI 使用活动智能体身份(配置或 IDENTITY.md)。
    assistant: {
      name: "OpenClaw",
      avatar: "CB" // emoji, short text, or image URL/data URI
    }
  }
}
```

<div id="gateway-gateway-server-mode-bind">
  ### `gateway`（Gateway 服务器模式与绑定）
</div>

使用 `gateway.mode` 显式声明本机是否应运行 Gateway。

默认值：

* mode：**未设置**（视为“不自动启动”）
* bind：`loopback`
* port：`18789`（WS 与 HTTP 共用单一端口）

```json5
{
  gateway: {
    mode: "local", // or "remote"
    port: 18789, // WS + HTTP multiplex
    bind: "loopback",
    // controlUi: { enabled: true, basePath: "/openclaw" }
    // auth: { mode: "token", token: "your-token" } // token 控制 WS + Control UI 访问
    // tailscale: { mode: "off" | "serve" | "funnel" }
  }
}
```

Control UI 基础路径：

* `gateway.controlUi.basePath` 用于设置提供 Control UI 的 URL 前缀。
* 示例：`"/ui"`、`"/openclaw"`、`"/apps/openclaw"`。
* 默认值：根路径（`/`）（不变）。
* `gateway.controlUi.allowInsecureAuth` 允许在省略设备身份时（通常通过 HTTP）对 Control UI 使用仅令牌认证。默认值：`false`。优先使用 HTTPS（Tailscale Serve）或 `127.0.0.1`。
* `gateway.controlUi.dangerouslyDisableDeviceAuth` 会为 Control UI 禁用设备身份校验（仅令牌/密码）。默认值：`false`。仅在紧急破例场景下使用。

相关文档：

* [Control UI](/zh/web/control-ui)
* [Web overview](/zh/web)
* [Tailscale](/zh/gateway/tailscale)
* [Remote access](/zh/gateway/remote)

受信任代理：

* `gateway.trustedProxies`：在 Gateway 前终止 TLS 的反向代理 IP 列表。
* 当连接来自这些 IP 之一时，OpenClaw 会使用 `x-forwarded-for`（或 `x-real-ip`）来确定用于本地配对检查和 HTTP 认证/本地检查的客户端 IP。
* 只应列出你完全控制的代理，并确保它们会**覆盖**传入请求中的 `x-forwarded-for`。

注意事项：

* 若未将 `gateway.mode` 设置为 `local`（且未传入覆盖标志），`openclaw gateway` 将拒绝启动。
* `gateway.port` 控制 WS + HTTP（Control UI、hooks、A2UI）共用的单个复用端口。
* OpenAI Chat Completions 端点：**默认禁用**；通过 `gateway.http.endpoints.chatCompletions.enabled: true` 启用。
* 优先级：`--port` &gt; `OPENCLAW_GATEWAY_PORT` &gt; `gateway.port` &gt; 默认 `18789`。
* Gateway 认证默认开启（令牌/密码或 Tailscale Serve 身份）。非回环地址绑定需要共享令牌/密码。
* 引导向导默认会生成一个 gateway 令牌（即使在回环地址上）。
* `gateway.remote.token` **仅**用于远程 CLI 调用；它不会启用本地 gateway 认证。此时会忽略 `gateway.token`。

认证与 Tailscale：

* `gateway.auth.mode` 设置握手要求（`token` 或 `password`）。未设置时，默认为令牌认证。
* `gateway.auth.token` 存储用于令牌认证的共享令牌（由同一台机器上的 CLI 使用）。
* 当设置了 `gateway.auth.mode` 后，只接受该种方式（外加可选的 Tailscale 头）。
* `gateway.auth.password` 可以在此设置，或通过 `OPENCLAW_GATEWAY_PASSWORD` 设置（推荐）。
* `gateway.auth.allowTailscale` 允许使用 Tailscale Serve 身份头
  （`tailscale-user-login`）在请求通过回环地址并携带 `x-forwarded-for`、`x-forwarded-proto` 和 `x-forwarded-host` 时完成认证。OpenClaw
  会在接受前通过 `tailscale whois` 解析 `x-forwarded-for` 地址以验证身份。当为 `true` 时，Serve 请求不需要令牌/密码；设置为 `false` 可强制要求显式凭据。当 `tailscale.mode = "serve"` 且认证模式不是 `password` 时，默认值为 `true`。
* `gateway.tailscale.mode: "serve"` 使用 Tailscale Serve（仅限 tailnet，回环绑定）。
* `gateway.tailscale.mode: "funnel"` 将控制面板公开暴露；需要认证。
* `gateway.tailscale.resetOnExit` 会在关闭时重置 Serve/Funnel 配置。

远程客户端默认值（CLI）：

* 当 `gateway.mode = "remote"` 时，`gateway.remote.url` 为 CLI 调用设置默认的 Gateway WebSocket URL。
* `gateway.remote.transport` 用于选择 macOS 远程传输方式（默认 `ssh`，`direct` 表示使用 ws/wss）。当为 `direct` 时，`gateway.remote.url` 必须为 `ws://` 或 `wss://`。`ws://host` 默认端口为 `18789`。
* `gateway.remote.token` 提供远程调用使用的 token（不设置则表示不启用认证）。
* `gateway.remote.password` 提供远程调用使用的密码（不设置则表示不启用认证）。

macOS 应用行为：

* OpenClaw.app 监视 `~/.openclaw/openclaw.json`，当 `gateway.mode` 或 `gateway.remote.url` 发生变化时，会实时切换模式。
* 如果 `gateway.mode` 未设置但设置了 `gateway.remote.url`，macOS 应用会将其视为远程模式。
* 当你在 macOS 应用中更改连接模式时，它会将 `gateway.mode`（以及远程模式下的 `gateway.remote.url` 和 `gateway.remote.transport`）写回配置文件。

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      password: "your-password"
    }
  }
}
```

直连传输示例（macOS 应用）：

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      transport: "direct",
      url: "wss://gateway.example.ts.net",
      token: "your-token"
    }
  }
}
```

<div id="gatewayreload-config-hot-reload">
  ### `gateway.reload`（配置热更新）
</div>

Gateway 监控 `~/.openclaw/openclaw.json`（或 `OPENCLAW_CONFIG_PATH`），并自动应用变更。

模式：

* `hybrid`（默认）：对可安全热更新的更改直接应用；对关键更改则重启 Gateway。
* `hot`：仅应用可安全热更新的更改；在需要重启时写入日志。
* `restart`：在任意配置变更时重启 Gateway。
* `off`：禁用热更新。

```json5
{
  gateway: {
    reload: {
      mode: "hybrid",
      debounceMs: 300
    }
  }
}
```

<div id="hot-reload-matrix-files-impact">
  #### 热重载矩阵（文件及影响）
</div>

监控的文件：

* `~/.openclaw/openclaw.json`（或 `OPENCLAW_CONFIG_PATH`）

支持热更新（无需完全重启 Gateway）：

* `hooks`（webhook 认证/路径/映射）+ `hooks.gmail`（重启 Gmail 监听器）
* `browser`（重启浏览器控制服务）
* `cron`（重启 cron 服务并更新并发配置）
* `agents.defaults.heartbeat`（重启心跳运行器）
* `web`（重启 WhatsApp Web 渠道）
* `telegram`、`discord`、`signal`、`imessage`（重启通道）
* `agent`、`models`、`routing`、`messages`、`session`、`whatsapp`、`logging`、`skills`、`ui`、`talk`、`identity`、`wizard`（动态读取）

需要完全重启 Gateway：

* `gateway`（端口/绑定/认证/Control UI/Tailscale）
* `bridge`（遗留组件）
* `discovery`
* `canvasHost`
* `plugins`
* 任何未知/不受支持的配置路径（出于安全考虑，默认要求重启）

<div id="multi-instance-isolation">
  ### 多实例隔离
</div>

要在同一主机上运行多个 Gateway（用于冗余或应急 Bot），需要为每个实例隔离状态和配置，并使用唯一端口：

* `OPENCLAW_CONFIG_PATH`（每个实例的配置）
* `OPENCLAW_STATE_DIR`（会话/凭据）
* `agents.defaults.workspace`（记忆数据）
* `gateway.port`（每个实例唯一）

便捷参数（CLI）：

* `openclaw --dev …` → 使用 `~/.openclaw-dev`，并基于基础端口 `19001` 进行端口偏移
* `openclaw --profile <name> …` → 使用 `~/.openclaw-<name>`（端口由配置/环境变量/参数决定）

参见 [Gateway 运行手册](/zh/gateway) 了解推导得到的端口映射关系（gateway/browser/canvas）。
参见 [多 Gateway 部署](/zh/gateway/multiple-gateways) 了解浏览器/CDP 端口隔离的详细信息。

示例：

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

<div id="hooks-gateway-webhooks">
  ### `hooks` (Gateway webhooks)
</div>

在 Gateway 的 HTTP 服务器上启用一个简单的 HTTP webhook 端点。

默认配置：

* enabled: `false`
* path: `/hooks`
* maxBodyBytes: `262144`（256 KB）

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    presets: ["gmail"],
    transformsDir: "~/.openclaw/hooks",
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate:
          "From: {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}",
        deliver: true,
        channel: "last",
        model: "openai/gpt-5.2-mini",
      },
    ],
  }
}
```

请求必须包含 hook token：

* `Authorization: Bearer <token>` **或**
* `x-openclaw-token: <token>` **或**
* `?token=<token>`

端点：

* `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
* `POST /hooks/agent` → `{ message, name?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }`
* `POST /hooks/<name>` → 通过 `hooks.mappings` 解析

`/hooks/agent` 始终会向主会话发送一条摘要（并且可以通过 `wakeMode: "now"` 选择性地触发一次立即心跳）。

映射说明：

* `match.path` 匹配 `/hooks` 之后的子路径（例如 `/hooks/gmail` → `gmail`）。
* `match.source` 匹配载荷字段（例如 `{ source: "gmail" }`），这样你就可以使用通用的 `/hooks/ingest` 路径。
* 像 `{{messages[0].subject}}` 这样的模板会从载荷中读取数据。
* `transform` 可以指向一个返回 hook 动作的 JS/TS 模块。
* `deliver: true` 会将最终回复发送到某个通道；`channel` 默认为 `last`（默认回退到 WhatsApp）。
* 如果之前没有发送路由，请显式设置 `channel` 和 `to`（对 Telegram/Discord/Google Chat/Slack/Signal/iMessage/MS Teams 是必需的）。
* `model` 会在本次 hook 运行中覆盖要使用的 LLM（`provider/model` 或别名；如果设置了 `agents.defaults.models`，则必须在允许列表中）。

Gmail 辅助配置（由 `openclaw webhooks gmail setup` / `run` 使用）：

```json5
{
  hooks: {
    gmail: {
      account: "openclaw@gmail.com",
      topic: "projects/<project-id>/topics/gog-gmail-watch",
      subscription: "gog-gmail-watch-push",
      pushToken: "shared-push-token",
      hookUrl: "http://127.0.0.1:18789/hooks/gmail",
      includeBody: true,
      maxBytes: 20000,
      renewEveryMinutes: 720,
      serve: { bind: "127.0.0.1", port: 8788, path: "/" },
      tailscale: { mode: "funnel", path: "/gmail-pubsub" },

      // 可选:为 Gmail 钩子处理使用成本更低的模型
      // 在认证/速率限制/超时时回退到 agents.defaults.model.fallbacks,然后是主模型
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      // Optional: default thinking level for Gmail hooks
      thinking: "off",
    }
  }
}
```

Gmail 钩子模型覆盖配置：

* `hooks.gmail.model` 指定用于处理 Gmail 钩子的模型（默认为会话的主模型）。
* 接受来自 `agents.defaults.models` 的 `provider/model` 引用或别名。
* 在鉴权失败 / 触发限流 / 超时时，依次回退到 `agents.defaults.model.fallbacks`，然后是 `agents.defaults.model.primary`。
* 如果设置了 `agents.defaults.models`，需要在允许列表中包含该钩子使用的模型。
* 在启动时，如果配置的模型不在模型目录或允许列表中，会发出警告。
* `hooks.gmail.thinking` 设置 Gmail 钩子的默认思考级别，可被每个钩子单独配置的 `thinking` 覆盖。

Gateway 自动启动：

* 如果 `hooks.enabled=true` 且已设置 `hooks.gmail.account`，Gateway 会在启动时运行
  `gog gmail watch serve` 并自动续期该 watch。
* 将 `OPENCLAW_SKIP_GMAIL_WATCHER=1` 设置为 1 可禁用自动启动（用于手动运行）。
* 避免在 Gateway 之外单独运行 `gog gmail watch serve`；否则会因为
  `listen tcp 127.0.0.1:8788: bind: address already in use` 而失败。

注意：当 `tailscale.mode` 为 on 时，OpenClaw 会将 `serve.path` 默认设为 `/`，
以便 Tailscale 能正确代理 `/gmail-pubsub`（它会去掉设置的路径前缀）。
如果你需要后端接收到带前缀的路径，请将
`hooks.gmail.tailscale.target` 设置为完整 URL（并相应调整 `serve.path`）。

<div id="canvashost-lantailnet-canvas-file-server-live-reload">
  ### `canvasHost`（LAN/Tailnet Canvas 文件服务器 + 实时重载）
</div>

Gateway 会通过 HTTP 提供一个 HTML/CSS/JS 目录，这样 iOS/Android 节点只需要执行 `canvas.navigate` 即可访问它。

默认根目录：`~/.openclaw/workspace/canvas`
默认端口：`18793`（为避免与 openclaw 浏览器 CDP 端口 `18792` 冲突）
服务器会监听在 **Gateway 绑定主机**（LAN 或 Tailnet）上，以便节点可以访问。

该服务器会：

* 提供 `canvasHost.root` 下的文件
* 向返回的 HTML 注入一个极小的实时重载客户端
* 监视该目录，并通过位于 `/__openclaw__/ws` 的 WebSocket 端点广播重载事件
* 当目录为空时自动创建一个初始的 `index.html`（这样你能立刻看到一些内容）
* 还会在 `/__openclaw__/a2ui/` 提供 A2UI，并以 `canvasHostUrl` 的形式通告给节点
  （节点在 Canvas/A2UI 中始终使用它）

如果目录很大，或者你遇到 `EMFILE`，可以禁用实时重载（及文件监视）：

* 配置：`canvasHost: { liveReload: false }`

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    port: 18793,
    liveReload: true
  }
}
```

对 `canvasHost.*` 的更改需要重启 Gateway（重新加载配置会触发重启）。

禁用方法：

* 配置：`canvasHost: { enabled: false }`
* 环境变量：`OPENCLAW_SKIP_CANVAS_HOST=1`

<div id="bridge-legacy-tcp-bridge-removed">
  ### `bridge`（旧版 TCP 桥接，已移除）
</div>

当前构建不再包含 TCP 桥接监听器；`bridge.*` 配置键将被忽略。
节点通过 Gateway 的 WebSocket 进行连接。本节仅保留作历史参考。

旧版行为：

* Gateway 可以为节点（iOS/Android）暴露一个简单的 TCP 桥接服务，通常监听在端口 `18790`。

默认值：

* enabled: `true`
* port: `18790`
* bind: `lan`（绑定到 `0.0.0.0`）

绑定模式：

* `lan`: `0.0.0.0`（可通过任意网络接口访问，包括 LAN/Wi‑Fi 和 Tailscale）
* `tailnet`: 仅绑定到该机器的 Tailscale IP（推荐用于 Vienna ⇄ London）
* `loopback`: `127.0.0.1`（仅本地）
* `auto`: 如果存在 tailnet IP 则优先使用，否则为 `lan`

TLS：

* `bridge.tls.enabled`: 为桥接连接启用 TLS（启用后仅允许 TLS）。
* `bridge.tls.autoGenerate`: 在不存在证书/密钥时生成自签名证书（默认：true）。
* `bridge.tls.certPath` / `bridge.tls.keyPath`: 桥接证书和私钥的 PEM 路径。
* `bridge.tls.caPath`: 可选的 PEM CA 包（自定义根证书或后续可能使用的 mTLS）。

启用 TLS 后，Gateway 会在 discovery TXT 记录中通告 `bridgeTls=1` 和 `bridgeTlsSha256`，
以便节点对证书进行固定（pin）。如果尚未存储指纹，手动连接将使用首次信任（trust-on-first-use）策略。
自动生成证书要求在 PATH 中可以使用 `openssl`；如果生成失败，桥接将不会启动。

```json5
{
  bridge: {
    enabled: true,
    port: 18790,
    bind: "tailnet",
    tls: {
      enabled: true,
      // 省略时使用 ~/.openclaw/bridge/tls/bridge-{cert,key}.pem
      // certPath: "~/.openclaw/bridge/tls/bridge-cert.pem",
      // keyPath: "~/.openclaw/bridge/tls/bridge-key.pem"
    }
  }
}
```

<div id="discoverymdns-bonjour-mdns-broadcast-mode">
  ### `discovery.mdns`（Bonjour / mDNS 广播模式）
</div>

控制局域网内的 mDNS 发现广播（`_openclaw-gw._tcp`）。

* `minimal`（默认）：在 TXT 记录中省略 `cliPath` 和 `sshPort`
* `full`：在 TXT 记录中包含 `cliPath` 和 `sshPort`
* `off`：完全禁用 mDNS 广播
* 主机名：默认为 `openclaw`（广播为 `openclaw.local`）。可通过 `OPENCLAW_MDNS_HOSTNAME` 覆盖。

```json5
{
  discovery: { mdns: { mode: "minimal" } }
}
```

<div id="discoverywidearea-wide-area-bonjour-unicast-dnssd">
  ### `discovery.wideArea`（广域 Bonjour / 单播 DNS‑SD）
</div>

启用后，Gateway 会在 `~/.openclaw/dns/` 下，为 `_openclaw-gw._tcp` 写入一个单播 DNS-SD 区域，使用已配置的发现域名（例如：`openclaw.internal.`）。

要让 iOS/Android 能够跨网络进行发现（维也纳 ⇄ 伦敦），需要配合以下设置使用：

* 在 Gateway 主机上运行一个为你所选域名提供解析服务的 DNS 服务器（推荐使用 CoreDNS）
* 使用 Tailscale 的 **分离 DNS（split DNS）**，使客户端通过该 Gateway DNS 服务器解析该域名

一次性设置辅助命令（在 Gateway 主机上运行）：

```bash
openclaw dns setup --apply
```

```json5
{
  discovery: { wideArea: { enabled: true } }
}
```

## 模板变量

模板占位符会在 `tools.media.*.models[].args` 和 `tools.media.models[].args` 中展开（以及未来任何支持模板的参数字段）。

| Variable | Description |
|----------|-------------|
| `{{Body}}` | 完整的入站消息正文 |
| `{{RawBody}}` | 原始入站消息正文（无历史记录/发送方包装；最适合用于命令解析） |
| `{{BodyStripped}}` | 已剔除群组提及的正文（作为智能体的默认值时效果最佳） |
| `{{From}}` | 发送方标识符（WhatsApp 使用 E.164；不同渠道可能不同） |
| `{{To}}` | 目标标识符 |
| `{{MessageSid}}` | 渠道消息 ID（若可用） |
| `{{SessionId}}` | 当前会话 UUID |
| `{{IsNewSession}}` | 创建了新会话时为 `"true"` |
| `{{MediaUrl}}` | 入站媒体伪 URL（若存在） |
| `{{MediaPath}}` | 本地媒体路径（若已下载） |
| `{{MediaType}}` | 媒体类型（image/audio/document/…） |
| `{{Transcript}}` | 音频转录文本（启用时） |
| `{{Prompt}}` | 为 CLI 条目解析得到的媒体提示词 |
| `{{MaxChars}}` | 为 CLI 条目解析得到的最大输出字符数 |
| `{{ChatType}}` | `"direct"` 或 `"group"` |
| `{{GroupSubject}}` | 群组主题（尽可能获取） |
| `{{GroupMembers}}` | 群组成员预览（尽可能获取） |
| `{{SenderName}}` | 发送方显示名称（尽可能获取） |
| `{{SenderE164}}` | 发送方电话号码（尽可能获取） |
| `{{Provider}}` | 提供方提示（whatsapp|telegram|discord|googlechat|slack|signal|imessage|msteams|webchat|…） |

<div id="cron-gateway-scheduler">
  ## Cron（Gateway 调度器）
</div>

Cron 是 Gateway 自带的调度器，用于处理唤醒和定时任务。有关功能概览和 CLI 示例，请参阅 [Cron jobs](/zh/automation/cron-jobs)。

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2
  }
}
```

***

*下一步：[Agent 代理运行时](/zh/concepts/agent)* 🦞
