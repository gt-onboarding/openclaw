---
title: ロギング
summary: "ログ出力先、ファイルログ、WS ログスタイル、コンソール出力フォーマット"
read_when:
  - ログ出力やフォーマットを変更する場合
  - CLI や Gateway の出力をデバッグする場合
---

<div id="logging">
  # ログ
</div>

ユーザー向けの概要（CLI + Control UI + 設定）については、[/logging](/ja/logging) を参照してください。

OpenClaw には、ログの「出力先」が 2 種類あります:

* **コンソール出力**（ターミナル / Debug UI に表示されるもの）
* Gateway ロガーによって出力される **ファイルログ**（JSON Lines 形式）

<div id="file-based-logger">
  ## ファイルベースのロガー
</div>

* デフォルトのローテーションログファイルは `/tmp/openclaw/` 配下にあります（1 日につき 1 ファイル）：`openclaw-YYYY-MM-DD.log`
  * 日付には Gateway ホストのローカルタイムゾーンが使われます。
* ログファイルのパスとレベルは `~/.openclaw/openclaw.json` で設定できます:
  * `logging.file`
  * `logging.level`

ファイル形式は 1 行につき 1 つの JSON オブジェクトです。

Control UI の Logs タブは、Gateway 経由でこのファイルを tail します（`logs.tail`）。
CLI でも同じことができます:

```bash
openclaw logs --follow
```

**詳細出力とログレベル**

* **ファイルログ** は `logging.level` のみで制御されます。
* `--verbose` は **コンソールの詳細度**（および WS のログスタイル）にだけ影響し、ファイルログのレベルを **引き上げることはありません**。
* `--verbose` でのみ出力される詳細情報もファイルログに記録したい場合は、`logging.level` を `debug` または
  `trace` に設定します。

<div id="console-capture">
  ## コンソールキャプチャ
</div>

CLI は `console.log/info/warn/error/debug/trace` をキャプチャしてログファイルに書き出す一方で、
標準出力／標準エラー出力にも引き続き表示します。

コンソール出力の冗長度は、次の設定で個別に調整できます。

* `logging.consoleLevel` (デフォルト `info`)
* `logging.consoleStyle` (`pretty` | `compact` | `json`)

<div id="tool-summary-redaction">
  ## ツール要約のマスキング
</div>

詳細なツール要約（例: `🛠️ Exec: ...`）では、コンソールストリームに出力される前に
機密トークンをマスクできます。これは**ツール専用**であり、ファイルログは変更されません。

* `logging.redactSensitive`: `off` | `tools`（デフォルト: `tools`）
* `logging.redactPatterns`: 正規表現文字列の配列（デフォルトを上書き）
  * 生の正規表現文字列（自動で `gi`）か、カスタムフラグが必要な場合は `/pattern/flags` を使用します。
  * マッチした文字列は、長さが 18 文字以上の場合は先頭 6 文字 + 末尾 4 文字を残してマスクされ、それ以外は `***` に置き換えられます。
  * デフォルトでは、一般的なキーの代入、CLI フラグ、JSON フィールド、Bearer 認証ヘッダー、PEM ブロック、よく使われるトークンのプレフィックスをカバーします。

<div id="gateway-websocket-logs">
  ## Gateway WebSocket ログ
</div>

Gateway は WebSocket プロトコルレベルのログを 2 つのモードで出力します:

* **通常モード（`--verbose` なし）**: 「重要な」RPC 結果のみをログ出力します:
  * エラー（`ok=false`）
  * 遅い呼び出し（デフォルトのしきい値: `>= 50ms`）
  * パースエラー
* **詳細モード（`--verbose`）**: すべての WS リクエスト/レスポンスのトラフィックをログ出力します。

<div id="ws-log-style">
  ### WS ログスタイル
</div>

`openclaw gateway` は Gateway 単位でスタイルを切り替えられます:

* `--ws-log auto` (デフォルト): 通常モードでは最適化された出力、詳細モードではコンパクトな出力を使用
* `--ws-log compact`: 詳細モード時にコンパクトな出力（リクエスト/レスポンスのペア表示）
* `--ws-log full`: 詳細モード時にフレーム単位のフル出力
* `--compact`: `--ws-log compact` のエイリアス

例:

```bash
# optimized (only errors/slow)
openclaw gateway

# show all WS traffic (paired)
openclaw gateway --verbose --ws-log compact

# すべてのWSトラフィックを表示（完全なメタデータ）
openclaw gateway --verbose --ws-log full
```

<div id="console-formatting-subsystem-logging">
  ## コンソールのフォーマット（サブシステムロギング）
</div>

コンソールフォーマッタは **TTY を認識** し、一貫したプレフィックス付きの行を出力します。
サブシステムロガーによって、出力はまとまりがあり、ざっと目を通しやすくなります。

動作仕様:

* **サブシステムのプレフィックス** を各行に付与（例: `[gateway]`, `[canvas]`, `[tailscale]`）
* **サブシステムごとの色分け**（サブシステムごとに安定した色）に加え、ログレベルによる色分け
* **出力先が TTY または環境がリッチなターミナルと判定された場合にカラー出力**（`TERM` / `COLORTERM` / `TERM_PROGRAM` を参照）、`NO_COLOR` を尊重
* **サブシステムプレフィックスの短縮**: 先頭の `gateway/` および `channels/` を削除し、末尾 2 セグメントを保持（例: `whatsapp/outbound`）
* **サブシステムごとのサブログガー**（自動プレフィックス + 構造化フィールド `{ subsystem }`）
* **`logRaw()`** は QR/UX 出力用（プレフィックスなし、フォーマットなし）
* **コンソールスタイル**（例: `pretty | compact | json`）
* **コンソールのログレベル** はファイルのログレベルとは別設定（`logging.level` が `debug` / `trace` のとき、ファイルログ側は完全な詳細を保持）
* **WhatsApp メッセージ本文** は `debug` レベルでログ出力される（表示するには `--verbose` を使用）

これにより、既存のファイルログは安定したまま、対話的な出力がざっと確認しやすくなります。