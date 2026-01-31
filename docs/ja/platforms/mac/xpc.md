---
title: Xpc
summary: "OpenClaw アプリ、Gateway ノードトランスポート、および PeekabooBridge 向けの macOS IPC アーキテクチャ"
read_when:
  - IPC コントラクトまたはメニューバーアプリの IPC を編集する場合
---

<div id="openclaw-macos-ipc-architecture">
  # OpenClaw macOS IPC アーキテクチャ
</div>

**現在のモデル:** ローカル Unix ソケットで **ノードホストサービス** と **macOS アプリ** を接続し、実行承認（exec approvals）と `system.run` を処理する。検出/接続チェック用に `openclaw-mac` デバッグ CLI が用意されており、エージェントのアクションは引き続き Gateway の WebSocket と `node.invoke` を経由して流れる。UI 自動化には PeekabooBridge を使用する。

<div id="goals">
  ## 目標
</div>

* 通知、画面収録、マイク、音声、AppleScript など、TCC が関与する作業をすべて担う単一の GUI アプリケーションインスタンスを用意すること。
* 自動化用のインターフェースは最小限に保つこと: Gateway + ノードのコマンドに加え、UI 自動化用の PeekabooBridge のみにする。
* 予測可能な権限設定を実現すること: 常に同じ署名付きバンドル ID を用い、launchd によって起動することで、TCC の許可が安定して維持されるようにする。

<div id="how-it-works">
  ## 動作の仕組み
</div>

<div id="gateway-node-transport">
  ### Gateway + ノード間トランスポート
</div>

* アプリは Gateway（ローカルモード）を起動し、ノードとして接続します。
* エージェントのアクションは `node.invoke`（例: `system.run`、`system.notify`、`canvas.*`）を通じて実行されます。

<div id="node-service-app-ipc">
  ### ノードサービスとアプリ間のIPC
</div>

* ヘッドレスなノードホストサービスが Gateway の WebSocket に接続します。
* `system.run` リクエストはローカルの Unix ソケット経由で macOS アプリに転送されます。
* アプリは UI コンテキストでコマンドを実行し、必要に応じてプロンプトを表示して、出力を返します。

図（SCI）:

```
Agent -> Gateway -> Node Service (WS)
                      |  IPC (UDS + token + HMAC + TTL)
                      v
                  Mac App (UI + TCC + system.run)
```

<div id="peekaboobridge-ui-automation">
  ### PeekabooBridge (UI 自動化)
</div>

* UI 自動化は、`bridge.sock` という名前の別の UNIX ソケットと PeekabooBridge JSON プロトコルを使用します。
* ホストの優先順位（クライアント側）：Peekaboo.app → Claude.app → OpenClaw.app → ローカル実行。
* セキュリティ：ブリッジホストには許可された TeamID が必要です。DEBUG 時のみ有効な、同一 UID 向けの退避手段は、`PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1`（Peekaboo の規約）によって保護されています。
* 詳細は [PeekabooBridge usage](/ja/platforms/mac/peekaboo) を参照してください。

<div id="operational-flows">
  ## 運用フロー
</div>

* 再起動/再ビルド: `SIGN_IDENTITY="Apple Development: <Developer Name> (<TEAMID>)" scripts/restart-mac.sh`
  * 既存のインスタンスを強制終了する
  * Swift をビルドしてパッケージングする
  * LaunchAgent を書き込み/ブートストラップ/起動する
* シングルインスタンス: 同じバンドル ID の別インスタンスが実行中の場合、アプリは即座に終了する。

<div id="hardening-notes">
  ## ハードニングに関する注意事項
</div>

* すべての特権的な領域に対して、可能な限り TeamID の一致を必須とする。
* PeekabooBridge: `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1`（DEBUG 専用）は、ローカル開発向けに同一 UID の呼び出し元を許可できる。
* すべての通信はローカル限定のままであり、ネットワークソケットは外部に公開しない。
* TCC プロンプトは GUI アプリバンドルからのみ発生するため、再ビルドをまたいでも署名済みバンドル ID が変わらないよう維持する。
* IPC のハードニング: ソケットモード `0600`、トークン、ピア UID のチェック、HMAC チャレンジ／レスポンス、短い TTL を使用する。