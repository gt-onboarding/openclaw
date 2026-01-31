---
title: 上下文
summary: "上下文：模型能看到什么、如何构建，以及如何检查它"
read_when:
  - 你想了解在 OpenClaw 中“上下文”是什么意思
  - 你正在调试为什么模型“知道”某些信息（或已经遗忘）
  - 你想减少上下文开销（/context、/status、/compact）
---

<div id="context">
  # 上下文
</div>

“上下文”是**OpenClaw 在一次运行中发送给模型的全部内容**。它受限于模型的**上下文窗口**（token 上限）。

初学者可以这样理解：

* **系统提示词**（由 OpenClaw 构建）：规则、工具、技能列表、时间/运行环境，以及注入的工作区文件。
* **对话历史**：你在此会话中的消息 + 助理的消息。
* **工具调用/结果 + 附件**：命令输出、文件 read、图片/音频等。

上下文*并不等同于*“记忆”：记忆可以存储在磁盘上并在之后重新加载；上下文是当前位于模型上下文窗口内的内容。

<div id="quick-start-inspect-context">
  ## 快速开始（检查上下文）
</div>

* `/status` → 快速查看“我的上下文窗口有多满？”以及会话设置。
* `/context list` → 查看当前注入到上下文中的内容 + 大致大小（按文件和总计）。
* `/context detail` → 更深入的拆分：每个文件、每个工具的 schema 大小、每个技能条目的大小，以及系统提示词的大小。
* `/usage tokens` → 在正常回复后附加每次回复的用量统计页脚。
* `/compact` → 将较早的对话历史汇总为一条精简记录，以释放窗口空间。

另请参阅：[斜杠命令](/zh/tools/slash-commands)、[Token 使用与费用](/zh/token-use)、[压缩](/zh/concepts/compaction)。

<div id="example-output">
  ## 示例输出
</div>

具体数值会因模型、提供方、工具策略以及工作区中的内容而有所不同。

<div id="context-list">
  ### `/context list`
</div>

```
🧠 上下文详情
工作区: <workspaceDir>
引导最大字符数/文件: 20,000 字符
沙箱: mode=non-main sandboxed=false
系统提示词 (运行): 38,412 字符 (~9,603 tok) (项目上下文 23,901 字符 (~5,976 tok))

已注入的工作区文件:
- AGENTS.md: OK | raw 1,742 字符 (~436 tok) | injected 1,742 字符 (~436 tok)
- SOUL.md: OK | raw 912 字符 (~228 tok) | injected 912 字符 (~228 tok)
- TOOLS.md: TRUNCATED | raw 54,210 字符 (~13,553 tok) | injected 20,962 字符 (~5,241 tok)
- IDENTITY.md: OK | raw 211 字符 (~53 tok) | injected 211 字符 (~53 tok)
- USER.md: OK | raw 388 字符 (~97 tok) | injected 388 字符 (~97 tok)
- HEARTBEAT.md: MISSING | raw 0 | injected 0
- BOOTSTRAP.md: OK | raw 0 字符 (~0 tok) | injected 0 字符 (~0 tok)

技能列表 (系统提示词文本): 2,184 字符 (~546 tok) (12 个技能)
工具: read, edit, write, exec, process, browser, message, sessions_send, …
工具列表 (系统提示词文本): 1,032 字符 (~258 tok)
工具模式 (JSON): 31,988 字符 (~7,997 tok) (计入上下文; 不显示为文本)
工具: (同上)

会话令牌 (已缓存): 总计 14,250 / ctx=32,000
```

<div id="context-detail">
  ### `/context detail`
</div>

```
🧠 Context breakdown (detailed)
…
Top skills (prompt entry size):
- frontend-design: 412 chars (~103 tok)
- oracle: 401 chars (~101 tok)
… (+10 more skills)

Top tools (schema size):
- browser: 9,812 chars (~2,453 tok)
- exec: 6,240 chars (~1,560 tok)
… (+N more tools)
```

<div id="what-counts-toward-the-context-window">
  ## 哪些内容计入上下文窗口
</div>

模型接收到的所有内容都会计入，其中包括：

* 系统提示词（所有部分）。
* 对话历史。
* 工具调用和工具结果。
* 附件/转录内容（图像/音频/文件）。
* 压缩摘要和裁剪产生的内容。
* 提供方的“包装器”或隐藏头信息（对用户不可见，但仍会被计入）。

<div id="how-openclaw-builds-the-system-prompt">
  ## OpenClaw 如何构建 system prompt
</div>

system prompt（系统提示词）由 **OpenClaw 管理**，并在每次运行时重新生成。它包括：

* 工具列表 + 简短描述。
* 技能列表（仅元数据；见下文）。
* 工作区位置。
* 时间（UTC + 若已配置则为转换后的用户时间）。
* 运行时元数据（host/OS/model/thinking）。
* 注入到 **Project Context** 下的工作区引导文件。

完整说明： [System Prompt](/zh/concepts/system-prompt)。

<div id="injected-workspace-files-project-context">
  ## 注入的工作区文件（项目上下文）
</div>

默认情况下，如果存在，OpenClaw 会注入一组固定的工作区文件：

* `AGENTS.md`
* `SOUL.md`
* `TOOLS.md`
* `IDENTITY.md`
* `USER.md`
* `HEARTBEAT.md`
* `BOOTSTRAP.md`（仅首次运行）

大文件会按文件使用 `agents.defaults.bootstrapMaxChars`（默认 `20000` 字符）进行截断。`/context` 会显示**原始大小 vs 注入后大小**，以及是否发生了截断。

<div id="skills-whats-injected-vs-loaded-on-demand">
  ## 技能：注入的内容 vs 按需加载
</div>

system prompt 会包含一个精简的 **技能列表**（名称 + 描述 + 位置）。这个列表本身是有实际开销的。

技能说明默认*不会*被包含在内。期望模型**只在需要时**才去 `read` 该技能的 `SKILL.md`。

<div id="tools-there-are-two-costs">
  ## 工具：有两类开销
</div>

工具会以两种方式占用上下文：

1. 系统提示中的**工具列表文本**（你在 “Tooling” 中看到的内容）。
2. **工具模式 (schema)**（JSON）。这些会被发送给模型，以便它能调用工具。即使你看不到它们的纯文本形式，它们也会计入上下文长度。

`/context detail` 会分解体积最大的工具模式 (schema)，这样你就能看出主要的占用来源。

<div id="commands-directives-and-inline-shortcuts">
  ## 命令、指令和“内联快捷方式”
</div>

斜杠命令由 Gateway 处理，大致有几种行为模式：

* **独立命令**：仅包含 `/...` 的消息会作为命令运行。
* **指令（directives）**：`/think`、`/verbose`、`/reasoning`、`/elevated`、`/model`、`/queue` 会在模型接收消息前被去掉。
  * 仅包含指令的消息会持久化会话设置。
  * 普通消息中的内联指令会作为该条消息的提示生效。
* **内联快捷方式**（仅允许列表中的发送方）：普通消息中的某些 `/...` token 可以立即运行（例如：“hey /status”），并会在模型接收剩余文本前被去掉。

详细说明：[斜杠命令](/zh/tools/slash-commands)。

<div id="sessions-compaction-and-pruning-what-persists">
  ## 会话、压缩与修剪（哪些内容会持久化）
</div>

跨多条消息会被持久化的内容取决于具体机制：

* **正常历史记录** 会保留在会话日志中，直到依据策略被压缩或修剪。
* **压缩** 会将摘要持久化写入会话日志，并保持最近的消息不变。
* **修剪** 会从一次运行的*内存中*提示上下文里移除旧的工具结果，但不会修改会话日志。

文档： [会话](/zh/concepts/session)、[压缩](/zh/concepts/compaction)、[会话修剪](/zh/concepts/session-pruning)。

<div id="what-context-actually-reports">
  ## `/context` 实际会报告什么
</div>

在可用的情况下，`/context` 会优先使用最新的**由运行构建的**系统提示报告：

* `System prompt (run)` = 从上一次嵌入式（具备工具能力）的运行中捕获，并持久化存储在会话存储中。
* `System prompt (estimate)` = 当不存在运行报告时（或者通过不会生成该报告的 CLI 后端运行时），即时计算得出。

无论哪种方式，它都会报告大小和主要贡献来源；它**不会**输出完整的系统提示或工具 schema。