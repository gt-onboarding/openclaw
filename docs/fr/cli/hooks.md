---
title: Hooks
summary: "Référence CLI pour `openclaw hooks` (hooks d’agent)"
read_when:
  - Vous souhaitez gérer des hooks d’agent
  - Vous souhaitez installer ou mettre à jour des hooks
---

<div id="openclaw-hooks">
  # `openclaw hooks`
</div>

Gérer les hooks d’agent (automatisations déclenchées par des événements pour des commandes comme `/new`, `/reset`, et le démarrage du Gateway).

Liens associés :

* Hooks : [Hooks](/fr/hooks)
* Hooks de plugin : [Plugins](/fr/plugin#plugin-hooks)

<div id="list-all-hooks">
  ## Répertorier tous les hooks
</div>

```bash
openclaw hooks list
```

Répertorie tous les hooks détectés dans les répertoires de l’espace de travail, gérés et intégrés.

**Options :**

* `--eligible` : Afficher uniquement les hooks éligibles (conditions requises remplies)
* `--json` : Sortie au format JSON
* `-v, --verbose` : Afficher des informations détaillées, y compris les conditions requises manquantes

**Exemple de sortie :**

```
Hooks (4/4 prêts)

Prêts :
  🚀 boot-md ✓ - Exécuter BOOT.md au démarrage de la passerelle
  📝 command-logger ✓ - Enregistrer tous les événements de commande dans un fichier d'audit centralisé
  💾 session-memory ✓ - Sauvegarder le contexte de session en mémoire lorsque la commande /new est émise
  😈 soul-evil ✓ - Échanger le contenu SOUL injecté pendant une fenêtre de purge ou par hasard
```

**Exemple (détaillé) :**

```bash
openclaw hooks list --verbose
```

Affiche les prérequis manquants pour les hooks non éligibles.

**Exemple (JSON) :**

```bash
openclaw hooks list --json
```

Renvoie un JSON structuré pour une utilisation par des programmes.

<div id="get-hook-information">
  ## Afficher les informations sur le hook
</div>

```bash
openclaw hooks info <name>
```

Afficher des informations détaillées sur un hook spécifique.

**Arguments :**

* `<name>` : Nom du hook (par exemple, `session-memory`)

**Options :**

* `--json` : Résultat au format JSON

**Exemple :**

```bash
openclaw hooks info session-memory
```

**Sortie :**

```
💾 session-memory ✓ Prêt

Enregistre le contexte de la session en mémoire lorsque la commande /new est émise

Details:
  Source: openclaw-bundled
  Path: /path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  Handler: /path/to/openclaw/hooks/bundled/session-memory/handler.ts
  Homepage: https://docs.openclaw.ai/hooks#session-memory
  Events: command:new

Requirements:
  Config: ✓ workspace.dir
```

<div id="check-hooks-eligibility">
  ## Vérifier l’éligibilité des hooks
</div>

```bash
openclaw hooks check
```

Afficher un résumé de l&#39;état d&#39;éligibilité des hooks (combien sont prêts et combien ne le sont pas).

**Options :**

* `--json`: Afficher au format JSON

**Exemple de sortie :**

```
Hooks Status

Total hooks: 4
Ready: 4
Not ready: 0
```

<div id="enable-a-hook">
  ## Activer un hook
</div>

```bash
openclaw hooks enable <name>
```

Activez un hook donné en l’ajoutant à votre fichier de configuration (`~/.openclaw/config.json`).

**Remarque :** Les hooks gérés par des plugins affichent `plugin:<id>` dans `openclaw hooks list` et
ne peuvent pas être activés ou désactivés ici. Activez ou désactivez plutôt le plugin.

**Arguments :**

* `<name>` : nom du hook (par exemple `session-memory`)

**Exemple :**

```bash
openclaw hooks enable session-memory
```

**Résultat :**

```
✓ Hook activé : 💾 session-memory
```

**Ce que cette commande fait :**

* Vérifie si le hook existe et est éligible
* Met à jour `hooks.internal.entries.<name>.enabled = true` dans votre configuration
* Enregistre la configuration sur le disque

**Après l’activation :**

* Redémarrez le Gateway pour recharger les hooks (redémarrage de l’app de barre de menu sous macOS, ou redémarrez votre processus Gateway en développement).

<div id="disable-a-hook">
  ## Désactiver un hook
</div>

```bash
openclaw hooks disable <name>
```

Désactivez un hook spécifique en mettant à jour votre configuration.

**Arguments :**

* `<name>` : nom du hook (par exemple `command-logger`)

**Exemple :**

```bash
openclaw hooks disable command-logger
```

**Sortie :**

```
⏸ Hook désactivé : 📝 command-logger
```

**Après la désactivation :**

* Redémarrez Gateway afin de recharger les hooks

<div id="install-hooks">
  ## Installer les hooks
</div>

```bash
openclaw hooks install <path-or-spec>
```

Installe un pack de hooks à partir d’un dossier ou d’une archive locaux, ou depuis npm.

**Fonctionnement :**

* Copie le pack de hooks dans `~/.openclaw/hooks/<id>`
* Active les hooks installés dans `hooks.internal.entries.*`
* Enregistre l’installation dans `hooks.internal.installs`

**Options :**

* `-l, --link` : Crée un lien vers un répertoire local au lieu de le copier (l’ajoute à `hooks.internal.load.extraDirs`)

**Archives prises en charge :** `.zip`, `.tgz`, `.tar.gz`, `.tar`

**Exemples :**

```bash
# Répertoire local
openclaw hooks install ./my-hook-pack

# Archive locale
openclaw hooks install ./my-hook-pack.zip

# Paquet npm
openclaw hooks install @openclaw/my-hook-pack

# Lier un répertoire local sans le copier
openclaw hooks install -l ./my-hook-pack
```

<div id="update-hooks">
  ## Hooks de mise à jour
</div>

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

Met à jour les packs de hooks installés (installations npm uniquement).

**Options :**

* `--all`: Met à jour tous les packs de hooks suivis
* `--dry-run`: Affiche ce qui changerait sans écrire quoi que ce soit

<div id="bundled-hooks">
  ## Hooks fournis par défaut
</div>

<div id="session-memory">
  ### session-memory
</div>

Enregistre le contexte de la session en mémoire lorsque vous lancez `/new`.

**Activation :**

```bash
openclaw hooks enable session-memory
```

**Sortie :** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md`

**Voir :** [documentation de session-memory](/fr/hooks#session-memory)

<div id="command-logger">
  ### command-logger
</div>

Consigne tous les événements liés aux commandes dans un fichier d’audit centralisé.

**Activation :**

```bash
openclaw hooks enable command-logger
```

**Sortie :** `~/.openclaw/logs/commands.log`

**Afficher les logs :**

```bash
# Commandes récentes
tail -n 20 ~/.openclaw/logs/commands.log

# Affichage formaté
cat ~/.openclaw/logs/commands.log | jq .

# Filtrer par action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**Consultez :** [la documentation de `command-logger`](/fr/hooks#command-logger)

<div id="soul-evil">
  ### soul-evil
</div>

Remplace le contenu injecté de `SOUL.md` par `SOUL_EVIL.md` pendant une fenêtre de purge ou au hasard.

**Activation :**

```bash
openclaw hooks enable soul-evil
```

**Voir aussi :** [SOUL Evil Hook](/fr/hooks/soul-evil)

<div id="boot-md">
  ### boot-md
</div>

Exécute `BOOT.md` lorsque le Gateway démarre (après le démarrage des canaux).

**Événements** : `gateway:startup`

**Activation** :

```bash
openclaw hooks enable boot-md
```

**Voir :** [documentation de boot-md](/fr/hooks#boot-md)
