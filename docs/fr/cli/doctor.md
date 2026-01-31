---
title: Doctor
summary: "Référence CLI pour `openclaw doctor` (vérifications d’intégrité + réparations guidées)"
read_when:
  - Vous rencontrez des problèmes de connectivité ou d’authentification et souhaitez des corrections guidées
  - Vous avez effectué une mise à jour et souhaitez un contrôle de cohérence rapide
---

<div id="openclaw-doctor">
  # `openclaw doctor`
</div>

Vérifications d&#39;état et correctifs rapides pour le Gateway et les canaux.

Voir aussi :

* Dépannage : [Troubleshooting](/fr/gateway/troubleshooting)
* Audit de sécurité : [Security](/fr/gateway/security)

<div id="examples">
  ## Exemples
</div>

```bash
openclaw doctor
openclaw doctor --repair
openclaw doctor --deep
```

Notes :

* Les invites interactives (comme les corrections liées au trousseau/OAuth) ne s’exécutent que lorsque stdin est un TTY et que `--non-interactive` n’est **pas** activé. Les exécutions en mode sans interface (cron, Telegram, sans terminal) ignorent ces invites.
* `--fix` (alias de `--repair`) enregistre une sauvegarde dans `~/.openclaw/openclaw.json.bak` et supprime les clés de configuration inconnues, en listant chaque suppression.

<div id="macos-launchctl-env-overrides">
  ## macOS : surcharges d&#39;environnement `launchctl`
</div>

Si vous avez précédemment exécuté `launchctl setenv OPENCLAW_GATEWAY_TOKEN ...` (ou `...PASSWORD`), cette valeur surcharge la configuration de votre fichier et peut entraîner des erreurs « unauthorized » persistantes.

```bash
launchctl getenv OPENCLAW_GATEWAY_TOKEN
launchctl getenv OPENCLAW_GATEWAY_PASSWORD

launchctl unsetenv OPENCLAW_GATEWAY_TOKEN
launchctl unsetenv OPENCLAW_GATEWAY_PASSWORD
```
