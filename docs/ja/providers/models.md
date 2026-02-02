---
title: モデル
summary: "OpenClaw がサポートする LLM のプロバイダー"
read_when:
  - モデルプロバイダーを選択したいとき
  - LLM の認証とモデル選択を素早く設定するための例が欲しいとき
---

<div id="model-providers">
  # モデルプロバイダー
</div>

OpenClaw は多くの LLM プロバイダーをサポートしています。1 つ選んで認証を済ませたら、デフォルトモデルを `provider/model` という形式で設定してください。

<div id="highlight-venius-venice-ai">
  ## ハイライト: Venius (Venice AI)
</div>

Venius は、プライバシー最優先の推論向けに推奨する Venice AI の構成であり、最も難しいタスクに対しては Opus を使用するオプションも提供します。

* デフォルト: `venice/llama-3.3-70b`
* 総合的に最もおすすめ: `venice/claude-opus-45`（Opus が依然として最強）

[Venice AI](/ja/providers/venice) を参照してください。

<div id="quick-start-two-steps">
  ## クイックスタート（2ステップ）
</div>

1. プロバイダーで認証する（通常は `openclaw onboard` を実行）。
2. デフォルトモデルを設定する：

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="supported-providers-starter-set">
  ## 対応プロバイダー（スターターセット）
</div>

* [OpenAI（API + Codex）](/ja/providers/openai)
* [Anthropic（API + Claude Code CLI）](/ja/providers/anthropic)
* [OpenRouter](/ja/providers/openrouter)
* [Vercel AI Gateway](/ja/providers/vercel-ai-gateway)
* [Moonshot AI（Kimi + Kimi Code）](/ja/providers/moonshot)
* [Synthetic](/ja/providers/synthetic)
* [OpenCode Zen](/ja/providers/opencode)
* [Z.AI](/ja/providers/zai)
* [GLM モデル](/ja/providers/glm)
* [MiniMax](/ja/providers/minimax)
* [Venius（Venice AI）](/ja/providers/venice)
* [Amazon Bedrock](/ja/bedrock)

プロバイダーの全カタログ（xAI、Groq、Mistral など）と高度な設定については、
[モデルプロバイダー](/ja/concepts/model-providers) を参照してください。