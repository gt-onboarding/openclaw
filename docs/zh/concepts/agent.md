---
title: Agent 代理
summary: "Agent 运行时（嵌入式 pi-mono）、工作区约定和会话引导/启动"
read_when:
  - 在修改智能体运行时、工作区引导流程或会话行为时阅读
---

<div id="agent-runtime">
  # Agent 运行时 🤖
</div>

OpenClaw 运行一个基于 **pi-mono** 的单一内置智能体运行时。

<div id="workspace-required">
  ## 工作区（必需）
</div>

OpenClaw 使用单个智能体工作区目录（`agents.defaults.workspace`）作为该智能体用于工具和上下文的**唯一**工作目录（`cwd`）。

建议：使用 `openclaw setup` 创建 `~/.openclaw/openclaw.json`（如果不存在），并初始化工作区文件。

完整的工作区结构和备份指南：[Agent 工作区](/zh/concepts/agent-workspace)

如果启用了 `agents.defaults.sandbox`，非主会话可以在 `agents.defaults.sandbox.workspaceRoot` 下使用按会话划分的工作区来覆盖该默认工作区（参见
[Gateway configuration](/zh/gateway/configuration)）。

<div id="bootstrap-files-injected">
  ## 启动引导文件（注入）
</div>

在 `agents.defaults.workspace` 中，OpenClaw 会查找以下可由用户编辑的文件：

* `AGENTS.md` — 操作说明 +「记忆」
* `SOUL.md` — 人设、边界、语气
* `TOOLS.md` — 用户维护的工具说明（例如 `imsg`、`sag`、约定）
* `BOOTSTRAP.md` — 一次性首次运行仪式（完成后会被删除）
* `IDENTITY.md` — Agent 代理的名称 / 风格 / emoji
* `USER.md` — 用户档案 + 首选称呼方式

在新会话的第一次轮次中，OpenClaw 会将这些文件的内容直接注入到智能体上下文中。

空文件会被跳过。大文件会被裁剪，并附加截断标记，以保持提示词精简（如需完整内容，请直接阅读文件）。

如果某个文件缺失，OpenClaw 会注入一行「缺失文件」标记（同时 `openclaw setup` 会创建一个安全的默认模板）。

`BOOTSTRAP.md` 只会在**全新工作区**（不存在其他启动引导文件）中创建。如果你在完成仪式后将其删除，后续重启时不应再次被创建。

若要完全禁用启动引导文件的创建（适用于预置内容的工作区），请设置：

```json5
{ agent: { skipBootstrap: true } }
```

<div id="built-in-tools">
  ## 内置工具
</div>

核心工具（`read`/`exec`/`edit`/`write` 以及相关系统工具）始终可用，
但受工具策略约束。`apply_patch` 是可选的，并由
`tools.exec.applyPatch` 控制。`TOOLS.md` **不会** 决定有哪些工具；它只是用来说明你*希望*这些工具被如何使用的指导。

<div id="skills">
  ## 技能
</div>

OpenClaw 会从三个位置加载技能（名称冲突时，以工作区为准）：

* 内置（随安装一起提供）
* 托管/本地：`~/.openclaw/skills`
* 工作区：`<workspace>/skills`

可以通过配置项或环境变量对技能进行控制（参见 [Gateway 配置](/zh/gateway/configuration) 中的 `skills`）。

<div id="pi-mono-integration">
  ## pi-mono 集成
</div>

OpenClaw 复用了 pi-mono 代码库中的部分组件（模型/工具），但**会话管理、发现和工具连接完全由 OpenClaw 自行管理**。

* 不包含 pi-coding 智能体运行时。
* 不会读取或使用 `~/.pi/agent` 或 `<workspace>/.pi` 中的设置。

<div id="sessions">
  ## 会话
</div>

会话记录以 JSONL 格式存储在：

* `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`

会话 ID 是固定的，由 OpenClaw 分配。
旧版 Pi/Tau 会话文件夹**不会**被读取。

<div id="steering-while-streaming">
  ## 流式传输过程中的引导（Steering）
</div>

当队列模式为 `steer` 时，传入消息会被注入到当前运行中。
队列会在**每次工具调用之后**检查；如果存在排队消息，
则会跳过当前 assistant 消息中剩余的工具调用（工具结果错误为 &quot;Skipped due to queued user message.&quot;），
然后在下一条 assistant 响应之前注入该排队的用户消息。

当队列模式为 `followup` 或 `collect` 时，传入消息会被保留直到当前轮次结束，
然后以这些排队载荷启动一个新的智能体轮次。有关模式 + 去抖（debounce）/容量上限（cap）行为，
参见 [Queue](/zh/concepts/queue)。

块流式传输会在 assistant 块完成后立即发送该块；默认**关闭**
（`agents.defaults.blockStreamingDefault: "off"`）。
通过 `agents.defaults.blockStreamingBreak` 调整边界（`text_end` 与 `message_end`；默认为 text&#95;end）。
使用 `agents.defaults.blockStreamingChunk` 控制软块切分（默认为 800–1200 字符；
优先在段落分隔处拆分，然后是换行；最后才是句子）。
使用 `agents.defaults.blockStreamingCoalesce` 合并流式分片以减少单行刷屏（在发送前基于空闲时间合并）。
非 Telegram 渠道需要显式设置 `*.blockStreaming: true` 才能启用块级回复。
详细工具摘要会在工具开始时输出（不做去抖处理）；Control UI 会在可用时通过智能体事件流式展示工具输出。
更多细节参见：[Streaming + chunking](/zh/concepts/streaming)。

<div id="model-refs">
  ## 模型引用
</div>

配置中的模型引用（例如 `agents.defaults.model` 和 `agents.defaults.models`）会通过在**第一个** `/` 处分割来解析。

* 在配置模型时使用 `provider/model` 格式。
* 如果模型 ID 本身包含 `/`（OpenRouter 风格），则需要包含提供方前缀（例如：`openrouter/moonshotai/kimi-k2`）。
* 如果你省略了提供方，OpenClaw 会将输入视为别名或**默认提供方**下的模型（仅当模型 ID 中不包含 `/` 时有效）。

<div id="configuration-minimal">
  ## 配置（最小）
</div>

至少需要设置：

* `agents.defaults.workspace`
* `channels.whatsapp.allowFrom`（强烈建议配置）

***

*下一步：[群聊](/zh/concepts/group-messages)* 🦞