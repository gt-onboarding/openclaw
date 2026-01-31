---
title: 输入状态指示器
summary: "OpenClaw 何时显示输入状态指示器，以及如何调整其行为"
read_when:
  - 需要更改输入状态指示器的行为或默认设置时
---

<div id="typing-indicators">
  # 输入指示器
</div>

在运行处于活动状态时，会向聊天频道发送输入指示器。使用
`agents.defaults.typingMode` 控制**何时**开始显示输入状态，使用 `typingIntervalSeconds`
控制它**多频繁**刷新。

<div id="defaults">
  ## 默认行为
</div>

当 `agents.defaults.typingMode` **未设置** 时，OpenClaw 会保持旧版行为：

- **私聊**：一旦模型循环开始，就立刻显示输入状态指示。
- **带 @ 提及的群聊**：立刻显示输入状态指示。
- **未被提及的群聊**：仅在消息文本开始流式输出时才显示输入状态指示。
- **心跳运行**：禁用输入状态指示。

<div id="modes">
  ## 模式
</div>

将 `agents.defaults.typingMode` 设置为以下之一：

- `never` — 从不显示正在输入指示器。
- `instant` — **一旦模型循环开始就立即**显示正在输入指示器，即使该次
  运行最终只返回静默回复标记。
- `thinking` — 在**首次推理增量**时开始显示正在输入指示器（该次运行需
  要设置 `reasoningLevel: "stream"`）。
- `message` — 在**首次非静默文本增量**时开始显示正在输入指示器（忽略
  `NO_REPLY` 静默标记）。

按“触发时间早晚”的顺序：
`never` → `message` → `thinking` → `instant`

<div id="configuration">
  ## 配置
</div>

```json5
{
  agent: {
    typingMode: "thinking",
    typingIntervalSeconds: 6
  }
}
```

你可以在每个会话中单独覆盖模式或节奏：

```json5
{
  session: {
    typingMode: "message",
    typingIntervalSeconds: 4
  }
}
```


<div id="notes">
  ## 说明
</div>

- 在仅静默回复的情况下，`message` 模式不会显示输入状态指示（例如用于抑制输出的 `NO_REPLY` 标记）。
- 只有当该运行以流式方式输出推理过程（`reasoningLevel: "stream"`）时，才会触发 `thinking`。
  如果模型不发出推理增量，则不会开始显示输入状态指示。
- 心跳在任何模式下都不会显示输入状态指示。
- `typingIntervalSeconds` 控制的是**刷新节奏**，而不是开始时间。
  默认值为 6 秒。