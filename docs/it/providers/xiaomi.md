---
title: Xiaomi
summary: "Usa Xiaomi MiMo (mimo-v2-flash) con OpenClaw"
read_when:
  - Vuoi usare i modelli Xiaomi MiMo in OpenClaw
  - Devi configurare XIAOMI_API_KEY
---

<div id="xiaomi-mimo">
  # Xiaomi MiMo
</div>

Xiaomi MiMo è la piattaforma API per i modelli **MiMo**. Fornisce API REST compatibili con
i formati OpenAI e Anthropic e utilizza chiavi API per l&#39;autenticazione. Crea la tua chiave API nella
[Xiaomi MiMo console](https://platform.xiaomimimo.com/#/console/api-keys). OpenClaw utilizza
il provider `xiaomi` con una chiave API Xiaomi MiMo.

<div id="model-overview">
  ## Panoramica del modello
</div>

* **mimo-v2-flash**: finestra di contesto di 262144 token, compatibile con l’API Anthropic Messages.
* URL di base: `https://api.xiaomimimo.com/anthropic`
* Autorizzazione: `Bearer $XIAOMI_API_KEY`

<div id="cli-setup">
  ## Configurazione della CLI
</div>

```bash
openclaw onboard --auth-choice xiaomi-api-key
# oppure in modalità non interattiva
openclaw onboard --auth-choice xiaomi-api-key --xiaomi-api-key "$XIAOMI_API_KEY"
```

<div id="config-snippet">
  ## Esempio di configurazione
</div>

```json5
{
  env: { XIAOMI_API_KEY: "your-key" },
  agents: { defaults: { model: { primary: "xiaomi/mimo-v2-flash" } } },
  models: {
    mode: "merge",
    providers: {
      xiaomi: {
        baseUrl: "https://api.xiaomimimo.com/anthropic",
        api: "anthropic-messages",
        apiKey: "XIAOMI_API_KEY",
        models: [
          {
            id: "mimo-v2-flash",
            name: "Xiaomi MiMo V2 Flash",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 262144,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

<div id="notes">
  ## Note
</div>

* Riferimento al modello: `xiaomi/mimo-v2-flash`.
* Il provider viene configurato automaticamente quando `XIAOMI_API_KEY` è impostata (o esiste un profilo di autenticazione).
* Consulta [/concepts/model-providers](/it/concepts/model-providers) per le regole sui provider.