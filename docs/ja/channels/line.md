---
title: LINE
summary: "LINE Messaging API プラグインのセットアップ、設定、および使用方法"
read_when:
  - OpenClaw を LINE に接続したい
  - LINE の Webhook と認証情報のセットアップが必要
  - LINE 固有のメッセージオプションを使用したい
---

<div id="line-plugin">
  # LINE (プラグイン)
</div>

LINE は LINE Messaging API を介して OpenClaw と接続します。このプラグインは
Gateway 上で webhook 受信用プロセスとして動作し、認証にはチャンネルアクセストークンと
チャンネルシークレットを使用します。

ステータス: プラグイン経由でサポートされています。ダイレクトメッセージ、グループチャット、
メディア、位置情報、Flex メッセージ、テンプレートメッセージ、クイックリプライに
対応しています。リアクションとスレッドには対応していません。

<div id="plugin-required">
  ## 必要なプラグイン
</div>

LINE プラグインをインストールしてください:

```bash
openclaw plugins install @openclaw/line
```

ローカルチェックアウト（Git リポジトリから実行する場合）：

```bash
openclaw plugins install ./extensions/line
```


<div id="setup">
  ## セットアップ
</div>

1. LINE Developers のアカウントを作成し、コンソールを開きます:
   https://developers.line.biz/console/
2. プロバイダーを作成（または選択）し、**Messaging API** チャネルを追加します。
3. チャネル設定から **Channel access token** と **Channel secret** をコピーします。
4. Messaging API の設定で **Use webhook** を有効にします。
5. webhook の URL を Gateway のエンドポイントに設定します（HTTPS が必須です）:

```
https://gateway-host/line/webhook
```

Gateway は LINE の webhook 検証 (GET) と受信イベント (POST) に応答します。
カスタムのパスが必要な場合は、`channels.line.webhookPath` または
`channels.line.accounts.<id>.webhookPath` を設定し、それに応じて URL を更新してください。


<div id="configure">
  ## 設定
</div>

最小構成:

```json5
{
  channels: {
    line: {
      enabled: true,
      channelAccessToken: "LINE_CHANNEL_ACCESS_TOKEN",
      channelSecret: "LINE_CHANNEL_SECRET",
      dmPolicy: "pairing"
    }
  }
}
```

環境変数（デフォルトアカウント専用）:

* `LINE_CHANNEL_ACCESS_TOKEN`
* `LINE_CHANNEL_SECRET`

トークン／シークレット用ファイル:

```json5
{
  channels: {
    line: {
      tokenFile: "/path/to/line-token.txt",
      secretFile: "/path/to/line-secret.txt"
    }
  }
}
```

複数アカウント：

```json5
{
  channels: {
    line: {
      accounts: {
        marketing: {
          channelAccessToken: "...",
          channelSecret: "...",
          webhookPath: "/line/marketing"
        }
      }
    }
  }
}
```


<div id="access-control">
  ## アクセス制御
</div>

ダイレクトメッセージは既定でペアリングモードになります。未知の送信者にはペアリングコードが発行され、承認されるまでそれらのメッセージは無視されます。

```bash
openclaw pairing list line
openclaw pairing approve line <CODE>
```

許可リストとポリシー:

* `channels.line.dmPolicy`: `pairing | allowlist | open | disabled`
* `channels.line.allowFrom`: DM 用の許可リスト登録済み LINE ユーザー ID
* `channels.line.groupPolicy`: `allowlist | open | disabled`
* `channels.line.groupAllowFrom`: グループ用の許可リスト登録済み LINE ユーザー ID
* グループごとの個別上書き設定: `channels.line.groups.<groupId>.allowFrom`

LINE ID は大文字と小文字が区別されます。有効な ID の形式は次のとおりです:

* ユーザー: `U` + 32 桁の 16 進数文字
* グループ: `C` + 32 桁の 16 進数文字
* ルーム: `R` + 32 桁の 16 進数文字


<div id="message-behavior">
  ## メッセージの動作
</div>

- テキストは 5000 文字単位で分割されます。
- Markdown の書式設定は除去され、コードブロックやテーブルは、可能な場合は Flex
  カードに変換されます。
- ストリーミング応答はバッファリングされ、エージェントが処理している間は、ローディング
  アニメーション付きのチャンク全体が LINE に送信されます。
- メディアのダウンロードは `channels.line.mediaMaxMb`（デフォルト 10）によって制限されます。

<div id="channel-data-rich-messages">
  ## チャンネルデータ（リッチメッセージ）
</div>

`channelData.line` を使用すると、クイック返信、位置情報、Flex カード、テンプレートメッセージを送信できます。

```json5
{
  text: "Here you go",
  channelData: {
    line: {
      quickReplies: ["Status", "Help"],
      location: {
        title: "Office",
        address: "123 Main St",
        latitude: 35.681236,
        longitude: 139.767125
      },
      flexMessage: {
        altText: "Status card",
        contents: { /* Flex payload */ }
      },
      templateMessage: {
        type: "confirm",
        text: "Proceed?",
        confirmLabel: "Yes",
        confirmData: "yes",
        cancelLabel: "No",
        cancelData: "no"
      }
    }
  }
}
```

LINE プラグインには、Flex メッセージ用プリセット向けの `/card` コマンドも用意されています。

```
/card info "Welcome" "Thanks for joining!"
```


<div id="troubleshooting">
  ## トラブルシューティング
</div>

- **Webhook 検証が失敗する:** Webhook URL が HTTPS であることと、
  `channelSecret` が LINE コンソールの値と一致していることを確認します。
- **インバウンドイベントが届かない:** Webhook パスが `channels.line.webhookPath`
  と一致していること、そして Gateway が LINE から到達可能であることを確認します。
- **メディアのダウンロードでエラーが発生する:** メディアサイズがデフォルト上限を超える場合は、
  `channels.line.mediaMaxMb` を引き上げてください。