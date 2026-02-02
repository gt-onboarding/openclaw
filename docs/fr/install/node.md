---
title: "Node.js + npm (PATH correct)"
summary: "Installation correcte de Node.js + npm : versions, PATH et installations globales"
read_when:
  - "Vous avez installé OpenClaw mais `openclaw` est « command not found »"
  - "Vous configurez Node.js/npm sur une nouvelle machine"
  - "npm install -g ... échoue en raison de problèmes de permissions ou de PATH"
---

<div id="nodejs-npm-path-sanity">
  # Node.js + npm (validité du PATH)
</div>

Le runtime d’OpenClaw repose sur **Node 22+**.

Si vous pouvez exécuter `npm install -g openclaw@latest` mais que vous voyez ensuite `openclaw: command not found`, c’est presque toujours un problème de **PATH** : le répertoire dans lequel npm installe les binaires globaux n’est pas présent dans le PATH de votre shell.

<div id="quick-diagnosis">
  ## Diagnostic rapide
</div>

Exécutez :

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

Si `$(npm prefix -g)/bin` (macOS/Linux) ou `$(npm prefix -g)` (Windows) n’apparaît **pas** dans la sortie de `echo "$PATH"`, votre shell ne peut pas trouver les exécutables npm globaux (y compris `openclaw`).


<div id="fix-put-npms-global-bin-dir-on-path">
  ## Correctif : ajouter le répertoire global des binaires npm au PATH
</div>

1. Trouvez votre préfixe global de npm :

```bash
npm prefix -g
```

2. Ajoutez le répertoire `bin` global de npm à votre fichier de démarrage du shell :

* zsh : `~/.zshrc`
* bash : `~/.bashrc`

Exemple (remplacez le chemin par la valeur renvoyée par `npm prefix -g`) :

```bash
# macOS / Linux
export PATH="/path/from/npm/prefix/bin:$PATH"
```

Ensuite, ouvrez un **nouveau terminal** (ou exécutez `rehash` dans zsh / `hash -r` dans bash).

Sous Windows, ajoutez le résultat de `npm prefix -g` à votre PATH.


<div id="fix-avoid-sudo-npm-install-g-permission-errors-linux">
  ## Correctif : éviter `sudo npm install -g` / erreurs de droits d’accès (Linux)
</div>

Si `npm install -g ...` échoue avec `EACCES`, modifiez le préfixe global de npm pour qu’il pointe vers un répertoire accessible en écriture par l’utilisateur :

```bash
mkdir -p "$HOME/.npm-global"
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

Rendez la ligne `export PATH=...` permanente dans le fichier de démarrage de votre shell.


<div id="recommended-node-install-options">
  ## Options recommandées pour l’installation de Node
</div>

Vous limiterez les mauvaises surprises si Node et npm sont installés de façon à :

- garder Node à jour (22+)
- rendre le répertoire `bin` global de npm stable et présent dans le PATH dans les nouveaux shells

Choix courants :

- macOS : Homebrew (`brew install node`) ou un gestionnaire de versions
- Linux : votre gestionnaire de versions préféré, ou une installation prise en charge par la distribution qui fournit Node 22+
- Windows : programme d’installation officiel de Node, `winget`, ou un gestionnaire de versions Node pour Windows

Si vous utilisez un gestionnaire de versions (nvm/fnm/asdf/etc.), assurez-vous qu’il est initialisé dans le shell que vous utilisez au quotidien (zsh vs bash) afin que le PATH qu’il définit soit présent lorsque vous exécutez des installateurs.