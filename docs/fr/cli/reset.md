---
title: Réinitialisation
summary: "Référence CLI pour `openclaw reset` (réinitialiser l'état/la configuration locaux)"
read_when:
  - Vous souhaitez effacer l'état local tout en conservant la CLI installée
  - Vous souhaitez un essai à blanc (dry-run) de ce qui serait supprimé
---

<div id="openclaw-reset">
  # `openclaw reset`
</div>

Réinitialise la configuration et l’état locaux (laisse la CLI installée).

```bash
openclaw reset
openclaw reset --dry-run
openclaw reset --scope config+creds+sessions --yes --non-interactive
```
