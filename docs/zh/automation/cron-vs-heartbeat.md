---
title: Cron 与心跳
summary: "在自动化场景下在心跳与 cron 任务之间进行选择的指南"
read_when:
  - 决定如何调度重复任务
  - 设置后台监控或通知
  - 为周期性检查优化 token 使用量
---

<div id="cron-vs-heartbeat-when-to-use-each">
  # Cron 与心跳：如何选择
</div>

心跳和 cron 任务都可以让你按计划执行任务。本指南将帮助你根据具体用例选择合适的机制。

<div id="quick-decision-guide">
  ## 快速决策指南
</div>

| 使用场景 | 建议 | 原因 |
|----------|-------------|-----|
| 每 30 分钟检查一次收件箱 | 心跳 | 与其他检查批量执行，具备上下文感知能力 |
| 在早上 9 点整发送每日报告 | Cron（隔离） | 需要精确时间点 |
| 监控日历中的即将发生的事件 | 心跳 | 非常适合周期性状态感知 |
| 每周运行一次深度分析 | Cron（隔离） | 独立任务，可使用不同模型 |
| 20 分钟后提醒我 | Cron（main，`--at`） | 一次性且时间精确 |
| 后台项目健康检查 | 心跳 | 复用现有周期循环 |

<div id="heartbeat-periodic-awareness">
  ## 心跳：周期性感知
</div>

心跳在**主会话**中按固定时间间隔运行（默认：30 分钟）。它的设计目的是让智能体检查当前状态，并显式提出任何重要事项。

<div id="when-to-use-heartbeat">
  ### 何时使用心跳
</div>

* **多次周期性检查**：与其使用 5 个独立的 cron 任务分别检查收件箱、日历、天气、通知和项目状态，不如用单个心跳一次性批量处理所有这些检查。
* **具备上下文感知的决策**：智能体拥有完整的主会话上下文，因此可以更智能地判断哪些事情紧急、哪些可以稍后处理。
* **对话连续性**：所有心跳执行都共享同一个会话，所以智能体能记住最近的对话，并自然地继续跟进。
* **低开销的监控**：一个心跳就能替代许多细碎的轮询任务。

<div id="heartbeat-advantages">
  ### 心跳的优势
</div>

* **批量执行多项检查**：一次智能体轮次即可同时检查收件箱、日历和通知。
* **减少 API 调用**：一次心跳比 5 个独立的 cron 任务成本更低。
* **具备上下文感知**：智能体知道你最近在做什么，并可据此合理排定优先级。
* **智能抑制**：如果没有需要处理的事项，智能体会回复 `HEARTBEAT_OK`，且不会发送任何消息。
* **自然的触发时机**：会根据队列负载略有漂移，但对大多数监控场景来说完全没问题。

<div id="heartbeat-example-heartbeatmd-checklist">
  ### 心跳示例：HEARTBEAT.md 检查清单
</div>

```md
# 心跳检查清单

- 检查邮件中的紧急消息
- 查看未来 2 小时内的日历事件
- 如果后台任务已完成,总结结果
- 如果空闲超过 8 小时,发送简短签到
```

智能体在每次心跳时都会读取该配置，并在单次对话轮次内处理所有项目。

<div id="configuring-heartbeat">
  ### 配置心跳
</div>

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",        // interval
        target: "last",      // where to deliver alerts
        activeHours: { start: "08:00", end: "22:00" }  // 可选
      }
    }
  }
}
```

有关完整配置，请参见 [心跳](/zh/gateway/heartbeat)。

<div id="cron-precise-scheduling">
  ## Cron：精确调度
</div>

Cron 任务在**精确的时间点**运行，并且可以在隔离的会话中独立运行，而不影响主上下文。

<div id="when-to-use-cron">
  ### 何时使用 cron
</div>

* **需要精确时间**：“每周一上午 9:00 发送此内容”（而不是“差不多 9 点左右”）。
* **独立任务**：不需要对话上下文的任务。
* **不同模型/思考方式**：涉及较重分析工作、适合使用更强大模型的任务。
* **一次性提醒**：使用 `--at` 的“20 分钟后提醒我”。
* **噪声较大/高频任务**：会让主会话历史变得杂乱的任务。
* **外部触发**：应当独立于智能体是否处于活动状态而运行的任务。

<div id="cron-advantages">
  ### Cron 优势
</div>

* **精确定时**：支持带时区的 5 字段 cron 表达式。
* **会话隔离**：在 `cron:<jobId>` 中运行，不会污染主会话历史记录。
* **模型覆写**：可为每个任务使用更便宜或更强大的模型。
* **投递控制**：可以直接投递到某个渠道；默认仍会向主会话发送一条摘要（可配置）。
* **无需智能体上下文**：即使主会话空闲或已压缩也能运行。
* **一次性任务支持**：使用 `--at` 精确定义未来时间点。

<div id="cron-example-daily-morning-briefing">
  ### Cron 示例：每日早间简报
</div>

```bash
openclaw cron add \
  --name "Morning briefing" \
  --cron "0 7 * * *" \
  --tz "America/New_York" \
  --session isolated \
  --message "Generate today's briefing: weather, calendar, top emails, news summary." \
  --model opus \
  --deliver \
  --channel whatsapp \
  --to "+15551234567"
```

该任务会在纽约时间早上 7:00 准点运行，使用 Opus 以保证质量，并将结果直接投递到 WhatsApp。

<div id="cron-example-one-shot-reminder">
  ### Cron 示例：单次提醒
</div>

```bash
openclaw cron add \
  --name "Meeting reminder" \
  --at "20m" \
  --session main \
  --system-event "Reminder: standup meeting starts in 10 minutes." \
  --wake now \
  --delete-after-run
```

完整的 CLI 参考文档请参阅 [Cron jobs](/zh/automation/cron-jobs)。

<div id="decision-flowchart">
  ## 决策流程图
</div>

```
Does the task need to run at an EXACT time?
  YES -> Use cron
  NO  -> Continue...

Does the task need isolation from main session?
  YES -> Use cron (isolated)
  NO  -> Continue...

Can this task be batched with other periodic checks?
  YES -> Use heartbeat (add to HEARTBEAT.md)
  NO  -> Use cron

Is this a one-shot reminder?
  YES -> Use cron with --at
  NO  -> Continue...

Does it need a different model or thinking level?
  YES -> Use cron (isolated) with --model/--thinking
  NO  -> Use heartbeat
```

<div id="combining-both">
  ## 组合使用两者
</div>

最高效的方案是**将两者结合使用**：

1. **心跳** 每 30 分钟批量执行一次，用于处理常规监控（收件箱、日历、通知）。
2. **Cron** 处理精确的定时计划（每日报告、每周回顾）以及一次性的提醒任务。

<div id="example-efficient-automation-setup">
  ### 示例：高效的自动化配置
</div>

**HEARTBEAT.md**（每 30 分钟检查一次）：

```md
# Heartbeat checklist
- Scan inbox for urgent emails
- Check calendar for events in next 2h
- Review any pending tasks
- Light check-in if quiet for 8+ hours
```

**Cron 任务**（精确时间调度）：

```bash
# Daily morning briefing at 7am
openclaw cron add --name "Morning brief" --cron "0 7 * * *" --session isolated --message "..." --deliver

# 每周一早上 9 点项目回顾
openclaw cron add --name "Weekly review" --cron "0 9 * * 1" --session isolated --message "..." --model opus

# One-shot reminder
openclaw cron add --name "Call back" --at "2h" --session main --system-event "Call back the client" --wake now
```

<div id="lobster-deterministic-workflows-with-approvals">
  ## Lobster：带审批的确定性工作流
</div>

Lobster 是用于需要确定性执行和显式审批的 **多步工具流水线** 的工作流运行时。
当任务不仅仅是单个智能体的一轮交互，并且你希望拥有带人工检查点、可恢复的工作流时，就使用它。

<div id="when-lobster-fits">
  ### 适合使用 Lobster 的场景
</div>

* **多步自动化**：你需要的是固定的工具调用流水线，而不是一次性的提示词。
* **审批关卡**：会产生副作用的步骤在执行前应暂停，待你审批后再继续。
* **可恢复运行**：在无需重新执行前面步骤的情况下，继续已暂停的工作流。

<div id="how-it-pairs-with-heartbeat-and-cron">
  ### 它如何与 heartbeat 和 cron 协同工作
</div>

* **Heartbeat/cron** 决定一次运行*何时*发生。
* **Lobster** 定义在运行开始后会发生*哪些步骤*。

对于定时工作流，使用 cron 或 heartbeat 触发一次调用 Lobster 的智能体轮次。
对于临时（ad-hoc）工作流，直接调用 Lobster。

<div id="operational-notes-from-the-code">
  ### 运行说明（来自代码）
</div>

* Lobster 以**本地子进程**（`lobster` CLI）形式在工具模式下运行，并返回一个 **JSON 封装对象（envelope）**。
* 如果工具返回 `needs_approval`，你需要使用 `resumeToken` 和 `approve` 标志位来恢复执行。
* 该工具是一个**可选插件**；推荐通过 `tools.alsoAllow: ["lobster"]` 叠加启用。
* 如果你传入 `lobsterPath`，它必须是一个**绝对路径**。

完整用法和示例见 [Lobster](/zh/tools/lobster)。

<div id="main-session-vs-isolated-session">
  ## 主会话 vs 隔离会话
</div>

心跳和 cron 都可以与主会话交互，但方式不同：

| | 心跳 | Cron（主会话） | Cron（隔离） |
|---|---|---|---|
| 会话 | 主会话 | 主会话（通过系统事件） | `cron:<jobId>` |
| 历史记录 | 共享 | 共享 | 每次运行全新开始 |
| 上下文 | 完整 | 完整 | 无（每次从空白开始） |
| 模型 | 主会话模型 | 主会话模型 | 可单独指定 |
| 输出 | 若结果不是 `HEARTBEAT_OK` 则发送 | 心跳提示 + 事件 | 摘要发布到主会话 |

<div id="when-to-use-main-session-cron">
  ### 何时使用主会话 cron
</div>

在以下情况下，使用 `--session main` 搭配 `--system-event`：

* 需要让提醒/事件出现在主会话上下文中
* 希望智能体在下一次心跳时基于完整上下文进行处理
* 不希望单独启动隔离运行

```bash
openclaw cron add \
  --name "Check project" \
  --every "4h" \
  --session main \
  --system-event "Time for a project health check" \
  --wake now
```

<div id="when-to-use-isolated-cron">
  ### 何时使用隔离的 cron
</div>

当你需要以下行为时，使用 `--session isolated`：

* 一个不带任何既有上下文的全新起点
* 不同的模型或思维设置
* 输出直接发送到某个渠道（摘要默认仍会发到主会话）
* 不会让主会话变得杂乱的历史记录

```bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 0" \
  --session isolated \
  --message "Weekly codebase analysis..." \
  --model opus \
  --thinking high \
  --deliver
```

<div id="cost-considerations">
  ## 成本考量
</div>

| 机制 | 成本特征 |
|-----------|--------------|
| Heartbeat | 每 N 分钟产生一轮对话；随 `HEARTBEAT.md` 大小线性增长 |
| Cron（main） | 仅向下一次心跳追加事件（不会单独产生一轮对话） |
| Cron（isolated） | 每个任务都会触发完整的智能体对话轮次；可使用更便宜的模型 |

**提示**：

* 尽量保持 `HEARTBEAT.md` 较小，以减少 token 开销。
* 将相似的检查合并到心跳中，而不是创建多个 cron 任务。
* 如果只需要内部处理，在心跳上使用 `target: "none"`。
* 对于日常例行任务，使用隔离 cron 搭配更便宜的模型。

<div id="related">
  ## 相关内容
</div>

* [Heartbeat](/zh/gateway/heartbeat) - 完整的心跳配置说明
* [Cron jobs](/zh/automation/cron-jobs) - 完整的 Cron CLI 和 API 参考
* [System](/zh/cli/system) - 系统事件和心跳控制