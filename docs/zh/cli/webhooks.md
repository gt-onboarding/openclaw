---
title: Webhooks（Webhook 回调）
summary: "用于 `openclaw webhooks` 的 CLI 参考（Webhook 辅助命令 + Gmail Pub/Sub）"
read_when:
  - 你想将 Gmail Pub/Sub 事件接入 OpenClaw
  - 你需要 Webhook 辅助命令
---

<div id="openclaw-webhooks">
  # `openclaw webhooks`
</div>

用于 Webhook 的辅助工具和集成（Gmail Pub/Sub、Webhook 辅助工具）。

相关内容：

* Webhooks：[Webhook](/zh/automation/webhook)
* Gmail Pub/Sub：[Gmail Pub/Sub](/zh/automation/gmail-pubsub)

<div id="gmail">
  ## Gmail
</div>

```bash
openclaw webhooks gmail setup --account you@example.com
openclaw webhooks gmail run
```

详情请参见 [Gmail Pub/Sub 文档](/zh/automation/gmail-pubsub)。
