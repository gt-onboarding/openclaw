---
title: Journaux
summary: "Référence de la CLI pour `openclaw logs` (suivre en temps réel les journaux du Gateway via RPC)"
read_when:
  - Vous devez suivre en temps réel les journaux du Gateway à distance (sans SSH)
  - Vous avez besoin de lignes de journaux JSON pour vos outils
---

<div id="openclaw-logs">
  # `openclaw logs`
</div>

Suivre en temps réel les fichiers journaux du service Gateway via RPC (fonctionne en mode distant).

Liens associés :

* Vue d’ensemble de la journalisation : [Logging](/fr/logging)

<div id="examples">
  ## Exemples
</div>

```bash
openclaw logs
openclaw logs --follow
openclaw logs --json
openclaw logs --limit 500
```
