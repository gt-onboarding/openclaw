---
title: Qwen
summary: "OpenClaw で Qwen OAuth（無料プラン）を使う"
read_when:
  - OpenClaw で Qwen を使いたい
  - Qwen Coder への無料プラン向け OAuth アクセスを使いたい
---

<div id="qwen">
  # Qwen
</div>

Qwen は、Qwen Coder モデルと Qwen Vision モデル向けに無料枠の OAuth フローを提供しています
（1 日あたり 2,000 リクエストで、Qwen が定めるレート制限の対象です）。

<div id="enable-the-plugin">
  ## プラグインを有効にする
</div>

```bash
openclaw plugins enable qwen-portal-auth
```

有効化後に Gateway を再起動してください。

<div id="authenticate">
  ## 認証
</div>

```bash
openclaw models auth login --provider qwen-portal --set-default
```

これは Qwen のデバイスコード方式の OAuth フローを実行し、`models.json` にプロバイダーのエントリ（および素早く切り替えるための `qwen` エイリアス）を書き込みます。

<div id="model-ids">
  ## モデル ID
</div>

* `qwen-portal/coder-model`
* `qwen-portal/vision-model`

モデルは次のように切り替えます：

```bash
openclaw models set qwen-portal/coder-model
```

<div id="reuse-qwen-code-cli-login">
  ## Qwen Code CLI ログインを再利用する
</div>

すでに Qwen Code CLI にログインしている場合、OpenClaw は認証ストアを読み込むときに
`~/.qwen/oauth_creds.json` から認証情報を同期します。なお、
`models.providers.qwen-portal` エントリは引き続き必要です（上記の login コマンドを使って作成してください）。

<div id="notes">
  ## 注意事項
</div>

* トークンは自動的に更新されます。更新に失敗した場合やアクセスが取り消された場合は、ログインコマンドを再実行してください。
* デフォルトのベースURL: `https://portal.qwen.ai/v1`（Qwen から別のエンドポイントが提供されている場合は、
  `models.providers.qwen-portal.baseUrl` で上書きしてください）。
* プロバイダー全体に適用される共通ルールについては、[Model providers](/ja/concepts/model-providers) を参照してください。