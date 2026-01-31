---
title: ブラウザ
summary: "`openclaw browser` の CLI リファレンス（プロファイル、タブ、アクション、拡張機能リレー）"
read_when:
  - "`openclaw browser` を使っていて、よくあるタスクの例が欲しいとき"
  - "ノードホスト経由で、別マシン上で動作しているブラウザを制御したいとき"
  - "Chrome 拡張機能リレー（ツールバーのボタンで接続／切断）を使いたいとき"
---

<div id="openclaw-browser">
  # `openclaw browser`
</div>

OpenClaw のブラウザー制御サーバーを管理し、タブ、スナップショット、スクリーンショット、ナビゲーション、クリック、文字入力などのブラウザー操作を実行します。

関連:

* ブラウザー用ツール + API: [Browser tool](/ja/tools/browser)
* Chrome 拡張機能リレー: [Chrome extension](/ja/tools/chrome-extension)

<div id="common-flags">
  ## 共通フラグ
</div>

* `--url <gatewayWsUrl>`: Gateway の WebSocket URL（デフォルトは設定値）。
* `--token <token>`: Gateway トークン（必要な場合）。
* `--timeout <ms>`: リクエストタイムアウト（ミリ秒）。
* `--browser-profile <name>`: 使用するブラウザープロファイルを選択（デフォルトは設定値）。
* `--json`: 機械可読な出力（対応している場合）。

<div id="quick-start-local">
  ## ローカル環境向けクイックスタート
</div>

```bash
openclaw browser --browser-profile chrome tabs
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

<div id="profiles">
  ## プロファイル
</div>

プロファイルは、名前付きのブラウザルーティング構成です。実際には次のように動作します：

* `openclaw`: OpenClaw が管理する専用の Chrome インスタンス（分離されたユーザーデータディレクトリ）を起動／接続します。
* `chrome`: Chrome 拡張機能によるリレーを介して、既存の Chrome タブ（群）を制御します。

```bash
openclaw browser profiles
openclaw browser create-profile --name work --color "#FF5A36"
openclaw browser delete-profile --name work
```

特定のプロファイルを使用する：

```bash
openclaw browser --browser-profile work tabs
```

<div id="tabs">
  ## タブ
</div>

```bash
openclaw browser tabs
openclaw browser open https://docs.openclaw.ai
openclaw browser focus <targetId>
openclaw browser close <targetId>
```

<div id="snapshot-screenshot-actions">
  ## スナップショット / スクリーンショット / アクション
</div>

スナップショット:

```bash
openclaw browser snapshot
```

スクリーンショット：

```bash
openclaw browser screenshot
```

画面遷移／クリック／入力（ref ベースの UI 自動化）:

```bash
openclaw browser navigate https://example.com
openclaw browser click <ref>
openclaw browser type <ref> "hello"
```

<div id="chrome-extension-relay-attach-via-toolbar-button">
  ## Chrome 拡張リレー（ツールバーのボタンから接続）
</div>

このモードでは、手動で接続した既存の Chrome タブをエージェントに操作させることができます（自動では接続されません）。

パッケージ化されていない拡張機能を、変更されないパスにインストールします：

```bash
openclaw browser extension install
openclaw browser extension path
```

次に Chrome で `chrome://extensions` を開き、「Developer mode」を有効にし、「Load unpacked」をクリックして、表示されたフォルダを選択します。

詳細なガイド: [Chrome extension](/ja/tools/chrome-extension)

<div id="remote-browser-control-node-host-proxy">
  ## リモートブラウザー制御（ノードホストプロキシ）
</div>

Gateway がブラウザーとは別のマシン上で動作している場合は、Chrome/Brave/Edge/Chromium が動作しているマシン上で **ノードホスト** を実行してください。Gateway はブラウザー操作をそのノード経由でプロキシします（専用のブラウザー制御サーバーは不要です）。

複数のノードが接続されている場合、`gateway.nodes.browser.mode` で自動ルーティングを制御し、`gateway.nodes.browser.node` で特定のノードに固定します。

セキュリティおよびリモート接続の設定: [Browser tool](/ja/tools/browser)、[Remote access](/ja/gateway/remote)、[Tailscale](/ja/gateway/tailscale)、[Security](/ja/gateway/security)