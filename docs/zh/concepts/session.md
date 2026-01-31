---
title: 会话
summary: "聊天会话的管理规则、键和持久化策略"
read_when:
  - 修改会话处理或存储
---

<div id="session-management">
  # 会话管理
</div>

OpenClaw 将**每个智能体的一对一会话**视为主会话。直接聊天会归并为 `agent:<agentId>:<mainKey>`（默认 `main`），而群组/频道聊天会拥有各自的键。`session.mainKey` 会被使用。

使用 `session.dmScope` 控制**私信**如何分组：

* `main`（默认）：所有私信共享主会话以保持连续性。
* `per-peer`：按发送方 ID 在各个渠道之间隔离。
* `per-channel-peer`：按频道 + 发送方隔离（推荐用于多用户收件箱）。
* `per-account-channel-peer`：按账号 + 频道 + 发送方隔离（推荐用于多账号收件箱）。

使用 `session.identityLinks` 将带提供方前缀的对端 ID 映射到一个规范化身份，这样在使用 `per-peer`、`per-channel-peer` 或 `per-account-channel-peer` 时，同一人可以在多个渠道间共享同一私信会话。

<div id="gateway-is-the-source-of-truth">
  ## Gateway 是权威数据源
</div>

所有会话状态都**由 Gateway 统一管理**（“主” OpenClaw 实例）。UI 客户端（macOS app、WebChat 等）必须向 Gateway 查询会话列表和 token 计数，而不是读取本地文件。

* 在**远程模式**下，你实际需要关心的会话存储位于远程 Gateway 主机上，而不是你的 Mac 上。
* UI 中显示的 token 计数来自 Gateway 的存储字段（`inputTokens`、`outputTokens`、`totalTokens`、`contextTokens`）。客户端不会去解析 JSONL 会话记录来“修正”总数。

<div id="where-state-lives">
  ## 状态存储位置
</div>

* 在 **Gateway 主机** 上：
  * 存储文件：`~/.openclaw/agents/<agentId>/sessions/sessions.json`（按智能体划分）。
* 对话记录：`~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`（Telegram 主题会话使用 `.../<SessionId>-topic-<threadId>.jsonl`）。
* 存储结构是一个映射 `sessionKey -> { sessionId, updatedAt, ... }`。删除这些条目是安全的；在需要时会自动重新创建。
* 群组条目可能包含 `displayName`、`channel`、`subject`、`room` 和 `space`，用于在 UI 中标记会话。
* 会话条目包含 `origin` 元数据（标签 + 路由提示），以便 UI 可以说明会话的来源。
* OpenClaw **不会** read 旧版 Pi/Tau 会话目录。

<div id="session-pruning">
  ## 会话修剪
</div>

OpenClaw 默认会在调用 LLM 之前，从内存中的上下文中清理掉**旧的工具结果**。
这**不会**修改 JSONL 历史记录。参见 [/concepts/session-pruning](/zh/concepts/session-pruning)。

<div id="pre-compaction-memory-flush">
  ## 预压缩内存刷新
</div>

当会话接近自动压缩时，OpenClaw 可以执行一次**静默内存刷新**
回合，用于提示模型将持久化笔记写入磁盘。此操作仅在工作区可写时运行。参见 [Memory](/zh/concepts/memory) 和
[Compaction](/zh/concepts/compaction)。

<div id="mapping-transports-session-keys">
  ## 传输通道 → 会话键映射
</div>

* 私聊遵循 `session.dmScope`（默认 `main`）。
  * `main`：`agent:<agentId>:<mainKey>`（在设备/渠道之间保持连续性）。
    * 多个电话号码和渠道可以映射到同一个 agent 主键；它们作为进入同一会话的传输通道。
  * `per-peer`：`agent:<agentId>:dm:<peerId>`。
  * `per-channel-peer`：`agent:<agentId>:<channel>:dm:<peerId>`。
  * `per-account-channel-peer`：`agent:<agentId>:<channel>:<accountId>:dm:<peerId>`（accountId 默认为 `default`）。
  * 如果 `session.identityLinks` 匹配带提供方前缀的 peer ID（例如 `telegram:123`），则使用规范键替换 `<peerId>`，使同一个人在多个渠道之间共享同一个会话。
* 群聊会隔离状态：`agent:<agentId>:<channel>:group:<id>`（房间/频道使用 `agent:<agentId>:<channel>:channel:<id>`）。
  * Telegram 论坛主题会在 group ID 后追加 `:topic:<threadId>` 以实现隔离。
  * 旧版的 `group:<id>` 键仍会被识别，以便迁移。
* 入站上下文仍可能使用 `group:<id>`；此时会根据 `Provider` 推断渠道，并规范化为标准形式 `agent:<agentId>:<channel>:group:<id>`。
* 其他来源：
  * Cron 任务：`cron:<job.id>`
  * Webhook：`hook:<uuid>`（除非由 hook 明确设置）
  * 节点执行：`node-<nodeId>`

<div id="lifecycle">
  ## 生命周期
</div>

* 重置策略：会话会被复用直到过期，过期会在下一条入站消息到达时进行评估。
* 每日重置：默认是 **Gateway 主机本地时间凌晨 4:00**。当某个会话的最后更新时间早于最近一次每日重置时间时，该会话即视为陈旧。
* 空闲重置（可选）：`idleMinutes` 会添加一个滑动空闲时间窗口。当同时配置了每日重置和空闲重置时，**先到期的那个**会强制创建新的会话。
* 传统仅空闲模式：如果你只设置了 `session.idleMinutes`，而没有任何 `session.reset`/`resetByType` 配置，OpenClaw 会保持在仅空闲重置模式，以保证向后兼容。
* 按类型覆盖（可选）：`resetByType` 允许你为 `dm`、`group` 和 `thread` 会话覆盖重置策略（thread = Slack/Discord 线程、Telegram 主题、Matrix 线程，前提是由连接器提供）。
* 按频道覆盖（可选）：`resetByChannel` 会覆盖某个频道的重置策略（应用于该频道的所有会话类型，并且优先于 `reset`/`resetByType`）。
* 重置触发器：精确匹配的 `/new` 或 `/reset`（加上 `resetTriggers` 中的任何额外触发词）会启动一个新的会话 id，并将剩余的消息内容继续传递下去。`/new <model>` 接受模型别名、`provider/model`，或提供方名称（模糊匹配）以设置新会话的模型。如果单独发送 `/new` 或 `/reset`，OpenClaw 会先运行一个简短的 “hello” 问候轮来确认重置。
* 手动重置：从存储中删除特定键，或删除 JSONL 转录文件；下一条消息会重新创建它们。
* 独立的 cron 任务在每次运行时都会生成一个新的 `sessionId`（不会复用空闲会话）。

<div id="send-policy-optional">
  ## 发送策略（可选）
</div>

为特定会话类型阻止消息投递，而无需单独列出每个 id。

```json5
{
  session: {
    sendPolicy: {
      rules: [
        { action: "deny", match: { channel: "discord", chatType: "group" } },
        { action: "deny", match: { keyPrefix: "cron:" } }
      ],
      default: "allow"
    }
  }
}
```

运行时覆盖（仅所有者）：

* `/send on` → 在此会话中启用
* `/send off` → 在此会话中禁用
* `/send inherit` → 清除覆盖并使用配置规则
  将这些命令作为独立消息发送，以便被正确识别。

<div id="configuration-optional-rename-example">
  ## 配置（可选的重命名示例）
</div>

```json5
// ~/.openclaw/openclaw.json
{
  session: {
    scope: "per-sender",      // 保持群组会话键独立
    dmScope: "main",          // 私聊连续性(共享收件箱请设置为 per-channel-peer/per-account-channel-peer)
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"]
    },
    reset: {
      // 默认值:mode=daily,atHour=4(Gateway 主机本地时间)。
      // 如果同时设置 idleMinutes,则以先到期的为准。
      mode: "daily",
      atHour: 4,
      idleMinutes: 120
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      dm: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 }
    },
    resetByChannel: {
      discord: { mode: "idle", idleMinutes: 10080 }
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    mainKey: "main",
  }
}
```

<div id="inspecting">
  ## 检查
</div>

* `openclaw status` — 显示存储路径和最近的会话。
* `openclaw sessions --json` — 输出每条记录（可使用 `--active <minutes>` 进行筛选）。
* `openclaw gateway call sessions.list --params '{}'` — 从正在运行的 Gateway 获取会话（远程访问 Gateway 时使用 `--url`/`--token`）。
* 在聊天中单独发送 `/status` 消息，以查看智能体是否可用、会话上下文已使用多少、当前思考/详细模式开关状态，以及你的 WhatsApp Web 凭据上次刷新的时间（有助于发现是否需要重新链接）。
* 发送 `/context list` 或 `/context detail`，查看系统提示词和注入的工作区文件内容（以及最大的上下文贡献来源）。
* 将 `/stop` 作为独立消息发送，以中止当前运行，清除该会话排队的后续任务，并停止由其派生的任何子智能体运行（回复中会包含被停止的数量）。
* 将 `/compact`（可附带可选说明）作为独立消息发送，以总结较旧的上下文并释放窗口空间。参见 [/concepts/compaction](/zh/concepts/compaction)。
* 可以直接打开 JSONL 记录文件来审阅完整的对话轮次。

<div id="tips">
  ## 提示
</div>

* 让主密钥专门用于 1:1 会话流量；各个群组应使用各自独立的密钥。
* 在自动化清理时，删除单个密钥而不是整个存储，以保留其他位置的上下文。

<div id="session-origin-metadata">
  ## 会话来源元数据
</div>

每条会话记录都会在 `origin` 字段中尽可能记录其来源信息：

* `label`: 人类可读标签（由会话标签 + 群组主题/频道解析而来）
* `provider`: 标准化频道 ID（包括扩展）
* `from`/`to`: 来自入站信封的原始路由 ID
* `accountId`: 提供方账号 ID（在多账号场景下）
* `threadId`: 当频道支持时的线程/话题 ID
  这些来源字段会为私信、频道和群组填充。如果某个连接器只更新投递路由（例如用来保持私信主会话处于最新状态），它仍应提供入站上下文，以便该会话保留其说明性元数据。扩展可以通过在入站上下文中发送 `ConversationLabel`、`GroupSubject`、`GroupChannel`、`GroupSpace` 和 `SenderName`，并调用 `recordSessionMetaFromInbound`（或将相同上下文传递给 `updateLastRoute`）来实现这一点。