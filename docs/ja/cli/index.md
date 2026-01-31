---
title: "CLI"
summary: "OpenClaw CLI リファレンス（`openclaw` コマンド、そのサブコマンドおよびオプション）"
read_when:
  - CLI コマンドやオプションを追加・変更する場合
  - 新しいコマンド群を文書化するとき
---

<div id="cli-reference">
  # CLI リファレンス
</div>

このページでは CLI の現在の動作について説明します。コマンドが変更された場合は、このドキュメントを更新してください。

<div id="command-pages">
  ## コマンドページ一覧
</div>

* [`setup`](/ja/cli/setup)
* [`onboard`](/ja/cli/onboard)
* [`configure`](/ja/cli/configure)
* [`config`](/ja/cli/config)
* [`doctor`](/ja/cli/doctor)
* [`dashboard`](/ja/cli/dashboard)
* [`reset`](/ja/cli/reset)
* [`uninstall`](/ja/cli/uninstall)
* [`update`](/ja/cli/update)
* [`message`](/ja/cli/message)
* [`agent`](/ja/cli/agent)
* [`agents`](/ja/cli/agents)
* [`acp`](/ja/cli/acp)
* [`status`](/ja/cli/status)
* [`health`](/ja/cli/health)
* [`sessions`](/ja/cli/sessions)
* [`gateway`](/ja/cli/gateway)
* [`logs`](/ja/cli/logs)
* [`system`](/ja/cli/system)
* [`models`](/ja/cli/models)
* [`memory`](/ja/cli/memory)
* [`nodes`](/ja/cli/nodes)
* [`devices`](/ja/cli/devices)
* [`node`](/ja/cli/node)
* [`approvals`](/ja/cli/approvals)
* [`sandbox`](/ja/cli/sandbox)
* [`tui`](/ja/cli/tui)
* [`browser`](/ja/cli/browser)
* [`cron`](/ja/cli/cron)
* [`dns`](/ja/cli/dns)
* [`docs`](/ja/cli/docs)
* [`hooks`](/ja/cli/hooks)
* [`webhooks`](/ja/cli/webhooks)
* [`pairing`](/ja/cli/pairing)
* [`plugins`](/ja/cli/plugins)（プラグイン関連コマンド）
* [`channels`](/ja/cli/channels)
* [`security`](/ja/cli/security)
* [`skills`](/ja/cli/skills)
* [`voicecall`](/ja/cli/voicecall)（プラグイン。インストールされている場合のみ）

<div id="global-flags">
  ## グローバルフラグ
</div>

* `--dev`: 状態を `~/.openclaw-dev` 以下に分離し、デフォルトポートを変更します。
* `--profile <name>`: 状態を `~/.openclaw-<name>` 以下に分離します。
* `--no-color`: ANSI カラーを無効にします。
* `--update`: `openclaw update` の省略形です（ソースコードからのインストール時のみ）。
* `-V`, `--version`, `-v`: バージョンを表示して終了します。

<div id="output-styling">
  ## 出力スタイル
</div>

* ANSI カラーと進行状況インジケーターは、TTY セッションでのみ表示されます。
* OSC-8 ハイパーリンクは、対応しているターミナルではクリック可能なリンクとして表示され、それ以外の場合はプレーンな URL にフォールバックします。
* `--json`（およびサポートされている場合は `--plain`）は、スタイル適用を無効化してクリーンな出力にします。
* `--no-color` は ANSI スタイリングを無効化します。`NO_COLOR=1` も有効です。
* 長時間実行されるコマンドでは、（対応している場合は OSC 9;4 による）進行状況インジケーターが表示されます。

<div id="color-palette">
  ## カラーパレット
</div>

OpenClaw は CLI 出力にロブスター調のカラーパレットを使用します。

* `accent` (#FF5A2D): 見出し、ラベル、主要なハイライト。
* `accentBright` (#FF7A3D): コマンド名、強調。
* `accentDim` (#D14A22): 補助的なハイライトテキスト。
* `info` (#FF8A5B): 情報系の値。
* `success` (#2FBF71): 成功状態。
* `warn` (#FFB020): 警告、フォールバック、注意喚起。
* `error` (#E23D2D): エラー、失敗。
* `muted` (#8B7F77): 抑えた表示、メタデータ。

パレットのソース・オブ・トゥルース（正準的な定義元）: `src/terminal/palette.ts`（別名 “lobster seam”）。

<div id="command-tree">
  ## コマンドツリー
</div>

```
openclaw [--dev] [--profile <name>] <command>
  setup
  onboard
  configure
  config
    get
    set
    unset
  doctor
  security
    audit
  reset
  uninstall
  update
  channels
    list
    status
    logs
    add
    remove
    login
    logout
  skills
    list
    info
    check
  plugins
    list
    info
    install
    enable
    disable
    doctor
  memory
    status
    index
    search
  message
  agent
  agents
    list
    add
    delete
  acp
  status
  health
  sessions
  gateway
    call
    health
    status
    probe
    discover
    install
    uninstall
    start
    stop
    restart
    run
  logs
  system
    event
    heartbeat last|enable|disable
    presence
  models
    list
    status
    set
    set-image
    aliases list|add|remove
    fallbacks list|add|remove|clear
    image-fallbacks list|add|remove|clear
    scan
    auth add|setup-token|paste-token
    auth order get|set|clear
  sandbox
    list
    recreate
    explain
  cron
    status
    list
    add
    edit
    rm
    enable
    disable
    runs
    run
  nodes
  devices
  node
    run
    status
    install
    uninstall
    start
    stop
    restart
  approvals
    get
    set
    allowlist add|remove
  browser
    status
    start
    stop
    reset-profile
    tabs
    open
    focus
    close
    profiles
    create-profile
    delete-profile
    screenshot
    snapshot
    navigate
    resize
    click
    type
    press
    hover
    drag
    select
    upload
    fill
    dialog
    wait
    evaluate
    console
    pdf
  hooks
    list
    info
    check
    enable
    disable
    install
    update
  webhooks
    gmail setup|run
  pairing
    list
    approve
  docs
  dns
    setup
  tui
```

注記: プラグインによっては、追加のトップレベルコマンド (例: `openclaw voicecall`) を提供する場合があります。

<div id="security">
  ## セキュリティ
</div>

* `openclaw security audit` — よくあるセキュリティ上の落とし穴がないか、設定とローカル状態を監査します。
* `openclaw security audit --deep` — Gateway に対するベストエフォートのライブプローブを実行します。
* `openclaw security audit --fix` — 安全なデフォルトを強化し、状態および設定ファイルのパーミッション (`chmod`) を適切に設定します。

<div id="plugins">
  ## プラグイン
</div>

拡張機能とその設定を管理します:

* `openclaw plugins list` — 利用可能なプラグインを一覧表示します（マシン向け出力には `--json` を使用）。
* `openclaw plugins info <id>` — 指定したプラグインの詳細を表示します。
* `openclaw plugins install <path|.tgz|npm-spec>` — プラグインをインストールします（またはプラグインのパスを `plugins.load.paths` に追加します）。
* `openclaw plugins enable <id>` / `disable <id>` — `plugins.entries.<id>.enabled` を切り替えます。
* `openclaw plugins doctor` — プラグインのロードエラーをレポートします。

ほとんどのプラグインに関する変更は Gateway の再起動が必要です。[/plugin](/ja/plugin) を参照してください。

<div id="memory">
  ## メモリ
</div>

`MEMORY.md` と `memory/*.md` に対するベクター検索:

* `openclaw memory status` — インデックスの統計情報を表示します。
* `openclaw memory index` — メモリファイルを再インデックス化します。
* `openclaw memory search "<query>"` — メモリに対してセマンティック検索を実行します。

<div id="chat-slash-commands">
  ## チャットのスラッシュコマンド
</div>

チャットメッセージでは `/...` コマンド（テキストおよびネイティブ）を利用できます。[/tools/slash-commands](/ja/tools/slash-commands) を参照してください。

主なコマンド:

* `/status` は簡易診断用。
* `/config` は永続化される設定変更用。
* `/debug` は実行時のみ有効な一時的設定オーバーライド用（メモリ上のみでディスクには書き込まれません。`commands.debug: true` が必要です）。

<div id="setup-onboarding">
  ## セットアップと導入
</div>

<div id="setup">
  ### `setup`
</div>

設定とワークスペースを初期化します。

Options:

* `--workspace <dir>`: エージェント用ワークスペースのパス（デフォルト `~/.openclaw/workspace`）。
* `--wizard`: オンボーディングウィザードを実行します。
* `--non-interactive`: プロンプトなしでウィザードを実行します。
* `--mode <local|remote>`: ウィザードのモード。
* `--remote-url <url>`: リモート Gateway の URL。
* `--remote-token <token>`: リモート Gateway のトークン。

`--non-interactive`、`--mode`、`--remote-url`、`--remote-token` など、いずれかのウィザード関連フラグが指定されている場合、ウィザードは自動的に実行されます。

<div id="onboard">
  ### `onboard`
</div>

Gateway、ワークスペース、スキルをセットアップするための対話型ウィザード。

オプション:

* `--workspace <dir>`
* `--reset` (ウィザード実行前に設定・認証情報・セッション・ワークスペースをリセット)
* `--non-interactive`
* `--mode <local|remote>`
* `--flow <quickstart|advanced|manual>` (manual は advanced のエイリアス)
* `--auth-choice <setup-token|token|chutes|openai-codex|openai-api-key|openrouter-api-key|ai-gateway-api-key|moonshot-api-key|kimi-code-api-key|synthetic-api-key|venice-api-key|gemini-api-key|zai-api-key|apiKey|minimax-api|minimax-api-lightning|opencode-zen|skip>`
* `--token-provider <id>` (非対話モード; `--auth-choice token` と併用)
* `--token <token>` (非対話モード; `--auth-choice token` と併用)
* `--token-profile-id <id>` (非対話モード; デフォルト: `<provider>:manual`)
* `--token-expires-in <duration>` (非対話モード; 例: `365d`, `12h`)
* `--anthropic-api-key <key>`
* `--openai-api-key <key>`
* `--openrouter-api-key <key>`
* `--ai-gateway-api-key <key>`
* `--moonshot-api-key <key>`
* `--kimi-code-api-key <key>`
* `--gemini-api-key <key>`
* `--zai-api-key <key>`
* `--minimax-api-key <key>`
* `--opencode-zen-api-key <key>`
* `--gateway-port <port>`
* `--gateway-bind <loopback|lan|tailnet|auto|custom>`
* `--gateway-auth <token|password>`
* `--gateway-token <token>`
* `--gateway-password <password>`
* `--remote-url <url>`
* `--remote-token <token>`
* `--tailscale <off|serve|funnel>`
* `--tailscale-reset-on-exit`
* `--install-daemon`
* `--no-install-daemon` (エイリアス: `--skip-daemon`)
* `--daemon-runtime <node|bun>`
* `--skip-channels`
* `--skip-skills`
* `--skip-health`
* `--skip-ui`
* `--node-manager <npm|pnpm|bun>` (pnpm 推奨; Gateway ランタイムでは bun は非推奨)
* `--json`

<div id="configure">
  ### `configure`
</div>

モデル、チャネル、スキル、Gateway を対話的に設定するウィザード。

<div id="config">
  ### `config`
</div>

非対話型の設定ヘルパー（get/set/unset）。サブコマンドなしで `openclaw config`
を実行するとウィザードが起動します。

サブコマンド:

* `config get <path>`: 設定値を出力します（ドット/ブラケット表記のパス）。
* `config set <path> <value>`: 値を設定します（JSON5 またはそのままの文字列）。
* `config unset <path>`: 値を削除します。

<div id="doctor">
  ### `doctor`
</div>

ヘルスチェックと簡易修正（config + Gateway + レガシーサービス）。

Options:

* `--no-workspace-suggestions`: ワークスペースのメモリーヒントを無効にする。
* `--yes`: 確認プロンプトなしでデフォルトを受け入れる（ヘッドレス）。
* `--non-interactive`: プロンプトをスキップし、安全なマイグレーションのみ適用する。
* `--deep`: システムサービスをスキャンして、追加の Gateway のインストールを検出する。

<div id="channel-helpers">
  ## チャンネル用ヘルパー
</div>

<div id="channels">
  ### `channels`
</div>

チャットチャネルアカウントを管理します (WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (プラグイン)/Signal/iMessage/MS Teams)。

サブコマンド:

* `channels list`: 設定済みチャネルと認証プロファイルを表示します。
* `channels status`: Gateway への到達性とチャネルのヘルス状態をチェックします（`--probe` は追加チェックを実行します。Gateway のヘルスチェックには `openclaw health` または `openclaw status --deep` を使用してください）。
* ヒント: `channels status` は、よくある誤設定を検出できた場合に、推奨される修正方法付きの警告を表示し、その上で `openclaw doctor` の実行を促します。
* `channels logs`: Gateway のログファイルから最近のチャネルログを表示します。
* `channels add`: フラグが渡されていない場合はウィザード形式でセットアップを行います。フラグを付けると非対話モードに切り替わります。
* `channels remove`: デフォルトでは無効化のみ行います。`--delete` を渡すと、確認プロンプトなしで設定エントリを削除します。
* `channels login`: 対話的なチャネルログインを行います（WhatsApp Web のみ）。
* `channels logout`: チャネルセッションからログアウトします（対応している場合）。

共通オプション:

* `--channel <name>`: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams`
* `--account <id>`: チャネルアカウント ID（デフォルト `default`）
* `--name <label>`: アカウントの表示名

`channels login` オプション:

* `--channel <channel>`（デフォルト `whatsapp`。`whatsapp`/`web` をサポート）
* `--account <id>`
* `--verbose`

`channels logout` オプション:

* `--channel <channel>`（デフォルト `whatsapp`）
* `--account <id>`

`channels list` オプション:

* `--no-usage`: モデルプロバイダーの利用状況/クォータのスナップショットをスキップします（OAuth/API バックエンドのみ）。
* `--json`: JSON を出力します（`--no-usage` が指定されていない限り利用状況も含みます）。

`channels logs` オプション:

* `--channel <name|all>`（デフォルト `all`）
* `--lines <n>`（デフォルト `200`）
* `--json`

詳細: [/concepts/oauth](/ja/concepts/oauth)

例:

```bash
openclaw channels add --channel telegram --account alerts --name "Alerts Bot" --token $TELEGRAM_BOT_TOKEN
openclaw channels add --channel discord --account work --name "Work Bot" --token $DISCORD_BOT_TOKEN
openclaw channels remove --channel discord --account work --delete
openclaw channels status --probe
openclaw status --deep
```

<div id="skills">
  ### `skills`
</div>

利用可能なスキルの一覧と詳細、および準備状況を表示します。

サブコマンド:

* `skills list`: スキルを一覧表示します (サブコマンド未指定時のデフォルト)。
* `skills info <name>`: 単一のスキルの詳細を表示します。
* `skills check`: 要件の充足状況 (ready / missing) の概要を表示します。

オプション:

* `--eligible`: 準備完了しているスキルのみを表示します。
* `--json`: JSON 形式で出力します (装飾なし)。
* `-v`, `--verbose`: 不足している要件の詳細を含めます。

ヒント: `npx clawhub` を使ってスキルの検索・インストール・同期を行えます。

<div id="pairing">
  ### `pairing`
</div>

各チャネルでの DM ペアリング要求を承認します。

サブコマンド:

* `pairing list <channel> [--json]`
* `pairing approve <channel> <code> [--notify]`

<div id="webhooks-gmail">
  ### `webhooks gmail`
</div>

Gmail Pub/Sub フックのセットアップおよび実行。[/automation/gmail-pubsub](/ja/automation/gmail-pubsub) を参照してください。

サブコマンド:

* `webhooks gmail setup` (`--account <email>` が必須。`--project`, `--topic`, `--subscription`, `--label`, `--hook-url`, `--hook-token`, `--push-token`, `--bind`, `--port`, `--path`, `--include-body`, `--max-bytes`, `--renew-minutes`, `--tailscale`, `--tailscale-path`, `--tailscale-target`, `--push-endpoint`, `--json` をサポート)
* `webhooks gmail run` (同じフラグを実行時に上書きするためのコマンド)

<div id="dns-setup">
  ### `dns setup`
</div>

広域ネットワーク向けディスカバリー用 DNS ヘルパー（CoreDNS + Tailscale）。[/gateway/discovery](/ja/gateway/discovery) を参照してください。

オプション：

* `--apply`: CoreDNS の設定をインストール／更新します（sudo が必要、macOS のみ）。

<div id="messaging-agent">
  ## メッセージング＋エージェント
</div>

<div id="message">
  ### `message`
</div>

送信メッセージおよびチャネル操作を一元的に扱います。

参照: [/cli/message](/ja/cli/message)

サブコマンド:

* `message send|poll|react|reactions|read|edit|delete|pin|unpin|pins|permissions|search|timeout|kick|ban`
* `message thread <create|list|reply>`
* `message emoji <list|upload>`
* `message sticker <send|upload>`
* `message role <info|add|remove>`
* `message channel <info|list>`
* `message member info`
* `message voice status`
* `message event <list|create>`

例:

* `openclaw message send --target +15555550123 --message "Hi"`
* `openclaw message poll --channel discord --target channel:123 --poll-question "Snack?" --poll-option Pizza --poll-option Sushi`

<div id="agent">
  ### `agent`
</div>

エージェントの 1 ターンを Gateway 経由（または `--local` によるローカル実行）で実行します。

必須:

* `--message <text>`

オプション:

* `--to <dest>`（セッションキーおよび任意の配信指定）
* `--session-id <id>`
* `--thinking <off|minimal|low|medium|high|xhigh>`（GPT-5.2 + Codex モデルのみ）
* `--verbose <on|full|off>`
* `--channel <whatsapp|telegram|discord|slack|mattermost|signal|imessage|msteams>`
* `--local`
* `--deliver`
* `--json`
* `--timeout <seconds>`

<div id="agents">
  ### `agents`
</div>

分離されたエージェント群（ワークスペース + 認証 + ルーティング）を管理します。

<div id="agents-list">
  #### `agents list`
</div>

設定済みのエージェント群を一覧表示します。

オプション:

* `--json`
* `--bindings`

<div id="agents-add-name">
  #### `agents add [name]`
</div>

新しい独立したエージェントを追加します。フラグ（または `--non-interactive`）が指定されていない場合は対話型ウィザードが実行されます。非対話モードでは `--workspace` が必須です。

オプション:

* `--workspace <dir>`
* `--model <id>`
* `--agent-dir <dir>`
* `--bind <channel[:accountId]>` (繰り返し指定可)
* `--non-interactive`
* `--json`

バインド指定は `channel[:accountId]` 形式を使用します。WhatsApp で `accountId` が省略された場合は、デフォルトのアカウント ID が使用されます。

<div id="agents-delete-id">
  #### `agents delete <id>`
</div>

エージェントを削除し、関連するワークスペースと状態をクリーンアップします。

オプション:

* `--force`
* `--json`

<div id="acp">
  ### `acp`
</div>

IDE と Gateway を接続する ACP ブリッジを起動します。

すべてのオプションと例については、[`acp`](/ja/cli/acp) を参照してください。

<div id="status">
  ### `status`
</div>

リンクされているセッションの正常性と最近の送信先を表示します。

Options:

* `--json`
* `--all` (詳細診断; 読み取り専用・貼り付け可能)
* `--deep` (チャネルを検査)
* `--usage` (モデルプロバイダーの使用量/クォータを表示)
* `--timeout <ms>`
* `--verbose`
* `--debug` (`--verbose` のエイリアス)

Notes:

* サマリーには、利用可能な場合は Gateway およびノードのホストサービス状態が含まれます。

<div id="usage-tracking">
  ### 利用状況トラッキング
</div>

OpenClaw は、OAuth/API 資格情報が利用可能な場合、プロバイダーの利用状況やクォータを表示できます。

表示先:

* `/status`（利用可能な場合、簡易なプロバイダー利用状況の行を追加）
* `openclaw status --usage`（プロバイダーごとの詳細な内訳を表示）
* macOS メニューバー（「Context」配下の「Usage」セクション）

補足:

* データはプロバイダーの利用状況エンドポイントから直接取得されます（推定値ではありません）。
* 対応プロバイダー: Anthropic、GitHub Copilot、OpenAI Codex OAuth、および該当プロバイダー用プラグインが有効な場合の Gemini CLI/Antigravity。
* 該当する資格情報が存在しない場合、利用状況は表示されません。
* 詳細は [利用状況トラッキング](/ja/concepts/usage-tracking) を参照してください。

<div id="health">
  ### `health`
</div>

実行中の Gateway からヘルスチェック情報を取得します。

オプション:

* `--json`
* `--timeout <ms>`
* `--verbose`

<div id="sessions">
  ### `sessions`
</div>

保存されている会話セッションの一覧を表示します。

オプション:

* `--json`
* `--verbose`
* `--store <path>`
* `--active <minutes>`

<div id="reset-uninstall">
  ## リセット／アンインストール
</div>

<div id="reset">
  ### `reset`
</div>

ローカルの設定および状態をリセットします（CLI 自体はアンインストールされません）。

オプション:

* `--scope <config|config+creds+sessions|full>`
* `--yes`
* `--non-interactive`
* `--dry-run`

注意:

* `--non-interactive` を使用する場合は、`--scope` と `--yes` が必須です。

<div id="uninstall">
  ### `uninstall`
</div>

Gateway サービスおよびローカルデータを削除します（CLI は残ります）。

オプション:

* `--service`
* `--state`
* `--workspace`
* `--app`
* `--all`
* `--yes`
* `--non-interactive`
* `--dry-run`

注意:

* `--non-interactive` には `--yes` と明示的なスコープ（または `--all`）が必要です。

## Gateway

<div id="gateway">
  ### `gateway`
</div>

WebSocket Gateway を起動します。

オプション:

* `--port <port>`
* `--bind <loopback|tailnet|lan|auto|custom>`
* `--token <token>`
* `--auth <token|password>`
* `--password <password>`
* `--tailscale <off|serve|funnel>`
* `--tailscale-reset-on-exit`
* `--allow-unconfigured`
* `--dev`
* `--reset` (開発用設定・認証情報・セッション・ワークスペースをリセット)
* `--force` (指定ポートで待ち受けている既存リスナーを強制終了)
* `--verbose`
* `--claude-cli-logs`
* `--ws-log <auto|full|compact>`
* `--compact` (`--ws-log compact` のエイリアス)
* `--raw-stream`
* `--raw-stream-path <path>`

<div id="gateway-service">
  ### `gateway service`
</div>

Gateway サービスを管理します (launchd/systemd/schtasks)。

サブコマンド:

* `gateway status` (デフォルトで Gateway RPC の疎通確認を実行)
* `gateway install` (サービスのインストール)
* `gateway uninstall`
* `gateway start`
* `gateway stop`
* `gateway restart`

注意:

* `gateway status` は、サービス側で解決されたポート/設定を使って、デフォルトで Gateway RPC の疎通確認を行います (`--url/--token/--password` で上書きできます)。
* `gateway status` はスクリプト用途向けに `--no-probe`、`--deep`、`--json` をサポートします。
* `gateway status` は、検出可能な場合にはレガシーまたは追加の Gateway サービスも表示します (`--deep` はシステムレベルのスキャンを追加します)。プロファイル名付きの OpenClaw サービスは第一級のサービスとして扱われ、「extra」とはマークされません。
* `gateway status` は、CLI が使用する設定ファイルパスと、サービス (サービス環境) 側がおそらく使用している設定ファイルパス、さらに解決済みのプローブ先 URL を出力します。
* `gateway install|uninstall|start|stop|restart` はスクリプト用途向けに `--json` をサポートします (デフォルトの出力は人間が読みやすい形式のままです)。
* `gateway install` のデフォルトは Node ランタイムです。bun は **推奨されません** (WhatsApp/Telegram の不具合があるため)。
* `gateway install` のオプション: `--port`, `--runtime`, `--token`, `--force`, `--json`。

<div id="logs">
  ### `logs`
</div>

RPC 経由で Gateway のファイルログを末尾追跡（tail）します。

Notes:

* TTY セッションではカラー表示付きの構造化ビューをレンダリングし、非 TTY 環境ではプレーンテキスト表示にフォールバックします。
* `--json` は行区切りの JSON（1 行あたり 1 件のログイベント）を出力します。

Examples:

```bash
openclaw logs --follow
openclaw logs --limit 200
openclaw logs --plain
openclaw logs --json
openclaw logs --no-color
```

<div id="gateway-subcommand">
  ### `gateway <subcommand>`
</div>

Gateway CLI ヘルパー（RPC サブコマンドで `--url`, `--token`, `--password`, `--timeout`, `--expect-final` を使用します）。

サブコマンド:

* `gateway call <method> [--params <json>]`
* `gateway health`
* `gateway status`
* `gateway probe`
* `gateway discover`
* `gateway install|uninstall|start|stop|restart`
* `gateway run`

代表的な RPC:

* `config.apply`（設定の検証 + 書き込み + 再起動 + 復帰）
* `config.patch`（部分更新をマージ + 再起動 + 復帰）
* `update.run`（アップデートを実行 + 再起動 + 復帰）

ヒント: すでに設定が存在する場合、`config.set`/`config.apply`/`config.patch` を直接呼び出すときは、`config.get` から取得した `baseHash` を渡してください。

<div id="models">
  ## モデル
</div>

フォールバック時の動作とスキャン戦略については、[/concepts/models](/ja/concepts/models) を参照してください。

推奨される Anthropic 認証方法 (setup-token):

```bash
claude setup-token
openclaw models auth setup-token --provider anthropic
openclaw models status
```

<div id="models-root">
  ### `models` (ルート)
</div>

`openclaw models` は `models status` のエイリアスです。

ルートオプション：

* `--status-json`（`models status --json` のエイリアス）
* `--status-plain`（`models status --plain` のエイリアス）

<div id="models-list">
  ### `models list`
</div>

オプション:

* `--all`
* `--local`
* `--provider <name>`
* `--json`
* `--plain`

<div id="models-status">
  ### `models status`
</div>

オプション:

* `--json`
* `--plain`
* `--check` (終了コード 1=有効期限切れ/不足、2=まもなく有効期限切れ)
* `--probe` (設定済み認証プロファイルへのライブプローブ)
* `--probe-provider <name>`
* `--probe-profile <id>` (複数指定またはカンマ区切り)
* `--probe-timeout <ms>`
* `--probe-concurrency <n>`
* `--probe-max-tokens <n>`

常に auth ストア内のプロファイルについて、認証の概要と OAuth の有効期限ステータスを表示します。
`--probe` はライブリクエストを送信します (トークンを消費し、レート制限を発生させる可能性があります)。

<div id="models-set-model">
  ### `models set <model>`
</div>

`agents.defaults.model.primary` を設定します。

<div id="models-set-image-model">
  ### `models set-image <model>`
</div>

`agents.defaults.imageModel.primary` を設定します。

<div id="models-aliases-listaddremove">
  ### `models aliases list|add|remove`
</div>

オプション:

* `list`: `--json`, `--plain`
* `add <alias> <model>`
* `remove <alias>`

<div id="models-fallbacks-listaddremoveclear">
  ### `models fallbacks list|add|remove|clear`
</div>

オプション：

* `list`: `--json`, `--plain`
* `add <model>`
* `remove <model>`
* `clear`

<div id="models-image-fallbacks-listaddremoveclear">
  ### `models image-fallbacks list|add|remove|clear`
</div>

オプション:

* `list`: `--json`, `--plain`
* `add <model>`
* `remove <model>`
* `clear`

<div id="models-scan">
  ### `models scan`
</div>

オプション:

* `--min-params <b>`
* `--max-age-days <days>`
* `--provider <name>`
* `--max-candidates <n>`
* `--timeout <ms>`
* `--concurrency <n>`
* `--no-probe`
* `--yes`
* `--no-input`
* `--set-default`
* `--set-image`
* `--json`

<div id="models-auth-addsetup-tokenpaste-token">
  ### `models auth add|setup-token|paste-token`
</div>

オプション:

* `add`: 対話的な認証支援ツール
* `setup-token`: `--provider <name>`（デフォルト `anthropic`）、`--yes`
* `paste-token`: `--provider <name>`、`--profile-id <id>`、`--expires-in <duration>`

<div id="models-auth-order-getsetclear">
  ### `models auth order get|set|clear`
</div>

オプション:

* `get`: `--provider <name>`, `--agent <id>`, `--json`
* `set`: `--provider <name>`, `--agent <id>`, `<profileIds...>`
* `clear`: `--provider <name>`, `--agent <id>`

<div id="system">
  ## システム
</div>

<div id="system-event">
  ### `system event`
</div>

システムイベントをキューに投入し、オプションでハートビートをトリガーします（Gateway RPC）。

必須:

* `--text <text>`

オプション:

* `--mode <now|next-heartbeat>`
* `--json`
* `--url`, `--token`, `--timeout`, `--expect-final`

<div id="system-heartbeat-lastenabledisable">
  ### `system heartbeat last|enable|disable`
</div>

ハートビートの制御（Gateway RPC）。

オプション:

* `--json`
* `--url`, `--token`, `--timeout`, `--expect-final`

<div id="system-presence">
  ### `system presence`
</div>

system presence のエントリを一覧表示します (Gateway RPC)。

オプション:

* `--json`
* `--url`, `--token`, `--timeout`, `--expect-final`

<div id="cron">
  ## Cron
</div>

スケジュールされたジョブを管理します（Gateway RPC 経由）。[/automation/cron-jobs](/ja/automation/cron-jobs) を参照してください。

サブコマンド:

* `cron status [--json]`
* `cron list [--all] [--json]`（デフォルトはテーブル出力。生データ出力には `--json` を使用）
* `cron add`（エイリアス: `create`。`--name` と、`--at` | `--every` | `--cron` のいずれか1つ、さらに `--system-event` | `--message` のペイロードをちょうど1つ指定する必要があります）
* `cron edit <id>`（フィールドをパッチ更新）
* `cron rm <id>`（エイリアス: `remove`, `delete`）
* `cron enable <id>`
* `cron disable <id>`
* `cron runs --id <id> [--limit <n>]`
* `cron run <id> [--force]`

すべての `cron` コマンドは `--url`, `--token`, `--timeout`, `--expect-final` を受け付けます。

<div id="node-host">
  ## ノードホスト
</div>

`node` は **ヘッドレスなノードホスト** を起動するか、バックグラウンドサービスとして実行・管理します。[`openclaw node`](/ja/cli/node) を参照してください。

サブコマンド:

* `node run --host <gateway-host> --port 18789`
* `node status`
* `node install [--host <gateway-host>] [--port <port>] [--tls] [--tls-fingerprint <sha256>] [--node-id <id>] [--display-name <name>] [--runtime <node|bun>] [--force]`
* `node uninstall`
* `node stop`
* `node restart`

<div id="nodes">
  ## ノード
</div>

`nodes` は Gateway と通信し、ペアリング済みのノードを操作します。[/nodes](/ja/nodes) を参照してください。

共通オプション:

* `--url`, `--token`, `--timeout`, `--json`

サブコマンド:

* `nodes status [--connected] [--last-connected <duration>]`
* `nodes describe --node <id|name|ip>`
* `nodes list [--connected] [--last-connected <duration>]`
* `nodes pending`
* `nodes approve <requestId>`
* `nodes reject <requestId>`
* `nodes rename --node <id|name|ip> --name <displayName>`
* `nodes invoke --node <id|name|ip> --command <command> [--params <json>] [--invoke-timeout <ms>] [--idempotency-key <key>]`
* `nodes run --node <id|name|ip> [--cwd <path>] [--env KEY=VAL] [--command-timeout <ms>] [--needs-screen-recording] [--invoke-timeout <ms>] <command...>` (Mac ノードまたはヘッドレスノードホスト)
* `nodes notify --node <id|name|ip> [--title <text>] [--body <text>] [--sound <name>] [--priority <passive|active|timeSensitive>] [--delivery <system|overlay|auto>] [--invoke-timeout <ms>]` (Mac のみ)

カメラ:

* `nodes camera list --node <id|name|ip>`
* `nodes camera snap --node <id|name|ip> [--facing front|back|both] [--device-id <id>] [--max-width <px>] [--quality <0-1>] [--delay-ms <ms>] [--invoke-timeout <ms>]`
* `nodes camera clip --node <id|name|ip> [--facing front|back] [--device-id <id>] [--duration <ms|10s|1m>] [--no-audio] [--invoke-timeout <ms>]`

キャンバス + 画面:

* `nodes canvas snapshot --node <id|name|ip> [--format png|jpg|jpeg] [--max-width <px>] [--quality <0-1>] [--invoke-timeout <ms>]`
* `nodes canvas present --node <id|name|ip> [--target <urlOrPath>] [--x <px>] [--y <px>] [--width <px>] [--height <px>] [--invoke-timeout <ms>]`
* `nodes canvas hide --node <id|name|ip> [--invoke-timeout <ms>]`
* `nodes canvas navigate <url> --node <id|name|ip> [--invoke-timeout <ms>]`
* `nodes canvas eval [<js>] --node <id|name|ip> [--js <code>] [--invoke-timeout <ms>]`
* `nodes canvas a2ui push --node <id|name|ip> (--jsonl <path> | --text <text>) [--invoke-timeout <ms>]`
* `nodes canvas a2ui reset --node <id|name|ip> [--invoke-timeout <ms>]`
* `nodes screen record --node <id|name|ip> [--screen <index>] [--duration <ms|10s>] [--fps <n>] [--no-audio] [--out <path>] [--invoke-timeout <ms>]`

位置情報:

* `nodes location get --node <id|name|ip> [--max-age <ms>] [--accuracy <coarse|balanced|precise>] [--location-timeout <ms>] [--invoke-timeout <ms>]`

<div id="browser">
  ## Browser
</div>

ブラウザ制御用 CLI（専用の Chrome/Brave/Edge/Chromium 用）。[`openclaw browser`](/ja/cli/browser) と [Browser ツール](/ja/tools/browser) を参照してください。

共通オプション:

* `--url`, `--token`, `--timeout`, `--json`
* `--browser-profile <name>`

管理系コマンド:

* `browser status`
* `browser start`
* `browser stop`
* `browser reset-profile`
* `browser tabs`
* `browser open <url>`
* `browser focus <targetId>`
* `browser close [targetId]`
* `browser profiles`
* `browser create-profile --name <name> [--color <hex>] [--cdp-url <url>]`
* `browser delete-profile --name <name>`

確認・キャプチャ:

* `browser screenshot [targetId] [--full-page] [--ref <ref>] [--element <selector>] [--type png|jpeg]`
* `browser snapshot [--format aria|ai] [--target-id <id>] [--limit <n>] [--interactive] [--compact] [--depth <n>] [--selector <sel>] [--out <path>]`

操作系コマンド:

* `browser navigate <url> [--target-id <id>]`
* `browser resize <width> <height> [--target-id <id>]`
* `browser click <ref> [--double] [--button <left|right|middle>] [--modifiers <csv>] [--target-id <id>]`
* `browser type <ref> <text> [--submit] [--slowly] [--target-id <id>]`
* `browser press <key> [--target-id <id>]`
* `browser hover <ref> [--target-id <id>]`
* `browser drag <startRef> <endRef> [--target-id <id>]`
* `browser select <ref> <values...> [--target-id <id>]`
* `browser upload <paths...> [--ref <ref>] [--input-ref <ref>] [--element <selector>] [--target-id <id>] [--timeout-ms <ms>]`
* `browser fill [--fields <json>] [--fields-file <path>] [--target-id <id>]`
* `browser dialog --accept|--dismiss [--prompt <text>] [--target-id <id>] [--timeout-ms <ms>]`
* `browser wait [--time <ms>] [--text <value>] [--text-gone <value>] [--target-id <id>]`
* `browser evaluate --fn <code> [--ref <ref>] [--target-id <id>]`
* `browser console [--level <error|warn|info>] [--target-id <id>]`
* `browser pdf [--target-id <id>]`

<div id="docs-search">
  ## ドキュメント検索
</div>

<div id="docs-query">
  ### `docs [query...]`
</div>

ライブドキュメントのインデックスを検索します。

## TUI

<div id="tui">
  ### `tui`
</div>

Gateway に接続するターミナル UI を開きます。

オプション:

* `--url <url>`
* `--token <token>`
* `--password <password>`
* `--session <key>`
* `--deliver`
* `--thinking <level>`
* `--message <text>`
* `--timeout-ms <ms>`（デフォルト: `agents.defaults.timeoutSeconds`）
* `--history-limit <n>`