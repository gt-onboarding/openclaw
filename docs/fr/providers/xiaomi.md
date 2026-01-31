---
title: Xiaomi
summary: "Utiliser Xiaomi MiMo (mimo-v2-flash) avec OpenClaw"
read_when:
  - Vous voulez utiliser des modèles Xiaomi MiMo dans OpenClaw
  - Vous devez configurer XIAOMI_API_KEY
---

<div id="xiaomi-mimo">
  # Xiaomi MiMo
</div>

Xiaomi MiMo est la plateforme d&#39;API pour les modèles **MiMo**. Elle fournit des API REST compatibles avec
les formats OpenAI et Anthropic et utilise des clés d&#39;API pour l&#39;authentification. Créez votre clé d&#39;API dans
la [console Xiaomi MiMo](https://platform.xiaomimimo.com/#/console/api-keys). OpenClaw utilise
le fournisseur `xiaomi` avec une clé d&#39;API Xiaomi MiMo.

<div id="model-overview">
  ## Aperçu du modèle
</div>

* **mimo-v2-flash** : fenêtre de contexte de 262 144 jetons, compatible avec l’API Messages d’Anthropic.
* URL de base : `https://api.xiaomimimo.com/anthropic`
* Autorisation : `Bearer $XIAOMI_API_KEY`

<div id="cli-setup">
  ## Configuration de la CLI
</div>

```bash
openclaw onboard --auth-choice xiaomi-api-key
# ou en mode non interactif
openclaw onboard --auth-choice xiaomi-api-key --xiaomi-api-key "$XIAOMI_API_KEY"
```

<div id="config-snippet">
  ## Exemple de configuration
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
  ## Notes
</div>

* Référence du modèle : `xiaomi/mimo-v2-flash`.
* Le fournisseur est injecté automatiquement lorsque `XIAOMI_API_KEY` est défini (ou qu’un profil d’authentification existe).
* Voir [/concepts/model-providers](/fr/concepts/model-providers) pour les règles applicables aux fournisseurs.