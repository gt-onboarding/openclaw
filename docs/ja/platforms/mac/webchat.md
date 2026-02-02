---
title: Webchat
summary: "macOS アプリが Gateway の WebChat をどのように埋め込み、どのようにデバッグするか"
read_when:
  - macOS の WebChat ビューまたはループバックポートをデバッグするとき
---

<div id="webchat-macos-app">
  # WebChat (macOS app)
</div>

macOS メニューバーアプリは、WebChat UI をネイティブな SwiftUI ビューとして組み込みます。
このアプリは Gateway に接続し、選択したエージェントの **メインセッション** をデフォルトとして使用します
（他のセッションにはセッションスイッチャーで切り替えられます）。

- **ローカルモード**：ローカルの Gateway WebSocket に直接接続します。
- **リモートモード**：Gateway の制御ポートを SSH 経由で転送し、そのトンネルをデータプレーンとして使用します。

<div id="launch-debugging">
  ## 起動とデバッグ
</div>

- 手動起動：Lobster メニュー → 「Open Chat」
- テスト用に自動起動する場合：
  ```bash
  dist/OpenClaw.app/Contents/MacOS/OpenClaw --webchat
  ```
- ログ：`./scripts/clawlog.sh`（サブシステム `bot.molt`、カテゴリ `WebChatSwiftUI`）

<div id="how-its-wired">
  ## 接続構成
</div>

- データプレーン: Gateway の WS メソッド `chat.history`、`chat.send`、`chat.abort`、
  `chat.inject` と、イベント `chat`、`agent`、`presence`、`tick`、`health`。
- セッション: 既定ではプライマリセッション（通常は `main`、スコープが global の場合は `global`）。UI からセッションを切り替えることができる。
- オンボーディングでは、初回セットアップを分離するために専用セッションを使用する。

<div id="security-surface">
  ## セキュリティ上の攻撃対象領域
</div>

- リモートモードでは、Gateway の WebSocket 制御ポートのみを SSH 経由で転送します。

<div id="known-limitations">
  ## 既知の制限事項
</div>

- UI はチャットセッションに最適化されており、完全なブラウザーサンドボックスにはなっていません。