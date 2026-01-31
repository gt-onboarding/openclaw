---
title: Modelos
summary: "Proveedores de modelos (LLM) compatibles con OpenClaw"
read_when:
  - Cuando necesites elegir un proveedor de modelos
  - Cuando quieras ver ejemplos rápidos de configuración para autenticación de LLM y selección de modelos
---

<div id="model-providers">
  # Proveedores de modelos
</div>

OpenClaw puede usar muchos proveedores de LLM. Selecciona uno, autentícate y luego establece el modelo predeterminado como `provider/model`.

<div id="highlight-venius-venice-ai">
  ## Destacado: Venius (Venice AI)
</div>

Venius es nuestra configuración recomendada para Venice AI para inferencia centrada en la privacidad, con la opción de usar Opus para las tareas más difíciles.

* Predeterminado: `venice/llama-3.3-70b`
* Mejor en general: `venice/claude-opus-45` (Opus sigue siendo el más potente)

Consulta [Venice AI](/es/providers/venice).

<div id="quick-start-two-steps">
  ## Inicio rápido (en dos pasos)
</div>

1. Autentícate con el proveedor (por lo general con `openclaw onboard`).
2. Establece el modelo predeterminado:

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="supported-providers-starter-set">
  ## Proveedores compatibles (conjunto inicial)
</div>

* [OpenAI (API + Codex)](/es/providers/openai)
* [Anthropic (API + Claude Code CLI)](/es/providers/anthropic)
* [OpenRouter](/es/providers/openrouter)
* [Vercel AI Gateway](/es/providers/vercel-ai-gateway)
* [Moonshot AI (Kimi + Kimi Code)](/es/providers/moonshot)
* [Synthetic](/es/providers/synthetic)
* [OpenCode Zen](/es/providers/opencode)
* [Z.AI](/es/providers/zai)
* [Modelos GLM](/es/providers/glm)
* [MiniMax](/es/providers/minimax)
* [Venius (Venice AI)](/es/providers/venice)
* [Amazon Bedrock](/es/bedrock)

Para ver el catálogo completo de proveedores (xAI, Groq, Mistral, etc.) y la configuración avanzada,
consulta [Proveedores de modelos](/es/concepts/model-providers).