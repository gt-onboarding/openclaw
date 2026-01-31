---
title: Grammy
summary: "セットアップに関する注意点を含む、grammY 経由の Telegram Bot API 連携"
read_when:
  - Telegram または grammY の経路を扱っているとき
---

<div id="grammy-integration-telegram-bot-api">
  # grammY 連携 (Telegram Bot API)
</div>

<div id="why-grammy">
  # なぜ grammY なのか
</div>

- TS-first の Bot API クライアントで、ロングポーリング + Webhook 用ヘルパー、ミドルウェア、エラーハンドリング、レートリミッターを内蔵。
- fetch + FormData を自前で書くよりも扱いやすいメディア用ヘルパーがあり、すべての Bot API メソッドをサポート。
- 拡張可能: カスタム fetch によるプロキシ対応、セッション用ミドルウェア（任意）、型安全なコンテキスト。

<div id="what-we-shipped">
  # 提供した内容
</div>

- **単一クライアントパス:** fetch ベースの実装を削除し、grammY が唯一の Telegram クライアント（送信 + Gateway）となりました。grammY throttler はデフォルトで有効です。
- **Gateway:** `monitorTelegramProvider` が grammY の `Bot` を構築し、メンション/許可リストによる制御、`getFile`/`download` によるメディアダウンロードを行い、`sendMessage/sendPhoto/sendVideo/sendAudio/sendDocument` で返信を配送します。`webhookCallback` によるロングポーリングまたは webhook の両方をサポートします。
- **プロキシ:** オプションの `channels.telegram.proxy` は、grammY の `client.baseFetch` 経由で `undici.ProxyAgent` を使用します。
- **Webhook サポート:** `webhook-set.ts` は `setWebhook/deleteWebhook` をラップし、`webhook.ts` はヘルスチェック + グレースフルシャットダウン付きでコールバックをホストします。Gateway は `channels.telegram.webhookUrl` が設定されている場合に webhook モードを有効化し（それ以外の場合はロングポーリングを行います）。
- **セッション:** 直接チャットはエージェントのメインセッション（`agent:<agentId>:<mainKey>`）に集約されます。グループは `agent:<agentId>:telegram:group:<chatId>` を使用し、返信は同じチャネルにルーティングされます。
- **設定項目:** `channels.telegram.botToken`, `channels.telegram.dmPolicy`, `channels.telegram.groups`（許可リスト + メンションのデフォルト）、`channels.telegram.allowFrom`, `channels.telegram.groupAllowFrom`, `channels.telegram.groupPolicy`, `channels.telegram.mediaMaxMb`, `channels.telegram.linkPreview`, `channels.telegram.proxy`, `channels.telegram.webhookSecret`, `channels.telegram.webhookUrl`。
- **ドラフトストリーミング:** オプションの `channels.telegram.streamMode` は、プライベートトピックチャットで `sendMessageDraft`（Bot API 9.3+）を使用します。これはチャネルブロックストリーミングとは別機能です。
- **テスト:** grammY のモックで DM + グループメンション制御およびアウトバウンド送信をカバーしています。メディア/webhook のフィクスチャ追加は引き続き歓迎します。

Open questions

- Bot API の 429 が発生した場合に備えた、オプションの grammY プラグイン（throttler）の扱い。
- より構造化されたメディアテスト（スタンプ、ボイスメッセージなど）の追加。
- Webhook のリッスンポートを設定可能にする（現在は Gateway 経由で配線しない限り 8787 に固定）。