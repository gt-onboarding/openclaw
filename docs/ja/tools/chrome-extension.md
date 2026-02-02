---
title: Chrome拡張機能
summary: "Chrome拡張機能: 既存のChromeタブの操作をOpenClawに任せる"
read_when:
  - エージェントに既存のChromeタブを操作させたい（ツールバーボタン）
  - リモートのGatewayと、Tailscale経由でのローカルブラウザ自動化が必要
  - ブラウザ制御の委譲によるセキュリティへの影響を理解したい
---

<div id="chrome-extension-browser-relay">
  # Chrome extension (browser relay)
</div>

OpenClaw の Chrome 拡張機能を使うと、別の openclaw が管理する Chrome プロファイルを起動する代わりに、エージェントがあなたの **既存の Chrome タブ**（通常の Chrome ウィンドウ）を制御できるようになります。

接続と切断は、**1つの Chrome ツールバー ボタン**で行えます。

<div id="what-it-is-concept">
  ## これは何か（コンセプト）
</div>

次の3つの要素から構成されます:

* **ブラウザ制御サービス**（Gateway または ノード）：エージェント/ツールが（Gateway 経由で）呼び出す API
* **ローカルリレーサーバー**（ループバック CDP）：制御サーバーと拡張機能の間をブリッジするコンポーネント（デフォルトは `http://127.0.0.1:18792`）
* **Chrome MV3 拡張機能**：`chrome.debugger` を使ってアクティブなタブに接続し、CDP メッセージをリレーに転送する

これにより OpenClaw は、通常どおり `browser` ツールのインターフェース（適切なプロファイルを選択）を通じて、接続されたタブを制御できます。

<div id="install-load-unpacked">
  ## インストール / 読み込み（パッケージ化されていない拡張機能）
</div>

1. 拡張機能を固定されたローカルパスにインストールします：

```bash
openclaw browser extension install
```

2. インストール済みの拡張機能のディレクトリパスを表示する:

```bash
openclaw browser extension path
```

3. Chrome → `chrome://extensions`

* 「デベロッパー モード」を有効にする
* 「パッケージ化されていない拡張機能を読み込む」 → 上で出力されたディレクトリを選択する

4. 拡張機能をピン留めする

<div id="updates-no-build-step">
  ## アップデート（ビルド手順なし）
</div>

この拡張機能は、OpenClaw リリース（npm パッケージ）に静的ファイルとして同梱されています。別途「ビルド」手順は不要です。

OpenClaw をアップグレードした後は、次を実行してください:

* `openclaw browser extension install` を再実行し、OpenClaw の状態ディレクトリ配下にインストールされているファイルを更新します。
* Chrome → `chrome://extensions` → 該当拡張機能の「再読み込み」をクリックします。

<div id="use-it-no-extra-config">
  ## 使い方（追加の設定不要）
</div>

OpenClaw には、デフォルトポート上の拡張機能リレーを対象にした、`chrome` という名前の組み込みブラウザプロファイルが含まれています。

使用方法:

* CLI: `openclaw browser --browser-profile chrome tabs`
* エージェントツール: `browser` に `profile="chrome"` を指定

別の名前や別のリレーポートを使いたい場合は、独自のプロファイルを作成してください:

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

<div id="attach-detach-toolbar-button">
  ## アタッチ / デタッチ（ツールバーボタン）
</div>

* OpenClaw に制御させたいタブを開きます。
* 拡張機能アイコンをクリックします。
  * アタッチされているときは、バッジに `ON` と表示されます。
* もう一度クリックするとデタッチされます。

<div id="which-tab-does-it-control">
  ## どのタブが対象になりますか？
</div>

* 「今表示しているタブ」を**自動的に**制御することはありません。
* ツールバーのボタンをクリックして**明示的にアタッチしたタブだけ**が制御対象になります。
* 切り替えるには、切り替え先のタブを開き、そのタブで拡張機能アイコンをクリックします。

<div id="badge-common-errors">
  ## バッジとよくあるエラー
</div>

* `ON`: 接続済み。OpenClaw がそのタブを制御できます。
* `…`: ローカルリレーに接続中。
* `!`: リレーに接続できません（よくある原因: このマシンでブラウザリレーサーバーが動作していない）。

`!` が表示された場合:

* Gateway がローカルで動作していることを確認してください（デフォルト構成）。Gateway が別の場所で動いている場合は、このマシン上で node ホストを実行してください。
* 拡張機能の Options ページを開いてください。リレーに接続可能かどうかが表示されます。

<div id="remote-gateway-use-a-node-host">
  ## リモート Gateway（ノードホスト経由）
</div>

<div id="local-gateway-same-machine-as-chrome-usually-no-extra-steps">
  ### ローカル Gateway（Chrome と同一マシン）— 通常は**追加手順は不要**
</div>

Gateway が Chrome と同じマシン上で動作している場合、ループバックインターフェース上でブラウザー制御サービスを起動し、
リレーサーバーを自動的に起動します。拡張機能はローカルのリレーサーバーと通信し、CLI/ツールからの呼び出しは Gateway に送られます。

<div id="remote-gateway-gateway-runs-elsewhere-run-a-node-host">
  ### Remote Gateway (Gateway runs elsewhere) — **ノードホストを実行する**
</div>

Gateway が別のマシンで動作している場合は、Chrome を実行しているマシン上でノードホストを起動します。
Gateway はブラウザでの操作をそのノードにプロキシし、拡張機能とリレーはブラウザを実行しているマシン上にとどまります。

複数のノードが接続されている場合は、`gateway.nodes.browser.node` を使って特定のノードを固定するか、`gateway.nodes.browser.mode` を設定します。

<div id="sandboxing-tool-containers">
  ## サンドボックス化（ツールコンテナ）
</div>

エージェントセッションがサンドボックス化されている場合（`agents.defaults.sandbox.mode != "off"`）、`browser` ツールは制限される可能性があります:

* 既定では、サンドボックス化されたセッションはホストの Chrome ではなく、**サンドボックスブラウザー**（`target="sandbox"`）をターゲットにすることが多いです。
* Chrome extension relay をテイクオーバーするには、**ホスト**側のブラウザー制御サーバーを制御する必要があります。

オプション:

* 最も簡単なのは、**非サンドボックス**のセッション／エージェントから拡張機能を使用することです。
* もしくは、サンドボックス化されたセッションに対してホストブラウザー制御を許可します:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: {
          allowHostControl: true
        }
      }
    }
  }
}
```

その後、ツールがツールポリシーによって拒否されていないことを確認し、必要に応じて `target="host"` を指定して `browser` を呼び出してください。

デバッグ用: `openclaw sandbox explain`

<div id="remote-access-tips">
  ## リモートアクセスのヒント
</div>

* Gateway とノードホストは同じ tailnet 上に配置し、リレーポートを LAN やパブリックインターネットに公開しないようにしてください。
* ノードは意図してペアリングを行い、リモート制御を行いたくない場合はブラウザプロキシ経由のルーティングを無効化してください（`gateway.nodes.browser.mode="off"`）。

<div id="how-extension-path-works">
  ## 「extension path」の動作
</div>

`openclaw browser extension path` は、拡張機能ファイルが含まれている **インストール済み** のディスク上のディレクトリを出力します。

CLI は意図的に `node_modules` のパスを **表示しません**。必ず最初に `openclaw browser extension install` を実行し、拡張機能を OpenClaw の状態ディレクトリ配下の安定した場所にコピーしてください。

そのインストールディレクトリを移動または削除すると、有効なパスから再読み込みするまで、Chrome は拡張機能を破損しているものとしてマークします。

<div id="security-implications-read-this">
  ## セキュリティへの影響（必ず読んでください）
</div>

これは強力ですがリスクも大きい機能です。モデルに「あなたのブラウザを直接操作させる」ようなものとして扱ってください。

* この拡張機能は Chrome の debugger API（`chrome.debugger`）を使用します。アタッチされると、モデルはそのタブで次のことができます:
  * クリック／文字入力／ページ遷移
  * ページ内容の読み取り
  * そのタブのログイン済みセッションでアクセス可能なものへのアクセス
* **これは** openclaw が管理する専用プロファイルのように **分離されてはいません**。
  * 日常用のプロファイル／タブにアタッチすると、そのアカウントの状態へのアクセスを許可することになります。

推奨事項:

* 拡張機能リレー用途には、個人用ブラウジングとは分離した専用の Chrome プロファイルを使用してください。
* Gateway とすべてのノードホストは tailnet 内からのみアクセス可能にし、Gateway 認証 + ノードのペアリングに依存してください。
* リレーポートを LAN（`0.0.0.0`）で公開せず、Funnel（パブリック公開）も避けてください。

関連情報:

* ブラウザツールの概要: [Browser](/ja/tools/browser)
* セキュリティ監査: [Security](/ja/gateway/security)
* Tailscale セットアップ: [Tailscale](/ja/gateway/tailscale)