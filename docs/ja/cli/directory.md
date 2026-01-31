---
title: ディレクトリ
summary: "`openclaw directory` の CLI リファレンス（self、peers、groups）"
read_when:
  - チャネル向けの連絡先／グループ／自分自身の ID を調べたいとき
  - チャネルディレクトリ用アダプタを開発しているとき
---

<div id="openclaw-directory">
  # `openclaw directory`
</div>

対応しているチャネルに対してディレクトリ検索（連絡先／ピア、グループ、自分自身を表す「me」）を行います。

<div id="common-flags">
  ## 共通フラグ
</div>

- `--channel <name>`: チャンネル ID またはエイリアス（複数のチャンネルが設定されている場合は必須。1 つだけ設定されている場合は自動）
- `--account <id>`: アカウント ID（デフォルト: チャンネルのデフォルトアカウント）
- `--json`: JSON 形式で出力

<div id="notes">
  ## 注意事項
</div>

- `directory` は、他のコマンド（特に `openclaw message send --target ...`）に貼り付けるための ID を探しやすくするためのものです。
- 多くのチャネルでは、結果はリアルタイムなプロバイダー側ディレクトリではなく、設定に基づいた（許可リスト / 設定済みグループ）情報になります。
- デフォルトの出力は、タブ区切りの `id`（および場合によっては `name`）です。スクリプト用途には `--json` を使用してください。

<div id="using-results-with-message-send">
  ## `message send` で結果を扱う
</div>

```bash
openclaw directory peers list --channel slack --query "U0"
openclaw message send --channel slack --target user:U012ABCDEF --message "hello"
```


<div id="id-formats-by-channel">
  ## ID 形式（チャネル別）
</div>

- WhatsApp: `+15551234567`（DM）、`1234567890-1234567890@g.us`（グループ）
- Telegram: `@username` または数値チャット ID。グループは数値 ID
- Slack: `user:U…` と `channel:C…`
- Discord: `user:<id>` と `channel:<id>`
- Matrix（プラグイン）: `user:@user:server`、`room:!roomId:server`、または `#alias:server`
- Microsoft Teams（プラグイン）: `user:<id>` と `conversation:<id>`
- Zalo（プラグイン）: ユーザー ID（Bot API）
- Zalo Personal / `zalouser`（プラグイン）: `zca`（`me`、`friend list`、`group list`）から取得したスレッド ID（DM/グループ）

<div id="self-me">
  ## Self（自分）
</div>

```bash
openclaw directory self --channel zalouser
```


<div id="peers-contactsusers">
  ## ピア（連絡先/ユーザー）
</div>

```bash
openclaw directory peers list --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory peers list --channel zalouser --limit 50
```


<div id="groups">
  ## グループ
</div>

```bash
openclaw directory groups list --channel zalouser
openclaw directory groups list --channel zalouser --query "work"
openclaw directory groups members --channel zalouser --group-id <id>
```
