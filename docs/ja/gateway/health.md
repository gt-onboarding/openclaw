---
title: ヘルス
summary: "チャンネル接続状況のヘルスチェック手順"
read_when:
  - WhatsApp チャンネルのヘルスを診断するとき
---

<div id="health-checks-cli">
  # ヘルスチェック (CLI)
</div>

チャネルの接続状況を推測に頼らず確認するための簡単なガイド。

<div id="quick-checks">
  ## クイックチェック
</div>

- `openclaw status` — ローカル概要: Gateway への到達性/モード、アップデート推奨の有無、リンク済みチャネルの認証経過時間、セッションと最近のアクティビティ。
- `openclaw status --all` — ローカルでのフル診断（読み取り専用、カラー表示、デバッグ用に貼り付けても安全）。
- `openclaw status --deep` — 起動中の Gateway もプローブ（サポートされている場合はチャネルごとのプローブ）。
- `openclaw health --json` — 起動中の Gateway に完全なヘルススナップショットを要求します（WS のみで動作; Baileys ソケットには直接接続しない）。
- WhatsApp/WebChat で `/status` を単独メッセージとして送信すると、エージェントを起動せずにステータス返信を取得できます。
- ログ: `/tmp/openclaw/openclaw-*.log` を tail し、`web-heartbeat`、`web-reconnect`、`web-auto-reply`、`web-inbound` でフィルタリングします。

<div id="deep-diagnostics">
  ## 詳細診断
</div>

- ディスク上の認証情報: `ls -l ~/.openclaw/credentials/whatsapp/<accountId>/creds.json`（更新時刻 `mtime` は最近の時刻になっている必要があります）。
- セッションストア: `ls -l ~/.openclaw/agents/<agentId>/sessions/sessions.json`（パスは設定で変更可能です）。件数と直近の送信先は `status` で確認できます。
- 再リンクフロー: ログにステータスコード 409–515 または `loggedOut` が現れた場合は `openclaw channels logout && openclaw channels login --verbose` を実行します。（注: ステータス 515 の場合は、ペアリング後に QR ログインフローが 1 回自動的に再開されます）。

<div id="when-something-fails">
  ## 障害が発生したとき
</div>

- `logged out` またはステータス 409–515 → `openclaw channels logout` を実行してから `openclaw channels login` で再リンクします。
- Gateway に接続できない → 次のコマンドで起動します: `openclaw gateway --port 18789`（ポートが使用中の場合は `--force` を使用）。
- 受信メッセージがない → リンク済みの電話がオンラインであり、送信者が許可されているかを確認します（`channels.whatsapp.allowFrom`）。グループチャットの場合は、許可リストおよびメンションルールの設定が適切であることを確認します（`channels.whatsapp.groups`, `agents.list[].groupChat.mentionPatterns`）。

<div id="dedicated-health-command">
  ## 専用の "health" コマンド
</div>

`openclaw health --json` は、実行中の Gateway に対してヘルススナップショットを要求します（CLI からチャネルソケットへ直接接続することはありません）。利用可能な場合は、リンクされたクレデンシャル／認証情報の発行からの経過時間、チャネルごとのプローブ結果サマリー、セッションストアのサマリー、およびプローブの実行時間をレポートします。Gateway に到達できない場合、またはプローブが失敗／タイムアウトした場合は、非ゼロ終了コードで終了します。デフォルトの 10 秒を上書きするには、`--timeout <ms>` を使用してください。