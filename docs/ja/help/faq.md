---
title: よくある質問
summary: "OpenClaw のセットアップ、設定、利用方法に関するよくある質問"
---

<div id="faq">
  # FAQ
</div>

ローカル開発、VPS、マルチエージェント構成、OAuth/API キー、モデルフェイルオーバーなど、実運用環境でのセットアップ向けに、簡潔な回答とより踏み込んだトラブルシューティングをまとめています。実行時診断については [Troubleshooting](/ja/gateway/troubleshooting) を参照してください。設定の完全なリファレンスについては [Configuration](/ja/gateway/configuration) を参照してください。

<div id="table-of-contents">
  ## 目次
</div>

* [クイックスタートと初回起動時のセットアップ](#quick-start-and-firstrun-setup)
  * [行き詰まりました。最速で抜け出すには？](#im-stuck-whats-the-fastest-way-to-get-unstuck)
  * [OpenClaw の推奨インストール方法とセットアップ方法は？](#whats-the-recommended-way-to-install-and-set-up-openclaw)
  * [オンボーディング後にダッシュボードを開くには？](#how-do-i-open-the-dashboard-after-onboarding)
  * [localhost とリモートの場合で、ダッシュボード（トークン）を認証する方法は？](#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
  * [どのランタイムが必要ですか？](#what-runtime-do-i-need)
  * [Raspberry Pi で動作しますか？](#does-it-run-on-raspberry-pi)
  * [Raspberry Pi へのインストールのコツはありますか？](#any-tips-for-raspberry-pi-installs)
  * [「wake up my friend」で止まる／オンボーディングが進みません。どうすればいいですか？](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
  * [オンボーディングをやり直さずに、新しいマシン（Mac mini）へセットアップを移行できますか？](#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
  * [最新バージョンの変更点はどこで確認できますか？](#where-do-i-see-whats-new-in-the-latest-version)
  * [docs.openclaw.ai にアクセスできません（SSL エラー）。どうすればいいですか？](#i-cant-access-docsopenclawai-ssl-error-what-now)
  * [stable と beta の違いは何ですか？](#whats-the-difference-between-stable-and-beta)
* [ベータ版はどうやってインストールしますか？ また、ベータ版と dev 版の違いは何ですか？](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
  * [最新の開発版を試すにはどうすればいいですか？](#how-do-i-try-the-latest-bits)
  * [インストールと初期設定には通常どれくらい時間がかかりますか？](#how-long-does-install-and-onboarding-usually-take)
  * [インストーラーが止まった場合、より詳細な出力を確認するには？](#installer-stuck-how-do-i-get-more-feedback)
  * [Windows へのインストール時に git が見つからない、または openclaw が認識されないと表示される](#windows-install-says-git-not-found-or-openclaw-not-recognized)
  * [ドキュメントを読んでも疑問が解決しません — どうすればよりよい回答を得られますか？](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
  * [Linux に OpenClaw をインストールするには？](#how-do-i-install-openclaw-on-linux)
  * [VPS に OpenClaw をインストールする方法](#how-do-i-install-openclaw-on-a-vps)
  * [クラウド/VPS 向けインストールガイドはどこにありますか？](#where-are-the-cloudvps-install-guides)
  * [OpenClaw に自身のアップデートを任せることはできますか？](#can-i-ask-openclaw-to-update-itself)
  * [オンボーディングウィザードでは実際に何を行いますか？](#what-does-the-onboarding-wizard-actually-do)
  * [これを動かすのに Claude または OpenAI のサブスクリプションは必要ですか？](#do-i-need-a-claude-or-openai-subscription-to-run-this)
  * [Claude Max のサブスクリプションは API キーなしで使えますか](#can-i-use-claude-max-subscription-without-an-api-key)
  * [Anthropic の「setup-token」認証はどのように機能しますか？](#how-does-anthropic-setuptoken-auth-work)
  * [Anthropic のセットアップトークンはどこで取得できますか？](#where-do-i-find-an-anthropic-setuptoken)
  * [Claude のサブスクリプション認証（Claude Code OAuth）には対応していますか？](#do-you-support-claude-subscription-auth-claude-code-oauth)
  * [Anthropic から `HTTP 429: rate_limit_error` が表示されるのはなぜですか？](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
  * [AWS Bedrock に対応していますか？](#is-aws-bedrock-supported)
  * [Codex 認証はどのように動作しますか？](#how-does-codex-auth-work)
  * [OpenAI のサブスクリプション認証（Codex OAuth）に対応していますか？](#do-you-support-openai-subscription-auth-codex-oauth)
  * [Gemini CLI で OAuth を設定するには？](#how-do-i-set-up-gemini-cli-oauth)
  * [カジュアルな雑談にはローカルモデルでも大丈夫ですか？](#is-a-local-model-ok-for-casual-chats)
  * [ホスト型モデルのトラフィックを特定のリージョン内にとどめるには？](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
  * [これをインストールするには Mac mini を購入する必要がありますか？](#do-i-have-to-buy-a-mac-mini-to-install-this)
  * [iMessage 対応には Mac mini が必要ですか？](#do-i-need-a-mac-mini-for-imessage-support)
  * [OpenClaw を実行するために Mac mini を購入した場合、それを MacBook Pro に接続できますか？](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
  * [Bun は使えますか？](#can-i-use-bun)
  * [Telegram：`allowFrom` には何を設定すればよいですか？](#telegram-what-goes-in-allowfrom)
  * [複数人が異なる OpenClaw インスタンスから同じ WhatsApp 番号を利用できますか？](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
  * [「fast chat」エージェントと「Opus for coding」エージェントを同時に実行できますか？](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
  * [Homebrew は Linux で動作しますか？](#does-homebrew-work-on-linux)
  * [ハックしやすい git インストールと npm インストールの違いは何ですか？](#whats-the-difference-between-the-hackable-git-install-and-npm-install)
  * [あとから npm インストールと git インストールを切り替えられますか？](#can-i-switch-between-npm-and-git-installs-later)
  * [Gateway は自分のノートPCと VPS のどちらで動かすべきですか？](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
  * [OpenClaw を専用マシンで動かすことはどの程度重要ですか？](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
  * [最低要件のVPSスペックと推奨OSは？](#what-are-the-minimum-vps-requirements-and-recommended-os)
  * [OpenClaw を VM 上で実行できますか？ その場合の要件は何ですか](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)
* [OpenClaw とは？](#what-is-openclaw)
  * [OpenClaw をひと言でいうと何ですか？](#what-is-openclaw-in-one-paragraph)
  * [バリュープロポジションは何ですか？](#whats-the-value-proposition)
  * [セットアップが終わりました。最初に何をすればいいですか？](#i-just-set-it-up-what-should-i-do-first)
  * [OpenClaw の日常的な代表ユースケースのトップ 5 は何ですか？](#what-are-the-top-five-everyday-use-cases-for-openclaw)
  * [OpenClaw は SaaS のリード獲得・アウトリーチ・広告・ブログ作成に役立ちますか？](#can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas)
  * [Web 開発において Claude Code と比べた利点は何ですか？](#what-are-the-advantages-vs-claude-code-for-web-development)
* [スキルと自動化](#skills-and-automation)
  * [リポジトリを汚さずにスキルをカスタマイズするにはどうすればよいですか？](#how-do-i-customize-skills-without-keeping-the-repo-dirty)
  * [カスタムフォルダーからスキルを読み込むことはできますか？](#can-i-load-skills-from-a-custom-folder)
  * [タスクごとに異なるモデルを使うにはどうすればよいですか？](#how-can-i-use-different-models-for-different-tasks)
  * [ボットが重い処理中にフリーズします。どのようにオフロードすればよいですか？](#the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that)
  * [Cron やリマインダーが実行されません。どこを確認すればよいですか？](#cron-or-reminders-do-not-fire-what-should-i-check)
  * [Linux でスキルをインストールするにはどうすればよいですか？](#how-do-i-install-skills-on-linux)
  * [OpenClaw はタスクをスケジュール実行したり、バックグラウンドで継続的に実行したりできますか？](#can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background)
  * [Linux から Apple/macOS 専用スキルを実行できますか？](#can-i-run-applemacosonly-skills-from-linux)
  * [Notion や HeyGen との連携はありますか？](#do-you-have-a-notion-or-heygen-integration)
  * [ブラウザー操作用の Chrome 拡張機能をインストールするにはどうすればよいですか？](#how-do-i-install-the-chrome-extension-for-browser-takeover)
* [サンドボックスとメモリ](#sandboxing-and-memory)
  * [サンドボックスに関する専用ドキュメントはありますか？](#is-there-a-dedicated-sandboxing-doc)
  * [ホストのフォルダをサンドボックスにマウントするにはどうすればよいですか？](#how-do-i-bind-a-host-folder-into-the-sandbox)
  * [メモリはどのように動作しますか？](#how-does-memory-work)
  * [メモリがすぐに忘れてしまいます。きちんと保持させるにはどうすればよいですか？](#memory-keeps-forgetting-things-how-do-i-make-it-stick)
  * [メモリは永続的ですか？どんな制限がありますか？](#does-memory-persist-forever-what-are-the-limits)
  * [セマンティックメモリ検索には OpenAI の API キーが必要ですか？](#does-semantic-memory-search-require-an-openai-api-key)
* [ディスク上の保存場所](#where-things-live-on-disk)
  * [OpenClaw で利用するすべてのデータはローカルに保存されますか？](#is-all-data-used-with-openclaw-saved-locally)
  * [OpenClaw はデータをどこに保存しますか？](#where-does-openclaw-store-its-data)
  * [AGENTS.md / SOUL.md / USER.md / MEMORY.md はどこに配置すべきですか？](#where-should-agentsmd-soulmd-usermd-memorymd-live)
  * [推奨されるバックアップ戦略は何ですか？](#whats-the-recommended-backup-strategy)
  * [OpenClaw を完全にアンインストールするにはどうすればよいですか？](#how-do-i-completely-uninstall-openclaw)
  * [エージェントはワークスペースの外でも動作できますか？](#can-agents-work-outside-the-workspace)
  * [リモートモードです — セッションストアはどこにありますか？](#im-in-remote-mode-where-is-the-session-store)
* [基本的な設定](#config-basics)
  * [設定ファイルはどんな形式で、どこにありますか？](#what-format-is-the-config-where-is-it)
  * [`gateway.bind: "lan"`（または `"tailnet"`）を設定したら、何も待ち受けなくなった / UI に「unauthorized」と表示されます](#i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized)
  * [なぜ今は localhost でもトークンが必要なのですか？](#why-do-i-need-a-token-on-localhost-now)
  * [設定を変更したら再起動しないといけませんか？](#do-i-have-to-restart-after-changing-config)
  * [Web 検索（および Web fetch）を有効にするにはどうすればよいですか？](#how-do-i-enable-web-search-and-web-fetch)
  * [`config.apply` で設定が消えました。どう復旧し、今後どう防げばよいですか？](#configapply-wiped-my-config-how-do-i-recover-and-avoid-this)
  * [複数デバイスに特化ワーカーを置いて中央の Gateway を動かすには？](#how-do-i-run-a-central-gateway-with-specialized-workers-across-devices)
  * [OpenClaw ブラウザはヘッドレスで実行できますか？](#can-the-openclaw-browser-run-headless)
  * [ブラウザ制御に Brave を使うにはどうすればよいですか？](#how-do-i-use-brave-for-browser-control)
* [リモート Gateway とノード](#remote-gateways-nodes)
  * [コマンドは Telegram、Gateway、ノード間でどのように伝わりますか？](#how-do-commands-propagate-between-telegram-the-gateway-and-nodes)
  * [Gateway がリモートにホストされている場合でも、エージェントが自分のコンピュータにアクセスできるようにするにはどうすればよいですか？](#how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely)
  * [Tailscale は接続されていますが、応答がありません。どうすればよいですか？](#tailscale-is-connected-but-i-get-no-replies-what-now)
  * [2 つの OpenClaw インスタンス同士を通信させることはできますか（ローカル + VPS）？](#can-two-openclaw-instances-talk-to-each-other-local-vps)
  * [複数のエージェント用に VPS を分ける必要がありますか](#do-i-need-separate-vpses-for-multiple-agents)
  * [VPS からの SSH の代わりに、自分のノート PC 上のノードを使う利点はありますか？](#is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps)
  * [ノードは Gateway サービスを実行するものですか？](#do-nodes-run-a-gateway-service)
  * [設定を適用するための api / RPC の手段はありますか？](#is-there-an-api-rpc-way-to-apply-config)
  * [初回インストール向けの、最低限かつ「まともな」構成は何ですか？](#whats-a-minimal-sane-config-for-a-first-install)
  * [VPS 上で Tailscale をセットアップし、Mac から接続するにはどうすればよいですか？](#how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac)
  * [Mac ノードをリモート Gateway（Tailscale Serve）に接続するにはどうすればよいですか？](#how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve)
  * [2 台目のノート PC にインストールすべきですか、それともノードを追加するだけでよいですか？](#should-i-install-on-a-second-laptop-or-just-add-a-node)
* [環境変数と.envの読み込み](#env-vars-and-env-loading)
  * [OpenClaw は環境変数をどのように読み込みますか？](#how-does-openclaw-load-environment-variables)
  * [「サービス経由で Gateway を起動したら、環境変数が消えてしまいました。」どうすればいいですか？](#i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now)
  * [`COPILOT_GITHUB_TOKEN` を設定したのに、モデルのステータスに「Shell env: off」と表示されます。なぜですか？](#i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why)
* [セッションと複数のチャット](#sessions-multiple-chats)
  * [新しい会話を一から始めるには？](#how-do-i-start-a-fresh-conversation)
  * [`/new` を一度も送信しなかった場合、セッションは自動的にリセットされますか？](#do-sessions-reset-automatically-if-i-never-send-new)
  * [OpenClaw インスタンスを、1 つを CEO、他を多数のエージェントとするチーム構成にする方法はありますか？](#is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents)
  * [なぜタスクの途中でコンテキストが切り捨てられたのですか？どう防げばよいですか？](#why-did-context-get-truncated-midtask-how-do-i-prevent-it)
  * [OpenClaw を完全にリセットして、インストール状態だけ維持するには？](#how-do-i-completely-reset-openclaw-but-keep-it-installed)
  * [「context too large」エラーが出ます ― どうリセットまたは圧縮すればよいですか？](#im-getting-context-too-large-errors-how-do-i-reset-or-compact)
  * [「LLM request rejected: messages.N.content.X.tool&#95;use.input: Field required」が表示されるのはなぜですか？](#why-am-i-seeing-llm-request-rejected-messagesncontentxtooluseinput-field-required)
  * [なぜ 30 分ごとにハートビートメッセージが届くのですか？](#why-am-i-getting-heartbeat-messages-every-30-minutes)
  * [WhatsApp グループに「ボットアカウント」を追加する必要がありますか？](#do-i-need-to-add-a-bot-account-to-a-whatsapp-group)
  * [WhatsApp グループの JID を取得する方法は？](#how-do-i-get-the-jid-of-a-whatsapp-group)
  * [なぜ OpenClaw はグループで返信しないのですか？](#why-doesnt-openclaw-reply-in-a-group)
  * [グループやスレッドは DM とコンテキストを共有しますか？](#do-groupsthreads-share-context-with-dms)
  * [いくつのワークスペースとエージェント群を作成できますか？](#how-many-workspaces-and-agents-can-i-create)
  * [複数のボットやチャットを同時に（Slack で）動かすことはできますか？その場合どのように設定すべきですか？](#can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up)
* [モデル: デフォルト、選択、エイリアス、切り替え](#models-defaults-selection-aliases-switching)
  * [「default model」とは何ですか？](#what-is-the-default-model)
  * [どのモデルを推奨しますか？](#what-model-do-you-recommend)
  * [設定を失わずにモデルを切り替えるにはどうすればよいですか？](#how-do-i-switch-models-without-wiping-my-config)
  * [自己ホスト型モデル（llama.cpp、vLLM、Ollama）を利用できますか？](#can-i-use-selfhosted-models-llamacpp-vllm-ollama)
  * [OpenClaw、Flawd、Krill はどのモデルを使っていますか？](#what-do-openclaw-flawd-and-krill-use-for-models)
  * [再起動せずにその場でモデルを切り替えるにはどうすればよいですか？](#how-do-i-switch-models-on-the-fly-without-restarting)
  * [日常タスクには GPT 5.2、コーディングには Codex 5.2 を使うことはできますか](#can-i-use-gpt-52-for-daily-tasks-and-codex-52-for-coding)
  * [「Model … is not allowed」と表示されて、返信が来ないのはなぜですか？](#why-do-i-see-model-is-not-allowed-and-then-no-reply)
  * [「Unknown model: minimax/MiniMax-M2.1」と表示されるのはなぜですか？](#why-do-i-see-unknown-model-minimaxminimaxm21)
  * [MiniMax をデフォルトにして、複雑なタスクには OpenAI を使うことはできますか？](#can-i-use-minimax-as-my-default-and-openai-for-complex-tasks)
  * [opus / sonnet / gpt は組み込みショートカットですか？](#are-opus-sonnet-gpt-builtin-shortcuts)
  * [モデルのショートカット（エイリアス）を定義／上書きするにはどうすればよいですか？](#how-do-i-defineoverride-model-shortcuts-aliases)
  * [OpenRouter や Z.AI など、他のプロバイダーのモデルを追加するにはどうすればよいですか？](#how-do-i-add-models-from-other-providers-like-openrouter-or-zai)
* [モデルのフェイルオーバーと「すべてのモデルが失敗しました」エラー](#model-failover-and-all-models-failed)
  * [フェイルオーバーはどのように機能しますか？](#how-does-failover-work)
  * [このエラーは何を意味しますか？](#what-does-this-error-mean)
  * [`No credentials found for profile "anthropic:default"` の修正用チェックリスト](#fix-checklist-for-no-credentials-found-for-profile-anthropicdefault)
  * [なぜ Google Gemini も試して失敗したのですか？](#why-did-it-also-try-google-gemini-and-fail)
* [認証プロファイルとは何か、その管理方法](#auth-profiles-what-they-are-and-how-to-manage-them)
  * [auth プロファイルとは何ですか？](#what-is-an-auth-profile)
  * [代表的なプロファイル ID にはどんなものがありますか？](#what-are-typical-profile-ids)
  * [どの auth プロファイルが優先して使用されるかを制御できますか？](#can-i-control-which-auth-profile-is-tried-first)
  * [OAuth と API キー: 何が違いますか？](#oauth-vs-api-key-whats-the-difference)
* [Gateway：ポート、「すでに実行中」エラー、リモートモード](#gateway-ports-already-running-and-remote-mode)
  * [Gateway はどのポートを使用しますか？](#what-port-does-the-gateway-use)
  * [`openclaw gateway status` が `Runtime: running` だが `RPC probe: failed` と表示されるのはなぜですか？](#why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed)
  * [`openclaw gateway status` で `Config (cli)` と `Config (service)` が異なって表示されるのはなぜですか？](#why-does-openclaw-gateway-status-show-config-cli-and-config-service-different)
  * [「another gateway instance is already listening」とはどういう意味ですか？](#what-does-another-gateway-instance-is-already-listening-mean)
  * [OpenClaw をリモートモード（クライアントが別の場所の Gateway に接続する）で実行するにはどうすればよいですか？](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere)
  * [Control UI が「unauthorized」と表示する（または再接続を繰り返す）場合はどうすればよいですか？](#the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now)
  * [`gateway.bind: "tailnet"` を設定したがバインドできない／何も待ち受けていないのはなぜですか？](#i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens)
  * [同じホスト上で複数の Gateway を実行できますか？](#can-i-run-multiple-gateways-on-the-same-host)
  * [「invalid handshake」／コード 1008 とはどういう意味ですか？](#what-does-invalid-handshake-code-1008-mean)
* [ログとデバッグ](#logging-and-debugging)
  * [ログはどこにありますか？](#where-are-logs)
  * [Gateway サービスを開始・停止・再起動するにはどうすればよいですか？](#how-do-i-startstoprestart-the-gateway-service)
  * [Windows でターミナルを閉じてしまいました — OpenClaw を再起動するには？](#i-closed-my-terminal-on-windows-how-do-i-restart-openclaw)
  * [Gateway は起動していますが応答が届きません。何を確認すべきですか？](#the-gateway-is-up-but-replies-never-arrive-what-should-i-check)
  * [&quot;Disconnected from gateway: no reason&quot; — 次に何をすればよいですか？](#disconnected-from-gateway-no-reason-what-now)
  * [Telegram の setMyCommands がネットワークエラーで失敗します。何を確認すべきですか？](#telegram-setmycommands-fails-with-network-errors-what-should-i-check)
  * [TUI に出力が表示されません。何を確認すべきですか？](#tui-shows-no-output-what-should-i-check)
  * [Gateway を完全に停止してから再起動するにはどうすればよいですか？](#how-do-i-completely-stop-then-start-the-gateway)
  * [ELI5: `openclaw gateway restart` と `openclaw gateway` の違いは？](#eli5-openclaw-gateway-restart-vs-openclaw-gateway)
  * [障害発生時に詳細を確認する最も手早い方法は？](#whats-the-fastest-way-to-get-more-details-when-something-fails)
* [メディアと添付ファイル](#media-attachments)
  * [スキルで画像やPDFを生成したのに、何も送信されない](#my-skill-generated-an-imagepdf-but-nothing-was-sent)
* [セキュリティとアクセス制御](#security-and-access-control)
  * [OpenClaw を外部からの DM に公開しても安全ですか？](#is-it-safe-to-expose-openclaw-to-inbound-dms)
  * [プロンプトインジェクションは公開ボットだけの懸念事項ですか？](#is-prompt-injection-only-a-concern-for-public-bots)
  * [ボット専用のメールアドレス、GitHub アカウント、電話番号を用意すべきですか？](#should-my-bot-have-its-own-email-github-account-or-phone-number)
  * [テキストメッセージの扱いをボットに一任しても安全ですか？](#can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe)
  * [個人アシスタント用途にはより安価なモデルを使ってもよいですか？](#can-i-use-cheaper-models-for-personal-assistant-tasks)
  * [Telegram で `/start` を実行しましたが、ペアリングコードが表示されませんでした](#i-ran-start-in-telegram-but-didnt-get-a-pairing-code)
  * [WhatsApp: ボットは自分の連絡先にメッセージを送りますか？ペアリングはどのように動作しますか？](#whatsapp-will-it-message-my-contacts-how-does-pairing-work)
* [チャットコマンド、タスクの中断、「止まらない」とき](#chat-commands-aborting-tasks-and-it-wont-stop)
  * [チャットに内部システムメッセージが表示されないようにするには？](#how-do-i-stop-internal-system-messages-from-showing-in-chat)
  * [実行中のタスクを停止・キャンセルするには？](#how-do-i-stopcancel-a-running-task)
  * [Telegram から Discord にメッセージを送信するには？（&quot;Cross-context messaging denied&quot; エラー）](#how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied)
  * [なぜボットがメッセージを立て続けに送るとそれらを「無視」しているように感じるのですか？](#why-does-it-feel-like-the-bot-ignores-rapidfire-messages)

<div id="first-60-seconds-if-somethings-broken">
  ## 何かが壊れているときの最初の60秒
</div>

1. **クイックステータス（最初の確認）**
   ```bash
   openclaw status
   ```
   高速なローカルサマリー: OS とアップデート状況、gateway/サービスの到達性、エージェント/セッション、プロバイダー設定とランタイムの問題（gateway に到達可能な場合）。

2. **コピーして貼り付けられるレポート（安全に共有可能）**
   ```bash
   openclaw status --all
   ```
   ログ末尾付きの読み取り専用診断（トークンはマスクされる）。

3. **デーモン + ポート状態**
   ```bash
   openclaw gateway status
   ```
   スーパーバイザーのランタイムと RPC 到達性、プローブ対象 URL、およびサービスが使用したと思われる設定を表示。

4. **詳細プローブ**
   ```bash
   openclaw status --deep
   ```
   gateway のヘルスチェックとプロバイダーへのプローブを実行（到達可能な gateway が必要）。[Health](/ja/gateway/health) を参照。

5. **最新ログの tail**
   ```bash
   openclaw logs --follow
   ```
   RPC が落ちている場合は、次を使用:
   ```bash
   tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
   ```
   ファイルログはサービスログとは別管理。[Logging](/ja/logging) および [Troubleshooting](/ja/gateway/troubleshooting) を参照。

6. **doctor の実行（修復）**
   ```bash
   openclaw doctor
   ```
   設定/状態の修復・マイグレーションとヘルスチェックの実行。[Doctor](/ja/gateway/doctor) を参照。

7. **Gateway スナップショット**
   ```bash
   openclaw health --json
   openclaw health --verbose   # エラー時に対象 URL と設定ファイルパスを表示
   ```
   実行中の gateway にフルスナップショットを要求（WS 限定）。[Health](/ja/gateway/health) を参照。

<div id="quick-start-and-first-run-setup">
  ## クイックスタートと初回起動時のセットアップ
</div>

<div id="im-stuck-whats-the-fastest-way-to-get-unstuck">
  ### 行き詰まりました 一番早く抜け出す方法は？
</div>

**あなたのマシンの状態を直接確認できる**ローカルのAIエージェントを使ってください。Discord で質問するより
はるかに効果的です。というのも、「行き詰まり」のほとんどはリモートのサポーターには確認できない
**ローカルの設定や環境の問題** だからです。

* **Claude Code**: https://www.anthropic.com/claude-code/
* **OpenAI Codex**: https://openai.com/codex/

これらのツールはリポジトリを読み取り、コマンドを実行し、ログを確認して、マシンレベルのセットアップ
(PATH、サービス、パーミッション、認証ファイル) の修正を手伝えます。hackable な (git) インストールを使って、
**ソース一式をフルチェックアウトしたもの** を渡してください。

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

これは OpenClaw を **git チェックアウトから** インストールします。そのためエージェントはコードとドキュメントを read でき、
あなたが実行している正確なバージョンについて推論できます。後からいつでも、
`--install-method git` なしでインストーラを再実行することで安定版に戻せます。

ヒント: エージェントに修正作業を（ステップバイステップで）**計画して監督させ**、自分では
必要なコマンドだけを実行してください。そうすることで変更範囲を小さく保てて、監査もしやすくなります。

もし実際のバグや修正を見つけたら、GitHub issue を作成するか PR を送ってください:
https://github.com/openclaw/openclaw/issues
https://github.com/openclaw/openclaw/pulls

まずは次のコマンドから始めてください（ヘルプを求めるときは出力を共有してください）:

```bash
openclaw status
openclaw models status
openclaw doctor
```

これらのコマンドでできること:

* `openclaw status`: Gateway とエージェントのヘルス状態と基本設定のスナップショットを素早く確認します。
* `openclaw models status`: プロバイダーの認証状態とモデルの利用可能性をチェックします。
* `openclaw doctor`: よくある設定や状態の問題を検証し、修復します。

その他の便利な CLI チェック: `openclaw status --all`, `openclaw logs --follow`,
`openclaw gateway status`, `openclaw health --verbose`.

簡易デバッグループ: [何かがおかしいときの最初の60秒](#first-60-seconds-if-somethings-broken)。
インストール関連ドキュメント: [Install](/ja/install), [Installer flags](/ja/install/installer), [Updating](/ja/install/updating)。

<div id="whats-the-recommended-way-to-install-and-set-up-openclaw">
  ### OpenClaw をインストールしてセットアップするには、どうするのが推奨ですか
</div>

このリポジトリでは、OpenClaw をソースコードから実行し、オンボーディングウィザードを使用することを推奨しています。

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
openclaw onboard --install-daemon
```

ウィザードは UI アセットも自動的にビルドできます。オンボーディング後は、通常 Gateway をポート **18789** で実行します。

ソースコードから（contributors/dev 向け）:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
pnpm ui:build # 初回実行時にUI依存関係を自動インストール
openclaw onboard
```

まだ OpenClaw をグローバルインストールしていない場合は、`pnpm openclaw onboard` を実行してください。

<div id="how-do-i-open-the-dashboard-after-onboarding">
  ### オンボーディング後にダッシュボードを開くにはどうすればよいですか
</div>

ウィザードは、オンボーディングの完了直後にトークン付きのダッシュボード URL でブラウザを自動的に開き、サマリーにもトークン入りの完全なリンクを表示します。そのタブは閉じずに開いたままにしておいてください。もしブラウザが起動しなかった場合は、同じマシン上で表示された URL をコピー＆ペーストして開いてください。トークンはホストローカルにとどまり、ブラウザからどこかに送信されたり取得されたりすることはありません。

<div id="how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote">
  ### localhost とリモートでダッシュボードトークンを認証する方法
</div>

**Localhost（同一マシン）の場合:**

* `http://127.0.0.1:18789/` を開きます。
* 認証を求められたら、`openclaw dashboard` を実行し、トークン付きリンク（`?token=...`）を使用します。
* トークンは `gateway.auth.token`（または `OPENCLAW_GATEWAY_TOKEN`）と同じ値で、初回読み込み後は UI によって保存されます。

**Localhost 以外の場合:**

* **Tailscale Serve**（推奨）: バインド先はループバックのままにして、`openclaw gateway --tailscale serve` を実行し、`https://<magicdns>/` を開きます。`gateway.auth.allowTailscale` が `true` の場合、アイデンティティヘッダーで認証が満たされるため、トークンは不要です。
* **Tailnet バインド**: `openclaw gateway --bind tailnet --token "<token>"` を実行し、`http://<tailscale-ip>:18789/` を開いて、ダッシュボード設定にトークンを貼り付けます。
* **SSH トンネル**: `ssh -N -L 18789:127.0.0.1:18789 user@host` を実行し、その後 `openclaw dashboard` から `http://127.0.0.1:18789/?token=...` を開きます。

バインドモードと認証の詳細は [Dashboard](/ja/web/dashboard) および [Web surfaces](/ja/web) を参照してください。

<div id="what-runtime-do-i-need">
  ### どのランタイムが必要ですか
</div>

Node **&gt;= 22** が必須です。`pnpm` の利用を推奨します。Bun は Gateway での使用は **非推奨** です。

<div id="does-it-run-on-raspberry-pi">
  ### Raspberry Pi で動作しますか？
</div>

はい。Gateway は軽量です。ドキュメントでは、個人利用であれば **512MB〜1GB RAM**、**1コア**、およそ **500MB** の
ディスク容量で十分と記載されており、**Raspberry Pi 4 で動作可能**であることも明記されています。

ログやメディア、その他のサービス用に余裕を持たせたい場合は **2GB を推奨**しますが、
これは厳密な最低要件ではありません。

Tip: 小さな Pi や VPS 上で Gateway をホストし、ノート PC やスマートフォンを **ノード** としてペアリングして、
ローカルの画面／カメラ／キャンバスやコマンドの実行に利用できます。[Nodes](/ja/nodes) を参照してください。

<div id="any-tips-for-raspberry-pi-installs">
  ### Raspberry Pi へのインストールのコツはありますか
</div>

要するに「動くことは動くが、多少の荒さは覚悟しておいてください」という話です。

* **64-bit** OS を使い、Node はバージョン 22 以上にしてください。
* ログを確認しやすく、素早くアップデートできるように、**hackable (git) インストール** を優先してください。
* 最初はチャネルやスキルなしで起動し、その後に 1 つずつ追加してください。
* 妙なバイナリ周りの問題に遭遇した場合、たいていは **ARM 互換性** が原因です。

ドキュメント: [Linux](/ja/platforms/linux), [Install](/ja/install).

<div id="it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now">
  ### wake up my friend のオンボーディング画面で止まって孵化しません。どうすればいいですか？
</div>

その画面は、Gateway に到達できていて認証が済んでいることに依存します。TUI は最初の孵化時に
自動で &quot;Wake up, my friend!&quot; を送信します。その行が表示されているのに **返信がなく**、
トークンが 0 のままの場合、エージェントは一度も起動されていません。

1. Gateway を再起動してください:

```bash
openclaw gateway restart
```

2. ステータスと認証の確認：

```bash
openclaw status
openclaw models status
openclaw logs --follow
```

3. それでもハングする場合は、次を実行してください:

```bash
openclaw doctor
```

Gateway がリモート環境にある場合は、トンネル/Tailscale 接続が確立されており、UI で正しい Gateway が指定されていることを確認してください。[Remote access](/ja/gateway/remote) を参照してください。

<div id="can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding">
  ### オンボーディングをやり直さずに、セットアップを新しい Mac mini に移行できますか
</div>

はい。**ステートディレクトリ** と **ワークスペース** をコピーしてから、Doctor を 1 回実行してください。これにより、**両方** の場所をコピーするかぎり、あなたのボットは（メモリ、セッション履歴、認証情報、チャネルの状態を含めて）「まったく同じ状態」のままになります。

1. 新しいマシンに OpenClaw をインストールします。
2. 古いマシンから `$OPENCLAW_STATE_DIR`（デフォルト: `~/.openclaw`）をコピーします。
3. ワークスペース（デフォルト: `~/.openclaw/workspace`）をコピーします。
4. `openclaw doctor` を実行し、Gateway サービスを再起動します。

これで、設定、認証プロファイル、WhatsApp の認証情報、セッション、およびメモリが保持されます。リモートモードの場合、Gateway ホスト側がセッションストアとワークスペースを保持していることを忘れないでください。

**重要:** ワークスペースだけを GitHub に commit/push している場合、バックアップされるのは **メモリ + ブートストラップファイル** だけであり、**セッション履歴や認証情報は含まれません**。それらは `~/.openclaw/`（例: `~/.openclaw/agents/<agentId>/sessions/`）の下に保存されています。

関連: [Migrating](/ja/install/migrating), [Where things live on disk](/ja/help/faq#where-does-openclaw-store-its-data),
[Agent workspace](/ja/concepts/agent-workspace), [Doctor](/ja/gateway/doctor),
[Remote mode](/ja/gateway/remote).

<div id="where-do-i-see-whats-new-in-the-latest-version">
  ### 最新バージョンの更新内容はどこで確認できますか
</div>

GitHub の changelog を確認してください:\
https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md

最新のエントリは一番上にあります。一番上のセクションが **Unreleased** と表示されている場合、その次の「日付付きセクション」が最新のリリース済みバージョンです。エントリは **Highlights**、**Changes**、**Fixes**（必要に応じて docs やその他のセクション）ごとにグループ化されています。

<div id="i-cant-access-docsopenclawai-ssl-error-what-now">
  ### docs.openclaw.ai にアクセスできず SSL エラーが出る 場合はどうすればよいですか
</div>

一部の Comcast/Xfinity 回線では、Xfinity Advanced Security によって
`docs.openclaw.ai` が誤ってブロックされることがあります。Xfinity
Advanced Security を無効化するか、`docs.openclaw.ai` を許可リストに追加してから再試行してください。詳細はこちらを参照してください:
[トラブルシューティング](/ja/help/troubleshooting#docsopenclawai-shows-an-ssl-error-comcastxfinity)。
こちらから問題を報告してブロック解除にご協力ください: https://spa.xfinity.com/check&#95;url&#95;status.

それでもサイトにアクセスできない場合、ドキュメントは GitHub 上にもミラーされています:
https://github.com/openclaw/openclaw/tree/main/docs

<div id="whats-the-difference-between-stable-and-beta">
  ### 安定版とベータ版はどう違いますか
</div>

**Stable** と **beta** は、別々のコードラインではなく **npm の dist‑tag** です:

* `latest` = 安定版 (stable)
* `beta` = テスト用の早期ビルド

私たちはまずビルドを **beta** としてリリースしてテストし、ビルドが十分に安定したら **同じバージョンを `latest` に昇格させます**。そのため、beta と stable が **同じバージョン** を指している場合があります。

変更内容はこちらを参照してください:\
https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md

<div id="how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev">
  ### ベータ版はどうやってインストールしますか？ベータ版と dev 版の違いは何ですか？
</div>

**Beta** は npm の dist‑tag `beta`（`latest` と一致する場合があります）です。
**Dev** は `main`（git）ブランチの先頭の移動する最新コミットであり、公開されると npm の dist‑tag `dev` が使われます。

ワンライナー（macOS/Linux）:

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.bot/install.sh | bash -s -- --beta
```

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.bot/install.sh | bash -s -- --install-method git
```

Windows 用インストーラー (PowerShell):
https://openclaw.ai/install.ps1

詳細については、[開発チャネル](/ja/install/development-channels) および [インストーラーのフラグ](/ja/install/installer) を参照してください。

<div id="how-long-does-install-and-onboarding-usually-take">
  ### インストールとオンボーディングには通常どのくらい時間がかかりますか
</div>

おおよその目安は次のとおりです:

* **インストール:** 2〜5分
* **オンボーディング:** 設定するチャネルやモデルの数にもよりますが、5〜15分

処理が止まったように見える場合は、[インストーラーが止まった](/ja/help/faq#installer-stuck-how-do-i-get-more-feedback) と、[抜け出せないとき](/ja/help/faq#im-stuck--whats-the-fastest-way-to-get-unstuck) に記載されている高速デバッグループを参照してください。

<div id="how-do-i-try-the-latest-bits">
  ### 最新のビルドを試すにはどうすればよいですか
</div>

方法は 2 つあります:

1. **Dev チャネル (git checkout):**

```bash
openclaw update --channel dev
```

これは `main` ブランチに切り替え、ソースリポジトリから最新の状態に更新します。

2. **カスタマイズしやすいインストール（インストーラーサイトから）：**

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

これで、ローカルに編集可能なリポジトリが用意され、git を使って更新できるようになります。

クリーンなクローンを手動で行いたい場合は、次を使用してください:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
```

ドキュメント: [アップデート](/ja/cli/update), [開発チャネル](/ja/install/development-channels), [インストール](/ja/install).

<div id="installer-stuck-how-do-i-get-more-feedback">
  ### インストーラーが固まったように見えます もっと詳細な情報を得るにはどうすればよいですか
</div>

インストーラーを **詳細ログ出力** を有効にして再実行します：

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --verbose
```

ベータ版を詳細ログ付きでインストール:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --beta --verbose
```

ハックしやすい（git）インストールの場合：

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git --verbose
```

その他のオプションについては、[インストーラーのフラグ](/ja/install/installer) を参照してください。

<div id="windows-install-says-git-not-found-or-openclaw-not-recognized">
  ### Windows のインストールで git が見つからない、または openclaw が認識されないと表示される
</div>

よくある Windows の問題が 2 つあります：

**1) npm error spawn git / git not found**

* **Git for Windows** をインストールし、`git` が PATH に通っていることを確認します。
* PowerShell を一度閉じて開き直し、インストーラーを再実行します。

**2) インストール後に openclaw が認識されない**

* npm のグローバル bin フォルダが PATH に含まれていません。
* 次のコマンドでパスを確認します：
  ```powershell
  npm config get prefix
  ```
* `<prefix>\\bin` が PATH に含まれていることを確認します（ほとんどの環境では `%AppData%\\npm` です）。
* PATH を更新した後、PowerShell を閉じてからもう一度開き直してください。

Windows でできるだけスムーズにセットアップしたい場合は、ネイティブの Windows ではなく **WSL2** の使用を検討してください。
ドキュメント: [Windows](/ja/platforms/windows).

<div id="the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer">
  ### ドキュメントを読んでも疑問が解決しません。どうすればもっとよい回答を得られますか
</div>

**hackable (git) install** を使ってソースコードとドキュメント一式をローカルに用意してから、
そのフォルダ内でボット（または Claude/Codex）に質問すると、
リポジトリを read したうえで、より正確に回答できるようになります。

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

詳細については、[インストール](/ja/install) および [インストーラーのフラグ](/ja/install/installer) を参照してください。

<div id="how-do-i-install-openclaw-on-linux">
  ### Linux に OpenClaw をインストールするにはどうすればよいですか
</div>

結論: Linux ガイドに従い、その後オンボーディングウィザードを実行します。

* Linux 向けクイック手順 + サービスインストール: [Linux](/ja/platforms/linux)
* 詳細な手順: [Getting Started](/ja/start/getting-started)
* インストーラー + アップデート: [Install &amp; updates](/ja/install/updating)

<div id="how-do-i-install-openclaw-on-a-vps">
  ### VPS に OpenClaw をインストールするにはどうすればよいですか
</div>

どの Linux VPS でも動作します。対象のサーバーにインストールし、その後 SSH/Tailscale 経由で Gateway に接続します。

インストールガイド: [exe.dev](/ja/platforms/exe-dev)、[Hetzner](/ja/platforms/hetzner)、[Fly.io](/ja/platforms/fly)\
リモートアクセス: [Gateway remote](/ja/gateway/remote)

<div id="where-are-the-cloudvps-install-guides">
  ### cloudVPS のインストールガイドはどこにありますか
</div>

一般的なプロバイダー向けの **ホスティングハブ** を用意しています。1つ選んでガイドに従ってください:

* [VPS hosting](/ja/vps) (すべてのプロバイダーを一箇所に集約)
* [Fly.io](/ja/platforms/fly)
* [Hetzner](/ja/platforms/hetzner)
* [exe.dev](/ja/platforms/exe-dev)

クラウドでの仕組み: **Gateway はサーバー上で動作**し、あなたは
Control UI（または Tailscale/SSH）経由でノートPC/スマートフォンからアクセスします。
状態とワークスペースはサーバー上に存在するため、そのホストを信頼できる唯一の情報源として扱い、必ずバックアップを取得してください。

そのクラウド上の Gateway に **ノード**（Mac/iOS/Android/ヘッドレス）をペアリングして、
ローカルの画面/カメラ/キャンバスへアクセスしたり、Gateway はクラウドに置いたまま
ノートPC上でコマンドを実行したりできます。

ハブ: [Platforms](/ja/platforms)。リモートアクセス: [Gateway remote](/ja/gateway/remote)。
ノード: [Nodes](/ja/nodes)、[Nodes CLI](/ja/cli/nodes)。

<div id="can-i-ask-openclaw-to-update-itself">
  ### OpenClaw 自身のアップデートを OpenClaw に任せられますか
</div>

要約: **可能だが、推奨はしない**。アップデート処理により Gateway が再起動されることがあり
（その場合、アクティブなセッションが失われます）、クリーンな git チェックアウトが
必要になることや、確認プロンプトが表示されることもあります。より安全なのは、
オペレーターとしてシェルからアップデートを実行することです。

CLI を使用します:

```bash
openclaw update
openclaw update status
openclaw update --channel stable|beta|dev
openclaw update --tag <dist-tag|version>
openclaw update --no-restart
```

やむを得ずエージェントから自動化する必要がある場合は:

```bash
openclaw update --yes --no-restart
openclaw gateway restart
```

関連ドキュメント: [Update](/ja/cli/update)、[Updating](/ja/install/updating)。

<div id="what-does-the-onboarding-wizard-actually-do">
  ### オンボーディングウィザードは実際に何をしてくれるのか
</div>

`openclaw onboard` は推奨されるセットアップ手順です。**ローカルモード** では、次の内容を順番に案内します:

* **モデル／認証のセットアップ**（Claude サブスクリプションには Anthropic の **setup-token** を推奨、OpenAI Codex の OAuth をサポート、API キーは任意、LM Studio のローカルモデルもサポート）
* **ワークスペース** の場所とブートストラップ用ファイル
* **Gateway の設定**（bind/port/auth/Tailscale）
* **プロバイダー**（WhatsApp、Telegram、Discord、Mattermost（プラグイン）、Signal、iMessage）
* **デーモンのインストール**（macOS では LaunchAgent、Linux/WSL2 では systemd user unit）
* **ヘルスチェック** と **スキル** の選択

また、設定したモデルが不明な場合や、認証情報が設定されていない／不足している場合には警告も行います。

<div id="do-i-need-a-claude-or-openai-subscription-to-run-this">
  ### これを実行するのに Claude や OpenAI のサブスクリプションは必要ですか
</div>

いいえ。OpenClaw は **API keys**（Anthropic / OpenAI / その他）を使って実行することも、
**ローカル専用モデル** だけで実行してデータを端末内にとどめておくこともできます。サブスクリプション（Claude
Pro/Max や OpenAI Codex）は、それらのプロバイダーに対して認証するための任意の手段にすぎません。

ドキュメント: [Anthropic](/ja/providers/anthropic)、[OpenAI](/ja/providers/openai)、
[ローカルモデル](/ja/gateway/local-models)、[モデル](/ja/concepts/models)。

<div id="can-i-use-claude-max-subscription-without-an-api-key">
  ### API キーなしで Claude Max サブスクリプションを利用できますか
</div>

はい、できます。API キーの代わりに **setup-token** を使って認証できます。これはサブスクリプション向けのパスです。

Claude Pro/Max サブスクリプションには **API キーは含まれていないため**、サブスクリプションアカウントではこの方法が正しいやり方になります。重要: この利用方法が、サブスクリプションポリシーおよび利用規約の範囲内で許可されているかどうか、Anthropic に確認してください。最も明確で公式にサポートされたパスを取りたい場合は、Anthropic の API キーを使用してください。

<div id="how-does-anthropic-setuptoken-auth-work">
  ### Anthropic setuptoken 認証はどのように機能しますか
</div>

`claude setup-token` は Claude Code CLI を通じて**トークン文字列**を生成します（Web コンソールからは利用できません）。これは **任意のマシン上で**実行できます。ウィザードでは **Anthropic token (paste setup-token)** を選択するか、`openclaw models auth paste-token --provider anthropic` を使ってトークンを貼り付けてください。トークンは **anthropic** プロバイダー用の認証プロファイルとして保存され、API キーのように使用されます（自動更新はありません）。詳細: [OAuth](/ja/concepts/oauth)。

<div id="where-do-i-find-an-anthropic-setuptoken">
  ### Anthropic のセットアップトークンはどこで見つかりますか
</div>

それは Anthropic Console には **ありません**。セットアップトークンは、**どのマシン上でも** **Claude Code CLI** によって生成されます。

```bash
claude setup-token
```

出力されたトークンをコピーし、ウィザードで **Anthropic token (paste setup-token)** を選択します。Gateway ホスト上で実行したい場合は、`openclaw models auth setup-token --provider anthropic` を使用します。別のホストで `claude setup-token` を実行した場合は、Gateway ホスト上で `openclaw models auth paste-token --provider anthropic` を実行し、そのトークンを貼り付けてください。[Anthropic](/ja/providers/anthropic) も参照してください。

<div id="do-you-support-claude-subscription-auth-claude-promax">
  ### Claude サブスクリプション認証（Claude Pro/Max）には対応していますか
</div>

はい、**setup-token** 経由で対応しています。OpenClaw はすでに Claude Code CLI の OAuth トークンを再利用しません。setup-token か Anthropic の API キーを使用してください。トークンは任意の場所で生成し、Gateway を動かしているホスト上に貼り付けてください。詳細は [Anthropic](/ja/providers/anthropic) および [OAuth](/ja/concepts/oauth) を参照してください。

注意: Claude のサブスクリプションによるアクセスは Anthropic の利用規約に従います。本番環境やマルチユーザーのワークロードでは、通常は API キーを使用する方が安全です。

<div id="why-am-i-seeing-http-429-ratelimiterror-from-anthropic">
  ### なぜ Anthropic から HTTP 429 ratelimiterror が返されるのですか
</div>

これは、現在の時間ウィンドウにおける **Anthropic のクォータ／レート制限** を使い切っていることを意味します。**Claude サブスクリプション**（setup‑token または Claude Code OAuth）を利用している場合は、そのウィンドウがリセットされるまで待つか、プランをアップグレードしてください。**Anthropic API key** を利用している場合は、Anthropic Console で利用状況／課金状況を確認し、必要に応じて制限を引き上げてください。

ヒント: **フォールバックモデル** を設定しておくと、あるプロバイダーでレート制限がかかっている間も OpenClaw が応答を継続できます。詳しくは [Models](/ja/cli/models) と [OAuth](/ja/concepts/oauth) を参照してください。

<div id="is-aws-bedrock-supported">
  ### AWS Bedrock はサポートされていますか
</div>

はい。pi‑ai の **Amazon Bedrock (Converse)** プロバイダー経由で、**手動設定** で利用できます。Gateway ホスト上で AWS の認証情報とリージョンを設定し、models 設定に Bedrock プロバイダーのエントリを追加する必要があります。[Amazon Bedrock](/ja/bedrock) および [Model providers](/ja/providers/models) を参照してください。マネージドなキー管理フローを好む場合は、Bedrock の前段に OpenAI 互換プロキシを置く構成も、依然として有効な選択肢です。

<div id="how-does-codex-auth-work">
  ### Codex 認証はどのように動作しますか？
</div>

OpenClaw は OAuth（ChatGPT サインイン）経由で **OpenAI Code（Codex）** をサポートしています。ウィザードは OAuth フローを実行し、必要に応じてデフォルトのモデルを `openai-codex/gpt-5.2` に設定します。詳しくは [モデル プロバイダー](/ja/concepts/model-providers) および [ウィザード](/ja/start/wizard) を参照してください。

<div id="do-you-support-openai-subscription-auth-codex-oauth">
  ### OpenAI サブスクリプション認証 Codex OAuth に対応していますか
</div>

はい。OpenClaw は **OpenAI Code（Codex）サブスクリプション用 OAuth** を完全にサポートしています。オンボーディングウィザードで OAuth フローを実行できます。

[OAuth](/ja/concepts/oauth)、[モデルプロバイダー](/ja/concepts/model-providers)、[ウィザード](/ja/start/wizard) を参照してください。

<div id="how-do-i-set-up-gemini-cli-oauth">
  ### Gemini CLI OAuth を設定するにはどうすればよいですか
</div>

Gemini CLI は、`openclaw.json` 内のクライアント ID やシークレットを使うのではなく、**プラグイン認証フロー** を使用します。

手順：

1. プラグインを有効化する: `openclaw plugins enable google-gemini-cli-auth`
2. ログインする: `openclaw models auth login --provider google-gemini-cli --set-default`

これにより、OAuth トークンは Gateway ホスト上の認証プロファイルに保存されます。詳細: [モデルプロバイダー](/ja/concepts/model-providers)。

<div id="is-a-local-model-ok-for-casual-chats">
  ### 気軽なチャットにローカルモデルを使っても問題ないですか
</div>

通常は推奨しません。OpenClaw には大きなコンテキスト長と強力な安全対策が必要であり、コンテキスト枠が小さいモデルだと入力が切り捨てられたり、情報が漏洩したりします。どうしても使う場合は、ローカル（LM Studio）で動かせる **最大** の MiniMax M2.1 ビルドを使用し、[/gateway/local-models](/ja/gateway/local-models) を参照してください。小さい／量子化されたモデルはプロンプトインジェクションのリスクを高めます。[Security](/ja/gateway/security) も参照してください。

<div id="how-do-i-keep-hosted-model-traffic-in-a-specific-region">
  ### ホスト型モデルのトラフィックを特定のリージョン内に保つにはどうすればよいですか
</div>

リージョンに固定されたエンドポイントを選択してください。OpenRouter は MiniMax、Kimi、GLM 向けに米国内ホストのオプションを提供しているので、データをリージョン内に保持したい場合は米国ホスト版を選びます。`models.mode: "merge"` を使えば、選択したリージョン固定のプロバイダーを優先しつつ、Anthropic や OpenAI も併記してフォールバックとして利用可能な状態を維持できます。

<div id="do-i-have-to-buy-a-mac-mini-to-install-this">
  ### これをインストールするのに Mac mini を買う必要がありますか
</div>

いいえ。OpenClaw は macOS か Linux（Windows は WSL2 経由）で動作します。Mac mini は必須ではありません。常時稼働用ホストとして購入する人もいますが、小さな VPS、自宅サーバー、Raspberry Pi クラスのマシンでも問題ありません。

Mac が必要になるのは、**macOS 専用ツールを使う場合だけ**です。iMessage の場合、Gateway は Linux 上で動作させたままにしつつ、任意の Mac で `imsg` を SSH 経由で実行し、`channels.imessage.cliPath` を SSH ラッパーに向ければ動作します。その他の macOS 専用ツールも使いたい場合は、Gateway を Mac 上で実行するか、macOS ノードをペアリングしてください。

ドキュメント: [iMessage](/ja/channels/imessage), [Nodes](/ja/nodes), [Mac リモートモード](/ja/platforms/mac/remote).

<div id="do-i-need-a-mac-mini-for-imessage-support">
  ### iMessage 対応には Mac mini が必要ですか
</div>

「メッセージ」アプリにサインインしている **何らかの macOS デバイス** が必要です。Mac mini である
**必要はありません** ― どの Mac でも構いません。OpenClaw の iMessage 連携は macOS
(BlueBubbles または `imsg`) 上で動作し、Gateway は別の場所で動かせます。

一般的な構成例:

* Gateway を Linux/VPS 上で動かし、`channels.imessage.cliPath` を、Mac 上で
  `imsg` を実行する SSH ラッパーに向ける。
* もっとも単純な単一マシン構成にしたい場合は、すべてを Mac 上で動かす。

ドキュメント: [iMessage](/ja/channels/imessage), [BlueBubbles](/ja/channels/bluebubbles),
[Mac remote mode](/ja/platforms/mac/remote).

<div id="if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro">
  ### OpenClaw を動かすために Mac mini を購入した場合、MacBook Pro と接続できますか
</div>

はい、可能です。**Mac mini 側で Gateway を実行**し、MacBook Pro を
**ノード**（コンパニオンデバイス）として接続できます。ノードは Gateway を実行せず、そのデバイスの画面／カメラ／キャンバス機能や `system.run` などの追加機能を提供します。

よくある構成例:

* Mac mini 上で Gateway を動かす（常時稼働）。
* MacBook Pro で macOS アプリまたはノードホストを動かし、Gateway とペアリングする。
* `openclaw nodes status` / `openclaw nodes list` を使って確認する。

ドキュメント: [Nodes](/ja/nodes), [Nodes CLI](/ja/cli/nodes).

<div id="can-i-use-bun">
  ### Bun を使えますか
</div>

Bun の利用は**推奨されません**。特に WhatsApp と Telegram でランタイムのバグが発生することがあります。
安定した Gateway には **Node** を使用してください。

それでも Bun を試したい場合は、WhatsApp/Telegram を使わない非本番環境の Gateway 上で行ってください。

<div id="telegram-what-goes-in-allowfrom">
  ### Telegram の allowFrom には何を設定するか
</div>

`channels.telegram.allowFrom` には、**人間ユーザー側の Telegram ユーザー ID**（数値、推奨）か `@username` を指定します。Bot のユーザー名ではありません。

より安全（サードパーティ Bot 不使用）の方法:

* 自分の Bot に DM を送り、`openclaw logs --follow` を実行して `from.id` を確認します。

公式 Bot API を使う方法:

* 自分の Bot に DM を送り、`https://api.telegram.org/bot<bot_token>/getUpdates` を呼び出して、`message.from.id` を確認します。

サードパーティ Bot（プライバシーはやや低い）を使う方法:

* `@userinfobot` または `@getidsbot` に DM を送ります。

[/channels/telegram](/ja/channels/telegram#access-control-dms--groups) を参照してください。

<div id="can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances">
  ### 複数人が 1 つの WhatsApp 番号を、別々の OpenClaw インスタンスで利用できますか
</div>

はい、**マルチエージェントルーティング**によって可能です。送信者ごとに、その人の WhatsApp **DM（ダイレクトメッセージ）**（peer `kind: "dm"`、送信者の E.164 形式番号 `+15551234567` など）を別々の `agentId` にバインドすると、各ユーザーは自分専用のワークスペースとセッションストアを持つことができます。返信はすべて**同じ WhatsApp アカウント**から送信され、DM のアクセス制御（`channels.whatsapp.dmPolicy` / `channels.whatsapp.allowFrom`）は WhatsApp アカウントごとにグローバル設定として適用されます。詳しくは [Multi-Agent Routing](/ja/concepts/multi-agent) および [WhatsApp](/ja/channels/whatsapp) を参照してください。

<div id="can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent">
  ### 高速チャット用エージェントと、コーディング用の Opus エージェントを同時に動かせますか
</div>

はい。マルチエージェントルーティングを使用します。各エージェントに専用のデフォルトモデルを割り当て、そのうえで着信ルート（プロバイダーアカウントや特定のピア）を各エージェントにバインドします。設定例は [Multi-Agent Routing](/ja/concepts/multi-agent) にあります。[Models](/ja/concepts/models) および [Configuration](/ja/gateway/configuration) も参照してください。

<div id="does-homebrew-work-on-linux">
  ### Homebrew は Linux でも動作しますか
</div>

はい。Homebrew は Linux（Linuxbrew）をサポートしています。クイックセットアップ:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.profile
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
brew install <formula>
```

OpenClaw を systemd 経由で実行する場合、非ログインシェルでも `brew` でインストールしたツールが利用できるように、サービスの PATH に `/home/linuxbrew/.linuxbrew/bin`（または使用している brew のプレフィックス）を含めてください。
最近のビルドでは、Linux の systemd サービスに対して一般的なユーザーの bin ディレクトリ（たとえば `~/.local/bin`、`~/.npm-global/bin`、`~/.local/share/pnpm`、`~/.bun/bin`）を PATH の先頭に追加し、設定されている場合は `PNPM_HOME`、`NPM_CONFIG_PREFIX`、`BUN_INSTALL`、`VOLTA_HOME`、`ASDF_DATA_DIR`、`NVM_DIR`、`FNM_DIR` を参照します。

<div id="whats-the-difference-between-the-hackable-git-install-and-npm-install">
  ### ハック可能な git インストールと npm インストールの違いは何ですか
</div>

* **ハック可能な (git) インストール:** ソース一式をチェックアウトし、自由に編集できます。コントリビューターに最適です。
  ローカルでビルドを実行し、コードやドキュメントにパッチを適用できます。
* **npm インストール:** グローバルな CLI インストールで、ローカルにリポジトリは作成されず、「とにかく動かしたい」場合に最適です。
  更新は npm の dist‑tags から配信されます。

ドキュメント: [はじめに](/ja/start/getting-started)、[アップデート](/ja/install/updating)。

<div id="can-i-switch-between-npm-and-git-installs-later">
  ### 後から npm と git のインストール方法を切り替えることはできますか
</div>

はい、可能です。もう一方のインストール方法を追加でインストールしてから Doctor を実行し、Gateway サービスが新しいエントリポイントを参照するようにします。
これは **データを削除しません**。変更されるのは OpenClaw のコードインストールだけです。状態
（`~/.openclaw`）とワークスペース（`~/.openclaw/workspace`）はそのまま残ります。

npm から git への切り替え:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
openclaw doctor
openclaw gateway restart
```

git から npm へ：

```bash
npm install -g openclaw@latest
openclaw doctor
openclaw gateway restart
```

Doctor は Gateway サービスのエントリポイントの不整合を検出し、現在のインストールに合わせてサービス設定を書き換えることを提案します（自動化では `--repair` を使用してください）。

バックアップのヒントについては、[バックアップ戦略](/ja/help/faq#whats-the-recommended-backup-strategy) を参照してください。

<div id="should-i-run-the-gateway-on-my-laptop-or-a-vps">
  ### Gateway はノートPC と VPS のどちらで動かすべきか
</div>

結論としては、**24/7 の安定稼働が必要なら VPS を使ってください**。導入の手間を最小限にしたくて、スリープや再起動が発生しても構わないなら、ローカル実行で問題ありません。

**ノートPC（ローカル Gateway）**

* **メリット:** サーバー費用が不要、ローカルファイルへ直接アクセス可能、ブラウザウィンドウをそのまま表示できる。
* **デメリット:** スリープ／ネットワーク切断で接続が落ちる、OS のアップデート／再起動で中断される、本体の電源を入れたままにしておく必要がある。

**VPS / クラウド**

* **メリット:** 常時稼働、安定したネットワーク、ノートPC のスリープ問題がない、継続運用が容易。
* **デメリット:** 多くはヘッドレス実行（スクリーンショットで確認）、ファイルはリモート越しでしか触れない、アップデート時は SSH でログインする必要がある。

**OpenClaw 固有の注意点:** WhatsApp / Telegram / Slack / Mattermost（プラグイン）/ Discord は、いずれも VPS 上から問題なく動作します。実質的なトレードオフは、**ヘッドレスブラウザ** にするか、目に見えるブラウザウィンドウを使うか、その程度です。[Browser](/ja/tools/browser) を参照してください。

**推奨デフォルト:** 以前に Gateway の切断が頻発したことがあるなら、VPS を推奨します。Mac を積極的に操作していて、ローカルファイルへのアクセスや、ブラウザを目視しながらの UI 自動化を行いたい場合は、ローカル実行が適しています。

<div id="how-important-is-it-to-run-openclaw-on-a-dedicated-machine">
  ### OpenClaw を専用マシンで動かす重要性はどれくらいありますか
</div>

必須ではありませんが、**信頼性と分離のために推奨です**。

* **専用ホスト (VPS / Mac mini / Pi):** 常時稼働しやすく、スリープや再起動による中断が少なく、権限まわりを整理しやすく、継続運用が簡単です。
* **共有ラップトップ / デスクトップ:** テストや日常的な利用にはまったく問題ありませんが、マシンのスリープやアップデート時に一時的な停止や中断が発生することを想定してください。

両方の良いところを取りたい場合は、Gateway を専用ホストで動かし、ラップトップをローカルの画面 / カメラ / exec ツール用の **ノード** としてペアリングする構成にするとよいでしょう。詳しくは [Nodes](/ja/nodes) を参照してください。
セキュリティに関するガイダンスは [Security](/ja/gateway/security) を参照してください。

<div id="what-are-the-minimum-vps-requirements-and-recommended-os">
  ### 最低限の VPS 要件と推奨 OS は何ですか
</div>

OpenClaw は軽量です。基本的な Gateway + チャットチャネル 1 つなら、次の構成で動作します。

* **最小構成:** 1 vCPU、1GB RAM、約 500MB のディスク。
* **推奨構成:** 1〜2 vCPU、2GB 以上の RAM（ログ、メディア、複数チャネルのための余裕）。ノードツールやブラウザ自動化はリソース消費が大きくなる場合があります。

OS: **Ubuntu LTS**（または任意のモダンな Debian/Ubuntu）を使用してください。Linux 向けのインストール手順はこの環境で最も検証されています。

ドキュメント: [Linux](/ja/platforms/linux)、[VPS ホスティング](/ja/vps)。

<div id="can-i-run-openclaw-in-a-vm-and-what-are-the-requirements">
  ### OpenClaw を VM 上で実行できますか？ また、要件は何ですか
</div>

はい。VM は VPS と同じように扱ってください。常時稼働し、外部から到達可能であり、Gateway と有効化する各チャネルに十分な RAM が必要です。

基本的な目安は次のとおりです:

* **絶対的な最小構成:** 1 vCPU、1GB RAM。
* **推奨:** 複数チャネル、ブラウザー自動化、メディア系ツールを実行する場合は 2GB RAM 以上。
* **OS:** Ubuntu LTS またはその他のモダンな Debian/Ubuntu 系ディストリビューション。

Windows を使用している場合、**WSL2 が最も簡単な VM 方式のセットアップ**であり、ツール群との互換性も最も高いです。[Windows](/ja/platforms/windows)、[VPS hosting](/ja/vps) を参照してください。
VM 上で macOS を実行している場合は、[macOS VM](/ja/platforms/macos-vm) を参照してください。

<div id="what-is-openclaw">
  ## OpenClaw とは？
</div>

<div id="what-is-openclaw-in-one-paragraph">
  ### 1段落で説明する OpenClaw とは何か
</div>

OpenClaw は、自分のデバイス上で自前運用するパーソナル AI アシスタントです。あなたがすでに使っているメッセージングサービス（WhatsApp、Telegram、Slack、Mattermost（プラグイン）、Discord、Google Chat、Signal、iMessage、WebChat）上で応答し、対応プラットフォームでは音声対話やライブ Canvas にも対応します。**Gateway** は常時稼働するコントロールプレーンであり、アシスタント本体が製品です。

<div id="whats-the-value-proposition">
  ### どんな価値があるのか
</div>

OpenClaw は「単なる Claude のラッパー」ではありません。**ローカルファーストのコントロールプレーン**として、
**あなた自身のハードウェア上**で高機能なアシスタントを動かし、すでに使っているチャットアプリからアクセスできるようにします。\
状態を保持するセッションやメモリ、ツールを備えつつ、ワークフローの主導権をホスト型 SaaS に渡す必要はありません。

主なポイント:

* **自分のデバイス・自分のデータ:** Gateway を好きな場所（Mac, Linux, VPS）で動かし、
  ワークスペースとセッション履歴をローカルに保持できます。
* **ただの Web サンドボックスではない「実運用のチャネル」:** WhatsApp/Telegram/Slack/Discord/Signal/iMessage などに加え、
  対応プラットフォーム上でのモバイル音声や Canvas に対応します。
* **モデル非依存:** Anthropic, OpenAI, MiniMax, OpenRouter などを利用でき、
  エージェントごとのルーティングやフェイルオーバーが可能です。
* **ローカルのみのオプション:** ローカルモデルを動かすことで、**すべてのデータをデバイス内に留める**構成も選べます。
* **マルチエージェントルーティング:** チャネル・アカウント・タスクごとにエージェントを分離し、
  それぞれに専用のワークスペースとデフォルト設定を持たせられます。
* **オープンソースでハックしやすい:** 中身を検証・拡張し、自前ホスティングできるため、
  ベンダーロックインを回避できます。

ドキュメント: [Gateway](/ja/gateway), [Channels](/ja/channels), [Multi‑agent](/ja/concepts/multi-agent),
[Memory](/ja/concepts/memory).

<div id="i-just-set-it-up-what-should-i-do-first">
  ### セットアップしたばかりですが、まず何をすればいいですか
</div>

最初のプロジェクト候補:

* Webサイトを作る（WordPress、Shopify、またはシンプルな静的サイト）。
* モバイルアプリのプロトタイプを作る（アウトライン、画面構成、API 計画）。
* ファイルとフォルダーを整理する（クリーンアップ、命名、タグ付け）。
* Gmail を連携して要約やフォローアップを自動化する。

大きなタスクも処理できますが、いくつかの段階に分割し、並行作業にはサブエージェントを使うと最も効果的に動作します。

<div id="what-are-the-top-five-everyday-use-cases-for-openclaw">
  ### OpenClaw の代表的な日常ユースケース上位 5 つは？
</div>

日常的な「ちょっとした勝ち」の例としては、次のようなものがあります:

* **パーソナルブリーフィング:** 自分にとって重要な受信トレイ、カレンダー、ニュースの要約。
* **リサーチと下書き作成:** メールやドキュメント向けの簡易な調査、要約、たたき台となる下書き作成。
* **リマインダーとフォローアップ:** cron やハートビートで駆動されるリマインダーやチェックリスト。
* **ブラウザ自動化:** フォーム入力、データ収集、定型的な Web 作業の繰り返し実行。
* **デバイスをまたいだ連携:** スマホからタスクを送信し、Gateway がサーバー上で実行し、その結果をチャットで受け取る。

<div id="can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas">
  ### OpenClaw は、SaaS 向けのリード獲得用アウトリーチ広告やブログ制作に役立ちますか
</div>

**リサーチ、リードの見込み度評価、ドラフト作成**には有用です。サイトをスキャンし、ショートリストを作成し、
見込み顧客を要約し、アウトリーチ文面や広告コピーのドラフトを書くことができます。

**アウトリーチ施策や広告配信**では、必ず人間がレビューするプロセスを維持してください。スパム行為を避け、各地域の法律と
プラットフォームのポリシーを順守し、送信前に必ず内容を確認してください。もっとも安全なパターンは、
OpenClaw にドラフトを書かせて、あなたがそれを承認する運用です。

ドキュメント: [Security](/ja/gateway/security).

<div id="what-are-the-advantages-vs-claude-code-for-web-development">
  ### Web 開発において Claude Code と比べた利点は何ですか
</div>

OpenClaw は **パーソナルアシスタント** かつコーディネーションレイヤーであり、IDE の代替ではありません。\
リポジトリ内で最速の直接コーディングループがほしい場合は、Claude Code や Codex を使ってください。\
永続的なメモリ、デバイス間をまたいだアクセス、ツールのオーケストレーションがほしい場合に OpenClaw を使ってください。

利点:

* セッションをまたいで保持される **永続メモリ + ワークスペース**
* **マルチプラットフォームアクセス**（WhatsApp、Telegram、TUI、WebChat）
* **ツールのオーケストレーション**（ブラウザ、ファイル、スケジューリング、フック）
* **常時稼働の Gateway**（VPS 上で動かし、どこからでも操作可能）
* ローカルのブラウザ／画面／カメラ／exec のための **ノード**

ショーケース: https://openclaw.ai/showcase

<div id="skills-and-automation">
  ## スキルと自動化
</div>

<div id="how-do-i-customize-skills-without-keeping-the-repo-dirty">
  ### リポジトリを汚さずにスキルをカスタマイズするにはどうすればよいですか
</div>

リポジトリのコピーを直接編集するのではなく、マネージド・オーバーライドを使用してください。変更内容は `~/.openclaw/skills/<name>/SKILL.md` に配置します（または `~/.openclaw/openclaw.json` の `skills.load.extraDirs` でフォルダを追加します）。優先順位は `<workspace>/skills` &gt; `~/.openclaw/skills` &gt; バンドル版 の順なので、git に触れることなくマネージド・オーバーライドが優先されます。リポジトリに含めるのは upstream に上げる価値のある変更だけにとどめ、それらは PR として送信してください。

<div id="can-i-load-skills-from-a-custom-folder">
  ### カスタムフォルダーからスキルを読み込めますか
</div>

はい。`~/.openclaw/openclaw.json` 内の `skills.load.extraDirs` で追加ディレクトリを指定できます（優先度は最も低くなります）。デフォルトの優先順位は `<workspace>/skills` → `~/.openclaw/skills` → バンドル済みのもの → `skills.load.extraDirs` の順のままです。`clawhub` はデフォルトで `./skills` にインストールし、OpenClaw はこれを `<workspace>/skills` として扱います。

<div id="how-can-i-use-different-models-for-different-tasks">
  ### タスクごとに異なるモデルを使うにはどうすればよいですか
</div>

現時点でサポートされているパターンは次のとおりです。

* **Cron ジョブ**: 個々のジョブごとに `model` の上書きを設定できます。
* **サブエージェント**: 異なるデフォルトモデルを持つ別のエージェント群にタスクをルーティングします。
* **オンデマンド切り替え**: いつでも `/model` を使って現在のセッションのモデルを切り替えられます。

[Cron ジョブ](/ja/automation/cron-jobs)、[マルチエージェントルーティング](/ja/concepts/multi-agent)、[Slash コマンド](/ja/tools/slash-commands) を参照してください。

<div id="the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that">
  ### Bot が重い処理中にフリーズします 負荷を分散するにはどうすればいいですか
</div>

長時間または並列のタスクには **サブエージェント** を使ってください。サブエージェントは独自のセッションで動作し、
要約を返しつつ、メインのチャットの応答性を保ちます。

Bot に「このタスク用にサブエージェントを起動して」と頼むか、`/subagents` を使います。
チャット内で `/status` を使うと、Gateway が現在何をしているか（ビジーかどうか）を確認できます。

トークンに関する注意: 長時間タスクとサブエージェントはどちらもトークンを消費します。コストが気になる場合は、
`agents.defaults.subagents.model` でサブエージェント向けにより安価なモデルを設定してください。

ドキュメント: [Sub-agents](/ja/tools/subagents).

<div id="cron-or-reminders-do-not-fire-what-should-i-check">
  ### Cron やリマインダーが動作しない 場合に確認すべきこと
</div>

Cron は Gateway プロセス内で動作します。Gateway が常時稼働していない場合、
スケジュールされたジョブは実行されません。

チェックリスト:

* cron が有効 (`cron.enabled`) であり、`OPENCLAW_SKIP_CRON` が設定されていないことを確認する。
* Gateway が 24/7 で稼働していることを確認する（スリープや再起動が発生していないこと）。
* ジョブのタイムゾーン設定（`--tz` とホストのタイムゾーン）を確認する。

デバッグ:

```bash
openclaw cron run <jobId> --force
openclaw cron runs --id <jobId> --limit 50
```

ドキュメント: [Cron ジョブ](/ja/automation/cron-jobs), [Cron とハートビート](/ja/automation/cron-vs-heartbeat).

<div id="how-do-i-install-skills-on-linux">
  ### Linux でスキルをインストールするにはどうすればよいですか
</div>

**ClawHub**（CLI）を使うか、スキルをワークスペースに配置します。macOS の Skills UI は Linux では利用できません。
スキルは https://clawhub.com で参照できます。

ClawHub CLI をインストールします（いずれか 1 つのパッケージマネージャーを選択）:

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```

<div id="can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background">
  ### OpenClaw はタスクをスケジュール実行したりバックグラウンドで継続実行できますか
</div>

はい。Gateway のスケジューラを使用します:

* スケジュールされたタスクや定期タスク用の **Cron ジョブ**（再起動後も保持されます）。
* 「メインセッション」の定期チェック用の **ハートビート**。
* 要約を投稿したりチャットに配信する自律型エージェント用の **分離ジョブ**。

ドキュメント: [Cron jobs](/ja/automation/cron-jobs), [Cron vs Heartbeat](/ja/automation/cron-vs-heartbeat),
[Heartbeat](/ja/gateway/heartbeat).

**Apple macOS 専用のスキルを Linux から実行できますか**

直接はできません。macOS スキルは `metadata.openclaw.os` と必要なバイナリによって制限されており、スキルは **Gateway ホスト** 上で利用可能な場合にのみシステムプロンプトに出現します。Linux 上では、`darwin` 専用スキル（`imsg`, `apple-notes`, `apple-reminders` など）は、この制限をオーバーライドしないかぎりロードされません。

サポートされているパターンは次の 3 つです:

**オプション A - Gateway を Mac 上で実行する（最も簡単）。**\
macOS バイナリが存在する場所で Gateway を実行し、その後 Linux から [remote mode](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere) または Tailscale 経由で接続します。Gateway ホストが macOS なので、スキルは通常どおりロードされます。

**オプション B - macOS ノードを使う（SSH 不要）。**\
Gateway を Linux 上で実行し、macOS ノード（メニューバーアプリ）をペアリングして、Mac 側の **Node Run Commands** を &quot;Always Ask&quot; または &quot;Always Allow&quot; に設定します。OpenClaw は、必要なバイナリがノード上に存在する場合、macOS 専用スキルを利用可能として扱えます。エージェントは `nodes` ツール経由でそれらのスキルを実行します。&quot;Always Ask&quot; を選択している場合、プロンプトで &quot;Always Allow&quot; を承認すると、そのコマンドが許可リストに追加されます。

**オプション C - macOS バイナリを SSH 経由でプロキシする（上級者向け）。**\
Gateway は Linux 上に置いたまま、必要な CLI バイナリが Mac 上で動作する SSH ラッパーに解決されるようにします。そのうえで、スキルをオーバーライドして Linux を許可し、スキルが利用可能な状態を維持します。

1. バイナリ用の SSH ラッパーを作成します（例: `imsg`）:
   ```bash
   #!/usr/bin/env bash
   set -euo pipefail
   exec ssh -T user@mac-host /opt/homebrew/bin/imsg "$@"
   ```
2. ラッパーを Linux ホストの `PATH` 上に配置します（例: `~/bin/imsg`）。
3. スキルのメタデータ（ワークスペースまたは `~/.openclaw/skills`）をオーバーライドして Linux を許可します:
   ```markdown
   ---
   name: imsg
   description: iMessage/SMS CLI for listing chats, history, watch, and sending.
   metadata: {"openclaw":{"os":["darwin","linux"],"requires":{"bins":["imsg"]}}}
   ---
   ```
4. 新しいセッションを開始して、スキルのスナップショットを更新させます。

特に iMessage については、`channels.imessage.cliPath` を SSH ラッパーに向けることもできます（OpenClaw が必要とするのは stdio だけです）。[iMessage](/ja/channels/imessage) を参照してください。

<div id="do-you-have-a-notion-or-heygen-integration">
  ### Notion や HeyGen との連携機能はありますか
</div>

現時点では、ビルトインでは提供していません。

オプション:

* **カスタムスキル / プラグイン:** 信頼性の高い API アクセスに最適です（Notion / HeyGen の両方に API があります）。
* **ブラウザ自動化:** コード不要で動作しますが、遅く、壊れやすい方法です。

クライアントごとにコンテキストを保持したい場合（エージェンシーのワークフローなど）には、シンプルなパターンとして次のような方法があります:

* クライアントごとに 1 つの Notion ページを作成する（コンテキスト + 設定や好み + 進行中の作業）。
* セッション開始時に、そのページを取得するようエージェントに指示する。

ネイティブ連携が欲しい場合は、機能要望を出すか、
それらの API をターゲットにしたスキルを自作してください。

スキルのインストール:

```bash
clawhub install <skill-slug>
clawhub update --all
```

ClawHub は現在のディレクトリ配下の `./skills` にインストールされます（または、設定済みの OpenClaw ワークスペースがフォールバック先として使用されます）。OpenClaw は次回のセッション時にそれを `<workspace>/skills` として扱います。エージェント間で共有して使うスキルは、`~/.openclaw/skills/<name>/SKILL.md` に配置してください。一部のスキルは Homebrew 経由でインストールされたバイナリを前提としています。Linux の場合は Linuxbrew が必要です（上記の Homebrew Linux FAQ の項目を参照してください）。[Skills](/ja/tools/skills) および [ClawHub](/ja/tools/clawhub) を参照してください。

<div id="how-do-i-install-the-chrome-extension-for-browser-takeover">
  ### ブラウザ制御用の Chrome 拡張機能はどうやってインストールすればよいですか
</div>

組み込みインストーラーを使用し、次に Chrome で「パッケージ化されていない拡張機能」を読み込んでください：

```bash
openclaw browser extension install
openclaw browser extension path
```

その後、Chrome を開き、`chrome://extensions` → 「デベロッパーモード」を有効化 → 「パッケージ化されていない拡張機能を読み込む」→ そのフォルダを選択します。

詳細な手順（リモート Gateway 利用時 + セキュリティの注意点を含む）: [Chrome extension](/ja/tools/chrome-extension)

Gateway が Chrome と同じマシン上で動いている場合（デフォルト構成）、通常は追加の設定は**不要**です。
Gateway が別の場所で動いている場合は、ブラウザが動いているマシン上でノードを実行し、Gateway がブラウザ操作をプロキシできるようにします。
制御したいタブでは、拡張機能ボタンをクリックする必要がある点は変わりません（自動でアタッチされることはありません）。

<div id="sandboxing-and-memory">
  ## サンドボックスとメモリ
</div>

<div id="is-there-a-dedicated-sandboxing-doc">
  ### サンドボックス専用のドキュメントはありますか
</div>

はい。 [Sandboxing](/ja/gateway/sandboxing) を参照してください。Docker 向けのセットアップ（Gateway 全体を Docker で動かす場合やサンドボックス用イメージ）は [Docker](/ja/install/docker) を参照してください。

**DM はプライベートのままにして、グループだけを 1 つのエージェントでサンドボックス化して公開できますか**

はい。プライベートなトラフィックが **DM** で、パブリックなトラフィックが **グループ** である場合に可能です。

`agents.defaults.sandbox.mode: "non-main"` を使用すると、グループ/チャンネルのセッション（non-main キー）が Docker 上で動作し、メインの DM セッションはホスト上に残ります。そのうえで、`tools.sandbox.tools` を使ってサンドボックス化されたセッションで利用可能なツールを制限します。

セットアップ手順と設定例: [Groups: personal DMs + public groups](/ja/concepts/groups#pattern-personal-dms-public-groups-single-agent)

主要な設定のリファレンス: [Gateway configuration](/ja/gateway/configuration#agentsdefaultssandbox)

<div id="how-do-i-bind-a-host-folder-into-the-sandbox">
  ### ホストフォルダをサンドボックスにバインドするにはどうすればよいですか
</div>

`agents.defaults.sandbox.docker.binds` を `["host:path:mode"]`（例: `"/home/user/src:/src:ro"`）に設定します。グローバル設定とエージェント単位のバインドはマージされますが、`scope: "shared"` の場合はエージェント単位のバインドは無視されます。機密性の高いパスには必ず `:ro` を指定し、バインドはサンドボックスによるファイルシステムの隔離を迂回してしまうことに注意してください。具体例と安全上の注意点については [Sandboxing](/ja/gateway/sandboxing#custom-bind-mounts) および [Sandbox vs Tool Policy vs Elevated](/ja/gateway/sandbox-vs-tool-policy-vs-elevated#bind-mounts-security-quick-check) を参照してください。

<div id="how-does-memory-work">
  ### メモリはどのように機能しますか
</div>

OpenClaw のメモリは、エージェントのワークスペース内にある単なる Markdown ファイルです:

* `memory/YYYY-MM-DD.md` に日次メモ
* `MEMORY.md` に厳選された長期メモ（メイン / プライベートセッションのみ）

OpenClaw は、モデルに対して自動コンパクションの前に長期保存用のメモを書き出すよう促すために、**サイレントな事前コンパクション・メモリフラッシュ**も実行します。これはワークスペースが書き込み可能な場合にのみ動作します（読み取り専用サンドボックスではスキップされます）。[Memory](/ja/concepts/memory) を参照してください。

<div id="memory-keeps-forgetting-things-how-do-i-make-it-stick">
  ### メモリがすぐに忘れてしまいます。どうすれば覚えたままにできますか
</div>

ボットに**その内容をメモリに書き込むように**依頼してください。長期的なメモは `MEMORY.md` に、
短期的なコンテキストは `memory/YYYY-MM-DD.md` に保存します。

このあたりは現在も改善を進めている領域です。モデルに対してメモリに保存するように都度伝えると効果があります。
モデルは何をすべきか理解しています。それでも忘れてしまう場合は、Gateway が毎回同じ
ワークスペースを使用しているか確認してください。

ドキュメント: [Memory](/ja/concepts/memory), [Agent workspace](/ja/concepts/agent-workspace).

<div id="does-semantic-memory-search-require-an-openai-api-key">
  ### セマンティックメモリ検索には OpenAI API キーが必要ですか
</div>

**OpenAI embeddings** を使う場合にだけ必要です。Codex OAuth は
chat/completions をカバーしますが、embeddings へのアクセスは **付与しません**。
そのため、**Codex（OAuth または Codex CLI のログイン）でサインインしても**
セマンティックメモリ検索には役立ちません。OpenAI embeddings には依然として
実際の API キー（`OPENAI_API_KEY` または `models.providers.openai.apiKey`）
が必要です。

プロバイダーを明示的に指定しない場合、OpenClaw は API キー
（認証プロファイル、`models.providers.*.apiKey`、または環境変数）を検出できるときに
プロバイダーを自動選択します。OpenAI のキーが検出できる場合は OpenAI を優先し、
そうでなければ Gemini のキーが検出できる場合は Gemini を優先します。
どちらのキーも利用できない場合は、設定するまでメモリ検索は無効のままです。
ローカルモデルパスが設定され、存在している場合は、OpenClaw は `local` を優先します。

処理をローカルにとどめたい場合は、`memorySearch.provider = "local"` を設定し
（必要に応じて `memorySearch.fallback = "none"` も設定します）。
Gemini embeddings を使いたい場合は `memorySearch.provider = "gemini"` を設定し、
`GEMINI_API_KEY`（または `memorySearch.remote.apiKey`）を指定してください。
**OpenAI、Gemini、または local** の embedding モデルをサポートしています。
セットアップの詳細は [Memory](/ja/concepts/memory) を参照してください。

<div id="does-memory-persist-forever-what-are-the-limits">
  ### メモリは永続しますか？ 制限はどのようなものですか？
</div>

メモリファイルはディスク上に保存され、削除しない限り保持されます。制限となるのはモデルではなくストレージ容量です。**セッションコンテキスト**はモデルのコンテキストウィンドウによって引き続き制限されるため、長い会話は圧縮されたり途中で切り捨てられたりすることがあります。そのために memory search があり、関連する部分だけをコンテキストに取り込み直します。

ドキュメント: [Memory](/ja/concepts/memory), [Context](/ja/concepts/context).

<div id="where-things-live-on-disk">
  ## ディスク上の保存場所
</div>

<div id="is-all-data-used-with-openclaw-saved-locally">
  ### OpenClaw で使われるデータはすべてローカルに保存されますか
</div>

いいえ。**OpenClaw の状態はローカル**ですが、**外部サービスには、あなたが送信した内容が見えます**。

* **基本的にはローカル:** セッション、メモリファイル、設定、ワークスペースは Gateway ホスト上
  (`~/.openclaw` + あなたのワークスペースディレクトリ) に存在します。
* **必要な部分はリモート:** モデルプロバイダー (Anthropic、OpenAI など) に送信するメッセージは
  それらの api に送られ、チャットプラットフォーム (WhatsApp、Telegram、Slack など) はメッセージデータを
  それぞれのサーバー上に保存します。
* **フットプリントはあなたが制御:** ローカルモデルを使うことでプロンプトはあなたのマシン上に留まりますが、
  チャネルのトラフィックは依然としてチャネルのサーバーを経由します。

関連: [エージェントのワークスペース](/ja/concepts/agent-workspace), [メモリ](/ja/concepts/memory).

<div id="where-does-openclaw-store-its-data">
  ### OpenClaw はどこにデータを保存しますか
</div>

すべて `$OPENCLAW_STATE_DIR` 配下に保存されます（デフォルト: `~/.openclaw`）:

| パス | 目的 |
|------|---------|
| `$OPENCLAW_STATE_DIR/openclaw.json` | メイン設定ファイル (JSON5) |
| `$OPENCLAW_STATE_DIR/credentials/oauth.json` | レガシーな OAuth インポート（初回使用時に認証プロファイルへコピーされる） |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/agent/auth-profiles.json` | 認証プロファイル (OAuth + API キー) |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/agent/auth.json` | 実行時認証キャッシュ（自動管理される） |
| `$OPENCLAW_STATE_DIR/credentials/` | プロバイダー状態（例: `whatsapp/<accountId>/creds.json`） |
| `$OPENCLAW_STATE_DIR/agents/` | エージェントごとの状態 (agentDir + セッション群) |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/` | 会話履歴と状態（エージェントごと） |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/sessions.json` | セッションのメタデータ（エージェントごと） |

旧来の単一エージェント用パス: `~/.openclaw/agent/*`（`openclaw doctor` によって移行される）。

**ワークスペース**（AGENTS.md、メモリファイル、スキルなど）は別で管理され、`agents.defaults.workspace` で設定します（デフォルト: `~/.openclaw/workspace`）。

<div id="where-should-agentsmd-soulmd-usermd-memorymd-live">
  ### AGENTSmd、SOULmd、USERmd、MEMORYmd はどこに置くべきか
</div>

これらのファイルは **エージェントのワークスペース** に配置し、`~/.openclaw` には置きません。

* **ワークスペース（エージェントごと）**: `AGENTS.md`, `SOUL.md`, `IDENTITY.md`, `USER.md`,
  `MEMORY.md`（または `memory.md`）、`memory/YYYY-MM-DD.md`、任意で `HEARTBEAT.md`
* **状態ディレクトリ（`~/.openclaw`）**: 設定、クレデンシャル、認証プロファイル、セッション、ログ、
  共有スキル（`~/.openclaw/skills`）

デフォルトのワークスペースは `~/.openclaw/workspace` で、次の方法で設定できます:

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } }
}
```

再起動後にボットが「忘れてしまう」場合は、Gateway が起動のたびに同じ
ワークスペースを使用しているか確認してください（なお、リモートモードでは、
あなたのローカルマシンではなく **Gateway ホスト側の**
ワークスペースが使われます）。

ヒント: 永続的な動作や設定を保存したい場合は、チャット履歴に頼むのではなく、
ボットに **AGENTS.md や MEMORY.md に書き込むよう依頼**してください。

[エージェントのワークスペース](/ja/concepts/agent-workspace) と [メモリ](/ja/concepts/memory) を参照してください。

<div id="whats-the-recommended-backup-strategy">
  ### 推奨されるバックアップ戦略は何ですか
</div>

**エージェントのワークスペース**を**非公開**の git リポジトリに置き、そのリポジトリを
非公開の場所（例: GitHub のプライベートリポジトリ）にバックアップしてください。これによりメモリと AGENTS/SOUL/USER
ファイルが丸ごと保存され、後からアシスタントの「頭脳」を復元できます。

`~/.openclaw` 配下（認証情報、セッション、トークン）は**絶対に**コミットしないでください。
完全な復元が必要な場合は、ワークスペースと state ディレクトリの両方を
個別にバックアップしてください（上の移行に関する質問を参照）。

ドキュメント: [Agent workspace](/ja/concepts/agent-workspace).

<div id="how-do-i-completely-uninstall-openclaw">
  ### OpenClaw を完全にアンインストールするにはどうすればよいですか
</div>

詳しくは専用ガイドを参照してください: [アンインストール](/ja/install/uninstall)。

<div id="can-agents-work-outside-the-workspace">
  ### エージェントはワークスペースの外でも動作できますか
</div>

はい。ワークスペースは**デフォルトの cwd（カレントワーキングディレクトリ）**かつメモリアンカーであり、厳密なサンドボックスではありません。
相対パスはワークスペース内で解決されますが、サンドボックス化を有効にしない限り、絶対パスはホスト上の他の場所にもアクセスできます。
分離が必要な場合は [`agents.defaults.sandbox`](/ja/gateway/sandboxing) またはエージェントごとのサンドボックス設定を使用してください。
特定のリポジトリをデフォルトの作業ディレクトリにしたい場合は、そのエージェントの
`workspace` をそのリポジトリのルートに設定してください。OpenClaw のリポジトリは単なるソースコードなので、
意図的にその中でエージェントを動かしたいのでない限り、ワークスペースは別にしておいてください。

例 (リポジトリをデフォルトの cwd とする場合):

```json5
{
  agents: {
    defaults: {
      workspace: "~/Projects/my-repo"
    }
  }
}
```

<div id="im-in-remote-mode-where-is-the-session-store">
  ### リモートモードで動作しているとき、セッションストアはどこにありますか
</div>

セッションの状態は **Gateway ホスト** が保持します。リモートモードの場合、対象となるセッションストアはあなたのローカルマシンではなく、リモートマシン上にあります。[セッション管理](/ja/concepts/session) を参照してください。

<div id="config-basics">
  ## コンフィグの基本
</div>

<div id="what-format-is-the-config-where-is-it">
  ### 設定ファイルの形式と場所
</div>

OpenClaw はオプションの **JSON5** 設定ファイルを `$OPENCLAW_CONFIG_PATH` から読み込みます（デフォルト: `~/.openclaw/openclaw.json`）：

```
$OPENCLAW_CONFIG_PATH
```

ファイルが存在しない場合は、比較的安全なデフォルト設定（デフォルトのワークスペース `~/.openclaw/workspace` を含む）が使用されます。

<div id="i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized">
  ### gatewaybind を lan や tailnet に設定したら何も待ち受けず、UI に「unauthorized」と表示されます
</div>

ループバック以外へのバインドには**認証が必須**です。`gateway.auth.mode` と `gateway.auth.token` を設定するか、`OPENCLAW_GATEWAY_TOKEN` を使用してください。

```json5
{
  gateway: {
    bind: "lan",
    auth: {
      mode: "token",
      token: "replace-me"
    }
  }
}
```

メモ:

* `gateway.remote.token` は **リモート CLI 呼び出し専用**であり、Gateway のローカル認証は有効にはしません。
* Control UI は `connect.params.auth.token`（app/UI 設定に保存）で認証します。トークンを URL に含めることは避けてください。

<div id="why-do-i-need-a-token-on-localhost-now">
  ### なぜ今は localhost でもトークンが必要なのですか
</div>

ウィザードはデフォルトで Gateway トークンを生成します（ループバック上でも同様）ので、**ローカルの WS クライアントも必ず認証が必要**になります。これにより、他のローカルプロセスが Gateway を勝手に呼び出すことを防ぎます。Control UI の設定（またはクライアント側の設定）にトークンを貼り付けて接続してください。

どうしてもループバックを **完全に開放したい** 場合は、設定から `gateway.auth` を削除してください。openclaw doctor コマンドを使えば、いつでもトークンを生成できます: `openclaw doctor --generate-gateway-token`。

<div id="do-i-have-to-restart-after-changing-config">
  ### 設定を変更したら再起動が必要ですか
</div>

Gateway は設定ファイルを監視しており、ホットリロードをサポートしています:

* `gateway.reload.mode: "hybrid"` (デフォルト): 安全な変更は即時反映し、重大な変更は再起動が必要になります
* `hot`、`restart`、`off` もサポートされています

<div id="how-do-i-enable-web-search-and-web-fetch">
  ### web search と web fetch を有効にする方法
</div>

`web_fetch` は APIキーなしで動作します。`web_search` には Brave Search APIキー
が必要です。**推奨:** `openclaw configure --section web` を実行して、
`tools.web.search.apiKey` に保存してください。環境変数を使う場合は、Gateway プロセス向けに `BRAVE_API_KEY` を設定してください。

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE",
        maxResults: 5
      },
      fetch: {
        enabled: true
      }
    }
  }
}
```

注意:

* 許可リストを使用している場合は、`web_search`/`web_fetch` または `group:web` を追加してください。
* `web_fetch` はデフォルトで有効です（明示的に無効化しない限り）。
* デーモンは `~/.openclaw/.env`（またはサービス環境）から環境変数を読み込みます。

ドキュメント: [Web tools](/ja/tools/web)。

<div id="how-do-i-run-a-central-gateway-with-specialized-workers-across-devices">
  ### 中央の Gateway を 1つ動かしつつ、複数デバイスで専門特化したワーカーを動かすには
</div>

一般的なパターンは、**1つの Gateway**（例: Raspberry Pi）に加えて、**ノード**と**エージェント**を組み合わせる構成です:

* **Gateway（中央）:** チャンネル（Signal/WhatsApp）、ルーティング、およびセッションを管理します。
* **ノード（デバイス）:** Mac/iOS/Android が周辺機器として接続し、ローカルツール（`system.run`、`canvas`、`camera`）を公開します。
* **エージェント（ワーカー）:** 特定の役割向けに分離された「頭脳」/ワークスペース（例: 「Hetzner ops」「Personal data」）です。
* **サブエージェント:** 並列実行したいときに、メインエージェントからバックグラウンド作業を生成します。
* **TUI:** Gateway に接続し、エージェントやセッションを切り替えます。

ドキュメント: [Nodes](/ja/nodes)、[Remote access](/ja/gateway/remote)、[Multi-Agent Routing](/ja/concepts/multi-agent)、[Sub-agents](/ja/tools/subagents)、[TUI](/ja/tui)。

<div id="can-the-openclaw-browser-run-headless">
  ### OpenClaw ブラウザをヘッドレスで実行できますか
</div>

はい。設定オプションで指定できます：

```json5
{
  browser: { headless: true },
  agents: {
    defaults: {
      sandbox: { browser: { headless: true } }
    }
  }
}
```

デフォルトは `false`（ヘッドフル）です。ヘッドレスは、一部のサイトでボット対策チェックを引き起こしやすくなります。[Browser](/ja/tools/browser) を参照してください。

ヘッドレスは **同じ Chromium エンジン** を使用しており、ほとんどの自動化（フォーム入力、クリック、スクレイピング、ログイン）で問題なく動作します。主な違いは次のとおりです。

* ブラウザウィンドウが表示されません（画面が必要な場合はスクリーンショットを使用してください）。
* 一部のサイトはヘッドレスモードでの自動化に対してより厳格です（CAPTCHA、ボット対策など）。
  たとえば、X/Twitter はヘッドレスセッションをブロックすることがよくあります。

<div id="how-do-i-use-brave-for-browser-control">
  ### ブラウザ制御に Brave を使うにはどうすればよいですか
</div>

`browser.executablePath` を Brave のバイナリ（または他の Chromium ベースのブラウザ）に設定し、Gateway を再起動してください。
詳しい設定例は [Browser](/ja/tools/browser#use-brave-or-another-chromium-based-browser) を参照してください。

<div id="remote-gateways-nodes">
  ## リモート Gateway とノード
</div>

<div id="how-do-commands-propagate-between-telegram-the-gateway-and-nodes">
  ### Telegram、Gateway、ノード間でコマンドはどのように伝播するのですか
</div>

Telegram メッセージは **gateway** が処理します。Gateway はエージェントを実行し、
ノードツールが必要になったときにだけ **Gateway WebSocket** 経由でノードを呼び出します:

Telegram → Gateway → Agent → `node.*` → Node → Gateway → Telegram

ノードはプロバイダーからの受信トラフィックを見ることはなく、ノード向けの RPC 呼び出しだけを受け取ります。

<div id="how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely">
  ### Gateway がリモートにホストされている場合に、エージェントはどうやって自分のコンピュータへアクセスできますか
</div>

結論: **自分のコンピュータをノードとしてペアリングします**。Gateway は別の場所で動作しますが、
Gateway の WebSocket 経由でローカルマシン上の `node.*` ツール（screen, camera, system）を呼び出せます。

典型的な構成:

1. Gateway を常時稼働ホスト（VPS/ホームサーバー）で実行する。
2. Gateway ホストと自分のコンピュータを同じ tailnet 上に置く。
3. Gateway の WS に到達できることを確認する（tailnet 経由で bind するか、SSH トンネルを張る）。
4. ローカルで macOS アプリを開き、**Remote over SSH** モード（または直接 tailnet）で接続して、
   ノードとして登録できるようにする。
5. Gateway 側でノードを承認する:
   ```bash
   openclaw nodes pending
   openclaw nodes approve <requestId>
   ```

別途 TCP ブリッジを用意する必要はありません。ノードは Gateway の WebSocket 経由で接続します。

セキュリティ上の注意: macOS ノードをペアリングすると、そのマシン上で `system.run` を実行できるようになります。
信頼できるデバイスだけをペアリングし、[Security](/ja/gateway/security) を確認してください。

ドキュメント: [Nodes](/ja/nodes)、[Gateway protocol](/ja/gateway/protocol)、[macOS remote mode](/ja/platforms/mac/remote)、[Security](/ja/gateway/security)。

<div id="tailscale-is-connected-but-i-get-no-replies-what-now">
  ### Tailscale には接続できているのに応答がありません。どうすればよいですか
</div>

まずは基本を確認します:

* Gateway が動作しているか: `openclaw gateway status`
* Gateway のヘルス状態: `openclaw status`
* Channel のヘルス状態: `openclaw channels status`

次に認証とルーティングを確認します:

* Tailscale Serve を使っている場合は、`gateway.auth.allowTailscale` が正しく設定されていることを確認してください。
* SSH トンネル経由で接続している場合は、ローカルトンネルが確立されており、正しいポートを指していることを確認してください。
* 許可リスト（DM またはグループ）に自分のアカウントが含まれていることを確認してください。

ドキュメント: [Tailscale](/ja/gateway/tailscale)、[Remote access](/ja/gateway/remote)、[Channels](/ja/channels)。

<div id="can-two-openclaw-instances-talk-to-each-other-local-vps">
  ### ローカルや VPS 上で動かしている 2 つの OpenClaw インスタンス同士を通信させることはできますか
</div>

はい。標準の「ボット間ブリッジ」はありませんが、いくつかの
信頼性の高い方法で実現できます。

**最も簡単な方法:** 両方のボットがアクセスできる通常のチャットチャネル（Telegram/Slack/WhatsApp など）を使います。
Bot A から Bot B へメッセージを送信させ、その後は Bot B に通常どおり返信させます。

**CLI ブリッジ（汎用）:** 他方の Gateway に対して
`openclaw agent --message ... --deliver` を呼び出すスクリプトを実行し、
相手のボットが待ち受けているチャットをターゲットにします。どちらかのボットがリモート VPS 上にある場合は、
SSH/Tailscale 経由で CLI をそのリモート Gateway に向けます（[リモートアクセス](/ja/gateway/remote) を参照）。

利用パターンの例（ターゲットの Gateway に到達可能なマシンで実行）:

```bash
openclaw agent --message "Hello from local bot" --deliver --channel telegram --reply-to <chat-id>
```

ヒント: 2つのボット同士が延々とループしないように、ガードレールを追加してください（メンションのみ、チャンネルの許可リスト、または「ボットからのメッセージには返信しない」というルールなど）。

ドキュメント: [リモートアクセス](/ja/gateway/remote)、[エージェント CLI](/ja/cli/agent)、[エージェント送信](/ja/tools/agent-send)。

<div id="do-i-need-separate-vpses-for-multiple-agents">
  ### 複数のエージェント用に別々の VPS が必要ですか
</div>

いいえ。1 つの Gateway で複数のエージェントをホストでき、それぞれに独自のワークスペース、モデルのデフォルト設定や
ルーティングを持たせることができます。これが通常の構成であり、エージェントごとに 1 台ずつ VPS を動かすよりも
はるかに安価でシンプルです。

厳格な分離（セキュリティ境界）が必要な場合や、共有したくないほど大きく異なる設定が必要な場合にのみ、VPS を分けてください。
それ以外では、Gateway は 1 つにしておき、複数のエージェントやサブエージェントを使ってください。

<div id="is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps">
  ### VPS から SSH する代わりに自分のノートPC上でノードを使うことにメリットはありますか
</div>

はい。ノードは、リモートの Gateway からあなたのノートPCに到達するための第一級の手段であり、
単なるシェルアクセス以上のことができます。Gateway は macOS/Linux（Windows は WSL2 経由）で動作し、
軽量です（小さな VPS や Raspberry Pi クラスのマシンで十分で、4 GB RAM あれば問題ありません）。そのため、
よくある構成としては、常時稼働のホストに加えて、あなたのノートPCをノードとして接続する構成になります。

* **受信側で SSH を開ける必要がない。** ノードは Gateway の WS（WebSocket）に外向き接続し、デバイスのペアリングを利用します。
* **より安全な実行制御。** `system.run` は、そのノートPC上のノードの許可リスト／承認によって制限されます。
* **より多くのデバイスツール。** ノードは `system.run` に加えて `canvas`、`camera`、`screen` を公開します。
* **ローカルブラウザの自動化。** Gateway は VPS 上で動かしたまま、Chrome はローカルで動かし、Chrome 拡張機能と
  ノートPC上のノードホストを組み合わせて制御を中継できます。

SSH はその場限りのシェルアクセスには十分ですが、継続的なエージェントのワークフローや
デバイス自動化にはノードの方がシンプルです。

Docs: [Nodes](/ja/nodes), [Nodes CLI](/ja/cli/nodes), [Chrome extension](/ja/tools/chrome-extension).

<div id="should-i-install-on-a-second-laptop-or-just-add-a-node">
  ### 2台目のラップトップにインストールすべきか、それともノードを追加すべきか
</div>

2台目のラップトップで **ローカルツール**（screen/camera/exec）だけが必要な場合は、それを
**ノード**として追加してください。Gateway を1つにまとめておけるため、設定の重複を避けられます。ローカルノードツールは
現時点では macOS 限定ですが、今後ほかの OS にも拡張していく予定です。

**ハードな分離**が必要な場合や、完全に独立した2つのボットが必要な場合にのみ、2つ目の Gateway をインストールしてください。

ドキュメント: [Nodes](/ja/nodes), [Nodes CLI](/ja/cli/nodes), [Multiple gateways](/ja/gateway/multiple-gateways).

<div id="do-nodes-run-a-gateway-service">
  ### ノードは Gateway サービスを実行しますか
</div>

いいえ。意図的に分離されたプロファイルを実行する場合を除き、ホストあたりに実行する **Gateway は 1 つだけ** にしてください（[Multiple gateways](/ja/gateway/multiple-gateways) を参照）。ノードは Gateway に接続する周辺デバイスです（iOS/Android ノード、またはメニューバーアプリの macOS「ノードモード」）。ヘッドレスなノードホストおよび CLI からの制御については、[Node host CLI](/ja/cli/node) を参照してください。

`gateway`、`discovery`、`canvasHost` の変更を反映するには、完全な再起動が必要です。

<div id="is-there-an-api-rpc-way-to-apply-config">
  ### 設定を適用するための API / RPC 経由の方法はありますか
</div>

はい。`config.apply` は設定全体を検証して書き込み、その処理の一環として Gateway を再起動します。

<div id="configapply-wiped-my-config-how-do-i-recover-and-avoid-this">
  ### configapply が設定を消してしまいました 復旧と再発防止方法は
</div>

`config.apply` は **設定全体** を置き換えます。部分的なオブジェクトだけを送ると、
それ以外はすべて削除されます。

復旧するには:

* バックアップから復元します（git またはコピーしておいた `~/.openclaw/openclaw.json`）。
* バックアップがない場合は、`openclaw doctor` を再実行し、チャネルやモデルを再設定します。
* 想定外の動作だった場合は、バグ報告を行い、最後に把握している設定ファイルまたは任意のバックアップを添付してください。
* ローカルのコーディング用エージェントであれば、多くの場合、ログや履歴から動作する設定を再構築できます。

再発防止:

* 小さな変更には `openclaw config set` を使用します。
* 対話的な編集には `openclaw configure` を使用します。

ドキュメント: [Config](/ja/cli/config), [Configure](/ja/cli/configure), [Doctor](/ja/gateway/doctor).

<div id="whats-a-minimal-sane-config-for-a-first-install">
  ### 初回インストール時の現実的な最小構成は？
</div>

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } }
}
```

これによりワークスペースが設定され、ボットを起動できるユーザーが制限されます。

<div id="how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac">
  ### VPS に Tailscale をセットアップして Mac から接続するには？
</div>

最小限の手順:

1. **VPS にインストールしてログインする**
   ```bash
   curl -fsSL https://tailscale.com/install.sh | sh
   sudo tailscale up
   ```
2. **Mac にインストールしてログインする**
   * Tailscale アプリを使って、同じ tailnet にサインインします。
3. **MagicDNS を有効化する（推奨）**
   * Tailscale の管理コンソールで MagicDNS を有効にし、VPS に安定した名前を割り当てます。
4. **tailnet のホスト名を使う**
   * SSH: `ssh user@your-vps.tailnet-xxxx.ts.net`
   * Gateway WS: `ws://your-vps.tailnet-xxxx.ts.net:18789`

SSH なしで Control UI にアクセスしたい場合は、VPS 上で Tailscale Serve を使用します:

```bash
openclaw gateway --tailscale serve
```

これにより Gateway はループバックインターフェイスにバインドされたままになり、HTTPS を Tailscale 経由で公開します。詳しくは [Tailscale](/ja/gateway/tailscale) を参照してください。

<div id="how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve">
  ### Mac ノードをリモート Gateway の Tailscale Serve に接続するにはどうすればよいですか
</div>

Serve は **Gateway Control UI と WS** を公開します。ノードは同じ Gateway の WS エンドポイント経由で接続します。

推奨セットアップ:

1. **VPS と Mac が同じ tailnet 上にあることを確認します**。
2. **macOS アプリを Remote モードで使用します**（SSH ターゲットには tailnet のホスト名を指定できます）。
   アプリが Gateway ポートをトンネルし、ノードとして接続します。
3. Gateway 上で **ノードを承認します**:
   ```bash
   openclaw nodes pending
   openclaw nodes approve <requestId>
   ```

ドキュメント: [Gateway protocol](/ja/gateway/protocol), [Discovery](/ja/gateway/discovery), [macOS remote mode](/ja/platforms/mac/remote).

<div id="env-vars-and-env-loading">
  ## 環境変数と .env ファイルの読み込み
</div>

<div id="how-does-openclaw-load-environment-variables">
  ### OpenClaw は環境変数をどのように読み込むのか
</div>

OpenClaw は親プロセス（シェル、launchd/systemd、CI など）が持つ環境変数を読み取り、加えて次を読み込みます:

* カレントディレクトリ（現在の作業ディレクトリ）の `.env`
* グローバルなフォールバック用 `.env`（`~/.openclaw/.env`、別名 `$OPENCLAW_STATE_DIR/.env`）

どちらの `.env` ファイルも、既に存在する環境変数の値を上書きしません。

また、設定ファイル内でインラインの環境変数を定義することもできます（プロセス環境に存在しない場合にのみ適用されます）:

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." }
  }
}
```

環境変数の優先順位と読み込み元の詳細については [/environment](/ja/environment) を参照してください。

<div id="i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now">
  ### サービス経由で Gateway を起動したら環境変数が消えました。どうすればいいですか
</div>

よくある対処法は次の 2 つです。

1. 不足しているキーを `~/.openclaw/.env` に書いてください。こうしておくと、サービスがシェル環境変数を引き継がない場合でも読み込まれます。
2. シェル環境のインポートを有効化する（任意の便利機能）:

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000
    }
  }
}
```

これはログインシェルを実行し、不足している想定済みキーのみをインポートします（既存の値を決して上書きしません）。対応する環境変数は次のとおりです：
`OPENCLAW_LOAD_SHELL_ENV=1`, `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`。

<div id="i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why">
  ### COPILOTGITHUBTOKEN を設定したのに models status に「Shell env: off」と出るのはなぜですか
</div>

`openclaw models status` は **shell env import** が有効かどうかを表示します。「Shell env: off」
と表示されていても、環境変数が存在しないという意味では **ありません**。これは単に OpenClaw が
ログインシェルを自動で読み込まない、という意味です。

Gateway をサービス（launchd/systemd）として動かしている場合は、シェル環境を
継承しません。次のいずれかで対処してください:

1. トークンを `~/.openclaw/.env` に記述します:
   ```
   COPILOT_GITHUB_TOKEN=...
   ```
2. もしくはシェル環境のインポートを有効にします（`env.shellEnv.enabled: true`）。
3. または設定ファイルの `env` ブロックに追加します（環境変数が存在しない場合にのみ適用されます）。

その後 Gateway を再起動して、もう一度確認してください:

```bash
openclaw models status
```

Copilot トークンは `COPILOT_GITHUB_TOKEN`（`GH_TOKEN` / `GITHUB_TOKEN` も使用できます）から読み取られます。
[/concepts/model-providers](/ja/concepts/model-providers) と [/environment](/ja/environment) を参照してください。

<div id="sessions-multiple-chats">
  ## セッションと複数のチャット
</div>

<div id="how-do-i-start-a-fresh-conversation">
  ### 新しい会話を開始するには？
</div>

`/new` または `/reset` を単独のメッセージとして送信します。[セッション管理](/ja/concepts/session) を参照してください。

<div id="do-sessions-reset-automatically-if-i-never-send-new">
  ### 新しいメッセージを送信しない場合、セッションは自動的にリセットされますか
</div>

はい。セッションは `session.idleMinutes`（デフォルト **60**）後に失効します。**次**の
メッセージで、そのチャットキー用に新しいセッションIDが開始されます。これは
トランスクリプトが削除されるという意味ではなく、単に新しいセッションが開始されるだけです。

```json5
{
  session: {
    idleMinutes: 240
  }
}
```

<div id="is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents">
  ### OpenClaw インスタンスで 1 人の CEO と多数のエージェントからなるチームを作る方法はあるか
</div>

はい、**マルチエージェント・ルーティング** と **サブエージェント** で実現できます。1 つのコーディネーター
エージェントと、独自のワークスペースとモデルを持つ複数のワーカーエージェントを作成できます。

とはいえ、これはどちらかというと **おもしろ半分の実験** と考えるのがよいです。トークン消費が多く、個別のセッションを持つ 1 つのボットを使う場合より
非効率になることが多いです。OpenClaw が想定している一般的なモデルは、「会話するボットは 1 つにして、並列作業にはセッションを分ける」という形です。
必要に応じて、そのボットがサブエージェントを起動できます。

ドキュメント: [マルチエージェント・ルーティング](/ja/concepts/multi-agent)、[サブエージェント](/ja/tools/subagents)、[Agents CLI](/ja/cli/agents)。

<div id="why-did-context-get-truncated-midtask-how-do-i-prevent-it">
  ### なぜタスクの途中でコンテキストが切り捨てられてしまったのか どう防げばよいか
</div>

セッションのコンテキストはモデルのコンテキストウィンドウによって制限されます。長いチャット、大きなツール出力、または多数の
ファイルがあると、圧縮や切り捨てが発生することがあります。

有効な対処方法:

* 現在の状態をボットに要約させて、ファイルに書き出させる。
* 長いタスクの前に `/compact` を使い、トピックを切り替えるときは `/new` を使う。
* 重要なコンテキストはワークスペースに保存し、必要なときにボットに読み込ませる。
* メインチャットを小さく保つために、長時間または並列作業にはサブエージェントを使う。
* これが頻発するようであれば、より大きなコンテキストウィンドウを持つモデルを選ぶ。

<div id="how-do-i-completely-reset-openclaw-but-keep-it-installed">
  ### OpenClaw をアンインストールせずに完全リセットするにはどうすればよいですか
</div>

`reset` コマンドを使用します。

```bash
openclaw reset
```

非対話的な完全リセット：

```bash
openclaw reset --scope full --yes --non-interactive
```

その後、オンボーディングを再実行してください:

```bash
openclaw onboard --install-daemon
```

補足:

* 既存の設定が見つかった場合、オンボーディングウィザードでも **Reset** を使用できます。詳しくは [Wizard](/ja/start/wizard) を参照してください。
* プロファイル（`--profile` / `OPENCLAW_PROFILE`）を使用している場合は、各 state ディレクトリをリセットしてください（デフォルトは `~/.openclaw-<profile>`）。
* 開発用リセット: `openclaw gateway --dev --reset`（開発環境専用。開発用の設定・認証情報・セッション・ワークスペースをすべて削除します）。

<div id="im-getting-context-too-large-errors-how-do-i-reset-or-compact">
  ### コンテキストが大きすぎるエラーが出ます。どうリセットまたは圧縮すればよいですか？
</div>

次のいずれかを使ってください:

* **Compact**（会話は維持しつつ、古いターンを要約）:
  ```
  /compact
  ```
  または `/compact <instructions>` で、要約の方針を指示します。

* **Reset**（同じチャットキーで新しいセッションIDを発行）:
  ```
  /new
  /reset
  ```

それでも同じエラーが発生する場合は、次を試してください:

* 古いツール出力を削るために **session pruning**（`agents.defaults.contextPruning`）を有効化または調整します。
* コンテキストウィンドウがより大きいモデルを使用します。

ドキュメント: [Compaction](/ja/concepts/compaction), [Session pruning](/ja/concepts/session-pruning), [Session management](/ja/concepts/session).

<div id="why-am-i-seeing-llm-request-rejected-messagesncontentxtooluseinput-field-required">
  ### LLM request rejected messagesNcontentXtooluseinput Field required というメッセージが表示されるのはなぜですか
</div>

これはプロバイダーによるバリデーションエラーです。モデルが必須の `input` を含めずに `tool_use` ブロックを出力しました。
多くの場合、長いスレッドのあとやツール／スキーマ変更後などで、セッション履歴が古くなっているか破損していることを意味します。

対処: `/new` を単独のメッセージとして送信し、新しいセッションを開始してください。

<div id="why-am-i-getting-heartbeat-messages-every-30-minutes">
  ### なぜ30分ごとにハートビートメッセージが届くのですか
</div>

ハートビートはデフォルトで **30m** 間隔で実行されます。必要に応じて設定を調整するか、無効にしてください。

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "2h"   // または "0m" で無効化
      }
    }
  }
}
```

`HEARTBEAT.md` が存在していても、実質的に空（空行と `# Heading` のような markdown
ヘッダーしかない状態）の場合、OpenClaw は API 呼び出しを節約するためにハートビート実行をスキップします。
ファイルが存在しない場合でも、ハートビートは実行され、モデルがどのように振る舞うかを判断します。

エージェント単位でのオーバーライドには `agents.list[].heartbeat` を使用します。ドキュメント: [Heartbeat](/ja/gateway/heartbeat)。

<div id="do-i-need-to-add-a-bot-account-to-a-whatsapp-group">
  ### WhatsApp グループに bot アカウントを追加する必要がありますか？
</div>

いいえ。OpenClaw は**あなた自身のアカウント**で動作するため、あなたがそのグループに参加していれば、OpenClaw もそのグループを参照できます。
デフォルトでは、送信者を許可するまで（`groupPolicy: "allowlist"`）グループへの返信はブロックされます。

**自分だけ**がグループ返信をトリガーできるようにしたい場合は:

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"]
    }
  }
}
```

<div id="how-do-i-get-the-jid-of-a-whatsapp-group">
  ### WhatsAppグループのJIDを取得するにはどうすればよいですか
</div>

オプション1（最速）：ログをtailで監視し、グループにテストメッセージを送信します：

```bash
openclaw logs --follow --json
```

`@g.us` で終わる `chatId`（または `from`）を探してください。例:
`1234567890-1234567890@g.us`。

オプション 2（すでに設定済み/許可リストに登録済みの場合）: config からグループを一覧表示します:

```bash
openclaw directory groups list --channel whatsapp
```

ドキュメント：[WhatsApp](/ja/channels/whatsapp)、[Directory](/ja/cli/directory)、[Logs](/ja/cli/logs)。

<div id="why-doesnt-openclaw-reply-in-a-group">
  ### なぜ OpenClaw はグループチャットで返信しないのか
</div>

よくある原因は次の2つが考えられます:

* メンション制御が有効（デフォルト）になっている。ボットに @メンションする（または `mentionPatterns` に一致させる）必要があります。
* `channels.whatsapp.groups` を `"*"` なしで設定しており、そのグループが許可リストに登録されていません。

[Groups](/ja/concepts/groups) と [Group messages](/ja/concepts/group-messages) を参照してください。

<div id="do-groupsthreads-share-context-with-dms">
  ### グループやスレッドは DM とコンテキストを共有しますか
</div>

ダイレクトチャットは、デフォルトではメインのセッションに集約されます。グループ／チャンネルにはそれぞれ独自のセッションキーがあり、Telegram のトピックや Discord のスレッドは別個のセッションとして扱われます。[グループ](/ja/concepts/groups)および[グループメッセージ](/ja/concepts/group-messages)を参照してください。

<div id="how-many-workspaces-and-agents-can-i-create">
  ### ワークスペースとエージェントはどれくらい作成できますか
</div>

ハードリミットはありません。数十（場合によっては数百）でも問題ありませんが、次の点に注意してください。

* **ディスク使用量の増加:** セッションとトランスクリプト（会話ログ）は `~/.openclaw/agents/<agentId>/sessions/` 配下に保存されます。
* **トークンコスト:** エージェントが多いほど、同時に利用されるモデルが増えます。
* **運用負荷:** エージェントごとの認証プロファイル、ワークスペース、チャンネルルーティングが増えます。

運用上のヒント:

* エージェントごとに **アクティブ** なワークスペースを 1 つに保つ（`agents.defaults.workspace`）。
* ディスク使用量が増えてきたら、古いセッション（JSONL またはストアのエントリ）を削除して整理する。
* `openclaw doctor` を使って、不要なワークスペースやプロファイルの不整合を検出する。

<div id="can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up">
  ### Slack で複数のボットやチャットを同時に動かすことはできますか？その場合の設定方法を教えてください
</div>

はい。**Multi‑Agent Routing** を使うことで、複数の互いに独立したエージェントを動かし、
チャネル／アカウント／相手ごとに受信メッセージをルーティングできます。Slack はチャネルとしてサポートされており、特定のエージェントにバインドできます。

ブラウザアクセスは強力ですが、「人間ができることを何でも代替できる」わけではありません。ボット対策や CAPTCHA、MFA によっては、自動化が依然としてブロックされる場合があります。最も信頼性の高いブラウザ制御を行うには、ブラウザを動かしているマシン上で Chrome extension relay（Chrome 拡張機能によるリレー）を使用してください（Gateway はどこにあっても構いません）。

推奨セットアップ:

* 常時稼働する Gateway ホスト（VPS／Mac mini など）。
* ロール（役割）ごとに 1 つのエージェント（バインディング）。
* それらのエージェントにバインドされた Slack チャネル群。
* 必要に応じて、拡張機能リレー（またはノード）経由のローカルブラウザ。

ドキュメント: [Multi‑Agent Routing](/ja/concepts/multi-agent), [Slack](/ja/channels/slack),
[Browser](/ja/tools/browser), [Chrome extension](/ja/tools/chrome-extension), [Nodes](/ja/nodes).

<div id="models-defaults-selection-aliases-switching">
  ## モデル: デフォルト、選択、エイリアス、切り替え
</div>

<div id="what-is-the-default-model">
  ### デフォルトのモデルは何ですか
</div>

OpenClaw のデフォルトモデルは、次のいずれかとして設定したものです。

```
agents.defaults.model.primary
```

モデルは `provider/model` の形式で参照されます（例: `anthropic/claude-opus-4-5`）。プロバイダーを省略した場合、現時点の OpenClaw は非推奨に伴う一時的な後方互換のために `anthropic` を仮定しますが、**必ず** `provider/model` を明示的に指定してください。

<div id="what-model-do-you-recommend">
  ### 推奨するモデルはどれですか
</div>

**推奨デフォルト:** `anthropic/claude-opus-4-5`\
**良い代替候補:** `anthropic/claude-sonnet-4-5`\
**安定（キャラクター性少なめ）:** `openai/gpt-5.2` - Opus にかなり近い性能ですが、個性はやや弱めです。\
**予算重視:** `zai/glm-4.7`

MiniMax M2.1 には専用ドキュメントがあります: [MiniMax](/ja/providers/minimax) および
[ローカルモデル](/ja/gateway/local-models)。

経験則としては、重要度が高い作業には **予算の許す範囲で最良のモデル** を使い、日常的なチャットや要約にはより安価なモデルを使ってください。エージェントごとにモデルをルーティングしたり、サブエージェントを使って長いタスクを並列化することができます（各サブエージェントがトークンを消費します）。[Models](/ja/concepts/models) と
[Sub-agents](/ja/tools/subagents) を参照してください。

重要な警告: 性能の低いモデルや過度に量子化されたモデルは、プロンプトインジェクションや安全でない挙動に対して脆弱です。[Security](/ja/gateway/security) を参照してください。

詳しくは: [Models](/ja/concepts/models)。

<div id="can-i-use-selfhosted-models-llamacpp-vllm-ollama">
  ### セルフホストのモデル（llamacpp、vLLM、Ollama）は使えますか
</div>

はい。ローカルサーバーが OpenAI 互換の api を公開していれば、そこにカスタムのプロバイダーを向けることができます。Ollama はネイティブにサポートされており、最も簡単な方法です。

セキュリティに関する注意: 小型モデルや強く量子化されたモデルは、プロンプトインジェクションに対してより脆弱です。ツールを使用できるあらゆるボットには、**大規模モデル**の利用を強く推奨します。それでも小型モデルを使いたい場合は、サンドボックスを有効化し、厳格なツール許可リストを設定してください。

ドキュメント: [Ollama](/ja/providers/ollama)、[ローカルモデル](/ja/gateway/local-models)、[モデルプロバイダー](/ja/concepts/model-providers)、[セキュリティ](/ja/gateway/security)、[サンドボックス](/ja/gateway/sandboxing)。

<div id="how-do-i-switch-models-without-wiping-my-config">
  ### 設定を消さずにモデルを切り替えるにはどうすればよいですか
</div>

**model コマンド**を使うか、**model** フィールドだけを編集してください。設定全体の置き換えは避けてください。

安全な選択肢:

* チャット内で `/model` を使う（簡易・セッション単位）
* `openclaw models set ...`（model の設定だけを更新）
* `openclaw configure --section models`（対話的）
* `~/.openclaw/openclaw.json` 内の `agents.defaults.model` を編集

設定全体を置き換える意図がない限り、部分的なオブジェクトだけを指定して `config.apply` を実行するのは避けてください。
もし設定を上書きしてしまった場合は、バックアップから復元するか、`openclaw doctor` を再実行して修復します。

ドキュメント: [Models](/ja/concepts/models), [Configure](/ja/cli/configure), [Config](/ja/cli/config), [Doctor](/ja/gateway/doctor).

<div id="what-do-openclaw-flawd-and-krill-use-for-models">
  ### OpenClaw、Flawd、Krill が使用しているモデル
</div>

* **OpenClaw + Flawd:** Anthropic Opus (`anthropic/claude-opus-4-5`) - [Anthropic](/ja/providers/anthropic) を参照。
* **Krill:** MiniMax M2.1 (`minimax/MiniMax-M2.1`) - [MiniMax](/ja/providers/minimax) を参照。

<div id="how-do-i-switch-models-on-the-fly-without-restarting">
  ### 再起動せずにその場でモデルを切り替えるにはどうすればよいですか
</div>

`/model` コマンドを単独のメッセージとして送信します：

```
/model sonnet
/model haiku
/model opus
/model gpt
/model gpt-mini
/model gemini
/model gemini-flash
```

利用可能なモデルは `/model`、`/model list`、`/model status` で一覧表示できます。

`/model`（および `/model list`）は、コンパクトな番号付きの選択メニューを表示します。番号を入力して選択します。

```
/model 3
```

また、プロバイダーに対して特定の認証プロファイルを（セッション単位で）強制的に指定することもできます。

```
/model opus@anthropic:default
/model opus@anthropic:work
```

ヒント: `/model status` は、どのエージェントがアクティブか、どの `auth-profiles.json` ファイルが使用されているか、次にどの認証プロファイルが試されるかを表示します。
利用可能な場合は、設定されているプロバイダーのエンドポイント（`baseUrl`）と API モード（`api`）も表示します。

**`profile` で設定したプロファイルのピン留めを解除するには**

`@profile` サフィックスを **付けずに** `/model` を再実行します。

```
/model anthropic/claude-opus-4-5
```

デフォルト設定に戻したい場合は、`/model` からそれを選択するか、`/model <default provider/model>` を送信してください。
`/model status` を実行して、どの認証プロファイルが有効か確認してください。

<div id="can-i-use-gpt-52-for-daily-tasks-and-codex-52-for-coding">
  ### 日常作業には GPT 5.2、コーディングには Codex 5.2 を使えますか
</div>

はい。どちらか一方をデフォルトに設定し、必要に応じて切り替えてください。

* **すばやく切り替え（セッションごと）：** 日常作業には `/model gpt-5.2`、コーディングには `/model gpt-5.2-codex` を使用します。
* **デフォルト + 切り替え:** `agents.defaults.model.primary` を `openai-codex/gpt-5.2` に設定し、コーディング時に `openai-codex/gpt-5.2-codex` に切り替えます（またはその逆でも構いません）。
* **サブエージェント:** デフォルトモデルが異なるサブエージェントにコーディングタスクをルーティングします。

[Models](/ja/concepts/models) と [Slash commands](/ja/tools/slash-commands) を参照してください。

<div id="why-do-i-see-model-is-not-allowed-and-then-no-reply">
  ### 「Model is not allowed」が表示されて、その後応答が返ってこないのはなぜですか
</div>

`agents.defaults.models` が設定されている場合、それは `/model` および任意の
セッションのオーバーライドに対する**許可リスト**として機能します。そのリストに含まれていないモデルを選択すると、次のエラーが返されます：

```
Model "provider/model" is not allowed. Use /model to list available models.
```

そのエラーは通常の返信の**代わりに**返されます。対処方法: モデルを
`agents.defaults.models` に追加するか、許可リストを削除するか、`/model list` からモデルを選択してください。

<div id="why-do-i-see-unknown-model-minimaxminimaxm21">
  ### Unknown model minimaxMiniMaxM21 が表示されるのはなぜですか
</div>

これは、**プロバイダーが設定されていない**（MiniMax のプロバイダー設定または認証
プロファイルが見つからない）ためにモデルを特定できていない、という意味です。この検出に対する修正は
**2026.1.12**（執筆時点では未リリース）で行われています。

修正チェックリスト:

1. **2026.1.12** にアップグレードする（またはソースの `main` から実行する）うえで、Gateway を再起動する。
2. MiniMax が設定されていること（ウィザードまたは JSON）、
   あるいは MiniMax API キーが env/auth プロファイル内に存在していて、
   プロバイダーが注入される状態になっていることを確認する。
3. 正確なモデル ID（大文字小文字を区別）を使用する:
   `minimax/MiniMax-M2.1` または
   `minimax/MiniMax-M2.1-lightning`。
4. 次を実行する:
   ```bash
   openclaw models list
   ```
   そしてリストから選択する（またはチャットで `/model list` を実行する）。

[MiniMax](/ja/providers/minimax) および [Models](/ja/concepts/models) を参照してください。

<div id="can-i-use-minimax-as-my-default-and-openai-for-complex-tasks">
  ### デフォルトには MiniMax を使い、複雑な処理には OpenAI を使えますか？
</div>

はい。通常は **MiniMax をデフォルト** として使い、必要に応じて **セッションごとにモデルを切り替え** てください。
フォールバックは「難しいタスク」のためではなく **エラー用** なので、`/model` を使うか、別のエージェントを用意してください。

**オプション A: セッションごとに切り替える**

```json5
{
  env: { MINIMAX_API_KEY: "sk-...", OPENAI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "minimax/MiniMax-M2.1" },
      models: {
        "minimax/MiniMax-M2.1": { alias: "minimax" },
        "openai/gpt-5.2": { alias: "gpt" }
      }
    }
  }
}
```

次に、

```
/model gpt
```

**オプションB: エージェントを分ける**

* エージェントAのデフォルト: MiniMax
* エージェントBのデフォルト: OpenAI
* エージェント単位でルーティングするか、`/agent` を使って切り替える

ドキュメント: [Models](/ja/concepts/models)、[Multi-Agent Routing](/ja/concepts/multi-agent)、[MiniMax](/ja/providers/minimax)、[OpenAI](/ja/providers/openai)。

<div id="are-opus-sonnet-gpt-builtin-shortcuts">
  ### opus、sonnet、gpt はビルトインのショートカットですか
</div>

はい。OpenClaw には、いくつかのデフォルトのショートカット（エイリアス）が用意されています（`agents.defaults.models` にそのモデルが存在する場合にのみ適用されます）：

* `opus` → `anthropic/claude-opus-4-5`
* `sonnet` → `anthropic/claude-sonnet-4-5`
* `gpt` → `openai/gpt-5.2`
* `gpt-mini` → `openai/gpt-5-mini`
* `gemini` → `google/gemini-3-pro-preview`
* `gemini-flash` → `google/gemini-3-flash-preview`

同じ名前で独自のエイリアスを設定した場合は、そちらの値が優先されます。

<div id="how-do-i-defineoverride-model-shortcuts-aliases">
  ### モデルのショートカット用エイリアスを定義／上書きするにはどうすればよいですか
</div>

エイリアスは `agents.defaults.models.<modelId>.alias` で定義します。例:

```json5
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-5" },
      models: {
        "anthropic/claude-opus-4-5": { alias: "opus" },
        "anthropic/claude-sonnet-4-5": { alias: "sonnet" },
        "anthropic/claude-haiku-4-5": { alias: "haiku" }
      }
    }
  }
}
```

すると `/model sonnet`（またはサポートされている場合は `/<alias>`）はその model ID として解釈されます。

<div id="how-do-i-add-models-from-other-providers-like-openrouter-or-zai">
  ### OpenRouter や ZAI など、他のプロバイダーのモデルを追加するには？
</div>

OpenRouter（トークン従量課金制・多数のモデルを利用可能）:

```json5
{
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },
      models: { "openrouter/anthropic/claude-sonnet-4-5": {} }
    }
  },
  env: { OPENROUTER_API_KEY: "sk-or-..." }
}
```

Z.AI（GLMモデル）:

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} }
    }
  },
  env: { ZAI_API_KEY: "..." }
}
```

プロバイダー／モデルを参照しているのに必要なプロバイダーキーが設定されていない場合、実行時の認証エラーが発生します（例：`No API key found for provider "zai"`）。

**新しいエージェントを追加した後に「No API key found for provider」と表示される**

これは通常、**新しいエージェント**の認証ストアが空であることを意味します。認証情報はエージェントごとに管理され、次の場所に保存されます：

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

対処方法:

* `openclaw agents add <id>` を実行し、ウィザード中に認証情報を設定します。
* あるいはメインエージェントの `agentDir` から `auth-profiles.json` を新しいエージェントの `agentDir` にコピーします。

**決して** 複数のエージェント間で `agentDir` を再利用しないでください。認証・セッションの競合を引き起こします。

<div id="model-failover-and-all-models-failed">
  ## モデルのフェイルオーバーと「All models failed」
</div>

<div id="how-does-failover-work">
  ### フェイルオーバーはどのように機能しますか
</div>

フェイルオーバーは次の 2 段階で行われます：

1. 同一プロバイダー内での **認証プロファイルのローテーション（切り替え）**。
2. `agents.defaults.model.fallbacks` に定義された次のモデルへの **モデルフォールバック**。

失敗したプロファイルにはクールダウン（指数バックオフ）が適用されるため、プロバイダーがレート制限されている場合や一時的に障害が発生している場合でも、OpenClaw は応答を継続できます。

<div id="what-does-this-error-mean">
  ### このエラーの意味
</div>

```
No credentials found for profile "anthropic:default"
```

これは、システムが認証プロファイル ID `anthropic:default` を使用しようとしましたが、想定している auth ストア内にその資格情報が見つからなかったことを意味します。

<div id="fix-checklist-for-no-credentials-found-for-profile-anthropicdefault">
  ### プロファイル anthropicdefault に対して「No credentials found for profile anthropicdefault」が出る場合のチェックリスト
</div>

* **認証プロファイルがどこに保存されているかを確認する**（新パス vs 旧パス）
  * 現在: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
  * 旧パス: `~/.openclaw/agent/*`（`openclaw doctor` によってマイグレーションされる）
* **環境変数が Gateway によって読み込まれているか確認する**
  * シェルで `ANTHROPIC_API_KEY` を設定していても、Gateway を systemd/launchd 経由で起動している場合は継承されない可能性がある。`~/.openclaw/.env` に記載するか、`env.shellEnv` を有効化する。
* **正しいエージェントを編集しているか確認する**
  * マルチエージェント構成では、複数の `auth-profiles.json` ファイルが存在しうる。
* **モデル／認証状態を簡易チェックする**
  * `openclaw models status` を使って、設定済みモデルとプロバイダーが認証されているかを確認する。

**プロファイル anthropic に対して「No credentials found for profile anthropic」が出る場合のチェックリスト**

これは、その実行が Anthropic の認証プロファイルにピン留めされているものの、
Gateway がそのプロファイルを認証ストア内に見つけられていないことを意味する。

* **setup-token を使う**
  * `claude setup-token` を実行し、その結果を `openclaw models auth setup-token --provider anthropic` に貼り付ける。
  * そのトークンが別のマシンで作成された場合は、`openclaw models auth paste-token --provider anthropic` を使う。
* **代わりに API key を使いたい場合**
  * **Gateway ホスト** 上の `~/.openclaw/.env` に `ANTHROPIC_API_KEY` を設定する。
  * 存在しないプロファイルを強制しているピン留め済みの order 設定をクリアする:
    ```bash
    openclaw models auth order clear --provider anthropic
    ```
* **Gateway ホスト上でコマンドを実行しているか確認する**
  * リモートモードでは、認証プロファイルはあなたのノート PC ではなく Gateway マシン側に保存される。

<div id="why-did-it-also-try-google-gemini-and-fail">
  ### なぜ Google Gemini も試して失敗したのか
</div>

モデル構成に Google Gemini がフォールバックとして含まれている場合（または Gemini のショートハンドに切り替えた場合）、OpenClaw はモデルフォールバック時に Gemini を試行します。Google の認証情報を設定していないと、`No API key found for provider "google"` が表示されます。

対処方法: Google の認証情報を設定するか、`agents.defaults.model.fallbacks` / エイリアスから Google モデルを削除・回避して、フォールバックでそこにルーティングされないようにします。

**LLM request rejected message thinking signature required google antigravity**

原因: セッション履歴に**署名なしの thinking ブロック**が含まれています（中断された / 途中までのストリームから発生することが多い）。Google Antigravity では thinking ブロックに署名が必要です。

対処方法: OpenClaw は現在、Google Antigravity Claude 用に署名のない thinking ブロックを除去します。まだ同じ問題が出る場合は、**新しいセッション**を開始するか、そのエージェントで `/thinking off` を設定してください。

<div id="auth-profiles-what-they-are-and-how-to-manage-them">
  ## 認証プロファイル：その概要と管理方法
</div>

関連: [/concepts/oauth](/ja/concepts/oauth) (OAuth フロー、トークンの保存、複数アカウント利用パターン)

<div id="what-is-an-auth-profile">
  ### auth プロファイルとは
</div>

auth プロファイルは、プロバイダーに関連付けられた名前付きの認証情報レコード（OAuth または API キー）です。プロファイルは次の場所に保存されます：

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

<div id="what-are-typical-profile-ids">
  ### 典型的なプロファイル ID にはどのようなものがありますか
</div>

OpenClaw は、プロバイダー名をプレフィックスに付けた ID を使用します:

* `anthropic:default`（メールアイデンティティが存在しない場合によく使われる）
* OAuth アイデンティティ向けの `anthropic:<email>`
* 任意に指定するカスタム ID（例: `anthropic:work`）

<div id="can-i-control-which-auth-profile-is-tried-first">
  ### どの認証プロファイルが最初に試されるかを制御できますか？
</div>

はい。コンフィグでは、プロファイル用のオプションのメタデータと、プロバイダーごとの順序（`auth.order.<provider>`）をサポートしています。これはシークレットを保存するものではなく、ID を プロバイダー／モード にマッピングし、ローテーション順を設定します。

OpenClaw は、短い **クールダウン** 状態（レートリミット／タイムアウト／認証失敗）や、より長い **無効化** 状態（課金／クレジット不足）の場合、そのプロファイルを一時的にスキップすることがあります。状態を確認するには、`openclaw models status --json` を実行し、`auth.unusableProfiles` を確認してください。チューニング用キー: `auth.cooldowns.billingBackoffHours*`。

CLI を使って、特定のエージェントの `auth-profiles.json` に保存される **エージェント単位** の順序上書きを設定することもできます。

```bash
# Defaults to the configured default agent (omit --agent)
openclaw models auth order get --provider anthropic

# Lock rotation to a single profile (only try this one)
openclaw models auth order set --provider anthropic anthropic:default

# Or set an explicit order (fallback within provider)
openclaw models auth order set --provider anthropic anthropic:work anthropic:default

# オーバーライドをクリア(config auth.order / ラウンドロビンにフォールバック)
openclaw models auth order clear --provider anthropic
```

特定のエージェントを指定するには:

```bash
openclaw models auth order set --provider anthropic --agent main anthropic:default
```

<div id="oauth-vs-api-key-whats-the-difference">
  ### OAuth と API キーの違いは何ですか
</div>

OpenClaw は両方に対応しています。

* **OAuth** は（利用可能な場合）サブスクリプションベースのアクセスを利用することがよくあります。
* **API keys** はトークン単位の従量課金制を利用します。

wizard は明示的に Anthropic の setup-token と OpenAI Codex の OAuth をサポートしており、API キーを保存することもできます。

<div id="gateway-ports-already-running-and-remote-mode">
  ## Gateway：ポート、「すでに実行中」エラー、リモートモード
</div>

<div id="what-port-does-the-gateway-use">
  ### Gateway はどのポートを使用しますか
</div>

`gateway.port` は、WebSocket と HTTP（Control UI、hooks など）を多重化する単一のポートを制御します。

優先順位:

```
--port > OPENCLAW_GATEWAY_PORT > gateway.port > default 18789
```

<div id="why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed">
  ### なぜ `openclaw gateway status` は Runtime running と表示されるのに RPC probe failed になるのか
</div>

「running」は **supervisor**（launchd/systemd/schtasks）から見た状態だからです。RPC プローブは、CLI が実際に Gateway の WebSocket に接続し、`status` を呼び出す処理です。

`openclaw gateway status` を実行し、次の行の内容を信頼してください:

* `Probe target:`（プローブが実際に使用した URL）
* `Listening:`（実際にそのポートでバインド／待ち受けしている内容）
* `Last gateway error:`（プロセスは生きているがポートが待ち受け状態になっていない場合によくある根本原因）

<div id="why-does-openclaw-gateway-status-show-config-cli-and-config-service-different">
  ### なぜ `openclaw gateway status` で Config cli と Config service が違うと表示されるのか
</div>

サービスが参照している設定ファイルとは別の設定ファイルをあなたが編集しているためです（多くの場合、`--profile` / `OPENCLAW_STATE_DIR` の不一致が原因です）。

対処方法:

```bash
openclaw gateway install --force
```

サービスで使用させたいものと同じ `--profile`／環境でそのコマンドを実行してください。

<div id="what-does-another-gateway-instance-is-already-listening-mean">
  ### another gateway instance is already listening とはどういう意味ですか
</div>

OpenClaw は、起動時にすぐ WebSocket リスナーをバインドすることでランタイムロックを行います（デフォルトは `ws://127.0.0.1:18789`）。このバインドが `EADDRINUSE` で失敗した場合、すでに別のインスタンスが待ち受けていることを示す `GatewayLockError` をスローします。

対処: 他のインスタンスを停止する、ポートを解放する、または `openclaw gateway --port <port>` で別ポートを指定して実行します。

<div id="how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere">
  ### OpenClaw をリモートモードで実行し、別の場所の Gateway にクライアントとして接続するにはどうすればよいですか
</div>

`gateway.mode: "remote"` を設定し、必要に応じてトークンやパスワードを付与して、リモートの WebSocket URL を指定します。

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      password: "your-password"
    }
  }
}
```

注意:

* `gateway.mode` が `local` の場合にのみ（またはオーバーライド用フラグを渡した場合に）`openclaw gateway` が起動します。
* macOS アプリは設定ファイルを監視しており、これらの値が変更されたときに、その場でモードを切り替えます。

<div id="the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now">
  ### Control UI に「unauthorized」と出る／再接続を繰り返す場合はどうすればよいですか
</div>

Gateway が認証有効（`gateway.auth.*`）で動いていますが、UI から一致するトークン／パスワードが送信されていません。

前提（コードから分かること）:

* Control UI はブラウザの localStorage キー `openclaw.control.settings.v1` にトークンを保存します。
* UI は `?token=...`（および／または `?password=...`）を一度だけ取り込み、その後は URL から削除します。

対処方法:

* 最速の方法: `openclaw dashboard`（トークン付きリンクを表示＋クリップボードにコピーし、可能ならブラウザを開く。ヘッドレスの場合は SSH 経由でのアクセス方法のヒントを表示）。
* まだトークンがない場合: `openclaw doctor --generate-gateway-token` を実行します。
* リモート接続の場合は、まずトンネルを張ります: `ssh -N -L 18789:127.0.0.1:18789 user@host` を実行し、その後 `http://127.0.0.1:18789/?token=...` を開きます。
* Gateway ホスト側で `gateway.auth.token`（または `OPENCLAW_GATEWAY_TOKEN`）を設定します。
* Control UI の設定画面で、同じトークンを貼り付ける（または一度きりの `?token=...` リンクで再設定する）。
* それでも解決しない場合は `openclaw status --all` を実行し、[Troubleshooting](/ja/gateway/troubleshooting) に従ってください。認証の詳細は [Dashboard](/ja/web/dashboard) を参照してください。

<div id="i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens">
  ### gatewaybind を tailnet に設定したが、バインドできず何も待ち受けていない
</div>

`tailnet` バインドは、ネットワークインターフェース上の Tailscale IP (100.64.0.0/10) を 1 つ選びます。マシンが Tailscale 上にない（あるいはインターフェースがダウンしている）場合、バインド先が存在しません。

対処方法:

* そのホストで Tailscale を起動する（100.x アドレスを持たせる）、または
* `gateway.bind: "loopback"` / `"lan"` に切り替える。

注意: `tailnet` は明示的な指定です。`auto` は loopback を優先します。tailnet のみへのバインドにしたい場合は `gateway.bind: "tailnet"` を使用してください。

<div id="can-i-run-multiple-gateways-on-the-same-host">
  ### 同一ホスト上で複数の Gateway を動かせますか
</div>

通常は推奨されません。1つの Gateway で複数のメッセージングチャネルとエージェントを動かせます。冗長構成（例: レスキューボット）や厳密な分離が必要な場合のみ、複数の Gateway を使ってください。

可能ですが、次を分離する必要があります:

* `OPENCLAW_CONFIG_PATH`（インスタンスごとの設定）
* `OPENCLAW_STATE_DIR`（インスタンスごとの状態）
* `agents.defaults.workspace`（ワークスペースの分離）
* `gateway.port`（一意のポート）

クイックセットアップ（推奨）:

* インスタンスごとに `openclaw --profile <name> …` を使う（自動的に `~/.openclaw-<name>` を作成）。
* 各プロファイル設定で一意の `gateway.port` を設定する（または手動実行時に `--port` を渡す）。
* プロファイルごとのサービスをインストールする: `openclaw --profile <name> gateway install`。

プロファイルはサービス名にもサフィックスとして付与されます（`bot.molt.<profile>`、レガシーな `com.openclaw.*`、`openclaw-gateway-<profile>.service`、`OpenClaw Gateway (<profile>)` など）。
詳細ガイド: [Multiple gateways](/ja/gateway/multiple-gateways)。

<div id="what-does-invalid-handshake-code-1008-mean">
  ### 無効なハンドシェイクコード 1008 はどういう意味ですか
</div>

Gateway は **WebSocket サーバー**であり、最初のメッセージとして
`connect` フレームが送られてくることを期待しています。別のものを受信した場合は、
**コード 1008**（ポリシー違反）で接続を閉じます。

よくある原因:

* WS クライアントではなく、ブラウザで **HTTP** の URL（`http://...`）を開いている。
* ポートまたはパスを間違えている。
* プロキシやトンネルが認証ヘッダーを削除してしまった、あるいは Gateway 向けではないリクエストを送信した。

簡単な対処方法:

1. WS の URL を使う: `ws://<host>:18789`（HTTPS の場合は `wss://...`）。
2. 通常のブラウザタブで WS ポートを直接開かない。
3. 認証が有効な場合は、`connect` フレームにトークン／パスワードを含める。

CLI または TUI を使用している場合、URL は次のような形式になります:

```
openclaw tui --url ws://<host>:18789 --token <token>
```

詳細は [Gateway プロトコル](/ja/gateway/protocol) を参照してください。

<div id="logging-and-debugging">
  ## ログとデバッグ
</div>

<div id="where-are-logs">
  ### ログの保存場所
</div>

ファイルログ（構造化形式）:

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

`logging.file` でログファイルのパスを固定できます。ファイルのログレベルは `logging.level` で制御します。コンソール出力の詳細度は `--verbose` と `logging.consoleLevel` で制御します。

ログを最速で tail するには:

```bash
openclaw logs --follow
```

サービス / スーパーバイザのログ（Gateway を launchd/systemd 経由で実行している場合）:

* macOS: `$OPENCLAW_STATE_DIR/logs/gateway.log` と `gateway.err.log`（デフォルト: `~/.openclaw/logs/...`。プロファイル使用時は `~/.openclaw-<profile>/logs/...`）
* Linux: `journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`
* Windows: `schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST`

詳細は [トラブルシューティング](/ja/gateway/troubleshooting#log-locations) を参照してください。

<div id="how-do-i-startstoprestart-the-gateway-service">
  ### Gateway サービスの開始・停止・再起動方法
</div>

Gateway のヘルパーを使用してください:

```bash
openclaw gateway status
openclaw gateway restart
```

Gateway を手動で実行している場合は、`openclaw gateway --force` を実行するとポートを強制的に確保し直せます。詳細は [Gateway](/ja/gateway) を参照してください。

<div id="i-closed-my-terminal-on-windows-how-do-i-restart-openclaw">
  ### Windows でターミナルを閉じてしまいました。OpenClaw を再起動するにはどうすればよいですか？
</div>

**2 つの Windows インストールモード**があります：

**1) WSL2（推奨）：** Gateway は Linux 上で動作します。

PowerShell を開き、WSL に入ってから再起動します：

```powershell
wsl
openclaw gateway status
openclaw gateway restart
```

サービスとしてインストールしていない場合は、フォアグラウンドで起動してください:

```bash
openclaw gateway run
```

**2) ネイティブ Windows（非推奨）：** Gateway を Windows 上で直接実行します。

PowerShell を開き、次を実行してください：

```powershell
openclaw gateway status
openclaw gateway restart
```

サービスとしてではなく手動で実行する場合は、次を実行してください：

```powershell
openclaw gateway run
```

ドキュメント：[Windows (WSL2)](/ja/platforms/windows)、[Gateway サービス運用ランブック](/ja/gateway)。

<div id="the-gateway-is-up-but-replies-never-arrive-what-should-i-check">
  ### Gateway は起動しているのに応答が返ってきません 何を確認すればよいですか
</div>

まずは簡単なヘルスチェックから始めましょう:

```bash
openclaw status
openclaw models status
openclaw channels status
openclaw logs --follow
```

主な原因:

* **Gateway ホスト**上でモデルの認証情報が読み込まれていない（`models status` を確認）。
* チャネルのペアリング/許可リストが返信をブロックしている（チャネル設定とログを確認）。
* WebChat/Dashboard が正しいトークンを付与せずに開かれている。

リモートから利用している場合は、トンネル/Tailscale 接続が確立されており、
Gateway の WebSocket に到達できることを確認してください。

ドキュメント: [Channels](/ja/channels)、[Troubleshooting](/ja/gateway/troubleshooting)、[Remote access](/ja/gateway/remote).

<div id="disconnected-from-gateway-no-reason-what-now">
  ### 理由もなく Gateway から切断された 次に何をすればいい？
</div>

これは多くの場合、UI が WebSocket 接続を失ったことを意味します。次を確認してください。

1. Gateway は動作していますか？ `openclaw gateway status`
2. Gateway は正常ですか？ `openclaw status`
3. UI は正しいトークンを持っていますか？ `openclaw dashboard`
4. リモート接続の場合、トンネル/Tailscale のリンクは生きていますか？

次にログを tail します:

```bash
openclaw logs --follow
```

ドキュメント: [ダッシュボード](/ja/web/dashboard)、[リモートアクセス](/ja/gateway/remote)、[トラブルシューティング](/ja/gateway/troubleshooting)。

<div id="telegram-setmycommands-fails-with-network-errors-what-should-i-check">
  ### Telegram の setMyCommands がネットワークエラーで失敗します。何を確認すべきですか
</div>

まずはログとチャネルのステータスを確認してください：

```bash
openclaw channels status
openclaw channels logs --channel telegram
```

VPS 上で実行している場合やプロキシ配下にある場合は、送信方向の HTTPS 通信が許可されており、DNS が正しく機能していることを確認してください。
Gateway がリモートにある場合は、必ず Gateway ホスト上のログを確認してください。

ドキュメント: [Telegram](/ja/channels/telegram), [チャネルのトラブルシューティング](/ja/channels/troubleshooting).

<div id="tui-shows-no-output-what-should-i-check">
  ### TUI に出力が表示されません 何を確認すればよいですか
</div>

まず、Gateway に接続できていて、エージェントが実行可能な状態であることを確認します。

```bash
openclaw status
openclaw models status
openclaw logs --follow
```

TUI では、`/status` を使って現在の状態を確認します。チャットチャネルで返信を受け取ることを想定している場合は、配信が有効になっていることを確認してください（`/deliver on`）。

ドキュメント: [TUI](/ja/tui), [スラッシュコマンド](/ja/tools/slash-commands).

<div id="how-do-i-completely-stop-then-start-the-gateway">
  ### Gateway を完全に停止してから再起動するには
</div>

サービスとしてインストールしている場合:

```bash
openclaw gateway stop
openclaw gateway start
```

これは、**supervised service**（macOS では launchd、Linux では systemd）を停止／起動します。
Gateway がデーモンとしてバックグラウンドで動作している場合に使用します。

フォアグラウンドで実行している場合は、Ctrl‑C で停止してから、次のコマンドを実行します:

```bash
openclaw gateway run
```

ドキュメント： [Gateway サービス運用ランブック](/ja/gateway)。

<div id="eli5-openclaw-gateway-restart-vs-openclaw-gateway">
  ### ELI5 openclaw gateway restart と openclaw gateway の違い
</div>

* `openclaw gateway restart`: **バックグラウンドサービス**（launchd/systemd）を再起動します。
* `openclaw gateway`: このターミナルセッションで Gateway を**フォアグラウンドで実行**します。

サービスをインストールしている場合は、Gateway コマンドを使ってください。単発でフォアグラウンド実行したいときは `openclaw gateway` を使います。

<div id="whats-the-fastest-way-to-get-more-details-when-something-fails">
  ### 何かが失敗したときに、より詳しい情報を得る最速の方法は？
</div>

Gateway を `--verbose` 付きで起動してコンソールの詳細出力を有効にします。そのうえで、チャネル認証、モデルルーティング、RPC エラーに関するログファイルを確認します。

<div id="media-attachments">
  ## メディアと添付ファイル
</div>

<div id="my-skill-generated-an-imagepdf-but-nothing-was-sent">
  ### スキルで imagePDF を生成したが、何も送信されない
</div>

エージェントから外部に送信する添付ファイルには、`MEDIA:&lt;path-or-url&gt;` という行を（それだけを 1 行として）含める必要があります。詳しくは [OpenClaw assistant setup](/ja/start/openclaw) および [Agent send](/ja/tools/agent-send) を参照してください。

CLI での送信:

```bash
openclaw message send --target +15555550123 --message "Here you go" --media /path/to/file.png
```

あわせて以下も確認してください:

* 対象チャネルがメディアの送信をサポートしており、許可リストによってブロックされていないこと。
* ファイルがプロバイダーのサイズ上限内であること（画像は最大 2048px にリサイズされます）。

[Images](/ja/nodes/images) も参照してください。

<div id="security-and-access-control">
  ## セキュリティとアクセス管理
</div>

<div id="is-it-safe-to-expose-openclaw-to-inbound-dms">
  ### OpenClaw を受信 DM に公開しても安全ですか
</div>

受信 DM は信頼できない入力として扱ってください。デフォルト設定はリスクを減らすように設計されています。

* DM 対応チャネルのデフォルトの挙動は **ペアリング** です:
  * 不明な送信者にはペアリングコードが送信され、ボットはそのメッセージを処理しません。
  * 次で承認します: `openclaw pairing approve <channel> <code>`
  * 保留中のリクエストは **チャネルごとに最大 3 件** に制限されます。コードが届かない場合は `openclaw pairing list <channel>` を確認してください。
* DM を誰からでも受け付けるように公開するには、明示的なオプトインが必要です（`dmPolicy: "open"` と許可リスト `"*"` を設定すると、任意のユーザーからの DM を無制限に受け付ける状態になります）。

`openclaw doctor` を実行して、リスクの高い DM ポリシーを洗い出してください。

<div id="is-prompt-injection-only-a-concern-for-public-bots">
  ### プロンプトインジェクションは公開ボットだけの懸念事項ですか
</div>

いいえ。プロンプトインジェクションの問題は、**信用できないコンテンツ**に関するものであり、ボットに誰が DM を送信できるかだけの話ではありません。
アシスタントが外部コンテンツ（ウェブ検索／フェッチ、ブラウザページ、メール、
ドキュメント、添付ファイル、貼り付けたログ）を読む場合、そのコンテンツにモデルを乗っ取ろうとする指示が含まれている可能性があります。これは、**送信者があなただけの場合でも**起こり得ます。

最大のリスクはツールが有効なときです。モデルがだまされて、コンテキストを外部に流出させたり、あなたの代わりにツールを呼び出したりする可能性があります。被害範囲を小さくするには、次のようにします。

* 信用できないコンテンツを要約するために、読み取り専用またはツール無効の「reader」エージェントを使う
* ツール有効なエージェントでは `web_search` / `web_fetch` / `browser` をオフにしておく
* サンドボックス化し、厳格なツール許可リストを設定する

詳細: [Security](/ja/gateway/security).

<div id="should-my-bot-have-its-own-email-github-account-or-phone-number">
  ### ボット用のメールアドレス、GitHub アカウント、電話番号を用意すべきですか
</div>

ほとんどの構成では、用意したほうがよいです。ボットを個別のアカウントや電話番号で分離しておくと、問題が起きた場合の被害範囲を小さくできます。これにより、個人アカウントに影響を与えずに認証情報のローテーションやアクセス権の取り消しを行いやすくなります。

まずは小さく始めてください。実際に必要なツールやアカウントにだけアクセス権を与え、必要に応じて後から拡張してください。

ドキュメント: [Security](/ja/gateway/security), [Pairing](/ja/start/pairing).

<div id="can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe">
  ### テキストメッセージを完全に自動化させても安全ですか
</div>

個人メッセージを完全に自動で処理させることは**推奨しません**。最も安全なパターンは次のとおりです。

* DM は **ペアリングモード** か、厳しく絞った許可リストに保つ。
* 代理でメッセージを送らせたい場合は、**別の電話番号やアカウント** を使う。
* 下書きまでは任せて、**送信前に必ず自分で承認する**。

試したい場合は、専用のアカウントで行い、他と分離してください。\
詳しくは [Security](/ja/gateway/security) を参照してください。

<div id="can-i-use-cheaper-models-for-personal-assistant-tasks">
  ### 個人用アシスタントのタスクに、より安価なモデルを使えますか
</div>

はい、**条件付きで可能です**。エージェントがチャット専用で、かつ入力が信頼できる場合に限ります。下位グレードのモデルはプロンプト／指示ハイジャックの影響を受けやすいため、ツールが有効なエージェントや、信頼できないコンテンツを読み込む場合には使用を避けてください。どうしても小さいモデルを使う必要がある場合は、利用可能なツールを最小限に絞り、サンドボックス内で実行してください。[Security](/ja/gateway/security) も参照してください。

<div id="i-ran-start-in-telegram-but-didnt-get-a-pairing-code">
  ### Telegram で /start を実行したのにペアリングコードが届きません
</div>

ペアリングコードが送られるのは、**未知の送信者** がボットにメッセージを送り、
`dmPolicy: "pairing"` が有効になっている場合のみです。`/start` コマンドだけではコードは生成されません。

保留中のリクエストを確認してください：

```bash
openclaw pairing list telegram
```

即時にアクセスできるようにしたい場合は、そのアカウントの送信元 ID を許可リストに追加するか、`dmPolicy: "open"`（全ユーザーからのDMを無制限に受け付ける設定）を設定してください。

<div id="whatsapp-will-it-message-my-contacts-how-does-pairing-work">
  ### WhatsApp は自分の連絡先にメッセージを送ってしまいますか ペアリングはどのように動作しますか
</div>

いいえ。デフォルトの WhatsApp DM ポリシーは**ペアリング**です。不明な送信者はペアリングコードを受け取るだけで、そのメッセージは**処理されません**。OpenClaw が返信するのは、自身が受信したチャットか、あなたが明示的にトリガーした送信に対してのみです。

ペアリングを承認するには、次を実行します:

```bash
openclaw pairing approve whatsapp <code>
```

保留中のリクエスト一覧:

```bash
openclaw pairing list whatsapp
```

ウィザードの電話番号プロンプト: これはあなたの **許可リスト/オーナー** を設定し、自分宛ての DM を許可するために使われます。自動送信のためには使用されません。自分の個人用 WhatsApp 番号で実行する場合は、その番号を指定し、`channels.whatsapp.selfChatMode` を有効にしてください。

<div id="chat-commands-aborting-tasks-and-it-wont-stop">
  ## チャットコマンド、タスクの中断、「止まらない」場合
</div>

<div id="how-do-i-stop-internal-system-messages-from-showing-in-chat">
  ### チャットに内部システムメッセージを表示させないようにするには
</div>

ほとんどの内部メッセージやツールメッセージは、そのセッションで **verbose** または **reasoning** が有効になっている場合にのみ表示されます。

そのメッセージが表示されているチャットで、次のように設定を変更します:

```
/verbose off
/reasoning off
```

それでもまだ出力がうるさい場合は、Control UI でセッション設定を確認し、`verbose` を **inherit** に設定してください。あわせて、設定ファイルで `verboseDefault` が `on` に設定された bot プロファイルを使用していないことも確認してください。

ドキュメント: [Thinking and verbose](/ja/tools/thinking), [Security](/ja/gateway/security#reasoning--verbose-output-in-groups).

<div id="how-do-i-stopcancel-a-running-task">
  ### 実行中のタスクを停止／キャンセルするにはどうすればよいですか
</div>

以下のいずれかを**単独のメッセージとして**（スラッシュなしで）送信してください:

```
stop
abort
esc
wait
exit
interrupt
```

これらは中断用のトリガーです（スラッシュコマンドではありません）。

バックグラウンドプロセス（`exec` ツールから起動したもの）の場合、エージェントに次のようなコマンドを実行させることができます:

```
process action:kill sessionId:XXX
```

スラッシュコマンドの概要については、[Slash commands](/ja/tools/slash-commands) を参照してください。

ほとんどのコマンドは、メッセージの先頭を `/` から始める **単独の** メッセージとして送信する必要がありますが、`/status` のようないくつかのショートカットは、許可リストに登録された送信者であればインラインでも利用できます。

<div id="how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied">
  ### Telegram から Discord にメッセージを送信しようとすると &quot;Crosscontext messaging denied&quot; になるのはなぜですか
</div>

OpenClaw は、デフォルトで **クロスプロバイダー間**メッセージングをブロックします。ツール呼び出しが Telegram にバインドされている場合、明示的に許可しない限り Discord には送信されません。

エージェントでクロスプロバイダー間メッセージングを有効にします:

```json5
{
  agents: {
    defaults: {
      tools: {
        message: {
          crossContext: {
            allowAcrossProviders: true,
            marker: { enabled: true, prefix: "[from {channel}] " }
          }
        }
      }
    }
  }
}
```

設定を編集したら Gateway を再起動してください。特定の 1 つのエージェントにのみ適用したい場合は、代わりに `agents.list[].tools.message` 配下で設定してください。

<div id="why-does-it-feel-like-the-bot-ignores-rapidfire-messages">
  ### なぜボットが連投メッセージを無視しているように感じるのか
</div>

キューモードは、新しいメッセージが実行中のランとどう連携するかを制御します。`/queue` を使ってモードを変更できます:

* `steer` - 新しいメッセージで現在のタスクの進行を切り替える
* `followup` - メッセージを1件ずつ順番に実行する
* `collect` - メッセージをまとめて1回だけ返信する（デフォルト）
* `steer-backlog` - まず今の実行を steer し、その後バックログを処理する
* `interrupt` - 現在のランを中断して新しく始める

`debounce:2s cap:25 drop:summarize` のようなオプションを followup 系モードに追加できます。

<div id="answer-the-exact-question-from-the-screenshotchat-log">
  ## スクリーンショット／チャットログにある質問そのものに答える
</div>

**Q: 「Anthropic の API キーを使う場合のデフォルトモデルは何ですか？」**

**A:** OpenClaw では、認証情報とモデル選択は別々に扱われます。`ANTHROPIC_API_KEY` を設定する（または auth プロファイルに Anthropic の API キーを保存する）と認証は有効になりますが、実際のデフォルトモデルは、`agents.defaults.model.primary` に設定したものになります（例: `anthropic/claude-sonnet-4-5` や `anthropic/claude-opus-4-5`）。`No credentials found for profile "anthropic:default"` というメッセージが表示される場合は、実行中のエージェントが参照している `auth-profiles.json` 内に、期待される Anthropic の認証情報を Gateway が見つけられなかったことを意味します。

***

まだ解決しない場合は、[Discord](https://discord.com/invite/clawd) で質問するか、[GitHub Discussion](https://github.com/openclaw/openclaw/discussions) でスレッドを作成してください。