---
title: Webhooks
summary: "`openclaw webhooks`（webhook ヘルパー + Gmail Pub/Sub）の CLI リファレンス"
read_when:
  - Gmail Pub/Sub のイベントを OpenClaw に連携したいとき
  - webhook ヘルパーコマンドを使いたいとき
---

<div id="openclaw-webhooks">
  # `openclaw webhooks`
</div>

Webhook 用ヘルパーおよび連携機能（Gmail Pub/Sub、Webhook 用ヘルパー）。

関連項目:

* Webhooks: [Webhook](/ja/automation/webhook)
* Gmail Pub/Sub: [Gmail Pub/Sub](/ja/automation/gmail-pubsub)

<div id="gmail">
  ## Gmail
</div>

```bash
openclaw webhooks gmail setup --account you@example.com
openclaw webhooks gmail run
```

詳しくは [Gmail Pub/Sub のドキュメント](/ja/automation/gmail-pubsub) を参照してください。
