---
title: エージェント送信
summary: "`openclaw agent` CLI を直接実行する（配信は任意）"
read_when:
  - エージェント CLI のエントリポイントを追加または変更するとき
---

<div id="openclaw-agent-direct-agent-runs">
  # `openclaw agent` (エージェントを直接実行)
</div>

`openclaw agent` は、受信チャットメッセージなしで、エージェントの単一ターンを実行します。
デフォルトでは **Gateway 経由** で実行されますが、現在のマシン上の組み込み
ランタイムを強制的に使用するには `--local` を追加します。

<div id="behavior">
  ## 動作
</div>

- 必須: `--message <text>`
- セッションの選択:
  - `--to <dest>` でセッションキーを決定します（グループ/チャンネル宛先は分離を維持し、ダイレクトチャットは `main` に統合）、**または**
  - `--session-id <id>` で既存のセッションを id で再利用、**または**
  - `--agent <id>` で設定済みエージェントを直接ターゲットにします（そのエージェントの `main` セッションキーを使用）
- 通常のインバウンド返信と同じ埋め込みエージェントランタイムを実行します。
- Thinking/verbose フラグはセッションストアに保持されます。
- 出力:
  - デフォルト: 返信テキスト（および `MEDIA:<url>` 行）を出力
  - `--json`: 構造化されたペイロード + メタデータを出力
- `--deliver` + `--channel` によるチャンネルへの任意の配信（ターゲット形式は `openclaw message --target` と同じ）。
- `--reply-channel` / `--reply-to` / `--reply-account` を使用して、セッションを変更せずに配信先を上書きできます。

Gateway に到達できない場合、CLI は埋め込みローカル実行に**フォールバック**します。

<div id="examples">
  ## 使用例
</div>

```bash
openclaw agent --to +15555550123 --message "status update"
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --to +15555550123 --message "Trace logs" --verbose on --json
openclaw agent --to +15555550123 --message "Summon reply" --deliver
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```


<div id="flags">
  ## フラグ
</div>

- `--local`: ローカルで実行（シェル環境にモデルプロバイダーの API キーが必要）
- `--deliver`: 応答を選択したチャネルに送信する
- `--channel`: 配信チャネル（`whatsapp|telegram|discord|googlechat|slack|signal|imessage`、デフォルト: `whatsapp`）
- `--reply-to`: 配信先の上書き
- `--reply-channel`: 配信チャネルの上書き
- `--reply-account`: 配信アカウント ID の上書き
- `--thinking <off|minimal|low|medium|high|xhigh>`: 思考レベルを永続化（GPT-5.2 + Codex モデルのみ）
- `--verbose <on|full|off>`: 詳細レベルを永続化
- `--timeout <seconds>`: エージェントのタイムアウトを上書き
- `--json`: 構造化された JSON で出力