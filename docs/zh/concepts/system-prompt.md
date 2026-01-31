---
title: 系统提示词
summary: "OpenClaw 系统提示词包含的内容以及其构成方式"
read_when:
  - 编辑系统提示词文本、工具列表或时间/心跳部分时
  - 更改工作区引导启动或技能注入行为时
---

<div id="system-prompt">
  # 系统提示词
</div>

OpenClaw 会为每次智能体运行构建一个自定义系统提示词。该提示词 **由 OpenClaw 统一管理（OpenClaw-owned）**，而不是使用 p-coding-agent 的默认提示词。

该提示词由 OpenClaw 组装，并注入到每一次智能体运行中。

<div id="structure">
  ## 结构
</div>

该提示刻意保持紧凑，并使用固定的分段结构：

* **Tooling**：当前可用工具列表及其简要说明。
* **Skills**（如果可用）：告知模型如何按需加载技能说明。
* **OpenClaw Self-Update**：如何运行 `config.apply` 和 `update.run`。
* **Workspace**：工作目录（`agents.defaults.workspace`）。
* **Documentation**：OpenClaw 文档的本地路径（代码仓库或 npm 包），以及在何种情况下应阅读这些文档。
* **Workspace Files (injected)**：表明引导文件已在下方注入。
* **Sandbox**（启用时）：指示当前处于沙箱运行环境、沙箱路径，以及是否允许以提升权限执行。
* **Current Date &amp; Time**：用户本地时间、时区和时间格式。
* **Reply Tags**：受支持提供方的可选回复标签语法。
* **Heartbeats**：心跳提示词和确认（ack）行为。
* **Runtime**：主机、操作系统、Node.js 版本、模型、仓库根目录（若已检测）、推理等级（单行）。
* **Reasoning**：当前可见性级别 + /reasoning 切换提示。

<div id="prompt-modes">
  ## 提示模式
</div>

OpenClaw 可以为子智能体渲染更精简的系统提示词。运行时会为每次执行设置一个
`promptMode`（这不是面向用户的配置项）：

* `full`（默认）：包含上面的所有部分。
* `minimal`：用于子智能体；省略 **Skills**、**Memory Recall**、**OpenClaw
  Self-Update**、**Model Aliases**、**User Identity**、**Reply Tags**、
  **Messaging**、**Silent Replies** 和 **Heartbeats**。Tooling、Workspace、
  Sandbox、当前日期和时间（如果已知）、Runtime 以及注入的上下文仍然可用。
* `none`：仅返回基础身份行。

当 `promptMode=minimal` 时，额外注入的提示会被标记为 **子智能体上下文**，
而不是 **群聊上下文**。

<div id="workspace-bootstrap-injection">
  ## 工作区引导注入
</div>

引导文件会被裁剪后追加到 **Project Context** 下，这样模型无需显式调用 read 也能看到身份和配置上下文：

* `AGENTS.md`
* `SOUL.md`
* `TOOLS.md`
* `IDENTITY.md`
* `USER.md`
* `HEARTBEAT.md`
* `BOOTSTRAP.md`（仅在全新工作区中）

大文件会被截断并带有标记。单个文件的最大大小由
`agents.defaults.bootstrapMaxChars` 控制（默认：20000）。缺失的文件则会注入一条简短的缺失文件标记。

内部 hook 可以通过 `agent:bootstrap` 拦截这一步，以修改或替换被注入的引导文件（例如把 `SOUL.md` 替换为另一个 persona 配置）。

若要检查每个注入文件各自贡献了多少上下文（原始内容 vs 注入内容、截断情况，以及工具 schema 的额外开销），使用 `/context list` 或 `/context detail`。参见 [Context](/zh/concepts/context)。

<div id="time-handling">
  ## 时间处理
</div>

当已知用户时区时，系统提示词中会包含一个单独的 **Current Date &amp; Time**（当前日期与时间）部分。为了保持提示词缓存的稳定性，现在其中只包含 **time zone**（时区）（不再包含动态时钟或时间格式）。

当智能体需要获取当前时间时，请使用 `session_status`；返回的状态卡片中会包含一行时间戳。

配置方式：

* `agents.defaults.userTimezone`
* `agents.defaults.timeFormat` (`auto` | `12` | `24`)

详细行为说明参见 [Date &amp; Time](/zh/date-time)。

<div id="skills">
  ## 技能
</div>

当存在符合条件的技能时，OpenClaw 会注入一个精简的 **可用技能列表**
（`formatSkillsForPrompt`），其中包含每个技能的 **文件路径**。该
提示会指示模型使用 `read` 在列出的路径位置（工作区、托管或内置）
加载 SKILL.md。如果没有符合条件的技能，则会省略技能部分。

```
<available_skills>
  <skill>
    <name>...</name>
    <description>...</description>
    <location>...</location>
  </skill>
</available_skills>
```

这样既能让基础提示保持精简，又能实现对特定技能的针对性使用。

<div id="documentation">
  ## 文档
</div>

如果可用，system prompt 会包含一个 **文档** 部分，该部分指向本地 OpenClaw 文档目录（要么是代码仓库工作区中的 `docs/`，要么是随附的 npm
包文档），并同时给出公开镜像站点、源代码仓库、社区 Discord，以及用于技能发现的 ClawHub (https://clawhub.com)。该 prompt 会指示模型在处理 OpenClaw
的行为、命令、配置或架构时优先查阅本地文档，并在可能的情况下自行运行
`openclaw status`（仅在无法自行访问时才询问用户）。