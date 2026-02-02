---
title: Proveedores de modelos
summary: "Descripción general de proveedores de modelos con ejemplos de configuración y flujos de CLI"
read_when:
  - Necesitas una referencia de configuración de modelos, proveedor por proveedor
  - Quieres ejemplos de configuración o comandos de incorporación en la CLI para proveedores de modelos
---

<div id="model-providers">
  # Proveedores de modelos
</div>

Esta página trata de **proveedores de modelos/LLM** (no de canales de chat como WhatsApp o Telegram).
Para las reglas de selección de modelos, consulta [/concepts/models](/es/concepts/models).

<div id="quick-rules">
  ## Reglas rápidas
</div>

* Las referencias de modelos usan `provider/model` (ejemplo: `opencode/claude-opus-4-5`).
* Si configuras `agents.defaults.models`, se convierte en la lista de permitidos.
* Comandos de ayuda de la CLI: `openclaw onboard`, `openclaw models list`, `openclaw models set <provider/model>`.

<div id="built-in-providers-pi-ai-catalog">
  ## Proveedores integrados (catálogo pi-ai)
</div>

OpenClaw viene con el catálogo pi‑ai. Estos proveedores **no** requieren ninguna configuración de `models.providers`; solo configura la autenticación y elige un modelo.

<div id="openai">
  ### OpenAI
</div>

* Proveedor: `openai`
* Autenticación: `OPENAI_API_KEY`
* Ejemplo de modelo: `openai/gpt-5.2`
* CLI: `openclaw onboard --auth-choice openai-api-key`

```json5
{
  agents: { defaults: { model: { primary: "openai/gpt-5.2" } } }
}
```

<div id="anthropic">
  ### Anthropic
</div>

* Proveedor: `anthropic`
* Autenticación: `ANTHROPIC_API_KEY` o `claude setup-token`
* Modelo de ejemplo: `anthropic/claude-opus-4-5`
* CLI: `openclaw onboard --auth-choice token` (pega el setup-token cuando se te solicite) o `openclaw models auth paste-token --provider anthropic`

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="openai-code-codex">
  ### OpenAI Code (Codex)
</div>

* Proveedor: `openai-codex`
* Autenticación: OAuth (ChatGPT)
* Modelo de ejemplo: `openai-codex/gpt-5.2`
* CLI: `openclaw onboard --auth-choice openai-codex` o `openclaw models auth login --provider openai-codex`

```json5
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.2" } } }
}
```

<div id="opencode-zen">
  ### OpenCode Zen
</div>

* Proveedor: `opencode`
* Autenticación: `OPENCODE_API_KEY` (o `OPENCODE_ZEN_API_KEY`)
* Modelo de ejemplo: `opencode/claude-opus-4-5`
* CLI: `openclaw onboard --auth-choice opencode-zen`

```json5
{
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-5" } } }
}
```

<div id="google-gemini-api-key">
  ### Google Gemini (clave de API)
</div>

* Proveedor: `google`
* Autenticación: `GEMINI_API_KEY`
* Modelo de ejemplo: `google/gemini-3-pro-preview`
* CLI: `openclaw onboard --auth-choice gemini-api-key`

<div id="google-vertex-antigravity-gemini-cli">
  ### Google Vertex / Antigravity / Gemini CLI
</div>

* Proveedores: `google-vertex`, `google-antigravity`, `google-gemini-cli`
* Auth: Vertex usa gcloud ADC; Antigravity/Gemini CLI usan sus respectivos flujos de autenticación
* Antigravity OAuth se distribuye como un complemento integrado (`google-antigravity-auth`, deshabilitado de forma predeterminada).
  * Habilitar: `openclaw plugins enable google-antigravity-auth`
  * Iniciar sesión: `openclaw models auth login --provider google-antigravity --set-default`
* Gemini CLI OAuth se distribuye como un complemento integrado (`google-gemini-cli-auth`, deshabilitado de forma predeterminada).
  * Habilitar: `openclaw plugins enable google-gemini-cli-auth`
  * Iniciar sesión: `openclaw models auth login --provider google-gemini-cli --set-default`
  * Nota: **no** debes pegar un client id ni un secret en `openclaw.json`. El flujo de inicio de sesión de la CLI almacena
    los tokens en perfiles de autenticación en el host del Gateway.

<div id="zai-glm">
  ### Z.AI (GLM)
</div>

* Proveedor: `zai`
* Autenticación: `ZAI_API_KEY`
* Modelo de ejemplo: `zai/glm-4.7`
* CLI: `openclaw onboard --auth-choice zai-api-key`
  * Alias: `z.ai/*` y `z-ai/*` se normalizan en `zai/*`

<div id="vercel-ai-gateway">
  ### Vercel AI Gateway
</div>

* Proveedor: `vercel-ai-gateway`
* Autenticación: `AI_GATEWAY_API_KEY`
* Modelo de ejemplo: `vercel-ai-gateway/anthropic/claude-opus-4.5`
* CLI: `openclaw onboard --auth-choice ai-gateway-api-key`

<div id="other-built-in-providers">
  ### Otros proveedores integrados
</div>

* OpenRouter: `openrouter` (`OPENROUTER_API_KEY`)
* Modelo de ejemplo: `openrouter/anthropic/claude-sonnet-4-5`
* xAI: `xai` (`XAI_API_KEY`)
* Groq: `groq` (`GROQ_API_KEY`)
* Cerebras: `cerebras` (`CEREBRAS_API_KEY`)
  * Los modelos GLM en Cerebras utilizan los ID `zai-glm-4.7` y `zai-glm-4.6`.
  * URL base compatible con OpenAI: `https://api.cerebras.ai/v1`.
* Mistral: `mistral` (`MISTRAL_API_KEY`)
* GitHub Copilot: `github-copilot` (`COPILOT_GITHUB_TOKEN` / `GH_TOKEN` / `GITHUB_TOKEN`)

<div id="providers-via-modelsproviders-custombase-url">
  ## Proveedores a través de `models.providers` (URL personalizada/base)
</div>

Usa `models.providers` (o `models.json`) para añadir proveedores **personalizados** o
proxies compatibles con OpenAI/Anthropic.

<div id="moonshot-ai-kimi">
  ### Moonshot AI (Kimi)
</div>

Moonshot utiliza endpoints compatibles con OpenAI, por lo que debes configurarlo como un proveedor personalizado:

* Proveedor: `moonshot`
* Autenticación: `MOONSHOT_API_KEY`
* Modelo de ejemplo: `moonshot/kimi-k2.5`
* IDs de modelo de Kimi K2:
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

Kimi Code usa un endpoint y una clave dedicados (independientes de Moonshot):

* Proveedor: `kimi-code`
* Autenticación: `KIMICODE_API_KEY`
* Modelo de ejemplo: `kimi-code/kimi-for-coding`

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
  ### Qwen OAuth (plan gratuito)
</div>

Qwen proporciona acceso OAuth a Qwen Coder + Vision a través de un flujo de código de dispositivo.
Activa el complemento integrado y luego inicia sesión:

```bash
openclaw plugins enable qwen-portal-auth
openclaw models auth login --provider qwen-portal --set-default
```

Referencias de modelos:

* `qwen-portal/coder-model`
* `qwen-portal/vision-model`

Consulta [/providers/qwen](/es/providers/qwen) para ver detalles de configuración y notas.

<div id="synthetic">
  ### Synthetic
</div>

Synthetic proporciona modelos compatibles con Anthropic a través del proveedor `synthetic`:

* Proveedor: `synthetic`
* Autenticación: `SYNTHETIC_API_KEY`
* Modelo de ejemplo: `synthetic/hf:MiniMaxAI/MiniMax-M2.1`
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

MiniMax se configura a través de `models.providers` porque utiliza endpoints personalizados:

* MiniMax (compatible con Anthropic): `--auth-choice minimax-api`
* Autenticación: `MINIMAX_API_KEY`

Consulta [/providers/minimax](/es/providers/minimax) para obtener detalles de configuración, opciones de modelo y ejemplos de configuración.

<div id="ollama">
  ### Ollama
</div>

Ollama es un entorno de ejecución local de LLM que proporciona una API compatible con OpenAI:

* Proveedor: `ollama`
* Autenticación: No se requiere (servidor local)
* Modelo de ejemplo: `ollama/llama3.3`
* Instalación: https://ollama.ai

```bash
# Instala Ollama, luego descarga un modelo:
ollama pull llama3.3
```

```json5
{
  agents: {
    defaults: { model: { primary: "ollama/llama3.3" } }
  }
}
```

Ollama se detecta automáticamente si se está ejecutando localmente en `http://127.0.0.1:11434/v1`. Consulta [/providers/ollama](/es/providers/ollama) para obtener recomendaciones de modelos y configuración personalizada.

<div id="local-proxies-lm-studio-vllm-litellm-etc">
  ### Proxies locales (LM Studio, vLLM, LiteLLM, etc.)
</div>

Ejemplo (compatible con OpenAI):

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

Notas:

* Para proveedores personalizados, `reasoning`, `input`, `cost`, `contextWindow` y `maxTokens` son opcionales.
  Si se omiten, OpenClaw utiliza los siguientes valores predeterminados:
  * `reasoning: false`
  * `input: ["text"]`
  * `cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 }`
  * `contextWindow: 200000`
  * `maxTokens: 8192`
* Recomendado: establece valores explícitos que coincidan con los límites de tu proxy y modelo.

<div id="cli-examples">
  ## Ejemplos de la CLI
</div>

```bash
openclaw onboard --auth-choice opencode-zen
openclaw models set opencode/claude-opus-4-5
openclaw models list
```

Consulta también: [/gateway/configuration](/es/gateway/configuration) para ver ejemplos completos de configuración.
