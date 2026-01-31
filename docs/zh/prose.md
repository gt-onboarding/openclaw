---
title: Prose
summary: "OpenProse：OpenClaw 中的 .prose 工作流、斜杠命令与状态"
read_when:
  - 当你想运行或编写 .prose 工作流时
  - 当你想启用 OpenProse 插件时
  - 当你需要了解状态存储时
---

<div id="openprose">
  # OpenProse
</div>

OpenProse 是一种可移植、以 Markdown 为中心的工作流格式，用于编排 AI 会话。在 OpenClaw 中，它以插件形式提供，该插件会安装一个 OpenProse 技能包以及一个 `/prose` 斜杠命令。程序存放在 `.prose` 文件中，并且可以启动多个具有显式控制流的子智能体。

官方网站：https://www.prose.md

<div id="what-it-can-do">
  ## 它可以做什么
</div>

* 支持显式并行的多智能体研究与综合。
* 可重复执行且审批安全的工作流（代码审查、事件分级处理、内容流水线）。
* 可复用的 `.prose` 程序，可在所有受支持的智能体运行时中运行。

<div id="install-enable">
  ## 安装并启用
</div>

内置插件默认处于禁用状态。启用 OpenProse：

```bash
openclaw plugins enable open-prose
```

启用插件后，请重启 Gateway。

开发/本地代码检出：`openclaw plugins install ./extensions/open-prose`

相关文档：[插件](/zh/plugin)、[插件清单](/zh/plugins/manifest)、[技能](/zh/tools/skills)。

<div id="slash-command">
  ## 斜杠命令
</div>

OpenProse 将 `/prose` 注册为用户可调用的技能命令。它会路由到 OpenProse VM 的指令集，并在底层使用 OpenClaw 工具。

常用命令：

```
/prose help
/prose run <file.prose>
/prose run <handle/slug>
/prose run <https://example.com/file.prose>
/prose compile <file.prose>
/prose examples
/prose update
```

<div id="example-a-simple-prose-file">
  ## 示例：简单的 `.prose` 文件
</div>

```prose
# Research + synthesis with two agents running in parallel.

input topic: "What should we research?"

agent researcher:
  model: sonnet
  prompt: "You research thoroughly and cite sources."

agent writer:
  model: opus
  prompt: "You write a concise summary."

parallel:
  findings = session: researcher
    prompt: "Research {topic}."
  draft = session: writer
    prompt: "Summarize {topic}."

session "Merge the findings + draft into a final answer."
context: { findings, draft }
```

<div id="file-locations">
  ## 文件位置
</div>

OpenProse 会在你的工作区中的 `.prose/` 目录下保存状态数据：

```
.prose/
├── .env
├── runs/
│   └── {YYYYMMDD}-{HHMMSS}-{random}/
│       ├── program.prose
│       ├── state.md
│       ├── bindings/
│       └── agents/  # 智能体
└── agents/  # 智能体
```

用户级持久化智能体位于：

```
~/.prose/agents/
```

<div id="state-modes">
  ## 状态模式
</div>

OpenProse 支持多种状态后端：

* **filesystem**（默认）：`.prose/runs/...`
* **in-context**：临时状态，适用于小型程序
* **sqlite**（实验性）：需要 `sqlite3` 可执行文件
* **postgres**（实验性）：需要 `psql` 和连接字符串

注意：

* sqlite/postgres 需要显式启用，且处于实验阶段。
* postgres 凭据会出现在子智能体日志中；请使用专用且权限最小化的数据库。

<div id="remote-programs">
  ## 远程程序
</div>

`/prose run <handle/slug>` 会映射到 `https://p.prose.md/<handle>/<slug>`。
直接提供的 URL 会被原样抓取。这会调用 `web_fetch` 工具（POST 请求则使用 `exec`）。

<div id="openclaw-runtime-mapping">
  ## OpenClaw 运行时映射
</div>

OpenProse 程序会映射到 OpenClaw 的基础原语：

| OpenProse 概念 | OpenClaw 工具 |
| --- | --- |
| 创建会话 / 任务工具 | `sessions_spawn` |
| 文件读/写 | `read` / `write` |
| Web 获取 | `web_fetch` |

如果你的工具允许列表阻止使用这些工具，OpenProse 程序将会失败。参见 [技能配置](/zh/tools/skills-config)。

<div id="security-approvals">
  ## 安全性与审批
</div>

将 `.prose` 文件视为代码对待。运行前先进行审阅。使用 OpenClaw 工具允许列表和审批关口来控制副作用。

对于具有确定性、带审批关口的工作流，请参考 [Lobster](/zh/tools/lobster)。