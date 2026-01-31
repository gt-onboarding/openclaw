---
title: Modelle
summary: "Von OpenClaw unterstützte Modellanbieter (LLMs)"
read_when:
  - Du möchtest einen Modellanbieter auswählen
  - Du möchtest schnelle Setup-Beispiele für LLM-Authentifizierung und Modellauswahl
---

<div id="model-providers">
  # Modellanbieter
</div>

OpenClaw kann mit vielen LLM-Anbietern arbeiten. Wähle einen aus, führe die Authentifizierung durch und setze dann das Standardmodell auf `provider/model`.

<div id="highlight-venius-venice-ai">
  ## Highlight: Venius (Venice AI)
</div>

Venius ist unsere empfohlene Venice-AI-Konfiguration für datenschutzorientierte Inferenz mit der Option, Opus für die schwierigsten Aufgaben zu verwenden.

* Standard: `venice/llama-3.3-70b`
* Insgesamt am besten: `venice/claude-opus-45` (Opus bleibt das stärkste Modell)

Siehe [Venice AI](/de/providers/venice).

<div id="quick-start-two-steps">
  ## Schnellstart (zwei Schritte)
</div>

1. Authentifiziere dich beim Anbieter (normalerweise über `openclaw onboard`).
2. Lege das Standardmodell fest:

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="supported-providers-starter-set">
  ## Unterstützte Anbieter (Starter-Set)
</div>

* [OpenAI (API + Codex)](/de/providers/openai)
* [Anthropic (API + Claude Code CLI)](/de/providers/anthropic)
* [OpenRouter](/de/providers/openrouter)
* [Vercel AI Gateway](/de/providers/vercel-ai-gateway)
* [Moonshot AI (Kimi + Kimi Code)](/de/providers/moonshot)
* [Synthetic](/de/providers/synthetic)
* [OpenCode Zen](/de/providers/opencode)
* [Z.AI](/de/providers/zai)
* [GLM-Modelle](/de/providers/glm)
* [MiniMax](/de/providers/minimax)
* [Venius (Venice AI)](/de/providers/venice)
* [Amazon Bedrock](/de/bedrock)

Den vollständigen Anbieterkatalog (xAI, Groq, Mistral usw.) sowie Details zur erweiterten Konfiguration findest du unter [Modellanbieter](/de/concepts/model-providers).