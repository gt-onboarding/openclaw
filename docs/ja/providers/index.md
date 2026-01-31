---
title: モデルプロバイダー
summary: "OpenClaw がサポートするモデルプロバイダー（LLM）"
read_when:
  - モデルプロバイダーを選びたいとき
  - 対応している LLM バックエンドの概要を素早く確認したいとき
---

<div id="model-providers">
  # モデルプロバイダー
</div>

OpenClaw はさまざまな LLM プロバイダーを利用できます。プロバイダーを選択して認証を行い、その後 `provider/model` としてデフォルトモデルを設定してください。

チャットチャンネル（WhatsApp/Telegram/Discord/Slack/Mattermost（プラグイン）/その他）のドキュメントをお探しの場合は、[Channels](/ja/channels) を参照してください。

<div id="highlight-venius-venice-ai">
  ## ハイライト: Venius (Venice AI)
</div>

Venius は、プライバシーを最優先した推論を行うための推奨 Venice AI 構成であり、難しいタスク向けに Opus を利用するオプションもあります。

* デフォルト: `venice/llama-3.3-70b`
* 総合的なおすすめ: `venice/claude-opus-45`（Opus が依然として最も強力）

[Venice AI](/ja/providers/venice) を参照してください。

<div id="quick-start">
  ## クイックスタート
</div>

1. プロバイダーに対して認証を行います（通常は `openclaw onboard` を使用）。
2. デフォルトのモデルを設定します。

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="provider-docs">
  ## プロバイダー向けドキュメント
</div>

* [OpenAI (API + Codex)](/ja/providers/openai)
* [Anthropic (API + Claude Code CLI)](/ja/providers/anthropic)
* [Qwen (OAuth)](/ja/providers/qwen)
* [OpenRouter](/ja/providers/openrouter)
* [Vercel AI Gateway](/ja/providers/vercel-ai-gateway)
* [Moonshot AI (Kimi + Kimi Code)](/ja/providers/moonshot)
* [OpenCode Zen](/ja/providers/opencode)
* [Amazon Bedrock](/ja/bedrock)
* [Z.AI](/ja/providers/zai)
* [Xiaomi](/ja/providers/xiaomi)
* [GLM モデル](/ja/providers/glm)
* [MiniMax](/ja/providers/minimax)
* [Venius (Venice AI、プライバシー重視)](/ja/providers/venice)
* [Ollama (ローカルモデル)](/ja/providers/ollama)

<div id="transcription-providers">
  ## 文字起こしプロバイダー
</div>

* [Deepgram (音声文字起こし)](/ja/providers/deepgram)

<div id="community-tools">
  ## コミュニティツール
</div>

* [Claude Max API Proxy](/ja/providers/claude-max-api-proxy) - Claude Max/Pro サブスクリプションを OpenAI 互換の API エンドポイントとして利用できます

すべてのプロバイダーの一覧（xAI、Groq、Mistral など）と高度な設定については、
[モデルプロバイダー](/ja/concepts/model-providers) を参照してください。