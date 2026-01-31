---
title: Gateway intégré
summary: "Exécution du Gateway sur macOS (service launchd externe)"
read_when:
  - Packaging d’OpenClaw.app
  - Débogage du service launchd du Gateway macOS
  - Installation de la CLI du Gateway pour macOS
---

<div id="gateway-on-macos-external-launchd">
  # Gateway sur macOS (launchd externe)
</div>

OpenClaw.app n’intègre plus Node/Bun ni le runtime du Gateway. L’application macOS
suppose désormais une installation `openclaw` CLI **externe**, ne démarre pas le Gateway en tant que
processus enfant et gère un service launchd par utilisateur pour maintenir le Gateway
en cours d’exécution (ou se connecte à une instance locale existante du Gateway si elle est déjà en cours d’exécution).

<div id="install-the-cli-required-for-local-mode">
  ## Installer la CLI (requis pour le mode local)
</div>

Vous devez disposer de Node 22+ sur votre Mac, puis installer `openclaw` globalement :

```bash
npm install -g openclaw@<version>
```

Le bouton **Install CLI** de l’application macOS lance la même procédure via npm/pnpm (bun n’est pas recommandé pour le runtime Gateway).


<div id="launchd-gateway-as-launchagent">
  ## Launchd (Gateway en tant que LaunchAgent)
</div>

Label :

- `bot.molt.gateway` (ou `bot.molt.<profile>` ; les anciens `com.openclaw.*` peuvent rester)

Emplacement du fichier plist (par utilisateur) :

- `~/Library/LaunchAgents/bot.molt.gateway.plist`
  (ou `~/Library/LaunchAgents/bot.molt.<profile>.plist`)

Gestionnaire :

- L’app macOS gère l’installation/la mise à jour du LaunchAgent en mode Local.
- La CLI peut aussi l’installer : `openclaw gateway install`.

Comportement :

- « OpenClaw Active » active/désactive le LaunchAgent.
- La fermeture de l’app **n’arrête pas** le Gateway (launchd le maintient actif).
- Si un Gateway est déjà en cours d’exécution sur le port configuré, l’app s’y
  connecte au lieu d’en démarrer un nouveau.

Journalisation :

- stdout/err de launchd : `/tmp/openclaw/openclaw-gateway.log`

<div id="version-compatibility">
  ## Compatibilité des versions
</div>

L’application macOS vérifie la version de Gateway par rapport à sa propre version. Si elles
ne sont pas compatibles, mettez à jour la CLI globale pour qu’elle corresponde à la version de l’application.

<div id="smoke-check">
  ## Test rapide
</div>

```bash
openclaw --version

OPENCLAW_SKIP_CHANNELS=1 \
OPENCLAW_SKIP_CANVAS_HOST=1 \
openclaw gateway --port 18999 --bind loopback
```

Ensuite :

```bash
openclaw gateway call health --url ws://127.0.0.1:18999 --timeout 3000
```
