---
title: Zai
summary: "Usa Z.AI (modelli GLM) con OpenClaw"
read_when:
  - Vuoi utilizzare i modelli Z.AI / GLM in OpenClaw
  - Hai bisogno di una configurazione semplice della variabile ZAI_API_KEY
---

<div id="zai">
  # Z.AI
</div>

Z.AI è la piattaforma API per i modelli **GLM**. Fornisce API REST per GLM e utilizza chiavi API
per l&#39;autenticazione. Crea una chiave API nella console Z.AI. OpenClaw utilizza il provider `zai`
con una chiave API Z.AI.

<div id="cli-setup">
  ## Configurazione della CLI
</div>

```bash
openclaw onboard --auth-choice zai-api-key
# oppure in modalità non interattiva
openclaw onboard --zai-api-key "$ZAI_API_KEY"
```

<div id="config-snippet">
  ## Esempio di configurazione
</div>

```json5
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-4.7" } } }
}
```

<div id="notes">
  ## Note
</div>

* I modelli GLM sono disponibili come `zai/<model>` (ad esempio: `zai/glm-4.7`).
* Consulta [/providers/glm](/it/providers/glm) per una panoramica della famiglia di modelli.
* Z.AI usa l&#39;autenticazione Bearer con la tua chiave API.