---
title: Lobster
summary: "适用于 OpenClaw 的强类型工作流运行时，支持可恢复的审批关卡。"
description: 适用于 OpenClaw 的强类型工作流运行时——带有审批关卡的可组合流水线。
read_when:
  - 你需要具有显式审批的确定性多步工作流
  - 你需要在无需重新执行前面步骤的情况下恢复工作流
---

<div id="lobster">
  # Lobster
</div>

Lobster 是一个工作流 shell，它让 OpenClaw 能够将多步工具序列作为单一、确定性的操作来运行，并在流程中设置明确的审批检查点。

<div id="hook">
  ## Hook
</div>

你的助手可以构建管理它自己的工具。你只要提出一个工作流需求，30 分钟后就能得到一个 CLI 加上一整套可以通过一次调用跑完的流水线。Lobster 就是缺失的关键一环：确定性的流水线、显式审批，以及可恢复的状态。

<div id="why">
  ## 为什么
</div>

在当下的复杂工作流中，通常需要大量往返的工具调用。每次调用都会消耗 token，而且每一步都需要由 LLM 来编排。Lobster 将这种编排移入一个带类型的运行时环境中：

* **一次调用替代多次调用**：OpenClaw 只需运行一次 Lobster 工具调用，就能获得结构化结果。
* **内置审批机制**：带有副作用的操作（发送邮件、发布评论）会暂停工作流，直到被显式批准。
* **可恢复**：被暂停的工作流会返回一个令牌；批准后即可恢复执行，而无需重新跑一遍所有步骤。

<div id="why-a-dsl-instead-of-plain-programs">
  ## 为什么要用 DSL 而不是普通程序？
</div>

Lobster 被刻意设计得很小。目标不是“造一门新语言”，而是提供一个可预测、对 AI 友好的流水线规范，并把审批和恢复令牌作为一等公民内建进去。

* **内建审批/恢复机制**：普通程序可以提示人类介入，但如果想用一个持久令牌来*暂停并恢复*执行，就必须由你自己发明并实现整套运行时。
* **确定性 + 可审计性**：流水线是数据，因此很容易记录日志、做 diff、重放和审查。
* **为 AI 收缩的能力暴露面**：极小的语法 + JSON 传递减少“创意性”代码路径，使校验变得现实可行。
* **安全策略内嵌**：超时、输出上限、沙箱检查和允许列表由运行时统一强制执行，而不是交给每个脚本各自实现。
* **依然可编程**：每一步都可以调用任意 CLI 或脚本。如果你想用 JS/TS，可以从代码生成 `.lobster` 文件。

<div id="how-it-works">
  ## 工作原理
</div>

OpenClaw 会以 **工具模式** 启动本地的 `lobster` CLI，并从 stdout 中解析出一个 JSON 封装结构。
如果流水线因等待审批而暂停，该工具会返回一个 `resumeToken`，用于稍后继续执行。

<div id="pattern-small-cli-json-pipes-approvals">
  ## 模式：小型 CLI + JSON 管道 + 审批
</div>

编写以 JSON 为输入输出的小型命令，然后将它们通过管道串联起来，在一次 Lobster 调用中执行。（下面是示例命令名——请替换为你自己的命令。）

```bash
inbox list --json
inbox categorize --json
inbox apply --json
```

```json
{
  "action": "run",
  "pipeline": "exec --json --shell 'inbox list --json' | exec --stdin json --shell 'inbox categorize --json' | exec --stdin json --shell 'inbox apply --json' | approve --preview-from-stdin --limit 5 --prompt 'Apply changes?'",
  "timeoutMs": 30000
}
```

如果流水线请求审批，请使用该令牌继续：

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

AI 触发工作流，Lobster 执行各个步骤。审批关口让所有副作用都保持显式且可审计。

示例：将输入项映射为工具调用：

```bash
gog.gmail.search --query 'newer_than:1d' \
  | openclaw.invoke --tool message --action send --each --item-key message --args-json '{"provider":"telegram","to":"..."}'
```

<div id="json-only-llm-steps-llm-task">
  ## 仅 JSON 格式的 LLM 步骤（llm-task）
</div>

对于需要**结构化 LLM 步骤**的工作流，启用可选的 `llm-task` 插件工具，并在 Lobster 中调用它。这样既能保持工作流的确定性，又能让你使用模型进行分类／总结／起草内容。

启用该工具：

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  },
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```

在流水线中使用它：

```lobster
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "根据输入的邮件,返回意图和草稿。",
  "input": { "subject": "Hello", "body": "Can you help?" },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```

有关详细信息和配置选项，请参阅 [LLM Task](/zh/tools/llm-task)。

<div id="workflow-files-lobster">
  ## 工作流文件（.lobster）
</div>

Lobster 可以运行包含 `name`、`args`、`steps`、`env`、`condition` 和 `approval` 字段的 YAML/JSON 格式工作流文件。在 OpenClaw 的工具调用中，将 `pipeline` 设置为该文件的路径。

```yaml
name: inbox-triage
args:
  tag:
    default: "family"
steps:
  - id: collect
    command: inbox list --json
  - id: categorize
    command: inbox categorize --json
    stdin: $collect.stdout
  - id: approve
    command: inbox apply --approve
    stdin: $categorize.stdout
    approval: required
  - id: execute
    command: inbox apply --execute
    stdin: $categorize.stdout
    condition: $approve.approved
```

Notes:

* `stdin: $step.stdout` 和 `stdin: $step.json` 会将前一个步骤的输出传递进来。
* `condition`（或 `when`）可以基于 `$step.approved` 对步骤进行条件控制。

<div id="install-lobster">
  ## 安装 Lobster
</div>

在运行 OpenClaw Gateway 的**同一台主机**上安装 Lobster CLI（参见 [Lobster 仓库](https://github.com/openclaw/lobster)），并确保可以在 `PATH` 中找到 `lobster`。
如果你想使用自定义的二进制文件路径，请在工具调用中传入一个**绝对路径**形式的 `lobsterPath`。

<div id="enable-the-tool">
  ## 启用该工具
</div>

Lobster 是一个**可选的**插件工具（默认未启用）。

推荐做法（可叠加、安全）：

```json
{
  "tools": {
    "alsoAllow": ["lobster"]
  }
}
```

或者按智能体划分：

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": {
          "alsoAllow": ["lobster"]
        }
      }
    ]
  }
}
```

除非你确实打算在受限的允许列表模式下运行，否则不要使用 `tools.allow: ["lobster"]`。

注意：对于可选插件，允许列表是“主动启用”（opt‑in）的。如果你的允许列表只包含
插件工具（例如 `lobster`），OpenClaw 会保持核心工具处于启用状态。若要限制核心
工具的使用，也需要在允许列表中包含你希望允许的核心工具或工具组。

<div id="example-email-triage">
  ## 示例：邮件分拣
</div>

在没有 Lobster 的情况下：

```
User: "Check my email and draft replies"
→ openclaw calls gmail.list
→ LLM summarizes
→ User: "draft replies to #2 and #5"
→ LLM drafts
→ User: "send #2"
→ openclaw calls gmail.send
(repeat daily, no memory of what was triaged)
```

在 Lobster 中：

```json
{
  "action": "run",
  "pipeline": "email.triage --limit 20",
  "timeoutMs": 30000
}
```

返回一个 JSON 封装（已截断）：

```json
{
  "ok": true,
  "status": "needs_approval",
  "output": [{ "summary": "5 need replies, 2 need action" }],
  "requiresApproval": {
    "type": "approval_request",
    "prompt": "发送 2 封草稿回复?",
    "items": [],
    "resumeToken": "..."
  }
}
```

用户批准后 → 继续：

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

单一工作流。行为确定。安全可靠。

<div id="tool-parameters">
  ## 工具参数
</div>

<div id="run">
  ### `run`
</div>

在工具模式下运行流水线。

```json
{
  "action": "run",
  "pipeline": "gog.gmail.search --query 'newer_than:1d' | email.triage",
  "cwd": "/path/to/workspace",
  "timeoutMs": 30000,
  "maxStdoutBytes": 512000
}
```

使用参数运行工作流文件：

```json
{
  "action": "run",
  "pipeline": "/path/to/inbox-triage.lobster",
  "argsJson": "{\"tag\":\"family\"}"
}
```

<div id="resume">
  ### `resume`
</div>

在获批后继续执行已中断的工作流。

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

<div id="optional-inputs">
  ### 可选输入
</div>

* `lobsterPath`：Lobster 可执行文件的绝对路径（省略则使用 `PATH`）。
* `cwd`：流水线的工作目录（默认为当前进程的工作目录）。
* `timeoutMs`：如果子进程运行时间超过该时长则终止（默认：20000）。
* `maxStdoutBytes`：如果子进程的 stdout 超过该大小则终止（默认：512000）。
* `argsJson`：传递给 `lobster run --args-json` 的 JSON 字符串（仅用于 workflow 文件）。

<div id="output-envelope">
  ## 输出封装（envelope）
</div>

Lobster 会返回一个带有以下三种状态之一的 JSON 封装：

* `ok` → 成功完成
* `needs_approval` → 已暂停；需要提供 `requiresApproval.resumeToken` 才能恢复执行
* `cancelled` → 被明确拒绝或取消

该工具会同时在 `content`（美化后的 JSON）和 `details`（原始对象）中返回该封装。

<div id="approvals">
  ## 审批
</div>

如果存在 `requiresApproval`，检查该提示内容并做出决定：

* `approve: true` → 恢复并继续执行会产生副作用的操作
* `approve: false` → 取消并结束该工作流

使用 `approve --preview-from-stdin --limit N` 为审批请求附加 JSON 预览，而无需编写自定义 jq/heredoc 胶水代码。恢复令牌现在更加精简：Lobster 会在其状态目录下存储工作流的恢复状态，并返回一个较小的令牌键。

<div id="openprose">
  ## OpenProse
</div>

OpenProse 非常适合与 Lobster 搭配使用：先使用 `/prose` 编排多智能体的前期准备流程，然后运行 Lobster 流水线以实现确定性的审批。如果某个 Prose 程序需要 Lobster，可通过 `tools.subagents.tools` 为子智能体启用 `lobster` 工具。参见 [OpenProse](/zh/prose)。

<div id="safety">
  ## 安全性
</div>

* **仅限本地子进程** — 插件本身不会发起任何网络请求。
* **不处理敏感凭据** — Lobster 不管理 OAuth；它调用负责这部分的 OpenClaw 工具。
* **沙箱感知** — 当工具上下文处于沙箱中时会自动禁用。
* **强化防护** — 如果指定了 `lobsterPath`，则必须为绝对路径；会强制执行超时和输出上限。

<div id="troubleshooting">
  ## 故障排查
</div>

* **`lobster subprocess timed out`** → 增加 `timeoutMs`，或将较长的 pipeline 拆分。
* **`lobster output exceeded maxStdoutBytes`** → 提高 `maxStdoutBytes` 或减少输出量。
* **`lobster returned invalid JSON`** → 确保 pipeline 以 tool 模式运行且仅输出 JSON。
* **`lobster failed (code …)`** → 在终端中以相同参数运行该 pipeline 并查看 stderr。

<div id="learn-more">
  ## 进一步了解
</div>

* [插件](/zh/plugin)
* [插件工具开发](/zh/plugins/agent-tools)

<div id="case-study-community-workflows">
  ## 案例研究：社区工作流
</div>

一个公开示例：一个“第二大脑”CLI + Lobster 流水线，用来管理三个 Markdown 知识库（个人、伴侣、共享）。该 CLI 输出用于统计、收件箱列表和过期项扫描的 JSON；Lobster 将这些命令串联成诸如 `weekly-review`、`inbox-triage`、`memory-consolidation` 和 `shared-task-sync` 之类的工作流，每个都有审批关卡。在 AI 可用时由其负责判断（分类），在不可用时则回退到确定性规则。

* 讨论帖：https://x.com/plattenschieber/status/2014508656335770033
* 仓库：https://github.com/bloomedai/brain-cli