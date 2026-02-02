---
title: はじめに
summary: "入門ガイド：ゼロから最初のメッセージまで（ウィザード、認証、チャネル、ペアリング）"
read_when:
  - ゼロから初期セットアップを行うとき
  - インストール → オンボーディング → 最初のメッセージまでを最短経路で進めたいとき
---

<div id="getting-started">
  # はじめに
</div>

目標: **ゼロ** から **最初に動作するチャット**（妥当なデフォルト付き）まで、できるだけ素早く到達すること。

最も手っ取り早いチャット: Control UI を開きます（チャンネル設定は不要）。`openclaw dashboard`
を実行してブラウザでチャットするか、Gateway ホスト上で `http://127.0.0.1:18789/` を開きます。
ドキュメント: [Dashboard](/ja/web/dashboard) と [Control UI](/ja/web/control-ui)。

推奨パス: **CLI オンボーディングウィザード**（`openclaw onboard`）を使います。これにより次の項目がセットアップされます:

* モデル/認証（OAuth 推奨）
* Gateway 設定
* チャンネル（WhatsApp/Telegram/Discord/Mattermost（プラグイン）/...）
* ペアリングのデフォルト（安全な DM）
* ワークスペースのブートストラップ + スキル
* （オプション）バックグラウンドサービス

より詳細なリファレンスページが必要な場合は、次を参照してください: [Wizard](/ja/start/wizard), [Setup](/ja/start/setup), [Pairing](/ja/start/pairing), [Security](/ja/gateway/security)。

サンドボックスに関する注意: `agents.defaults.sandbox.mode: "non-main"` は `session.mainKey`（デフォルトは `"main"`）を使用するため、
グループ/チャンネルのセッションはサンドボックス化されます。メインエージェントを常に
ホスト上で動かしたい場合は、エージェントごとの明示的なオーバーライドを設定してください:

```json
{
  "routing": {
    "agents": {
      "main": {
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      }
    }
  }
}
```

<div id="0-prereqs">
  ## 0) 前提条件
</div>

* Node `>=22`
* `pnpm`（任意。ソースコードからビルドする場合は推奨）
* **推奨:** Web 検索用の Brave Search API キー。最も簡単な方法は次のとおりです:
  `openclaw configure --section web`（`tools.web.search.apiKey` に保存されます）。
  [Web tools](/ja/tools/web) を参照してください。

macOS: アプリをビルドする予定がある場合は、Xcode / CLT をインストールしてください。CLI と Gateway のみであれば、Node だけで十分です。
Windows: **WSL2**（Ubuntu 推奨）を使用してください。WSL2 を強く推奨します。ネイティブ Windows は未検証で問題が発生しやすく、ツールの互換性も劣ります。まず WSL2 をインストールし、その中の Linux 環境で以降の Linux 向け手順を実行してください。[Windows (WSL2)](/ja/platforms/windows) を参照してください。

<div id="1-install-the-cli-recommended">
  ## 1) CLI をインストールする（推奨）
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

インストーラーのオプション（インストール方法、非対話型、GitHub からのインストール）については、[Install](/ja/install) を参照してください。

Windows（PowerShell）:

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

別の方法（グローバルインストール）:

```bash
npm install -g openclaw@latest
```

```bash
pnpm add -g openclaw@latest
```

<div id="2-run-the-onboarding-wizard-and-install-the-service">
  ## 2) オンボーディングウィザードを実行する（サービスをインストール）
</div>

```bash
openclaw onboard --install-daemon
```

ここで決める項目:

* **ローカル vs リモート** Gateway
* **認証**: OpenAI Code (Codex) サブスクリプション (OAuth) または API キー。Anthropic には API キーを推奨します。`claude setup-token` にも対応しています。
* **プロバイダー**: WhatsApp の QR ログイン、Telegram/Discord の bot トークン、Mattermost プラグインのトークンなど。
* **デーモン**: バックグラウンドへのインストール（launchd/systemd。WSL2 は systemd を使用）
  * **ランタイム**: ノード（推奨。WhatsApp/Telegram では必須）。Bun は**非推奨**です。
* **Gateway トークン**: ウィザードは（ループバック接続であっても）デフォルトで 1 つ生成し、`gateway.auth.token` に保存します。

ウィザードのドキュメント: [Wizard](/ja/start/wizard)

<div id="auth-where-it-lives-important">
  ### 認証情報: どこに保存されるか（重要）
</div>

* **推奨される Anthropic の設定方法:** API キーを設定します（ウィザードでサービス用に保存できます）。Claude Code の認証情報を再利用したい場合は `claude setup-token` も利用できます。

* OAuth 認証情報（レガシーインポート）：`~/.openclaw/credentials/oauth.json`

* 認証プロファイル（OAuth + API キー）：`~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

ヘッドレス/サーバー環境向けのヒント: まず通常のマシンで OAuth を実行し、その後 `oauth.json` を Gateway ホストにコピーしてください。

<div id="3-start-the-gateway">
  ## 3) Gateway を起動する
</div>

オンボーディング中にサービスをインストールしていれば、Gatewayはすでに起動しているはずです。

```bash
openclaw gateway status
```

手動での実行（フォアグラウンド）:

```bash
openclaw gateway --port 18789 --verbose
```

ダッシュボード（ローカルループバック）：`http://127.0.0.1:18789/`
トークンを設定している場合は、Control UI の設定画面に貼り付けてください（`connect.params.auth.token` として保存されます）。

⚠️ **Bun に関する注意（WhatsApp + Telegram）：** Bun はこれらのチャンネルで既知の問題が報告されています。
WhatsApp または Telegram を使用する場合は、Gateway を **Node** で実行してください。

<div id="35-quick-verify-2-min">
  ## 3.5) かんたん動作確認 (2分)
</div>

```bash
openclaw status
openclaw health
openclaw security audit --deep
```

<div id="4-pair-connect-your-first-chat-surface">
  ## 4) 最初のチャットチャネルをペアリングして接続する
</div>

<div id="whatsapp-qr-login">
  ### WhatsApp（QRログイン）
</div>

```bash
openclaw channels login
```

WhatsApp を開き、「Settings」→「Linked Devices」からスキャンします。

WhatsApp のドキュメント: [WhatsApp](/ja/channels/whatsapp)

<div id="telegram-discord-others">
  ### Telegram / Discord / その他
</div>

ウィザードがトークンや設定を自動作成してくれます。手動で設定したい場合は、次を参照してください:

* Telegram: [Telegram](/ja/channels/telegram)
* Discord: [Discord](/ja/channels/discord)
* Mattermost (プラグイン): [Mattermost](/ja/channels/mattermost)

**Telegram の DM に関するヒント:** 最初の DM を送るとペアリングコードが返されます。それを承認しないと（次の手順を参照）、ボットは返信しません。

<div id="5-dm-safety-pairing-approvals">
  ## 5) DM の安全性（ペアリング承認）
</div>

デフォルトの動作: 未知の送信者からの DM には短いコードが表示され、承認されるまでメッセージは処理されません。
最初の DM を送っても返信がない場合は、ペアリングを承認してください。

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <code>
```

ペアリングのドキュメント：[Pairing](/ja/start/pairing)

<div id="from-source-development">
  ## ソースから（開発版）
</div>

OpenClaw 自体の開発を行っている場合は、ソースコードから実行してください。

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # 初回実行時にUI依存関係を自動インストールします
pnpm build
openclaw onboard --install-daemon
```

まだグローバルインストールをしていない場合は、リポジトリ内で `pnpm openclaw ...` を実行してオンボーディング手順を実行してください。
`pnpm build` は A2UI アセットもバンドルします。そのステップだけを実行したい場合は、`pnpm canvas:a2ui:bundle` を使用してください。

Gateway（このリポジトリから）:

```bash
node openclaw.mjs gateway --port 18789 --verbose
```

<div id="7-verify-end-to-end">
  ## 7) エンドツーエンドで検証する
</div>

新しいターミナルを開き、テストメッセージを送信します：

```bash
openclaw message send --target +15555550123 --message "Hello from OpenClaw"
```

`openclaw health` が「no auth configured」と表示された場合は、ウィザードに戻って OAuth / API キー認証を設定してください — それがないとエージェントは応答できません。

ヒント: `openclaw status --all` は、貼り付けに適した読み取り専用のデバッグレポートとして最適です。
ヘルスチェック: `openclaw health`（または `openclaw status --deep`）は、実行中の Gateway にヘルス状態のスナップショットを問い合わせます。

<div id="next-steps-optional-but-great">
  ## 次のステップ（任意ですが強く推奨）
</div>

* macOS メニューバーアプリ + 音声ウェイク: [macOS app](/ja/platforms/macos)
* iOS/Android ノード（Canvas/カメラ/音声）: [Nodes](/ja/nodes)
* リモートアクセス（SSHトンネル/Tailscale Serve）: [Remote access](/ja/gateway/remote) と [Tailscale](/ja/gateway/tailscale)
* 常時稼働 / VPN セットアップ: [Remote access](/ja/gateway/remote)、[exe.dev](/ja/platforms/exe-dev)、[Hetzner](/ja/platforms/hetzner)、[macOS remote](/ja/platforms/mac/remote)