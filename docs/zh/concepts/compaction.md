---
title: 压缩
summary: "上下文窗口与压缩：OpenClaw 如何让会话保持在模型限制范围内"
read_when:
  - 你想了解自动压缩机制以及 /compact 命令
  - 你正在调试因会话过长而触及上下文限制的问题
---

<div id="context-window-compaction">
  # 上下文窗口与压缩
</div>

每个模型都有一个**上下文窗口**（一次最多能处理的 token 数）。长时间运行的会话会不断累积消息和工具结果；一旦窗口接近上限，OpenClaw 会对较旧的历史记录进行**压缩处理**，以保持在限制范围内。

<div id="what-compaction-is">
  ## 什么是压缩
</div>

压缩会将**较早的对话内容**汇总为一个精简的摘要条目，并保持最近的消息原样保留。该摘要会被存储在会话历史中，因此后续请求会使用：

* 压缩生成的摘要
* 压缩点之后的最近消息

压缩结果会**持久化**存储在会话的 JSONL 历史中。

<div id="configuration">
  ## 配置
</div>

请参见[压缩配置与模式](/zh/concepts/compaction)，了解 `agents.defaults.compaction` 的设置。

<div id="auto-compaction-default-on">
  ## 自动压缩（默认开启）
</div>

当会话接近或超过模型的上下文窗口时，OpenClaw 会触发自动压缩，并可能使用压缩后的上下文重试原始请求。

你会看到：

* 在 verbose 模式中出现 `🧹 Auto-compaction complete`
* `/status` 中显示 `🧹 Compactions: <count>`

在压缩之前，OpenClaw 可以先执行一次**静默内存刷新**轮次，把需要持久化的笔记写入磁盘。详情和配置参见 [Memory](/zh/concepts/memory)。

<div id="manual-compaction">
  ## 手动压缩
</div>

使用 `/compact`（可以附带指令）来强制执行一次压缩操作：

```
/compact Focus on decisions and open questions
```

<div id="context-window-source">
  ## 上下文窗口来源
</div>

上下文窗口取决于具体模型。OpenClaw 使用已配置提供方目录中的模型定义来确定限制。

<div id="compaction-vs-pruning">
  ## Compaction vs pruning
</div>

* **Compaction**：对内容进行总结，并以 JSONL 形式**持久化**存储。
* **Session pruning**：只**裁剪**旧的**工具结果**，**仅在内存中**，按请求执行。

有关会话裁剪的详细信息，请参见 [/concepts/session-pruning](/zh/concepts/session-pruning)。

<div id="tips">
  ## 提示
</div>

* 当会话变得陈旧或上下文过于臃肿时，使用 `/compact`。
* 体积较大的工具输出会自动被截断；通过进一步清理可以减少工具结果的堆积。
* 如果你需要一个全新的上下文，`/new` 或 `/reset` 会启动一个新的会话 id。