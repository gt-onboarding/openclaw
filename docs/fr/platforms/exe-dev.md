---
title: Exe Dev
summary: "Exécuter le Gateway OpenClaw sur exe.dev (VM + proxy HTTPS) pour un accès à distance"
read_when:
  - Vous souhaitez un hôte Linux bon marché et toujours disponible pour le Gateway
  - Vous souhaitez un accès distant au Control UI sans gérer votre propre VPS
---

<div id="exedev">
  # exe.dev
</div>

Objectif : le Gateway OpenClaw exécuté sur une VM exe.dev, accessible depuis votre ordinateur portable via : `https://<vm-name>.exe.xyz`

Cette page part du principe que vous utilisez l’image **exeuntu** par défaut fournie par exe.dev. Si vous avez choisi une autre distribution, adaptez les paquets en conséquence.

<div id="beginner-quick-path">
  ## Parcours rapide pour débutants
</div>

1. [https://exe.new/openclaw](https://exe.new/openclaw)
2. Saisissez votre clé ou jeton d’authentification si nécessaire
3. Cliquez sur « Agent » à côté de votre VM, et patientez…
4. ???
5. Profit

<div id="what-you-need">
  ## Prérequis
</div>

* un compte exe.dev
* un accès `ssh exe.dev` aux machines virtuelles [exe.dev](https://exe.dev) (facultatif)

<div id="automated-install-with-shelley">
  ## Installation automatisée avec Shelley
</div>

Shelley, l’agent de [exe.dev](https://exe.dev), peut installer OpenClaw instantanément à l’aide de notre
prompt. Le prompt utilisé est le suivant :

```
Installez OpenClaw (https://docs.openclaw.ai/install) sur cette VM. Utilisez les options non-interactive et accept-risk pour l'intégration openclaw. Ajoutez l'authentification ou le jeton fourni si nécessaire. Configurez nginx pour rediriger depuis le port par défaut 18789 vers l'emplacement racine dans la configuration de site activée par défaut, en veillant à activer la prise en charge WebSocket. L'appairage s'effectue via "openclaw devices list" et "openclaw device approve <request id>". Assurez-vous que le tableau de bord indique que l'état de santé d'OpenClaw est OK. exe.dev gère la redirection du port 8000 vers le port 80/443 et HTTPS pour nous, donc l'adresse "reachable" finale devrait être <vm-name>.exe.xyz, sans spécification de port.
```

<div id="manual-installation">
  ## Installation manuelle
</div>

<div id="1-create-the-vm">
  ## 1) Créer la VM
</div>

Depuis votre machine locale :

```bash
ssh exe.dev new 
```

Ensuite, connectez-vous :

```bash
ssh <vm-name>.exe.xyz
```

Astuce : gardez cette VM **stateful**. OpenClaw enregistre son état sous `~/.openclaw/` et `~/.openclaw/workspace/`.

<div id="2-install-prerequisites-on-the-vm">
  ## 2) Installer les prérequis (sur la VM)
</div>

```bash
sudo apt-get update
sudo apt-get install -y git curl jq ca-certificates openssl
```

<div id="3-install-openclaw">
  ## 3) Installer OpenClaw
</div>

Exécutez le script d&#39;installation d&#39;OpenClaw :

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

<div id="4-setup-nginx-to-proxy-openclaw-to-port-8000">
  ## 4) Configurer nginx en proxy inverse pour OpenClaw sur le port 8000
</div>

Modifiez `/etc/nginx/sites-enabled/default` en y mettant

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    listen 8000;
    listen [::]:8000;

    server_name _;

    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_http_version 1.1;

        # WebSocket support
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Standard proxy headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Paramètres de timeout pour les connexions persistantes
        proxy_read_timeout 86400s;
        proxy_send_timeout 86400s;
    }
}
```

<div id="5-access-openclaw-and-grant-privileges">
  ## 5) Accéder à OpenClaw et accorder des privilèges
</div>

Accédez à `https://<vm-name>.exe.xyz/?token=YOUR-TOKEN-FROM-TERMINAL`. Approuvez
les appareils avec `openclaw devices list` et `openclaw device approve`. En cas de doute,
utilisez Shelley dans votre navigateur !

<div id="remote-access">
  ## Accès distant
</div>

L&#39;accès distant est géré par le mécanisme d&#39;authentification de [exe.dev](https://exe.dev). Par défaut, le trafic HTTP du port 8000 est transféré vers `https://<vm-name>.exe.xyz` avec une authentification par e-mail.

<div id="updating">
  ## Mise à jour
</div>

```bash
npm i -g openclaw@latest
openclaw doctor
openclaw gateway restart
openclaw health
```

Guide : [Mise à jour](/fr/install/updating)
