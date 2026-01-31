---
title: Modelos locales
summary: "Ejecuta OpenClaw con modelos LLM locales (LM Studio, vLLM, LiteLLM, endpoints personalizados de OpenAI)"
read_when:
  - Quieres servir modelos desde tu propia máquina con GPU
  - Estás integrando LM Studio o un proxy compatible con OpenAI
  - Necesitas la orientación más segura sobre modelos locales
---

<div id="local-models">
  # Modelos locales
</div>

Lo local es viable, pero OpenClaw espera un contexto amplio y defensas sólidas contra la inyección de prompts. Las tarjetas pequeñas recortan el contexto y debilitan la seguridad. Apunta alto: **≥2 Mac Studio totalmente equipados o un equipo de GPU equivalente (~30 000 USD o más)**. Una sola GPU de **24 GB** solo sirve para prompts más ligeros, con mayor latencia. Usa la **variante de modelo más grande / de tamaño completo que puedas ejecutar**; los checkpoints agresivamente cuantizados o “pequeños” aumentan el riesgo de inyección de prompts (consulta [Security](/es/gateway/security)).

<div id="recommended-lm-studio-minimax-m21-responses-api-full-size">
  ## Recomendado: LM Studio + MiniMax M2.1 (Responses API, tamaño completo)
</div>

Mejor stack local disponible actualmente. Carga MiniMax M2.1 en LM Studio, habilita el servidor local (por defecto `http://127.0.0.1:1234`) y usa Responses API para mantener separado el razonamiento del texto final.

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" }
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

**Lista de verificación de configuración**

* Instala LM Studio: https://lmstudio.ai
* En LM Studio, descarga la **versión más grande de MiniMax M2.1 disponible** (evita variantes “small”/fuertemente cuantizadas), inicia el servidor y confirma que `http://127.0.0.1:1234/v1/models` lo liste.
* Mantén el modelo cargado; una carga en frío añade latencia de arranque.
* Ajusta `contextWindow`/`maxTokens` si tu versión de LM Studio es diferente.
* Para WhatsApp, usa la Responses API para que solo se envíe el texto final.

Mantén los modelos hospedados configurados incluso cuando ejecutes en local; usa `models.mode: "merge"` para que los fallbacks sigan disponibles.

<div id="hybrid-config-hosted-primary-local-fallback">
  ### Configuración híbrida: principal en la nube, respaldo local
</div>

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["lmstudio/minimax-m2.1-gs32", "anthropic/claude-opus-4-5"]
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "lmstudio/minimax-m2.1-gs32": { alias: "MiniMax Local" },
        "anthropic/claude-opus-4-5": { alias: "Opus" }
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

<div id="local-first-with-hosted-safety-net">
  ### Prioridad local con red de seguridad en la nube
</div>

Intercambia el orden entre principal y respaldo; mantén el mismo bloque de proveedores y `models.mode: "merge"` para poder recurrir a Sonnet u Opus cuando la máquina local esté caída.

<div id="regional-hosting-data-routing">
  ### Alojamiento regional / enrutamiento de datos
</div>

* También existen variantes alojadas de MiniMax/Kimi/GLM en OpenRouter con endpoints con región fija (por ejemplo, alojados en EE. UU.). Elige allí la variante regional para mantener el tráfico en la jurisdicción que prefieras mientras sigues usando `models.mode: "merge"` para los fallbacks de Anthropic/OpenAI.
* El modo solo local sigue siendo la opción más sólida en términos de privacidad; el enrutamiento regional alojado es el punto intermedio cuando necesitas características del proveedor pero quieres controlar el flujo de datos.

<div id="other-openai-compatible-local-proxies">
  ## Otros proxies locales compatibles con OpenAI
</div>

vLLM, LiteLLM, OAI-proxy o gateways personalizados funcionan si exponen un endpoint `/v1` al estilo de OpenAI. Reemplaza el bloque de proveedor anterior por tu endpoint e ID de modelo:

```json5
{
  models: {
    mode: "merge",
    providers: {
      local: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "sk-local",
        api: "openai-responses",
        models: [
          {
            id: "my-local-model",
            name: "Local Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 120000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

Mantén `models.mode: "merge"` para que los modelos hospedados sigan disponibles como opción de respaldo.

<div id="troubleshooting">
  ## Solución de problemas
</div>

* ¿Puede Gateway llegar al proxy? `curl http://127.0.0.1:1234/v1/models`.
* ¿Modelo de LM Studio descargado de memoria? Vuelve a cargarlo; un arranque en frío es una causa frecuente de cuelgues.
* ¿Errores de contexto? Reduce `contextWindow` o aumenta el límite de tu servidor.
* Seguridad: los modelos locales no pasan por los filtros del lado del proveedor; mantén los agentes acotados y la compactación activada para limitar el radio de explosión de la inyección de prompts.