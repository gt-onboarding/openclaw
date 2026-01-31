---
title: Dépannage du navigateur sous Linux
summary: "Résoudre les problèmes de démarrage CDP de Chrome/Brave/Edge/Chromium pour le contrôle du navigateur OpenClaw sous Linux"
read_when: "Le contrôle du navigateur échoue sous Linux, en particulier avec Chromium installé via snap"
---

<div id="browser-troubleshooting-linux">
  # Dépannage du navigateur sous Linux
</div>

<div id="problem-failed-to-start-chrome-cdp-on-port-18800">
  ## Problème : &quot;Failed to start Chrome CDP on port 18800&quot;
</div>

Le serveur de contrôle du navigateur d’OpenClaw ne parvient pas à lancer Chrome/Brave/Edge/Chromium et retourne l’erreur :

```
{"error":"Error: Failed to start Chrome CDP on port 18800 for profile \"openclaw\"."}
```


<div id="root-cause">
  ### Cause racine
</div>

Sous Ubuntu (et de nombreuses distributions Linux), l’installation par défaut de Chromium est un **paquet Snap**. Le confinement AppArmor de Snap interfère avec la manière dont OpenClaw lance et surveille le processus du navigateur.

La commande `apt install chromium` installe un paquet factice qui redirige vers Snap :

```
Note, selecting 'chromium-browser' instead of 'chromium'
chromium-browser is already the newest version (2:1snap1-0ubuntu2).
```

Ce n&#39;est PAS un vrai navigateur — ce n&#39;est qu&#39;un simple wrapper.


<div id="solution-1-install-google-chrome-recommended">
  ### Solution 1 : Installer Google Chrome (recommandé)
</div>

Installez le paquet `.deb` officiel de Google Chrome, qui n’est pas exécuté dans un sandbox géré par Snap :

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt --fix-broken install -y  # si des erreurs de dépendances surviennent
```

Mettez ensuite à jour la configuration d’OpenClaw (`~/.openclaw/openclaw.json`) :

```json
{
  "browser": {
    "enabled": true,
    "executablePath": "/usr/bin/google-chrome-stable",
    "headless": true,
    "noSandbox": true
  }
}
```


<div id="solution-2-use-snap-chromium-with-attach-only-mode">
  ### Solution 2 : utiliser Chromium Snap avec le mode attach-only
</div>

Si vous devez utiliser Chromium Snap, configurez OpenClaw pour se connecter à un navigateur démarré manuellement :

1. Mettez à jour la configuration :

```json
{
  "browser": {
    "enabled": true,
    "attachOnly": true,
    "headless": true,
    "noSandbox": true
  }
}
```

2. Démarrez Chromium manuellement :

```bash
chromium-browser --headless --no-sandbox --disable-gpu \
  --remote-debugging-port=18800 \
  --user-data-dir=$HOME/.openclaw/browser/openclaw/user-data \
  about:blank &
```

3. Créez éventuellement une unité systemd utilisateur pour lancer Chrome automatiquement :

```ini
# ~/.config/systemd/user/openclaw-browser.service
[Unit]
Description=OpenClaw Browser (Chrome CDP)
After=network.target

[Service]
ExecStart=/snap/bin/chromium --headless --no-sandbox --disable-gpu --remote-debugging-port=18800 --user-data-dir=%h/.openclaw/browser/openclaw/user-data about:blank
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

Activez-le avec : `systemctl --user enable --now openclaw-browser.service`


<div id="verifying-the-browser-works">
  ### Vérifier le bon fonctionnement du navigateur
</div>

Vérifiez l’état :

```bash
curl -s http://127.0.0.1:18791/ | jq '{running, pid, chosenBrowser}'
```

Tester la navigation web :

```bash
curl -s -X POST http://127.0.0.1:18791/start
curl -s http://127.0.0.1:18791/tabs
```


<div id="config-reference">
  ### Référence de configuration
</div>

| Option | Description | Valeur par défaut |
|--------|-------------|-------------------|
| `browser.enabled` | Activer le contrôle du navigateur | `true` |
| `browser.executablePath` | Chemin vers un binaire de navigateur basé sur Chromium (Chrome/Brave/Edge/Chromium) | détecté automatiquement (privilégie le navigateur par défaut lorsqu’il est basé sur Chromium) |
| `browser.headless` | Exécuter sans interface graphique | `false` |
| `browser.noSandbox` | Ajouter l’option `--no-sandbox` (nécessaire pour certaines configurations Linux) | `false` |
| `browser.attachOnly` | Ne pas lancer le navigateur, seulement se connecter à une instance existante | `false` |
| `browser.cdpPort` | Port du Chrome DevTools Protocol | `18800` |

<div id="problem-chrome-extension-relay-is-running-but-no-tab-is-connected">
  ### Problème : « Chrome extension relay is running, but no tab is connected »
</div>

Vous utilisez le profil `chrome` (extension relay). Ce profil suppose que l’extension
de navigateur OpenClaw soit connectée à un onglet actif.

Solutions possibles :

1. **Utiliser le navigateur géré :** `openclaw browser start --browser-profile openclaw`
   (ou définir `browser.defaultProfile: "openclaw"`).
2. **Utiliser le relais d’extension :** installez l’extension, ouvrez un onglet, puis cliquez
   sur l’icône de l’extension OpenClaw pour la connecter à cet onglet.

Notes :

- Le profil `chrome` utilise, lorsque c’est possible, votre **navigateur Chromium par défaut du système**.
- Les profils locaux `openclaw` attribuent automatiquement `cdpPort`/`cdpUrl` ; ne définissez ces valeurs que pour un CDP distant.