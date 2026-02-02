---
title: 会话管理与压缩整理
summary: "深入解析：会话存储与对话记录、生命周期以及（自动）压缩整理的内部机制"
read_when:
  - 你需要调试会话 ID、对话记录 JSONL 或 sessions.json 字段
  - 你正在更改自动压缩整理行为或添加“预压缩整理”清理任务
  - 你想要实现记忆清空或静默系统轮次
---

<div id="session-management-compaction-deep-dive">
  # 会话管理与压缩（深度解析）
</div>

本文档说明 OpenClaw 如何端到端管理会话：

* **会话路由**（入站消息如何映射到 `sessionKey`）
* **会话存储**（`sessions.json`）以及它跟踪的内容
* **对话记录持久化**（`*.jsonl`）及其结构
* **对话记录清理**（运行前基于不同提供方的特定修正）
* **上下文限制**（上下文窗口 vs 被跟踪的 tokens）
* **压缩**（手动 + 自动压缩）以及在何处挂接压缩前的工作
* **静默维护**（例如不应产生用户可见输出的内存写操作）

如果你想先看更高层的概览，请从以下内容开始：

* [/concepts/session](/zh/concepts/session)
* [/concepts/compaction](/zh/concepts/compaction)
* [/concepts/session-pruning](/zh/concepts/session-pruning)
* [/reference/transcript-hygiene](/zh/reference/transcript-hygiene)

***

<div id="source-of-truth-the-gateway">
  ## 权威来源：Gateway
</div>

OpenClaw 的设计基于单一的 **Gateway 进程**，由它来持有会话状态。

* 各种 UI（macOS 应用、web Control UI、TUI）都应向 Gateway 查询会话列表和 token 数。
* 在远程模式下，会话文件存放在远程主机上；“检查你本地 Mac 上的文件”并不能反映 Gateway 实际在使用的会话数据。

***

<div id="two-persistence-layers">
  ## 两个持久化层
</div>

OpenClaw 在两个层面持久化会话：

1. **会话存储（`sessions.json`）**
   * 键/值映射：`sessionKey -> SessionEntry`
   * 体量小、可变，适合编辑（或删除条目）
   * 记录会话元数据（当前会话 id、最近活动时间、开关状态、token 计数器等）

2. **会话记录（`<sessionId>.jsonl`）**
   * 仅追加写入且具有树形结构的记录文件（条目具有 `id` + `parentId`）
   * 存储实际对话内容 + 工具调用 + 压缩过程生成的摘要
   * 用于在后续轮次中重建模型上下文

***

<div id="on-disk-locations">
  ## 磁盘上的存储位置
</div>

在 Gateway 主机上，按智能体划分：

* 存储：`~/.openclaw/agents/<agentId>/sessions/sessions.json`
* 会话日志：`~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl`
  * Telegram 话题会话：`.../<sessionId>-topic-<threadId>.jsonl`

OpenClaw 通过 `src/config/sessions.ts` 解析这些路径。

<div id="session-keys-sessionkey">
  ## 会话键（`sessionKey`）
</div>

`sessionKey` 用来标识你当前属于*哪个会话分组*（用于路由和隔离）。

常见模式：

* 主/直接聊天（按智能体划分）：`agent:<agentId>:<mainKey>`（默认 `main`）
* 群组：`agent:<agentId>:<channel>:group:<id>`
* 房间/频道（Discord/Slack）：`agent:<agentId>:<channel>:channel:<id>` 或 `...:room:<id>`
* Cron：`cron:<job.id>`
* Webhook：`hook:<uuid>`（除非被覆盖）

规范性规则见 [/concepts/session](/zh/concepts/session)。

<div id="session-ids-sessionid">
  ## 会话 ID（`sessionId`）
</div>

每个 `sessionKey` 都指向一个当前的 `sessionId`（用于继续该会话的对话记录文件）。

经验法则：

* **重置**（`/new`、`/reset`）会为该 `sessionKey` 创建一个新的 `sessionId`。
* **每日重置**（默认在 Gateway 主机本地时间凌晨 4:00）会在越过重置边界后收到的下一条消息上创建一个新的 `sessionId`。
* **空闲过期**（`session.reset.idleMinutes` 或旧版 `session.idleMinutes`）会在消息在空闲时间窗口之后才到达时创建一个新的 `sessionId`。当每日重置和空闲过期都已配置时，以先到期的为准。

实现细节：该决策逻辑位于 `src/auto-reply/reply/session.ts` 中的 `initSessionState()`。

<div id="session-store-schema-sessionsjson">
  ## 会话存储结构（`sessions.json`）
</div>

存储的值类型为 `src/config/sessions.ts` 中的 `SessionEntry`。

关键字段（不完全列举）：

* `sessionId`：当前记录 ID（文件名由此推导，除非设置了 `sessionFile`）
* `updatedAt`：最近一次活动的时间戳
* `sessionFile`：可选的显式对话记录路径覆盖值
* `chatType`：`direct | group | room`（用于辅助 UI 和发送策略）
* `provider`、`subject`、`room`、`space`、`displayName`：用于群组/频道标记的元数据
* 开关选项：
  * `thinkingLevel`、`verboseLevel`、`reasoningLevel`、`elevatedLevel`
  * `sendPolicy`（按会话级别覆盖）
* 模型选择：
  * `providerOverride`、`modelOverride`、`authProfileOverride`
* Token 计数器（尽力统计 / 依赖提供方）：
  * `inputTokens`、`outputTokens`、`totalTokens`、`contextTokens`
* `compactionCount`：该会话键自动压缩已完成的次数
* `memoryFlushAt`：上一次预压缩内存清空的时间戳
* `memoryFlushCompactionCount`：上一次清空执行时对应的压缩计数

该存储可以安全地手动编辑，但 Gateway 才是权威来源：在会话运行期间，它可能重写或重新构建（rehydrate）条目。

***

<div id="transcript-structure-jsonl">
  ## 转录结构（`*.jsonl`）
</div>

转录由 `@mariozechner/pi-coding-agent` 的 `SessionManager` 管理。

文件为 JSONL 格式：

* 第一行：会话头信息（`type: "session"`，包含 `id`、`cwd`、`timestamp`，以及可选的 `parentSession`）
* 随后：带有 `id` 和 `parentId` 的会话条目（树形结构）

主要条目类型：

* `message`：user/assistant/toolResult 消息
* `custom_message`：由扩展注入、**会**进入模型上下文的消息（可从 UI 中隐藏）
* `custom`：**不会**进入模型上下文的扩展状态
* `compaction`：持久化的压缩摘要，包含 `firstKeptEntryId` 和 `tokensBefore`
* `branch_summary`：在导航树分支时持久化的摘要

OpenClaw 刻意**不会**“修复”这些转录；Gateway 使用 `SessionManager` 来读/写它们。

***

<div id="context-windows-vs-tracked-tokens">
  ## 上下文窗口 vs 已跟踪的 token
</div>

有两个不同的概念很重要：

1. **模型上下文窗口（model context window）**：针对每个模型的硬性上限（模型可见的 token）
2. **会话存储计数器（session store counters）**：写入 `sessions.json` 的滚动统计信息（用于 /status 和各类仪表盘）

如果你在调整这些限制：

* 上下文窗口来自模型目录（并且可以通过配置覆盖）。
* 存储中的 `contextTokens` 是一个运行时的估算/统计值；不要把它当作严格保证。

更多内容参见 [/token-use](/zh/token-use)。

***

<div id="compaction-what-it-is">
  ## 压缩：含义
</div>

压缩会将较早的对话内容汇总为一条持久化的 `compaction` 条目保存在对话记录中，并保持最近的消息不变。

在压缩之后，后续轮次将看到：

* 压缩摘要
* `firstKeptEntryId` 之后的消息

压缩是**持久化的**（不同于会话修剪）。参见 [/concepts/session-pruning](/zh/concepts/session-pruning)。

***

<div id="when-auto-compaction-happens-pi-runtime">
  ## 自动压缩在何时触发（Pi 运行时）
</div>

在嵌入式 Pi 智能体中，自动压缩会在两种情况下触发：

1. **溢出恢复**：模型返回上下文溢出错误 → 执行压缩 → 重试。
2. **阈值维护**：在一次调用成功完成后，当：

`contextTokens > contextWindow - reserveTokens`

其中：

* `contextWindow` 是模型的上下文窗口
* `reserveTokens` 是为提示词和下一次模型输出预留的冗余 token 空间

这些是 Pi 运行时自身的语义（OpenClaw 只消费这些事件，由 Pi 决定何时执行压缩）。

***

<div id="compaction-settings-reservetokens-keeprecenttokens">
  ## 压缩设置（`reserveTokens`、`keepRecentTokens`）
</div>

Pi 的压缩设置位于 Pi 设置中：

```json5
{
  compaction: {
    enabled: true,
    reserveTokens: 16384,
    keepRecentTokens: 20000
  }
}
```

OpenClaw 还会对嵌入式运行强制执行一个安全下限：

* 如果 `compaction.reserveTokens < reserveTokensFloor`，OpenClaw 会将其提升到该下限值。
* 默认下限为 `20000` 个 token。
* 将 `agents.defaults.compaction.reserveTokensFloor: 0` 设为 0 可禁用该下限。
* 如果当前值已经更高，OpenClaw 将保持不变。

原因：在压缩变得不可避免之前，为多轮“维护性”操作（如内存写入）保留足够的余量。

实现：`src/agents/pi-settings.ts` 中的 `ensurePiCompactionReserveTokens()`
（由 `src/agents/pi-embedded-runner.ts` 调用）。

***

<div id="user-visible-surfaces">
  ## 用户可见界面
</div>

你可以通过以下方式查看压缩和会话状态：

* `/status`（在任意聊天会话中）
* `openclaw status`（CLI）
* `openclaw sessions` / `sessions --json`
* 详细输出模式：`🧹 Auto-compaction complete` + 压缩次数

***

<div id="silent-housekeeping-no_reply">
  ## 静默后台维护（`NO_REPLY`）
</div>

OpenClaw 支持用于后台维护任务的“静默”对话轮次，在这些轮次中用户不应看到中间输出。

约定：

* 助手以 `NO_REPLY` 开头输出，用于指示“不要向用户发送任何回复”。
* OpenClaw 会在投递层去除/屏蔽这一标记。

自 `2026.1.10` 起，当某个部分数据块以 `NO_REPLY` 开头时，OpenClaw 也会抑制**草稿/输入中状态的流式输出**，从而避免静默操作在轮次中途泄露部分输出。

***

<div id="pre-compaction-memory-flush-implemented">
  ## 预压缩“内存刷新”（已实现）
</div>

目标：在自动压缩发生之前，运行一次静默的智能体轮次，把持久化状态写入磁盘（例如智能体工作区中的 `memory/YYYY-MM-DD.md`），从而避免压缩过程擦除关键上下文。

OpenClaw 采用的是 **预阈值刷新（pre-threshold flush）** 方案：

1. 监控会话上下文的使用情况。
2. 当使用量跨过一个“软阈值”（低于 Pi 的压缩阈值）时，对智能体运行一次静默的
   “立即写入内存（write memory now）”指令。
3. 使用 `NO_REPLY`，因此用户不会看到任何内容。

配置（`agents.defaults.compaction.memoryFlush`）：

* `enabled`（默认：`true`）
* `softThresholdTokens`（默认：`4000`）
* `prompt`（用于刷新轮次的用户消息）
* `systemPrompt`（附加到刷新轮次的额外 system prompt）

说明：

* 默认的 prompt/system prompt 会包含一个 `NO_REPLY` 提示，用于抑制消息投递。
* 每个压缩周期只会运行一次刷新（在 `sessions.json` 中跟踪）。
* 刷新仅对嵌入式 Pi 会话生效（CLI 后端会跳过此步骤）。
* 当会话工作区是只读时会跳过刷新（`workspaceAccess: "ro"` 或 `"none"`）。
* 有关工作区文件布局和写入模式，参见 [Memory](/zh/concepts/memory)。

Pi 还在扩展 API 中暴露了一个 `session_before_compact` 钩子，但目前 OpenClaw 的刷新逻辑是在 Gateway 这一侧实现的。

***

<div id="troubleshooting-checklist">
  ## 故障排查清单
</div>

* 会话 key 是否错误？先阅读 [/concepts/session](/zh/concepts/session)，并在 `/status` 中确认 `sessionKey`。
* Store 与 transcript 不一致？确认 Gateway 主机以及在 `openclaw status` 中显示的存储路径。
* 压缩操作刷屏？检查：
  * 模型上下文窗口（是否过小）
  * 压缩设置（`reserveTokens` 相对于模型窗口过大可能会导致更早压缩）
  * 工具结果膨胀：启用/调优会话清理（pruning）
* 静默轮次有泄漏？确认回复以 `NO_REPLY`（精确 token）开头，并且你正在使用已包含流式抑制修复的构建版本。