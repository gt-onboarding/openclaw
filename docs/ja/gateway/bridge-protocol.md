---
title: ブリッジプロトコル
summary: "ブリッジプロトコル（レガシーノード）：TCP JSONL、ペアリング、スコープ付き RPC"
read_when:
  - ノードクライアント（iOS/Android/macOS ノードモード）を構築またはデバッグしているとき
  - ペアリングまたはブリッジ認証の失敗を調査しているとき
  - Gateway が公開しているノードのインターフェースを監査しているとき
---

<div id="bridge-protocol-legacy-node-transport">
  # Bridge プロトコル（レガシーなノード用トランスポート）
</div>

Bridge プロトコルは **レガシー** なノード用トランスポート方式（TCP JSONL）です。新しいノードクライアントは、代わりに統一された Gateway の WebSocket プロトコルを使用する必要があります。

オペレーターやノードクライアントを実装する場合は、
[Gateway プロトコル](/ja/gateway/protocol) を使用してください。

**注意:** 現在の OpenClaw ビルドには TCP bridge リスナーは含まれていません。このドキュメントは履歴参照用として保持されています。
レガシーな `bridge.*` 設定キーは、もはや設定スキーマの一部ではありません。

<div id="why-we-have-both">
  ## なぜ両方あるのか
</div>

* **セキュリティ境界**: bridge は Gateway の完全な API サーフェス全体ではなく、
  限定された許可リストだけを公開します。
* **ペアリング + ノード ID**: ノードの受け入れは Gateway が管理し、
  ノードごとのトークンにひも付けられます。
* **ディスカバリ UX**: ノードは LAN 上では Bonjour 経由で Gateway を検出でき、
  tailnet 経由で直接接続することもできます。
* **ループバック WS**: WS ベースの完全なコントロールプレーンは、SSH でトンネルしない限りローカルにとどまります。

<div id="transport">
  ## トランスポート
</div>

* TCP、1 行ごとに 1 つの JSON オブジェクト（JSONL）。
* `bridge.tls.enabled` が true の場合は TLS をオプションとして利用可能。
* レガシーのデフォルト待ち受けポートは `18790` でした（現在のビルドでは TCP ブリッジは起動しません）。

TLS が有効な場合、ディスカバリ用の TXT レコードには `bridgeTls=1` と
`bridgeTlsSha256` が含まれ、ノードは証明書のピン留めを行えます。

<div id="handshake-pairing">
  ## ハンドシェイク＋ペアリング
</div>

1. クライアントが、ノードのメタデータとトークン（すでにペアリング済みならそのトークン）を含む `hello` を送信する。
2. ペアリングされていない場合、Gateway は `error`（`NOT_PAIRED` / `UNAUTHORIZED`）で応答する。
3. クライアントが `pair-request` を送信する。
4. Gateway は承認を待ち、その後 `pair-ok` と `hello-ok` を送信する。

`hello-ok` は `serverName` を返し、必要に応じて `canvasHostUrl` も含める。

<div id="frames">
  ## フレーム
</div>

Client → Gateway:

* `req` / `res`: スコープ付き Gateway RPC（chat、sessions、config、health、voicewake、skills.bins）
* `event`: ノードからのシグナル（音声書き起こし、エージェントリクエスト、チャット購読、exec のライフサイクル）

Gateway → Client:

* `invoke` / `invoke-res`: ノードコマンド（`canvas.*`、`camera.*`、`screen.record`、
  `location.get`、`sms.send`）
* `event`: 購読中のセッションに対するチャットの更新
* `ping` / `pong`: キープアライブ

レガシーの許可リスト適用処理は `src/gateway/server-bridge.ts` 内に存在していたが、削除済み。

<div id="exec-lifecycle-events">
  ## Exec ライフサイクルイベント
</div>

ノードは `system.run` のアクティビティを通知するために `exec.finished` または `exec.denied` イベントを送信できます。
これらは Gateway 内のシステムイベントにマッピングされます（レガシー ノードは引き続き `exec.started` を送信する場合があります）。

ペイロードフィールド（特記なき限り任意）:

* `sessionKey`（必須）: システムイベントを受信するエージェント セッション。
* `runId`: グルーピングのための一意の exec ID。
* `command`: 生または整形済みのコマンド文字列。
* `exitCode`, `timedOut`, `success`, `output`: 完了時の詳細（`finished` のみ）。
* `reason`: 拒否理由（`denied` のみ）。

<div id="tailnet-usage">
  ## Tailnet の利用
</div>

* bridge を tailnet IP にバインドする: `~/.openclaw/openclaw.json` の
  `bridge.bind: "tailnet"` を設定します。
* クライアントは MagicDNS 名または tailnet IP を使って接続します。
* Bonjour はネットワークをまたいでは **動作しません**。必要に応じてホスト/ポートを手動指定するか、広域 DNS‑SD を使用してください。

<div id="versioning">
  ## バージョニング
</div>

Bridge は現在、**暗黙の v1**（min/max のネゴシエーションなし）です。後方互換性が前提であり、破壊的変更を行う前に bridge プロトコルのバージョンフィールドを追加してください。