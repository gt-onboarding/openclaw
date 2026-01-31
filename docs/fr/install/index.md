---
title: Installation
summary: "Installer OpenClaw (installateur recommandé, installation globale ou à partir des sources)"
read_when:
  - Installation d’OpenClaw
  - Vous souhaitez installer à partir de GitHub
---

<div id="install">
  # Installation
</div>

Utilisez le programme d’installation, sauf si vous avez une bonne raison de ne pas le faire. Il configure la CLI et lance la procédure d’onboarding.

<div id="quick-install-recommended">
  ## Installation rapide (recommandée)
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Windows (PowerShell) :

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

Étape suivante (si vous avez ignoré l’assistant de configuration) :

```bash
openclaw onboard --install-daemon
```

<div id="system-requirements">
  ## Configuration système requise
</div>

* **Node &gt;= 22**
* macOS, Linux ou Windows via WSL2
* `pnpm` uniquement si vous compilez depuis les sources

<div id="choose-your-install-path">
  ## Choisissez votre méthode d&#39;installation
</div>

<div id="1-installer-script-recommended">
  ### 1) Script d&#39;installation (recommandé)
</div>

Installe `openclaw` en installation globale via npm et lance la procédure de prise en main.

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Options d’installation :

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --help
```

Détails : [Fonctionnement interne de l’installateur](/fr/install/installer).

Mode non interactif (ignorer l’onboarding) :

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --no-onboard
```

<div id="2-global-install-manual">
  ### 2) Installation globale (manuelle)
</div>

Si Node.js est déjà installé :

```bash
npm install -g openclaw@latest
```

Si vous avez `libvips` installé au niveau du système (courant sur macOS via Homebrew) et que l’installation de `sharp` échoue, forcez l’utilisation de binaires précompilés :

```bash
SHARP_IGNORE_GLOBAL_LIBVIPS=1 npm install -g openclaw@latest
```

Si vous voyez `sharp: Please add node-gyp to your dependencies`, installez soit les outils de compilation (macOS : Xcode CLT + `npm install -g node-gyp`), soit utilisez la solution de contournement `SHARP_IGNORE_GLOBAL_LIBVIPS=1` ci-dessus pour éviter la compilation native.

Ou :

```bash
pnpm add -g openclaw@latest
```

Ensuite :

```bash
openclaw onboard --install-daemon
```

<div id="3-from-source-contributorsdev">
  ### 3) À partir des sources (contributeurs/développeurs)
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # installe automatiquement les dépendances de l'UI à la première exécution
pnpm build
openclaw onboard --install-daemon
```

Astuce : si vous n’avez pas encore d’installation globale, exécutez les commandes du dépôt avec `pnpm openclaw ...`.

<div id="4-other-install-options">
  ### 4) Autres méthodes d&#39;installation
</div>

* Docker : [Docker](/fr/install/docker)
* Nix : [Nix](/fr/install/nix)
* Ansible : [Ansible](/fr/install/ansible)
* Bun (CLI uniquement) : [Bun](/fr/install/bun)

<div id="after-install">
  ## Après l’installation
</div>

* Lancer l’onboarding : `openclaw onboard --install-daemon`
* Vérification rapide : `openclaw doctor`
* Vérifier l’état du Gateway : `openclaw status` + `openclaw health`
* Ouvrir le tableau de bord : `openclaw dashboard`

<div id="install-method-npm-vs-git-installer">
  ## Méthode d&#39;installation : npm ou git (programme d&#39;installation)
</div>

Le programme d&#39;installation prend en charge deux méthodes :

* `npm` (par défaut) : `npm install -g openclaw@latest`
* `git` : cloner/compilier depuis GitHub et exécuter depuis une copie locale du dépôt

<div id="cli-flags">
  ### Options de la CLI
</div>

```bash
# npm explicite
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method npm

# Installation depuis GitHub (récupération des sources)
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

Options courantes :

* `--install-method npm|git`
* `--git-dir <path>` (valeur par défaut : `~/openclaw`)
* `--no-git-update` (ignore `git pull` lors de l&#39;utilisation d&#39;un dépôt existant)
* `--no-prompt` (désactive les invites ; requis en CI/automatisation)
* `--dry-run` (affiche ce qui se passerait ; n&#39;apporte aucune modification)
* `--no-onboard` (ignore la procédure d&#39;onboarding)

<div id="environment-variables">
  ### Variables d&#39;environnement
</div>

Variables d&#39;environnement équivalentes (utiles pour l&#39;automatisation) :

* `OPENCLAW_INSTALL_METHOD=git|npm`
* `OPENCLAW_GIT_DIR=...`
* `OPENCLAW_GIT_UPDATE=0|1`
* `OPENCLAW_NO_PROMPT=1`
* `OPENCLAW_DRY_RUN=1`
* `OPENCLAW_NO_ONBOARD=1`
* `SHARP_IGNORE_GLOBAL_LIBVIPS=0|1` (valeur par défaut : `1` ; évite que `sharp` soit compilé en lien avec la bibliothèque système libvips)

<div id="troubleshooting-openclaw-not-found-path">
  ## Dépannage : `openclaw` introuvable dans PATH
</div>

Diagnostic rapide :

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

Si `$(npm prefix -g)/bin` (macOS/Linux) ou `$(npm prefix -g)` (Windows) **n’apparaît pas** dans la sortie de `echo "$PATH"`, votre shell ne peut pas trouver les binaires npm globaux (y compris `openclaw`).

Correctif : ajoutez ce chemin à votre fichier de démarrage du shell (zsh : `~/.zshrc`, bash : `~/.bashrc`) :

```bash
# macOS / Linux
export PATH="$(npm prefix -g)/bin:$PATH"
```

Sous Windows, ajoutez le résultat de la commande `npm prefix -g` à votre PATH.

Ensuite, ouvrez un nouveau terminal (ou exécutez `rehash` dans zsh / `hash -r` dans bash).

<div id="update-uninstall">
  ## Mise à jour / désinstallation
</div>

* Mise à jour : [Mise à jour](/fr/install/updating)
* Migration vers une nouvelle machine : [Migration](/fr/install/migrating)
* Désinstallation : [Désinstallation](/fr/install/uninstall)