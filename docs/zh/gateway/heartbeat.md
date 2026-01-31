---
title: 心跳
summary: "心跳轮询消息和通知规则"
read_when:
  - 调整心跳频率或消息内容
  - 在为计划任务选择使用心跳还是 cron 时做出决策
---

<div id="heartbeat-gateway">
  # 心跳（Gateway）
</div>

> **心跳 vs Cron？** 请参见 [Cron vs Heartbeat](/zh/automation/cron-vs-heartbeat)，了解在什么情况下分别使用它们。

心跳会在主会话中运行**智能体的周期性轮次**，这样模型就能在不频繁打扰你的前提下，主动提示任何需要你关注的事项。

<div id="quick-start-beginner">
  ## 快速开始（入门）
</div>

1. 保持心跳功能开启（默认间隔为 `30m`，Anthropic OAuth/setup-token 为 `1h`），或设置你自己的触发频率。
2. 在智能体工作区中创建一个精简的 `HEARTBEAT.md` 清单（可选但推荐）。
3. 决定心跳消息应发送到哪里（默认是 `target: "last"`）。
4. 可选：启用心跳推理结果投递以提升透明度。
5. 可选：将心跳限制在活跃时段内（本地时间）。

示例配置：

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
        // activeHours: { start: "08:00", end: "24:00" },
        // includeReasoning: true, // 可选:同时发送单独的 `Reasoning:` 消息
      }
    }
  }
}
```

<div id="defaults">
  ## 默认值
</div>

* 间隔：`30m`（当检测到 Anthropic OAuth/setup-token 为认证模式时，则为 `1h`）。可通过 `agents.defaults.heartbeat.every` 或按智能体配置 `agents.list[].heartbeat.every` 进行设置；使用 `0m` 可禁用。
* 提示词正文（可通过 `agents.defaults.heartbeat.prompt` 配置）：
  `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`
* 心跳提示词会作为用户消息被**原样**发送。system 提示词中会包含一个 “Heartbeat” 段落，并且本次运行会在内部被标记。
* 在配置的时区内检查活跃时间段（`heartbeat.activeHours`）。在时间窗口之外，心跳会被跳过，直到下一次处于窗口内的心跳触发。

<div id="what-the-heartbeat-prompt-is-for">
  ## 心跳提示词的用途
</div>

默认提示词是刻意设计得比较宽泛的：

* **后台任务**：默认文案 “Consider outstanding tasks” 会引导智能体检查后续事项（收件箱、日历、提醒、排队中的工作），并把任何紧急内容提出来。
* **人工状态检查**：默认文案 “Checkup sometimes on your human during day time” 会促使智能体偶尔在白天发送一条类似 “anything you need?” 的轻量级问候消息，同时会根据你配置的本地时区，避免在夜间刷屏（参见 [/concepts/timezone](/zh/concepts/timezone)）。

如果你希望心跳执行非常具体的动作（例如 “check Gmail PubSub
stats” 或 “verify gateway health”），请将 `agents.defaults.heartbeat.prompt`（或
`agents.list[].heartbeat.prompt`）设置为自定义内容（会被原样发送）。

<div id="response-contract">
  ## 响应约定
</div>

* 如果没有任何需要关注的事项，回复 **`HEARTBEAT_OK`**。
* 在心跳执行期间，当 `HEARTBEAT_OK` 出现在回复的**开头或结尾**时，OpenClaw 会将其视为确认（ack）。该标记会被移除，如果剩余内容长度 **≤ `ackMaxChars`**（默认：300），则整条回复会被丢弃。
* 如果 `HEARTBEAT_OK` 出现在回复的**中间位置**，则不会被特殊处理。
* 对于告警，**不要**包含 `HEARTBEAT_OK`；只返回告警文本。

在非心跳场景中，出现在消息开头/结尾的多余 `HEARTBEAT_OK` 会被移除并记录日志；如果一条消息内容仅为 `HEARTBEAT_OK`，则会被丢弃。

<div id="config">
  ## 配置
</div>

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",           // default: 30m (0m disables)
        model: "anthropic/claude-opus-4-5",
        includeReasoning: false, // default: false (deliver separate Reasoning: message when available)
        target: "last",         // last | none | <channel id> (core or plugin, e.g. "bluebubbles")
        to: "+15551234567",     // optional channel-specific override
        prompt: "如果存在 HEARTBEAT.md(工作区上下文),请 read 它。严格遵循其内容。不要推断或重复之前对话中的旧任务。如果没有需要关注的事项,请回复 HEARTBEAT_OK。",
        ackMaxChars: 300         // max chars allowed after HEARTBEAT_OK
      }
    }
  }
}
```

<div id="scope-and-precedence">
  ### 作用范围与优先级
</div>

* `agents.defaults.heartbeat` 用于设置全局心跳行为。
* `agents.list[].heartbeat` 在其基础上进行合并；如果某个智能体配置了 `heartbeat` 块，**只有这些智能体** 会运行心跳。
* `channels.defaults.heartbeat` 为所有通道设置默认可见性。
* `channels.<channel>.heartbeat` 覆盖该通道的默认设置。
* `channels.<channel>.accounts.<id>.heartbeat`（多账号通道）覆盖对应通道级别的设置。

<div id="per-agent-heartbeats">
  ### 每个智能体的心跳
</div>

如果任意 `agents.list[]` 条目包含一个 `heartbeat` 块，**只有这些智能体**
才会执行心跳。每个智能体的块会在 `agents.defaults.heartbeat` 的基础上进行合并
（因此你可以先设置一次共享默认值，再按智能体进行覆盖）。

示例：有两个智能体，只有第二个智能体执行心跳。

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last"
      }
    },
    list: [
      { id: "main", default: true },
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "whatsapp",
          to: "+15551234567",
          prompt: "如果存在 HEARTBEAT.md(工作区上下文),请 read 它。严格遵循其内容。不要推断或重复之前对话中的旧任务。如果没有需要关注的事项,请回复 HEARTBEAT_OK。"
        }
      }
    ]
  }
}
```

<div id="field-notes">
  ### 字段说明
</div>

* `every`：心跳间隔（持续时间字符串；默认单位 = 分钟）。
* `model`：心跳运行时使用的可选模型覆写（`provider/model`）。
* `includeReasoning`：启用时，在可用情况下也会单独发送 `Reasoning:` 消息（结构与 `/reasoning on` 相同）。
* `session`：用于心跳运行的可选会话键。
  * `main`（默认）：Agent 主会话。
  * 显式会话键（从 `openclaw sessions --json` 或 [sessions CLI](/zh/cli/sessions) 中复制）。
  * 会话键格式：参见 [Sessions](/zh/concepts/session) 和 [Groups](/zh/concepts/groups)。
* `target`：
  * `last`（默认）：发送到最近一次使用的外部通道。
  * 显式通道：`whatsapp` / `telegram` / `discord` / `googlechat` / `slack` / `msteams` / `signal` / `imessage`。
  * `none`：执行心跳但**不向外部发送**。
* `to`：可选的收件人覆写（通道特定 id，例如用于 WhatsApp 的 E.164 号码或 Telegram 的聊天 id）。
* `prompt`：覆写默认提示词正文（不做合并）。
* `ackMaxChars`：在 `HEARTBEAT_OK` 之后、触发投递前所允许的最大字符数。

<div id="delivery-behavior">
  ## 投递行为
</div>

* 默认情况下，心跳在 Agent 代理的主会话中运行（`agent:<id>:<mainKey>`），
  当 `session.scope = "global"` 时则在 `global` 中运行。通过设置 `session`
  可改为特定的渠道会话（Discord/WhatsApp/等）。
* `session` 只影响运行上下文；实际投递由 `target` 和 `to` 控制。
* 若要投递到特定渠道/收件人，请设置 `target` 和 `to`。使用
  `target: "last"` 时，将使用该会话上次使用的外部渠道进行投递。
* 如果主队列繁忙，则跳过本次心跳，并在稍后重试。
* 如果 `target` 解析不到任何外部目标，运行仍会执行，但不会发送任何
  出站消息。
* 仅由心跳产生的回复**不会**保持会话存活；`updatedAt` 会被还原，使空闲过期行为保持正常。

<div id="visibility-controls">
  ## 可见性控制
</div>

默认情况下，在投递告警内容时，不会显示 `HEARTBEAT_OK` 确认消息。你可以按通道或账号分别进行调整：

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false      # Hide HEARTBEAT_OK (default)
      showAlerts: true   # Show alert messages (default)
      useIndicator: true # Emit indicator events (default)
  telegram:
    heartbeat:
      showOk: true       # Show OK acknowledgments on Telegram
  whatsapp:
    accounts:
      work:
        heartbeat:
          showAlerts: false # 禁止向此账户投递告警
```

优先顺序：按账号 → 按渠道 → 渠道默认值 → 内置默认值。

<div id="what-each-flag-does">
  ### 每个标志的作用
</div>

* `showOk`：当模型只返回 OK 类型的回复时，发送一个 `HEARTBEAT_OK` 确认。
* `showAlerts`：当模型返回非 OK 的回复时，发送告警内容。
* `useIndicator`：为 UI 状态界面发出指示事件。

如果这三个标志**全部**为 false，OpenClaw 将完全跳过本次心跳执行（不会进行模型调用）。

<div id="per-channel-vs-per-account-examples">
  ### 按渠道与按账号的示例
</div>

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false
      showAlerts: true
      useIndicator: true
  slack:
    heartbeat:
      showOk: true # 所有 Slack 账号
    accounts:
      ops:
        heartbeat:
          showAlerts: false # 仅对 ops 账号禁用告警
  telegram:
    heartbeat:
      showOk: true
```

<div id="common-patterns">
  ### 常见模式
</div>

| 目标 | 配置 |
| --- | --- |
| 默认行为（正常状态静默，仅在有告警时通知） | *(无需额外配置)* |
| 完全静默（无消息、无指示器） | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: false }` |
| 仅指示器（无消息） | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: true }` |
| 仅在一个通道中显示正常状态 | `channels.telegram.heartbeat: { showOk: true }` |

<div id="heartbeatmd-optional">
  ## HEARTBEAT.md（可选）
</div>

如果工作区中存在 `HEARTBEAT.md` 文件，默认提示词会指示
智能体去 read 它。可以把它当作你的“心跳检查清单”：体量小、内容稳定，
并且每 30 分钟包含一次是安全的。

如果 `HEARTBEAT.md` 存在但实际上是空的（只有空行和 Markdown
标题，比如 `# Heading`），OpenClaw 会跳过这次心跳运行以节省 API 调用次数。
如果文件缺失，心跳仍然会运行，由模型决定要做什么。

保持文件精简（简短的检查清单或提醒），以避免提示词膨胀。

`HEARTBEAT.md` 示例：

```md
# 心跳检查清单

- 快速扫描:收件箱中是否有紧急事项?
- 如果是白天,在没有其他待处理事项时进行轻量级检查。
- 如果任务被阻塞,记录下*缺少什么*并在下次询问 Peter。
```

<div id="can-the-agent-update-heartbeatmd">
  ### 智能体可以更新 HEARTBEAT.md 吗？
</div>

可以——前提是你让它这么做。

`HEARTBEAT.md` 只是智能体工作区中的一个普通文件，所以你可以在
正常对话中对智能体说类似的话：

* “把 `HEARTBEAT.md` 更新一下，加上一个每日日历检查。”
* “重写 `HEARTBEAT.md`，让它更短，并专注于收件箱跟进。”

如果你希望这是自动发生的，你也可以在心跳提示词中显式加入一行，比如：
“如果检查清单变得陈旧，就用一个更好的版本更新 HEARTBEAT.md。”

安全提示：不要把机密信息（API 密钥、电话号码、私有令牌）放进
`HEARTBEAT.md` —— 它会被包含到提示上下文中。

<div id="manual-wake-on-demand">
  ## 手动唤醒（按需）
</div>

你可以将一个系统事件加入队列，并立即触发一次心跳，命令如下：

```bash
openclaw system event --text "检查紧急跟进事项" --mode now
```

如果多个智能体配置了 `heartbeat`，手动唤醒会立即运行这些智能体的心跳。

使用 `--mode next-heartbeat` 来等待下一次预定的心跳触发。

<div id="reasoning-delivery-optional">
  ## 推理内容传递（可选）
</div>

默认情况下，心跳只会发送最终的“答案”负载。

如果你希望有更多可见性，可以开启：

* `agents.defaults.heartbeat.includeReasoning: true`

开启后，心跳还会额外发送一条以 `Reasoning:` 作为前缀的独立消息
（结构与 `/reasoning on` 相同）。当智能体在管理多个会话/codex 时，
你可能希望了解它为什么决定来“戳”你一下——但这也可能泄露比你期望更多的内部细节。
在群聊中建议保持关闭。

<div id="cost-awareness">
  ## 成本意识
</div>

心跳会执行完整的一轮智能体调用。更短的间隔会消耗更多的 tokens。尽量保持
`HEARTBEAT.md` 精简，如果你只需要内部状态更新，可以考虑使用更便宜的
`model`，或设置 `target: "none"`。