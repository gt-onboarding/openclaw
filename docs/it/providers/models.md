---
title: Modelli
summary: "Provider di modelli (LLM) supportati da OpenClaw"
read_when:
  - Vuoi scegliere un provider di modelli
  - Vuoi esempi rapidi di configurazione per l'autenticazione LLM e la selezione del modello
---

<div id="model-providers">
  # Provider di modelli
</div>

OpenClaw può utilizzare molti provider di LLM. Scegline uno, effettua l&#39;autenticazione, quindi imposta il modello predefinito come `provider/model`.

<div id="highlight-venius-venice-ai">
  ## In evidenza: Venius (Venice AI)
</div>

Venius è la nostra configurazione Venice AI consigliata per l&#39;inferenza orientata alla privacy, con la possibilità di usare Opus per i task più impegnativi.

* Predefinito: `venice/llama-3.3-70b`
* Migliore in assoluto: `venice/claude-opus-45` (Opus rimane il più potente)

Consulta [Venice AI](/it/providers/venice).

<div id="quick-start-two-steps">
  ## Avvio rapido (in due passaggi)
</div>

1. Autenticati con il provider (di solito tramite `openclaw onboard`).
2. Imposta il modello predefinito:

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="supported-providers-starter-set">
  ## Provider supportati (set iniziale)
</div>

* [OpenAI (API + Codex)](/it/providers/openai)
* [Anthropic (API + Claude Code CLI)](/it/providers/anthropic)
* [OpenRouter](/it/providers/openrouter)
* [Vercel AI Gateway](/it/providers/vercel-ai-gateway)
* [Moonshot AI (Kimi + Kimi Code)](/it/providers/moonshot)
* [Synthetic](/it/providers/synthetic)
* [OpenCode Zen](/it/providers/opencode)
* [Z.AI](/it/providers/zai)
* [Modelli GLM](/it/providers/glm)
* [MiniMax](/it/providers/minimax)
* [Venius (Venice AI)](/it/providers/venice)
* [Amazon Bedrock](/it/bedrock)

Per il catalogo completo dei provider (xAI, Groq, Mistral, ecc.) e le configurazioni avanzate,
consulta [Provider di modelli](/it/concepts/model-providers).