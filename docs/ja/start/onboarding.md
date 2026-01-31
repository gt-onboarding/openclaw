---
title: オンボーディング
summary: "OpenClaw（macOSアプリ）の初回起動時オンボーディングフロー"
read_when:
  - macOS のオンボーディングアシスタントを設計するとき
  - 認証や ID 設定を実装するとき
---

<div id="onboarding-macos-app">
  # オンボーディング（macOS アプリ）
</div>

このドキュメントでは、**現在の**初回起動時のオンボーディングフローについて説明します。目的は、Gateway をどこで動かすかを選び、認証を設定し、ウィザードを実行し、エージェントが自律的にブートストラップできるようにすることで、スムーズな「Day 0」体験を提供することです。

<div id="page-order-current">
  ## ページ順序（現在）
</div>

1. Welcome 画面 + セキュリティに関する注意事項
2. **Gateway の選択**（ローカル / リモート / 後で設定する）
3. **認証（Anthropic OAuth）** — ローカルのみ
4. **セットアップウィザード**（Gateway 主導）
5. **権限**（TCC のプロンプト）
6. **CLI**（オプション）
7. **オンボーディング用チャット**（専用のセッション）
8. 準備完了

<div id="1-local-vs-remote">
  ## 1) ローカルかリモートか
</div>

**Gateway** はどこで動かしますか？

* **ローカル（この Mac）:** オンボーディング処理で OAuth フローを実行し、認証情報をローカルに書き込めます。
* **リモート（SSH/Tailnet 経由）:** オンボーディング処理はローカルでは OAuth を実行 **しません**。認証情報は Gateway ホスト上に存在している必要があります。
* **あとで設定する:** セットアップをスキップし、アプリを未設定のままにします。

Gateway 認証のヒント:

* ウィザードは現在、ループバック用であっても **トークン** を生成するため、ローカルの WS クライアントも認証が必要です。
* 認証を無効にすると、任意のローカルプロセスが接続できるようになります。これは完全に信頼できるマシンでのみ使用してください。
* 複数マシンからのアクセスや非ループバックでのバインドには **トークン** を使用してください。

<div id="2-local-only-auth-anthropic-oauth">
  ## 2) ローカルのみの認証 (Anthropic OAuth)
</div>

macOS アプリは Anthropic OAuth (Claude Pro/Max) をサポートしています。フローは次のとおりです:

* OAuth (PKCE) のためにブラウザを開く
* ユーザーに `code#state` の値を貼り付けるよう求める
* 資格情報を `~/.openclaw/credentials/oauth.json` に書き込む

その他のプロバイダー (OpenAI やカスタム API など) は、現時点では環境変数または
設定ファイルで設定します。

<div id="3-setup-wizard-gatewaydriven">
  ## 3) セットアップウィザード（Gateway 駆動）
</div>

このアプリは、CLI と同じセットアップウィザードを実行できます。これにより、オンボーディングが Gateway 側の挙動と同期され、SwiftUI 側でロジックを重複実装せずに済みます。

<div id="4-permissions">
  ## 4) 権限
</div>

オンボーディング時に必要となる TCC 権限は次のとおりです:

* 通知
* アクセシビリティ
* 画面収録
* マイク / 音声認識
* 自動化 (AppleScript)

<div id="5-cli-optional">
  ## 5) CLI（オプション）
</div>

このアプリは、グローバルな `openclaw` CLI を npm/pnpm 経由でインストールできるため、
ターミナルでのワークフローや launchd タスクをすぐに利用できるようになります。

<div id="6-onboarding-chat-dedicated-session">
  ## 6) オンボーディング用チャット（専用セッション）
</div>

セットアップ後、アプリはオンボーディング専用のチャットセッションを開き、エージェントが
自己紹介を行い、次に何をすればよいかを案内できるようにします。これにより、初回起動時のガイダンスを
通常の会話とは切り離しておくことができます。

<div id="agent-bootstrap-ritual">
  ## エージェントのブートストラップ手順
</div>

エージェントを最初に実行すると、OpenClaw はワークスペース（デフォルト `~/.openclaw/workspace`）をブートストラップします：

* `AGENTS.md`、`BOOTSTRAP.md`、`IDENTITY.md`、`USER.md` を作成する
* 短い Q&amp;A 形式の対話（一度に 1 質問ずつ）を実行する
* `IDENTITY.md`、`USER.md`、`SOUL.md` にアイデンティティおよび設定をを書き込む
* 終了時に `BOOTSTRAP.md` を削除し、この処理が 1 回だけ実行されるようにする

<div id="optional-gmail-hooks-manual">
  ## オプション: Gmail フック（手動）
</div>

Gmail Pub/Sub のセットアップは現在手動で行う必要があります。以下を使用してください:

```bash
openclaw webhooks gmail setup --account you@gmail.com
```

詳細は、[/automation/gmail-pubsub](/ja/automation/gmail-pubsub) を参照してください。

<div id="remote-mode-notes">
  ## リモートモードに関する注意事項
</div>

Gateway が別のマシンで動作している場合、認証情報とワークスペースのファイルは
**そのホスト上に** 保存されます。リモートモードで OAuth が必要な場合は、次のファイルを作成します。

* `~/.openclaw/credentials/oauth.json`
* `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

これらは Gateway ホスト上に作成してください。