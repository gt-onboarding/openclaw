---
title: ノード
summary: "`openclaw nodes`（list/status/approve/invoke、camera/canvas/screen）の CLI リファレンス"
read_when:
  - ペアリング済みノード（カメラ、画面、キャンバス）を管理する必要がある
  - リクエストを承認したり、ノードのコマンドを実行する必要がある
---

<div id="openclaw-nodes">
  # `openclaw nodes`
</div>

ペアリング済みノード（デバイス）を管理し、ノードの機能を呼び出します。

関連項目:

* ノード概要: [Nodes](/ja/nodes)
* カメラ: [Camera nodes](/ja/nodes/camera)
* 画像: [Image nodes](/ja/nodes/images)

共通のオプション:

* `--url`, `--token`, `--timeout`, `--json`

<div id="common-commands">
  ## よく使うコマンド
</div>

```bash
openclaw nodes list
openclaw nodes list --connected
openclaw nodes list --last-connected 24h
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes status
openclaw nodes status --connected
openclaw nodes status --last-connected 24h
```

`nodes list` は、保留中 / ペアリング済みノードの一覧表を表示します。ペアリング済みの行には、直近の最終接続からの経過時間（Last Connect）が含まれます。
`--connected` を使うと、現在接続中のノードのみを表示します。`--last-connected <duration>` を使うと、
指定した期間内に接続したノードだけに絞り込めます（例: `24h`、`7d`）。

<div id="invoke-run">
  ## 呼び出し / 実行
</div>

```bash
openclaw nodes invoke --node <id|name|ip> --command <command> --params <json>
openclaw nodes run --node <id|name|ip> <command...>
openclaw nodes run --raw "git status"
openclaw nodes run --agent main --node <id|name|ip> --raw "git status"
```

Invoke フラグ:

* `--params <json>`: JSON オブジェクト文字列（デフォルト `{}`）。
* `--invoke-timeout <ms>`: ノード呼び出しのタイムアウト（デフォルト `15000`）。
* `--idempotency-key <key>`: 省略可能な冪等性キー。

<div id="exec-style-defaults">
  ### Exec スタイルのデフォルト
</div>

`nodes run` はモデルの exec 動作（デフォルト + 承認）を反映します:

* `tools.exec.*`（および `agents.list[].tools.exec.*` のオーバーライド）を参照します。
* `system.run` を呼び出す前に exec の承認フロー（`exec.approval.request`）を使用します。
* `tools.exec.node` が設定されている場合、`--node` は省略できます。
* `system.run` を公開しているノード（macOS コンパニオンアプリまたはヘッドレスなノードホスト）が必要です。

フラグ:

* `--cwd <path>`: 作業ディレクトリ。
* `--env <key=val>`: 環境変数の上書き（繰り返し指定可）。
* `--command-timeout <ms>`: コマンドのタイムアウト。
* `--invoke-timeout <ms>`: ノード呼び出しのタイムアウト（デフォルト `30000`）。
* `--needs-screen-recording`: 画面録画の権限を必須にします。
* `--raw <command>`: シェル文字列を実行します（`/bin/sh -lc` または `cmd.exe /c`）。
* `--agent <id>`: エージェントスコープの承認/許可リスト（設定済みエージェントがデフォルト）。
* `--ask <off|on-miss|always>`, `--security <deny|allowlist|full>`: デフォルト動作のオーバーライド。