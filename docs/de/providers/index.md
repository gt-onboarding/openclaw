---
title: Anbieter
summary: "Von OpenClaw unterstützte Modellanbieter (LLMs)"
read_when:
  - Wenn du einen Modellanbieter auswählen möchtest
  - Wenn du einen schnellen Überblick über unterstützte LLM-Backends brauchst
---

<div id="model-providers">
  # Modellanbieter
</div>

OpenClaw kann viele LLM-Anbieter nutzen. Wähle einen Anbieter, authentifiziere dich und setze dann das
Standardmodell auf `provider/model`.

Suchst du nach Dokumentation zu Chat-Kanälen (WhatsApp/Telegram/Discord/Slack/Mattermost (Plugin)/etc.)? Siehe [Channels](/de/channels).

<div id="highlight-venius-venice-ai">
  ## Highlight: Venius (Venice AI)
</div>

Venius ist unsere empfohlene Venice-AI-Konfiguration für datenschutzfreundliche Inferenz, mit der Option, Opus für anspruchsvolle Aufgaben zu verwenden.

* Standard: `venice/llama-3.3-70b`
* Insgesamt am besten: `venice/claude-opus-45` (Opus bleibt das stärkste Modell)

Siehe [Venice AI](/de/providers/venice).

<div id="quick-start">
  ## Schnellstart
</div>

1. Melde dich beim Anbieter an (normalerweise über `openclaw onboard`).
2. Setze das Standardmodell:

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="provider-docs">
  ## Anbieter-Dokumentation
</div>

* [OpenAI (API + Codex)](/de/providers/openai)
* [Anthropic (API + Claude Code CLI)](/de/providers/anthropic)
* [Qwen (OAuth)](/de/providers/qwen)
* [OpenRouter](/de/providers/openrouter)
* [Vercel AI Gateway](/de/providers/vercel-ai-gateway)
* [Moonshot AI (Kimi + Kimi Code)](/de/providers/moonshot)
* [OpenCode Zen](/de/providers/opencode)
* [Amazon Bedrock](/de/bedrock)
* [Z.AI](/de/providers/zai)
* [Xiaomi](/de/providers/xiaomi)
* [GLM-Modelle](/de/providers/glm)
* [MiniMax](/de/providers/minimax)
* [Venius (Venice AI, datenschutzorientiert)](/de/providers/venice)
* [Ollama (lokale Modelle)](/de/providers/ollama)

<div id="transcription-providers">
  ## Transkriptionsanbieter
</div>

* [Deepgram (Audiotranskription)](/de/providers/deepgram)

<div id="community-tools">
  ## Community-Tools
</div>

* [Claude Max API Proxy](/de/providers/claude-max-api-proxy) - Verwende ein Claude Max-/Pro-Abonnement als OpenAI-kompatiblen API-Endpunkt

Den vollständigen Anbieterkatalog (xAI, Groq, Mistral usw.) sowie Informationen zur erweiterten Konfiguration findest du unter [Modellanbieter](/de/concepts/model-providers).