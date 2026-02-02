---
title: チャンネル
summary: "`openclaw channels` の CLI リファレンス（アカウント、ステータス、ログイン/ログアウト、ログ）"
read_when:
  - チャンネルアカウント（WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost（プラグイン）/Signal/iMessage）を追加・削除したいとき
  - チャンネルのステータスを確認したい、またはチャンネルログを tail で監視したいとき
---

<div id="openclaw-channels">
  # `openclaw channels`
</div>

Gateway 上でチャットチャネルアカウントとその実行時の状態を管理します。

関連ドキュメント:

* チャネルガイド: [Channels](/ja/channels/index)
* Gateway の設定: [Configuration](/ja/gateway/configuration)

<div id="common-commands">
  ## よく使うコマンド
</div>

```bash
openclaw channels list
openclaw channels status
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels logs --channel all
```

<div id="add-remove-accounts">
  ## アカウントの追加／削除
</div>

```bash
openclaw channels add --channel telegram --token <bot-token>
openclaw channels remove --channel telegram --delete
```

ヒント: `openclaw channels add --help` を付けて実行すると、各チャネル固有のフラグ（token、app token、signal-cli のパスなど）を確認できます。

<div id="login-logout-interactive">
  ## ログイン / ログアウト（対話型）
</div>

```bash
openclaw channels login --channel whatsapp
openclaw channels logout --channel whatsapp
```

<div id="troubleshooting">
  ## トラブルシューティング
</div>

* 包括的な確認として `openclaw status --deep` を実行します。
* ガイド付きの修正には `openclaw doctor` を使用します。
* `openclaw channels list` が `Claude: HTTP 403 ... user:profile` と表示する場合 → usage スナップショットには `user:profile` スコープが必要です。`--no-usage` を使うか、claude.ai のセッションキー（`CLAUDE_WEB_SESSION_KEY` / `CLAUDE_WEB_COOKIE`）を指定するか、Claude Code CLI で再認証を行ってください。

<div id="capabilities-probe">
  ## 機能プローブ
</div>

プロバイダーの機能ヒント（利用可能な場合は intent/scope）および静的な機能サポートを取得します。

```bash
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
```

Notes:

* `--channel` は省略可能です。すべてのチャンネル（拡張機能を含む）を一覧表示する場合は指定しないでください。
* `--target` は `channel:&lt;id&gt;` または数値のチャンネル ID を受け付け、Discord にのみ適用されます。
* Probe の内容はプロバイダーごとに異なります。たとえば、Discord の intents ＋ 必要に応じたチャンネル権限、Slack の bot ＋ user スコープ、Telegram の bot フラグ ＋ webhook、Signal のデーモンバージョン、MS Teams のアプリトークン ＋ Graph のロール/スコープ（判明しているものについては注釈付き）です。Probe が定義されていないチャンネルは `Probe: unavailable` と表示されます。

<div id="resolve-names-to-ids">
  ## 名前から ID を取得する
</div>

プロバイダー ディレクトリを使用して、チャンネル名やユーザー名から ID を取得します。

```bash
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels resolve --channel discord "My Server/#support" "@someone"
openclaw channels resolve --channel matrix "Project Room"
```

注意:

* 対象の種類を指定するには `--kind user|group|auto` を使用します。
* 複数のエントリが同じ名前を持つ場合、名前解決時にはアクティブなエントリが優先されます。
