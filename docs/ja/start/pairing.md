---
title: ペアリング
summary: "ペアリングの概要: 誰があなたにDMを送れるか、どのノードが参加できるかを承認する"
read_when:
  - DMアクセス制御を設定するとき
  - 新しい iOS/Android ノードをペアリングするとき
  - OpenClaw のセキュリティポスチャを確認するとき
---

<div id="pairing">
  # ペアリング
</div>

「ペアリング」は、OpenClaw における明示的な**所有者による承認**ステップです。
次の 2 つの用途で使われます:

1. **DM ペアリング**（ボットと会話することを許可される相手）
2. **ノード ペアリング**（どのデバイス／ノードが Gateway ネットワークに参加できるか）

セキュリティ全体の文脈: [Security](/ja/gateway/security)

<div id="1-dm-pairing-inbound-chat-access">
  ## 1) DM ペアリング（受信チャットアクセス）
</div>

チャネルが DM ポリシー `pairing` で設定されている場合、未知の送信者には短いコードが発行され、そのメッセージはあなたが承認するまで**処理されません**。

デフォルトの DM ポリシーについては [Security](/ja/gateway/security) を参照してください。

ペアリングコード:

* 8 文字、英大文字のみで、紛らわしい文字（`0O1I`）は使用しません。
* **1 時間後に失効します**。ボットがペアリングメッセージを送信するのは、新しいリクエストが作成されたときのみです（送信者ごとにおおよそ 1 時間に 1 回）。
* 保留中の DM ペアリング要求は、デフォルトでは**チャネルごとに最大 3 件**に制限されます。上限を超えた要求は、いずれかが期限切れになるか承認されるまで無視されます。

<div id="approve-a-sender">
  ### 送信者を承認する
</div>

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

サポートされているチャネル: `telegram`, `whatsapp`, `signal`, `imessage`, `discord`, `slack`。

<div id="where-the-state-lives">
  ### 状態が保存される場所
</div>

`~/.openclaw/credentials/` 以下に保存されています:

* 保留中のリクエスト: `<channel>-pairing.json`
* 承認済みの許可リストストア: `<channel>-allowFrom.json`

これらは機密情報として扱ってください（アシスタントへのアクセスを制御する情報です）。

<div id="2-node-device-pairing-iosandroidmacosheadless-nodes">
  ## 2) ノードデバイスのペアリング (iOS/Android/macOS/ヘッドレスノード)
</div>

ノードは `role: node` を持つ**デバイス**として Gateway に接続します。Gateway は、承認が必要なデバイスペアリング要求を作成します。

<div id="approve-a-node-device">
  ### ノードデバイスを承認する
</div>

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

### 状態の保存場所

`~/.openclaw/devices/` に保存されます:

* `pending.json`（短期間のみ有効。保留中のリクエストは期限切れになる）
* `paired.json`（ペアリング済みデバイスとトークン）

<div id="notes">
  ### 注意
</div>

* 従来の `node.pair.*` API（CLI: `openclaw nodes pending/approve`）は、
  Gateway が管理する別のペアリング用ストアです。WS ノードは依然としてデバイスのペアリングが必要です。

<div id="related-docs">
  ## 関連ドキュメント
</div>

* セキュリティモデルとプロンプトインジェクション: [Security](/ja/gateway/security)
* 安全にアップデートする（doctor の実行）: [Updating](/ja/install/updating)
* チャネル設定:
  * Telegram: [Telegram](/ja/channels/telegram)
  * WhatsApp: [WhatsApp](/ja/channels/whatsapp)
  * Signal: [Signal](/ja/channels/signal)
  * iMessage: [iMessage](/ja/channels/imessage)
  * Discord: [Discord](/ja/channels/discord)
  * Slack: [Slack](/ja/channels/slack)