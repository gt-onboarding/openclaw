---
title: Mattermost
summary: "Mattermost ボットのセットアップと OpenClaw の設定"
read_when:
  - Mattermost をセットアップするとき
  - Mattermost のルーティングをデバッグするとき
---

<div id="mattermost-plugin">
  # Mattermost (プラグイン)
</div>

ステータス: プラグイン経由でサポートされています（bot トークン + WebSocket イベント）。チャンネル、グループ、DM に対応しています。
Mattermost はセルフホスト可能なチームメッセージングプラットフォームです。製品の詳細やダウンロードについては公式サイト
[mattermost.com](https://mattermost.com)
を参照してください。

<div id="plugin-required">
  ## プラグインが必要
</div>

Mattermost はプラグインとして提供されており、標準インストールには含まれていません。

CLI（npm レジストリ経由）でインストールします:

```bash
openclaw plugins install @openclaw/mattermost
```

ローカルでのチェックアウト（Git リポジトリから実行する場合）:

```bash
openclaw plugins install ./extensions/mattermost
```

設定/オンボーディング時に Mattermost を選択し、Git リポジトリのチェックアウトが検出されると、
OpenClaw はローカルのインストールパスを自動的に提案します。

詳細: [プラグイン](/ja/plugin)

<div id="quick-setup">
  ## クイックセットアップ
</div>

1. Mattermost プラグインをインストールします。
2. Mattermost のボットアカウントを作成し、**bot token** をコピーします。
3. Mattermost の **ベース URL**（例：`https://chat.example.com`）をコピーします。
4. OpenClaw を構成して Gateway を起動します。

最小構成:

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing"
    }
  }
}
```

<div id="environment-variables-default-account">
  ## 環境変数（デフォルトアカウント）
</div>

環境変数で設定したい場合は、Gateway のホスト上で次を設定します:

* `MATTERMOST_BOT_TOKEN=...`
* `MATTERMOST_URL=https://chat.example.com`

環境変数は **デフォルト** アカウント（`default`）にのみ適用されます。その他のアカウントでは設定ファイルの値を使用してください。

<div id="chat-modes">
  ## チャットモード
</div>

Mattermost は DM には自動的に応答します。チャンネルでの挙動は `chatmode` で制御されます:

* `oncall` (デフォルト): チャンネル内で @メンションされたときのみ応答します。
* `onmessage`: すべてのチャンネルメッセージに応答します。
* `onchar`: メッセージがトリガー用の接頭辞で始まったときに応答します。

設定例:

```json5
{
  channels: {
    mattermost: {
      chatmode: "onchar",
      oncharPrefixes: [">", "!"]
    }
  }
}
```

Notes:

* `onchar` は、明示的な @メンションには引き続き応答します。
* レガシー設定では `channels.mattermost.requireMention` が引き続き有効ですが、`chatmode` の利用が推奨されます。

<div id="access-control-dms">
  ## アクセス制御（DM）
</div>

* デフォルト: `channels.mattermost.dmPolicy = "pairing"`（未知の送信者にはペアリングコードが表示されます）。
* 承認は次のコマンドで行います:
  * `openclaw pairing list mattermost`
  * `openclaw pairing approve mattermost <CODE>`
* パブリックDM: `channels.mattermost.dmPolicy="open"`（任意のユーザーからのDMを無制限に受け付ける設定）と `channels.mattermost.allowFrom=["*"]` を併用します。

<div id="channels-groups">
  ## チャンネル（グループ）
</div>

* デフォルト: `channels.mattermost.groupPolicy = "allowlist"`（メンション必須）。
* `channels.mattermost.groupAllowFrom`（ユーザー ID または `@username`）で、送信を許可する送信者（許可リスト）を指定します。
* オープンチャンネル: `channels.mattermost.groupPolicy="open"`（メンション必須。`open` 設定は、すべてのユーザーからのメッセージ受信を制限なく許可します）。

<div id="targets-for-outbound-delivery">
  ## アウトバウンド配信のターゲット
</div>

`openclaw message send` または cron/webhook で、次のターゲット形式を使用します:

* `channel:<id>` チャンネル宛て
* `user:<id>` DM（ダイレクトメッセージ）宛て
* `@username` DM（Mattermost API 経由で解決されるダイレクトメッセージ）宛て

ID だけを指定した場合は、チャンネルとして扱われます。

<div id="multi-account">
  ## 複数アカウント
</div>

Mattermost は `channels.mattermost.accounts` 配下で複数アカウントをサポートしています：

```json5
{
  channels: {
    mattermost: {
      accounts: {
        default: { name: "Primary", botToken: "mm-token", baseUrl: "https://chat.example.com" },
        alerts: { name: "Alerts", botToken: "mm-token-2", baseUrl: "https://alerts.example.com" }
      }
    }
  }
}
```

<div id="troubleshooting">
  ## トラブルシューティング
</div>

* チャンネルで返信が来ない場合: Bot がチャンネルに参加していることを確認し、メンションする（oncall）、トリガープレフィックスを使う（onchar）、または `chatmode: "onmessage"` を設定してください。
* 認証エラー: Bot トークン、ベース URL、アカウントが有効になっているかを確認してください。
* 複数アカウントの問題: 環境変数は `default` アカウントにのみ適用されます。