---
title: Configuration de l’environnement de développement
summary: "Guide de mise en place pour les développeurs travaillant sur l’app macOS OpenClaw"
read_when:
  - Configuration de l’environnement de développement macOS
---

<div id="macos-developer-setup">
  # Configuration de l’environnement de développement macOS
</div>

Ce guide décrit les étapes nécessaires pour construire et exécuter l’application macOS OpenClaw à partir du code source.

<div id="prerequisites">
  ## Prérequis
</div>

Avant de compiler l’app, assurez-vous d’avoir installé les éléments suivants :

1.  **Xcode 26.2+** : Nécessaire au développement Swift.
2.  **Node.js 22+ & pnpm** : Nécessaires pour Gateway, la CLI et les scripts de packaging.

<div id="1-install-dependencies">
  ## 1. Installer les dépendances
</div>

Installez les dépendances globales du projet :

```bash
pnpm install
```


<div id="2-build-and-package-the-app">
  ## 2. Générer et packager l’application
</div>

Pour générer l’application macOS et la packager dans `dist/OpenClaw.app`, exécutez :

```bash
./scripts/package-mac-app.sh
```

Si vous n&#39;avez pas de certificat Apple Developer ID, le script utilisera automatiquement la **signature ad hoc** (`-`).

Pour les modes d&#39;exécution de développement, les options de signature et la résolution des problèmes liés à l&#39;ID d&#39;équipe, consultez le README de l&#39;app macOS :
https://github.com/openclaw/openclaw/blob/main/apps/macos/README.md

> **Remarque** : Les apps signées ad hoc peuvent déclencher des alertes de sécurité. Si l&#39;app se bloque immédiatement avec « Abort trap 6 », consultez la section [Troubleshooting](#troubleshooting).


<div id="3-install-the-cli">
  ## 3. Installer la CLI
</div>

L’app macOS suppose qu’une CLI globale `openclaw` soit installée pour gérer les tâches en arrière-plan.

**Pour l’installer (recommandé) :**

1. Ouvrez l’app OpenClaw.
2. Allez dans l’onglet de réglages **General**.
3. Cliquez sur **&quot;Install CLI&quot;**.

Sinon, installez-la manuellement :

```bash
npm install -g openclaw@<version>
```


<div id="troubleshooting">
  ## Résolution des problèmes
</div>

<div id="build-fails-toolchain-or-sdk-mismatch">
  ### Échec de la compilation : incompatibilité de toolchain ou de SDK
</div>

La compilation de l’app macOS nécessite le dernier SDK macOS et la toolchain Swift 6.2.

**Dépendances système (obligatoires) :**

* **Dernière version de macOS disponible via Mise à jour de logiciels** (requise par les SDK Xcode 26.2)
* **Xcode 26.2** (toolchain Swift 6.2)

**Vérifications :**

```bash
xcodebuild -version
xcrun swift --version
```

Si les versions ne correspondent pas, mettez à jour macOS/Xcode et relancez la compilation.


<div id="app-crashes-on-permission-grant">
  ### L’app se bloque lors de l’octroi des autorisations
</div>

Si l’app se bloque lorsque vous essayez d’autoriser l’accès à la **reconnaissance vocale** ou au **microphone**, cela peut être dû à un cache TCC corrompu ou à un problème de correspondance de signature.

**Correctif :**

1. Réinitialisez les autorisations TCC :
   ```bash
   tccutil reset All bot.molt.mac.debug
   ```
2. Si cela échoue, modifiez temporairement le `BUNDLE_ID` dans [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) pour forcer une remise à zéro complète côté macOS.

<div id="gateway-starting-indefinitely">
  ### Gateway &quot;Starting...&quot; indefinitely
</div>

Si le statut du Gateway reste bloqué sur &quot;Starting...&quot;, vérifiez si un processus zombie occupe le port :

```bash
openclaw gateway status
openclaw gateway stop

# Si vous n'utilisez pas de LaunchAgent (mode développement / exécutions manuelles), trouvez le processus en écoute :
lsof -nP -iTCP:18789 -sTCP:LISTEN
```

Si une exécution manuelle occupe le port, arrêtez ce processus (Ctrl+C). En dernier recours, tuez le PID que vous avez trouvé ci-dessus.
