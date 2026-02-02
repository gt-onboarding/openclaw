---
title: Nextcloud Talk
summary: "Nextcloud Talk のサポート状況、機能、および設定"
read_when:
  - Nextcloud Talk チャンネル機能の作業中
---

<div id="nextcloud-talk-plugin">
  # Nextcloud Talk（プラグイン）
</div>

対応状況: プラグイン（Webhook ボット）経由で利用可能です。ダイレクトメッセージ、ルーム、リアクション、および Markdown メッセージに対応しています。

<div id="plugin-required">
  ## 必要なプラグイン
</div>

Nextcloud Talk はプラグインとして提供されており、コアのインストールには含まれていません。

CLI（npm レジストリ）経由でインストールします：

```bash
openclaw plugins install @openclaw/nextcloud-talk
```

ローカルでのチェックアウト（Git リポジトリから実行する場合）:

```bash
openclaw plugins install ./extensions/nextcloud-talk
```

設定／オンボーディングの途中で Nextcloud Talk を選択し、Git チェックアウトが検出されると、
OpenClaw はローカルのインストールパスを自動的に提示します。

詳細については、[プラグイン](/ja/plugin) を参照してください。

<div id="quick-setup-beginner">
  ## クイックセットアップ（初心者向け）
</div>

1. Nextcloud Talk プラグインをインストールします。
2. Nextcloud サーバー上でボットを作成します:
   ```bash
   ./occ talk:bot:install "OpenClaw" "<shared-secret>" "<webhook-url>" --feature reaction
   ```
3. 対象ルームの設定でボットを有効にします。
4. OpenClaw を設定します:
   * Config: `channels.nextcloud-talk.baseUrl` + `channels.nextcloud-talk.botSecret`
   * または env（環境変数）: `NEXTCLOUD_TALK_BOT_SECRET`（デフォルトのアカウントのみ）
5. Gateway を再起動します（またはオンボーディングを完了します）。

最小限の設定:

```json5
{
  channels: {
    "nextcloud-talk": {
      enabled: true,
      baseUrl: "https://cloud.example.com",
      botSecret: "shared-secret",
      dmPolicy: "pairing"
    }
  }
}
```

<div id="notes">
  ## 注意事項
</div>

* Bot は DM を開始できません。ユーザー側から先に Bot にメッセージを送信する必要があります。
* Webhook URL は Gateway からアクセス可能でなければなりません。プロキシ配下にある場合は `webhookPublicUrl` を設定してください。
* メディアのアップロードは bot API ではサポートされていません。メディアは URL として送信されます。
* Webhook ペイロードは DM とルームを区別しません。ルーム種別のルックアップを有効にするには `apiUser` と `apiPassword` を設定してください（設定しない場合、DM はルームとして扱われます）。

<div id="access-control-dms">
  ## アクセス制御（DM）
</div>

* デフォルト: `channels.nextcloud-talk.dmPolicy = "pairing"`。未知の送信者にはペアリングコードが表示されます。
* 承認方法:
  * `openclaw pairing list nextcloud-talk`
  * `openclaw pairing approve nextcloud-talk <CODE>`
* パブリックDM: `channels.nextcloud-talk.dmPolicy="open"`（`open` は、どのユーザーからのメッセージも無制限に受け付ける設定）と `channels.nextcloud-talk.allowFrom=["*"]` を併用します。

<div id="rooms-groups">
  ## ルーム（グループ）
</div>

* デフォルト: `channels.nextcloud-talk.groupPolicy = "allowlist"` （メンション必須）。
* `channels.nextcloud-talk.rooms` でルームを許可リストに登録します:

```json5
{
  channels: {
    "nextcloud-talk": {
      rooms: {
        "room-token": { requireMention: true }
      }
    }
  }
}
```

* いずれのルームも許可しない場合は、許可リストを空にしておくか、`channels.nextcloud-talk.groupPolicy="disabled"` を設定してください。

<div id="capabilities">
  ## 機能
</div>

| 機能 | ステータス |
|---------|--------|
| ダイレクトメッセージ | サポート |
| ルーム | サポート |
| スレッド | 未サポート |
| メディア | URLのみ |
| リアクション | サポート |
| ネイティブコマンド | 未サポート |

<div id="configuration-reference-nextcloud-talk">
  ## 設定リファレンス (Nextcloud Talk)
</div>

完全な設定: [Configuration](/ja/gateway/configuration)

プロバイダーオプション:

* `channels.nextcloud-talk.enabled`: チャンネル起動の有効化/無効化。
* `channels.nextcloud-talk.baseUrl`: Nextcloud インスタンスの URL。
* `channels.nextcloud-talk.botSecret`: ボット用共有シークレット。
* `channels.nextcloud-talk.botSecretFile`: シークレットファイルのパス。
* `channels.nextcloud-talk.apiUser`: ルーム検索用の API ユーザー (DM 検出)。
* `channels.nextcloud-talk.apiPassword`: ルーム検索用の API/アプリパスワード。
* `channels.nextcloud-talk.apiPasswordFile`: API パスワードファイルのパス。
* `channels.nextcloud-talk.webhookPort`: webhook リスナーポート (デフォルト: 8788)。
* `channels.nextcloud-talk.webhookHost`: webhook ホスト (デフォルト: 0.0.0.0)。
* `channels.nextcloud-talk.webhookPath`: webhook パス (デフォルト: /nextcloud-talk-webhook)。
* `channels.nextcloud-talk.webhookPublicUrl`: 外部からアクセス可能な webhook URL。
* `channels.nextcloud-talk.dmPolicy`: `pairing | allowlist | open | disabled`。
* `channels.nextcloud-talk.allowFrom`: DM 許可リスト (ユーザー ID)。`open` の場合は `"*"` が必須。
* `channels.nextcloud-talk.groupPolicy`: `allowlist | open | disabled`。
* `channels.nextcloud-talk.groupAllowFrom`: グループ許可リスト (ユーザー ID)。
* `channels.nextcloud-talk.rooms`: ルーム単位の設定と許可リスト。
* `channels.nextcloud-talk.historyLimit`: グループ履歴上限 (0 で無効)。
* `channels.nextcloud-talk.dmHistoryLimit`: DM 履歴上限 (0 で無効)。
* `channels.nextcloud-talk.dms`: DM 単位の上書き設定 (historyLimit)。
* `channels.nextcloud-talk.textChunkLimit`: 送信テキストチャンクサイズ (文字数)。
* `channels.nextcloud-talk.chunkMode`: `length` (デフォルト) または `newline`。長さによる分割の前に、空行 (段落境界) で分割。
* `channels.nextcloud-talk.blockStreaming`: このチャンネルでのブロックストリーミングを無効化。
* `channels.nextcloud-talk.blockStreamingCoalesce`: ブロックストリーミングの結合動作の調整。
* `channels.nextcloud-talk.mediaMaxMb`: 受信メディアサイズの上限 (MB)。