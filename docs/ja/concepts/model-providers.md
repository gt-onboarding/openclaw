---
title: モデルプロバイダー
summary: "モデルプロバイダーの概要（設定例 + CLI フロー付き）"
read_when:
  - プロバイダーごとのモデルセットアップ用リファレンスが必要なとき
  - モデルプロバイダー用の設定例や、CLI による初期設定用コマンドが欲しいとき
---

<div id="model-providers">
  # モデルプロバイダー
</div>

このページでは、**LLM／モデルプロバイダー**（WhatsApp や Telegram のようなチャットチャネルではなく）について説明します。
モデル選択ルールについては、[/concepts/models](/ja/concepts/models) を参照してください。

<div id="quick-rules">
  ## 簡易ルール
</div>

* モデル参照には `provider/model` を使用します（例: `opencode/claude-opus-4-5`）。
* `agents.defaults.models` を設定すると、それが許可リストとして扱われます。
* CLI ヘルパーコマンド: `openclaw onboard`, `openclaw models list`, `openclaw models set <provider/model>`。

<div id="built-in-providers-pi-ai-catalog">
  ## 組み込みプロバイダー（pi-ai カタログ）
</div>

OpenClaw には pi‑ai カタログが標準で含まれています。これらのプロバイダーを利用するために **一切**
`models.providers` 設定は不要です。認証情報を設定して、モデルを選ぶだけです。

<div id="openai">
  ### OpenAI
</div>

* プロバイダー: `openai`
* 認証: `OPENAI_API_KEY`
* モデル例: `openai/gpt-5.2`
* CLI: `openclaw onboard --auth-choice openai-api-key`

```json5
{
  agents: { defaults: { model: { primary: "openai/gpt-5.2" } } }
}
```

<div id="anthropic">
  ### Anthropic
</div>

* プロバイダー: `anthropic`
* 認証: `ANTHROPIC_API_KEY` または `claude setup-token`
* モデル例: `anthropic/claude-opus-4-5`
* CLI: `openclaw onboard --auth-choice token`（setup-token を貼り付け）または `openclaw models auth paste-token --provider anthropic`

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="openai-code-codex">
  ### OpenAI Code（Codex）
</div>

* プロバイダー: `openai-codex`
* 認証: OAuth（ChatGPT）
* 代表的なモデル: `openai-codex/gpt-5.2`
* CLI: `openclaw onboard --auth-choice openai-codex` または `openclaw models auth login --provider openai-codex`

```json5
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.2" } } }
}
```

<div id="opencode-zen">
  ### OpenCode Zen
</div>

* プロバイダー: `opencode`
* 認証: `OPENCODE_API_KEY` (または `OPENCODE_ZEN_API_KEY`)
* モデル例: `opencode/claude-opus-4-5`
* CLI: `openclaw onboard --auth-choice opencode-zen`

```json5
{
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-5" } } }
}
```

<div id="google-gemini-api-key">
  ### Google Gemini（APIキー）
</div>

* プロバイダー: `google`
* 認証: `GEMINI_API_KEY`
* モデル例: `google/gemini-3-pro-preview`
* CLI: `openclaw onboard --auth-choice gemini-api-key`

<div id="google-vertex-antigravity-gemini-cli">
  ### Google Vertex / Antigravity / Gemini CLI
</div>

* プロバイダー: `google-vertex`, `google-antigravity`, `google-gemini-cli`
* 認証: Vertex は gcloud ADC を使用し、Antigravity/Gemini CLI はそれぞれ固有の認証フローを使用します
* Antigravity の OAuth はバンドル済みプラグイン（`google-antigravity-auth`、デフォルトでは無効）として提供されています。
  * 有効化: `openclaw plugins enable google-antigravity-auth`
  * ログイン: `openclaw models auth login --provider google-antigravity --set-default`
* Gemini CLI の OAuth はバンドル済みプラグイン（`google-gemini-cli-auth`、デフォルトでは無効）として提供されています。
  * 有効化: `openclaw plugins enable google-gemini-cli-auth`
  * ログイン: `openclaw models auth login --provider google-gemini-cli --set-default`
  * 注意: `openclaw.json` にクライアント ID やクライアントシークレットを貼り付ける必要は**ありません**。CLI のログインフローが、
    トークンを Gateway が動作しているホスト上の認証プロファイルに保存します。

<div id="zai-glm">
  ### Z.AI (GLM)
</div>

* プロバイダー: `zai`
* 認証: `ZAI_API_KEY`
* モデル例: `zai/glm-4.7`
* CLI: `openclaw onboard --auth-choice zai-api-key`
  * エイリアス: `z.ai/*` および `z-ai/*` は `zai/*` に正規化されます

<div id="vercel-ai-gateway">
  ### Vercel AI Gateway
</div>

* プロバイダー: `vercel-ai-gateway`
* 認証情報: `AI_GATEWAY_API_KEY`
* モデル例: `vercel-ai-gateway/anthropic/claude-opus-4.5`
* CLI: `openclaw onboard --auth-choice ai-gateway-api-key`

<div id="other-built-in-providers">
  ### その他の組み込みプロバイダー
</div>

* OpenRouter: `openrouter` (`OPENROUTER_API_KEY`)
* モデル例: `openrouter/anthropic/claude-sonnet-4-5`
* xAI: `xai` (`XAI_API_KEY`)
* Groq: `groq` (`GROQ_API_KEY`)
* Cerebras: `cerebras` (`CEREBRAS_API_KEY`)
  * Cerebras 上の GLM モデルは `zai-glm-4.7` および `zai-glm-4.6` の ID を使用します。
  * OpenAI 互換のベース URL: `https://api.cerebras.ai/v1`。
* Mistral: `mistral` (`MISTRAL_API_KEY`)
* GitHub Copilot: `github-copilot` (`COPILOT_GITHUB_TOKEN` / `GH_TOKEN` / `GITHUB_TOKEN`)

<div id="providers-via-modelsproviders-custombase-url">
  ## `models.providers`（カスタム / ベース URL）経由のプロバイダー
</div>

`models.providers`（または `models.json`）を使用して、**カスタム**プロバイダーや
OpenAI／Anthropic 互換プロキシを追加します。

<div id="moonshot-ai-kimi">
  ### Moonshot AI (Kimi)
</div>

Moonshot は OpenAI 互換のエンドポイントを使用しているため、カスタムプロバイダーとして設定します:

* プロバイダー: `moonshot`
* 認証: `MOONSHOT_API_KEY`
* モデル例: `moonshot/kimi-k2.5`
* Kimi K2 モデル ID:
  {/* moonshot-kimi-k2-model-refs:start */}
  * `moonshot/kimi-k2.5`
  * `moonshot/kimi-k2-0905-preview`
  * `moonshot/kimi-k2-turbo-preview`
  * `moonshot/kimi-k2-thinking`
  * `moonshot/kimi-k2-thinking-turbo`
  {/* moonshot-kimi-k2-model-refs:end */}

```json5
{
  agents: {
    defaults: { model: { primary: "moonshot/kimi-k2.5" } }
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [{ id: "kimi-k2.5", name: "Kimi K2.5" }]
      }
    }
  }
}
```

<div id="kimi-code">
  ### Kimi Code
</div>

Kimi Code は Moonshot とは別の専用エンドポイントとキーを使用します:

* プロバイダー: `kimi-code`
* API キー: `KIMICODE_API_KEY`
* モデルの例: `kimi-code/kimi-for-coding`

```json5
{
  env: { KIMICODE_API_KEY: "sk-..." },
  agents: {
    defaults: { model: { primary: "kimi-code/kimi-for-coding" } }
  },
  models: {
    mode: "merge",
    providers: {
      "kimi-code": {
        baseUrl: "https://api.kimi.com/coding/v1",
        apiKey: "${KIMICODE_API_KEY}",
        api: "openai-completions",
        models: [{ id: "kimi-for-coding", name: "Kimi For Coding" }]
      }
    }
  }
}
```

<div id="qwen-oauth-free-tier">
  ### Qwen OAuth（無料プラン）
</div>

Qwen は、デバイスコードフローにより Qwen Coder + Vision への OAuth アクセスを提供しています。
バンドル済みのプラグインを有効にしてから、ログインします：

```bash
openclaw plugins enable qwen-portal-auth
openclaw models auth login --provider qwen-portal --set-default
```

モデル参照:

* `qwen-portal/coder-model`
* `qwen-portal/vision-model`

セットアップの詳細と注意事項については、[/providers/qwen](/ja/providers/qwen) を参照してください。

<div id="synthetic">
  ### Synthetic
</div>

Synthetic は、`synthetic` プロバイダー経由で Anthropic 互換モデルを提供します：

* プロバイダー: `synthetic`
* 認証情報: `SYNTHETIC_API_KEY`
* モデル例: `synthetic/hf:MiniMaxAI/MiniMax-M2.1`
* CLI: `openclaw onboard --auth-choice synthetic-api-key`

```json5
{
  agents: {
    defaults: { model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" } }
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [{ id: "hf:MiniMaxAI/MiniMax-M2.1", name: "MiniMax M2.1" }]
      }
    }
  }
}
```

<div id="minimax">
  ### MiniMax
</div>

MiniMax はカスタムエンドポイントを使用するため、`models.providers` で設定します：

* MiniMax（Anthropic 互換）: `--auth-choice minimax-api`
* 認証: `MINIMAX_API_KEY`

セットアップ手順、モデルオプション、設定スニペットについては [/providers/minimax](/ja/providers/minimax) を参照してください。

<div id="ollama">
  ### Ollama
</div>

Ollama は、OpenAI 互換 API を提供するローカル LLM ランタイムです：

* プロバイダー: `ollama`
* 認証: 不要（ローカルサーバー）
* モデル例: `ollama/llama3.3`
* インストール: https://ollama.ai

```bash
# Ollamaをインストールし、モデルをプルする:
ollama pull llama3.3
```

```json5
{
  agents: {
    defaults: { model: { primary: "ollama/llama3.3" } }
  }
}
```

Ollama はローカル環境で `http://127.0.0.1:11434/v1` で実行されている場合、自動的に検出されます。推奨モデルやカスタム設定については [/providers/ollama](/ja/providers/ollama) を参照してください。

<div id="local-proxies-lm-studio-vllm-litellm-etc">
  ### ローカルプロキシ（LM Studio、vLLM、LiteLLM など）
</div>

例（OpenAI 互換）：

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: { "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" } }
    }
  },
  models: {
    providers: {
      lmstudio: {
        baseUrl: "http://localhost:1234/v1",
        apiKey: "LMSTUDIO_KEY",
        api: "openai-completions",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 200000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

Notes:

* カスタムプロバイダーの場合、`reasoning`、`input`、`cost`、`contextWindow`、`maxTokens` は省略可能です。
  省略された場合、OpenClaw のデフォルト値は次のとおりです:
  * `reasoning: false`
  * `input: ["text"]`
  * `cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 }`
  * `contextWindow: 200000`
  * `maxTokens: 8192`
* 推奨: 利用しているプロキシ／モデルの制約に合わせて、明示的な値を設定してください。

<div id="cli-examples">
  ## CLI の例
</div>

```bash
openclaw onboard --auth-choice opencode-zen
openclaw models set opencode/claude-opus-4-5
openclaw models list
```

完全な設定例については、[/gateway/configuration](/ja/gateway/configuration) も参照してください。
