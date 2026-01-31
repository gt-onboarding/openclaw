---
title: Canaux
summary: "Référence CLI pour `openclaw channels` (comptes, statut, connexion/déconnexion, journaux)"
read_when:
  - Vous souhaitez ajouter ou supprimer des comptes de canaux (WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (plugin)/Signal/iMessage)
  - Vous souhaitez consulter le statut des canaux ou suivre en continu leurs journaux
---

<div id="openclaw-channels">
  # `openclaw channels`
</div>

Gérer les comptes de canaux de discussion et leur statut d’exécution au sein du Gateway.

Documentation associée :

* Guides des canaux de discussion : [Channels](/fr/channels/index)
* Configuration du Gateway : [Configuration](/fr/gateway/configuration)

<div id="common-commands">
  ## Commandes courantes
</div>

```bash
openclaw channels list
openclaw channels status
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels logs --channel all
```

<div id="add-remove-accounts">
  ## Ajouter et supprimer des comptes
</div>

```bash
openclaw channels add --channel telegram --token <bot-token>
openclaw channels remove --channel telegram --delete
```

Astuce : `openclaw channels add --help` affiche les options propres à chaque canal (jeton, jeton d&#39;application, chemins de signal-cli, etc.).

<div id="login-logout-interactive">
  ## Connexion / déconnexion (mode interactif)
</div>

```bash
openclaw channels login --channel whatsapp
openclaw channels logout --channel whatsapp
```

<div id="troubleshooting">
  ## Dépannage
</div>

* Exécutez `openclaw status --deep` pour une analyse approfondie.
* Utilisez `openclaw doctor` pour des corrections assistées.
* `openclaw channels list` affiche `Claude: HTTP 403 ... user:profile` → l’instantané d’utilisation nécessite la portée `user:profile`. Utilisez `--no-usage`, ou fournissez une clé de session claude.ai (`CLAUDE_WEB_SESSION_KEY` / `CLAUDE_WEB_COOKIE`), ou réauthentifiez-vous via la CLI Claude Code.

<div id="capabilities-probe">
  ## Analyse des capacités
</div>

Récupère des informations sur les capacités du fournisseur (intents/scopes lorsqu’ils sont disponibles), ainsi que sur la prise en charge des fonctionnalités statiques :

```bash
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
```

Notes :

* `--channel` est optionnel ; omettez-le pour lister tous les canaux (extensions incluses).
* `--target` accepte `channel:<id>` ou un identifiant de canal numérique brut et ne s’applique qu’à Discord.
* Les sondes sont spécifiques au fournisseur : intents Discord + autorisations de canal optionnelles ; bot Slack + portées utilisateur ; indicateurs du bot Telegram + webhook ; version du démon Signal ; jeton d’application MS Teams + rôles/portées Graph (annotés le cas échéant). Les canaux sans sonde affichent `Probe: unavailable`.

<div id="resolve-names-to-ids">
  ## Résoudre les noms en identifiants
</div>

Résolvez les noms de canaux et d’utilisateurs en identifiants à l’aide du répertoire de fournisseurs :

```bash
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels resolve --channel discord "My Server/#support" "@someone"
openclaw channels resolve --channel matrix "Project Room"
```

Remarques :

* Utilisez `--kind user|group|auto` pour forcer le type cible.
* La résolution privilégie les correspondances actives lorsque plusieurs entrées portent le même nom.
