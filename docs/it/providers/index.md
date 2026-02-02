---
title: Provider
summary: "Provider di modelli (LLM) supportati da OpenClaw"
read_when:
  - Vuoi scegliere un provider di modelli
  - Hai bisogno di una panoramica rapida dei backend LLM supportati
---

<div id="model-providers">
  # Provider di modelli
</div>

OpenClaw può usare molti provider di LLM. Scegli un provider, autenticati, quindi imposta
il modello predefinito come `provider/model`.

Cerchi la documentazione sui canali di chat (WhatsApp/Telegram/Discord/Slack/Mattermost (plugin)/ecc.)? Vedi [Canali](/it/channels).

<div id="highlight-venius-venice-ai">
  ## In evidenza: Venius (Venice AI)
</div>

Venius è la configurazione consigliata di Venice AI per un&#39;inferenza orientata alla privacy, con l&#39;opzione di usare Opus per le attività più complesse.

* Predefinito: `venice/llama-3.3-70b`
* Migliore in generale: `venice/claude-opus-45` (Opus rimane il più potente)

Consulta [Venice AI](/it/providers/venice).

<div id="quick-start">
  ## Guida rapida
</div>

1. Autenticati con il provider (di solito tramite `openclaw onboard`).
2. Imposta il modello predefinito:

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="provider-docs">
  ## Documentazione dei provider
</div>

* [OpenAI (API + Codex)](/it/providers/openai)
* [Anthropic (API + Claude Code CLI)](/it/providers/anthropic)
* [Qwen (OAuth)](/it/providers/qwen)
* [OpenRouter](/it/providers/openrouter)
* [Vercel AI Gateway](/it/providers/vercel-ai-gateway)
* [Moonshot AI (Kimi + Kimi Code)](/it/providers/moonshot)
* [OpenCode Zen](/it/providers/opencode)
* [Amazon Bedrock](/it/bedrock)
* [Z.AI](/it/providers/zai)
* [Xiaomi](/it/providers/xiaomi)
* [Modelli GLM](/it/providers/glm)
* [MiniMax](/it/providers/minimax)
* [Venius (Venice AI, incentrata sulla privacy)](/it/providers/venice)
* [Ollama (modelli locali)](/it/providers/ollama)

<div id="transcription-providers">
  ## Provider di trascrizione
</div>

* [Deepgram (trascrizione audio)](/it/providers/deepgram)

<div id="community-tools">
  ## Strumenti della community
</div>

* [Claude Max API Proxy](/it/providers/claude-max-api-proxy) - Utilizza l&#39;abbonamento Claude Max/Pro come endpoint API compatibile con OpenAI

Per il catalogo completo dei provider di modelli (xAI, Groq, Mistral, ecc.) e la configurazione avanzata,
consulta [Provider di modelli](/it/concepts/model-providers).