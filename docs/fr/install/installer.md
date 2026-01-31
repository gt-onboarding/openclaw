---
title: Programme d’installation
summary: "Fonctionnement des scripts d’installation (install.sh + install-cli.sh), options et automatisation"
read_when:
  - Vous souhaitez comprendre `openclaw.bot/install.sh`
  - Vous souhaitez automatiser les installations (CI / mode sans interface)
  - Vous souhaitez installer depuis un dépôt GitHub
---

<div id="installer-internals">
  # Fonctionnement interne de l’installateur
</div>

OpenClaw fournit deux scripts d’installation (servis depuis `openclaw.ai`) :

* `https://openclaw.bot/install.sh` — installateur « recommandé » (installation npm globale par défaut ; peut également installer à partir d’un clone GitHub)
* `https://openclaw.bot/install-cli.sh` — installateur CLI compatible sans droits root (installe dans un préfixe avec sa propre instance Node.js)
* `https://openclaw.ai/install.ps1` — installateur Windows PowerShell (npm par défaut ; installation git facultative)

Pour voir les options/comportements actuels, exécutez :

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --help
```

Aide pour Windows (PowerShell) :

```powershell
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -?
```

Si le programme d’installation se termine correctement mais que `openclaw` n’est pas trouvé dans un nouveau terminal, il s’agit généralement d’un problème de PATH Node/npm. Voir : [Installation](/fr/install#nodejs--npm-path-sanity).

<div id="installsh-recommended">
  ## install.sh (recommandé)
</div>

Ce que fait ce script (vue d’ensemble) :

* Détecte l’OS (macOS / Linux / WSL).
* Vérifie la présence de Node.js **22+** (macOS via Homebrew ; Linux via NodeSource).
* Choisit la méthode d’installation :
  * `npm` (par défaut) : `npm install -g openclaw@latest`
  * `git` : clone/compile un checkout du code source et installe un script wrapper
* Sous Linux : évite les erreurs de permissions npm globales en basculant le préfixe npm vers `~/.npm-global` si nécessaire.
* En cas de mise à niveau d’une installation existante : exécute `openclaw doctor --non-interactive` (dans la mesure du possible).
* Pour les installations git : exécute `openclaw doctor --non-interactive` après l’installation/la mise à jour (dans la mesure du possible).
* Atténue les écueils d’installation native de `sharp` en définissant par défaut `SHARP_IGNORE_GLOBAL_LIBVIPS=1` (évite la compilation contre le libvips système).

Si vous *souhaitez* que `sharp` se lie à une bibliothèque libvips installée globalement (ou si vous déboguez), définissez :

```bash
SHARP_IGNORE_GLOBAL_LIBVIPS=0 curl -fsSL https://openclaw.bot/install.sh | bash
```

<div id="discoverability-git-install-prompt">
  ### Découvrabilité / invite « git install »
</div>

Si vous exécutez l’installateur alors que vous êtes **déjà dans un checkout du code source OpenClaw** (détecté via `package.json` + `pnpm-workspace.yaml`), il affiche :

* mettre à jour et utiliser ce checkout (`git`)
* ou migrer vers l’installation npm globale (`npm`)

Dans des contextes non interactifs (pas de TTY / `--no-prompt`), vous devez fournir `--install-method git|npm` (ou définir `OPENCLAW_INSTALL_METHOD`), sinon le script se termine avec le code `2`.

<div id="why-git-is-needed">
  ### Pourquoi Git est nécessaire
</div>

Git est requis lorsque vous utilisez `--install-method git` (clone / pull).

Pour les installations via `npm`, Git n’est *généralement* pas nécessaire, mais certains environnements finissent malgré tout par en avoir besoin (par exemple lorsqu’un package ou une dépendance est récupéré via une URL git). Le programme d’installation vérifie actuellement que Git est présent afin d’éviter les mauvaises surprises de type `spawn git ENOENT` sur des distributions fraîchement installées.

<div id="why-npm-hits-eacces-on-fresh-linux">
  ### Pourquoi npm rencontre l’erreur `EACCES` sur une installation Linux neuve
</div>

Sur certaines configurations Linux (en particulier après avoir installé Node via le gestionnaire de paquets du système ou via NodeSource), le préfixe global de npm pointe vers un emplacement appartenant à root. Dans ce cas, `npm install -g ...` échoue avec des erreurs de permissions `EACCES` / `mkdir`.

`install.sh` atténue ce problème en basculant le préfixe vers :

* `~/.npm-global` (et en l’ajoutant au `PATH` dans `~/.bashrc` / `~/.zshrc` lorsqu’ils existent)

<div id="install-clish-non-root-cli-installer">
  ## install-cli.sh (programme d’installation CLI sans privilèges root)
</div>

Ce script installe `openclaw` dans un préfixe (par défaut : `~/.openclaw`) et installe également un environnement d’exécution Node dédié sous ce préfixe, afin de pouvoir fonctionner sur des machines où vous ne souhaitez pas modifier l’installation système de Node/npm.

Aide :

```bash
curl -fsSL https://openclaw.bot/install-cli.sh | bash -s -- --help
```

<div id="installps1-windows-powershell">
  ## install.ps1 (Windows PowerShell)
</div>

Ce qu’il fait (vue d’ensemble) :

* Vérifie que Node.js **22+** est installé (via winget/Chocolatey/Scoop ou manuellement).
* Sélectionne la méthode d’installation :
  * `npm` (par défaut) : `npm install -g openclaw@latest`
  * `git` : clone/compile un checkout du code source et installe un script wrapper
* Exécute `openclaw doctor --non-interactive` lors des mises à jour et des installations via git (dans la mesure du possible).

Exemples :

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex -InstallMethod git
```

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex -InstallMethod git -GitDir "C:\\openclaw"
```

Variables d&#39;environnement :

* `OPENCLAW_INSTALL_METHOD=git|npm`
* `OPENCLAW_GIT_DIR=...`

Exigence Git :

Si vous choisissez `-InstallMethod git` et que Git n&#39;est pas installé, le programme d&#39;installation affichera le lien
Git for Windows (`https://git-scm.com/download/win`) puis se fermera.

Problèmes Windows fréquents :

* **npm error spawn git / ENOENT** : installez Git for Windows, rouvrez PowerShell, puis relancez le programme d&#39;installation.
* **&quot;openclaw&quot; is not recognized** : votre dossier `bin` global de `npm` n&#39;est pas dans `PATH`. La plupart des systèmes utilisent
  `%AppData%\\npm`. Vous pouvez aussi exécuter `npm config get prefix` et ajouter `\\bin` à `PATH`, puis rouvrir PowerShell.
