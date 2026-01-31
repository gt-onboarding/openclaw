---
title: ノード
summary: "ノード: ペアリング、機能、権限、および canvas/camera/screen/system 用 CLI ヘルパー"
read_when:
  - iOS/Android ノードを Gateway にペアリングするとき
  - node canvas/camera をエージェントのコンテキスト情報として使用するとき
  - 新しい node コマンドや CLI ヘルパーを追加するとき
---

<div id="nodes">
  # ノード
</div>

**ノード** は Gateway の **WebSocket**（オペレーターと同じポート）に `role: "node"` として接続し、`node.invoke` を通じてコマンドインターフェイス（例: `canvas.*`, `camera.*`, `system.*`）を公開するコンパニオンデバイス（macOS/iOS/Android/ヘッドレス）です。プロトコルの詳細: [Gateway protocol](/ja/gateway/protocol)。

レガシーなトランスポート: [Bridge protocol](/ja/gateway/bridge-protocol)（TCP JSONL。現行のノードでは非推奨／削除済みです）。

macOS は **ノードモード** でも動作できます。メニューバーアプリが Gateway の WS サーバーに接続し、そのローカルの canvas/camera コマンドをノードとして公開します（そのため `openclaw nodes …` をこの Mac に対して実行できます）。

補足:

* ノードは **周辺機器** であり、Gateway ではありません。Gateway サービス自体は実行しません。
* Telegram/WhatsApp などのメッセージはノードではなく **Gateway** に届きます。

<div id="pairing-status">
  ## ペアリングとステータス
</div>

**WS ノードはデバイス ペアリングを使用します。** ノードは `connect` 時にデバイス ID を提示し、Gateway は
`role: node` 用のデバイス ペアリング要求を作成します。devices CLI（または UI）で承認します。

CLI のクイック例:

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
```

Notes:

* `nodes status` は、デバイスのペアリングロールに `node` が含まれている場合、そのノードを **paired** としてマークします。
* `node.pair.*`（CLI: `openclaw nodes pending/approve/reject`）は Gateway が管理する独立したノード用ペアリングストアであり、WS の `connect` ハンドシェイクの可否を制御するものでは **ありません**。

<div id="remote-node-host-systemrun">
  ## リモートノードホスト (system.run)
</div>

Gateway をあるマシン上で実行し、別のマシン上でコマンドを実行したい場合は、**ノードホスト** を使用します。モデルは引き続き **Gateway** と通信し、`host=node` が選択されている場合、Gateway が `exec` 呼び出しを **ノードホスト** に転送します。

<div id="what-runs-where">
  ### どこで何が動作するか
</div>

* **Gateway ホスト**: メッセージを受信し、モデルを実行し、ツール呼び出しをルーティングします。
* **ノード ホスト**: ノードマシン上で `system.run` / `system.which` を実行します。
* **承認**: ノード ホスト上で `~/.openclaw/exec-approvals.json` に基づいて適用されます。

<div id="start-a-node-host-foreground">
  ### フォアグラウンドでノードホストを起動する
</div>

ノード側のマシンで：

```bash
openclaw node run --host <gateway-host> --port 18789 --display-name "Build Node"
```

<div id="start-a-node-host-service">
  ### ノードホスト（サービス）を開始する
</div>

```bash
openclaw node install --host <gateway-host> --port 18789 --display-name "Build Node"
openclaw node restart
```

<div id="pair-name">
  ### ペアリング + 名前付け
</div>

Gateway ホスト上で実行:

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes list
```

名前の指定方法:

* ノード側で `openclaw node run` / `openclaw node install` に `--display-name` を指定（ノード上の `~/.openclaw/node.json` に永続化されます）。
* `openclaw nodes rename --node <id|name|ip> --name "Build Node"`（Gateway 側での上書き設定）。

<div id="allowlist-the-commands">
  ### コマンドを許可リストに登録する
</div>

Exec の承認は**各ノードホスト単位**です。Gateway から許可リストにエントリを追加します。

```bash
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/uname"
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/sw_vers"
```

承認情報はノードホスト上の `~/.openclaw/exec-approvals.json` に格納されています。

<div id="point-exec-at-the-node">
  ### exec の宛先をノードに設定する
</div>

デフォルト値を設定（Gateway の設定）:

```bash
openclaw config set tools.exec.host node
openclaw config set tools.exec.security allowlist
openclaw config set tools.exec.node "<id-or-name>"
```

あるいはセッション単位で:

```
/exec host=node security=allowlist node=<id-or-name>
```

一度設定すると、`host=node` を指定したすべての `exec` 呼び出しは、ノードの許可リスト／承認に従ってノードホスト上で実行されます。

関連:

* [ノードホスト CLI](/ja/cli/node)
* [Exec ツール](/ja/tools/exec)
* [Exec 承認](/ja/tools/exec-approvals)

<div id="invoking-commands">
  ## コマンドの実行
</div>

低レベル（生のRPC）:

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command canvas.eval --params '{"javaScript":"location.href"}'
```

一般的な「エージェントに MEDIA 添付を渡す」ワークフロー向けに、より高レベルなヘルパーも用意されています。

<div id="screenshots-canvas-snapshots">
  ## スクリーンショット（キャンバススナップショット）
</div>

ノードが Canvas（WebView）を表示している場合、`canvas.snapshot` は `{ format, base64 }` を返します。

CLI ヘルパー（一時ファイルに書き込み、`MEDIA:<path>` を出力）:

```bash
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format png
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format jpg --max-width 1200 --quality 0.9
```

<div id="canvas-controls">
  ### キャンバス操作
</div>

```bash
openclaw nodes canvas present --node <idOrNameOrIp> --target https://example.com
openclaw nodes canvas hide --node <idOrNameOrIp>
openclaw nodes canvas navigate https://example.com --node <idOrNameOrIp>
openclaw nodes canvas eval --node <idOrNameOrIp> --js "document.title"
```

メモ:

* `canvas present` は URL またはローカルファイルパス（`--target`）を受け取り、位置指定用のオプション `--x/--y/--width/--height` も利用できます。
* `canvas eval` はインライン JS（`--js`）または位置引数を受け取ります。

<div id="a2ui-canvas">
  ### A2UI (Canvas)
</div>

```bash
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --text "Hello"
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --jsonl ./payload.jsonl
openclaw nodes canvas a2ui reset --node <idOrNameOrIp>
```

Notes:

* A2UI v0.8 JSONL のみ対応しています（v0.9/createSurface は受け付けません）。

<div id="photos-videos-node-camera">
  ## 写真と動画（ノードのカメラ）
</div>

写真（`jpg`）:

```bash
openclaw nodes camera list --node <idOrNameOrIp>
openclaw nodes camera snap --node <idOrNameOrIp>            # デフォルト: 両面 (2つのMEDIA行)
openclaw nodes camera snap --node <idOrNameOrIp> --facing front
```

動画クリップ（`mp4`）:

```bash
openclaw nodes camera clip --node <idOrNameOrIp> --duration 10s
openclaw nodes camera clip --node <idOrNameOrIp> --duration 3000 --no-audio
```

Notes:

* `canvas.*` および `camera.*` を使用するには、ノードが**前面に表示されている状態**である必要があります（バックグラウンドからの呼び出しは `NODE_BACKGROUND_UNAVAILABLE` を返します）。
* クリップの長さには上限があります（現在は `<= 60s`）。これは base64 ペイロードが大きくなりすぎるのを防ぐためです。
* Android では、可能な場合に `CAMERA` / `RECORD_AUDIO` パーミッションを要求します。パーミッションが拒否された場合は、`*_PERMISSION_REQUIRED` として失敗します。

<div id="screen-recordings-nodes">
  ## 画面録画（ノード）
</div>

ノードは `screen.record`（mp4）機能を提供します。例：

```bash
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10 --no-audio
```

Notes:

* `screen.record` を使用する場合は、ノードアプリがフォアグラウンドにある必要があります。
* Android では、録画前にシステムの画面キャプチャプロンプトが表示されます。
* 画面録画は `<= 60s` に制限されます。
* `--no-audio` はマイク音声のキャプチャを無効にします（iOS/Android でサポートされています。macOS はシステムのオーディオキャプチャを使用します）。
* 複数の画面がある場合は、`--screen <index>` を使用してディスプレイを選択します。

<div id="location-nodes">
  ## 位置情報（ノード）
</div>

設定で「Location」が有効になっている場合、ノードは `location.get` を提供します。

CLI ヘルパー:

```bash
openclaw nodes location get --node <idOrNameOrIp>
openclaw nodes location get --node <idOrNameOrIp> --accuracy precise --max-age 15000 --location-timeout 10000
```

Notes:

* 位置情報は**デフォルトでオフ**です。
* 「常に」を選択するにはシステム権限が必要です。バックグラウンドでの取得はベストエフォートで行われます。
* レスポンスには緯度/経度、精度（メートル）、タイムスタンプが含まれます。

<div id="sms-android-nodes">
  ## SMS（Android ノード）
</div>

Android ノードは、ユーザーが **SMS** への権限を付与し、かつデバイスが通話機能をサポートしている場合に `sms.send` を公開できます。

低レベルの呼び出し:

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command sms.send --params '{"to":"+15555550123","message":"Hello from OpenClaw"}'
```

Notes:

* 機能がアドバタイズされる前に、Android デバイス上の権限プロンプトを許可する必要があります。
* 音声通話機能のない Wi‑Fi 専用デバイスは `sms.send` をアドバタイズしません。

<div id="system-commands-node-host-mac-node">
  ## システムコマンド（ノードホスト / Mac ノード）
</div>

macOS ノードは `system.run`、`system.notify`、`system.execApprovals.get/set` を提供します。
ヘッドレスのノードホストは `system.run`、`system.which`、`system.execApprovals.get/set` を提供します。

例：

```bash
openclaw nodes run --node <idOrNameOrIp> -- echo "Hello from mac node"
openclaw nodes notify --node <idOrNameOrIp> --title "Ping" --body "Gateway ready"
```

Notes:

* `system.run` は、stdout/stderr/終了コードを含むペイロードを返します。
* `system.notify` は、macOS アプリ上の通知の許可状態に従います。
* `system.run` は `--cwd`、`--env KEY=VAL`、`--command-timeout`、`--needs-screen-recording` をサポートします。
* `system.notify` は `--priority <passive|active|timeSensitive>` および `--delivery <system|overlay|auto>` をサポートします。
* macOS ノードは `PATH` の上書きを無視します。ヘッドレス ノードホストは、ノードホストの PATH を前置する場合にのみ `PATH` を受け入れます。
* macOS ノードモードでは、`system.run` は macOS アプリの実行承認設定（Settings → Exec approvals）によって制御されます。
  Ask/allowlist/full はヘッドレス ノードホストと同様に動作し、拒否されたプロンプトは `SYSTEM_RUN_DENIED` を返します。
* ヘッドレス ノードホストでは、`system.run` は実行承認設定（`~/.openclaw/exec-approvals.json`）によって制御されます。

<div id="exec-node-binding">
  ## exec ノードのバインド
</div>

複数のノードが利用可能な場合、`exec` を特定のノードにバインドできます。
これにより、`exec host=node` におけるデフォルトのノードが設定されます（エージェント単位で上書き可能です）。

グローバルデフォルト:

```bash
openclaw config set tools.exec.node "node-id-or-name"
```

エージェントごとのオーバーライド：

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

未設定の場合、任意のノードを許可します:

```bash
openclaw config unset tools.exec.node
openclaw config unset agents.list[0].tools.exec.node
```

<div id="permissions-map">
  ## 権限マップ
</div>

ノードは `node.list` / `node.describe` 内に、権限名（例: `screenRecording`, `accessibility`）をキーとし、値に真偽値（`true` = 付与済み）を取る `permissions` マップを含めることができます。

<div id="headless-node-host-cross-platform">
  ## ヘッドレスノードホスト（クロスプラットフォーム）
</div>

OpenClaw は、Gateway の WebSocket に接続し、`system.run` / `system.which` を公開する
UI なしの **ヘッドレスノードホスト** を実行できます。これは Linux/Windows 上や、
サーバーと同じマシン上で最小限構成のノードを動かす場合に有用です。

起動するには:

```bash
openclaw node run --host <gateway-host> --port 18789
```

Notes:

* ペアリングは引き続き必要です（Gateway 側にノード承認プロンプトが表示されます）。
* ノードホストは、自身の node id、token、display name、および Gateway への接続情報を `~/.openclaw/node.json` に保存します。
* Exec approvals は `~/.openclaw/exec-approvals.json` の内容に基づいてローカルで適用されます
  （[Exec approvals](/ja/tools/exec-approvals) を参照）。
* macOS では、ヘッドレスノードホストは到達可能な場合はコンパニオンアプリの exec ホストを優先し、
  アプリが利用できない場合はローカル実行にフォールバックします。アプリの利用を必須にするには `OPENCLAW_NODE_EXEC_HOST=app` を、フォールバックを無効化するには `OPENCLAW_NODE_EXEC_FALLBACK=0` を設定します。
* Gateway の WS が TLS を使用している場合は `--tls` / `--tls-fingerprint` を追加します。

<div id="mac-node-mode">
  ## Mac ノードモード
</div>

* macOS メニューバーアプリはノードとして Gateway の WS サーバーに接続します（これにより、この Mac を対象に `openclaw nodes …` コマンドを実行できます）。
* リモートモードでは、アプリは Gateway ポート向けに SSH トンネルを開き、`localhost` に接続します。