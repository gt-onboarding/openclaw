---
title: Nix
summary: "Installer OpenClaw de manière déclarative avec Nix"
read_when:
  - Vous souhaitez des installations reproductibles, avec possibilité de retour arrière
  - Vous utilisez déjà Nix/NixOS/Home Manager
  - Vous souhaitez que tout soit figé et géré de manière déclarative
---

<div id="nix-installation">
  # Installation avec Nix
</div>

La méthode recommandée pour exécuter OpenClaw avec Nix est d’utiliser **[nix-openclaw](https://github.com/openclaw/nix-openclaw)** — un module Home Manager prêt à l’emploi.

<div id="quick-start">
  ## Démarrage rapide
</div>

Collez ce qui suit dans votre agent IA (Claude, Cursor, etc.) :

```text
I want to set up nix-openclaw on my Mac.
Repository: github:openclaw/nix-openclaw

What I need you to do:
1. Check if Determinate Nix is installed (if not, install it)
2. Create a local flake at ~/code/openclaw-local using templates/agent-first/flake.nix
3. Help me create a Telegram bot (@BotFather) and get my chat ID (@userinfobot)
4. Set up secrets (bot token, Anthropic key) - plain files at ~/.secrets/ is fine
5. Fill in the template placeholders and run home-manager switch
6. Verify: launchd running, bot responds to messages

Reference the nix-openclaw README for module options.
```

> **📦 Guide complet : [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)**
>
> Le dépôt nix-openclaw est la source de référence pour l’installation de Nix. Cette page ne propose qu’un aperçu rapide.

<div id="what-you-get">
  ## Ce que vous obtenez
</div>

* Gateway + app macOS + outils (whisper, spotify, caméras) — tous en versions figées
* Service launchd qui persiste aux redémarrages
* Système de plugin avec configuration déclarative
* Restauration instantanée : `home-manager switch --rollback`

***

<div id="nix-mode-runtime-behavior">
  ## Comportement à l’exécution en mode Nix
</div>

Lorsque `OPENCLAW_NIX_MODE=1` est défini (automatique avec nix-openclaw) :

OpenClaw prend en charge un **mode Nix** qui rend la configuration déterministe et désactive les flux d’auto‑installation.
Activez‑le en exportant :

```bash
OPENCLAW_NIX_MODE=1
```

Sous macOS, l&#39;application GUI n&#39;hérite pas automatiquement des variables d&#39;environnement du shell. Vous pouvez aussi activer le mode Nix avec `defaults` :

```bash
defaults write bot.molt.mac openclaw.nixMode -bool true
```

<div id="config-state-paths">
  ### Chemins de configuration et d’état
</div>

OpenClaw lit la configuration JSON5 à partir de `OPENCLAW_CONFIG_PATH` et stocke les données mutables dans `OPENCLAW_STATE_DIR`.

* `OPENCLAW_STATE_DIR` (par défaut : `~/.openclaw`)
* `OPENCLAW_CONFIG_PATH` (par défaut : `$OPENCLAW_STATE_DIR/openclaw.json`)

Lors de l’exécution en mode Nix, définissez-les explicitement sur des emplacements gérés par Nix afin que l’état d’exécution et la configuration restent en dehors du Nix store immuable.

<div id="runtime-behavior-in-nix-mode">
  ### Comportement à l&#39;exécution en mode Nix
</div>

* Les mécanismes d&#39;auto-installation et d&#39;auto-mutation sont désactivés
* Les dépendances manquantes déclenchent l&#39;affichage de messages de résolution spécifiques à Nix
* L&#39;UI affiche une bannière indiquant le mode Nix en lecture seule lorsqu&#39;il est actif

<div id="packaging-note-macos">
  ## Remarque sur le packaging (macOS)
</div>

Le processus de packaging macOS attend un modèle Info.plist stable à :

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) copie ce modèle dans le bundle de l’app et met à jour les champs dynamiques
(bundle ID, version/build, Git SHA, clés Sparkle). Cela garantit un plist déterministe pour l’empaquetage SwiftPM
et les builds Nix (qui ne s’appuient pas sur une chaîne d’outils Xcode complète).

<div id="related">
  ## Ressources associées
</div>

* [nix-openclaw](https://github.com/openclaw/nix-openclaw) — guide d&#39;installation complet
* [Assistant de configuration](/fr/start/wizard) — configuration de la CLI sans Nix
* [Docker](/fr/install/docker) — configuration conteneurisée