---
title: État de santé
summary: "Référence CLI pour `openclaw health` (endpoint de santé du Gateway via RPC)"
read_when:
  - Vous souhaitez vérifier rapidement l’état de santé du Gateway en cours d’exécution
---

<div id="openclaw-health">
  # `openclaw health`
</div>

Récupérer l&#39;état de santé du Gateway en cours d&#39;exécution.

```bash
openclaw health
openclaw health --json
openclaw health --verbose
```

Remarques :

* `--verbose` exécute des tests en temps réel et affiche les temps par compte lorsque plusieurs comptes sont configurés.
* La sortie inclut les stockages de sessions par agent lorsque plusieurs agents sont configurés.
