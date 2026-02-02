---
title: Xiaomi
summary: "Verwende Xiaomi MiMo (mimo-v2-flash) mit OpenClaw"
read_when:
  - Du möchtest Xiaomi MiMo-Modelle in OpenClaw verwenden
  - Du musst XIAOMI_API_KEY einrichten
---

<div id="xiaomi-mimo">
  # Xiaomi MiMo
</div>

Xiaomi MiMo ist die API‑Plattform für **MiMo**‑Modelle. Sie stellt REST‑APIs bereit, die mit
den Formaten von OpenAI und Anthropic kompatibel sind, und verwendet API‑Schlüssel für die Authentifizierung. Erstelle deinen API‑Schlüssel in der
[Xiaomi MiMo‑Konsole](https://platform.xiaomimimo.com/#/console/api-keys). OpenClaw verwendet
den `xiaomi` Anbieter mit einem Xiaomi‑MiMo‑API‑Schlüssel.

<div id="model-overview">
  ## Modellübersicht
</div>

* **mimo-v2-flash**: Kontextfenster von 262144 Tokens, kompatibel mit der Anthropic Messages API.
* Basis-URL: `https://api.xiaomimimo.com/anthropic`
* Autorisierung: `Bearer $XIAOMI_API_KEY`

<div id="cli-setup">
  ## CLI-Setup
</div>

```bash
openclaw onboard --auth-choice xiaomi-api-key
# oder nicht-interaktiv
openclaw onboard --auth-choice xiaomi-api-key --xiaomi-api-key "$XIAOMI_API_KEY"
```

<div id="config-snippet">
  ## Konfigurationsbeispiel
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
  ## Hinweise
</div>

* Modell-Ref: `xiaomi/mimo-v2-flash`.
* Der anbieter wird automatisch verwendet, wenn `XIAOMI_API_KEY` gesetzt ist (oder ein Auth-Profil vorhanden ist).
* Siehe [/concepts/model-providers](/de/concepts/model-providers) für Regeln zu anbietern.