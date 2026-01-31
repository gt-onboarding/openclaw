---
title: Zai
summary: "Utiliser Z.AI (modèles GLM) avec OpenClaw"
read_when:
  - Vous voulez des modèles Z.AI / GLM dans OpenClaw
  - Vous avez besoin d'une configuration simple de ZAI_API_KEY
---

<div id="zai">
  # Z.AI
</div>

Z.AI est la plateforme d&#39;API pour les modèles **GLM**. Elle fournit des API REST pour GLM et utilise des clés d&#39;API
pour l&#39;authentification. Créez votre clé d&#39;API dans la console Z.AI. OpenClaw utilise le fournisseur `zai`
avec une clé d&#39;API Z.AI.

<div id="cli-setup">
  ## Configuration de la CLI
</div>

```bash
openclaw onboard --auth-choice zai-api-key
# ou en mode non-interactif
openclaw onboard --zai-api-key "$ZAI_API_KEY"
```

<div id="config-snippet">
  ## Exemple de configuration
</div>

```json5
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-4.7" } } }
}
```

<div id="notes">
  ## Notes
</div>

* Les modèles GLM sont disponibles sous la forme `zai/<model>` (exemple : `zai/glm-4.7`).
* Voir [/providers/glm](/fr/providers/glm) pour une vue d’ensemble de la famille de modèles.
* Z.AI utilise l’authentification Bearer avec votre clé API.