---
title: Agent 循环
summary: "Agent 循环的生命周期、流式处理和等待语义"
read_when:
  - 你需要对 Agent 循环或其生命周期事件进行精确的逐步讲解
---

<div id="agent-loop-openclaw">
  # Agent Loop (OpenClaw)
</div>

一个智能体循环（agentic loop）就是一次完整、真实的智能体运行：输入 → 上下文组装 → 模型推理 → 工具执行 → 流式回复 → 持久化。它是将一条消息转换为一系列动作和最终回复的标准路径，同时保持会话状态的一致性。

在 OpenClaw 中，一个循环是在单个会话内进行的一次串行运行，会在模型思考、调用工具和流式输出的过程中发出生命周期事件和流事件。本文档解释这个真实的循环是如何端到端串接起来的。

<div id="entry-points">
  ## 入口
</div>

* Gateway RPC：`agent` 和 `agent.wait`。
* CLI：`agent` 命令。

<div id="how-it-works-high-level">
  ## 工作原理（高层概览）
</div>

1. `agent` RPC 校验参数，解析会话标识（sessionKey/sessionId），持久化会话元数据，并立即返回 `{ runId, acceptedAt }`。
2. `agentCommand` 运行智能体：
   * 解析模型以及思考/verbose 默认配置
   * 加载技能快照
   * 调用 `runEmbeddedPiAgent`（pi-agent-core 运行时）
   * 如果嵌入式循环未发出 **lifecycle end/error**，则由其补发
3. `runEmbeddedPiAgent`：
   * 通过每会话队列和全局队列将运行串行化
   * 解析模型和认证配置（auth profile），并构建 pi 会话
   * 订阅 pi 事件并流式发送 assistant/tool 增量数据
   * 强制执行超时控制 -&gt; 超时则中止运行
   * 返回 payload 和用量元数据
4. `subscribeEmbeddedPiSession` 将 pi-agent-core 事件桥接到 OpenClaw `agent` 流：
   * 工具事件 =&gt; `stream: "tool"`
   * assistant 增量 =&gt; `stream: "assistant"`
   * 生命周期事件 =&gt; `stream: "lifecycle"`（`phase: "start" | "end" | "error"`）
5. `agent.wait` 使用 `waitForAgentJob`：
   * 等待对应 `runId` 的 **lifecycle end/error**
   * 返回 `{ status: ok|error|timeout, startedAt, endedAt, error? }`

<div id="queueing-concurrency">
  ## 排队与并发
</div>

* 每个会话键（session lane）上的运行都会被串行化，并且还可以选择通过一个全局 lane 串行化。
* 这样可以防止工具/会话的竞争条件，并保持会话历史的一致性。
* 消息通道可以选择队列模式（collect/steer/followup）来接入这个 lane 系统。
  参见 [命令队列](/zh/concepts/queue)。

<div id="session-workspace-preparation">
  ## 会话与工作区准备
</div>

* 解析并创建工作区；在沙箱运行时，可能会重定向到沙箱工作区根目录。
* 加载技能（或从快照中复用），并注入到运行环境和提示词中。
* 解析引导/上下文文件，并注入到系统提示报告中。
* 获取会话写锁；在开始流式传输之前打开并准备 `SessionManager`。

<div id="prompt-assembly-system-prompt">
  ## 提示组装与 system prompt
</div>

* system prompt 由 OpenClaw 的基础提示、技能提示、启动上下文和每次运行的覆盖配置组合构建。
* 会强制应用模型特定的限制，并预留用于压缩的 token。
* 参见 [System prompt](/zh/concepts/system-prompt) 了解模型实际接收到的内容。

<div id="hook-points-where-you-can-intercept">
  ## 钩子点（可拦截的位置）
</div>

OpenClaw 有两套钩子系统：

* **内部钩子**（Gateway 钩子）：用于处理命令和生命周期事件的事件驱动脚本。
* **插件钩子**：嵌入在智能体/工具生命周期和 Gateway 处理流水线中的扩展点。

<div id="internal-hooks-gateway-hooks">
  ### 内部钩子（Gateway 钩子）
</div>

* **`agent:bootstrap`**：在系统提示词最终确定之前、构建引导文件时运行。
  可用它来添加或移除引导阶段的上下文文件。
* **命令钩子**：`/new`、`/reset`、`/stop` 以及其他命令事件（参见 Hooks 文档）。

有关配置和示例，请参见 [Hooks](/zh/hooks)。

<div id="plugin-hooks-agent-gateway-lifecycle">
  ### 插件钩子（智能体 + Gateway 生命周期）
</div>

这些钩子在智能体循环或 Gateway 处理流水线中运行：

* **`before_agent_start`**：在运行开始前注入上下文或覆盖系统提示词（system prompt）。
* **`agent_end`**：在完成后检查最终消息列表和运行元数据。
* **`before_compaction` / `after_compaction`**：观察或标注压缩周期。
* **`before_tool_call` / `after_tool_call`**：拦截工具参数/结果。
* **`tool_result_persist`**：在工具结果写入会话转录之前同步转换这些结果。
* **`message_received` / `message_sending` / `message_sent`**：入站与出站消息钩子。
* **`session_start` / `session_end`**：会话生命周期边界。
* **`gateway_start` / `gateway_stop`**：Gateway 生命周期事件。

参见 [Plugins](/zh/plugin#plugin-hooks) 了解钩子 API 与注册细节。

<div id="streaming-partial-replies">
  ## 流式传输与部分回复
</div>

* Assistant 增量内容由 pi-agent-core 以流式方式推送，并作为 `assistant` 事件发出。
* 块级流式传输可以在 `text_end` 或 `message_end` 时发出部分回复。
* 推理流式传输可以作为单独的数据流发出，或以块级回复的形式发出。
* 有关分块与块级回复行为，请参见 [Streaming](/zh/concepts/streaming)。

<div id="tool-execution-messaging-tools">
  ## 工具执行与消息类工具
</div>

* 工具的 start/update/end 事件会在 `tool` 流上发出。
* 在记录/发出前，会根据结果体积和图像负载对工具结果进行清理和裁剪。
* 会跟踪消息工具的发送，以抑制助手的重复确认回复。

<div id="reply-shaping-suppression">
  ## 回复整形与抑制
</div>

* 最终载荷由以下部分组装而成：
  * 助手文本（以及可选的推理内容）
  * 内联工具摘要（在详尽模式且被允许时）
  * 当模型报错时的助手错误文本
* `NO_REPLY` 被视为静默标记，会从发出的载荷中被过滤掉。
* 消息工具的重复项会从最终载荷列表中移除。
* 如果在工具报错的情况下，没有任何可渲染的载荷剩余，则会发出一个后备的工具错误回复
  （除非某个消息工具已经发送了用户可见的回复）。

<div id="compaction-retries">
  ## 压缩与重试
</div>

* 自动压缩会发出 `compaction` 流事件，并可以触发重试。
* 在重试时，会重置内存缓冲和工具摘要，以避免产生重复输出。
* 有关压缩管线，请参见 [Compaction](/zh/concepts/compaction)。

<div id="event-streams-today">
  ## 事件流（当前）
</div>

* `lifecycle`：由 `subscribeEmbeddedPiSession` 触发（在回退路径中由 `agentCommand` 触发）
* `assistant`：来自 pi-agent-core 的增量数据流
* `tool`：来自 pi-agent-core 的工具事件流

<div id="chat-channel-handling">
  ## 聊天通道处理
</div>

* 助手的增量更新会被缓冲为聊天 `delta` 消息。
* 在**生命周期结束或出错**时会发送聊天 `final` 消息。

<div id="timeouts">
  ## 超时
</div>

* `agent.wait` 默认值：30 秒（仅指等待时间）。可通过 `timeoutMs` 参数覆盖。
* Agent 运行时环境：`agents.defaults.timeoutSeconds` 默认 600 秒；由 `runEmbeddedPiAgent` 的中止计时器强制执行。

<div id="where-things-can-end-early">
  ## 可能提前结束的情况
</div>

* Agent 代理超时（中止）
* AbortSignal（取消）
* Gateway 断开连接或 RPC 超时
* `agent.wait` 超时（仅等待，不会停止智能体）