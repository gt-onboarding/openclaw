---
title: OpenRouter
summary: "Verwende die einheitliche API von OpenRouter, um in OpenClaw Zugriff auf viele Modelle zu erhalten"
read_when:
  - Du möchtest einen einzigen API-Schlüssel für viele LLMs
  - Du möchtest Modelle über OpenRouter in OpenClaw ausführen
---

<div id="openrouter">
  # OpenRouter
</div>

OpenRouter stellt eine **einheitliche API** bereit, die Anfragen an viele Modelle hinter einem einzigen
Endpunkt und API-Schlüssel weiterleitet. Sie ist OpenAI-kompatibel, daher funktionieren die meisten OpenAI-SDKs, indem lediglich die Basis-URL geändert wird.

<div id="cli-setup">
  ## CLI-Setup
</div>

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

<div id="config-snippet">
  ## Konfigurationsbeispiel
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
  ## Hinweise
</div>

* Modellreferenzen haben das Format `openrouter/<provider>/<model>`.
* Weitere Modell-/Anbieteroptionen findest du unter [/concepts/model-providers](/de/concepts/model-providers).
* OpenRouter verwendet intern ein Bearer-Token mit deinem API-Schlüssel.