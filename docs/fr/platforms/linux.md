---
title: Linux
summary: "Prise en charge de Linux + statut de l'application compagnon"
read_when:
  - Consultation du statut de l'application compagnon Linux
  - Planification de la couverture de la plateforme ou des contributions
---

<div id="linux-app">
  # Application Linux
</div>

Le Gateway est entièrement pris en charge sur Linux. **Node est l’environnement d’exécution recommandé**.
Bun n’est pas recommandé pour Gateway (bogues avec WhatsApp/Telegram).

Des applications compagnons natives pour Linux sont prévues. Les contributions sont les bienvenues si vous souhaitez aider à en développer une.

<div id="beginner-quick-path-vps">
  ## Parcours rapide pour débutants (VPS)
</div>

1. Installez Node 22+
2. `npm i -g openclaw@latest`
3. `openclaw onboard --install-daemon`
4. Depuis votre ordinateur portable : `ssh -N -L 18789:127.0.0.1:18789 <user>@<host>`
5. Ouvrez `http://127.0.0.1:18789/` et collez votre jeton

Guide VPS pas à pas détaillé : [exe.dev](/fr/platforms/exe-dev)

<div id="install">
  ## Installation
</div>

* [Prise en main](/fr/start/getting-started)
* [Installation et mises à jour](/fr/install/updating)
* Méthodes facultatives : [Bun (expérimental)](/fr/install/bun), [Nix](/fr/install/nix), [Docker](/fr/install/docker)

<div id="gateway">
  ## Gateway
</div>

* [Runbook du Gateway](/fr/gateway)
* [Configuration](/fr/gateway/configuration)

<div id="gateway-service-install-cli">
  ## Installation du service Gateway (CLI)
</div>

Utilisez l&#39;une des méthodes suivantes :

```
openclaw onboard --install-daemon
```

Ou :

```
openclaw gateway install
```

Ou :

```
openclaw configure
```

Sélectionnez **Gateway service** lorsque vous y êtes invité.

Réparer/migrer :

```
openclaw doctor
```

<div id="system-control-systemd-user-unit">
  ## Contrôle du système (unité systemd utilisateur)
</div>

OpenClaw installe par défaut un service systemd **utilisateur**. Utilisez un service
**système** pour les serveurs mutualisés ou toujours actifs. L’exemple complet
d’unité et les recommandations se trouvent dans le [runbook du Gateway](/fr/gateway).

Configuration minimale :

Créez `~/.config/systemd/user/openclaw-gateway[-<profile>].service` :

```
[Unit]
Description=OpenClaw Gateway (profile: <profile>, v<version>)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

Activez l’unité :

```
systemctl --user enable --now openclaw-gateway[-<profile>].service
```
