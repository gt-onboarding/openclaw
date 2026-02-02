---
title: Proveedores
summary: "Proveedores de modelos (LLMs) compatibles con OpenClaw"
read_when:
  - Quieres elegir un proveedor de modelos
  - Necesitas un resumen rápido de los backends de LLM admitidos
---

<div id="model-providers">
  # Proveedores de modelos
</div>

OpenClaw puede usar muchos proveedores de LLM. Selecciona un proveedor, autentícate y luego establece el
modelo predeterminado como `provider/model`.

¿Buscas la documentación de los canales de chat (WhatsApp/Telegram/Discord/Slack/Mattermost (complemento)/etc.)? Consulta [Canales](/es/channels).

<div id="highlight-venius-venice-ai">
  ## Destacado: Venius (Venice AI)
</div>

Venius es nuestra configuración recomendada de Venice AI para inferencia centrada en la privacidad, con la opción de usar Opus para tareas difíciles.

* Predeterminado: `venice/llama-3.3-70b`
* Mejor en general: `venice/claude-opus-45` (Opus sigue siendo el más potente)

Consulta [Venice AI](/es/providers/venice).

<div id="quick-start">
  ## Inicio rápido
</div>

1. Autentícate con el proveedor (normalmente mediante `openclaw onboard`).
2. Configura el modelo predeterminado:

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="provider-docs">
  ## Documentación de proveedores
</div>

* [OpenAI (API + Codex)](/es/providers/openai)
* [Anthropic (API + Claude Code CLI)](/es/providers/anthropic)
* [Qwen (OAuth)](/es/providers/qwen)
* [OpenRouter](/es/providers/openrouter)
* [Vercel AI Gateway](/es/providers/vercel-ai-gateway)
* [Moonshot AI (Kimi + Kimi Code)](/es/providers/moonshot)
* [OpenCode Zen](/es/providers/opencode)
* [Amazon Bedrock](/es/bedrock)
* [Z.AI](/es/providers/zai)
* [Xiaomi](/es/providers/xiaomi)
* [Modelos GLM](/es/providers/glm)
* [MiniMax](/es/providers/minimax)
* [Venius (Venice AI, enfocado en la privacidad)](/es/providers/venice)
* [Ollama (modelos locales)](/es/providers/ollama)

<div id="transcription-providers">
  ## Proveedores de transcripción
</div>

* [Deepgram (transcripción de audio)](/es/providers/deepgram)

<div id="community-tools">
  ## Herramientas de la comunidad
</div>

* [Claude Max API Proxy](/es/providers/claude-max-api-proxy) - Usa tu suscripción Claude Max/Pro como endpoint de API compatible con OpenAI

Para consultar el catálogo completo de proveedores (xAI, Groq, Mistral, etc.) y la configuración avanzada,
revisa [Proveedores de modelos](/es/concepts/model-providers).