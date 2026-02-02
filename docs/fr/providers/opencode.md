---
title: Opencode
summary: "Utiliser OpenCode Zen (modèles sélectionnés) avec OpenClaw"
read_when:
  - Vous souhaitez utiliser OpenCode Zen pour l’accès aux modèles
  - Vous souhaitez une liste sélectionnée de modèles adaptés au développement
---

<div id="opencode-zen">
  # OpenCode Zen
</div>

OpenCode Zen est une **liste de modèles soigneusement sélectionnés** recommandés par l'équipe OpenCode pour les agents de code.
C'est une voie d'accès hébergée, facultative, aux modèles, qui utilise une clé API et le fournisseur `opencode`.
Zen est actuellement en phase bêta.

<div id="cli-setup">
  ## Configuration de la CLI
</div>

```bash
openclaw onboard --auth-choice opencode-zen
# ou en mode non interactif
openclaw onboard --opencode-zen-api-key "$OPENCODE_API_KEY"
```


<div id="config-snippet">
  ## Exemple de configuration
</div>

```json5
{
  env: { OPENCODE_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-5" } } }
}
```


<div id="notes">
  ## Remarques
</div>

- `OPENCODE_ZEN_API_KEY` est également pris en charge.
- Connectez-vous à Zen, ajoutez vos informations de facturation, puis copiez votre clé API.
- OpenCode Zen facture par requête ; consultez le tableau de bord OpenCode pour plus de détails.