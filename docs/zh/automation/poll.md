---
title: 投票
summary: "通过 Gateway 和 CLI 发送投票"
read_when:
  - 添加或修改投票支持时
  - 调试通过 CLI 或 Gateway 发送的投票时
---

<div id="polls">
  # 投票
</div>

<div id="supported-channels">
  ## 支持的渠道
</div>

- WhatsApp（网页端）
- Discord
- MS Teams（Adaptive Cards）

<div id="cli">
  ## CLI
</div>

```bash
# WhatsApp
openclaw message poll --target +15555550123 \
  --poll-question "Lunch today?" --poll-option "Yes" --poll-option "No" --poll-option "Maybe"
openclaw message poll --target 123456789@g.us \
  --poll-question "Meeting time?" --poll-option "10am" --poll-option "2pm" --poll-option "4pm" --poll-multi

# Discord
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Snack?" --poll-option "Pizza" --poll-option "Sushi"
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Plan?" --poll-option "A" --poll-option "B" --poll-duration-hours 48

# MS Teams
openclaw message poll --channel msteams --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" --poll-option "Pizza" --poll-option "Sushi"
```

选项：

* `--channel`：`whatsapp`（默认）、`discord` 或 `msteams`
* `--poll-multi`：允许多选
* `--poll-duration-hours`：仅限 Discord（省略时默认为 24 小时）


<div id="gateway-rpc">
  ## Gateway RPC
</div>

方法：`poll`

参数：

- `to`（string，必填）
- `question`（string，必填）
- `options`（string[]，必填）
- `maxSelections`（number，可选）
- `durationHours`（number，可选）
- `channel`（string，可选，默认值：`WhatsApp`）
- `idempotencyKey`（string，必填）

<div id="channel-differences">
  ## 渠道差异
</div>

- WhatsApp：2–12 个选项，`maxSelections` 必须在选项数量范围内，`durationHours` 将被忽略。
- Discord：2–10 个选项，`durationHours` 被限制在 1–768 小时之间（默认 24 小时）。`maxSelections > 1` 启用多选；Discord 不支持严格的固定选择数。
- MS Teams：使用 Adaptive Card 投票（由 OpenClaw 管理）。无原生投票 API；`durationHours` 将被忽略。

<div id="agent-tool-message">
  ## Agent 工具（消息）
</div>

使用 `message` 工具配合 `poll` 动作（`to`、`pollQuestion`、`pollOption`，可选的 `pollMulti`、`pollDurationHours`、`channel`）。

注意：Discord 不支持“精确选择 N 个”的模式；`pollMulti` 表示允许多选。
Teams 投票会渲染为 Adaptive Cards，并且要求 Gateway 保持在线，
以便将投票记录到 `~/.openclaw/msteams-polls.json` 文件中。