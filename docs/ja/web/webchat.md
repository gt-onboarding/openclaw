---
title: WebChat
summary: "ループバック WebChat の静的ホスティングと、チャット UI 向け Gateway WS の利用方法"
read_when:
  - WebChat へのアクセスをデバッグまたは構成するとき
---

<div id="webchat-gateway-websocket-ui">
  # WebChat (Gateway WebSocket UI)
</div>

現状: macOS/iOSのSwiftUI製チャットUIはGatewayのWebSocketに直接接続します。

<div id="what-it-is">
  ## 概要
</div>

* Gateway 用のネイティブチャット UI（埋め込みブラウザやローカルの静的サーバーは不要）。
* 他のチャネルと同じセッションとルーティングルールを使用します。
* 決定的なルーティング: 返信は必ず WebChat に戻ります。

<div id="quick-start">
  ## クイックスタート
</div>

1. Gateway を起動します。
2. WebChat UI（macOS/iOS アプリ）または Control UI のチャットタブを開きます。
3. Gateway の認証が設定されていることを確認します（ループバック接続であっても、デフォルトで必須です）。

<div id="how-it-works-behavior">
  ## 動作の仕組み（挙動）
</div>

* UI は Gateway の WebSocket に接続し、`chat.history`、`chat.send`、`chat.inject` を使用します。
* `chat.inject` はアシスタントのメモを会話ログに直接追加し、それを UI にブロードキャストします（エージェントの実行は発生しません）。
* 履歴は常に Gateway から取得されます（ローカルファイルの監視は行いません）。
* Gateway に接続できない場合、WebChat は閲覧のみ可能な読み取り専用モードになります。

<div id="remote-use">
  ## リモート利用
</div>

* リモートモードでは、Gateway の WebSocket を SSH/Tailscale 経由でトンネルします。
* 別途 WebChat サーバーを起動する必要はありません。

<div id="configuration-reference-webchat">
  ## 設定リファレンス（WebChat）
</div>

設定の全体像: [Configuration](/ja/gateway/configuration)

チャネルオプション:

* 専用の `webchat.*` ブロックはありません。WebChat は、以下の Gateway エンドポイントおよび認証設定を使用します。

関連するグローバルオプション:

* `gateway.port`, `gateway.bind`: WebSocket のホスト／ポート。
* `gateway.auth.mode`, `gateway.auth.token`, `gateway.auth.password`: WebSocket 認証。
* `gateway.remote.url`, `gateway.remote.token`, `gateway.remote.password`: リモート Gateway の接続先。
* `session.*`: セッションストレージとメインキーのデフォルト値。