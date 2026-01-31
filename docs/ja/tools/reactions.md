---
title: リアクション
summary: "チャネル間で共通のリアクションのセマンティクス"
read_when:
  - あらゆるチャネルでリアクションを扱うとき
---

<div id="reaction-tooling">
  # リアクション用ツール
</div>

チャネル間で共通のリアクションのセマンティクス:

- リアクションを追加する際は `emoji` が必須。
- `emoji=""` は、サポートされている場合にボットのリアクションを削除する。
- `remove: true` は、サポートされている場合に指定した絵文字を削除する（`emoji` が必要）。

チャネル別の注意点:

- **Discord/Slack**: 空の `emoji` は、そのメッセージ上のボットのすべてのリアクションを削除する。`remove: true` はその絵文字だけを削除する。
- **Google Chat**: 空の `emoji` は、そのメッセージ上のアプリのリアクションを削除する。`remove: true` はその絵文字だけを削除する。
- **Telegram**: 空の `emoji` はボットのリアクションを削除する。`remove: true` もリアクションを削除するが、ツール検証のために空でない `emoji` が引き続き必要。
- **WhatsApp**: 空の `emoji` はボットのリアクションを削除する。`remove: true` は空の emoji にマッピングされる（それでも `emoji` が必要）。
- **Signal**: 受信したリアクション通知は、`channels.signal.reactionNotifications` が有効な場合にシステムイベントを生成する。