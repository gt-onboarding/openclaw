---
title: 認証
summary: "モデル認証：OAuth、API キー、setup-token"
read_when:
  - モデル認証や OAuth の有効期限切れをデバッグするとき
  - 認証や認証情報の保存方法を文書化するとき
---

<div id="authentication">
  # 認証
</div>

OpenClaw はモデルプロバイダー用に OAuth と API キーの両方をサポートしています。Anthropic
アカウントの場合は、**API キー**の使用を推奨します。Claude のサブスクリプションアクセスには、
`claude setup-token` によって作成される長期間有効なトークンを使用してください。

OAuth の全体的なフローおよびストレージレイアウトについては、[/concepts/oauth](/ja/concepts/oauth) を参照してください。

<div id="recommended-anthropic-setup-api-key">
  ## 推奨される Anthropic のセットアップ（API キー）
</div>

Anthropic を直接利用する場合は、API キーを使用してください。

1. Anthropic Console で API キーを作成します。
2. そのキーを **Gateway ホスト**（`openclaw gateway` を実行しているマシン）に保存します。

```bash
export ANTHROPIC_API_KEY="..."
openclaw models status
```

3. Gateway が systemd/launchd 上で動作している場合は、デーモンがそれを読み取れるように
   `~/.openclaw/.env` にキーを配置することを推奨します：

```bash
cat >> ~/.openclaw/.env <<'EOF'
ANTHROPIC_API_KEY=...
EOF
```

その後、デーモン（または Gateway プロセス）を再起動し、もう一度確認してください。

```bash
openclaw models status
openclaw doctor
```

自分で環境変数を管理したくない場合は、オンボーディングウィザードを使って
デーモンが利用する API キーを保存できます: `openclaw onboard`。

環境変数の継承 (`env.shellEnv`、`~/.openclaw/.env`、systemd/launchd) の詳細は [Help](/ja/help) を参照してください。

<div id="anthropic-setup-token-subscription-auth">
  ## Anthropic: setup-token（サブスクリプション認証）
</div>

Anthropic では、推奨される手順は **API key** を利用することです。Claude の
サブスクリプションを利用している場合は、setup-token フローもサポートされています。**Gateway ホスト**上で実行してください。

```bash
claude setup-token
```

次に、それを OpenClaw に貼り付けます。

```bash
openclaw models auth setup-token --provider anthropic
```

トークンを別のマシンで作成した場合は、ここに手動で貼り付けてください。

```bash
openclaw models auth paste-token --provider anthropic
```

次のような Anthropic のエラーが表示された場合は、

```
This credential is only authorized for use with Claude Code and cannot be used for other API requests.
```

…代わりに Anthropic の API キーを使用してください。

手動でのトークン入力（任意のプロバイダー；`auth-profiles.json` への書き込みと設定の更新）:

```bash
openclaw models auth paste-token --provider anthropic
openclaw models auth paste-token --provider openrouter
```

自動化しやすいチェック（期限切れ・未設定の場合は終了コード `1`、有効期限が近い場合は `2` で終了）:

```bash
openclaw models status --check
```

オプションの運用用スクリプト（systemd / Termux）については、こちらに記載しています：
[/automation/auth-monitoring](/ja/automation/auth-monitoring)

> `claude setup-token` には対話的な TTY が必要です。

<div id="checking-model-auth-status">
  ## モデル認証状態の確認
</div>

```bash
openclaw models status
openclaw doctor
```

<div id="controlling-which-credential-is-used">
  ## どの認証情報を使用するかを制御する
</div>

<div id="per-session-chat-command">
  ### セッション単位（チャットコマンド）
</div>

現在のセッションで使用する特定のプロバイダー認証情報を固定するには、`/model <alias-or-id>@<profileId>` を使用します（プロファイルIDの例: `anthropic:default`, `anthropic:work`）。

簡易な選択メニューとしては `/model`（または `/model list`）を使用します。詳細ビュー（候補 + 次に使用される認証プロファイル、さらに設定されている場合はプロバイダーのエンドポイント詳細）には `/model status` を使用します。

<div id="per-agent-cli-override">
  ### エージェントごと（CLI での上書き）
</div>

エージェントに対して明示的な認証プロファイル順序のオーバーライドを設定します（そのエージェントの `auth-profiles.json` に保存されます）:

```bash
openclaw models auth order get --provider anthropic
openclaw models auth order set --provider anthropic anthropic:default
openclaw models auth order clear --provider anthropic
```

特定のエージェントを指定するには `--agent <id>` を使用します。省略すると、設定済みのデフォルトのエージェントが使用されます。

<div id="troubleshooting">
  ## トラブルシューティング
</div>

<div id="no-credentials-found">
  ### 「認証情報が見つかりません」
</div>

Anthropic のトークンプロファイルが見つからない場合は、**Gateway ホスト**上で `claude setup-token` を実行し、その後もう一度確認してください。

```bash
openclaw models status
```

<div id="token-expiringexpired">
  ### トークンの有効期限が近い／期限切れ
</div>

どのプロファイルのトークンが有効期限間近かを確認するには、`openclaw models status` を実行します。プロファイルが表示されない場合は、`claude setup-token` を再実行し、トークンを再度貼り付けてください。

<div id="requirements">
  ## 要件
</div>

* Claude Max または Pro のサブスクリプション（`claude setup-token` コマンド用）
* Claude Code CLI がインストールされており、`claude` コマンドが使用可能であること