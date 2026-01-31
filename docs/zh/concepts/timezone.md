---
title: 时区
summary: "Agent 代理、封装（envelope）和提示词的时区处理"
read_when:
  - 你需要了解如何为模型标准化时间戳
  - 你需要为系统提示词配置用户时区
---

<div id="timezones">
  # 时区
</div>

OpenClaw 会统一时间戳，使模型只看到**单一参考时间**。

<div id="message-envelopes-local-by-default">
  ## 消息封装（默认使用本地时间）
</div>

传入消息会被封装在类似下面这样的 envelope 中：

```
[Provider ... 2026-01-05 16:26 PST] message text
```

消息封套中的时间戳**默认使用主机本地时区**，精确到分钟级。

你可以通过以下方式进行覆盖：

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
* `envelopeTimezone: "user"` 使用 `agents.defaults.userTimezone`（若未设置则回退为主机时区）。
* 使用显式 IANA 时区标识（例如 `"Europe/Vienna"`）来获得固定偏移量。
* `envelopeTimestamp: "off"` 会从 envelope 头部移除绝对时间戳。
* `envelopeElapsed: "off"` 会移除耗时后缀（例如 `+2m` 这种样式）。

<div id="examples">
  ### 示例
</div>

**本地（默认）：**

```
[Signal Alice +1555 2026-01-18 00:19 PST] hello
```

**固定时区：**

```
[Signal Alice +1555 2026-01-18 06:19 GMT+1] hello
```

**耗时：**

```
[Signal Alice +1555 +2m 2026-01-18T05:19Z] follow-up
```

<div id="tool-payloads-raw-provider-data-normalized-fields">
  ## 工具负载（原始提供方数据 + 标准化字段）
</div>

工具调用（`channels.discord.readMessages`、`channels.slack.readMessages` 等）会返回**原始提供方时间戳**。
我们还会附带标准化字段以保持一致性：

* `timestampMs`（UTC 纪元毫秒数）
* `timestampUtc`（ISO 8601 UTC 字符串）

原始提供方的字段会被保留。

<div id="user-timezone-for-the-system-prompt">
  ## 系统提示中的用户时区
</div>

设置 `agents.defaults.userTimezone` 来告知模型用户的本地时区。如果未设置，OpenClaw 会在**运行时获取主机时区**（不会修改配置）。

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } }
}
```

系统提示词包括：

* 包含本地时间和时区的 `Current Date & Time` 部分
* `Time format: 12-hour` 或 `24-hour`

你可以通过 `agents.defaults.timeFormat`（`auto` | `12` | `24`）控制系统提示中的时间显示格式。

有关完整行为和示例，请参见 [Date &amp; Time](/zh/date-time)。
