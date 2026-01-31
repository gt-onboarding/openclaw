---
title: 流式处理
summary: "流式与分块行为（块级回复、草稿流式输出、限制）"
read_when:
  - 需要解释各渠道上的流式或分块工作方式时
  - 需要修改块级流式输出或渠道分块行为时
  - 在调试重复/过早的块级回复或草稿流式输出时
---

<div id="streaming-chunking">
  # 流式输出与分块
</div>

OpenClaw 有两个彼此独立的“流式输出”层：

- **块级流式输出（channels）：** 在助手生成内容时按完成的**块**进行输出。这些是正常的频道消息（不是 token 增量）。
- **近似 token 级的流式输出（仅限 Telegram）：** 在生成期间，通过更新**草稿气泡**来展示部分文本；最终消息会在结束时一次性发送。

目前对外部频道消息**没有真正的 token 级流式输出**。Telegram 草稿流式输出是唯一的部分流式能力入口。

<div id="block-streaming-channel-messages">
  ## 分块流式传输（通道消息）
</div>

分块流式传输会在助手输出变得可用时，将内容按较大的数据块发送出去。

```
Model output
  └─ text_delta/events
       ├─ (blockStreamingBreak=text_end)
       │    └─ chunker emits blocks as buffer grows
       └─ (blockStreamingBreak=message_end)
            └─ chunker flushes at message_end
                   └─ channel send (block replies)
```

图例：

* `text_delta/events`：模型流式事件（对于非流式模型可能会非常稀疏）。
* `chunker`：`EmbeddedBlockChunker`，应用最小/最大边界和断点偏好。
* `channel send`：实际发出的消息（分块回复）。

**控制项：**

* `agents.defaults.blockStreamingDefault`：`"on"`/`"off"`（默认为 off）。
* 通道覆盖：`*.blockStreaming`（以及按账号的变体），用于对每个通道强制 `"on"`/`"off"`。
* `agents.defaults.blockStreamingBreak`：`"text_end"` 或 `"message_end"`。
* `agents.defaults.blockStreamingChunk`：`{ minChars, maxChars, breakPreference? }`。
* `agents.defaults.blockStreamingCoalesce`：`{ minChars?, maxChars?, idleMs? }`（在发送前合并已流式的分块）。
* 通道硬上限：`*.textChunkLimit`（例如 `channels.whatsapp.textChunkLimit`）。
* 通道分块模式：`*.chunkMode`（默认 `length`，`newline` 会在按长度分块前，先在空行（段落边界）处分割）。
* Discord 软上限：`channels.discord.maxLinesPerMessage`（默认 17）会拆分过长的回复以避免 UI 截断。

**边界语义：**

* `text_end`：一旦 chunker 产出分块就立刻流式发送；在每个 `text_end` 时执行一次 flush。
* `message_end`：等待助手消息结束后，再一次性 flush 缓冲输出。

`message_end` 在缓冲文本超过 `maxChars` 时仍然会使用 chunker，因此在结尾处可以输出多个分块。


<div id="chunking-algorithm-lowhigh-bounds">
  ## 分块算法（上下界）
</div>

块分块由 `EmbeddedBlockChunker` 实现：

- **下界：** 在缓冲区长度 &gt;= `minChars` 之前不输出（除非被强制）。
- **上界：** 优先在 `maxChars` 之前拆分；如被强制，则在 `maxChars` 处拆分。
- **断点优先级：** `paragraph` → `newline` → `sentence` → `whitespace` → 硬分割。
- **代码围栏：** 绝不在代码围栏内部拆分；当被迫在 `maxChars` 处拆分时，先关闭再重新打开围栏，以保持 Markdown 的有效性。

`maxChars` 会被限制在该通道的 `textChunkLimit` 之内，因此你无法超过每个通道的上限。

<div id="coalescing-merge-streamed-blocks">
  ## 合并（合并流式块）
</div>

启用块级流式输出时，OpenClaw 可以在发送之前**合并连续的块片段**。这在仍然提供逐步输出的同时，减少了“单行刷屏”。

- 合并会在刷新（flush）前等待**空闲间隔**（`idleMs`）。
- 缓冲区由 `maxChars` 限制，超过该值会触发刷新。
- `minChars` 会阻止微小片段被发送，直到累积到足够多的文本
  （最终刷新始终会发送剩余文本）。
- 拼接方式由 `blockStreamingChunk.breakPreference` 决定
  （`paragraph` → `\n\n`，`newline` → `\n`，`sentence` → 空格）。
- 可通过 `*.blockStreamingCoalesce` 进行通道级覆盖（包括按账号配置）。
- 对于 Signal/Slack/Discord，默认合并的 `minChars` 会提升到 1500，除非被覆盖。

<div id="human-like-pacing-between-blocks">
  ## 类似人类节奏的分块间停顿
</div>

启用分块流式输出后，你可以在分块回复之间添加**随机停顿**（从第二个块开始）。
这会让多气泡回复显得更加自然。

- 配置：`agents.defaults.humanDelay`（可通过 `agents.list[].humanDelay` 为每个智能体单独覆盖）。
- 模式：`off`（默认）、`natural`（800–2500ms）、`custom`（`minMs`/`maxMs`）。
- 仅对**分块回复**生效，不影响最终回复或工具摘要。

<div id="stream-chunks-or-everything">
  ## “按块流式传输还是一次性全部发送”
</div>

对应关系如下：

- **按块流式传输：** `blockStreamingDefault: "on"` + `blockStreamingBreak: "text_end"`（生成即发送）。非 Telegram 渠道还需要设置 `*.blockStreaming: true`。
- **结束时一次性发送全部：** `blockStreamingBreak: "message_end"`（结束时统一刷新输出一次，如果消息很长，可能仍然会拆成多个块）。
- **不进行块级流式传输：** `blockStreamingDefault: "off"`（只发送最终回复）。

**渠道说明：** 对于非 Telegram 渠道，块级流式传输在 **除非显式** 将
`*.blockStreaming` 设为 `true`，否则是关闭的。Telegram 可以流式发送草稿
（`channels.telegram.streamMode`），而无需块级回复。

配置位置提醒：`blockStreaming*` 默认值位于 `agents.defaults` 下，
而不是根级配置。

<div id="telegram-draft-streaming-token-ish">
  ## Telegram 草稿流式传输（类似 token）
</div>

Telegram 是唯一支持草稿流式传输的渠道：

* 在**带话题的私聊**中使用 Bot API `sendMessageDraft`。
* `channels.telegram.streamMode: "partial" | "block" | "off"`。
  * `partial`：草稿会随最新的流式文本实时更新。
  * `block`：草稿按分块更新（遵循相同的分块规则）。
  * `off`：关闭草稿流式传输。
* 草稿分块配置（仅适用于 `streamMode: "block"`）：`channels.telegram.draftChunk`（默认值：`minChars: 200`，`maxChars: 800`）。
* 草稿流式传输与块流式传输相互独立；分块回复默认关闭，只会在非 Telegram 渠道上通过 `*.blockStreaming: true` 启用。
* 最终回复仍然是普通消息。
* `/reasoning stream` 会将推理内容写入草稿气泡（仅限 Telegram）。

当草稿流式传输启用时，OpenClaw 会对该条回复禁用块流式传输，以避免重复流式传输。

```
Telegram（私聊 + 话题）
  └─ sendMessageDraft（草稿气泡）
       ├─ streamMode=partial → 更新最新文本
       └─ streamMode=block   → 分块器更新草稿
  └─ 最终回复 → 普通消息
```

图例：

* `sendMessageDraft`: Telegram 草稿气泡（并非真正发送的消息）。
* `final reply`: 正常发送的 Telegram 消息。
