---
title: Markdown 格式化
summary: "出站通道的 Markdown 格式化流水线"
read_when:
  - 你正在更改出站通道的 Markdown 格式化或分块行为
  - 你正在添加新的通道格式化器或样式映射
  - 你正在排查跨通道的格式回归问题
---

<div id="markdown-formatting">
  # Markdown 格式化
</div>

OpenClaw 在渲染各渠道的特定输出之前，会先将输出的 Markdown 转换成一个共享的中间表示（IR）。IR 在保持源文本完整不变的同时，携带样式/链接区段信息，从而使分片和渲染在各个渠道之间保持一致。

<div id="goals">
  ## 目标
</div>

* **一致性：** 仅解析一次，由多个渲染器复用同一结果。
* **安全分块：** 在渲染前对文本进行切分，确保行内格式在分块时不会被拆开。
* **渠道适配：** 将同一 IR 映射到 Slack mrkdwn、Telegram HTML 和 Signal 样式区间，而无需重新解析 Markdown。

<div id="pipeline">
  ## 流水线
</div>

1. **解析 Markdown -&gt; IR**
   * IR 由纯文本加上样式范围（粗体/斜体/删除线/代码/剧透）以及链接范围组成。
   * 偏移量使用 UTF-16 码元，这样 Signal 的样式范围就能与其 api 对齐。
   * 只有当某个渠道选择启用表格转换时，表格才会被解析。
2. **对 IR 分块（format-first）**
   * 分块发生在渲染前的 IR 文本上。
   * 行内格式不会跨块拆分；样式范围会按分块进行切片。
3. **按渠道渲染**
   * **Slack：** 使用 mrkdwn 标记（粗体/斜体/删除线/代码），链接为 `<url|label>` 形式。
   * **Telegram：** 使用 HTML 标签（`<b>`、`<i>`、`<s>`、`<code>`、`<pre><code>`、`<a href>`）。
   * **Signal：** 纯文本 + `text-style` 范围；当标签文本与 URL 不同时，链接会变为 `label (url)`。

<div id="ir-example">
  ## IR 示例
</div>

输入的 Markdown：

```markdown
Hello **world** — see [docs](https://docs.openclaw.ai).
```

IR（示意图）：

```json
{
  "text": "Hello world — see docs.",
  "styles": [
    { "start": 6, "end": 11, "style": "bold" }
  ],
  "links": [
    { "start": 19, "end": 23, "href": "https://docs.openclaw.ai" }
  ]
}
```

<div id="where-it-is-used">
  ## 使用范围
</div>

* Slack、Telegram 和 Signal 的出站适配器会基于 IR 渲染内容。
* 其他渠道（WhatsApp、iMessage、MS Teams、Discord）仍然使用纯文本或各自的格式规则，在启用时会在分块前先执行 Markdown 表格转换。

<div id="table-handling">
  ## 表格处理
</div>

不同聊天客户端对 Markdown 表格的支持并不一致。使用
`markdown.tables` 来按频道（以及按账号）控制具体的转换方式。

* `code`：将表格渲染为代码块（大多数频道的默认行为）。
* `bullets`：将每一行转换为项目符号列表（Signal 和 WhatsApp 的默认行为）。
* `off`：禁用表格解析和转换；原始表格文本将原样透传。

配置键：

```yaml
channels:
  discord:
    markdown:
      tables: code
    accounts:
      work:
        markdown:
          tables: off
```

<div id="chunking-rules">
  ## 分块规则
</div>

* 分块上限来自 channel 适配器/配置，并应用于 IR 文本。
* 代码块会作为一个整体保留，并在末尾追加换行，以便各个 channel
  正确渲染它们。
* 列表前缀和引用前缀属于 IR 文本的一部分，因此分块时不会在前缀中间被拆开。
* 内联样式（粗体/斜体/删除线/内联代码/剧透隐藏）永远不会被拆分到不同的块中；渲染器会在每个块内重新打开这些样式。

如果你需要了解更多在不同 channel 上的分块行为，请参见
[Streaming + chunking](/zh/concepts/streaming)。

<div id="link-policy">
  ## 链接策略
</div>

* **Slack：** `[label](url)` -&gt; `<url|label>`；裸 URL 保持不变。解析时禁用自动链接，以避免重复加链接。
* **Telegram：** `[label](url)` -&gt; `<a href="url">label</a>`（HTML 解析模式）。
* **Signal：** `[label](url)` -&gt; `label (url)`，除非 label 与 URL 完全相同。

<div id="spoilers">
  ## 剧透
</div>

剧透标记（`||spoiler||`）仅会在 Signal 中被解析，并映射为
SPOILER 样式范围。其他渠道会将它们视为普通文本。

<div id="how-to-add-or-update-a-channel-formatter">
  ## 如何添加或更新频道格式化器
</div>

1. **单次解析：** 使用共享的 `markdownToIR(...)` 辅助函数，并根据频道选择合适的选项（自动链接、标题样式、引用前缀）。
2. **渲染：** 使用 `renderMarkdownWithMarkers(...)` 和样式标记映射表（或 Signal 样式区间）实现一个渲染器。
3. **分块：** 在渲染前调用 `chunkMarkdownIR(...)`；对每个分块分别渲染。
4. **连接适配器：** 更新该频道的出站适配器以使用新的分块器和渲染器。
5. **测试：** 添加或更新格式测试；如果频道使用分块，还要添加一个出站发送测试。

<div id="common-gotchas">
  ## 常见易错点
</div>

* 必须保留 Slack 尖括号形式的令牌（`<@U123>`、`<#C123>`、`<https://...>`）；要安全地转义原始 HTML。
* Telegram 的 HTML 要求对标签外的文本进行转义，以避免破坏标记结构。
* Signal 的样式范围依赖 UTF-16 偏移量；不要使用码点偏移量。
* 对于围栏代码块，要保留末尾的换行符，这样闭合标记才能单独位于一行。