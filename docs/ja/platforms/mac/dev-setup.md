---
title: 開発環境セットアップ
summary: "OpenClaw の macOS アプリを開発するためのセットアップガイド"
read_when:
  - macOS の開発環境をセットアップするとき
---

<div id="macos-developer-setup">
  # macOS 開発者向けセットアップ
</div>

本ガイドでは、OpenClaw の macOS アプリケーションをソースコードからビルドして実行するために必要な手順を説明します。

<div id="prerequisites">
  ## 前提条件
</div>

アプリをビルドする前に、次のものがインストールされていることを確認してください:

1.  **Xcode 26.2+**: Swift 開発に必要です。
2.  **Node.js 22+ & pnpm**: Gateway、CLI、およびパッケージング用のスクリプトに必要です。

<div id="1-install-dependencies">
  ## 1. 依存関係のインストール
</div>

プロジェクト全体で使用する依存パッケージをインストールします：

```bash
pnpm install
```


<div id="2-build-and-package-the-app">
  ## 2. アプリのビルドとパッケージ化
</div>

macOS アプリをビルドし、`dist/OpenClaw.app` にパッケージするには、次を実行します。

```bash
./scripts/package-mac-app.sh
```

Apple Developer ID 証明書がない場合、スクリプトは自動的に **アドホック署名**（`-`）を使用します。

開発用の実行モードや署名フラグ、Team ID に関するトラブルシューティングについては、macOS アプリの README を参照してください:
https://github.com/openclaw/openclaw/blob/main/apps/macos/README.md

> **注意**: アドホック署名されたアプリは、セキュリティ関連の確認プロンプトが表示される場合があります。アプリが起動直後に &quot;Abort trap 6&quot; というメッセージとともにクラッシュする場合は、[Troubleshooting](#troubleshooting) セクションを参照してください。


<div id="3-install-the-cli">
  ## 3. CLI をインストールする
</div>

macOS アプリは、バックグラウンドタスクを管理するために、グローバルにインストールされた `openclaw` CLI の存在を想定しています。

**インストールする方法（推奨）:**

1. OpenClaw アプリを開きます。
2. 設定の **General** タブに移動します。
3. **&quot;Install CLI&quot;** をクリックします。

別の方法として、手動でインストールすることもできます:

```bash
npm install -g openclaw@<version>
```


<div id="troubleshooting">
  ## トラブルシューティング
</div>

<div id="build-fails-toolchain-or-sdk-mismatch">
  ### ビルドが失敗する：ツールチェーンまたは SDK の不一致
</div>

macOS アプリのビルドには、最新の macOS SDK と Swift 6.2 ツールチェーンが必要です。

**システム要件（必須）：**

* **ソフトウェアアップデートで利用可能な最新の macOS バージョン**（Xcode 26.2 の SDK に必須）
* **Xcode 26.2**（Swift 6.2 ツールチェーン）

**確認事項：**

```bash
xcodebuild -version
xcrun swift --version
```

バージョンが一致していない場合は、macOS と Xcode を更新してからビルドを再実行してください。


<div id="app-crashes-on-permission-grant">
  ### 権限付与時にアプリがクラッシュする
</div>

**Speech Recognition** または **Microphone** へのアクセスを許可しようとした際にアプリがクラッシュする場合、破損した TCC キャッシュや署名の不一致が原因の可能性があります。

**対処方法:**

1. TCC 権限をリセットします:
   ```bash
   tccutil reset All bot.molt.mac.debug
   ```
2. それでも解決しない場合は、macOS に「初期状態」として認識させるために、[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) 内の `BUNDLE_ID` を一時的に変更します。

<div id="gateway-starting-indefinitely">
  ### Gateway が「Starting...」のまま終わらない
</div>

Gateway のステータスが「Starting...」のまま変わらない場合、ゾンビプロセスがポートを占有していないか確認してください。

```bash
openclaw gateway status
openclaw gateway stop

# LaunchAgentを使用していない場合(開発モード/手動実行)は、リスナーを確認:
lsof -nP -iTCP:18789 -sTCP:LISTEN
```

手動で起動したプロセスがポートを占有している場合は、そのプロセスを停止してください（Ctrl+C）。最後の手段として、上で特定した PID を kill して強制終了してください。
