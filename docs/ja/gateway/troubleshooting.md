---
title: トラブルシューティング
summary: "一般的な OpenClaw の障害に対するクイックトラブルシューティングガイド"
read_when:
  - 実行時の問題や障害を調査するとき
---

<div id="troubleshooting">
  # トラブルシューティング 🔧
</div>

OpenClaw の調子がおかしいときは、次の手順で対処してください。

すぐに簡易的な切り分けを行いたい場合は、まず FAQ の [最初の60秒](/ja/help/faq#first-60-seconds-if-somethings-broken) から始めてください。このページでは、実行時の障害と診断について、さらに踏み込んで解説します。

プロバイダーごとのトラブルシューティング: [/channels/troubleshooting](/ja/channels/troubleshooting)

<div id="status-diagnostics">
  ## ステータスと診断
</div>

クイックトリアージ用コマンド（上から順に）:

| Command | 内容 | 使用タイミング |
|---|---|---|
| `openclaw status` | ローカルサマリー: OS とアップデート、Gateway の到達性/モード、サービス、エージェント群/セッション、プロバイダー設定状態 | 最初の確認、ざっくり状況を把握したいとき |
| `openclaw status --all` | ローカル環境の詳細診断（読み取り専用・コピペしやすく・比較的安全な内容）とログ末尾を含む | デバッグレポートを共有する必要があるとき |
| `openclaw status --deep` | Gateway のヘルスチェックを実行（プロバイダー疎通検査を含む；Gateway へ到達可能である必要あり） | 「設定できている」ことが「実際に動いている」ことを意味しないとき |
| `openclaw gateway probe` | Gateway の検出 + 到達性チェック（ローカルおよびリモートの両方） | 間違った Gateway をプローブしている疑いがあるとき |
| `openclaw channels status --probe` | 起動中の Gateway にチャネル状態を問い合わせ（必要に応じてプローブも実行） | Gateway には到達できるがチャネルの挙動がおかしいとき |
| `openclaw gateway status` | スーパーバイザー状態（launchd/systemd/schtasks）、実行中の PID/終了コード、直近の Gateway エラー | 「サービスは動いていそう」に見えるのに何も動いていないとき |
| `openclaw logs --follow` | ライブログ（ランタイム問題の最も確かなシグナル） | 実際の失敗理由を知りたいとき |

**出力の共有:** `openclaw status --all` を優先して使ってください（トークンをマスクします）。`openclaw status` の結果を貼る場合は、事前に `OPENCLAW_SHOW_SECRETS=0`（トークンプレビュー抑制）を設定することを検討してください。

関連項目: [Health checks](/ja/gateway/health) および [Logging](/ja/logging)。

<div id="common-issues">
  ## よくある問題
</div>

<div id="no-api-key-found-for-provider-anthropic">
  ### プロバイダー &quot;anthropic&quot; の API キーが見つかりません
</div>

これは、**エージェントの auth ストアが空**であるか、Anthropic の認証情報が存在しないことを意味します。
認証情報は **エージェントごと**であり、新しいエージェントはメインエージェントのキーを継承しません。

対処方法：

* オンボーディングを再実行し、そのエージェント用に **Anthropic** を選択します。
* あるいは **Gateway ホスト** 上で setup-token を貼り付けて実行します：
  ```bash
  openclaw models auth setup-token --provider anthropic
  ```
* もしくはメインエージェントのディレクトリから新しいエージェントのディレクトリへ `auth-profiles.json` をコピーします。

確認：

```bash
openclaw models status
```

<div id="oauth-token-refresh-failed-anthropic-claude-subscription">
  ### OAuth トークンのリフレッシュに失敗しました（Anthropic Claude サブスクリプション）
</div>

これは、保存されている Anthropic の OAuth トークンの有効期限が切れており、リフレッシュ処理にも失敗したことを意味します。
Claude のサブスクリプション（API キーなし）を利用している場合、最も確実な対処方法は
**Claude Code setup-token** に切り替え、それを **Gateway のホスト** に貼り付けることです。

**推奨（setup-token 利用）：**

```bash
# Gatewayホスト上で実行（setup-tokenを貼り付ける）
openclaw models auth setup-token --provider anthropic
openclaw models status
```

トークンを別の場所で生成した場合：

```bash
openclaw models auth paste-token --provider anthropic
openclaw models status
```

詳細は [Anthropic](/ja/providers/anthropic) と [OAuth](/ja/concepts/oauth) を参照してください。

<div id="control-ui-fails-on-http-device-identity-required-connect-failed">
  ### Control UI が HTTP で動作しない（&quot;device identity required&quot; / &quot;connect failed&quot;）
</div>

ダッシュボードをプレーンな HTTP で開く場合（例: `http://<lan-ip>:18789/` や
`http://<tailscale-ip>:18789/`）、ブラウザは**非セキュアコンテキスト**で動作し、
WebCrypto がブロックされるため、デバイスアイデンティティを生成できません。

**対処法:**

* [Tailscale Serve](/ja/gateway/tailscale) 経由の HTTPS を優先して使用してください。
* もしくは Gateway ホスト上でローカルに開きます: `http://127.0.0.1:18789/`。
* どうしても HTTP を使い続ける必要がある場合は、`gateway.controlUi.allowInsecureAuth: true` を有効にし、
  Gateway トークンを使用します（トークンのみで認証し、デバイスアイデンティティ／ペアリングは使用しません）。[Control UI](/ja/web/control-ui#insecure-http) を参照してください。

<div id="ci-secrets-scan-failed">
  ### CI Secrets スキャンが失敗した
</div>

これは、`detect-secrets` がベースラインにまだ含まれていない新たな候補を検出したことを意味します。
[Secret scanning](/ja/gateway/security#secret-scanning-detect-secrets) の手順に従って対応してください。

<div id="service-installed-but-nothing-is-running">
  ### サービスはインストールされているが何も起動していない
</div>

Gateway サービスはインストールされているものの、プロセスがすぐに終了してしまう場合、実際には何も動作していないのに、サービスが「ロード済み」と表示されることがあります。

**確認:**

```bash
openclaw gateway status
openclaw doctor
```

Doctor/service はランタイムの状態（PID／直近の終了状況）と、ログに関するヒントを表示します。

**ログ:**

* 推奨: `openclaw logs --follow`
* ファイルログ（常に記録）: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`（または設定した `logging.file`）
* macOS LaunchAgent（インストール済みの場合）: `$OPENCLAW_STATE_DIR/logs/gateway.log` および `gateway.err.log`
* Linux systemd（インストール済みの場合）: `journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`
* Windows: `schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST`

**ログの詳細度を上げる:**

* ファイルログの詳細度を上げる（永続化された JSONL）:
  ```json
  { "logging": { "level": "debug" } }
  ```
* コンソールの詳細度を上げる（TTY 出力のみ）:
  ```json
  { "logging": { "consoleLevel": "debug", "consoleStyle": "pretty" } }
  ```
* ワンポイント: `--verbose` は **コンソール** 出力のみに影響します。ファイルログは引き続き `logging.level` によって制御されます。

フォーマット、設定、アクセス方法の全体像については [/logging](/ja/logging) を参照してください。

<div id="gateway-start-blocked-set-gatewaymodelocal">
  ### &quot;Gateway start blocked: set gateway.mode=local&quot;
</div>

これは、設定ファイル自体は存在するものの、`gateway.mode` が未設定（または `local` ではない）ため、
Gateway を起動できないことを意味します。

**対応（推奨）：**

* ウィザードを実行して、Gateway の実行モードを **Local** に設定します:
  ```bash
  openclaw configure
  ```
* または直接設定します:
  ```bash
  openclaw config set gateway.mode local
  ```

**リモート Gateway を起動するつもりだった場合：**

* リモート URL を設定し、`gateway.mode=remote` のままにします:
  ```bash
  openclaw config set gateway.mode remote
  openclaw config set gateway.remote.url "wss://gateway.example.com"
  ```

**アドホック/開発用途のみ：** `--allow-unconfigured` を指定して、`gateway.mode=local` を設定せずに
Gateway を起動します。

**まだ設定ファイルがない場合：** `openclaw setup` を実行して初期設定ファイルを作成し、その後
Gateway を再度起動します。

<div id="service-environment-path-runtime">
  ### サービス環境（PATH と実行環境）
</div>

Gateway サービスは、シェル／マネージャ由来の不要な設定を避けるために **最小限の PATH** で動作します：

* macOS: `/opt/homebrew/bin`, `/usr/local/bin`, `/usr/bin`, `/bin`
* Linux: `/usr/local/bin`, `/usr/bin`, `/bin`

これは意図的にバージョンマネージャ（nvm/fnm/volta/asdf）やパッケージマネージャ（pnpm/npm）を含めていません。サービスはユーザーのシェル初期化ファイルを読み込まないためです。`DISPLAY` のような実行時変数は `~/.openclaw/.env` に定義してください（Gateway によって早期に読み込まれます）。
`host=gateway` 上で実行される Exec は、ログインシェルの `PATH` を実行環境にマージします。そのため、ツールが見つからない場合は、シェル初期化ファイルがそれらを export していない（または `tools.exec.pathPrepend` を設定していない）可能性があります。[/tools/exec](/ja/tools/exec) を参照してください。

WhatsApp と Telegram チャンネルは **Node** を必要とし、Bun はサポートされていません。サービスを Bun やバージョンマネージャ経由の Node パスでインストールしている場合は、`openclaw doctor` を実行してシステムの Node インストールへ移行してください。

<div id="skill-missing-api-key-in-sandbox">
  ### サンドボックス内の Skill に API キーが設定されていない
</div>

**症状:** Skill はホスト上では動作するが、サンドボックス内では API キーが見つからないエラーで失敗する。

**理由:** サンドボックス実行は Docker 内で行われ、ホストの `process.env` を**継承しない**。

**対処法:**

* `agents.defaults.sandbox.docker.env`（またはエージェント単位の `agents.list[].sandbox.docker.env`）を設定する
* あるいはカスタムサンドボックスイメージにキーを埋め込む
* その後 `openclaw sandbox recreate --agent <id>`（または `--all`）を実行する

<div id="service-running-but-port-not-listening">
  ### サービスは動作中だがポートが待ち受けていない
</div>

サービスが **running** と報告しているのに Gateway ポートで何も待ち受けていない場合、
Gateway がバインドを拒否した可能性が高いです。

**ここでの「running」の意味**

* `Runtime: running` は、supervisor（launchd/systemd/schtasks）が「プロセスは生きている」と判断していることを意味します。
* `RPC probe` は、CLI が実際に Gateway の WebSocket に接続して `status` を呼び出せたことを意味します。
* 実際に「どこに対して試行したか」は、常に `Probe target:` と `Config (service):` の行を信頼してください。

**確認事項:**

* `openclaw gateway` とサービスの両方について、`gateway.mode` は `local` である必要があります。
* `gateway.mode=remote` を設定すると、**CLI のデフォルト** はリモート URL になります。サービスはローカルで動作し続けていても、CLI が別の場所をプローブしている可能性があります。`openclaw gateway status` を使って、サービスが解決したポートとプローブ先（または `--url` の指定）を確認してください。
* `openclaw gateway status` と `openclaw doctor` は、サービスが動作しているように見えるのにポートが閉じている場合、ログから取得した **最後の Gateway エラー** を表示します。
* 非ループバックでのバインド（`lan` / `tailnet` / `custom`、またはループバックが利用できない場合の `auto`）には認証が必要です:
  `gateway.auth.token`（または `OPENCLAW_GATEWAY_TOKEN`）。
* `gateway.remote.token` はリモートからの CLI 呼び出し専用であり、ローカル認証は **有効化しません**。
* `gateway.token` は無視されます。`gateway.auth.token` を使用してください。

**`openclaw gateway status` に設定の不一致が表示される場合**

* `Config (cli): ...` と `Config (service): ...` は通常一致している必要があります。
* 一致していない場合、ほぼ確実に、「片方の設定を編集している一方で、サービスは別の設定を使って動作している」状態になっています。
* 対処: サービスに使わせたい `--profile` / `OPENCLAW_STATE_DIR` と同じものから `openclaw gateway install --force` を再実行してください。

**`openclaw gateway status` がサービス設定の問題を報告する場合**

* supervisor 設定（launchd/systemd/schtasks）が現在のデフォルトを反映していません。
* 対処: `openclaw doctor` を実行して更新するか、全書き換えしたい場合は `openclaw gateway install --force` を実行してください。

**`Last gateway error:` に「refusing to bind … without auth」とある場合**

* `gateway.bind` を非ループバックのモード（`lan` / `tailnet` / `custom`、またはループバックが利用できない場合の `auto`）に設定したにもかかわらず、認証を設定していません。
* 対処: `gateway.auth.mode` と `gateway.auth.token` を設定する（または `OPENCLAW_GATEWAY_TOKEN` を export する）うえで、サービスを再起動してください。

**`openclaw gateway status` が `bind=tailnet` と表示するのに tailnet インターフェイスが見つからない場合**

* Gateway は Tailscale の IP（100.64.0.0/10）へのバインドを試みましたが、ホスト上で検出されませんでした。
* 対処: そのマシン上で Tailscale を起動するか、`gateway.bind` を `loopback` / `lan` に変更してください。

**`Probe note:` にプローブがループバックを使用していると書かれている場合**

* これは `bind=lan` では想定どおりの動作です: Gateway は `0.0.0.0`（全インターフェイス）で待ち受けますが、ローカルからはループバックで接続できるはずです。
* リモートクライアントから接続する場合は、実際の LAN IP（`0.0.0.0` ではなく）とポートを使用し、認証が設定されていることを確認してください。

<div id="address-already-in-use-port-18789">
  ### アドレスは既に使用中です（ポート 18789）
</div>

これは、何かが Gateway のポートで既に待ち受けていることを意味します。

**確認:**

```bash
openclaw gateway status
```

リスナーと考えられる原因（Gateway が既に動作中、SSH トンネルなど）が表示されます。
必要に応じてサービスを停止するか、別のポートを選択してください。

<div id="extra-workspace-folders-detected">
  ### 追加のワークスペースフォルダが検出されました
</div>

以前のインストールからアップグレードした場合、ディスク上に `~/openclaw` が残っている可能性があります。
複数のワークスペースディレクトリがあると、有効になるワークスペースは 1 つだけのため、
認証や状態に不整合が生じて分かりにくくなることがあります。

**対処:** 有効なワークスペースは 1 つに統一し、残りはアーカイブまたは削除してください。詳しくは
[エージェントのワークスペース](/ja/concepts/agent-workspace#extra-workspace-folders) を参照してください。

<div id="main-chat-running-in-a-sandbox-workspace">
  ### メインチャットがサンドボックスのワークスペースで動作している
</div>

症状：`pwd` やファイル系ツールが、期待していたホスト側のワークスペースではなく
`~/.openclaw/sandboxes/...` を表示する。

**理由:** `agents.defaults.sandbox.mode: "non-main"` は `session.mainKey`（デフォルトは `"main"`）を基準に動作します。
グループ/チャンネルのセッションはそれぞれ固有のキーを使用するため、`non-main` 扱いとなり、
サンドボックスのワークスペースが割り当てられます。

**対処オプション:**

* そのエージェントに対してホストのワークスペースを使いたい場合: `agents.list[].sandbox.mode: "off"` を設定します。
* サンドボックス内からホストのワークスペースへアクセスしたい場合: そのエージェントの `workspaceAccess: "rw"` を設定します。

<div id="agent-was-aborted">
  ### &quot;エージェントが中断されました&quot;
</div>

エージェントは応答の途中で処理が中断されました。

**原因:**

* ユーザーが `stop`、`abort`、`esc`、`wait`、または `exit` を送信した
* タイムアウトに達した
* プロセスがクラッシュした

**対処:** 別のメッセージを送信してください。セッションは継続しています。

<div id="agent-failed-before-reply-unknown-model-anthropicclaude-haiku-3-5">
  ### &quot;Agent failed before reply: Unknown model: anthropic/claude-haiku-3-5&quot;
</div>

OpenClaw は、意図的に **古い／安全性の低いモデル**（特にプロンプトインジェクションに
脆弱なもの）を拒否します。このエラーが表示される場合、そのモデル名はもはやサポートされていません。

**対処方法:**

* 該当プロバイダーの**最新**モデルを選び、設定またはモデルエイリアスを更新します。
* 利用可能なモデルがわからない場合は、`openclaw models list` または
  `openclaw models scan` を実行し、サポートされているモデルを選択します。
* より詳細な失敗理由については、Gateway のログを確認します。

あわせて参照: [Models CLI](/ja/cli/models) および [Model providers](/ja/concepts/model-providers)。

<div id="messages-not-triggering">
  ### メッセージがトリガーされない
</div>

**確認 1:** 送信者は許可リストに入っていますか？

```bash
openclaw status
```

出力内で `AllowFrom: ...` を探してください。

**チェック 2:** グループチャットではメンションが必要になっていますか？

```bash
# メッセージは mentionPatterns または明示的なメンションと一致する必要があります。デフォルト値はチャネルのグループ/ギルドに存在します。
# マルチエージェント: `agents.list[].groupChat.mentionPatterns` がグローバルパターンを上書きします。
grep -n "agents\\|groupChat\\|mentionPatterns\\|channels\\.whatsapp\\.groups\\|channels\\.telegram\\.groups\\|channels\\.imessage\\.groups\\|channels\\.discord\\.guilds" \
  "${OPENCLAW_CONFIG_PATH:-$HOME/.openclaw/openclaw.json}"
```

**チェック3：** ログを確認する

```bash
openclaw logs --follow
# または、クイックフィルタが必要な場合:
tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)" | grep "blocked\\|skip\\|unauthorized"
```

<div id="pairing-code-not-arriving">
  ### ペアリングコードが届かない
</div>

`dmPolicy` が `pairing` の場合、不明な送信者にはコードが送信され、承認されるまでメッセージは無視されます。

**確認 1:** 保留中のリクエストがすでに存在していませんか？

```bash
openclaw pairing list <channel>
```

保留中のDMペアリングリクエストは、デフォルトでは**チャンネルごとに最大3件**に制限されています。リストが上限に達している場合は、いずれかが承認されるか有効期限切れになるまで、新しいリクエストにはコードが発行されません。

**チェック2:** リクエストは作成されたが、返信が送信されていないか？

```bash
openclaw logs --follow | grep "pairing request"
```

**Check 3:** そのチャネルで `dmPolicy` が `open`（全ユーザーから無制限にメッセージを受け付ける設定）や `allowlist` になっていないことを確認します。

<div id="image-mention-not-working">
  ### 画像＋メンションが動作しない
</div>

既知の問題: 画像をメンション「だけ」（ほかのテキストなし）で送信すると、WhatsApp がメンションのメタデータを付与しない場合があります。

**回避策:** メンションと一緒に短いテキストも追加してください:

* ❌ `@openclaw` + 画像
* ✅ `@openclaw check this` + 画像

<div id="session-not-resuming">
  ### セッションが再開されない
</div>

**確認 1:** セッションファイルは存在しますか？

```bash
ls -la ~/.openclaw/agents/<agentId>/sessions/
```

**チェック 2：** リセットウィンドウの設定が短すぎないか？

```json
{
  "session": {
    "reset": {
      "mode": "daily",
      "atHour": 4,
      "idleMinutes": 10080  // 7日間
    }
  }
}
```

**チェック 3:** 誰かが `/new`、`/reset`、またはリセットトリガーを送信しましたか？

<div id="agent-timing-out">
  ### エージェントのタイムアウト
</div>

デフォルトのタイムアウトは 30 分です。長時間かかるタスクの場合:

```json
{
  "reply": {
    "timeoutSeconds": 3600  // 1時間
  }
}
```

または `process` ツールを使用して、長時間実行されるコマンドをバックグラウンドで実行します。

<div id="whatsapp-disconnected">
  ### WhatsApp の切断
</div>

```bash
# Check local status (creds, sessions, queued events)
openclaw status
# 実行中のGateway + チャンネルを調査（WA接続 + Telegram + Discord API）
openclaw status --deep

# View recent connection events
openclaw logs --limit 200 | grep "connection\\|disconnect\\|logout"
```

**対処方法:** 通常は、Gateway が起動していれば自動的に再接続されます。うまくいかない場合は、（どのようにプロセスを監視・管理していても）Gateway プロセスを再起動するか、詳細なログ出力（verbose）を有効にして手動実行してください。

```bash
openclaw gateway --verbose
```

ログアウトしている、またはアカウントのリンクが解除されている場合：

```bash
openclaw channels logout
trash "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/credentials" # ログアウトですべてを完全に削除できない場合
openclaw channels login --verbose       # re-scan QR
```

<div id="media-send-failing">
  ### メディアの送信が失敗する
</div>

**確認 1：** ファイルパスは正しいですか？

```bash
ls -la /path/to/your/image.jpg
```

**チェック2：** サイズが大きすぎないか確認する

* 画像：最大6MB
* 音声／動画：最大16MB
* ドキュメント：最大100MB

**チェック3：** メディアログを確認する

```bash
grep "media\\|fetch\\|download" "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)" | tail -20
```

<div id="high-memory-usage">
  ### メモリ使用量が多い
</div>

OpenClaw では会話履歴をメモリ内に保持します。

**対処法:** 定期的に再起動するか、セッション数の上限を設定します。

```json
{
  "session": {
    "historyLimit": 100  // 保持するメッセージの最大数
  }
}
```

<div id="common-troubleshooting">
  ## よくあるトラブルシューティング
</div>

<div id="gateway-wont-start-configuration-invalid">
  ### 「Gateway が起動しない — 設定が無効」
</div>

設定に未知のキー、壊れた形式の値、または無効な型が含まれている場合、OpenClaw は起動しません。
これは安全性のための意図的な挙動です。

Doctor で修正します:

```bash
openclaw doctor
openclaw doctor --fix
```

Notes:

* `openclaw doctor` は、すべての無効な項目を報告します。
* `openclaw doctor --fix` は、マイグレーションや修復を適用し、設定ファイルを書き換えます。
* `openclaw logs`、`openclaw health`、`openclaw status`、`openclaw gateway status`、`openclaw gateway probe` のような診断コマンドは、設定が無効な状態でも実行できます。

<div id="all-models-failed-what-should-i-check-first">
  ### 「All models failed」と表示される — まず何を確認すればよいですか？
</div>

* 試行しているプロバイダー用の**認証情報**（認証プロファイル + 環境変数）が設定されているか。
* **モデルルーティング**：`agents.defaults.model.primary` およびフォールバック先が、あなたが利用可能なモデルになっているか。
* `/tmp/openclaw/…` 配下の **Gateway ログ** で、該当プロバイダーの具体的なエラー内容を確認。
* **モデルのステータス**：`/model status`（チャット）または `openclaw models status`（CLI）を使って確認。

<div id="im-running-on-my-personal-whatsapp-number-why-is-self-chat-weird">
  ### 自分の個人用 WhatsApp 番号で動かしているのに、なぜ自分とのチャットの挙動がおかしいのですか？
</div>

セルフチャット用のモードを有効にし、自分の番号を許可リストに追加してください。

```json5
{
  channels: {
    whatsapp: {
      selfChatMode: true,
      dmPolicy: "allowlist",
      allowFrom: ["+15555550123"]
    }
  }
}
```

[WhatsApp のセットアップ](/ja/channels/whatsapp) を参照してください。

<div id="whatsapp-logged-me-out-how-do-i-reauth">
  ### WhatsApp からログアウトされました。再認証するには？
</div>

再度ログインコマンドを実行し、QR コードをスキャンしてください。

```bash
openclaw channels login
```

<div id="build-errors-on-main-whats-the-standard-fix-path">
  ### `main` でのビルドエラー — 標準的な復旧手順は？
</div>

1. `git pull origin main && pnpm install`
2. `openclaw doctor`
3. GitHub の issue または Discord を確認する
4. 一時的な回避策: 過去のコミットをチェックアウトする

<div id="npm-install-fails-allow-build-scripts-missing-tar-or-yargs-what-now">
  ### npm install が失敗する（allow-build-scripts / tar や yargs が見つからないなど）。どうすればよいですか？
</div>

ソースから実行している場合は、このリポジトリで指定されているパッケージマネージャである **pnpm**（推奨）を使用してください。
このリポジトリでは `packageManager: "pnpm@…"` が宣言されています。

一般的な復旧手順:

```bash
git status   # リポジトリのルートにいることを確認
pnpm install
pnpm build
openclaw doctor
openclaw gateway restart
```

理由: このリポジトリでは pnpm がパッケージマネージャーとして設定されているためです。

<div id="how-do-i-switch-between-git-installs-and-npm-installs">
  ### git インストールと npm インストールを切り替えるにはどうすればよいですか？
</div>

**website installer** を使用し、フラグ付きのインストール方法を選択します。
その場でアップグレードが行われ、Gateway サービスが新しいインストール先を指すように書き換えられます。

**git インストールに切り替える**:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git --no-onboard
```

**npm のグローバルインストール** に切り替える：

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Notes:

* git フローは、リポジトリがクリーンな場合にのみ rebase を実行します。まず変更を commit するか stash してください。
* 切り替え後に次を実行してください:
  ```bash
  openclaw doctor
  openclaw gateway restart
  ```

<div id="telegram-block-streaming-isnt-splitting-text-between-tool-calls-why">
  ### Telegram のブロックストリーミングがツール呼び出し間でテキストを分割しません。なぜですか？
</div>

ブロックストリーミングは **完了したテキストブロック** のみを送信します。メッセージが 1 つにまとまってしまう主な理由は次のとおりです。

* `agents.defaults.blockStreamingDefault` がまだ `"off"` のままになっている。
* `channels.telegram.blockStreaming` が `false` に設定されている。
* `channels.telegram.streamMode` が `partial` または `block` で、かつ **draft streaming が有効**
  （プライベートチャット + トピック）の場合。このケースでは draft streaming がブロックストリーミングを無効化します。
* `minChars` / coalesce の設定値が高すぎて、チャンクがまとめられてしまっている。
* モデルが 1 つの大きなテキストブロックしか出力しておらず（途中でフラッシュされるポイントがない）、分割されない。

対処チェックリスト:

1. ブロックストリーミングの設定は、ルートではなく `agents.defaults` の下に置く。
2. 本当に複数メッセージのブロック返信にしたい場合は、`channels.telegram.streamMode: "off"` に設定する。
3. デバッグ中は、チャンク / coalesce のしきい値を小さくして試す。

[Streaming](/ja/concepts/streaming) を参照してください。

<div id="discord-doesnt-reply-in-my-server-even-with-requiremention-false-why">
  ### `requireMention: false` にしているのに、Discord がサーバーで返信しません。なぜですか？
</div>

`requireMention` は、そのチャンネルが許可リストを通過した**後**の「メンション必須かどうか」だけを制御します。
デフォルトでは `channels.discord.groupPolicy` は **allowlist** のため、ギルド（サーバー）は明示的に有効化する必要があります。
`channels.discord.guilds.<guildId>.channels` を設定した場合は、記載したチャンネルのみが許可されます。ギルド内のすべてのチャンネルを許可したい場合は、このキー自体を省略してください。

修正チェックリスト:

1. `channels.discord.groupPolicy: "open"` を設定する **または** ギルドの許可リストエントリ（必要に応じてチャンネルの許可リストも）を追加する。
2. `channels.discord.guilds.<guildId>.channels` には **数値のチャンネル ID** を使用する。
3. `requireMention: false` を `channels.discord.guilds`（グローバルまたはチャンネル単位）の**下に**配置する。
   トップレベルの `channels.discord.requireMention` はサポートされていないキーです。
4. Bot に **Message Content Intent** とチャンネル権限が付与されていることを確認する。
5. 調査の手がかりとして `openclaw channels status --probe` を実行する。

ドキュメント: [Discord](/ja/channels/discord), [Channels troubleshooting](/ja/channels/troubleshooting).

<div id="cloud-code-assist-api-error-invalid-tool-schema-400-what-now">
  ### Cloud Code Assist API エラー: invalid tool schema (400)。どうすればよいですか？
</div>

ほとんどの場合、これは**ツールスキーマ互換性**の問題です。Cloud Code Assist
エンドポイントは、JSON Schema の厳密なサブセットしか受け付けません。OpenClaw は
現在の `main` ではツールスキーマをスクラブ／正規化しますが、その修正は
（2026年1月13日時点で）まだ最新リリースには含まれていません。

対処チェックリスト:

1. **OpenClaw を更新する**:
   * ソースから実行できる場合は、`main` を pull して Gateway を再起動してください。
   * それが難しい場合は、schema scrubber を含む次のリリースを待ってください。
2. `anyOf/oneOf/allOf`、`patternProperties`、
   `additionalProperties`、`minLength`、`maxLength`、`format` などの
   サポートされていないキーワードを避けてください。
3. カスタムツールを定義する場合は、トップレベルのスキーマを `type: "object"` とし、
   `properties` と単純な enum のみを使うようにしてください。

[Tools](/ja/tools) および [TypeBox schemas](/ja/concepts/typebox) を参照してください。

<div id="macos-specific-issues">
  ## macOS 特有の問題
</div>

<div id="app-crashes-when-granting-permissions-speechmic">
  ### 権限を許可するとアプリがクラッシュする（音声/マイク）
</div>

プライバシーの許可ダイアログで「Allow（許可）」をクリックしたときに、アプリが消えたり「Abort trap 6」と表示される場合：

**対処法 1: TCC キャッシュをリセットする**

```bash
tccutil reset All bot.molt.mac.debug
```

**対処法 2: 新しい Bundle ID を強制する**
リセットしても改善しない場合は、[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) 内の `BUNDLE_ID` を変更します（例: `.test` というサフィックスを付ける）。そのうえで再ビルドしてください。これにより、macOS はそれを新しいアプリとして扱うようになります。

<div id="gateway-stuck-on-starting">
  ### Gateway が「Starting...」のままになる
</div>

このアプリは、ポート `18789` でローカルの Gateway に接続します。これがずっとこの状態のままの場合は、次を試してください:

**対処法 1: supervisor を停止する（推奨）**
Gateway が launchd によって監視されている場合、PID を kill してもすぐに再起動されてしまいます。まず supervisor を停止してください:

```bash
openclaw gateway status
openclaw gateway stop
# または: launchctl bootout gui/$UID/bot.molt.gateway (bot.molt.<profile> に置き換える; レガシーの com.openclaw.* も引き続き動作します)
```

**対処 2: ポートが使用中（リッスンしているプロセスを特定する）**

```bash
lsof -nP -iTCP:18789 -sTCP:LISTEN
```

対話していないバックグラウンドプロセスの場合は、まずは正常終了（graceful stop）を試し、それでも停止しない場合は、より強い停止手段へ段階的にエスカレートしてください。

```bash
kill -TERM <PID>
sleep 1
kill -9 <PID> # 最後の手段
```

**対処3: CLI のインストールを確認する**
グローバルな `openclaw` CLI がインストールされており、アプリのバージョンと一致していることを確認してください。

```bash
openclaw --version
npm install -g openclaw@<version>
```

<div id="debug-mode">
  ## デバッグモード
</div>

詳細なログを取得するには：

```bash
# 設定でトレースログを有効化:
#   ${OPENCLAW_CONFIG_PATH:-$HOME/.openclaw/openclaw.json} -> { logging: { level: "trace" } }
#
# 次に、デバッグ出力を標準出力にミラーリングするため、verbose コマンドを実行:
openclaw gateway --verbose
openclaw channels login --verbose
```

<div id="log-locations">
  ## ログの保存場所
</div>

| ログ | 場所 |
|-----|----------|
| Gateway ファイルログ（構造化形式） | `/tmp/openclaw/openclaw-YYYY-MM-DD.log`（または `logging.file`） |
| Gateway サービスログ（supervisor 管理） | macOS：`$OPENCLAW_STATE_DIR/logs/gateway.log` + `gateway.err.log`（デフォルト：`~/.openclaw/logs/...`、プロファイル利用時：`~/.openclaw-<profile>/logs/...`）<br />Linux：`journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`<br />Windows：`schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST` |
| セッションファイル | `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/` |
| メディアキャッシュ | `$OPENCLAW_STATE_DIR/media/` |
| 資格情報 | `$OPENCLAW_STATE_DIR/credentials/` |

<div id="health-check">
  ## ヘルスチェック
</div>

```bash
# Supervisor + probe target + config paths
openclaw gateway status
# システムレベルのスキャンを含める (レガシー/追加サービス、ポートリスナー)
openclaw gateway status --deep

# Is the gateway reachable?
openclaw health --json
# If it fails, rerun with connection details:
openclaw health --verbose

# Is something listening on the default port?
lsof -nP -iTCP:18789 -sTCP:LISTEN

# Recent activity (RPC log tail)
openclaw logs --follow
# Fallback if RPC is down
tail -20 /tmp/openclaw/openclaw-*.log
```

<div id="reset-everything">
  ## すべてをリセットする
</div>

最終手段：

```bash
openclaw gateway stop
# サービスをインストール済みでクリーンインストールしたい場合:
# openclaw gateway uninstall

trash "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
openclaw channels login         # WhatsAppを再ペアリング
openclaw gateway restart           # または: openclaw gateway
```

⚠️ これによりすべてのセッションが消去され、WhatsAppの再ペアリングが必要になります。

<div id="getting-help">
  ## ヘルプの求め方
</div>

1. まずログを確認する：`/tmp/openclaw/`（デフォルト：`openclaw-YYYY-MM-DD.log`、または設定済みの `logging.file`）
2. GitHub の既存の Issue を検索する
3. 新しい Issue を作成する際は、次を含める：
   * OpenClaw のバージョン
   * 関連するログの抜粋
   * 再現手順
   * あなたの設定（機密情報は必ずマスクすること！）

***

*「一度電源を切って、入れ直してみましたか？」* — 世界中のあらゆる IT 担当者

🦞🔧

<div id="browser-not-starting-linux">
  ### ブラウザが起動しない (Linux)
</div>

`"Failed to start Chrome CDP on port 18800"` というメッセージが表示される場合:

**最もよくある原因:** Ubuntu 上の Snapパッケージ版 Chromium。

**簡易対処:** 代わりに Google Chrome をインストールしてください:

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
```

次に、設定ファイルで次のように設定します：

```json
{
  "browser": {
    "executablePath": "/usr/bin/google-chrome-stable"
  }
}
```

**詳細なガイド:** [browser-linux-troubleshooting](/ja/tools/browser-linux-troubleshooting) を参照してください
