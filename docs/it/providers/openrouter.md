---
title: Openrouter
summary: "Usa l'API unificata di OpenRouter per accedere a numerosi modelli in OpenClaw"
read_when:
  - Vuoi un'unica chiave API per molti LLM
  - Vuoi eseguire modelli tramite OpenRouter in OpenClaw
---

<div id="openrouter">
  # OpenRouter
</div>

OpenRouter fornisce una **API unificata** che instrada le richieste verso molti modelli tramite un unico
endpoint e una chiave API. È compatibile con OpenAI, quindi la maggior parte degli SDK OpenAI funziona semplicemente modificando la URL di base.

<div id="cli-setup">
  ## Configurazione della CLI
</div>

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

<div id="config-snippet">
  ## Esempio di configurazione
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
  ## Note
</div>

* I riferimenti ai modelli sono `openrouter/<provider>/<model>`.
* Per ulteriori opzioni di modelli/provider, consulta [/concepts/model-providers](/it/concepts/model-providers).
* OpenRouter utilizza un token Bearer con la tua chiave API a livello interno.