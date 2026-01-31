---
title: Statut
summary: "Référence CLI pour `openclaw status` (diagnostics, sondes, instantanés d'utilisation)"
read_when:
  - Vous souhaitez un diagnostic rapide de la santé des canaux et des destinataires des sessions récentes
  - Vous souhaitez un statut `all` facilement copiable-collable pour le débogage
---

<div id="openclaw-status">
  # `openclaw status`
</div>

Diagnostics des canaux et des sessions.

```bash
openclaw status
openclaw status --all
openclaw status --deep
openclaw status --usage
```

Notes :

* `--deep` exécute des sondes en temps réel (WhatsApp Web + Telegram + Discord + Google Chat + Slack + Signal).
* Le résultat inclut les stores de session par agent lorsque plusieurs agents sont configurés.
* La vue d’ensemble inclut l’état de l’installation et de l’exécution du service hôte du Gateway et du nœud lorsqu’ils sont disponibles.
* La vue d’ensemble inclut le canal de mise à jour et le SHA Git (pour les déploiements depuis les sources).
* Les informations de mise à jour apparaissent dans la vue d’ensemble ; si une mise à jour est disponible, `status` affiche une indication pour exécuter `openclaw update` (voir [Updating](/fr/install/updating)).
