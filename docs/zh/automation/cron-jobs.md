---
title: Cron 任务
summary: "用于 Gateway 调度器的 Cron 任务与唤醒机制"
read_when:
  - 调度后台任务或唤醒操作时
  - 编排需要与心跳一起或并行运行的自动化任务时
  - 为计划任务在心跳与 cron 之间做选择时
---

<div id="cron-jobs-gateway-scheduler">
  # Cron 任务（Gateway 调度器）
</div>

> **Cron 与心跳的区别？** 请参见 [Cron vs Heartbeat](/zh/automation/cron-vs-heartbeat)，了解各自的适用场景。

Cron 是 Gateway 内置的调度器。它会保存并持久化任务，在正确的时间唤醒智能体，并且可以选择性地将输出发送回某个聊天会话。

如果你想要 *「每天早上运行这个」* 或 *「在 20 分钟后触发一下智能体」*，就使用 cron。

<div id="tldr">
  ## TL;DR
</div>

* Cron 在 **Gateway 内部运行**（不是在模型内部运行）。
* 任务持久化存储在 `~/.openclaw/cron/` 下，因此重启不会丢失调度信息。
* 两种执行方式：
  * **主会话**：将一个系统事件入队，然后在下一次心跳时执行。
  * **隔离执行**：在 `cron:<jobId>` 中运行一个专用的智能体轮次，可选地发送输出结果。
* 唤醒是“一等公民”：任务可以请求“立即唤醒”，而不是等到“下一次心跳”。

<div id="beginner-friendly-overview">
  ## 面向初学者的概览
</div>

可以把 cron 任务理解为：**什么时候** 运行 + **要做什么**。

1. **选择调度计划**
   * 一次性提醒 → `schedule.kind = "at"`（CLI：`--at`）
   * 重复任务 → `schedule.kind = "every"` 或 `schedule.kind = "cron"`
   * 如果你的 ISO 时间戳省略了时区，则会按 **UTC** 处理。

2. **选择在哪里运行**
   * `sessionTarget: "main"` → 在下一次心跳时，以主会话上下文运行。
   * `sessionTarget: "isolated"` → 在 `cron:<jobId>` 中运行一个独立智能体轮次。

3. **选择载荷类型**
   * 主会话 → `payload.kind = "systemEvent"`
   * 独立会话 → `payload.kind = "agentTurn"`

可选：`deleteAfterRun: true` 会在一次性任务成功执行后，将其从存储中删除。

<div id="concepts">
  ## 基本概念
</div>

<div id="jobs">
  ### 任务
</div>

一个 cron 任务是一条存储的记录，包含：

* 一个**调度计划**（何时运行），
* 一个**任务载荷**（要做什么），
* 可选的**投递配置**（将输出发送到哪里），
* 可选的**智能体绑定**（`agentId`）：在指定智能体下运行该任务；如果
  缺失或无效，Gateway 将回退到默认智能体。

任务通过稳定的 `jobId` 标识（供 CLI/Gateway API 使用）。
在智能体工具调用中，以 `jobId` 为准；为兼容性也接受旧的 `id`。
任务可以通过 `deleteAfterRun: true` 在成功的一次性运行后自动删除。

<div id="schedules">
  ### 调度计划
</div>

Cron 支持三种调度类型：

* `at`：一次性时间戳（自 Unix 纪元以来的毫秒数）。Gateway 接受 ISO 8601 格式并会转换为 UTC。
* `every`：固定时间间隔（毫秒）。
* `cron`：包含 5 个字段的 cron 表达式，可选 IANA 时区。

Cron 表达式由 `croner` 处理。如果省略时区，则使用 Gateway 所在主机的本地时区。

<div id="main-vs-isolated-execution">
  ### 主会话执行与隔离执行
</div>

<div id="main-session-jobs-system-events">
  #### 主会话作业（系统事件）
</div>

主作业会入队一个系统事件，并可选地唤醒心跳运行器。
它们必须使用 `payload.kind = "systemEvent"`。

* `wakeMode: "next-heartbeat"`（默认）：事件会等待下一次计划的心跳。
* `wakeMode: "now"`：事件会触发一次立即执行的心跳。

当你需要常规的心跳提示词 + 主会话上下文时，这是最合适的选择。
参见 [Heartbeat](/zh/gateway/heartbeat)。

<div id="isolated-jobs-dedicated-cron-sessions">
  #### 隔离任务（专用 cron 会话）
</div>

隔离任务会在会话 `cron:&lt;jobId&gt;` 中运行一次专用的智能体轮次。

关键行为：

* 为了便于追踪，提示词会加上前缀 `[cron:&lt;jobId&gt; &lt;job name&gt;]`。
* 每次运行都会启动一个**全新的会话 ID**（不会继承之前的对话）。
* 一份摘要会被发送到主会话（前缀为 `Cron`，可配置）。
* `wakeMode: "now"` 会在发送摘要后立刻触发一次心跳。
* 如果 `payload.deliver: true`，输出会被投递到某个通道；否则只在内部保留。

将隔离任务用于嘈杂、频繁，或不应刷屏主会话历史的「后台杂务」。

<div id="payload-shapes-what-runs">
  ### 负载结构（实际运行的内容）
</div>

支持两种负载类型：

* `systemEvent`：仅限主会话，通过心跳提示词路由。
* `agentTurn`：仅限隔离会话，运行一次专用的智能体轮次调用。

通用的 `agentTurn` 字段：

* `message`：必需的文本提示词。
* `model` / `thinking`：可选覆盖配置（见下文）。
* `timeoutSeconds`：可选的超时覆盖。
* `deliver`：为 `true` 时，将输出发送到某个渠道目标。
* `channel`：`last` 或某个特定渠道。
* `to`：渠道特定的目标（电话/聊天/频道 id）。
* `bestEffortDeliver`：如果投递失败，避免将整个任务标记为失败。

隔离选项（仅用于 `session=isolated`）：

* `postToMainPrefix`（CLI：`--post-prefix`）：发送到主会话的系统事件前缀。
* `postToMainMode`：`summary`（默认）或 `full`。
* `postToMainMaxChars`：当 `postToMainMode=full` 时的最大字符数（默认 8000）。

<div id="model-and-thinking-overrides">
  ### 模型和思维级别覆盖
</div>

独立作业（`agentTurn`）可以覆盖模型和思维级别：

* `model`：提供方/模型字符串（例如 `anthropic/claude-sonnet-4-20250514`），或别名（例如 `opus`）
* `thinking`：思维级别（`off`、`minimal`、`low`、`medium`、`high`、`xhigh`；仅适用于 GPT-5.2 和 Codex 模型）

注意：你也可以在主会话作业上设置 `model`，但这会更改共享的主会话模型。建议仅对独立作业使用模型覆盖，以避免出现意外的上下文切换。

生效优先级：

1. 作业负载中的覆盖（最高）
2. 特定 hook 的默认值（例如 `hooks.gmail.model`）
3. Agent 代理配置中的默认值

<div id="delivery-channel-target">
  ### 投递（channel + target）
</div>

独立的任务可以将输出投递到某个 channel。任务负载可以指定：

* `channel`: `whatsapp` / `telegram` / `discord` / `slack` / `mattermost` (plugin) / `signal` / `imessage` / `last`
* `to`: 按照各个 channel 定义的收件目标

如果省略 `channel` 或 `to`，cron 会回退到主会话的 “last route”
（Agent 代理上次回复发送到的那个位置）。

投递注意事项：

* 如果设置了 `to`，即使省略了 `deliver`，cron 也会自动投递 Agent 代理的最终输出。
* 当你希望使用 last-route 投递但不显式提供 `to` 时，请使用 `deliver: true`。
* 即使存在 `to`，如果希望输出仅在内部保留，则使用 `deliver: false`。

目标格式提示：

* Slack/Discord/Mattermost (plugin) 目标应使用显式前缀（例如 `channel:<id>`、`user:<id>`）以避免歧义。
* Telegram 主题应使用 `:topic:` 形式（见下文）。

<div id="telegram-delivery-targets-topics-forum-threads">
  #### Telegram 投递目标（主题 / 论坛话题）
</div>

Telegram 通过 `message_thread_id` 支持论坛话题。对于 cron 投递，你可以在 `to` 字段中编码
主题/话题信息：

* `-1001234567890`（仅聊天 id）
* `-1001234567890:topic:123`（推荐：显式主题标记）
* `-1001234567890:123`（简写：数字后缀）

带前缀的目标如 `telegram:...` / `telegram:group:...` 也支持：

* `telegram:group:-1001234567890:topic:123`

<div id="storage-history">
  ## 存储与历史记录
</div>

* 作业存储：`~/.openclaw/cron/jobs.json`（由 Gateway 管理的 JSON）。
* 运行历史：`~/.openclaw/cron/runs/<jobId>.jsonl`（JSONL，自动清理）。
* 自定义存储路径：在配置中使用 `cron.store` 设置。

<div id="configuration">
  ## 配置
</div>

```json5
{
  cron: {
    enabled: true, // 默认 true
    store: "~/.openclaw/cron/jobs.json",
    maxConcurrentRuns: 1 // 默认 1
  }
}
```

完全禁用 cron：

* `cron.enabled: false`（配置项）
* `OPENCLAW_SKIP_CRON=1`（环境变量）

<div id="cli-quickstart">
  ## CLI 快速开始
</div>

一次性提醒（UTC ISO，成功后自动删除）：

```bash
openclaw cron add \
  --name "Send reminder" \
  --at "2026-01-12T18:00:00Z" \
  --session main \
  --system-event "Reminder: submit expense report." \
  --wake now \
  --delete-after-run
```

单次提醒（主会话，立即唤醒）：

```bash
openclaw cron add \
  --name "Calendar check" \
  --at "20m" \
  --session main \
  --system-event "Next heartbeat: check calendar." \
  --wake now
```

周期性独立任务（发送到 WhatsApp）：

```bash
openclaw cron add \
  --name "Morning status" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize inbox + calendar for today." \
  --deliver \
  --channel whatsapp \
  --to "+15551234567"
```

周期性隔离作业（发送到某个 Telegram 话题）：

```bash
openclaw cron add \
  --name "Nightly summary (topic)" \
  --cron "0 22 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize today; send to the nightly topic." \
  --deliver \
  --channel telegram \
  --to "-1001234567890:topic:123"
```

带有模型和思维覆写的隔离作业：

````bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 1" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Weekly deep analysis of project progress." \
  --model "opus" \
  --thinking high \
  --deliver \
  --channel whatsapp \
  --to "+15551234567"

Agent 代理选择(多智能体设置):
```bash
# 将任务固定到智能体 "ops"(如果该智能体缺失则回退到默认智能体)
openclaw cron add --name "Ops sweep" --cron "0 6 * * *" --session isolated --message "Check ops queue" --agent ops

# 切换或清除现有任务上的智能体
openclaw cron edit <jobId> --agent ops
openclaw cron edit <jobId> --clear-agent
````

````

手动运行(调试):
```bash
openclaw cron run <jobId> --force
````

编辑现有任务（以补丁方式更新字段）：

```bash
openclaw cron edit <jobId> \
  --message "Updated prompt" \
  --model "opus" \
  --thinking low
```

执行历史：

```bash
openclaw cron runs --id <jobId> --limit 50
```

无需创建计划任务的即时系统事件：

```bash
openclaw system event --mode now --text "Next heartbeat: check battery."
```

<div id="gateway-api-surface">
  ## Gateway API 范围
</div>

* `cron.list`, `cron.status`, `cron.add`, `cron.update`, `cron.remove`
* `cron.run`（强制执行或到期执行）, `cron.runs`
  对于无需关联作业的即时系统事件，请使用 [`openclaw system event`](/zh/cli/system)。

<div id="troubleshooting">
  ## 故障排查
</div>

<div id="nothing-runs">
  ### “没有任务在运行”
</div>

* 检查是否启用了 cron：`cron.enabled` 和 `OPENCLAW_SKIP_CRON`。
* 检查 Gateway 是否在持续运行（cron 在 Gateway 进程内执行）。
* 对于 `cron` 调度任务：确认时区参数（`--tz`）与宿主机时区是否一致。

<div id="telegram-delivers-to-the-wrong-place">
  ### Telegram 投递到了错误的地方
</div>

* 对于论坛主题，请使用 `-100…:topic:<id>`，这样更明确且毫无歧义。
* 如果你在日志或存储的“last route”目标中看到 `telegram:...` 前缀，这是正常的；
  cron 投递会接受它们，并且仍然能正确解析主题 ID。