---
title: 队列
summary: "用于将入站自动回复执行串行化的命令队列设计"
read_when:
  - 更改自动回复的执行方式或并发度时
---

<div id="command-queue-2026-01-16">
  # 命令队列（2026-01-16）
</div>

我们通过一个很小的进程内队列将所有通道的入站自动回复执行串行化，以防止多个智能体运行发生冲突，同时仍然允许在会话之间安全地并行运行。

<div id="why">
  ## 为什么
</div>

- 自动回复的运行成本可能很高（LLM 调用），并且在多条入站消息在短时间内同时到达时可能发生冲突。
- 串行化处理可以避免对共享资源（会话文件、日志、CLI 标准输入）的竞争，并降低触发上游限流的风险。

<div id="how-it-works">
  ## 工作原理
</div>

- 一个具备 lane 感知能力的 FIFO 队列，以可配置的并发上限（未配置的 lane 默认 1；`main` 默认 4，`subagent` 默认 8）来消费每个 lane。
- `runEmbeddedPiAgent` 按 **session key**（lane `session:<key>`）入队，以保证每个会话在任意时刻只有一个活动执行。
- 每个会话执行随后会被排入一个**全局 lane**（默认是 `main`），因此整体并行度会被 `agents.defaults.maxConcurrent` 所限制。
- 当启用详细日志时，如果排队的执行在开始前等待了超过约 2 秒，会输出一条简短提示。
- “正在输入”指示器在入队时仍会立即触发（如果该通道支持），因此在等待轮到我们期间，用户体验保持不变。

<div id="queue-modes-per-channel">
  ## 队列模式（按渠道）
</div>

入站消息可以引导当前运行、等待后续轮次，或两者兼有：

* `steer`：立即注入当前运行中（在下一个工具边界之后取消挂起的工具调用）。如果未启用流式传输，则回退为后续模式。
* `followup`：在当前运行结束后，入队到下一个智能体轮次。
* `collect`：将所有排队消息合并为**单个**后续轮次（默认）。如果消息目标为不同的渠道/线程，为保持路由，将分别出队处理。
* `steer-backlog`（又名 `steer+backlog`）：立即引导当前运行，**并且**保留该消息用于后续轮次。
* `interrupt`（旧版）：终止该会话的活动运行，然后运行最新的消息。
* `queue`（旧版别名）：等同于 `steer`。

Steer-backlog 意味着在已引导的运行之后，你仍然可以再获得一个后续响应，因此
在流式界面上看起来可能像是出现了重复。如果你希望每条入站消息只得到一个响应，请优先使用 `collect`/`steer`。
将 `/queue collect` 作为独立命令发送（按会话），或设置 `messages.queue.byChannel.discord: "collect"`。

默认值（在配置中未设置时）：

* 所有界面 → `collect`

通过 `messages.queue` 进行全局或按渠道配置：

```json5
{
  messages: {
    queue: {
      mode: "collect",
      debounceMs: 1000,
      cap: 20,
      drop: "summarize",
      byChannel: { discord: "collect" }
    }
  }
}
```


<div id="queue-options">
  ## 队列选项
</div>

这些选项适用于 `followup`、`collect` 和 `steer-backlog`（以及在回退为 followup 时的 `steer`）：

- `debounceMs`：在开始一次 followup 轮次前先等待一小段静默时间（防止连续“continue, continue”）。
- `cap`：每个会话中队列允许的最大消息数。
- `drop`：溢出策略（`old`、`new`、`summarize`）。

`summarize` 会保留一份被丢弃消息的简短要点列表，并将其作为一个合成的 followup 提示注入。
默认值：`debounceMs: 1000`、`cap: 20`、`drop: summarize`。

<div id="per-session-overrides">
  ## 会话级覆盖
</div>

- 将 `/queue <mode>` 作为独立命令发送，以为当前会话保存模式。
- 选项可以组合使用：`/queue collect debounce:2s cap:25 drop:summarize`
- 使用 `/queue default` 或 `/queue reset` 清除该会话的覆盖设置。

<div id="scope-and-guarantees">
  ## 范围与保证
</div>

- 适用于所有使用 Gateway 回复管线的入站渠道上的自动回复智能体运行（WhatsApp web、Telegram、Slack、Discord、Signal、iMessage、webchat 等）。
- 默认 lane（`main`）在整个进程范围内处理入站请求和主心跳；通过设置 `agents.defaults.maxConcurrent` 允许多个会话并行。
- 可以存在额外 lane（例如 `cron`、`subagent`），使后台作业可并行运行而不阻塞入站回复。
- 按会话划分的 lane 保证在任意时刻只有一次智能体运行会访问给定会话。
- 无外部依赖或后台工作线程；基于纯 TypeScript + Promise 实现。

<div id="troubleshooting">
  ## 故障排查
</div>

- 如果命令看起来卡住了，启用详细日志，并查找包含 “queued for …ms” 的行，以确认队列确实在排空。
- 如果你需要查看队列深度，启用详细日志并留意队列时序相关的日志行。