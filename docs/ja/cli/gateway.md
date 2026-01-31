---
title: Gateway
summary: "OpenClaw Gateway CLI (`openclaw gateway`) — Gateway を実行・問い合わせ・検出する"
read_when:
  - CLI から Gateway を実行するとき（開発環境またはサーバー）
  - Gateway の認証、バインドモード、接続状態をデバッグするとき
  - Bonjour 経由で Gateway を検出するとき（LAN + tailnet）
---

<div id="gateway-cli">
  # Gateway CLI
</div>

Gateway は OpenClaw の WebSocket サーバーです（チャネル、ノード、セッション、フックを扱います）。

このページで説明するサブコマンドは、すべて `openclaw gateway …` サブコマンドとして実行します。

関連ドキュメント:

* [/gateway/bonjour](/ja/gateway/bonjour)
* [/gateway/discovery](/ja/gateway/discovery)
* [/gateway/configuration](/ja/gateway/configuration)

<div id="run-the-gateway">
  ## Gateway を起動する
</div>

ローカルで Gateway プロセスを起動します:

```bash
openclaw gateway
```

フォアグラウンド実行用エイリアス：

```bash
openclaw gateway run
```

Notes:

* デフォルトでは、`~/.openclaw/openclaw.json` に `gateway.mode=local` が設定されていない限り、Gateway は起動しません。アドホック／開発用途での実行には `--allow-unconfigured` を使用してください。
* 認証なしでループバックアドレス以外へバインドすることはブロックされます（安全のためのガードレール）。
* `SIGUSR1` は、許可されている場合にプロセス内での再起動をトリガーします（`commands.restart` を有効化するか、Gateway ツール／config apply/update を使用してください）。
* `SIGINT`/`SIGTERM` ハンドラは Gateway プロセスを停止しますが、カスタムのターミナル状態は復元しません。CLI を TUI や raw モード入力でラップしている場合は、終了前にターミナルを復元してください。

<div id="options">
  ### オプション
</div>

* `--port <port>`: WebSocket ポート（デフォルト値は config/env から取得され、通常は `18789`）。
* `--bind <loopback|lan|tailnet|auto|custom>`: リスナーのバインドモード。
* `--auth <token|password>`: 認証モードを上書きする。
* `--token <token>`: トークンを上書きする（プロセスに対して `OPENCLAW_GATEWAY_TOKEN` も設定する）。
* `--password <password>`: パスワードを上書きする（プロセスに対して `OPENCLAW_GATEWAY_PASSWORD` も設定する）。
* `--tailscale <off|serve|funnel>`: Tailscale 経由で Gateway を公開する。
* `--tailscale-reset-on-exit`: シャットダウン時に Tailscale の serve/funnel 設定をリセットする。
* `--allow-unconfigured`: config に `gateway.mode=local` がなくても Gateway の起動を許可する。
* `--dev`: 開発用の config とワークスペースがなければ作成する（BOOTSTRAP.md をスキップする）。
* `--reset`: 開発用の config・認証情報・セッション・ワークスペースをリセットする（`--dev` が必要）。
* `--force`: 起動前に、選択したポートで動作している既存リスナーを強制終了する。
* `--verbose`: 詳細ログを有効にする。
* `--claude-cli-logs`: コンソールには claude-cli のログのみを表示する（あわせてその stdout/stderr を有効化する）。
* `--ws-log <auto|full|compact>`: WS ログのスタイル（デフォルトは `auto`）。
* `--compact`: `--ws-log compact` のエイリアス。
* `--raw-stream`: モデルの生ストリームイベントを jsonl 形式でログ出力する。
* `--raw-stream-path <path>`: 生ストリーム jsonl のパス。

<div id="query-a-running-gateway">
  ## 起動中の Gateway へのクエリ
</div>

すべてのクエリ系コマンドは WebSocket RPC を使用します。

出力モード:

* デフォルト: 人間が読みやすい形式（TTY ではカラー表示）。
* `--json`: マシン判読可能な JSON（スタイル/スピナーなし）。
* `--no-color`（または `NO_COLOR=1`）: レイアウトはそのままに ANSI カラーを無効化。

共通オプション（対応コマンドのみ）:

* `--url <url>`: Gateway の WebSocket URL。
* `--token <token>`: Gateway トークン。
* `--password <password>`: Gateway パスワード。
* `--timeout <ms>`: タイムアウト/時間予算（コマンドごとに異なる）。
* `--expect-final`: 「最終」レスポンスを待機（エージェント呼び出し時）。

<div id="gateway-health">
  ### `gateway health`
</div>

```bash
openclaw gateway health --url ws://127.0.0.1:18789
```

<div id="gateway-status">
  ### `gateway status`
</div>

`gateway status` は Gateway サービス（launchd/systemd/schtasks）および（オプションで）RPC プローブの結果を表示します。

```bash
openclaw gateway status
openclaw gateway status --json
```

Options:

* `--url &lt;url&gt;`: プローブ先の URL を上書き指定します。
* `--token &lt;token&gt;`: プローブでトークン認証を使用します。
* `--password &lt;password&gt;`: プローブでパスワード認証を使用します。
* `--timeout &lt;ms&gt;`: プローブのタイムアウト（デフォルトは `10000`）。
* `--no-probe`: RPC プローブをスキップします（サービスのみを表示）。
* `--deep`: システムレベルのサービスもスキャンします。

<div id="gateway-probe">
  ### `gateway probe`
</div>

`gateway probe` は「なんでもデバッグする」ためのコマンドです。常に次の対象をプローブします:

* 設定済みのリモート Gateway（設定されている場合）、および
* localhost（ループバック）— **リモートが設定されている場合でも必ず**

複数の Gateway に到達できる場合は、そのすべてを表示します。複数の Gateway 構成は、（レスキューボットのように）分離されたプロファイルやポートを使う場合にサポートされますが、ほとんどの環境では Gateway は 1 つだけが動作しています。

```bash
openclaw gateway probe
openclaw gateway probe --json
```

<div id="remote-over-ssh-mac-app-parity">
  #### SSH 経由のリモート接続（Mac アプリと同等）
</div>

macOS アプリの「Remote over SSH」モードはローカルポートフォワーディングを使用し、ループバックのみにバインドされている可能性があるリモート Gateway へ `ws://127.0.0.1:<port>` でアクセスできるようにします。

CLI での同等のコマンド:

```bash
openclaw gateway probe --ssh user@gateway-host
```

Options:

* `--ssh <target>`: `user@host` または `user@host:port`（ポートのデフォルトは `22`）。
* `--ssh-identity <path>`: SSH 秘密鍵ファイル。
* `--ssh-auto`: 検出された最初の Gateway ホストを SSH の接続先として選択します（LAN/WAB のみ）。

Config（任意、デフォルト値として使用）:

* `gateway.remote.sshTarget`
* `gateway.remote.sshIdentity`

<div id="gateway-call-method">
  ### `gateway call <method>`
</div>

低レベルな RPC 用の補助コマンド。

```bash
openclaw gateway call status
openclaw gateway call logs.tail --params '{"sinceMs": 60000}'
```

<div id="manage-the-gateway-service">
  ## Gateway サービスの管理
</div>

```bash
openclaw gateway install
openclaw gateway start
openclaw gateway stop
openclaw gateway restart
openclaw gateway uninstall
```

Notes:

* `gateway install` は `--port`、`--runtime`、`--token`、`--force`、`--json` をサポートします。
* ライフサイクルコマンドはスクリプト連携用に `--json` を受け付けます。

<div id="discover-gateways-bonjour">
  ## Gateway の検出 (Bonjour)
</div>

`gateway discover` は Gateway ビーコン (`_openclaw-gw._tcp`) をスキャンします。

* マルチキャスト DNS-SD: `local.`
* ユニキャスト DNS-SD (Wide-Area Bonjour): ドメインを選択し (例: `openclaw.internal.`)、スプリット DNS と DNS サーバーを構成します。[/gateway/bonjour](/ja/gateway/bonjour) を参照してください。

Bonjour 検出が有効になっている Gateway のみが (デフォルト) ビーコンをアドバタイズします。

Wide-Area 検出レコードには以下が含まれます (TXT):

* `role` (Gateway のロールに関するヒント)
* `transport` (トランスポート種別のヒント。例: `gateway`)
* `gatewayPort` (WebSocket ポート。通常は `18789`)
* `sshPort` (SSH ポート。指定がなければデフォルトで `22`)
* `tailnetDns` (利用可能な場合の MagicDNS ホスト名)
* `gatewayTls` / `gatewayTlsSha256` (TLS 有効フラグ + 証明書フィンガープリント)
* `cliPath` (リモート環境でのインストールパスに関する任意のヒント)

<div id="gateway-discover">
  ### `gateway discover`
</div>

```bash
openclaw gateway discover
```

Options:

* `--timeout <ms>`: コマンド単位のタイムアウト（browse/resolve）。デフォルトは `2000`。
* `--json`: 機械可読な出力（出力スタイルとスピナーも無効化）。

Examples:

```bash
openclaw gateway discover --timeout 4000
openclaw gateway discover --json | jq '.beacons[].wsUrl'
```
