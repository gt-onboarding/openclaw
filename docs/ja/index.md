---
title: インデックス
summary: "OpenClaw の概要・機能・目的に関するトップレベルの説明"
read_when:
  - OpenClaw を初めての人に紹介するとき
---

<div id="openclaw">
  # OpenClaw 🦞
</div>

> *「角質除去せよ！角質除去せよ！」* — たぶん宇宙ロブスター

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text-dark.png" />

    <img src="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text.png" alt="OpenClaw" width="500" />
  </picture>
</p>

<p align="center">
  <strong>任意の OS で動作し、WhatsApp/Telegram/Discord/iMessage を AI エージェント（Pi）向け Gateway にするツール。</strong><br />
  プラグインで Mattermost なども追加可能。
  メッセージを送れば、ポケットの中からエージェントが応答します。
</p>

<p align="center">
  <a href="https://github.com/openclaw/openclaw">GitHub</a> ·
  <a href="https://github.com/openclaw/openclaw/releases">Releases</a> ·
  <a href="/ja/">Docs</a> ·
  <a href="/ja/start/openclaw">OpenClaw assistant のセットアップ</a>
</p>

OpenClaw は、WhatsApp（WhatsApp Web / Baileys 経由）、Telegram（Bot API / grammY）、Discord（Bot API / channels.discord.js）、iMessage（imsg CLI）を、[Pi](https://github.com/badlogic/pi-mono) のようなコーディングエージェントにつなぎます。プラグインによって Mattermost（Bot API + WebSocket）なども追加できます。
OpenClaw は OpenClaw assistant の基盤でもあります。

<div id="start-here">
  ## ここから始める
</div>

* **ゼロからの新規インストール:** [はじめに](/ja/start/getting-started)
* **ガイド付きセットアップ（推奨）:** [Wizard](/ja/start/wizard) (`openclaw onboard`)
* **ダッシュボードを開く（ローカル Gateway）:** http://127.0.0.1:18789/ （または http://localhost:18789/）

Gateway が同じコンピュータ上で動作している場合、そのリンクにアクセスするとブラウザで Control UI がすぐに開きます。開かない場合は、先に Gateway を起動してください: `openclaw gateway`。

<div id="dashboard-browser-control-ui">
  ## ダッシュボード（ブラウザ Control UI）
</div>

ダッシュボードは、チャット、設定、ノード、セッションなどを操作するためのブラウザ版 Control UI です。
ローカルのデフォルト URL: http://127.0.0.1:18789/
リモートアクセス: [Web surfaces](/ja/web) および [Tailscale](/ja/gateway/tailscale)

<p align="center">
  <img src="/whatsapp-openclaw.jpg" alt="OpenClaw" width="420" />
</p>

<div id="how-it-works">
  ## 動作の仕組み
</div>

```
WhatsApp / Telegram / Discord / iMessage (+ plugins)
        │
        ▼
  ┌───────────────────────────┐
  │          Gateway          │  ws://127.0.0.1:18789 (loopback-only)
  │     (single source)       │
  │                           │  http://<gateway-host>:18793
  │                           │    /__openclaw__/canvas/ (Canvas host)
  └───────────┬───────────────┘
              │
              ├─ Pi agent (RPC)
              ├─ CLI (openclaw …)
              ├─ Chat UI (SwiftUI)
              ├─ macOS app (OpenClaw.app)
              ├─ iOS node via Gateway WS + pairing
              └─ Android ノード via Gateway WS + ペアリング
```

ほとんどの処理は **Gateway**（`openclaw gateway`）を経由して実行されます。これはチャネル接続と WebSocket 制御プレーンを管理する単一の常駐プロセスです。

<div id="network-model">
  ## ネットワークモデル
</div>

* **ホストごとに 1 つの Gateway（推奨）**: WhatsApp Web セッションを占有できるプロセスは 1 つだけです。レスキューボットや厳密な分離が必要な場合は、プロファイルおよびポートを分離した複数の Gateway を起動してください。詳しくは [Multiple gateways](/ja/gateway/multiple-gateways) を参照してください。
* **ループバック優先**: Gateway の WS エンドポイントのデフォルトは `ws://127.0.0.1:18789` です。
  * ウィザードは、（ループバック向けであっても）デフォルトで gateway トークンを生成します。
  * Tailnet からアクセスする場合は `openclaw gateway --bind tailnet --token ...` を実行します（ループバック以外へのバインドにはトークンが必須です）。
* **ノード**: 必要に応じて LAN / tailnet / SSH 経由で Gateway の WebSocket に接続します。従来の TCP ブリッジは非推奨となり、削除されています。
* **Canvas ホスト**: `canvasHost.port`（デフォルト `18793`）上の HTTP ファイルサーバーで、ノードの WebView 向けに `/__openclaw__/canvas/` を提供します。詳しくは [Gateway configuration](/ja/gateway/configuration)（`canvasHost`）を参照してください。
* **リモート利用**: SSH トンネルまたは tailnet/VPN を使用します。詳しくは [Remote access](/ja/gateway/remote) および [Discovery](/ja/gateway/discovery) を参照してください。

<div id="features-high-level">
  ## 機能（ハイレベル）
</div>

* 📱 **WhatsApp 連携** — WhatsApp Web プロトコル用に Baileys を使用
* ✈️ **Telegram Bot** — grammY を用いた DM およびグループ
* 🎮 **Discord Bot** — channels.discord.js を用いた DM およびギルドチャンネル
* 🧩 **Mattermost Bot（プラグイン）** — Bot トークン＋WebSocket イベント
* 💬 **iMessage** — ローカル imsg CLI 連携（macOS）
* 🤖 **エージェントブリッジ** — Pi（RPC モード）によるツールストリーミング
* ⏱️ **ストリーミング＋チャンク分割** — ブロックストリーミング＋Telegram 下書きストリーミングの詳細（[/concepts/streaming](/ja/concepts/streaming)）
* 🧠 **マルチエージェントルーティング** — プロバイダーアカウント／ピアを分離されたエージェントへルーティング（ワークスペース＋エージェントごとのセッション）
* 🔐 **サブスクリプション認証** — Anthropic（Claude Pro/Max）＋ OpenAI（ChatGPT/Codex）を OAuth 経由で利用
* 💬 **セッション** — 直接チャットは共有 `main`（デフォルト）に統合され、グループは個別に保持される
* 👥 **グループチャット対応** — デフォルトはメンションベース；オーナーは `/activation always|mention` を切り替え可能
* 📎 **メディア対応** — 画像・音声・ドキュメントの送受信
* 🎤 **ボイスメモ** — オプションの文字起こしフック
* 🖥️ **WebChat + macOS アプリ** — ローカル UI ＋オペレーション／音声起動用メニューバーコンパニオン
* 📱 **iOS ノード** — ノードとしてペアリングされ、Canvas サーフェスを提供
* 📱 **Android ノード** — ノードとしてペアリングされ、Canvas＋Chat＋Camera を提供

Note: 旧 Claude/Codex/Gemini/Opencode パスは削除されています。Pi が唯一のコーディングエージェントパスです。

<div id="quick-start">
  ## クイックスタート
</div>

ランタイム要件：**Node ≥ 22**。

```bash
# Recommended: global install (npm/pnpm)
npm install -g openclaw@latest
# or: pnpm add -g openclaw@latest

# Onboard + install the service (launchd/systemd user service)
openclaw onboard --install-daemon

# Pair WhatsApp Web (shows QR)
openclaw channels login

# オンボード後、Gateway はサービス経由で実行されます; 手動実行も可能:
openclaw gateway --port 18789
```

後から npm 版と git 版のインストールを切り替えるのは簡単です。もう一方の方式をインストールし、`openclaw doctor` を実行して Gateway サービスのエントリーポイントを更新してください。

ソースから（開発用）:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # 初回実行時にUI依存関係を自動インストール
pnpm build
openclaw onboard --install-daemon
```

まだグローバルインストールを行っていない場合は、リポジトリ内で `pnpm openclaw ...` を実行してオンボーディングを行ってください。

マルチインスタンス用クイックスタート（オプション）:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

テストメッセージを送信します（Gateway が起動している必要があります）:

```bash
openclaw message send --target +15555550123 --message "Hello from OpenClaw"
```

<div id="configuration-optional">
  ## 設定（任意）
</div>

設定ファイルは `~/.openclaw/openclaw.json` にあります。

* **何もしない** 場合、OpenClaw は同梱の Pi バイナリを RPC モードで使用し、送信者ごとのセッションを利用します。
* より厳しく制限したい場合は、まず `channels.whatsapp.allowFrom` と（グループ向けには）メンションルールの設定から始めてください。

例:

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } }
    }
  },
  messages: { groupChat: { mentionPatterns: ["@openclaw"] } }
}
```

<div id="docs">
  ## ドキュメント
</div>

* まずはこちらから:
  * [ドキュメントハブ（全ページへのリンク）](/ja/start/hubs)
  * [ヘルプ](/ja/help) ← *よくある問題の修正 + トラブルシューティング*
  * [設定](/ja/gateway/configuration)
  * [設定例](/ja/gateway/configuration-examples)
  * [スラッシュコマンド](/ja/tools/slash-commands)
  * [マルチエージェント・ルーティング](/ja/concepts/multi-agent)
  * [アップデート / ロールバック](/ja/install/updating)
  * [ペアリング（DM + ノード）](/ja/start/pairing)
  * [Nix モード](/ja/install/nix)
  * [OpenClaw アシスタントのセットアップ](/ja/start/openclaw)
  * [スキル](/ja/tools/skills)
  * [スキル設定](/ja/tools/skills-config)
  * [ワークスペース テンプレート](/ja/reference/templates/AGENTS)
  * [RPC アダプター](/ja/reference/rpc)
  * [Gateway ランブック](/ja/gateway)
  * [ノード（iOS/Android）](/ja/nodes)
  * [Web サーフェイス（Control UI）](/ja/web)
  * [ディスカバリ + トランスポート](/ja/gateway/discovery)
  * [リモートアクセス](/ja/gateway/remote)
* プロバイダーと UX:
  * [WebChat](/ja/web/webchat)
  * [Control UI（ブラウザ）](/ja/web/control-ui)
  * [Telegram](/ja/channels/telegram)
  * [Discord](/ja/channels/discord)
  * [Mattermost（プラグイン）](/ja/channels/mattermost)
  * [iMessage](/ja/channels/imessage)
  * [グループ](/ja/concepts/groups)
  * [WhatsApp グループメッセージ](/ja/concepts/group-messages)
  * [メディア: 画像](/ja/nodes/images)
  * [メディア: 音声](/ja/nodes/audio)
* コンパニオンアプリ:
  * [macOS アプリ](/ja/platforms/macos)
  * [iOS アプリ](/ja/platforms/ios)
  * [Android アプリ](/ja/platforms/android)
  * [Windows（WSL2）](/ja/platforms/windows)
  * [Linux アプリ](/ja/platforms/linux)
* 運用と安全性:
  * [セッション](/ja/concepts/session)
  * [Cron ジョブ](/ja/automation/cron-jobs)
  * [Webhook](/ja/automation/webhook)
  * [Gmail フック（Pub/Sub）](/ja/automation/gmail-pubsub)
  * [セキュリティ](/ja/gateway/security)
  * [トラブルシューティング](/ja/gateway/troubleshooting)

<div id="the-name">
  ## 名前について
</div>

**OpenClaw = CLAW + TARDIS** — すべての宇宙ロブスターには、時空マシンが必要だから。

***

*「みんな、結局は自分のプロンプトで遊んでるだけだよ」* — あるAIの発言。おそらくトークンをキメすぎてハイになっていたときのもの

<div id="credits">
  ## クレジット
</div>

* **Peter Steinberger** ([@steipete](https://twitter.com/steipete)) — クリエイター、ロブスター使い
* **Mario Zechner** ([@badlogicc](https://twitter.com/badlogicgames)) — Pi の作者、セキュリティ・ペンテスター
* **Clawd** — より良い名前を要求した宇宙ロブスター

<div id="core-contributors">
  ## コア貢献者
</div>

* **Maxim Vovshin** (@Hyaxia, 36747317+Hyaxia@users.noreply.github.com) — Blogwatcher スキル
* **Nacho Iacovino** (@nachoiacovino, nacho.iacovino@gmail.com) — 位置情報解析（Telegram + WhatsApp）

<div id="license">
  ## ライセンス
</div>

MIT — 海を泳ぐロブスターみたいに自由 🦞

***

*「みんな結局、自分のプロンプトで遊んでいるだけさ。」* — きっとトークンでハイになっているどこかのAI