---
title: 消息
summary: "消息流转、会话、排队与推理可见性"
read_when:
  - 用于解释传入消息如何转化为回复
  - 用于澄清会话、排队模式或流式行为
  - 用于说明推理可见性及其对使用的影响
---

<div id="messages">
  # 消息
</div>

本页整体说明了 OpenClaw 如何处理传入消息、会话、队列、流式处理，以及推理过程的可见性。

<div id="message-flow-high-level">
  ## 消息流（高层概览）
</div>

```
Inbound message
  -> routing/bindings -> session key
  -> queue (if a run is active)
  -> agent run (streaming + tools)
  -> outbound replies (channel limits + chunking)
```

关键控制项位于配置中：

* `messages.*`：用于前缀、排队和分组行为。
* `agents.defaults.*`：用于阻塞式流式输出和分块的默认值。
* 通道覆盖配置（`channels.whatsapp.*`、`channels.telegram.*` 等）：用于上限和流式开关。

完整 schema 请参见 [Configuration](/zh/gateway/configuration)。

<div id="inbound-dedupe">
  ## 入站去重
</div>

通道在重连后可能会再次投递同一条消息。OpenClaw 会维护一个以 channel/account/peer/session/message id 为键的短时缓存，这样重复投递就不会再次触发智能体运行。

<div id="inbound-debouncing">
  ## 入站防抖
</div>

来自**同一发送方**的快速连续消息，可以通过 `messages.inbound` 合并为单个
智能体轮次。防抖以「频道 + 会话」为范围，并使用最新的一条消息用于回复线程和 ID 关联。

配置（全局默认 + 按频道覆盖）：

```json5
{
  messages: {
    inbound: {
      debounceMs: 2000,
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

* 防抖仅适用于**纯文本**消息；媒体/附件会立即发送出去。
* 控制命令会绕过防抖处理，因此始终作为独立消息发送。

<div id="sessions-and-devices">
  ## 会话与设备
</div>

会话归 Gateway 所有，而不是归客户端所有。

* 直接聊天会归并到该智能体的主会话 key 中。
* 群组 / 频道会拥有各自的会话 key。
* 会话存储和对话记录保存在 Gateway 主机上。

多个设备 / 通道可以映射到同一个会话，但历史记录不会完整
同步回每一个客户端。建议：对于长对话，尽量只使用一个主设备，以避免上下文分叉。Control UI 和 TUI 始终展示由 Gateway 支持的会话对话记录，因此它们是唯一可信的事实来源。

详细信息：[会话管理](/zh/concepts/session)。

<div id="inbound-bodies-and-history-context">
  ## 入站正文与历史上下文
</div>

OpenClaw 将**提示正文（prompt body）**与**命令正文（command body）**分开：

* `Body`：发送给智能体的提示文本。它可以包含渠道封装（channel envelopes）和
  可选的历史包装内容。
* `CommandBody`：用于指令/命令解析的原始用户文本。
* `RawBody`：`CommandBody` 的旧版别名（为兼容性保留）。

当某个渠道提供历史记录时，会使用统一的包装格式：

* `[Chat messages since your last reply - for context]`
* `[Current message - respond to this]`

对于**非直接聊天**（群组/频道/聊天室），**当前消息正文**会加上发送者标签前缀
（与历史记录条目使用相同的样式）。这样可以保持实时消息与排队/历史消息在
智能体提示中的一致性。

历史缓冲区是**仅待处理（pending-only）**的：它会包含*未*触发运行的群组消息（例如需要被提及才会触发的消息），并且**不包含**已经写入会话记录的消息。

指令内容剥离仅应用于**当前消息**部分，因此历史内容保持不变。对历史进行包装的渠道应将
`CommandBody`（或 `RawBody`）设置为原始消息文本，并将 `Body` 保持为组合后的提示。
历史缓冲区可通过 `messages.groupChat.historyLimit`（全局默认）以及
`channels.slack.historyLimit` 或
`channels.telegram.accounts.<id>.historyLimit` 等每渠道覆盖项进行配置
（设置为 `0` 可禁用）。

<div id="queueing-and-followups">
  ## 排队与后续处理
</div>

如果某个运行已处于活动状态，后续进入的消息可以被排队、导入当前运行，或收集起来留作后续轮次使用。

* 通过 `messages.queue`（以及 `messages.queue.byChannel`）进行配置。
* 模式：`interrupt`、`steer`、`followup`、`collect`，以及这些模式的 backlog 变体。

详情参见：[队列处理（Queueing）](/zh/concepts/queue)。

<div id="streaming-chunking-and-batching">
  ## 流式传输、分块与批处理
</div>

块级流式传输会在模型生成文本块时发送部分回复。
分块会遵守通道的文本长度限制，并避免拆分围栏代码块。

关键设置：

* `agents.defaults.blockStreamingDefault`（`on|off`，默认 off）
* `agents.defaults.blockStreamingBreak`（`text_end|message_end`）
* `agents.defaults.blockStreamingChunk`（`minChars|maxChars|breakPreference`）
* `agents.defaults.blockStreamingCoalesce`（基于空闲的批处理）
* `agents.defaults.humanDelay`（块级回复之间的类人停顿）
* 通道级覆盖：`*.blockStreaming` 和 `*.blockStreamingCoalesce`（非 Telegram 通道需要显式设置 `*.blockStreaming: true`）

详情：[流式传输与分块](/zh/concepts/streaming)。

<div id="reasoning-visibility-and-tokens">
  ## 推理可见性与 token
</div>

OpenClaw 可以显示或隐藏模型推理内容：

* `/reasoning on|off|stream` 用于控制可见性。
* 当由模型生成时，推理内容仍会计入 token 消耗。
* Telegram 支持将推理以流式形式写入草稿气泡。

详细信息：参见 [思考与推理指令](/zh/tools/thinking) 和 [token 使用](/zh/token-use)。

<div id="prefixes-threading-and-replies">
  ## 前缀、线程和回复
</div>

出站消息格式由 `messages` 统一控制：

* `messages.responsePrefix`（出站前缀）和 `channels.whatsapp.messagePrefix`（WhatsApp 入站前缀）
* 通过 `replyToMode` 和各渠道的默认配置实现回复线程

详情见：[配置](/zh/gateway/configuration#messages) 和各渠道文档。