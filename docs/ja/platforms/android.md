---
title: Android
summary: "Android アプリ（ノード）：接続ランブック + Canvas/Chat/Camera"
read_when:
  - Android ノードのペアリングまたは再接続を行うとき
  - Android アプリからの Gateway 検出または認証をデバッグするとき
  - クライアント間でのチャット履歴の整合性を確認するとき
---

<div id="android-app-node">
  # Android アプリ（ノード）
</div>

<div id="support-snapshot">
  ## サポートスナップショット
</div>

* 役割: コンパニオンノードアプリ（Android 自体は Gateway をホストしません）。
* Gateway 必須: はい（macOS、Linux、または Windows（WSL2 経由）で実行してください）。
* インストール: [はじめに](/ja/start/getting-started) + [ペアリング](/ja/gateway/pairing)。
* Gateway: [ランブック](/ja/gateway) + [設定](/ja/gateway/configuration)。
  * プロトコル: [Gateway プロトコル](/ja/gateway/protocol)（ノード + コントロールプレーン）。

<div id="system-control">
  ## システム制御
</div>

システム制御（launchd/systemd）は Gateway ホスト上で動作します。[Gateway](/ja/gateway) を参照してください。

<div id="connection-runbook">
  ## 接続ランブック
</div>

Android ノードアプリ ⇄ (mDNS/NSD + WebSocket) ⇄ **Gateway**

Android は Gateway の WebSocket（デフォルトは `ws://<host>:18789`）に直接接続し、Gateway 管理のペアリングを使用します。

<div id="prerequisites">
  ### 前提条件
</div>

* 「マスター」マシン上で Gateway を実行できること。
* Android デバイス／エミュレータから Gateway の WebSocket に接続できること:
  * mDNS/NSD を利用した同一 LAN 上、**または**
  * Wide-Area Bonjour / ユニキャスト DNS-SD（下記参照）を利用した同一 Tailscale tailnet 上、**または**
  * 手動指定の Gateway ホスト／ポート（フォールバック）
* Gateway マシン上で（または SSH 経由で）CLI（`openclaw`）を実行できること。

<div id="1-start-the-gateway">
  ### 1) Gateway を起動する
</div>

```bash
openclaw gateway --port 18789 --verbose
```

ログに次のような行が出力されていることを確認してください:

* `listening on ws://0.0.0.0:18789`

tailnet 専用の構成（Vienna ⇄ London では推奨）では、Gateway を tailnet の IP にバインドします:

* Gateway ホスト上の `~/.openclaw/openclaw.json` で `gateway.bind: "tailnet"` を設定します。
* Gateway / macOS メニューバーアプリを再起動します。

<div id="2-verify-discovery-optional">
  ### 2) ディスカバリを確認する（任意）
</div>

Gateway マシン上で実行します:

```bash
dns-sd -B _openclaw-gw._tcp local.
```

詳しいデバッグメモは [Bonjour](/ja/gateway/bonjour) を参照してください。

<div id="tailnet-vienna-london-discovery-via-unicast-dns-sd">
  #### Tailnet（ウィーン ⇄ ロンドン）間でのユニキャスト DNS-SD を使ったディスカバリ
</div>

Android の NSD/mDNS ディスカバリはネットワークをまたいでは動作しません。Android ノードと Gateway が異なるネットワーク上にあり、Tailscale で接続されている場合は、代わりに Wide-Area Bonjour／ユニキャスト DNS-SD を使用してください:

1. Gateway ホスト上で DNS-SD ゾーン（例: `openclaw.internal.`）を設定し、`_openclaw-gw._tcp` レコードを公開します。
2. 選択したドメインについて、その DNS サーバーを参照するように Tailscale の split DNS を設定します。

詳細および CoreDNS 設定例については、[Bonjour](/ja/gateway/bonjour) を参照してください。

<div id="3-connect-from-android">
  ### 3) Android から接続する
</div>

Android アプリでは、次を実行します:

* アプリは **フォアグラウンドサービス**（常駐通知）を使って Gateway への接続を維持します。
* **Settings** を開きます。
* **Discovered Gateways** の下から自分の Gateway を選択し、**Connect** をタップします。
* mDNS がブロックされている場合は、**Advanced → Manual Gateway**（ホスト + ポート）を使い、**Connect (Manual)** を実行します。

最初のペアリングに成功すると、Android は起動時に自動で再接続します:

* 手動エンドポイント（有効な場合）。それ以外の場合は、
* 最後に検出された Gateway（ベストエフォート）。

<div id="4-approve-pairing-cli">
  ### 4) ペアリングを承認する (CLI)
</div>

Gateway を実行しているマシン上で:

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

ペアリングの詳細については: [Gateway ペアリング](/ja/gateway/pairing) を参照してください。

<div id="5-verify-the-node-is-connected">
  ### 5) ノードが接続されていることを確認する
</div>

* ノードのステータスで確認:
  ```bash
  openclaw nodes status
  ```
* Gateway 経由で確認:
  ```bash
  openclaw gateway call node.list --params "{}"
  ```

<div id="6-chat-history">
  ### 6) チャット + 履歴
</div>

Android ノードの Chat シートは Gateway の**プライマリセッションキー**（`main`）を使用するため、履歴と返信は WebChat や他のクライアントと共有されます:

* 履歴: `chat.history`
* 送信: `chat.send`
* プッシュ型の更新（ベストエフォート）: `chat.subscribe` → `event:"chat"`

<div id="7-canvas-camera">
  ### 7) Canvas + カメラ
</div>

<div id="gateway-canvas-host-recommended-for-web-content">
  #### Gateway Canvas ホスト（Web コンテンツ向け推奨）
</div>

エージェントがディスク上で編集できる実際の HTML/CSS/JS をノードに表示させたい場合、ノードの表示先として Gateway の canvas ホストを指定します。

注意: ノードはスタンドアロンの canvas ホストを `canvasHost.port`（デフォルト `18793`）で使用します。

1. Gateway ホスト上に `~/.openclaw/workspace/canvas/index.html` を作成します。

2. ノードからその場所（URL）へアクセスさせます（LAN 経由）:

```bash
openclaw nodes invoke --node "<Android Node>" --command canvas.navigate --params '{"url":"http://<gateway-hostname>.local:18793/__openclaw__/canvas/"}'
```

Tailnet（任意）：両方のデバイスが Tailscale に接続されている場合は、`.local` の代わりに MagicDNS 名または tailnet IP を使用します（例：`http://<gateway-magicdns>:18793/__openclaw__/canvas/`）。

このサーバーは HTML にライブリロード用クライアントを注入し、ファイル変更時にリロードします。
A2UI ホストは `http://<gateway-host>:18793/__openclaw__/a2ui/` にあります。

Canvas コマンド（フォアグラウンドのみ）:

* `canvas.eval`、`canvas.snapshot`、`canvas.navigate`（デフォルトのスキャフォールドに戻るには `{"url":""}` または `{"url":"/"}` を使用）。`canvas.snapshot` は `{ format, base64 }` を返します（デフォルトの `format="jpeg"`）。
* A2UI: `canvas.a2ui.push`、`canvas.a2ui.reset`（`canvas.a2ui.pushJSONL` はレガシーエイリアス）

Camera コマンド（フォアグラウンドのみ・権限による制御あり）:

* `camera.snap`（jpg）
* `camera.clip`（mp4）

パラメータと CLI ヘルパーについては [Camera ノード](/ja/nodes/camera) を参照してください。
