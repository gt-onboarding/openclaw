---
title: 環境
summary: "OpenClaw が環境変数をどこから読み込むかと、その優先順位"
read_when:
  - どの環境変数が読み込まれるかと、その読み込み順序を知る必要がある
  - Gateway で API キーが見つからない問題をデバッグしている
  - プロバイダーの認証やデプロイ環境についてドキュメントを作成している
---

<div id="environment-variables">
  # 環境変数
</div>

OpenClaw は複数のソースから環境変数を取得します。原則として、**既存の値を絶対に上書きしない** ようにします。

<div id="precedence-highest-lowest">
  ## 優先順位（高い → 低い）
</div>

1. **プロセス環境**（親シェル／デーモンから Gateway プロセスがすでに引き継いでいるもの）。
2. **現在の作業ディレクトリ内の `.env`**（dotenv のデフォルト。既存の値は上書きしない）。
3. **グローバル `.env`**（`~/.openclaw/.env`、別名 `$OPENCLAW_STATE_DIR/.env`。既存の値は上書きしない）。
4. `~/.openclaw/openclaw.json` 内の **Config `env` ブロック**（値が存在しない場合にのみ適用）。
5. **オプションのログインシェルからの取り込み**（`env.shellEnv.enabled` または `OPENCLAW_LOAD_SHELL_ENV=1`。想定されるキーが欠けている場合にのみ適用）。

設定ファイルがまったく存在しない場合、手順 4 はスキップされます。シェルからの取り込みは、有効化されていれば引き続き実行されます。

<div id="config-env-block">
  ## Config `env` ブロック
</div>

インラインの環境変数を設定する方法は同等なものが 2 つあります（どちらも既存の値を上書きしません）。

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-..."
    }
  }
}
```

<div id="shell-env-import">
  ## シェル環境の取り込み
</div>

`env.shellEnv` はログインシェルを実行し、想定されている環境変数キーのうち **まだ存在しないものだけ** を取り込みます。

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

同等の環境変数:

* `OPENCLAW_LOAD_SHELL_ENV=1`
* `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

<div id="env-var-substitution-in-config">
  ## 設定における環境変数の展開
</div>

設定の文字列値では、`${VAR_NAME}` 構文を使って環境変数を直接参照できます。

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}"
      }
    }
  }
}
```

詳しくは、[Configuration: 環境変数の置換](/ja/gateway/configuration#env-var-substitution-in-config) を参照してください。

<div id="related">
  ## 関連項目
</div>

* [Gateway の設定](/ja/gateway/configuration)
* [FAQ: 環境変数と .env の読み込み](/ja/help/faq#env-vars-and-env-loading)
* [モデルの概要](/ja/concepts/models)