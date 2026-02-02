---
title: トラブルシューティング
summary: "チャネル別トラブルシューティング用ショートカット（Discord／Telegram／WhatsApp）"
read_when:
  - チャネルは接続されているがメッセージが送受信されない場合
  - チャネルの設定ミス（intents、権限、プライバシーモード）を調査している場合
---

<div id="channel-troubleshooting">
  # チャンネルのトラブルシューティング
</div>

まずは次の項目から確認してください:

```bash
openclaw doctor
openclaw channels status --probe
```

`channels status --probe` は、一般的なチャネルの誤設定を検出できた場合に警告を表示し、あわせて認証情報や一部の権限／メンバーシップに対する簡易なライブチェックも実行します。

<div id="channels">
  ## チャンネル
</div>

* Discord: [/channels/discord#troubleshooting](/ja/channels/discord#troubleshooting)
* Telegram: [/channels/telegram#troubleshooting](/ja/channels/telegram#troubleshooting)
* WhatsApp: [/channels/whatsapp#troubleshooting-quick](/ja/channels/whatsapp#troubleshooting-quick)

<div id="telegram-quick-fixes">
  ## Telegram クイック対処
</div>

* ログに `HttpError: Network request for 'sendMessage' failed` または `sendChatAction` が表示される → IPv6 DNS を確認してください。`api.telegram.org` が IPv6 を優先して名前解決されており、ホスト側に IPv6 での外向き通信がない場合は、IPv4 を強制するか IPv6 を有効化してください。[/channels/telegram#troubleshooting](/ja/channels/telegram#troubleshooting) を参照してください。
* ログに `setMyCommands failed` が表示される → `api.telegram.org` へのアウトバウンド HTTPS および DNS の到達性を確認してください（ロックダウンされた VPS やプロキシ環境でよく発生します）。