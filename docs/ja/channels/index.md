---
title: チャネル
summary: "OpenClaw が連携できるメッセージングプラットフォーム"
read_when:
  - OpenClaw 用のチャットチャネルを選択したいとき
  - サポートされているメッセージングプラットフォームの概要を手早く知りたいとき
---

<div id="chat-channels">
  # チャットチャネル
</div>

OpenClaw は、あなたがすでに使っているあらゆるチャットアプリ上でやり取りできます。各チャネルは Gateway 経由で接続されます。
テキストメッセージはすべてのチャネルでサポートされていますが、メディアやリアクションの対応状況はチャネルによって異なります。

<div id="supported-channels">
  ## 対応チャネル
</div>

* [WhatsApp](/ja/channels/whatsapp) — 最も一般的。Baileys を使用し、QR ペアリングが必要です。
* [Telegram](/ja/channels/telegram) — grammY 経由の Bot API。グループに対応。
* [Discord](/ja/channels/discord) — Discord Bot API + Gateway。サーバー、チャンネル、DM に対応。
* [Slack](/ja/channels/slack) — Bolt SDK。ワークスペース向けアプリ。
* [Google Chat](/ja/channels/googlechat) — HTTP Webhook 経由の Google Chat API アプリ。
* [Mattermost](/ja/channels/mattermost) — Bot API + WebSocket。チャンネル、グループ、DM に対応（プラグイン。別途インストールが必要）。
* [Signal](/ja/channels/signal) — signal-cli。プライバシー重視。
* [BlueBubbles](/ja/channels/bluebubbles) — **iMessage 用として推奨**。BlueBubbles macOS サーバーの REST API を使用し、編集、送信取り消し、エフェクト、リアクション、グループ管理などの機能をフルサポート（編集は現在 macOS 26 Tahoe では動作しません）。
* [iMessage](/ja/channels/imessage) — macOS のみ。imsg によるネイティブ統合（レガシー。新規セットアップには BlueBubbles の使用を検討してください）。
* [Microsoft Teams](/ja/channels/msteams) — Bot Framework。エンタープライズ向けサポート（プラグイン。別途インストールが必要）。
* [LINE](/ja/channels/line) — LINE Messaging API ボット（プラグイン。別途インストールが必要）。
* [Nextcloud Talk](/ja/channels/nextcloud-talk) — Nextcloud Talk を用いたセルフホスト型チャット（プラグイン。別途インストールが必要）。
* [Matrix](/ja/channels/matrix) — Matrix プロトコル（プラグイン。別途インストールが必要）。
* [Nostr](/ja/channels/nostr) — NIP-04 による分散型 DM（プラグイン。別途インストールが必要）。
* [Tlon](/ja/channels/tlon) — Urbit ベースのメッセンジャー（プラグイン。別途インストールが必要）。
* [Twitch](/ja/channels/twitch) — IRC 接続経由の Twitch チャット（プラグイン。別途インストールが必要）。
* [Zalo](/ja/channels/zalo) — Zalo Bot API。ベトナムで人気のメッセンジャー（プラグイン。別途インストールが必要）。
* [Zalo Personal](/ja/channels/zalouser) — QR ログインによる Zalo 個人アカウント（プラグイン。別途インストールが必要）。
* [WebChat](/ja/web/webchat) — WebSocket 経由の Gateway WebChat UI。

<div id="notes">
  ## 注意事項
</div>

* チャンネルは同時に実行できます。複数を設定すると、OpenClaw がチャット単位でルーティングします。
* 最も簡単にセットアップできるのは通常 **Telegram** です（シンプルなボットトークンのみで動作）。WhatsApp では QR ペアリングが必要で、より多くの状態をディスク上に保存します。
* グループでの動作はチャンネルごとに異なります。[Groups](/ja/concepts/groups) を参照してください。
* 安全性のため、DM ペアリングと許可リストが強制されます。[Security](/ja/gateway/security) を参照してください。
* Telegram の内部仕様については [grammY notes](/ja/channels/grammy) を参照してください。
* トラブルシューティングについては [Channel troubleshooting](/ja/channels/troubleshooting) を参照してください。
* モデルプロバイダーは別途ドキュメント化されています。[Model Providers](/ja/providers/models) を参照してください。