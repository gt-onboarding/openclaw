---
title: Glm
summary: "Présentation de la famille de modèles GLM et de leur utilisation dans OpenClaw"
read_when:
  - Vous souhaitez utiliser des modèles GLM avec OpenClaw
  - Vous avez besoin de connaître la convention de nommage et la configuration des modèles
---

<div id="glm-models">
  # Modèles GLM
</div>

GLM est une **famille de modèles** (et non une entreprise) disponible via la plateforme Z.AI. Dans OpenClaw, les modèles GLM sont accessibles via le fournisseur `zai` et des identifiants de modèle comme `zai/glm-4.7`.

<div id="cli-setup">
  ## Configuration de la CLI
</div>

```bash
openclaw onboard --auth-choice zai-api-key
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
  ## Remarques
</div>

* Les versions et la disponibilité de GLM peuvent évoluer ; consultez la documentation de Z.AI pour obtenir les informations les plus récentes.
* Exemples d’ID de modèles : `glm-4.7` et `glm-4.6`.
* Pour plus de détails sur le fournisseur, consultez [/providers/zai](/fr/providers/zai).