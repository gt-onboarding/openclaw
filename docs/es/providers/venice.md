---
title: Venice
summary: "Usa los modelos de Venice AI centrados en la privacidad en OpenClaw"
read_when:
  - Quieres realizar inferencias centradas en la privacidad en OpenClaw
  - Quieres una guía para configurar Venice AI
---

<div id="venice-ai-venice-highlight">
  # Venice AI (configuración destacada de Venice)
</div>

**Venice** es nuestra configuración destacada de Venice para inferencia centrada en la privacidad, con acceso opcional y anonimizado a modelos propietarios.

Venice AI proporciona inferencia de IA enfocada en la privacidad, con soporte para modelos sin censurar y acceso a los principales modelos propietarios a través de su proxy anonimizado. Toda la inferencia es privada de forma predeterminada: no se entrena con tus datos ni se registran logs.

<div id="why-venice-in-openclaw">
  ## Por qué usar Venice en OpenClaw
</div>

- **Inferencia privada** para modelos de código abierto (sin registro de datos).
- **Modelos sin censura** cuando los necesites.
- **Acceso anonimizado** a modelos propietarios (Opus/GPT/Gemini) cuando la calidad importa.
- Endpoints `/v1` compatibles con OpenAI.

<div id="privacy-modes">
  ## Modos de privacidad
</div>

Venice ofrece dos niveles de privacidad; entenderlos es clave para elegir tu modelo:

| Modo | Descripción | Modelos |
|------|-------------|--------|
| **Privado** | Totalmente privado. Los prompts y respuestas **nunca se almacenan ni se registran**; son efímeros. | Llama, Qwen, DeepSeek, Venice Uncensored, etc. |
| **Anonimizado** | Se canaliza a través de Venice con los metadatos eliminados. El proveedor subyacente (OpenAI, Anthropic) solo ve peticiones anonimizadas. | Claude, GPT, Gemini, Grok, Kimi, MiniMax |

<div id="features">
  ## Características
</div>

- **Centrado en la privacidad**: Elige entre "private" (totalmente privado) y "anonymized" (con proxy) como modos de funcionamiento
- **Modelos sin censura**: Acceso a modelos sin restricciones de contenido
- **Acceso a los principales modelos**: Usa Claude, GPT-5.2, Gemini y Grok a través del proxy anonimizado de Venice
- **API compatible con OpenAI**: Endpoints estándar `/v1` para una integración sencilla
- **Streaming**: ✅ Compatible con todos los modelos
- **Llamadas a funciones**: ✅ Compatible con modelos seleccionados (consulta las capacidades del modelo)
- **Visión**: ✅ Compatible con modelos con capacidad de visión
- **Sin límites de tasa estrictos**: Puede aplicarse limitación por uso justo en casos de uso extremo

<div id="setup">
  ## Configuración
</div>

<div id="1-get-api-key">
  ### 1. Obtener tu clave de API
</div>

1. Regístrate en [venice.ai](https://venice.ai)
2. Ve a **Settings → API Keys → Create new key**
3. Copia tu clave de API (formato: `vapi_xxxxxxxxxxxx`)

<div id="2-configure-openclaw">
  ### 2. Configurar OpenClaw
</div>

**Opción A: Variable de entorno**

```bash
export VENICE_API_KEY="vapi_xxxxxxxxxxxx"
```

**Opción B: Configuración interactiva (recomendada)**

```bash
openclaw onboard --auth-choice venice-api-key
```

Esto hará lo siguiente:

1. Te pedirá tu clave de API (o usará la `VENICE_API_KEY` existente)
2. Mostrará todos los modelos de Venice disponibles
3. Te permitirá elegir tu modelo predeterminado
4. Configurará el proveedor automáticamente

**Opción C: No interactiva**

```bash
openclaw onboard --non-interactive \
  --auth-choice venice-api-key \
  --venice-api-key "vapi_xxxxxxxxxxxx"
```


<div id="3-verify-setup">
  ### 3. Verifica la configuración
</div>

```bash
openclaw chat --model venice/llama-3.3-70b "Hello, are you working?"
```


<div id="model-selection">
  ## Selección de modelo
</div>

Tras la configuración inicial, OpenClaw muestra todos los modelos de Venice disponibles. Elige según tus necesidades:

* **Predeterminado (nuestra elección)**: `venice/llama-3.3-70b` para un rendimiento privado y equilibrado.
* **Mejor calidad general**: `venice/claude-opus-45` para tareas difíciles (Opus sigue siendo el más potente).
* **Privacidad**: Elige modelos &quot;private&quot; para inferencias completamente privadas.
* **Capacidad**: Elige modelos &quot;anonymized&quot; para acceder a Claude, GPT y Gemini a través del proxy de Venice.

Puedes cambiar tu modelo predeterminado en cualquier momento:

```bash
openclaw models set venice/claude-opus-45
openclaw models set venice/llama-3.3-70b
```

Listar todos los modelos disponibles:

```bash
openclaw models list | grep venice
```


<div id="configure-via-openclaw-configure">
  ## Configura con `openclaw configure`
</div>

1. Ejecuta `openclaw configure`
2. Selecciona **Model/auth**
3. Elige **Venice AI**

<div id="which-model-should-i-use">
  ## ¿Qué modelo debería usar?
</div>

| Caso de uso | Modelo recomendado | Por qué |
|----------|-------------------|-----|
| **Chat general** | `llama-3.3-70b` | Buen modelo general, totalmente privado |
| **Mejor calidad general** | `claude-opus-45` | Opus sigue siendo el más sólido para tareas difíciles |
| **Privacidad + calidad de Claude** | `claude-opus-45` | Mejor razonamiento mediante un proxy anonimizado |
| **Programación** | `qwen3-coder-480b-a35b-instruct` | Optimizado para código, contexto de 262k |
| **Tareas de visión** | `qwen3-vl-235b-a22b` | Mejor modelo de visión privado |
| **Sin censura** | `venice-uncensored` | Sin restricciones de contenido |
| **Rápido y barato** | `qwen3-4b` | Ligero, pero aún capaz |
| **Razonamiento complejo** | `deepseek-v3.2` | Gran capacidad de razonamiento, privado |

<div id="available-models-25-total">
  ## Modelos disponibles (25 en total)
</div>

<div id="private-models-15-fully-private-no-logging">
  ### Modelos privados (15) — Totalmente privados, sin registro de actividad
</div>

| ID de modelo | Nombre | Contexto (tokens) | Características |
|----------|------|------------------|----------|
| `llama-3.3-70b` | Llama 3.3 70B | 131k | General |
| `llama-3.2-3b` | Llama 3.2 3B | 131k | Rápido, ligero |
| `hermes-3-llama-3.1-405b` | Hermes 3 Llama 3.1 405B | 131k | Tareas complejas |
| `qwen3-235b-a22b-thinking-2507` | Qwen3 235B Thinking | 131k | Razonamiento |
| `qwen3-235b-a22b-instruct-2507` | Qwen3 235B Instruct | 131k | General |
| `qwen3-coder-480b-a35b-instruct` | Qwen3 Coder 480B | 262k | Código |
| `qwen3-next-80b` | Qwen3 Next 80B | 262k | General |
| `qwen3-vl-235b-a22b` | Qwen3 VL 235B | 262k | Visión |
| `qwen3-4b` | Venice Small (Qwen3 4B) | 32k | Rápido, para razonamiento |
| `deepseek-v3.2` | DeepSeek V3.2 | 163k | Razonamiento |
| `venice-uncensored` | Venice Uncensored | 32k | Sin censura |
| `mistral-31-24b` | Venice Medium (Mistral) | 131k | Visión |
| `google-gemma-3-27b-it` | Gemma 3 27B Instruct | 202k | Visión |
| `openai-gpt-oss-120b` | OpenAI GPT OSS 120B | 131k | General |
| `zai-org-glm-4.7` | GLM 4.7 | 202k | Razonamiento, multilingüe |

<div id="anonymized-models-10-via-venice-proxy">
  ### Modelos anonimizados (10) — mediante Venice Proxy
</div>

| Model ID | Original | Contexto (tokens) | Capacidades |
|----------|----------|-------------------|-------------|
| `claude-opus-45` | Claude Opus 4.5 | 202k | Razonamiento, visión |
| `claude-sonnet-45` | Claude Sonnet 4.5 | 202k | Razonamiento, visión |
| `openai-gpt-52` | GPT-5.2 | 262k | Razonamiento |
| `openai-gpt-52-codex` | GPT-5.2 Codex | 262k | Razonamiento, visión |
| `gemini-3-pro-preview` | Gemini 3 Pro | 202k | Razonamiento, visión |
| `gemini-3-flash-preview` | Gemini 3 Flash | 262k | Razonamiento, visión |
| `grok-41-fast` | Grok 4.1 Fast | 262k | Razonamiento, visión |
| `grok-code-fast-1` | Grok Code Fast 1 | 262k | Razonamiento, código |
| `kimi-k2-thinking` | Kimi K2 Thinking | 262k | Razonamiento |
| `minimax-m21` | MiniMax M2.1 | 202k | Razonamiento |

<div id="model-discovery">
  ## Descubrimiento de modelos
</div>

OpenClaw descubre automáticamente modelos a partir de la API de Venice cuando `VENICE_API_KEY` está definida. Si la API no es accesible, recurre a un catálogo estático.

El endpoint `/models` es público (no requiere autenticación para listar modelos), pero la inferencia requiere una clave de API válida.

<div id="streaming-tool-support">
  ## Streaming y compatibilidad con herramientas
</div>

| Característica | Compatibilidad |
|----------------|----------------|
| **Streaming** | ✅ Todos los modelos |
| **Function calling** | ✅ La mayoría de los modelos (consulta `supportsFunctionCalling` en la API) |
| **Visión/Imágenes** | ✅ Modelos marcados con la característica “Vision” |
| **Modo JSON** | ✅ Compatible mediante `response_format` |

<div id="pricing">
  ## Precios
</div>

Venice usa un sistema basado en créditos. Consulta [venice.ai/pricing](https://venice.ai/pricing) para ver las tarifas actuales:

- **Modelos privados**: Por lo general más económicos
- **Modelos anonimizados**: Similares al precio de la api directa más una pequeña comisión de Venice

<div id="comparison-venice-vs-direct-api">
  ## Comparación: Venice vs API directa
</div>

| Aspecto | Venice (anonimizado) | API directa |
|--------|---------------------|------------|
| **Privacidad** | Metadatos eliminados y anonimizados | Tu cuenta asociada |
| **Latencia** | +10-50 ms (proxy) | Directa |
| **Funciones** | La mayoría de las funciones admitidas | Todas las funciones |
| **Facturación** | Créditos de Venice | Facturación del proveedor |

<div id="usage-examples">
  ## Ejemplos de uso
</div>

```bash
# Use default private model
openclaw chat --model venice/llama-3.3-70b

# Usar Claude a través de Venice (anonimizado)
openclaw chat --model venice/claude-opus-45

# Use uncensored model
openclaw chat --model venice/venice-uncensored

# Use vision model with image
openclaw chat --model venice/qwen3-vl-235b-a22b

# Use coding model
openclaw chat --model venice/qwen3-coder-480b-a35b-instruct
```


<div id="troubleshooting">
  ## Solución de problemas
</div>

<div id="api-key-not-recognized">
  ### Clave de API no reconocida
</div>

```bash
echo $VENICE_API_KEY
openclaw models list | grep venice
```

Asegúrate de que la clave empiece por `vapi_`.


<div id="model-not-available">
  ### Modelo no disponible
</div>

El catálogo de modelos de Venice se actualiza de forma dinámica. Ejecuta `openclaw models list` para ver los modelos disponibles actualmente. Algunos modelos pueden estar temporalmente no disponibles.

<div id="connection-issues">
  ### Problemas de conexión
</div>

La API de Venice se encuentra en `https://api.venice.ai/api/v1`. Asegúrate de que tu red permita conexiones HTTPS.

<div id="config-file-example">
  ## Ejemplo de archivo de configuración
</div>

```json5
{
  env: { VENICE_API_KEY: "vapi_..." },
  agents: { defaults: { model: { primary: "venice/llama-3.3-70b" } } },
  models: {
    mode: "merge",
    providers: {
      venice: {
        baseUrl: "https://api.venice.ai/api/v1",
        apiKey: "${VENICE_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "llama-3.3-70b",
            name: "Llama 3.3 70B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 131072,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```


<div id="links">
  ## Enlaces
</div>

- [Venice AI](https://venice.ai)
- [Documentación de la API](https://docs.venice.ai)
- [Precios](https://venice.ai/pricing)
- [Estado](https://status.venice.ai)