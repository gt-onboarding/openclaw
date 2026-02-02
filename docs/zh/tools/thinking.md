---
title: 思考
summary: "用于 /think 和 /verbose 的指令语法，以及它们如何影响模型推理"
read_when:
  - 在调整 /think 或 /verbose 指令的解析方式或默认值时
---

<div id="thinking-levels-think-directives">
  # 思考层级（/think 指令）
</div>

<div id="what-it-does">
  ## 功能说明
</div>

* 可在任意入站消息正文中使用的行内指令：`/t <level>`、`/think:<level>` 或 `/thinking <level>`。
* 等级（及其别名）：`off | minimal | low | medium | high | xhigh`（仅适用于 GPT-5.2 和 Codex 模型）
  * minimal → “think”
  * low → “think hard”
  * medium → “think harder”
  * high → “ultrathink”（最大预算）
  * xhigh → “ultrathink+”（仅适用于 GPT-5.2 和 Codex 模型）
  * `highest`、`max` 映射为 `high`。
* 提供方说明：
  * Z.AI（`zai/*`）只支持二元思考模式（`on`/`off`）。任何非 `off` 的等级都会被视为 `on`（映射为 `low`）。

<div id="resolution-order">
  ## 生效顺序
</div>

1. 消息中的内联指令（仅对该条消息生效）。
2. 会话级覆盖（通过发送仅包含指令的消息来设置）。
3. 全局默认值（配置中的 `agents.defaults.thinkingDefault`）。
4. 回退策略：对具备推理能力的模型为 `low`，否则为 `off`。

<div id="setting-a-session-default">
  ## 设置会话默认值
</div>

* 发送一条**只包含**指令的消息（允许有空白），例如 `/think:medium` 或 `/t high`。
* 该设置会在当前会话中持续生效（默认按发送方区分）；在收到 `/think:off` 或会话因空闲被重置时会被清除。
* 会发送确认回复（`Thinking level set to high.` / `Thinking disabled.`）。如果级别无效（例如 `/thinking big`），命令会被拒绝并给出提示，会话状态保持不变。
* 发送不带参数的 `/think`（或 `/think:`）可以查看当前思考级别。

<div id="application-by-agent">
  ## 按智能体划分的应用
</div>

* **Embedded Pi**：解析后的级别会传递给进程内的 Pi Agent 代理运行时。

<div id="verbose-directives-verbose-or-v">
  ## 冗长指令（/verbose 或 /v）
</div>

* 级别：`on`（最小）| `full` | `off`（默认）。
* 仅包含指令的消息会切换会话的冗长模式，并回复 `Verbose logging enabled.` / `Verbose logging disabled.`；无效级别会返回提示，但不会改变当前状态。
* `/verbose off` 会存储一个显式的会话覆盖设置；可在 Sessions UI 中选择 `inherit` 来清除。
* 内联指令只影响该条消息；其他情况使用会话 / 全局默认值。
* 发送 `/verbose`（或 `/verbose:`）且不带参数即可查看当前冗长级别。
* 当冗长开启时，发出结构化工具结果的智能体（Pi、其他 JSON 智能体）会将每次工具调用作为一条仅含元数据的独立消息发回，在可用时（路径 / 命令）以前缀 `<emoji> <tool-name>: <arg>` 标记。这些工具摘要会在每个工具开始时立即发送（独立气泡），而不是以流式增量的方式发送。
* 当冗长为 `full` 时，工具输出也会在完成后被转发（独立气泡，并截断到安全长度）。如果在一次运行进行中切换 `/verbose on|full|off`，之后的工具气泡会遵循新的设置。

<div id="reasoning-visibility-reasoning">
  ## 推理可见性（/reasoning）
</div>

* 级别：`on|off|stream`。
* 此仅限指令的消息用于切换回复中是否显示思考块。
* 启用时，推理将作为**单独消息**发送，并以 `Reasoning:` 为前缀。
* `stream`（仅限 Telegram）：在生成回复期间，将推理内容流式写入 Telegram 草稿气泡中，然后发送不含推理的最终答案。
* 别名：`/reason`。
* 发送 `/reasoning`（或 `/reasoning:`）且不带参数以查看当前的推理级别。

<div id="related">
  ## 相关内容
</div>

* Elevated 模式的文档位于 [Elevated mode](/zh/tools/elevated)。

<div id="heartbeats">
  ## 心跳
</div>

* 心跳探测正文是已配置的心跳提示词（默认：`Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`）。心跳消息中的内联指令照常生效（但请避免在心跳中更改会话默认值）。
* 心跳投递默认仅发送最终载荷。若还需要发送单独的 `Reasoning:` 消息（在可用时），请将 `agents.defaults.heartbeat.includeReasoning: true` 设置为 `true`，或为单个智能体设置 `agents.list[].heartbeat.includeReasoning: true`。

<div id="web-chat-ui">
  ## Web 聊天 UI
</div>

* Web 聊天的思考级别选择器在页面加载时，会从入站会话存储/配置中同步为该会话已存储的级别。
* 选择其他级别只对下一条消息生效（`thinkingOnce`）；发送之后，选择器会恢复为已存储的会话级别。
* 若要更改会话默认级别，发送一个 `/think:&lt;level&gt;` 指令（与之前相同）；选择器会在下次刷新页面后反映该更改。