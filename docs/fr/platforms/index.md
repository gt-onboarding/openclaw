---
title: Plateformes
summary: "Aperçu de la prise en charge des plateformes (Gateway + applications associées)"
read_when:
  - Recherche de la prise en charge des systèmes d'exploitation ou des chemins d'installation
  - Choix de l'emplacement d'exécution de Gateway
---

<div id="platforms">
  # Plateformes
</div>

Le cœur d’OpenClaw est écrit en TypeScript. **Node.js est l’environnement d’exécution recommandé**.
Bun n’est pas recommandé pour le Gateway (bugs avec WhatsApp/Telegram).

Des applications compagnon existent pour macOS (application de barre de menus) et pour les nœuds mobiles (iOS/Android). Des applications compagnon Windows et
Linux sont prévues, mais le Gateway est déjà entièrement pris en charge aujourd’hui.
Des applications compagnon natives pour Windows sont également prévues ; l’utilisation du Gateway via WSL2 est recommandée.

<div id="choose-your-os">
  ## Choisissez votre système d’exploitation
</div>

* macOS : [macOS](/fr/platforms/macos)
* iOS : [iOS](/fr/platforms/ios)
* Android : [Android](/fr/platforms/android)
* Windows : [Windows](/fr/platforms/windows)
* Linux : [Linux](/fr/platforms/linux)

<div id="vps-hosting">
  ## VPS &amp; hébergement
</div>

* Hub VPS : [Hébergement VPS](/fr/vps)
* Fly.io : [Fly.io](/fr/platforms/fly)
* Hetzner (Docker) : [Hetzner](/fr/platforms/hetzner)
* GCP (Compute Engine) : [GCP](/fr/platforms/gcp)
* exe.dev (VM + proxy HTTPS) : [exe.dev](/fr/platforms/exe-dev)

<div id="common-links">
  ## Liens utiles
</div>

* Guide d&#39;installation : [Démarrage](/fr/start/getting-started)
* Runbook du service Gateway : [Gateway](/fr/gateway)
* Configuration du Gateway : [Configuration](/fr/gateway/configuration)
* État du service : `openclaw gateway status`

<div id="gateway-service-install-cli">
  ## Installation du service Gateway (CLI)
</div>

Utilisez l’une des options suivantes (toutes sont prises en charge) :

* Assistant (recommandé) : `openclaw onboard --install-daemon`
* Direct : `openclaw gateway install`
* Flux de configuration : `openclaw configure` → sélectionnez **Gateway service**
* Réparation/migration : `openclaw doctor` (propose d’installer ou de réparer le service)

La cible du service dépend du système d’exploitation :

* macOS : LaunchAgent (`bot.molt.gateway` ou `bot.molt.<profile>` ; anciennement `com.openclaw.*`)
* Linux/WSL2 : service utilisateur systemd (`openclaw-gateway[-<profile>].service`)