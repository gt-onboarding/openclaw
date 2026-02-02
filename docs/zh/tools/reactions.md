---
title: Reaction
summary: "各渠道通用的 Reaction 语义"
read_when:
  - 在任何渠道中开发或配置 Reaction 功能时
---

<div id="reaction-tooling">
  # Reaction 工具
</div>

各渠道通用的 reaction 语义：

- 添加 reaction 时必须提供 `emoji`。
- `emoji=""` 在支持的情况下会移除机器人添加的 reaction。
- `remove: true` 在支持的情况下会移除指定的 emoji（需要同时提供 `emoji`）。

各渠道说明：

- **Discord/Slack**：空的 `emoji` 会移除该消息上机器人添加的所有 reaction；`remove: true` 只会移除指定的那个 emoji。
- **Google Chat**：空的 `emoji` 会移除该消息上应用添加的所有 reaction；`remove: true` 只会移除指定的那个 emoji。
- **Telegram**：空的 `emoji` 会移除机器人添加的 reaction；`remove: true` 同样会移除 reaction，但工具校验仍然要求提供非空的 `emoji`。
- **WhatsApp**：空的 `emoji` 会移除机器人的 reaction；`remove: true` 会映射为空的 emoji（仍然要求提供 `emoji`）。
- **Signal**：当启用 `channels.signal.reactionNotifications` 时，传入的 reaction 通知会触发系统事件。