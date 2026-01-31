---
title: iOS
summary: "iOS ノードアプリ: Gateway への接続、ペアリング、キャンバス機能、トラブルシューティング"
read_when:
  - iOS ノードのペアリングや再接続を行うとき
  - iOS アプリをソースから実行するとき
  - Gateway の検出やキャンバスコマンドをデバッグするとき
---

<div id="ios-app-node">
  # iOS App (Node)
</div>

提供状況: 内部プレビュー版。iOS アプリはまだ一般公開されていません。

<div id="what-it-does">
  ## 機能
</div>

* WebSocket（LAN または tailnet）経由で Gateway に接続します。
* ノードとしての機能を提供します: Canvas、画面スナップショット、カメラキャプチャ、位置情報、トークモード、音声ウェイク。
* `node.invoke` コマンドを受信し、ノードのステータスイベントを送信します。

<div id="requirements">
  ## 必要条件
</div>

* 別のデバイス上で動作している Gateway（macOS、Linux、または WSL2 経由の Windows）。
* ネットワーク経路:
  * Bonjour を利用した同一 LAN 上、**または**
  * unicast DNS-SD を利用した Tailnet（ドメイン名の例: `openclaw.internal.`）、**または**
  * 手動指定のホスト／ポート（フォールバック用）。

<div id="quick-start-pair-connect">
  ## クイックスタート（ペアリングと接続）
</div>

1. Gateway を起動します。

```bash
openclaw gateway --port 18789
```

2. iOS アプリで Settings を開いて、検出された Gateway を選択する（または Manual Host を有効にして host/port を入力する）。

3. Gateway ホストでペアリングリクエストを承認する。

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

4. 接続を確認する:

```bash
openclaw nodes status
openclaw gateway call node.list --params "{}"
```

<div id="discovery-paths">
  ## 検出方法
</div>

<div id="bonjour-lan">
  ### Bonjour (LAN)
</div>

Gateway は `local.` 上で `_openclaw-gw._tcp` をアドバタイズします。iOS アプリはこれらを自動検出して一覧表示します。

<div id="tailnet-cross-network">
  ### Tailnet（クロスネットワーク）
</div>

mDNS がブロックされている場合は、ユニキャスト DNS-SD ゾーン（ドメインを 1 つ選択します。例: `openclaw.internal.`）と Tailscale の Split DNS 機能を使用してください。
CoreDNS の例については [Bonjour](/ja/gateway/bonjour) を参照してください。

<div id="manual-hostport">
  ### 手動ホスト／ポート
</div>

Settings で **Manual Host** を有効にし、Gateway のホストとポート（デフォルトは `18789`）を入力します。

<div id="canvas-a2ui">
  ## Canvas + A2UI
</div>

iOS ノードは WKWebView キャンバスをレンダリングします。`node.invoke` を使ってキャンバスを制御します。

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.navigate --params '{"url":"http://<gateway-host>:18793/__openclaw__/canvas/"}'
```

Notes:

* Gateway の canvas ホストは `/__openclaw__/canvas/` と `/__openclaw__/a2ui/` を提供します。
* iOS ノードは、canvas ホストの URL がアドバタイズされている場合、接続時に自動的に A2UI へ遷移します。
* `canvas.navigate` と `{"url":""}` を使うと、組み込みのスキャフォールドに戻せます。

<div id="canvas-eval-snapshot">
  ### Canvas の評価 / スナップショット
</div>

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.eval --params '{"javaScript":"(() => { const {ctx} = window.__openclaw; ctx.clearRect(0,0,innerWidth,innerHeight); ctx.lineWidth=6; ctx.strokeStyle=\"#ff2d55\"; ctx.beginPath(); ctx.moveTo(40,40); ctx.lineTo(innerWidth-40, innerHeight-40); ctx.stroke(); return \"ok\"; })()"}'
```

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.snapshot --params '{"maxWidth":900,"format":"jpeg"}'
```

<div id="voice-wake-talk-mode">
  ## 音声ウェイク + トークモード
</div>

* 音声ウェイクとトークモードは、設定画面から利用できます。
* iOS がバックグラウンドのオーディオを一時停止する場合があります。アプリがアクティブでないときの音声機能は、ベストエフォートとして扱ってください。

<div id="common-errors">
  ## よくあるエラー
</div>

* `NODE_BACKGROUND_UNAVAILABLE`: iOS アプリを前面に表示してください（canvas/camera/screen コマンドには前面表示が必要です）。
* `A2UI_HOST_NOT_CONFIGURED`: Gateway が canvas ホスト URL をアドバタイズしていません。[Gateway configuration](/ja/gateway/configuration) の `canvasHost` を確認してください。
* ペアリングのプロンプトが表示されない: `openclaw nodes pending` を実行し、手動で承認してください。
* 再インストール後に再接続できない: キーチェーンのペアリングトークンが消去されています。ノードを再ペアリングしてください。

<div id="related-docs">
  ## 関連ドキュメント
</div>

* [ペアリング](/ja/gateway/pairing)
* [ディスカバリ](/ja/gateway/discovery)
* [Bonjour](/ja/gateway/bonjour)