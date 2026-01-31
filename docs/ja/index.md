---
title: Index
summary: "Top-level overview of OpenClaw, features, and purpose"
read_when:
  - Introducing OpenClaw to newcomers
---

<div id="openclaw">
  # OpenClaw 🦞
</div>

> *&quot;EXFOLIATE! EXFOLIATE!&quot;* — A space lobster, probably

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text-dark.png" />

    <img src="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text.png" alt="OpenClaw" width="500" />
  </picture>
</p>

<p align="center">
  <strong>Any OS + WhatsApp/Telegram/Discord/iMessage gateway for AI agents (Pi).</strong><br />
  Plugins add Mattermost and more.
  Send a message, get an agent response — from your pocket.
</p>

<p align="center">
  <a href="https://github.com/openclaw/openclaw">GitHub</a> ·
  <a href="https://github.com/openclaw/openclaw/releases">Releases</a> ·
  <a href="/ja/">Docs</a> ·
  <a href="/ja/start/openclaw">OpenClaw assistant setup</a>
</p>

OpenClawは、WhatsApp(WhatsApp Web / Baileys経由)、Telegram(Bot API / grammY)、Discord(Bot API / channels.discord.js)、およびiMessage(imsg CLI)を、[Pi](https://github.com/badlogic/pi-mono)のようなコーディングエージェントに接続します。プラグインによりMattermost(Bot API + WebSocket)などを追加できます。
OpenClawはOpenClawアシスタントも駆動します。

<div id="start-here">
  ## ここから始める
</div>

* **ゼロから新規インストール:** [はじめに](/ja/start/getting-started)
* **ガイド付きセットアップ（推奨）:** [Wizard](/ja/start/wizard) (`openclaw onboard`)
* **ダッシュボードを開く（ローカル Gateway）:** http://127.0.0.1:18789/ （または http://localhost:18789/）

Gateway が同じコンピュータ上で稼働している場合、そのリンクにアクセスするとブラウザで Control UI が
すぐに表示されます。開けない場合は、先に Gateway を起動してください（`openclaw gateway`）。

<div id="dashboard-browser-control-ui">
  ## ダッシュボード（ブラウザ版 Control UI）
</div>

ダッシュボードは、チャット、設定、ノード、セッションなどを操作するためのブラウザベースの Control UI です。
ローカルでのデフォルト: http://127.0.0.1:18789/
リモートアクセス: [Web インターフェース](/ja/web) および [Tailscale](/ja/gateway/tailscale)

<p align="center">
  <img src="/whatsapp-openclaw.jpg" alt="OpenClaw" width="420" />
</p>

<div id="how-it-works">
  ## 動作概要
</div>

```
WhatsApp / Telegram / Discord / iMessage (+ プラグイン)
        │
        ▼
  ┌───────────────────────────┐
  │          Gateway          │  ws://127.0.0.1:18789 (ループバックのみ)
  │     (単一ソース)           │
  │                           │  http://<gateway-host>:18793
  │                           │    /__openclaw__/canvas/ (Canvasホスト)
  └───────────┬───────────────┘
              │
              ├─ Piエージェント (RPC)
              ├─ CLI (openclaw …)
              ├─ チャットUI (SwiftUI)
              ├─ macOS app (OpenClaw.app)
              ├─ iOSノード (Gateway WS + ペアリング経由)
              └─ Androidノード (Gateway WS + ペアリング経由)
```

ほとんどの操作は **Gateway**（`openclaw gateway`）を経由して処理されます。Gateway は、チャネル接続と WebSocket コントロールプレーンを保持・管理する、単一の常駐プロセスです。

<div id="network-model">
  ## ネットワークモデル
</div>

* **ホストあたり 1 つの Gateway（推奨）**: WhatsApp Web セッションを所有できるプロセスはこの Gateway のみです。バックアップ用ボットや厳密な分離が必要な場合は、プロファイルとポートを分離した複数の Gateway を起動してください。詳しくは [Multiple gateways](/ja/gateway/multiple-gateways) を参照してください。
* **ループバック優先**: Gateway の WS のデフォルトは `ws://127.0.0.1:18789` です。
  * ウィザードは現在、デフォルトで（ループバックであっても）Gateway トークンを生成します。
  * Tailnet 経由でアクセスする場合は `openclaw gateway --bind tailnet --token ...` を実行します（ループバック以外にバインドする場合はトークンが必須です）。
* **ノード**: 必要に応じて LAN / tailnet / SSH 経由で Gateway の WebSocket に接続します。従来の TCP ブリッジは非推奨となり、削除されています。
* **Canvas ホスト**: `canvasHost.port`（デフォルト `18793`）上で動作する HTTP ファイルサーバーで、ノードの WebView 向けに `/__openclaw__/canvas/` を配信します。詳細は [Gateway configuration](/ja/gateway/configuration)（`canvasHost`）を参照してください。
* **リモートからの利用**: SSH トンネルまたは tailnet/VPN を使用します。詳しくは [Remote access](/ja/gateway/remote) および [Discovery](/ja/gateway/discovery) を参照してください。

<div id="features-high-level">
  ## 機能（ハイレベル）
</div>

* 📱 **WhatsApp 連携** — Baileys を使用した WhatsApp Web プロトコル
* ✈️ **Telegram Bot** — grammY を用いた DM とグループ対応
* 🎮 **Discord Bot** — channels.discord.js を用いた DM とギルドチャンネル対応
* 🧩 **Mattermost Bot（プラグイン）** — Bot トークンおよび WebSocket イベント
* 💬 **iMessage** — ローカル imsg CLI 連携（macOS）
* 🤖 **エージェントブリッジ** — Pi（RPC モード）でのツールストリーミング
* ⏱️ **ストリーミング + チャンク分割** — ブロックストリーミングおよび Telegram 下書きストリーミングの詳細（[/concepts/streaming](/ja/concepts/streaming)）
* 🧠 **マルチエージェントルーティング** — プロバイダーアカウント／ピアを分離されたエージェントにルーティング（ワークスペース + エージェントごとのセッション）
* 🔐 **サブスクリプション認証** — Anthropic（Claude Pro/Max）+ OpenAI（ChatGPT/Codex）を OAuth 経由で利用
* 💬 **セッション** — ダイレクトチャットは共有 `main`（デフォルト）に集約され、グループは分離される
* 👥 **グループチャット対応** — デフォルトはメンションベース。オーナーは `/activation always|mention` を切り替え可能
* 📎 **メディア対応** — 画像、音声、ドキュメントの送受信
* 🎤 **ボイスメモ** — オプションの音声文字起こしフック
* 🖥️ **WebChat + macOS アプリ** — ローカル UI と、運用および音声ウェイク用のメニューバーコンパニオン
* 📱 **iOS ノード** — ノードとしてペアリングされ、Canvas サーフェスを提供
* 📱 **Android ノード** — ノードとしてペアリングされ、Canvas + チャット + カメラを提供

注意: 旧来の Claude/Codex/Gemini/Opencode 経路は削除されており、Pi が唯一のコーディング用エージェント経路です。

<div id="quick-start">
  ## クイックスタート
</div>

実行環境要件: **Node ≥ 22**。

```bash
# Recommended: global install (npm/pnpm)
npm install -g openclaw@latest
# or: pnpm add -g openclaw@latest

# Onboard + install the service (launchd/systemd user service)
openclaw onboard --install-daemon

# Pair WhatsApp Web (shows QR)
openclaw channels login

# オンボーディング完了後は Gateway はサービスとして実行されますが、手動実行も引き続き可能です:
openclaw gateway --port 18789
```

後からnpmインストールとgitインストールを切り替えるのは簡単です。もう一方の方法でインストールし、`openclaw doctor`を実行してGatewayサービスのエントリーポイントを更新してください。

ソースから(開発用):

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # 初回実行時にUI依存関係を自動インストール
pnpm build
openclaw onboard --install-daemon
```

まだグローバルインストールしていない場合は、リポジトリ内で `pnpm openclaw ...` を実行してオンボーディング手順を実行してください。

マルチインスタンス向けクイックスタート（任意）:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

テストメッセージを送信する（Gateway が起動している必要があります）:

```bash
openclaw message send --target +15555550123 --message "Hello from OpenClaw"
```

<div id="configuration-optional">
  ## 設定（任意）
</div>

設定ファイルは `~/.openclaw/openclaw.json` にあります。

* **何もしない** 場合、OpenClaw は同梱の Pi バイナリを RPC モードで使用し、送信者ごとにセッションを分けて利用します。
* より厳しく制限したい場合は、まず `channels.whatsapp.allowFrom` と（グループ向けには）メンションに関するルールを設定してください。

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
  * [ドキュメントハブ（全ページ一覧）](/ja/start/hubs)
  * [ヘルプ](/ja/help) ← *よくある問題の解決 + トラブルシューティング*
  * [設定](/ja/gateway/configuration)
  * [設定例](/ja/gateway/configuration-examples)
  * [スラッシュコマンド](/ja/tools/slash-commands)
  * [マルチエージェントルーティング](/ja/concepts/multi-agent)
  * [アップデート / ロールバック](/ja/install/updating)
  * [ペアリング（DM + ノード）](/ja/start/pairing)
  * [Nix モード](/ja/install/nix)
  * [OpenClaw アシスタントのセットアップ](/ja/start/openclaw)
  * [スキル](/ja/tools/skills)
  * [スキル設定](/ja/tools/skills-config)
  * [ワークスペーステンプレート](/ja/reference/templates/AGENTS)
  * [RPC アダプター](/ja/reference/rpc)
  * [Gateway ランブック](/ja/gateway)
  * [ノード（iOS/Android）](/ja/nodes)
  * [Web インターフェース（Control UI）](/ja/web)
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
  * [Webhook（ウェブフック）](/ja/automation/webhook)
  * [Gmail フック（Pub/Sub）](/ja/automation/gmail-pubsub)
  * [セキュリティ](/ja/gateway/security)
  * [トラブルシューティング](/ja/gateway/troubleshooting)

<div id="the-name">
  ## 名前の由来
</div>

**OpenClaw = CLAW + TARDIS** — すべての宇宙ロブスターには、時空を旅するマシンが必要だからです。

***

*「みんな結局、自分のプロンプトで遊んでるだけさ。」* — おそらくトークンでハイになっているAI

<div id="credits">
  ## 謝辞
</div>

* **Peter Steinberger** ([@steipete](https://twitter.com/steipete)) — クリエイター、ロブスターささやき係
* **Mario Zechner** ([@badlogicc](https://twitter.com/badlogicgames)) — Pi の生みの親、セキュリティペネトレーションテスター
* **Clawd** — もっとマシな名前を要求した宇宙ロブスター

<div id="core-contributors">
  ## コアコントリビューター
</div>

* **Maxim Vovshin** (@Hyaxia, 36747317+Hyaxia@users.noreply.github.com) — Blogwatcher スキル
* **Nacho Iacovino** (@nachoiacovino, nacho.iacovino@gmail.com) — 位置情報のパース（Telegram + WhatsApp）

<div id="license">
  ## ライセンス
</div>

MIT — 海のロブスターみたいに自由 🦞

***

*「私たちは皆、自分のプロンプトで遊んでいるだけだ」* — たぶんトークンに酔っぱらっているAI