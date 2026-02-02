---
title: Plugins
summary: "Référence CLI pour `openclaw plugins` (liste, installation, activation/désactivation, diagnostic)"
read_when:
  - Vous souhaitez installer ou gérer des plugins Gateway intégrés (in-process)
  - Vous souhaitez déboguer des échecs de chargement de plugins
---

<div id="openclaw-plugins">
  # `openclaw plugins`
</div>

Gérer les plugins/extensions du Gateway (chargés dans le même processus).

Rubriques associées :

* Système de plugins : [Plugins](/fr/plugin)
* Manifeste de plugin + schéma : [Plugin manifest](/fr/plugins/manifest)
* Durcissement de la sécurité : [Security](/fr/gateway/security)

<div id="commands">
  ## Commandes
</div>

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

Les plugins fournis avec OpenClaw sont désactivés par défaut. Utilisez `plugins enable`
pour les activer.

Tous les plugins doivent être fournis avec un fichier `openclaw.plugin.json`
contenant un schéma JSON intégré (inline) (`configSchema`, même vide). Des manifestes ou
schémas manquants ou non valides empêchent le chargement du plugin et entraînent l’échec
de la validation de la configuration.

<div id="install">
  ### Installer
</div>

```bash
openclaw plugins install <path-or-spec>
```

Note de sécurité : traitez l’installation de plugins comme du code exécutable. Privilégiez les versions figées.

Archives prises en charge : `.zip`, `.tgz`, `.tar.gz`, `.tar`.

Utilisez `--link` pour éviter de copier un répertoire local (ajoute à `plugins.load.paths`) :

```bash
openclaw plugins install -l ./my-plugin
```

<div id="update">
  ### Mettre à jour
</div>

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

Les mises à jour ne s’appliquent qu’aux plugins installés depuis npm (répertoriés dans `plugins.installs`).
