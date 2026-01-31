---
title: Rpc
summary: "外部 CLI（signal-cli、imsg）および Gateway パターン向けの RPC アダプタ"
read_when:
  - 外部 CLI 連携を追加・変更するとき
  - RPC アダプタ（signal-cli、imsg）をデバッグするとき
---

<div id="rpc-adapters">
  # RPC アダプター
</div>

OpenClaw は外部 CLI を JSON-RPC 経由で統合します。現在は 2 つのパターンが用意されています。

<div id="pattern-a-http-daemon-signal-cli">
  ## パターン A: HTTP デーモン (signal-cli)
</div>

* `signal-cli` は HTTP 上の JSON-RPC を用いるデーモンとして動作します。
* イベントストリームは SSE (`/api/v1/events`) です。
* ヘルスチェック用エンドポイント: `/api/v1/check`。
* `channels.signal.autoStart=true` の場合、OpenClaw がライフサイクルを管理します。

セットアップとエンドポイントについては [Signal](/ja/channels/signal) を参照してください。

<div id="pattern-b-stdio-child-process-imsg">
  ## パターン B: stdio 子プロセス (imsg)
</div>

* OpenClaw が `imsg rpc` を子プロセスとして生成します。
* JSON-RPC は stdin/stdout 上で行区切りでやり取りします（1 行につき 1 つの JSON オブジェクト）。
* TCP ポートもデーモンも不要です。

使用される主要メソッド:

* `watch.subscribe` → 通知（`method: "message"`）
* `watch.unsubscribe`
* `send`
* `chats.list`（プローブ／診断）

セットアップとアドレッシング方法については [iMessage](/ja/channels/imessage) を参照してください（`chat_id` を推奨）。

<div id="adapter-guidelines">
  ## アダプターのガイドライン
</div>

* Gateway がプロセスを管理する（開始／停止はプロバイダーのライフサイクルに連動させる）。
* RPC クライアントは障害に強く設計する：タイムアウトを設定し、終了時には再起動する。
* 表示用の文字列よりも、`chat_id` などの安定した ID を優先して使用する。