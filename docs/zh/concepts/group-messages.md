---
title: 群组消息
summary: "WhatsApp 群组消息处理行为与配置（mentionPatterns 在各个渠道间共享）"
read_when:
  - 修改群组消息规则或提及设置时
---

<div id="group-messages-whatsapp-web-channel">
  # 群消息（WhatsApp 网页通道）
</div>

目标：让 Clawd 加入 WhatsApp 群聊中，只在被 @ 提及时唤醒，并将该群聊会话与个人私聊会话区分开来。

注意：`agents.list[].groupChat.mentionPatterns` 现在也被 Telegram/Discord/Slack/iMessage 使用；本文档重点说明 WhatsApp 特定行为。对于多智能体部署，请为每个智能体分别设置 `agents.list[].groupChat.mentionPatterns`（或者使用 `messages.groupChat.mentionPatterns` 作为全局兜底配置）。

<div id="whats-implemented-2025-12-03">
  ## 当前已实现内容（2025-12-03）
</div>

- 激活模式：`mention`（默认）或 `always`。`mention` 需要一次 ping（通过 `mentionedJids` 实际的 WhatsApp @ 提及、正则匹配模式，或文本中任意位置出现机器人的 E.164）。`always` 会在每一条消息上唤醒智能体，但只有在它能提供有意义的价值时才应回复；否则返回静默 token `NO_REPLY`。默认值可在配置（`channels.whatsapp.groups`）中设置，并可通过 `/activation` 为每个群单独覆盖。当设置了 `channels.whatsapp.groups` 时，它同时也充当群组允许列表（包含 `"*"` 以允许所有）。
- 群组策略：`channels.whatsapp.groupPolicy` 控制是否接受群消息（`open|disabled|allowlist`）。`allowlist` 使用 `channels.whatsapp.groupAllowFrom`（回退：显式的 `channels.whatsapp.allowFrom`）。默认是 `allowlist`（在你添加发送方之前一律阻止）。
- 按群分离的会话：会话键形如 `agent:<agentId>:whatsapp:group:<jid>`，因此诸如 `/verbose on` 或 `/think high` 之类的命令（作为单独消息发送）都会限定在该群组的 scope 内；个人私聊 DM 的状态不会受影响。群线程不会运行心跳。
- 上下文注入：**仅限待处理（pending-only）的** 群消息（默认 50 条），即那些*没有*触发一次运行的消息，会以 `[Chat messages since your last reply - for context]` 作为前缀注入，其后是触发本次运行的那一行，标记为 `[Current message - respond to this]`。已经存在于会话中的消息不会被重新注入。
- 发送方标注：每一批群组消息现在都会以 `[from: Sender Name (+E164)]` 结尾，以便 Pi 知道当前是谁在说话。
- 临时消息 / 查看一次即消失（view-once）：我们会在抽取文本/提及之前先将这些消息展开处理，因此其中的 ping 依然可以触发。
- 群组系统提示词：在群会话的第一轮（以及每当 `/activation` 改变模式时），我们会向系统提示词中注入一段简短说明，例如：`You are replying inside the WhatsApp group "<subject>". Group members: Alice (+44...), Bob (+43...), … Activation: trigger-only … Address the specific sender noted in the message context.` 如果元数据不可用，我们仍然会告诉智能体这是一个群聊。

<div id="config-example-whatsapp">
  ## 配置示例（WhatsApp）
</div>

在 `~/.openclaw/openclaw.json` 中添加一个 `groupChat` 配置块，这样即使 WhatsApp 在文本正文中去掉了可见的 `@`，基于显示名称的提醒也能生效：

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true }
      }
    }
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          historyLimit: 50,
          mentionPatterns: [
            "@?openclaw",
            "\\+?15555550123"
          ]
        }
      }
    ]
  }
}
```

Notes:

* 这些正则表达式大小写不敏感；它们既匹配类似 `@openclaw` 的显示名提及，也匹配带或不带 `+` / 空格的原始号码。
* 当有人点击联系人时，WhatsApp 仍会通过 `mentionedJids` 发送规范化提及，因此几乎用不到号码后备匹配，但作为安全兜底机制仍然很有用。


<div id="activation-command-owner-only">
  ### 激活命令（仅限所有者使用）
</div>

在群聊中使用以下命令：

- `/activation mention`
- `/activation always`

只有所有者的号码（来自 `channels.whatsapp.allowFrom`，如果未设置则为机器人自己的 E.164 号码）可以更改此设置。在群组中单独发送一条 `/status` 消息以查看当前激活模式。

<div id="how-to-use">
  ## 如何使用
</div>

1) 将你的 WhatsApp 账号（运行 OpenClaw 的那个）加入该群组。
2) 发送 `@openclaw …`（或附带号码）。只有在允许列表中的发送方才能触发它，除非你将 `groupPolicy: "open"` 设为 `open`（表示允许任何用户不受限制地发送消息）。
3) 智能体的提示词会包含最近的群组上下文以及结尾的 `[from: …]` 标记，这样它就能准确地回应到对应的人。
4) 会话级指令（`/verbose on`、`/think high`、`/new` 或 `/reset`、`/compact`）只作用于该群组的会话；请将它们作为独立消息发送，以便被正确识别。你个人的私信（DM）会话保持独立、不受影响。

<div id="testing-verification">
  ## 测试 / 验证
</div>

- 手动冒烟测试：
  - 在群组中发送一个 `@openclaw` ping，并确认回复中引用了发送者名称。
  - 再发送一次 ping，确认历史块在本轮回复中被包含，并在下一轮对话时被清空。
- 检查 Gateway 日志（使用 `--verbose` 运行），查看 `inbound web message` 条目中是否显示 `from: <groupJid>` 以及带有 `[from: …]` 的后缀。

<div id="known-considerations">
  ## 已知注意事项
</div>

- 为避免产生过多广播噪声，群组有意不发送心跳。
- 回声抑制使用合并后的批处理字符串；如果你在没有提及的情况下连续发送两次完全相同的文本，只有第一次会收到响应。
- 会话存储记录会在会话存储中显示为 `agent:&lt;agentId&gt;:whatsapp:group:&lt;jid&gt;`（默认路径：`~/.openclaw/agents/&lt;agentId&gt;/sessions/sessions.json`）；如果不存在该条目，只表示该群组尚未触发过运行。
- 群组中的“正在输入”指示符遵循 `agents.defaults.typingMode`（默认值：在未被提及时为 `message`）。