---
title: Doctor
summary: "Doctor コマンド: ヘルスチェック、設定マイグレーション、修復手順"
read_when:
  - doctor のマイグレーションを追加または変更するとき
  - 破壊的な設定変更を導入するとき
---

<div id="doctor">
  # Doctor
</div>

`openclaw doctor` は OpenClaw 用の修復＋マイグレーションツールです。古くなった
構成や状態を修正し、ヘルスチェックを行い、すぐに実行できる具体的な修復手順を提示します。

<div id="quick-start">
  ## クイックスタート
</div>

```bash
openclaw doctor
```

<div id="headless-automation">
  ### ヘッドレス／自動化
</div>

```bash
openclaw doctor --yes
```

デフォルトをすべて確認なしで自動的に受け入れます（必要に応じて、再起動／サービス／サンドボックスの修復手順も含みます）。

```bash
openclaw doctor --repair
```

推奨される修復処理をユーザーへの確認なしに適用します（安全と判断できる場合は修復と再起動を自動実行）。

```bash
openclaw doctor --repair --force
```

より踏み込んだ修復も行います（カスタムの supervisor 設定を上書きします）。

```bash
openclaw doctor --non-interactive
```

プロンプトなしで実行し、安全なマイグレーション（設定の正規化＋ディスク上の状態データの移動）のみを適用します。ユーザーの確認が必要な再起動／サービス／サンドボックスに関するアクションはスキップされます。
旧形式の状態データに対するマイグレーションは、検出された場合は自動的に実行されます。

```bash
openclaw doctor --deep
```

システムサービス（launchd/systemd/schtasks）をスキャンし、余分な Gateway のインストールを検出します。

書き込む前に変更内容を確認したい場合は、先に設定ファイルを開いてください。

```bash
cat ~/.openclaw/openclaw.json
```

<div id="what-it-does-summary">
  ## 何をするか（概要）
</div>

* git インストール向けの任意の事前アップデート処理（対話モード時のみ）。
* UI プロトコルの鮮度チェック（プロトコルスキーマが新しい場合に Control UI を再ビルド）。
* ヘルスチェック + 再起動プロンプト。
* スキル状態のサマリー（利用可能 / 不足 / ブロック）。
* レガシー値に対する設定の正規化。
* OpenCode Zen プロバイダーのオーバーライドに関する警告（`models.providers.opencode`）。
* レガシーなオンディスク状態データの移行（セッション / エージェントディレクトリ / WhatsApp 認証）。
* 状態の整合性とパーミッションチェック（セッション、トランスクリプト、state ディレクトリ）。
* ローカル実行時の設定ファイルのパーミッションチェック（chmod 600）。
* モデル認証のヘルスチェック：OAuth の有効期限を確認し、期限切れ間近のトークンを更新し、auth-profile のクールダウン / 無効化状態をレポート。
* 追加のワークスペースディレクトリ検出（`~/openclaw`）。
* サンドボックスが有効な場合のサンドボックスイメージ修復。
* レガシーサービスの移行および余分な Gateway インスタンスの検出。
* Gateway ランタイムチェック（サービスはインストールされているが動作していない状態、キャッシュされた launchd ラベル）。
* チャネルステータスの警告（稼働中の Gateway からのプローブ結果）。
* スーパーバイザー設定の監査（launchd/systemd/schtasks）と任意の修復。
* Gateway ランタイムのベストプラクティスチェック（Node と Bun、バージョンマネージャーのパス）。
* Gateway ポート衝突診断（デフォルト `18789`）。
* open DM ポリシーに対するセキュリティ警告。
* `gateway.auth.token` が設定されていない場合の Gateway 認証警告（ローカルモード；トークン生成を提案）。
* Linux 上での systemd linger チェック。
* ソースからのインストールに関するチェック（pnpm ワークスペースの不一致、UI アセット不足、tsx バイナリ不足）。
* 更新された設定とウィザードメタデータの書き込み。

<div id="detailed-behavior-and-rationale">
  ## 動作の詳細とその背景
</div>

<div id="0-optional-update-git-installs">
  ### 0) 任意の更新 (git インストール)
</div>

git チェックアウトされたコピーからの実行で、かつ `doctor` が対話的に実行されている場合、`doctor` を実行する前に
更新 (fetch/rebase/build) を行うかどうかを確認するプロンプトを出します。

<div id="1-config-normalization">
  ### 1) 設定の正規化
</div>

設定内にレガシーな値の形式（たとえば、チャンネル固有のオーバーライドがない `messages.ackReaction`
など）が含まれている場合、doctor はそれらを現在のスキーマに正規化します。

<div id="2-legacy-config-key-migrations">
  ### 2) レガシー設定キーのマイグレーション
</div>

設定に非推奨キーが含まれている場合、他のコマンドは実行を拒否し、
代わりに `openclaw doctor` を実行するよう求めます。

Doctor は以下を実行します:

* 検出されたレガシーキーを報告します。
* 適用したマイグレーション内容を表示します。
* 更新されたスキーマで `~/.openclaw/openclaw.json` を書き換えます。

Gateway も、レガシーな設定フォーマットを検出した場合には起動時に
自動で doctor のマイグレーションを実行するため、古い設定は手動の操作なしで修復されます。

現在のマイグレーション:

* `routing.allowFrom` → `channels.whatsapp.allowFrom`
* `routing.groupChat.requireMention` → `channels.whatsapp/telegram/imessage.groups."*".requireMention`
* `routing.groupChat.historyLimit` → `messages.groupChat.historyLimit`
* `routing.groupChat.mentionPatterns` → `messages.groupChat.mentionPatterns`
* `routing.queue` → `messages.queue`
* `routing.bindings` → トップレベルの `bindings`
* `routing.agents`/`routing.defaultAgentId` → `agents.list` + `agents.list[].default`
* `routing.agentToAgent` → `tools.agentToAgent`
* `routing.transcribeAudio` → `tools.media.audio.models`
* `bindings[].match.accountID` → `bindings[].match.accountId`
* `identity` → `agents.list[].identity`
* `agent.*` → `agents.defaults` + `tools.*` (tools/elevated/exec/sandbox/subagents)
* `agent.model`/`allowedModels`/`modelAliases`/`modelFallbacks`/`imageModelFallbacks`
  → `agents.defaults.models` + `agents.defaults.model.primary/fallbacks` + `agents.defaults.imageModel.primary/fallbacks`

<div id="2b-opencode-zen-provider-overrides">
  ### 2b) OpenCode Zen プロバイダーのオーバーライド
</div>

`models.providers.opencode`（または `opencode-zen`）を手動で追加している場合、
`@mariozechner/pi-ai` に含まれる組み込みの OpenCode Zen カタログを
上書きしてしまいます。これにより、すべてのモデルが単一の API に
まとめてルーティングされたり、コストが 0 と見なされてしまう可能性があります。
Doctor コマンドはこの状況を検知して警告を出し、オーバーライドを削除して
モデルごとの API ルーティングとコスト計算を元に戻せるようにします。

<div id="3-legacy-state-migrations-disk-layout">
  ### 3) レガシー状態のマイグレーション（ディスクレイアウト）
</div>

Doctor は、古いディスク上のレイアウトを現在の構造にマイグレーションできます:

* セッションストア + トランスクリプト:
  * `~/.openclaw/sessions/` から `~/.openclaw/agents/<agentId>/sessions/` へ
* Agent ディレクトリ:
  * `~/.openclaw/agent/` から `~/.openclaw/agents/<agentId>/agent/` へ
* WhatsApp 認証状態（Baileys）:
  * レガシーな `~/.openclaw/credentials/*.json`（`oauth.json` を除く）から
  * `~/.openclaw/credentials/whatsapp/<accountId>/...` へ（デフォルトの account id: `default`）

これらのマイグレーションはベストエフォートかつ冪等的であり、Doctor は
レガシーフォルダをバックアップとして残した場合には警告を出力します。
Gateway/CLI も起動時にレガシーな sessions + agent ディレクトリを自動マイグレーションし、
履歴/認証情報/モデルが手動で doctor を実行しなくても
エージェントごとのパスに配置されるようにします。WhatsApp の認証状態は、
意図的に `openclaw doctor` によってのみマイグレーションされます。

<div id="4-state-integrity-checks-session-persistence-routing-and-safety">
  ### 4) 状態の完全性チェック（セッション永続化、ルーティング、安全性）
</div>

state ディレクトリは運用上の「脳幹」です。これが消えると、
セッション、認証情報、ログ、設定を失います（どこかにバックアップがない限り）。

Doctor チェック:

* **State dir missing**: 破滅的な状態喪失について警告し、ディレクトリの再作成を促し、
  欠損したデータは復旧できないことを念押しします。
* **State dir permissions**: 書き込み可能か検証し、パーミッションの修復を提案します
  （owner/group の不一致が検出された場合は `chown` のヒントを出力します）。
* **Session dirs missing**: 履歴を永続化し `ENOENT` によるクラッシュを避けるために、
  `sessions/` とセッションストアディレクトリが必須です。
* **Transcript mismatch**: 直近のセッションエントリに対応する transcript ファイルが
  欠落している場合に警告します。
* **Main session “1-line JSONL”**: メイン transcript が 1 行しかない場合
  （履歴が蓄積されていない状態）、これをフラグします。
* **Multiple state dirs**: 複数の `~/.openclaw` フォルダがホームディレクトリ間に存在する場合、
  または `OPENCLAW_STATE_DIR` が別の場所を指している場合に警告します
  （インストール間で履歴が分断される可能性があります）。
* **Remote mode reminder**: `gateway.mode=remote` の場合、state はリモート側に存在するため、
  リモートホスト上で doctor を実行するようリマインドします。
* **Config file permissions**: `~/.openclaw/openclaw.json` が group/world readable
  になっている場合に警告し、パーミッションを `600` まで絞ることを提案します。

<div id="5-model-auth-health-oauth-expiry">
  ### 5) モデル認証ヘルス（OAuth 有効期限）
</div>

Doctor は認証ストア内の OAuth プロファイルを検査し、トークンの有効期限が
近い／すでに失効している場合に警告し、安全な場合はトークンをリフレッシュします。Anthropic Claude Code
プロファイルが古くなっている場合は、`claude setup-token` を実行する（または setup-token を貼り付ける）
ように提案します。
リフレッシュ用のプロンプトは対話的モード（TTY）で実行している場合にのみ表示されます。`--non-interactive`
を指定するとリフレッシュは試行されません。

Doctor は、次の理由で一時的に使用不能になっている認証プロファイルも報告します：

* 短時間のクールダウン（レート制限／タイムアウト／認証失敗）
* 長期の無効化（課金／クレジット関連の失敗）

<div id="6-hooks-model-validation">
  ### 6) Hooks のモデル検証
</div>

`hooks.gmail.model` が設定されている場合、doctor はモデル参照がカタログおよび許可リストに存在するかを検証し、解決できない場合や許可されていない場合に警告します。

<div id="7-sandbox-image-repair">
  ### 7) サンドボックスイメージの修復
</div>

サンドボックス機能が有効な場合、doctor コマンドは Docker イメージをチェックし、
現在のイメージが存在しないときは新規ビルドを提案するか、従来のイメージ名への切り替えを提案します。

<div id="8-gateway-service-migrations-and-cleanup-hints">
  ### 8) Gateway サービスの移行とクリーンアップのヒント
</div>

Doctor はレガシーな Gateway サービス (launchd/systemd/schtasks) を検出し、
それらを削除して、現在の Gateway ポートを使用する OpenClaw サービスを
インストールするよう提案します。Gateway に類似した余分なサービスをスキャンし、
クリーンアップのヒントを出力することもできます。
プロファイル名付きの OpenClaw Gateway サービスは第一級のものとして扱われ、
「余分な」ものとしてはフラグ付けされません。

<div id="9-security-warnings">
  ### 9) セキュリティ警告
</div>

Doctor は、プロバイダーが許可リストなしで DM（ダイレクトメッセージ）を受け付けるよう `open`（任意のユーザーからのメッセージを無制限に受け入れる設定）にされている場合や、
ポリシーが危険な形で設定されている場合に警告を出します。

<div id="10-systemd-linger-linux">
  ### 10) systemd linger (Linux)
</div>

systemd のユーザーサービスとして実行している場合、doctor は linger が有効になっていることを確認し、
ログアウト後も Gateway が稼働し続けるようにします。

<div id="11-skills-status">
  ### 11) スキルのステータス
</div>

Doctor は、現在のワークスペースについて、利用可能 / 不足 / ブロックされているスキルの簡単な概要を出力します。

<div id="12-gateway-auth-checks-local-token">
  ### 12) Gateway 認証チェック（ローカルトークン）
</div>

Doctor は、ローカルの Gateway で `gateway.auth` が設定されていない場合に警告を出し、
トークンの生成を提案します。自動化処理でトークンの作成を強制したい場合は、
`openclaw doctor --generate-gateway-token` を使用してください。

<div id="13-gateway-health-check-restart">
  ### 13) Gateway のヘルスチェック + 再起動
</div>

Doctor はヘルスチェックを実行し、Gateway が正常でないと判断された場合は再起動を提案します。

<div id="14-channel-status-warnings">
  ### 14) チャネル状態の警告
</div>

Gateway が正常な状態であれば、doctor はチャネル状態プローブを実行し、
推奨される修正方法とともに警告を報告します。

<div id="15-supervisor-config-audit-repair">
  ### 15) Supervisor 設定の監査と修復
</div>

Doctor はインストール済みの supervisor 設定（launchd/systemd/schtasks）について、
不足している、または古くなった既定値（例: systemd の network-online 依存関係や
再起動ディレイ）をチェックします。不整合を検出すると、更新を推奨し、
サービスファイル／タスクを現在の既定値（デフォルト）に書き換えることができます。

補足:

* `openclaw doctor` は supervisor 設定を書き換える前に確認プロンプトを出します。
* `openclaw doctor --yes` は修復に関するデフォルトのプロンプトを自動的に承諾します。
* `openclaw doctor --repair` はプロンプトなしで推奨される修正を適用します。
* `openclaw doctor --repair --force` はカスタムの supervisor 設定を上書きします。
* `openclaw gateway install --force` を使えば、常に設定の完全な書き換えを強制できます。

<div id="16-gateway-runtime-port-diagnostics">
  ### 16) Gateway ランタイム + ポート診断
</div>

Doctor はサービスのランタイム（PID、直近の終了ステータス）を検査し、
サービスがインストールされているにもかかわらず実際には稼働していない場合に警告を出します。
また、Gateway ポート（デフォルトは `18789`）でのポート競合をチェックし、
その原因として考えられる要因（既に Gateway が稼働している、SSH トンネル経由でポートが使用されているなど）を報告します。

<div id="17-gateway-runtime-best-practices">
  ### 17) Gateway ランタイムのベストプラクティス
</div>

Doctor は、Gateway サービスが Bun 上やバージョン管理された Node のパス
（`nvm`、`fnm`、`volta`、`asdf` など）で動作している場合に警告します。WhatsApp および Telegram チャンネルには Node が必要であり、サービスはあなたのシェルの初期化スクリプトを読み込まないため、バージョンマネージャーのパスはアップグレード後に動作しなくなる可能性があります。Doctor は、利用可能な場合にはシステムにインストールされた Node（Homebrew/apt/choco）への移行を提案します。

<div id="18-config-write-wizard-metadata">
  ### 18) Config の書き込み + wizard メタデータ
</div>

Doctor は、行われたあらゆる Config の変更を保存し、doctor の実行を記録するために wizard メタデータを付与します。

<div id="19-workspace-tips-backup-memory-system">
  ### 19) ワークスペースのヒント（バックアップ + メモリシステム）
</div>

Doctor コマンドは、ワークスペースにメモリシステムが存在しない場合はその導入を提案し、
ワークスペースがまだ git 管理下にない場合はバックアップに関するヒントを出力します。

ワークスペース構造と git バックアップ（プライベートな GitHub または GitLab を推奨）に関する
詳細なガイドについては、[/concepts/agent-workspace](/ja/concepts/agent-workspace) を参照してください。