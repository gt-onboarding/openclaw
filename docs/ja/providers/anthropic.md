---
title: Anthropic
summary: "OpenClaw で API キーまたは setup-token を使って Anthropic Claude モデルを利用する"
read_when:
  - OpenClaw で Anthropic モデルを利用したい
  - API キーの代わりに setup-token を使いたい
---

<div id="anthropic-claude">
  # Anthropic (Claude)
</div>

Anthropic は **Claude** モデルファミリーを開発しており、API 経由で利用できます。
OpenClaw では、API キーまたは **setup-token** を使って認証を行えます。

<div id="option-a-anthropic-api-key">
  ## オプション A: Anthropic API キー
</div>

**推奨用途:** 標準的な API アクセスと従量課金での利用。
Anthropic Console で API キーを作成してください。

<div id="cli-setup">
  ### CLI のセットアップ
</div>

```bash
openclaw onboard
# 選択: Anthropic API キー

# or non-interactive
openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
```

### 設定例

```json5
{
  env: { ANTHROPIC_API_KEY: "sk-ant-..." },
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="prompt-caching-anthropic-api">
  ## プロンプトキャッシュ (Anthropic API)
</div>

OpenClaw は、設定しない限り Anthropic のデフォルトのキャッシュ TTL を**上書きしません**。
これは **API のみ** 有効であり、サブスクリプション認証では TTL 設定は考慮されません。

モデルごとに TTL を設定するには、モデルの `params` 内で `cacheControlTtl` を使用します。

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": {
          params: { cacheControlTtl: "5m" } // または "1h"
        }
      }
    }
  }
}
```

OpenClaw には Anthropic API リクエスト向けの `extended-cache-ttl-2025-04-11` ベータフラグが含まれています。プロバイダーのヘッダーを上書きする場合でも、このフラグは残しておいてください（[/gateway/configuration](/ja/gateway/configuration) を参照）。

<div id="option-b-claude-setup-token">
  ## オプションB: Claude setup-token
</div>

**最適な用途:** 自分の Claude サブスクリプションを利用する場合。

<div id="where-to-get-a-setup-token">
  ### setup-token の取得場所
</div>

setup-token は **Claude Code CLI** で作成します。Anthropic Console では作成できません。これは **どのマシンでも** 実行できます。

```bash
claude setup-token
```

トークンを OpenClaw のウィザード（**Anthropic token (paste setup-token)**）に貼り付けるか、Gateway ホスト上で次のコマンドを実行します：

```bash
openclaw models auth setup-token --provider anthropic
```

別のマシンでトークンを生成した場合は、ここに貼り付けてください:

```bash
openclaw models auth paste-token --provider anthropic
```

<div id="cli-setup">
  ### CLI のセットアップ
</div>

```bash
# オンボーディング時にsetup-tokenを貼り付ける
openclaw onboard --auth-choice setup-token
```

<div id="config-snippet">
  ### 設定スニペット
</div>

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="notes">
  ## 注意事項
</div>

* `claude setup-token` で setup-token を生成して貼り付けるか、Gateway ホスト上で `openclaw models auth setup-token` を実行します。
* Claude サブスクリプションで「OAuth token refresh failed …」と表示された場合は、setup-token を使って再認証してください。詳しくは [/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription](/ja/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription) を参照してください。
* 認証に関する詳細および再利用ルールは [/concepts/oauth](/ja/concepts/oauth) に記載されています。

<div id="troubleshooting">
  ## トラブルシューティング
</div>

**401 エラー / トークンが突然無効になる**

* Claude のサブスクリプション認証は有効期限切れや取り消しが発生することがあります。`claude setup-token` を再実行し、生成されたトークンを **Gateway ホスト** に貼り付けてください。
* Claude CLI のログインが別マシンにある場合は、Gateway ホスト上で
  `openclaw models auth paste-token --provider anthropic` を実行します。

**プロバイダー &quot;anthropic&quot; の API キーが見つからない**

* 認証は **エージェント単位** です。新しいエージェントはメインエージェントのキーを継承しません。
* そのエージェントでオンボーディングを再実行するか、Gateway ホスト上で setup-token / API key を貼り付け、`openclaw models status` で確認してください。

**プロファイル `anthropic:default` の認証情報が見つからない**

* `openclaw models status` を実行して、どの認証プロファイルがアクティブか確認します。
* オンボーディングを再実行するか、そのプロファイル用の setup-token / API key を貼り付けてください。

**利用可能な認証プロファイルがない（すべてクールダウン中 / 利用不可）**

* `openclaw models status --json` を実行し、`auth.unusableProfiles` を確認します。
* 追加の Anthropic プロファイルを作成するか、クールダウンが終わるまで待ってください。

詳細: [/gateway/troubleshooting](/ja/gateway/troubleshooting) および [/help/faq](/ja/help/faq) を参照してください。