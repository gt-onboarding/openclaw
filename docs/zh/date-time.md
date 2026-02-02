---
title: 日期与时间
summary: "在封套、提示词、工具和连接器中处理日期和时间"
read_when:
  - 你正在修改向模型或用户展示时间戳的方式
  - 你正在调试消息或系统提示词输出中的时间格式问题
---

<div id="date-time">
  # 日期与时间
</div>

OpenClaw 默认对**消息传输时间戳使用主机本地时间**，并且只在**system prompt 中使用用户时区**。
提供方时间戳会被保留，以便工具维持其原生语义（当前时间可通过 `session_status` 获取）。

<div id="message-envelopes-local-by-default">
  ## 消息封装（默认使用本地时间）
</div>

入站消息会被封装上一个时间戳（精度为分钟）：

```
[Provider ... 2026-01-05 16:26 PST] message text
```

此封套的时间戳**默认使用主机本地时间**，与提供方所在的时区无关。

你可以更改此行为：

```json5
{
  agents: {
    defaults: {
      envelopeTimezone: "local", // "utc" | "local" | "user" | IANA 时区
      envelopeTimestamp: "on", // "on" | "off"
      envelopeElapsed: "on" // "on" | "off"
    }
  }
}
```

* `envelopeTimezone: "utc"` 使用 UTC。
* `envelopeTimezone: "local"` 使用主机时区。
* `envelopeTimezone: "user"` 使用 `agents.defaults.userTimezone`（回退到主机时区）。
* 使用显式 IANA 时区（例如 `"America/Chicago"`）来指定固定时区。
* `envelopeTimestamp: "off"` 移除 envelope 头部中的绝对时间戳。
* `envelopeElapsed: "off"` 移除耗时后缀（例如 `+2m` 这种样式）。

<div id="examples">
  ### 示例
</div>

**本地（默认）：**

```
[WhatsApp +1555 2026-01-18 00:19 PST] hello
```

**用户时区：**

```
[WhatsApp +1555 2026-01-18 00:19 CST] hello
```

**启用耗时统计：**

```
[WhatsApp +1555 +30s 2026-01-18T05:19Z] follow-up
```

<div id="system-prompt-current-date-time">
  ## 系统提示词：当前日期与时间
</div>

如果已知用户的时区，系统提示词会包含一个专门的
**当前日期与时间** 小节，其中只包含 **时区信息**（不包含具体时间 / 时间格式），
以保持 prompt 缓存的稳定性：

```
Time zone: America/Chicago
```

当智能体需要获取当前时间时，使用 `session_status` 工具；状态卡片中包含一行时间戳。

<div id="system-event-lines-local-by-default">
  ## 系统事件行（默认使用本地时区）
</div>

已排队的系统事件在插入到智能体上下文时，会带有时间戳前缀，
该时间戳使用与消息封装相同的时区设置（默认：主机本地时区）。

```
System: [2026-01-12 12:19:17 PST] Model switched.
```

<div id="configure-user-timezone-format">
  ### 配置用户时区和时间格式
</div>

```json5
{
  agents: {
    defaults: {
      userTimezone: "America/Chicago",
      timeFormat: "auto" // auto | 12 | 24（自动 | 12 小时制 | 24 小时制）
    }
  }
}
```

* `userTimezone` 为提示上下文设置 **用户本地时区**。
* `timeFormat` 控制提示中的 **12 小时 / 24 小时显示方式**。`auto` 会遵循操作系统的偏好设置。

<div id="time-format-detection-auto">
  ## 时间格式检测（自动）
</div>

当 `timeFormat: "auto"` 时，OpenClaw 会检查操作系统的偏好设置（macOS/Windows），
否则会回退为按区域设置进行格式化。检测到的值会**在每个进程内缓存**，
以避免重复的系统调用。

<div id="tool-payloads-connectors-raw-provider-time-normalized-fields">
  ## 工具负载与连接器（原始提供方时间 + 规范化字段）
</div>

通道工具会返回**提供方原生时间戳**，并附加规范化字段以保持一致性：

* `timestampMs`：纪元毫秒数（UTC）
* `timestampUtc`：ISO 8601 UTC 字符串

会保留原始提供方字段，因此不会丢失任何信息。

* Slack：来自 API 的类纪元时间字符串
* Discord：UTC ISO 时间戳
* Telegram/WhatsApp：各提供方特定的数值/ISO 时间戳

如果你需要本地时间，请在下游基于已知时区进行转换。

<div id="related-docs">
  ## 相关文档
</div>

* [系统提示](/zh/concepts/system-prompt)
* [时区](/zh/concepts/timezone)
* [消息](/zh/concepts/messages)