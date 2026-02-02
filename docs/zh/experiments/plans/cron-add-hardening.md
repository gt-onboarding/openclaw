---
title: Cron Add 安全加固
summary: "加固 cron.add 的输入处理，对齐 schema，并改进 cron UI/智能体工具"
owner: "openclaw"
status: "complete"
last_updated: "2026-01-05"
---

<div id="cron-add-hardening-schema-alignment">
  # Cron 添加操作加固与 Schema 对齐
</div>

<div id="context">
  ## 上下文
</div>

近期的 Gateway 日志显示，多次 `cron.add` 调用因参数无效而失败（缺少 `sessionTarget`、`wakeMode`、`payload`，以及格式错误的 `schedule`）。这表明至少有一个客户端（很可能是 Agent 工具调用路径）正在发送被包装或仅部分指定的任务负载。另一个问题是，TypeScript 中的 cron 提供方枚举、Gateway schema、CLI 标志和 UI 表单类型之间存在不一致，并且在 `cron.status` 上存在 UI 字段不匹配（UI 期望 `jobCount`，而 Gateway 实际返回的是 `jobs`）。

<div id="goals">
  ## 目标
</div>

* 通过规范常见封装 payload 并推断缺失的 `kind` 字段，避免 `cron.add` 的 INVALID&#95;REQUEST 垃圾日志刷屏。
* 在 Gateway schema、cron 类型、CLI 文档和 UI 表单中统一 cron 提供方列表。
* 将 Agent 代理的 cron 工具 schema 明确化，使 LLM 能生成正确的作业 payload。
* 修复 Control UI 中 cron 状态的作业数量显示问题。
* 添加测试以覆盖规范化逻辑和工具行为。

<div id="non-goals">
  ## 非目标
</div>

* 不会更改 cron 的调度语义或任务执行行为。
* 不会添加新的调度类型或 cron 表达式解析机制。
* 除必要的字段修复外，不会对 cron 的 UI/UX 进行整体改造。

<div id="findings-current-gaps">
  ## 发现（当前差距）
</div>

* Gateway 中的 `CronPayloadSchema` 排除了 `signal` 和 `imessage`，但 TypeScript 类型中包含它们。
* Control UI 的 CronStatus 期望字段为 `jobCount`，但 Gateway 返回的是 `jobs`。
* Agent 代理的 cron 工具 schema 允许任意 `job` 对象，从而可能接收格式错误的输入。
* Gateway 对 `cron.add` 进行严格校验且不做规范化处理，因此封装过的 payload 会校验失败。

<div id="what-changed">
  ## 变更内容
</div>

* `cron.add` 和 `cron.update` 现在会规范化常见的包装结构，并推断缺失的 `kind` 字段。
* Agent 代理的 cron 工具 schema 现已与 Gateway 的 schema 保持一致，从而减少无效负载。
* 提供方枚举在 Gateway、CLI、UI 和 macOS 选择器之间已对齐。
* Control UI 现在使用 Gateway 的 `jobs` 计数字段来显示状态。

<div id="current-behavior">
  ## 当前行为
</div>

* **规范化：** 已包装的 `data`/`job` 负载会被解包；在安全的情况下，会推断 `schedule.kind` 和 `payload.kind`。
* **默认值：** 当缺失时，会为 `wakeMode` 和 `sessionTarget` 应用安全的默认值。
* **提供方：** Discord/Slack/Signal/iMessage 现在会在 CLI/UI 中以一致的方式呈现。

有关规范化后的结构和示例，请参见 [Cron jobs](/zh/automation/cron-jobs)。

<div id="verification">
  ## 验证
</div>

* 监控 Gateway 日志，确认 `cron.add` 的 INVALID&#95;REQUEST 错误有所减少。
* 确认 Control UI 中的 cron 状态在刷新后显示任务数量。

<div id="optional-follow-ups">
  ## 可选后续事项
</div>

* 手动在 Control UI 上做一次冒烟检查：为每个提供方添加一个 cron 任务，并核对状态任务数量。

<div id="open-questions">
  ## 未决问题
</div>

* `cron.add` 是否应该接受来自客户端的显式 `state`（目前在 schema 中不被允许）？
* 我们是否应该允许将 `webchat` 作为显式的投递提供方（目前在投递解析中会被过滤）？