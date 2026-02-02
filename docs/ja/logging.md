---
title: ロギング
summary: "ロギングの概要: ファイルログ、コンソール出力、CLI のテイル表示、Control UI"
read_when:
  - ロギングについての初心者向け概要が必要なとき
  - ログレベルやフォーマットを設定したいとき
  - トラブルシューティング中で、すぐにログを確認する必要があるとき
---

<div id="logging">
  # ログ
</div>

OpenClaw は次の 2 か所にログを出力します:

* Gateway によって書き込まれる **ファイルログ**（JSON 行形式）。
* ターミナルおよび Control UI に表示される **コンソール出力**。

このページでは、ログの場所、ログの読み取り方法、およびログレベルやフォーマットの設定方法について説明します。

<div id="where-logs-live">
  ## ログの保存場所
</div>

デフォルトでは、Gateway はローテーションされるログファイルを次のパスに出力します：

`/tmp/openclaw/openclaw-YYYY-MM-DD.log`

日付部分には、Gateway が動作しているホストのローカルタイムゾーンが使用されます。

この設定は `~/.openclaw/openclaw.json` で上書きできます：

```json
{
  "logging": {
    "file": "/path/to/openclaw.log"
  }
}
```

<div id="how-to-read-logs">
  ## ログの確認方法
</div>

<div id="cli-live-tail-recommended">
  ### CLI: リアルタイム tail（推奨）
</div>

CLI を使用して、RPC 経由で Gateway のログファイルをリアルタイムで tail します：

```bash
openclaw logs --follow
```

出力モード:

* **TTY セッション**: 見やすいカラー表示の構造化ログ行。
* **非 TTY セッション**: プレーンテキスト。
* `--json`: 行区切りの JSON（1 行につき 1 つのログイベント）。
* `--plain`: TTY セッションでもプレーンテキストを強制。
* `--no-color`: ANSI カラーを無効化。

JSON モードでは、CLI は `type` タグ付きオブジェクトを出力します:

* `meta`: ストリームのメタデータ（ファイル、カーソル、サイズ）
* `log`: 解析済みログエントリ
* `notice`: 省略 / ローテーションに関するヒント
* `raw`: 未解析のログ行

Gateway に到達できない場合、CLI は次を実行するための簡単なヒントを出力します:

```bash
openclaw doctor
```

<div id="control-ui-web">
  ### Control UI（ウェブ）
</div>

Control UI の **Logs** タブは、`logs.tail` を使用して同じファイルの末尾を追跡します。
開き方については [/web/control-ui](/ja/web/control-ui) を参照してください。

<div id="channel-only-logs">
  ### チャンネル専用ログ
</div>

チャンネルのアクティビティ（WhatsApp/Telegram など）だけをフィルタリングするには、次を使用します:

```bash
openclaw channels logs --channel whatsapp
```

<div id="log-formats">
  ## ログ形式
</div>

<div id="file-logs-jsonl">
  ### ファイルログ（JSONL）
</div>

ログファイルの各行は JSON オブジェクトになっています。CLI と Control UI はこれらのエントリを解析し、時刻、レベル、サブシステム、メッセージといった情報を含む構造化された出力として表示します。

<div id="console-output">
  ### コンソール出力
</div>

コンソールログは **TTY 対応** で、可読性を重視した形式になります：

* サブシステムのプレフィックス（例: `gateway/channels/whatsapp`）
* レベルごとの色分け（info/warn/error）
* 任意のコンパクトモードまたは JSON モード

コンソールのフォーマットは `logging.consoleStyle` によって制御されます。

<div id="configuring-logging">
  ## ログの設定
</div>

すべてのログ関連の設定は、`~/.openclaw/openclaw.json` の `logging` セクションで行います。

```json
{
  "logging": {
    "level": "info",
    "file": "/tmp/openclaw/openclaw-YYYY-MM-DD.log",
    "consoleLevel": "info",
    "consoleStyle": "pretty",
    "redactSensitive": "tools",
    "redactPatterns": [
      "sk-.*"
    ]
  }
}
```

<div id="log-levels">
  ### ログレベル
</div>

* `logging.level`: **ファイルログ**（JSONL）の出力レベル。
* `logging.consoleLevel`: **コンソール**出力の詳細度レベル。

`--verbose` はコンソール出力のみに影響し、ファイルログのレベルは変更しません。

<div id="console-styles">
  ### コンソールスタイル
</div>

`logging.consoleStyle`:

* `pretty`: 人間が読みやすい形式で、カラー表示かつタイムスタンプ付き。
* `compact`: 出力がコンパクト（長時間のセッションに最適）。
* `json`: 1 行ごとの JSON（ログ処理ツール向け）。

<div id="redaction">
  ### マスキング（Redaction）
</div>

ツールのサマリーは、コンソールに出力される前に機密トークンをマスキングできます。

* `logging.redactSensitive`: `off` | `tools`（デフォルト: `tools`）
* `logging.redactPatterns`: 既定のセットを上書きするための正規表現文字列のリスト

マスキングは**コンソール出力のみに**影響し、ファイルログは変更しません。

<div id="diagnostics-opentelemetry">
  ## Diagnostics + OpenTelemetry
</div>

Diagnostics は、モデルの実行 **および**
メッセージフローのテレメトリ（Webhook、キュー処理、セッション状態）のための、構造化された機械可読イベントです。ログを置き換えるものではなく、メトリクスやトレース、その他のエクスポーターにデータを供給するために存在します。

Diagnostics イベントはプロセス内で発行されますが、エクスポーターは、Diagnostics とエクスポーター用プラグインが両方とも有効化されている場合にのみアタッチされます。

<div id="opentelemetry-vs-otlp">
  ### OpenTelemetry と OTLP
</div>

* **OpenTelemetry (OTel)**: トレース・メトリクス・ログ用のデータモデルと SDK 群。
* **OTLP**: OTel データをコレクタ／バックエンドにエクスポートするためのワイヤプロトコル。
* OpenClaw は現在 **OTLP/HTTP (protobuf)** 経由でエクスポートします。

<div id="signals-exported">
  ### エクスポートされるシグナル
</div>

* **メトリクス**: カウンタとヒストグラム（トークン使用量、メッセージフロー、キューイング）。
* **トレース**: モデル利用および webhook/メッセージ処理のスパン。
* **ログ**: `diagnostics.otel.logs` が有効な場合に OTLP 経由でエクスポートされます。ログ
  量は多くなり得るため、`logging.level` とエクスポーター側のフィルタを考慮してください。

<div id="diagnostic-event-catalog">
  ### 診断イベントカタログ
</div>

モデル利用状況:

* `model.usage`: トークン数、コスト、所要時間、コンテキスト、プロバイダー/モデル/チャネル、セッション ID。

メッセージフロー:

* `webhook.received`: チャネルごとの Webhook 受信。
* `webhook.processed`: Webhook 処理完了 + 所要時間。
* `webhook.error`: Webhook ハンドラーのエラー。
* `message.queued`: 処理用にキュー投入されたメッセージ。
* `message.processed`: 結果 + 所要時間 + 任意のエラー。

キュー + セッション:

* `queue.lane.enqueue`: コマンドキューレーンへの enqueue + 深さ。
* `queue.lane.dequeue`: コマンドキューレーンからの dequeue + 待ち時間。
* `session.state`: セッション状態遷移 + 理由。
* `session.stuck`: セッションハングの警告 + 経過時間。
* `run.attempt`: 実行の再試行/試行メタデータ。
* `diagnostic.heartbeat`: 集計カウンター (webhook/queue/session)。

<div id="enable-diagnostics-no-exporter">
  ### 診断を有効化する（エクスポーターなし）
</div>

プラグインやカスタムシンクで診断イベントを利用できるようにしたい場合に使います。

```json
{
  "diagnostics": {
    "enabled": true
  }
}
```

<div id="diagnostics-flags-targeted-logs">
  ### Diagnostics フラグ（対象ログ）
</div>

`logging.level` を引き上げることなく、フラグを使って特定対象向けの追加デバッグログを有効にできます。
フラグは大文字小文字を区別せず、ワイルドカード（例: `telegram.*` や `*`）もサポートします。

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

環境変数の一回限りの上書き:

```
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload
```

注意:

* フラグのログは、標準のログファイル（`logging.file` と同じ）に書き込まれます。
* 出力は、`logging.redactSensitive` の設定に従って引き続きマスクされます。
* 詳細なガイド: [/diagnostics/flags](/ja/diagnostics/flags)。

<div id="export-to-opentelemetry">
  ### OpenTelemetry へのエクスポート
</div>

診断情報は `diagnostics-otel` プラグイン (OTLP/HTTP) を通じてエクスポートできます。
これは、OTLP/HTTP を受け付けるあらゆる OpenTelemetry コレクター／バックエンドで動作します。

```json
{
  "plugins": {
    "allow": ["diagnostics-otel"],
    "entries": {
      "diagnostics-otel": {
        "enabled": true
      }
    }
  },
  "diagnostics": {
    "enabled": true,
    "otel": {
      "enabled": true,
      "endpoint": "http://otel-collector:4318",
      "protocol": "http/protobuf",
      "serviceName": "openclaw-gateway",
      "traces": true,
      "metrics": true,
      "logs": true,
      "sampleRate": 0.2,
      "flushIntervalMs": 60000
    }
  }
}
```

注意:

* `openclaw plugins enable diagnostics-otel` でもプラグインを有効化できます。
* `protocol` は現在 `http/protobuf` のみをサポートします。`grpc` は無視されます。
* メトリクスには、トークン使用量、コスト、コンテキストサイズ、実行時間、およびメッセージフロー
  カウンタ/ヒストグラム（webhook、キューイング、セッション状態、キュー深度/待ち時間）が含まれます。
* トレースとメトリクスは `traces` / `metrics` で切り替え可能です（デフォルト: オン）。トレースには、
  有効化されている場合、モデル使用スパンに加えて webhook/メッセージ処理スパンが含まれます。
* コレクタが認証を要求する場合は `headers` を設定してください。
* サポートされる環境変数: `OTEL_EXPORTER_OTLP_ENDPOINT`,
  `OTEL_SERVICE_NAME`, `OTEL_EXPORTER_OTLP_PROTOCOL`。

<div id="exported-metrics-names-types">
  ### エクスポートされるメトリクス（名前と型）
</div>

モデル利用状況:

* `openclaw.tokens`（カウンター、属性: `openclaw.token`, `openclaw.channel`,
  `openclaw.provider`, `openclaw.model`)
* `openclaw.cost.usd`（カウンター、属性: `openclaw.channel`, `openclaw.provider`,
  `openclaw.model`)
* `openclaw.run.duration_ms`（ヒストグラム、属性: `openclaw.channel`,
  `openclaw.provider`, `openclaw.model`)
* `openclaw.context.tokens`（ヒストグラム、属性: `openclaw.context`,
  `openclaw.channel`, `openclaw.provider`, `openclaw.model`)

メッセージフロー:

* `openclaw.webhook.received`（カウンター、属性: `openclaw.channel`,
  `openclaw.webhook`)
* `openclaw.webhook.error`（カウンター、属性: `openclaw.channel`,
  `openclaw.webhook`)
* `openclaw.webhook.duration_ms`（ヒストグラム、属性: `openclaw.channel`,
  `openclaw.webhook`)
* `openclaw.message.queued`（カウンター、属性: `openclaw.channel`,
  `openclaw.source`)
* `openclaw.message.processed`（カウンター、属性: `openclaw.channel`,
  `openclaw.outcome`)
* `openclaw.message.duration_ms`（ヒストグラム、属性: `openclaw.channel`,
  `openclaw.outcome`)

キューおよびセッション:

* `openclaw.queue.lane.enqueue`（カウンター、属性: `openclaw.lane`)
* `openclaw.queue.lane.dequeue`（カウンター、属性: `openclaw.lane`)
* `openclaw.queue.depth`（ヒストグラム、属性: `openclaw.lane` または
  `openclaw.channel=heartbeat`)
* `openclaw.queue.wait_ms`（ヒストグラム、属性: `openclaw.lane`)
* `openclaw.session.state`（カウンター、属性: `openclaw.state`, `openclaw.reason`)
* `openclaw.session.stuck`（カウンター、属性: `openclaw.state`)
* `openclaw.session.stuck_age_ms`（ヒストグラム、属性: `openclaw.state`)
* `openclaw.run.attempt`（カウンター、属性: `openclaw.attempt`)

<div id="exported-spans-names-key-attributes">
  ### エクスポートされるスパン（名前と主要属性）
</div>

* `openclaw.model.usage`
  * `openclaw.channel`, `openclaw.provider`, `openclaw.model`
  * `openclaw.sessionKey`, `openclaw.sessionId`
  * `openclaw.tokens.*` (input/output/cache&#95;read/cache&#95;write/total)
* `openclaw.webhook.processed`
  * `openclaw.channel`, `openclaw.webhook`, `openclaw.chatId`
* `openclaw.webhook.error`
  * `openclaw.channel`, `openclaw.webhook`, `openclaw.chatId`,
    `openclaw.error`
* `openclaw.message.processed`
  * `openclaw.channel`, `openclaw.outcome`, `openclaw.chatId`,
    `openclaw.messageId`, `openclaw.sessionKey`, `openclaw.sessionId`,
    `openclaw.reason`
* `openclaw.session.stuck`
  * `openclaw.state`, `openclaw.ageMs`, `openclaw.queueDepth`,
    `openclaw.sessionKey`, `openclaw.sessionId`

<div id="sampling-flushing">
  ### サンプリングとフラッシュ
</div>

* トレースサンプリング: `diagnostics.otel.sampleRate` (0.0～1.0、ルートスパンのみ)。
* メトリクスのエクスポート間隔: `diagnostics.otel.flushIntervalMs` (最小 1000ms)。

<div id="protocol-notes">
  ### プロトコルに関する注意事項
</div>

* OTLP/HTTP エンドポイントは `diagnostics.otel.endpoint` または
  `OTEL_EXPORTER_OTLP_ENDPOINT` で設定できます。
* エンドポイントにすでに `/v1/traces` または `/v1/metrics` が含まれている場合、そのまま使用されます。
* エンドポイントにすでに `/v1/logs` が含まれている場合、ログについてはそのまま使用されます。
* `diagnostics.otel.logs` はメインロガー出力の OTLP ログエクスポートを有効にします。

<div id="log-export-behavior">
  ### ログのエクスポート動作
</div>

* OTLP ログは、`logging.file` に書き込まれるものと同じ構造化レコードを使用します。
* `logging.level`（ファイルのログレベル）に従います。コンソール出力でのレダクションは OTLP ログには**適用されません**。
* 大規模環境や高トラフィック環境では、OTLP Collector 側でのサンプリング／フィルタリングを優先して利用してください。

<div id="troubleshooting-tips">
  ## トラブルシューティングのヒント
</div>

* **Gateway に接続できない？** まず `openclaw doctor` を実行してください。
* **ログが空？** Gateway が起動していて、`logging.file` で指定したファイルパスに書き込んでいるか確認してください。
* **さらに詳細が必要？** `logging.level` を `debug` または `trace` に設定してから再実行してください。