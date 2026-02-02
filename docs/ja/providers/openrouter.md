---
title: Openrouter
summary: "OpenClaw で多くのモデルにアクセスするために OpenRouter の統合 API を使う"
read_when:
  - 複数の LLM を 1 つの API キーで利用したい場合
  - OpenClaw で OpenRouter 経由でモデルを実行したい場合
---

<div id="openrouter">
  # OpenRouter
</div>

OpenRouter は、単一のエンドポイントと API キーで多数のモデルへのリクエストを振り分ける、**統一 API** を提供します。OpenAI と互換性があるため、ほとんどの OpenAI SDK は base URL を切り替えるだけで動作します。

<div id="cli-setup">
  ## CLI の設定
</div>

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

<div id="config-snippet">
  ## 設定例
</div>

```json5
{
  env: { OPENROUTER_API_KEY: "sk-or-..." },
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" }
    }
  }
}
```

<div id="notes">
  ## 注意事項
</div>

* モデル参照は `openrouter/<provider>/<model>` です。
* 利用可能なモデルおよびプロバイダーのオプションについては、[/concepts/model-providers](/ja/concepts/model-providers) を参照してください。
* OpenRouter は内部的に、API キーを Bearer トークンとして利用します。