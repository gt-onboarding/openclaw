---
title: Surveillance de l'authentification
summary: "Surveiller l'expiration OAuth des fournisseurs de modèles"
read_when:
  - Configuration de la surveillance ou des alertes d'expiration OAuth
  - Automatisation des vérifications de renouvellement OAuth pour Claude Code / Codex
---

<div id="auth-monitoring">
  # Supervision de l’authentification
</div>

OpenClaw expose un indicateur d’état d’expiration OAuth via `openclaw models status`. Utilisez-le pour l’automatisation et la génération d’alertes ; les scripts sont des compléments facultatifs pour les workflows sur mobile.

<div id="preferred-cli-check-portable">
  ## Option recommandée : contrôle via la CLI (portable)
</div>

```bash
openclaw models status --check
```

Codes de retour :

* `0` : OK
* `1` : identifiants expirés ou manquants
* `2` : expiration imminente (dans les 24 h)

Cela fonctionne avec cron/systemd et ne nécessite pas de scripts supplémentaires.


<div id="optional-scripts-ops-phone-workflows">
  ## Scripts optionnels (ops / workflows mobiles)
</div>

Ils se trouvent dans `scripts/` et sont **optionnels**. Ils supposent un accès SSH à l’hôte du Gateway et sont optimisés pour systemd + Termux.

- `scripts/claude-auth-status.sh` utilise désormais `openclaw models status --json` comme
  source de vérité (avec repli sur des lectures directes de fichiers si la CLI n’est pas disponible),
  donc laissez `openclaw` dans le `PATH` pour les timers.
- `scripts/auth-monitor.sh` : cible de timer cron/systemd ; envoie des alertes (ntfy ou téléphone).
- `scripts/systemd/openclaw-auth-monitor.{service,timer}` : timer utilisateur systemd.
- `scripts/claude-auth-status.sh` : vérificateur d’auth Claude Code + OpenClaw (complet/json/simple).
- `scripts/mobile-reauth.sh` : flux de réauthentification guidé via SSH.
- `scripts/termux-quick-auth.sh` : widget en un seul appui pour l’état + ouverture de l’URL d’auth.
- `scripts/termux-auth-widget.sh` : flux de widget guidé complet.
- `scripts/termux-sync-widget.sh` : synchronisation des identifiants Claude Code → OpenClaw.

Si vous n’avez pas besoin d’automatisation mobile ou de timers systemd, ignorez ces scripts.