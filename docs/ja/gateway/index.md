---
title: Gateway
summary: "Gateway サービスのランブック（ライフサイクルと運用）"
read_when:
  - Gateway プロセスを実行またはデバッグするとき
---

<div id="gateway-service-runbook">
  # Gateway サービス運用ランブック
</div>

最終更新日: 2025-12-09

<div id="what-it-is">
  ## これは何か
</div>

* 単一の Baileys/Telegram 接続と、制御プレーン／イベントプレーンを管理する常時稼働プロセスです。
* レガシーな `gateway` コマンドを置き換えます。CLI のエントリポイントは `openclaw gateway` です。
* 停止されるまで動作し、致命的なエラーが発生した場合は非ゼロのステータスコードで終了して、supervisor による再起動をトリガーします。

<div id="how-to-run-local">
  ## ローカルでの実行方法
</div>

```bash
openclaw gateway --port 18789
# for full debug/trace logs in stdio:
openclaw gateway --port 18789 --verbose
# ポートが使用中の場合、リスナーを終了してから起動:
openclaw gateway --force
# dev loop (auto-reload on TS changes):
pnpm gateway:watch
```

* 設定のホットリロードは `~/.openclaw/openclaw.json`（または `OPENCLAW_CONFIG_PATH`）を監視します。
  * デフォルトのモード: `gateway.reload.mode="hybrid"`（安全な変更はホット適用し、重大な変更時は再起動）。
  * ホットリロードは、必要に応じて **SIGUSR1** を使ってプロセス内での再起動を行います。
  * 無効化するには `gateway.reload.mode="off"` を指定します。
* WebSocket のコントロールプレーンを `127.0.0.1:<port>`（デフォルト 18789）にバインドします。
* 同じポートで HTTP も提供します（Control UI、hooks、A2UI）。単一ポートでの多重化です。
  * OpenAI Chat Completions (HTTP): [`/v1/chat/completions`](/ja/gateway/openai-http-api)。
  * OpenResponses (HTTP): [`/v1/responses`](/ja/gateway/openresponses-http-api)。
  * Tools Invoke (HTTP): [`/tools/invoke`](/ja/gateway/tools-invoke-http-api)。
* デフォルトで `canvasHost.port`（デフォルト `18793`）上に Canvas ファイルサーバーを起動し、`~/.openclaw/workspace/canvas` を `http://<gateway-host>:18793/__openclaw__/canvas/` として配信します。`canvasHost.enabled=false` または `OPENCLAW_SKIP_CANVAS_HOST=1` で無効化できます。
* ログは stdout に出力されます。launchd/systemd を使ってプロセスを常駐させ、ログローテーションを行ってください。
* トラブルシューティング時には `--verbose` を渡して、ログファイルのデバッグログ（ハンドシェイク、リクエスト/レスポンス、イベント）を stdio にミラーリングします。
* `--force` は、指定ポートで待ち受けているプロセスを `lsof` で検出し、SIGTERM を送信してどのプロセスを終了させたかをログに記録したうえで Gateway を起動します（`lsof` が存在しない場合はすぐに失敗します）。
* スーパーバイザ（launchd/systemd/mac アプリの子プロセスモード）の配下で実行している場合、停止/再起動時には通常 **SIGTERM** が送信されます。古いビルドでは、これが `pnpm` の `ELIFECYCLE` 終了コード **143**（SIGTERM）として現れることがありますが、これは正常なシャットダウンであり、クラッシュではありません。
* **SIGUSR1** は、認可済みの場合にプロセス内での再起動をトリガーします（Gateway ツール/設定の適用・更新、または手動リスタート用に `commands.restart` を有効化した場合）。
* Gateway 認証はデフォルトで必須です: `gateway.auth.token`（または `OPENCLAW_GATEWAY_TOKEN`）もしくは `gateway.auth.password` を設定してください。クライアントは、Tailscale Serve のアイデンティティを使用する場合を除き、`connect.params.auth.token/password` を送信する必要があります。
* ウィザードは、ループバック上であってもデフォルトでトークンを生成するようになりました。
* ポートの優先順位: `--port` &gt; `OPENCLAW_GATEWAY_PORT` &gt; `gateway.port` &gt; デフォルトの `18789`。

<div id="remote-access">
  ## リモートアクセス
</div>

* 可能であれば Tailscale/VPN を利用し、それ以外の場合は SSH トンネルを使用します:
  ```bash
  ssh -N -L 18789:127.0.0.1:18789 user@host
  ```
* クライアントは、トンネル経由で `ws://127.0.0.1:18789` に接続します。
* トークンを設定している場合、トンネル経由の接続であっても、クライアントは `connect.params.auth.token` にそのトークンを含める必要があります。

<div id="multiple-gateways-same-host">
  ## 複数 Gateway（同一ホスト）
</div>

通常は不要です。1つの Gateway で複数のメッセージングチャネルやエージェントを処理できます。複数の Gateway を使うのは、冗長化や厳密な分離が必要な場合（例: 救援用ボット）に限定してください。

状態と設定を分離し、ポート番号を一意にすれば利用できます。詳細ガイド: [Multiple gateways](/ja/gateway/multiple-gateways)。

サービス名はプロファイルごとに異なります:

* macOS: `bot.molt.<profile>`（レガシーな `com.openclaw.*` が残っている場合があります）
* Linux: `openclaw-gateway-<profile>.service`
* Windows: `OpenClaw Gateway (<profile>)`

インストールメタデータはサービス設定に埋め込まれています:

* `OPENCLAW_SERVICE_MARKER=openclaw`
* `OPENCLAW_SERVICE_KIND=gateway`
* `OPENCLAW_SERVICE_VERSION=<version>`

Rescue-Bot パターン: プロファイル、状態ディレクトリ、ワークスペース、およびベースポート間隔を分離した第2の Gateway を専用として保持します。詳細ガイド: [Rescue-bot guide](/ja/gateway/multiple-gateways#rescue-bot-guide)。

<div id="dev-profile-dev">
  ### Dev profile (`--dev`)
</div>

手っ取り早い方法: メインのセットアップには一切手を触れず、config/state/ワークスペース が完全に分離された開発用インスタンスを実行します。

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
# 次に開発インスタンスをターゲットにする:
openclaw --dev status
openclaw --dev health
```

デフォルト値（環境変数/フラグ/config でオーバーライド可能）:

* `OPENCLAW_STATE_DIR=~/.openclaw-dev`
* `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
* `OPENCLAW_GATEWAY_PORT=19001`（Gateway の WS + HTTP）
* ブラウザ制御サービスのポート = `19003`（算出元: `gateway.port+2`, ループバックのみ）
* `canvasHost.port=19005`（算出元: `gateway.port+4`）
* `--dev` 付きで `setup` / `onboard` を実行した場合、`agents.defaults.workspace` のデフォルトは `~/.openclaw/workspace-dev` になる。

算出ポート（経験則）:

* ベースポート = `gateway.port`（または `OPENCLAW_GATEWAY_PORT` / `--port`）
* ブラウザ制御サービスのポート = ベース + 2（ループバックのみ）
* `canvasHost.port = base + 4`（または `OPENCLAW_CANVAS_HOST_PORT` / config で上書き）
* ブラウザプロファイルの CDP ポートは `browser.controlPort + 9 .. + 108` の範囲から自動割り当て（プロファイルごとに永続化）。

インスタンスごとのチェックリスト:

* 一意な `gateway.port`
* 一意な `OPENCLAW_CONFIG_PATH`
* 一意な `OPENCLAW_STATE_DIR`
* 一意な `agents.defaults.workspace`
* 個別の WhatsApp 番号（WA を使用する場合）

プロファイルごとのサービスインストール:

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

例：

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

<div id="protocol-operator-view">
  ## プロトコル（オペレーター視点）
</div>

* 詳細なドキュメント: [Gateway protocol](/ja/gateway/protocol) および [Bridge protocol (legacy)](/ja/gateway/bridge-protocol)。
* クライアントから送信される必須の最初のフレーム: `req {type:"req", id, method:"connect", params:{minProtocol,maxProtocol,client:{id,displayName?,version,platform,deviceFamily?,modelIdentifier?,mode,instanceId?}, caps, auth?, locale?, userAgent? } }`。
* Gateway は `res {type:"res", id, ok:true, payload:hello-ok }`（または `ok:false` とエラーを返した後にクローズ）で応答する。
* ハンドシェイク後:
  * リクエスト: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
  * イベント: `{type:"event", event, payload, seq?, stateVersion?}`
* 構造化された presence エントリ: `{host, ip, version, platform?, deviceFamily?, modelIdentifier?, mode, lastInputSeconds?, ts, reason?, tags?[], instanceId? }`（WS クライアントの場合、`instanceId` は `connect.client.instanceId` から取得される）。
* `agent` のレスポンスは 2 段階構成になっている。まず `res` による ack `{runId,status:"accepted"}` が返され、その後、実行完了時に最終的な `res` `{runId,status:"ok"|"error",summary}` が返る。ストリーミング出力は `event:"agent"` として届く。

<div id="methods-initial-set">
  ## メソッド（初期セット）
</div>

* `health` — 完全なヘルススナップショット（`openclaw health --json` と同じ構造）。
* `status` — 簡潔な概要。
* `system-presence` — 現在のプレゼンス一覧。
* `system-event` — プレゼンス／システムのメモを投稿（構造化データ）。
* `send` — アクティブなチャネル経由でメッセージを送信。
* `agent` — エージェントのターンを実行（同じ接続上でイベントをストリーミングで返す）。
* `node.list` — ペアリング済み＋現在接続中のノードを一覧表示（`caps`, `deviceFamily`, `modelIdentifier`, `paired`, `connected` と、アドバタイズされている `commands` を含む）。
* `node.describe` — ノードの詳細を表示（ケイパビリティとサポートされている `node.invoke` コマンド。ペアリング済みノードと、現在接続中の未ペアリングノードの両方で動作）。
* `node.invoke` — ノード上のコマンドを呼び出し（例: `canvas.*`, `camera.*`）。
* `node.pair.*` — ペアリングのライフサイクル（`request`, `list`, `approve`, `reject`, `verify`）。

あわせて: プレゼンスの生成／重複排除方法と、安定した `client.instanceId` が重要となる理由については [Presence](/ja/concepts/presence) を参照。

<div id="events">
  ## Events
</div>

* `agent` — エージェント実行からストリーミングされるツール／出力イベント（シーケンス・タグ付き）。
* `presence` — すべての接続クライアントにプッシュされるプレゼンス更新（stateVersion を含む差分）。
* `tick` — 生存確認のための定期的なキープアライブ／no-op（処理なし）。
* `shutdown` — Gateway が終了します。ペイロードには `reason` と任意の `restartExpectedMs` が含まれます。クライアントは再接続すること。

<div id="webchat-integration">
  ## WebChat 連携
</div>

* WebChat はネイティブの SwiftUI ベースの UI で、履歴の取得や送信の実行、中断、各種イベント処理のために Gateway の WebSocket と直接通信します。
* リモート利用時も同じ SSH/Tailscale トンネルを経由します。Gateway トークンが設定されている場合、クライアントは `connect` 時にそれを含めて送信します。
* macOS アプリは単一の WS（共有接続）で接続し、最初のスナップショットから presence 情報を復元し、`presence` イベントを監視して UI を更新します。

<div id="typing-and-validation">
  ## 型付けとバリデーション
</div>

* サーバーは、プロトコル定義から生成された JSON Schema に対して、AJV を用いて受信するすべてのフレームを検証します。
* クライアント（TS/Swift）は生成済みの型を利用します（TS は直接、Swift はリポジトリのジェネレーター経由）。
* プロトコル定義が唯一の信頼できる情報源です。スキーマ／モデルは次のコマンドで再生成します:
  * `pnpm protocol:gen`
  * `pnpm protocol:gen:swift`

<div id="connection-snapshot">
  ## 接続スナップショット
</div>

* `hello-ok` には、`presence`、`health`、`stateVersion`、`uptimeMs` に加え、`policy {maxPayload,maxBufferedBytes,tickIntervalMs}` を含む `snapshot` が含まれており、クライアントは追加のリクエストなしに即座にレンダリングできます。
* `health` / `system-presence` は手動での更新用として引き続き利用可能ですが、接続時に必須ではありません。

<div id="error-codes-reserror-shape">
  ## エラーコード（`res.error` の構造）
</div>

* エラーは `{ code, message, details?, retryable?, retryAfterMs? }` の形式です。
* 標準コード:
  * `NOT_LINKED` — WhatsApp が認証されていません。
  * `AGENT_TIMEOUT` — エージェントが設定された期限内に応答しませんでした。
  * `INVALID_REQUEST` — スキーマ／パラメータの検証に失敗しました。
  * `UNAVAILABLE` — Gateway がシャットダウン中、または依存コンポーネントが利用できません。

<div id="keepalive-behavior">
  ## キープアライブ動作
</div>

* クライアントが、トラフィックが発生していない場合でも Gateway が稼働していることを認識できるよう、`tick` イベント（または WS の ping/pong）が定期的に送信されます。
* 送信／エージェントの確認応答は、それぞれ独立したレスポンスとして扱い、送信に対して tick を流用しないでください。

<div id="replay-gaps">
  ## リプレイ / ギャップ
</div>

* イベントはリプレイされません。クライアントは seq のギャップを検出したら、続行する前に `health` と `system-presence` を呼び出してリフレッシュする必要があります。WebChat と macOS クライアントは、現在ギャップ発生時に自動でリフレッシュします。

<div id="supervision-macos-example">
  ## 監視（macOS の例）
</div>

* launchd を使用してサービスを常駐させる:
  * Program: `openclaw` へのパス
  * Arguments: `gateway`
  * KeepAlive: true
  * StandardOut/Err: ファイルパスまたは `syslog`
* 障害発生時には launchd が再起動を行う。致命的な誤設定の場合は、プロセスが繰り返し終了し続けることでオペレーターが異常に気付ける。
* LaunchAgents はユーザー単位で、ログイン中のセッションが必要となる。ヘッドレス構成ではカスタム LaunchDaemon（同梱されていない）を使用する。
  * `openclaw gateway install` は `~/Library/LaunchAgents/bot.molt.gateway.plist`
    （または `bot.molt.<profile>.plist`。レガシーな `com.openclaw.*` はクリーンアップされる）を書き込む。
  * `openclaw doctor` は LaunchAgent の設定を検査し、現在のデフォルトに更新できる。

<div id="gateway-service-management-cli">
  ## Gateway サービス管理 (CLI)
</div>

インストール・起動・停止・再起動・状態確認には Gateway CLI を使用します:

```bash
openclaw gateway status
openclaw gateway install
openclaw gateway stop
openclaw gateway restart
openclaw logs --follow
```

Notes:

* `gateway status` は、デフォルトでサービスの解決済みポート/設定を使って Gateway の RPC をプローブします（`--url` で上書き可能）。
* `gateway status --deep` は、システムレベルのスキャン（LaunchDaemons/system units）を追加します。
* `gateway status --no-probe` は RPC プローブをスキップします（ネットワークダウン時に有用）。
* `gateway status --json` は、スクリプトからの利用に適した安定した出力形式です。
* `gateway status` は **スーパーバイザの実行状態**（launchd/systemd が稼働中か）と **RPC 到達性**（WS 接続 + status RPC）を分けて報告します。
* `gateway status` は、「localhost と LAN バインドの取り違え」やプロファイル不整合を避けるために、設定ファイルパスとプローブ対象を表示します。
* `gateway status` は、サービスが動いているように見えるがポートが閉じている場合、直近の gateway エラーログ 1 行を含めて表示します。
* `logs` は RPC 経由で Gateway のファイルログを tail します（手動で `tail`/`grep` する必要はありません）。
* 他の gateway 風サービスが検出された場合、それらが OpenClaw プロファイルサービスでない限り、CLI が警告を出します。
  ほとんどの構成では **1 マシンあたり 1 つの gateway** を推奨します。冗長構成やレスキューボット用途では、分離されたプロファイル/ポートを使用してください。[Multiple gateways](/ja/gateway/multiple-gateways) を参照してください。
  * クリーンアップ: `openclaw gateway uninstall`（現在のサービス）および `openclaw doctor`（レガシー移行処理）。
* `gateway install` は、すでにインストール済みの場合は何もしません。再インストール（プロファイル/環境変数/パスの変更）には `openclaw gateway install --force` を使用します。

Bundled mac app:

* OpenClaw.app は Node ベースの gateway リレーを同梱でき、ユーザーごとの LaunchAgent をラベル
  `bot.molt.gateway`（または `bot.molt.<profile>`。レガシーの `com.openclaw.*` ラベルも引き続き正常にアンロードされます）としてインストールできます。
* 正常に停止するには、`openclaw gateway stop`（または `launchctl bootout gui/$UID/bot.molt.gateway`）を使用します。
* 再起動するには、`openclaw gateway restart`（または `launchctl kickstart -k gui/$UID/bot.molt.gateway`）を使用します。
  * `launchctl` は LaunchAgent がインストールされている場合にのみ機能します。それ以外の場合は、先に `openclaw gateway install` を実行してください。
  * 名前付きプロファイルを実行している場合は、ラベルを `bot.molt.<profile>` に置き換えてください。

<div id="supervision-systemd-user-unit">
  ## 監視（systemd ユーザーサービス）
</div>

OpenClaw は Linux/WSL2 上にデフォルトで **systemd ユーザーサービス** をインストールします。
単一ユーザーのマシン（環境がシンプルで、ユーザーごとの設定にしたい場合）では、ユーザーサービスの利用を推奨します。
マルチユーザー環境や常時稼働させるサーバー（`lingering` 設定が不要で、共有のサービス監視が必要な場合）では、
**systemd システムサービス** を使用してください。

`openclaw gateway install` はユーザーユニットファイルを作成します。`openclaw doctor` はその
ユニットを診断し、現在推奨されるデフォルト設定に一致するように更新できます。

`~/.config/systemd/user/openclaw-gateway[-<profile>].service` を作成してください:

```
[Unit]
Description=OpenClaw Gateway (profile: <profile>, v<version>)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5
Environment=OPENCLAW_GATEWAY_TOKEN=
WorkingDirectory=/home/youruser

[Install]
WantedBy=default.target
```

linger を有効にします（ユーザーサービスをログアウト／アイドル状態後も維持するために必須です）:

```
sudo loginctl enable-linger youruser
```

このオンボーディング手順では、Linux/WSL2 上でこれを実行します（`sudo` の入力を求められる場合があり、`/var/lib/systemd/linger` に書き込みます）。
その後、サービスを有効化します。

```
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

**代替案（system サービス）** - 常時稼働またはマルチユーザーのサーバーでは、
ユーザーユニットの代わりに systemd の **system** ユニットをインストールできます（linger の設定は不要です）。
`/etc/systemd/system/openclaw-gateway[-<profile>].service` を作成し（上記のユニットをコピーし、
`WantedBy=multi-user.target` に変更し、`User=` と `WorkingDirectory=` を設定）、そのうえで次を実行します:

```
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

<div id="windows-wsl2">
  ## Windows (WSL2)
</div>

Windows でのインストールには **WSL2** を使用し、上記の Linux systemd セクションに従ってください。

<div id="operational-checks">
  ## 運用チェック
</div>

* 稼働確認（Liveness）：WS を開き、`req:connect` を送信 → スナップショット付きで `payload.type="hello-ok"` を含む `res` が返ることを確認。
* 準備完了確認（Readiness）：`health` を呼び出す → `ok: true` であり、（該当する場合）`linkChannel` にリンク済みチャネルが含まれていることを確認。
* デバッグ：`tick` および `presence` イベントを購読し、`status` にリンク／認証の経過時間が表示されていること、`presence` エントリに Gateway ホストおよび接続クライアントが表示されていることを確認。

<div id="safety-guarantees">
  ## 安全性の保証
</div>

* デフォルトではホストごとに 1 つの Gateway を前提とします。複数のプロファイルを実行する場合は、ポートと状態を分離し、必ず正しいインスタンスをターゲットにしてください。
* Baileys への直接接続へのフォールバックは行いません。Gateway がダウンしている場合、送信はすぐに失敗します（fail fast）。
* 最初のフレームが接続用ではない、または JSON が不正な場合は拒否し、ソケットをクローズします。
* グレースフルシャットダウン: クローズ前に `shutdown` イベントを発行します。クライアントはクローズと再接続処理を必ず実装してください。

<div id="cli-helpers">
  ## CLI ヘルパー
</div>

* `openclaw gateway health|status` — Gateway の WS 経由でヘルスチェック/ステータスを要求します。
* `openclaw message send --target <num> --message "hi" [--media ...]` — Gateway 経由で送信します（WhatsApp 向けには冪等的です）。
* `openclaw agent --message "hi" --to <num>` — エージェントのターンを実行します（デフォルトで最終応答まで待機します）。
* `openclaw gateway call <method> --params '{"k":"v"}'` — デバッグ用の生のメソッド呼び出しコマンドです。
* `openclaw gateway stop|restart` — 監視対象の Gateway サービス（launchd/systemd）を停止/再起動します。
* Gateway 用ヘルパーサブコマンドは、`--url` で指定された稼働中の Gateway を前提としており、もはや自動起動は行いません。

<div id="migration-guidance">
  ## 移行ガイダンス
</div>

* `openclaw gateway` とレガシーな TCP 制御ポートの使用を廃止してください。
* クライアントを更新し、必須の接続シーケンスと構造化された presence を伴う WS プロトコルで通信するようにしてください。