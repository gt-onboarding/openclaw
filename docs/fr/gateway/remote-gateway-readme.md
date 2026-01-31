---
title: Guide du Gateway distant
summary: "Configuration d'un tunnel SSH pour connecter OpenClaw.app à un Gateway distant"
read_when: "Connexion de l'app macOS à un Gateway distant via SSH"
---

<div id="running-openclawapp-with-a-remote-gateway">
  # Utilisation d’OpenClaw.app avec un Gateway distant
</div>

OpenClaw.app utilise un tunnel SSH pour se connecter à un Gateway distant. Ce guide explique comment le configurer.

<div id="overview">
  ## Aperçu
</div>

```
┌─────────────────────────────────────────────────────────────┐
│                        Client Machine                          │
│                                                              │
│  OpenClaw.app ──► ws://127.0.0.1:18789 (local port)           │
│                     │                                        │
│                     ▼                                        │
│  SSH Tunnel ────────────────────────────────────────────────│
│                     │                                        │
└─────────────────────┼──────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                         Remote Machine                        │
│                                                              │
│  Gateway WebSocket ──► ws://127.0.0.1:18789 ──►              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```


<div id="quick-setup">
  ## Configuration rapide
</div>

<div id="step-1-add-ssh-config">
  ### Étape 1 : Ajouter la configuration SSH
</div>

Modifiez `~/.ssh/config` et ajoutez :

```ssh
Host remote-gateway
    HostName <REMOTE_IP>          # p. ex., 172.27.187.184
    User <REMOTE_USER>            # p. ex., jefferson
    LocalForward 18789 127.0.0.1:18789
    IdentityFile ~/.ssh/id_rsa
```

Remplacez `<REMOTE_IP>` et `<REMOTE_USER>` par vos propres valeurs.


<div id="step-2-copy-ssh-key">
  ### Étape 2 : Copier la clé SSH
</div>

Copiez votre clé publique vers la machine distante (saisissez le mot de passe une seule fois) :

```bash
ssh-copy-id -i ~/.ssh/id_rsa <REMOTE_USER>@<REMOTE_IP>
```


<div id="step-3-set-gateway-token">
  ### Étape 3 : Configurer le jeton Gateway
</div>

```bash
launchctl setenv OPENCLAW_GATEWAY_TOKEN "<your-token>"
```


<div id="step-4-start-ssh-tunnel">
  ### Étape 4 : Démarrez le tunnel SSH
</div>

```bash
ssh -N remote-gateway &
```


<div id="step-5-restart-openclawapp">
  ### Étape 5 : Redémarrer OpenClaw.app
</div>

```bash
# Quittez OpenClaw.app (⌘Q), puis rouvrez :
open /path/to/OpenClaw.app
```

L’app va maintenant se connecter au Gateway distant par le tunnel SSH.

***


<div id="auto-start-tunnel-on-login">
  ## Démarrage automatique du tunnel à la connexion
</div>

Pour que le tunnel SSH démarre automatiquement lorsque vous vous connectez, créez un Launch Agent.

<div id="create-the-plist-file">
  ### Créer le fichier PLIST
</div>

Enregistrez ce fichier sous `~/Library/LaunchAgents/bot.molt.ssh-tunnel.plist` :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>bot.molt.ssh-tunnel</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/ssh</string>
        <string>-N</string>
        <string>remote-gateway</string>
    </array>
    <key>KeepAlive</key>
    <true/>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```


<div id="load-the-launch-agent">
  ### Charger l’agent de lancement
</div>

```bash
launchctl bootstrap gui/$UID ~/Library/LaunchAgents/bot.molt.ssh-tunnel.plist
```

Le tunnel va désormais :

* Démarrer automatiquement lorsque vous vous connectez
* Redémarrer en cas de plantage
* Continuer à s’exécuter en arrière-plan

Remarque concernant l’ancienne configuration : supprimez tout LaunchAgent `com.openclaw.ssh-tunnel` résiduel, s’il existe.

***


<div id="troubleshooting">
  ## Dépannage
</div>

**Vérifiez que le tunnel est en cours d’exécution :**

```bash
ps aux | grep "ssh -N remote-gateway" | grep -v grep
lsof -i :18789
```

**Redémarrez le tunnel :**

```bash
launchctl kickstart -k gui/$UID/bot.molt.ssh-tunnel
```

**Arrêtez le tunnel :**

```bash
launchctl bootout gui/$UID/bot.molt.ssh-tunnel
```

***


<div id="how-it-works">
  ## Fonctionnement
</div>

| Composant | Rôle |
|-----------|------|
| `LocalForward 18789 127.0.0.1:18789` | Redirige le port local 18789 vers le port distant 18789 |
| `ssh -N` | SSH sans exécution de commandes distantes (uniquement le transfert de port) |
| `KeepAlive` | Redémarre automatiquement le tunnel en cas de panne |
| `RunAtLoad` | Démarre le tunnel au chargement de l’agent |

OpenClaw.app se connecte à `ws://127.0.0.1:18789` sur votre machine cliente. Le tunnel SSH redirige cette connexion vers le port 18789 sur la machine distante où le Gateway s’exécute.