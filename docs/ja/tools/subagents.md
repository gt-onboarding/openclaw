---
title: サブエージェント
summary: "サブエージェント: 結果を要求元のチャットに通知する、分離されたエージェント実行を起動する"
read_when:
  - エージェント経由でバックグラウンド/並列処理を行いたい
  - sessions_spawn やサブエージェントのツールポリシーを変更している
---

<div id="sub-agents">
  # サブエージェント
</div>

サブエージェントは、既存のエージェント実行から派生して起動されるバックグラウンドのエージェント実行です。サブエージェントは独自のセッション（`agent:<agentId>:subagent:<uuid>`）内で動作し、完了すると、その結果をリクエスト元のチャットチャネルに**報告**します。

<div id="slash-command">
  ## スラッシュコマンド
</div>

`/subagents` を使用して、**現在のセッション** のサブエージェント実行を確認または制御できます:

- `/subagents list`
- `/subagents stop <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` は、実行メタデータ（ステータス、タイムスタンプ、セッション ID、トランスクリプトパス、クリーンアップ処理）を表示します。

主な目的は次のとおりです:

- 「リサーチ / 長時間タスク / 遅いツール」の作業を並列化し、メインの実行フローをブロックしない。
- デフォルトでサブエージェントを分離しておく（セッション分離 + オプションのサンドボックス化）。
- ツールの利用面を乱用しにくく保つ: サブエージェントはデフォルトではセッションツールを取得 **しない**。
- ネストした fan-out を避ける: サブエージェントはさらにサブエージェントを生成できない。

コストに関する注意: 各サブエージェントは **独自** のコンテキストとトークン使用量を持ちます。重い、または繰り返し実行されるタスクについては、サブエージェントにはより安価なモデルを設定し、メインのエージェントには高品質なモデルを維持してください。
これは `agents.defaults.subagents.model` またはエージェントごとのオーバーライドで設定できます。

<div id="tool">
  ## ツール
</div>

`sessions_spawn` を使用します:

- サブエージェントの実行を開始します（`deliver: false`、グローバルレーン: `subagent`）
- その後 announce ステップを実行し、announce の応答をリクエスト元のチャットチャネルに投稿します
- デフォルトモデル: `agents.defaults.subagents.model`（またはエージェントごとの `agents.list[].subagents.model`）を設定しない限り呼び出し元を継承します。明示的な `sessions_spawn.model` がある場合はそれが優先されます。

ツールパラメータ:

- `task`（必須）
- `label?`（任意）
- `agentId?`（任意; 許可されている場合は別のエージェント ID の配下でサブエージェントを起動します）
- `model?`（任意; サブエージェントのモデルを上書きします。不正な値はスキップされ、警告がツール結果に出たうえでサブエージェントはデフォルトモデルで実行されます）
- `thinking?`（任意; サブエージェント実行の thinking レベルを上書き）
- `runTimeoutSeconds?`（デフォルト `0`; 設定した場合、N 秒後にサブエージェント実行を中止します）
- `cleanup?`（`delete|keep`、デフォルト `keep`）

許可リスト:

- `agents.list[].subagents.allowAgents`: `agentId` 経由でターゲットにできるエージェント ID のリスト（任意の ID を許可するには `["*"]`）。デフォルト: リクエスト元エージェントのみ。

ディスカバリ:

- 現在 `sessions_spawn` に対して許可されているエージェント ID を確認するには `agents_list` を使用します。

自動アーカイブ:

- サブエージェントのセッションは `agents.defaults.subagents.archiveAfterMinutes`（デフォルト: 60）経過後に自動的にアーカイブされます。
- アーカイブでは `sessions.delete` を使用し、トランスクリプト名を同じフォルダー内で `*.deleted.<timestamp>` にリネームします。
- `cleanup: "delete"` を指定すると、announce 直後に即時アーカイブされます（トランスクリプトはリネームにより保持されます）。
- 自動アーカイブはベストエフォートです。Gateway が再起動されると保留中のタイマーは失われます。
- `runTimeoutSeconds` は自動アーカイブを行い **ません**。実行を停止するだけです。セッションは自動アーカイブが行われるまで残ります。

<div id="authentication">
  ## 認証
</div>

サブエージェントの認証は、セッションタイプではなく **agent id** によって判定されます:

- サブエージェントのセッションキーは `agent:<agentId>:subagent:<uuid>` です。
- 認証ストアは、そのエージェントの `agentDir` から読み込まれます。
- メインエージェントの認証プロファイルが **フォールバック** としてマージされます。競合がある場合は、サブエージェント側のプロファイルがメイン側のプロファイルを上書きします。

注: マージは追加的に行われるため、メイン側のプロファイルは常にフォールバックとして利用可能です。エージェントごとに完全に分離された認証は、まだサポートされていません。

<div id="announce">
  ## アナウンス
</div>

サブエージェントはアナウンスステップを通じて報告します:

- アナウンスステップはサブエージェントのセッション内で実行されます（呼び出し元のセッションではありません）。
- サブエージェントが `ANNOUNCE_SKIP` のみを正確に返信した場合は、何も投稿されません。
- それ以外の場合、アナウンスの返信はフォローアップの `agent` 呼び出し（`deliver=true`）経由で、呼び出し元のチャットチャネルに投稿されます。
- アナウンスの返信は、可能な場合はスレッド／トピックのルーティングを維持します（Slack スレッド、Telegram トピック、Matrix スレッド）。
- アナウンスメッセージは一定のテンプレート形式に正規化されます:
  - `Status:` 実行結果から導出されます（`success`、`error`、`timeout`、または `unknown`）。
  - `Result:` アナウンスステップからの要約コンテンツ（欠けている場合は `(not available)`）。
  - `Notes:` エラーの詳細やその他の有用なコンテキスト。
- `Status` はモデル出力から推論されるのではなく、実行時の結果シグナルから取得されます。

アナウンスペイロードには末尾に統計行が含まれます（他のメッセージにラップされている場合でも）:

- 実行時間（例: `runtime 5m12s`）
- トークン使用量（入力／出力／合計）
- モデルの価格が設定されている場合の推定コスト（`models.providers.*.models[].cost`）
- `sessionKey`、`sessionId`、およびトランスクリプトパス（メインエージェントが `sessions_history` 経由で履歴を取得したり、ディスク上のファイルを参照したりできるようにするため）

<div id="tool-policy-sub-agent-tools">
  ## ツールポリシー（サブエージェントのツール）
</div>

既定では、サブエージェントは **セッション系ツールを除くすべてのツール** を利用できます:

* `sessions_list`
* `sessions_history`
* `sessions_send`
* `sessions_spawn`

設定で変更できます:

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1
      }
    }
  },
  tools: {
    subagents: {
      tools: {
        // deny が優先
        deny: ["gateway", "cron"],
        // allow を設定すると許可リスト方式になる(deny は引き続き優先)
        // allow: ["read", "exec", "process"]
      }
    }
  }
}
```


<div id="concurrency">
  ## 同時実行
</div>

サブエージェントは、専用のインプロセスキュー・レーンを使用します：

- レーン名：`subagent`
- 同時実行数：`agents.defaults.subagents.maxConcurrent`（デフォルト `8`）

<div id="stopping">
  ## 停止
</div>

- リクエスターのチャットで `/stop` を送信すると、リクエスターのセッションが中断され、そこから生成された実行中のサブエージェントの実行もすべて停止します。

<div id="limitations">
  ## 制約事項
</div>

- サブエージェントの announce は **ベストエフォート方式** です。Gateway が再起動すると、保留中の「announce back」処理は失われます。
- サブエージェントは同じ Gateway プロセスのリソースを引き続き共有します。`maxConcurrent` は安全弁として扱ってください。
- `sessions_spawn` は常にノンブロッキング動作です。直ちに `{ status: "accepted", runId, childSessionKey }` を返します。
- サブエージェントのコンテキストには `AGENTS.md` と `TOOLS.md` のみが注入されます（`SOUL.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md`、`BOOTSTRAP.md` は含まれません）。