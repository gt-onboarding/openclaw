---
title: ウィザード
summary: "CLI オンボーディングウィザード: Gateway、ワークスペース、チャネル、スキルのガイド付きセットアップ"
read_when:
  - オンボーディングウィザードを実行または設定しているとき
  - 新しいマシンをセットアップしているとき
---

<div id="onboarding-wizard-cli">
  # オンボーディングウィザード (CLI)
</div>

オンボーディングウィザードは、macOS、
Linux、または Windows（WSL2 経由を強く推奨）上で OpenClaw をセットアップするための**推奨**手順です。
ローカルの Gateway またはリモート Gateway への接続に加えて、チャネル、スキル、
およびワークスペースのデフォルト設定を、1 つのガイド付きフローでまとめて構成します。

主なエントリポイント:

```bash
openclaw onboard
```

最速でチャットを始めるには、Control UI を開きます（チャンネルの事前設定は不要です）。`openclaw dashboard` を実行し、ブラウザ上でチャットします。ドキュメント: [Dashboard](/ja/web/dashboard)。

後から再設定する場合:

```bash
openclaw configure
```

推奨：エージェントが `web_search` を使えるように Brave Search の APIキー を設定します
（`web_fetch` はキーなしでも動作します）。最も簡単な手順は `openclaw configure --section web`
を実行することで、`tools.web.search.apiKey` に保存されます。ドキュメント: [Web tools](/ja/tools/web)。

<div id="quickstart-vs-advanced">
  ## QuickStart と Advanced の違い
</div>

ウィザードは、最初に **QuickStart**（デフォルト）と **Advanced**（完全な制御）のどちらを使うかを選択します。

**QuickStart** では、以下のデフォルト設定をそのまま使用します:

* ローカル Gateway（ループバック）
* ワークスペースのデフォルト（または既存のワークスペース）
* Gateway ポート **18789**
* Gateway 認証 **Token**（ループバックでも自動生成）
* Tailscale への公開 **Off**
* Telegram と WhatsApp の DM はデフォルトで **許可リスト**（電話番号の入力を求められます）

**Advanced** では、すべてのステップ（モード、ワークスペース、Gateway、チャネル、デーモン、スキル）を個別に設定できます。

<div id="what-the-wizard-does">
  ## ウィザードの処理内容
</div>

**ローカルモード（デフォルト）** では、次の項目を順に案内します:

* モデル／認証（OpenAI Code (Codex) サブスクリプションの OAuth、Anthropic API キー（推奨）または setup-token（貼り付け）、および MiniMax/GLM/Moonshot/AI Gateway のオプション）
* ワークスペースの場所 + ブートストラップ用ファイル
* Gateway 設定（ポート／バインド／認証／Tailscale）
* プロバイダー（Telegram、WhatsApp、Discord、Google Chat、Mattermost（プラグイン）、Signal）
* デーモンのインストール（LaunchAgent / systemd ユーザーユニット）
* ヘルスチェック
* スキル（推奨）

**リモートモード** は、別の場所にある Gateway に接続するためのローカルクライアントのみを設定します。
リモートホスト上では、何もインストールも変更も **行いません**。

より分離されたエージェント（個別のワークスペース + セッション + 認証）を追加するには、次を使用します:

```bash
openclaw agents add <name>
```

ヒント: `--json` は非対話モードを**意味するわけではありません**。スクリプトでは `--non-interactive`（および `--workspace`）を使用してください。

<div id="flow-details-local">
  ## フロー詳細（ローカル）
</div>

1. **既存コンフィグの検出**
   * `~/.openclaw/openclaw.json` が存在する場合、**Keep / Modify / Reset** から選択します。
   * ウィザードを再実行しても、明示的に **Reset** を選ばない限り、何も消去されません。
     （`--reset` を指定した場合もリセットされます）。
   * コンフィグが不正、またはレガシーなキーを含んでいる場合、ウィザードはいったん停止し、
     続行前に `openclaw doctor` を実行するよう求めます。
   * Reset では `rm` ではなく `trash` を使用し、以下のスコープを提示します:
     * コンフィグのみ
     * コンフィグ + 資格情報 + セッション
     * フルリセット（ワークスペースも削除）

2. **モデル/認証**
   * **Anthropic API key（推奨）**: `ANTHROPIC_API_KEY` があればそれを使用し、なければキー入力を促して、デーモン用に保存します。
   * **Anthropic OAuth（Claude Code CLI）**: macOS では Keychain の &quot;Claude Code-credentials&quot; を確認します（launchd 起動時にブロックされないよう &quot;Always Allow&quot; を選択してください）。Linux/Windows では `~/.claude/.credentials.json` があればそれを再利用します。
   * **Anthropic token（setup-token を貼り付け）**: 任意のマシンで `claude setup-token` を実行し、そのトークンを貼り付けます（名前を付けられます。空欄 = デフォルト）。
   * **OpenAI Code (Codex) subscription（Codex CLI）**: `~/.codex/auth.json` が存在する場合、ウィザードはそれを再利用できます。
   * **OpenAI Code (Codex) subscription（OAuth）**: ブラウザフローで実行し、`code#state` を貼り付けます。
     * モデルが未設定または `openai/*` のとき、`agents.defaults.model` を `openai-codex/gpt-5.2` に設定します。
   * **OpenAI API key**: `OPENAI_API_KEY` があればそれを使用し、なければキー入力を促して、`~/.openclaw/.env` に保存し、launchd が読めるようにします。
   * **OpenCode Zen（マルチモデルプロキシ）**: `OPENCODE_API_KEY`（または `OPENCODE_ZEN_API_KEY`）の入力を促します（https://opencode.ai/auth で取得）。
   * **API key**: 入力されたキーを保存します。
   * **Vercel AI Gateway（マルチモデルプロキシ）**: `AI_GATEWAY_API_KEY` の入力を促します。
   * 詳細: [Vercel AI Gateway](/ja/providers/vercel-ai-gateway)
   * **MiniMax M2.1**: コンフィグは自動生成されます。
   * 詳細: [MiniMax](/ja/providers/minimax)
   * **Synthetic（Anthropic 互換）**: `SYNTHETIC_API_KEY` の入力を促します。
   * 詳細: [Synthetic](/ja/providers/synthetic)
   * **Moonshot（Kimi K2）**: コンフィグは自動生成されます。
   * **Kimi Code**: コンフィグは自動生成されます。
   * 詳細: [Moonshot AI (Kimi + Kimi Code)](/ja/providers/moonshot)
   * **Skip**: まだ認証情報は設定しません。
   * 検出されたオプションからデフォルトモデルを選択（またはプロバイダー/モデルを手動入力）します。
   * ウィザードはモデルチェックを実行し、設定されたモデルが不明、または認証情報が不足している場合に警告します。

* OAuth 資格情報は `~/.openclaw/credentials/oauth.json` に保存され、認証プロファイルは `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` に保存されます（API キー + OAuth）。
  * 詳細: [/concepts/oauth](/ja/concepts/oauth)

3. **Workspace**
   * デフォルトは `~/.openclaw/workspace`（設定可能）。
   * エージェントのブートストラップ処理に必要なワークスペースファイルを初期配置する。
   * ワークスペースの完全なレイアウトとバックアップガイド: [Agent workspace](/ja/concepts/agent-workspace)

4. **Gateway**
   * ポート、bind、認証モード、Tailscale での公開。
   * 認証の推奨設定: ローカルの WS クライアントにも認証を必須にするため、ループバック接続であっても **Token** を維持すること。
   * ローカルのすべてのプロセスを完全に信頼できる場合のみ認証を無効化すること。
   * ループバック以外の bind では、引き続き認証が必須。

5. **Channels**
   * [WhatsApp](/ja/channels/whatsapp): QR ログインは任意。
   * [Telegram](/ja/channels/telegram): bot トークン。
   * [Discord](/ja/channels/discord): bot トークン。
   * [Google Chat](/ja/channels/googlechat): サービスアカウント JSON + webhook audience。
   * [Mattermost](/ja/channels/mattermost)（プラグイン）: bot トークン + ベース URL。
   * [Signal](/ja/channels/signal): `signal-cli` のインストール（任意）+ アカウント設定。
   * [iMessage](/ja/channels/imessage): ローカルの `imsg` CLI パス + DB へのアクセス。
   * DM セキュリティ: デフォルトはペアリング。最初の DM でコードが送信され、`openclaw pairing approve <channel> <code>` で承認するか、許可リストを使用する。

6. **Daemon install**
   * macOS: LaunchAgent
     * ログイン中のユーザーセッションが必要。ヘッドレス運用にはカスタム LaunchDaemon（同梱されていない）を使用すること。
   * Linux（および WSL2 経由の Windows）: systemd ユーザーユニット
     * ウィザードは `loginctl enable-linger <user>` を実行して lingering を有効化し、ログアウト後も Gateway が動作し続けるよう試みる。
     * 場合によっては sudo が必要（`/var/lib/systemd/linger` に書き込み）。まずは sudo なしで試行する。
   * **ランタイム選択:** Node（推奨。WhatsApp/Telegram には必須）。Bun は**非推奨**。

7. **Health check**
   * 必要に応じて Gateway を起動し、`openclaw health` を実行する。
   * ヒント: `openclaw status --deep` はステータス出力に Gateway のヘルスプローブを追加する（到達可能な Gateway が必要）。

8. **Skills (recommended)**
   * 利用可能なスキルを読み取り、要件をチェックする。
   * パッケージマネージャーとして **npm / pnpm** を選択できる（bun は非推奨）。
   * オプションの依存関係をインストールする（一部は macOS で Homebrew を使用）。

9. **Finish**
   * 追加機能用の iOS/Android/macOS アプリを含む概要と次のステップを表示。

* GUI が検出されない場合、ウィザードはブラウザを開く代わりに Control UI 用の SSH ポートフォワード手順を表示する。
  * Control UI のアセットが存在しない場合、ウィザードはそれらのビルドを試みる。フォールバックは `pnpm ui:build`（UI の依存関係を自動インストール）となる。

<div id="remote-mode">
  ## リモートモード
</div>

リモートモードでは、ローカルクライアントが別の場所の Gateway に接続するよう構成します。

ここで設定する項目:

* リモート Gateway の URL (`ws://...`)
* リモート Gateway が認証を要求する場合のトークン（推奨）

注意事項:

* リモート側でのインストールやデーモン設定の変更は一切行いません。
* Gateway がループバック専用の場合は、SSH トンネルまたは tailnet を使用してください。
* ディスカバリ用のヒント:
  * macOS: Bonjour (`dns-sd`)
  * Linux: Avahi (`avahi-browse`)

<div id="add-another-agent">
  ## 別のエージェントを追加する
</div>

`openclaw agents add <name>` を使用して、専用のワークスペース、
セッション、および認証プロファイルを持つ別のエージェントを作成します。`--workspace` を付けずに実行するとウィザードが起動します。

設定される項目:

* `agents.list[].name`
* `agents.list[].workspace`
* `agents.list[].agentDir`

補足:

* デフォルトのワークスペースは `~/.openclaw/workspace-<agentId>` という形式になります。
* 受信メッセージをルーティングするには `bindings` を追加します（ウィザードでも設定可能です）。
* 非対話用フラグ: `--model`, `--agent-dir`, `--bind`, `--non-interactive`。

<div id="noninteractive-mode">
  ## 非対話型モード
</div>

`--non-interactive` を使って、オンボーディングを自動化したりスクリプトから実行したりします。

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

機械可読な要約を得るには `--json` を追加します。

Gemini の例:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice gemini-api-key \
  --gemini-api-key "$GEMINI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Z.AI の例:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice zai-api-key \
  --zai-api-key "$ZAI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Vercel AI Gateway の例：

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Moonshot の例：

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice moonshot-api-key \
  --moonshot-api-key "$MOONSHOT_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

擬似的な例:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice synthetic-api-key \
  --synthetic-api-key "$SYNTHETIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

OpenCode Zen の例：

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice opencode-zen \
  --opencode-zen-api-key "$OPENCODE_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

エージェントを追加する（非対話型）例：

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

<div id="gateway-wizard-rpc">
  ## Gateway ウィザード RPC
</div>

Gateway は、RPC（`wizard.start`、`wizard.next`、`wizard.cancel`、`wizard.status`）を通じてウィザードのフローを提供します。
クライアント（macOS アプリ、Control UI）は、オンボーディングロジックを再実装することなくステップをレンダリングできます。

<div id="signal-setup-signal-cli">
  ## Signal セットアップ (signal-cli)
</div>

ウィザードでは GitHub のリリースから `signal-cli` をインストールできます:

* 適切なリリースアセットをダウンロードします。
* `~/.openclaw/tools/signal-cli/<version>/` 配下に保存します。
* 設定ファイルに `channels.signal.cliPath` を書き込みます。

注意事項:

* JVM ビルドには **Java 21** が必要です。
* 利用可能な場合はネイティブビルドが使用されます。
* Windows では WSL2 を使用し、signal-cli のインストールは WSL 内で Linux 向けフローに従います。

<div id="what-the-wizard-writes">
  ## ウィザードが書き込む内容
</div>

`~/.openclaw/openclaw.json` に書き込まれる代表的なフィールドは次のとおりです:

* `agents.defaults.workspace`
* `agents.defaults.model` / `models.providers`（Minimax を選んだ場合）
* `gateway.*`（mode, bind, auth, tailscale）
* `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
* プロンプトで opt-in（有効化）を選択した場合のチャネル許可リスト（Slack/Discord/Matrix/Microsoft Teams）。可能な場合、名前は ID に解決されます。
* `skills.install.nodeManager`
* `wizard.lastRunAt`
* `wizard.lastRunVersion`
* `wizard.lastRunCommit`
* `wizard.lastRunCommand`
* `wizard.lastRunMode`

`openclaw agents add` は `agents.list[]` と任意の `bindings` フィールドに書き込みます。

WhatsApp の認証情報は `~/.openclaw/credentials/whatsapp/<accountId>/` 配下に配置されます。
セッションは `~/.openclaw/agents/<agentId>/sessions/` 配下に保存されます。

一部のチャネルはプラグインとして提供されています。オンボーディング中にいずれかを選択すると、
ウィザードは設定可能になる前に、それを（npm 経由またはローカルパスから）インストールするよう促します。

<div id="related-docs">
  ## 関連ドキュメント
</div>

* macOS アプリの初期設定: [オンボーディング](/ja/start/onboarding)
* 設定リファレンス: [Gateway 設定](/ja/gateway/configuration)
* プロバイダー: [WhatsApp](/ja/channels/whatsapp), [Telegram](/ja/channels/telegram), [Discord](/ja/channels/discord), [Google Chat](/ja/channels/googlechat), [Signal](/ja/channels/signal), [iMessage](/ja/channels/imessage)
* スキル: [スキル](/ja/tools/skills), [スキル設定](/ja/tools/skills-config)