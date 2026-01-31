---
title: Openrouter
summary: "Utiliza la API unificada de OpenRouter para acceder a muchos modelos en OpenClaw"
read_when:
  - Quieres una única clave de API para muchos LLM
  - Quieres ejecutar modelos a través de OpenRouter en OpenClaw
---

<div id="openrouter">
  # OpenRouter
</div>

OpenRouter proporciona una **API unificada** que dirige peticiones a muchos modelos a través de un único
endpoint y clave de API. Es compatible con OpenAI, por lo que la mayoría de los SDK de OpenAI funcionan simplemente cambiando la URL base.

<div id="cli-setup">
  ## Configuración de la CLI
</div>

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

<div id="config-snippet">
  ## Ejemplo de configuración
</div>

```json5
{
  env: { OPENROUTER_API_KEY: "sk-or-..." },
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" }
    }
  }
}
```

<div id="notes">
  ## Notas
</div>

* Las referencias a modelos son `openrouter/<provider>/<model>`.
* Para más opciones de modelos y proveedores, consulta [/concepts/model-providers](/es/concepts/model-providers).
* OpenRouter utiliza un token Bearer con tu clave de API internamente.