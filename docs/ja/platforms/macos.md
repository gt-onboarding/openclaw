---
title: macOS
summary: "OpenClaw macOS コンパニオンアプリ（メニューバー + Gateway ブローカー）"
read_when:
  - macOS アプリの機能を実装する場合
  - macOS 上の Gateway のライフサイクルやノードブリッジを変更する場合
---

<div id="openclaw-macos-companion-menu-bar-gateway-broker">
  # OpenClaw macOS Companion (メニューバー + Gateway ブローカー)
</div>

この macOS アプリは、OpenClaw 用の **メニューバーコンパニオン**です。必要な権限を保持し、
Gateway をローカルで管理・接続（launchd または手動）し、macOS の
機能をノードとしてエージェントに公開します。

<div id="what-it-does">
  ## 機能
</div>

* メニューバーにネイティブ通知とステータスを表示します。
* TCC プロンプト（通知、アクセシビリティ、画面収録、マイク、
  音声認識、自動化/AppleScript）を一元的に扱います。
* Gateway を（ローカルまたはリモートで）起動または接続します。
* macOS 専用ツール（Canvas、Camera、画面収録、`system.run`）を提供します。
* **remote** モードではローカルのノードホストサービスを（launchd で）起動し、**local** モードでは停止します。
* オプションで UI 自動化のために **PeekabooBridge** をホストします。
* 要求に応じて npm/pnpm 経由でグローバル CLI（`openclaw`）をインストールします（Gateway ランタイムでの bun の使用は推奨されません）。

<div id="local-vs-remote-mode">
  ## ローカルモードとリモートモード
</div>

* **Local**（デフォルト）：アプリは、ローカルで Gateway が稼働していればそれに接続し、
  稼働していなければ `openclaw gateway install` を使って launchd サービスを有効化します。
* **Remote**：アプリは SSH/Tailscale 経由で Gateway に接続し、
  ローカルの Gateway プロセスは起動しません。
  アプリは、この Mac にリモートの Gateway からアクセスできるように、ローカルの **ノードホストサービス** を起動します。
  アプリは Gateway を子プロセスとして生成しません。

<div id="launchd-control">
  ## Launchd 制御
</div>

このアプリは、ユーザーごとの LaunchAgent を `bot.molt.gateway`
（`--profile` / `OPENCLAW_PROFILE` を使用する場合は `bot.molt.<profile>`、旧来の `com.openclaw.*` も引き続き unload されます）というラベルで管理します。

```bash
launchctl kickstart -k gui/$UID/bot.molt.gateway
launchctl bootout gui/$UID/bot.molt.gateway
```

名前付きプロファイルを実行している場合は、ラベルを `bot.molt.<profile>` に変更してください。

LaunchAgent がインストールされていない場合は、アプリから有効化するか、`openclaw gateway install` コマンドを実行してください。

<div id="node-capabilities-mac">
  ## ノードの機能 (mac)
</div>

macOS アプリはノードとして動作します。主なコマンドは次のとおりです:

* Canvas: `canvas.present`, `canvas.navigate`, `canvas.eval`, `canvas.snapshot`, `canvas.a2ui.*`
* Camera: `camera.snap`, `camera.clip`
* Screen: `screen.record`
* System: `system.run`, `system.notify`

ノードは `permissions` マップを送信し、エージェントが何が許可されているかを判断できるようにします。

ノードサービスとアプリ間の IPC:

* ヘッドレスのノードホストサービスが実行中 (リモートモード) のとき、Gateway の WS にノードとして接続します。
* `system.run` はローカル Unix ソケット経由で macOS アプリ (UI/TCC コンテキスト) 内で実行され、プロンプトと出力はアプリ内にとどまります。

ダイアグラム (SCI):

```
Gateway -> Node Service (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac App (UI + TCC + system.run)
```

<div id="exec-approvals-systemrun">
  ## Exec approvals (system.run)
</div>

`system.run` は、macOS アプリの **Exec approvals**（Settings → Exec approvals）によって制御されます。
Security、ask、許可リストは Mac 上にローカルで保存されます:

```
~/.openclaw/exec-approvals.json
```

例：

```json
{
  "version": 1,
  "defaults": {
    "security": "deny",
    "ask": "on-miss"
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [
        { "pattern": "/opt/homebrew/bin/rg" }
      ]
    }
  }
}
```

Notes:

* `allowlist` のエントリは、解決済みのバイナリパスに対するグロブパターンです。
* プロンプトで「Always Allow」を選択すると、そのコマンドが許可リストに追加されます。
* `system.run` の環境変数のオーバーライドはフィルタリングされ（`PATH`, `DYLD_*`, `LD_*`, `NODE_OPTIONS`, `PYTHON*`, `PERL*`, `RUBYOPT` を削除）、その後アプリの環境と結合されます。

<div id="deep-links">
  ## ディープリンク
</div>

このアプリは、ローカルアクション用に `openclaw://` という URL スキームを登録します。

<div id="openclawagent">
  ### `openclaw://agent`
</div>

Gateway に対する `agent` リクエストを発行します。

```bash
open 'openclaw://agent?message=Hello%20from%20deep%20link'
```

クエリパラメータ:

* `message` (必須)
* `sessionKey` (任意)
* `thinking` (任意)
* `deliver` / `to` / `channel` (任意)
* `timeoutSeconds` (任意)
* `key` (任意の無人実行モード用キー)

安全性:

* `key` がない場合、アプリは確認を求めます。
* 有効な `key` がある場合、実行はユーザーの操作なしに行われます（個人向け自動化を想定）。

<div id="onboarding-flow-typical">
  ## オンボーディングフロー（一般的）
</div>

1. **OpenClaw.app** をインストールして起動します。
2. 権限チェックリスト（TCC プロンプト）に一通り対応します。
3. **Local** モードが有効になっており、Gateway が稼働していることを確認します。
4. ターミナルから操作したい場合は CLI をインストールします。

<div id="build-dev-workflow-native">
  ## ビルド &amp; 開発ワークフロー（ネイティブ）
</div>

* `cd apps/macos && swift build`
* `swift run OpenClaw`（または Xcode）
* アプリのパッケージング: `scripts/package-mac-app.sh`

<div id="debug-gateway-connectivity-macos-cli">
  ## Gateway 接続性のデバッグ (macOS CLI)
</div>

デバッグ用 CLI を使用して、macOS アプリを起動することなく、macOS アプリが使用するのと同じ Gateway WebSocket ハンドシェイクおよびディスカバリー ロジックを検証します。

```bash
cd apps/macos
swift run openclaw-mac connect --json
swift run openclaw-mac discover --timeout 3000 --json
```

接続オプション:

* `--url <ws://host:port>`: 設定を上書き
* `--mode <local|remote>`: 設定から判定（デフォルト: 設定または local）
* `--probe`: 新規のヘルスチェックを強制実行
* `--timeout <ms>`: リクエストのタイムアウト（デフォルト: `15000`）
* `--json`: 差分比較用の構造化出力

ディスカバリオプション:

* `--include-local`: 「ローカル」としてフィルタされる Gateway も含める
* `--timeout <ms>`: 全体のディスカバリウィンドウ（デフォルト: `2000`）
* `--json`: 差分比較用の構造化出力

ヒント: macOS アプリのディスカバリパイプライン（NWBrowser + tailnet DNS‑SD フォールバック）が
Node CLI の `dns-sd` ベースのディスカバリと異なるかどうかを確認するには、
`openclaw gateway discover --json` の結果と比較して確認してください。

<div id="remote-connection-plumbing-ssh-tunnels">
  ## リモート接続の基盤 (SSH トンネル)
</div>

macOS アプリが **Remote** モードで動作しているときは、SSH トンネルを確立し、ローカルの UI
コンポーネントから、あたかも localhost 上にあるかのようにリモートの Gateway と通信できるようにします。

<div id="control-tunnel-gateway-websocket-port">
  ### 制御トンネル（Gateway WebSocket ポート）
</div>

* **目的:** ヘルスチェック、ステータス、Web Chat、設定、およびその他のコントロールプレーン向け呼び出し。
* **ローカルポート:** Gateway ポート（デフォルト `18789`）。常に一定。
* **リモートポート:** リモートホスト上の同じ Gateway ポート。
* **動作:** ランダムなローカルポートは使用せず、アプリは既存の正常なトンネルを再利用し、
  必要に応じて再起動する。
* **SSH 形態:** `ssh -N -L <local>:127.0.0.1:<remote>` に BatchMode +
  ExitOnForwardFailure + keepalive オプションを付与。
* **IP レポート:** SSH トンネルはループバックを使用するため、Gateway からはノードの IP は
  `127.0.0.1` として見える。Gateway に実際のクライアント IP を認識させたい場合は **Direct (ws/wss)** トランスポートを使用すること
  （[macOS remote access](/ja/platforms/mac/remote) を参照）。

セットアップ手順については [macOS remote access](/ja/platforms/mac/remote) を参照。プロトコルの詳細については
[Gateway protocol](/ja/gateway/protocol) を参照。

<div id="related-docs">
  ## 関連ドキュメント
</div>

* [Gateway ランブック](/ja/gateway)
* [Gateway（macOS）](/ja/platforms/mac/bundled-gateway)
* [macOS の権限設定](/ja/platforms/mac/permissions)
* [Canvas](/ja/platforms/mac/canvas)