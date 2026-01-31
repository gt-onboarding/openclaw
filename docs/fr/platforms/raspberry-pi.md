---
title: Raspberry Pi
summary: "OpenClaw sur Raspberry Pi (configuration auto-hébergée économique)"
read_when:
  - Configuration d’OpenClaw sur un Raspberry Pi
  - Exécution d’OpenClaw sur des appareils ARM
  - Construire une IA personnelle toujours disponible à faible coût
---

<div id="openclaw-on-raspberry-pi">
  # OpenClaw sur Raspberry Pi
</div>

<div id="goal">
  ## Objectif
</div>

Exécuter en continu le Gateway OpenClaw sur un Raspberry Pi pour un coût unique d’environ **35 à 80 $** (aucun frais mensuel).

Parfait pour :

* Assistant IA personnel 24/7
* Hub domotique
* Bot Telegram/WhatsApp à faible consommation et toujours disponible

<div id="hardware-requirements">
  ## Exigences matérielles
</div>

| Modèle de Pi | RAM | Fonctionne ? | Remarques |
|--------------|-----|-------------|-----------|
| **Pi 5** | 4 Go / 8 Go | ✅ Idéal | Le plus rapide, recommandé |
| **Pi 4** | 4 Go | ✅ Bon | Bon compromis pour la plupart des utilisateurs |
| **Pi 4** | 2 Go | ✅ Correct | Fonctionne, ajouter du swap |
| **Pi 4** | 1 Go | ⚠️ Limite | Possible avec du swap, configuration minimale |
| **Pi 3B+** | 1 Go | ⚠️ Lent | Fonctionne mais très lent |
| **Pi Zero 2 W** | 512 Mo | ❌ | Non recommandé |

**Configuration minimale :** 1 Go de RAM, 1 cœur, 500 Mo de stockage\
**Recommandé :** 2 Go+ de RAM, OS 64 bits, carte SD 16 Go+ (ou SSD USB)

<div id="what-youll-need">
  ## Ce dont vous aurez besoin
</div>

* Raspberry Pi 4 ou 5 (2 Go ou plus recommandés)
* Carte microSD (16 Go ou plus) ou SSD USB (meilleures performances)
* Alimentation (bloc d&#39;alimentation officiel pour Raspberry Pi recommandé)
* Connexion réseau (Ethernet ou Wi-Fi)
* ~30 minutes

<div id="1-flash-the-os">
  ## 1) Flasher l’OS
</div>

Utilisez **Raspberry Pi OS Lite (64-bit)** — aucune interface graphique n’est nécessaire pour un serveur sans écran.

1. Téléchargez [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2. Choisissez l’OS : **Raspberry Pi OS Lite (64-bit)**
3. Cliquez sur l’icône d’engrenage (⚙️) pour préconfigurer :
   * Définir le nom d’hôte : `gateway-host`
   * Activer SSH
   * Définir le nom d’utilisateur et le mot de passe
   * Configurer le Wi-Fi (si vous n’utilisez pas l’Ethernet)
4. Flashez la carte SD / le lecteur USB
5. Insérez la carte et démarrez le Pi

<div id="2-connect-via-ssh">
  ## 2) Se connecter en SSH
</div>

```bash
ssh user@gateway-host
# ou utilisez l'adresse IP
ssh user@192.168.x.x
```

<div id="3-system-setup">
  ## 3) Configuration du système
</div>

```bash
# Mettre à jour le système
sudo apt update && sudo apt upgrade -y

# Installer les paquets essentiels
sudo apt install -y git curl build-essential

# Définir le fuseau horaire (important pour cron/rappels)
sudo timedatectl set-timezone America/Chicago  # Remplacez par votre fuseau horaire
```

<div id="4-install-nodejs-22-arm64">
  ## 4) Installez Node.js 22 (ARM64)
</div>

```bash
# Install Node.js via NodeSource
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# Verify
node --version  # Devrait afficher v22.x.x
npm --version
```

<div id="5-add-swap-important-for-2gb-or-less">
  ## 5) Ajouter du swap (Important pour 2 Go ou moins)
</div>

Le swap permet d’éviter les plantages liés au manque de mémoire :

```bash
# Create 2GB swap file
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Optimiser pour RAM faible (réduire le swappiness)
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

<div id="6-install-openclaw">
  ## 6) Installez OpenClaw
</div>

<div id="option-a-standard-install-recommended">
  ### Option A : installation standard (recommandée)
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

<div id="option-b-hackable-install-for-tinkering">
  ### Option B : Installation modifiable (pour bidouiller)
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
npm install
npm run build
npm link
```

L’installation « hackable » vous donne un accès direct aux logs et au code, ce qui est utile pour déboguer les problèmes spécifiques à l’architecture ARM.

<div id="7-run-onboarding">
  ## 7) Lancer la procédure d’onboarding
</div>

```bash
openclaw onboard --install-daemon
```

Suivez l’assistant :

1. **Mode Gateway :** Local
2. **Authentification :** clés API recommandées (OAuth peut être capricieux sur un Pi sans interface graphique)
3. **Canaux :** Telegram est le plus simple pour commencer
4. **Démon :** Oui (systemd)

<div id="8-verify-installation">
  ## 8) Vérifiez l’installation
</div>

```bash
# Check status
openclaw status

# Vérifier le service
sudo systemctl status openclaw

# View logs
journalctl -u openclaw -f
```

<div id="9-access-the-dashboard">
  ## 9) Accéder au tableau de bord
</div>

Comme le Pi fonctionne sans écran, utilisez un tunnel SSH :

```bash
# Depuis votre ordinateur portable/de bureau
ssh -L 18789:localhost:18789 user@gateway-host

# Then open in browser
open http://localhost:18789
```

Ou utilisez Tailscale pour un accès continu :

```bash
# On the Pi
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# Mise à jour de la configuration
openclaw config set gateway.bind tailnet
sudo systemctl restart openclaw
```

***

<div id="performance-optimizations">
  ## Optimisation des performances
</div>

<div id="use-a-usb-ssd-huge-improvement">
  ### Utiliser un SSD USB (amélioration majeure)
</div>

Les cartes SD sont lentes et s&#39;usent vite. Un SSD USB améliore considérablement les performances :

```bash
# Vérifier si le démarrage se fait depuis USB
lsblk
```

Consultez le [guide de démarrage USB du Raspberry Pi](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#usb-mass-storage-boot) pour la configuration.

<div id="reduce-memory-usage">
  ### Réduire la consommation de mémoire
</div>

```bash
# Désactiver l'allocation de mémoire GPU (sans interface graphique)
echo 'gpu_mem=16' | sudo tee -a /boot/config.txt

# Disable Bluetooth if not needed
sudo systemctl disable bluetooth
```

<div id="monitor-resources">
  ### Surveillance des ressources
</div>

```bash
# Check memory
free -h

# Vérifier la température du processeur
vcgencmd measure_temp

# Live monitoring
htop
```

***

<div id="arm-specific-notes">
  ## Remarques spécifiques à ARM
</div>

<div id="binary-compatibility">
  ### Compatibilité binaire
</div>

La plupart des fonctionnalités d&#39;OpenClaw fonctionnent sur ARM64, mais certains binaires externes peuvent nécessiter des builds ARM :

| Outil | Statut ARM64 | Remarques |
|-------|--------------|-----------|
| Node.js | ✅ | Fonctionne très bien |
| WhatsApp (Baileys) | ✅ | 100 % JS, aucun problème |
| Telegram | ✅ | 100 % JS, aucun problème |
| gog (Gmail CLI) | ⚠️ | Vérifiez l&#39;existence d&#39;une version ARM |
| Chromium (browser) | ✅ | `sudo apt install chromium-browser` |

Si une compétence échoue, vérifiez si son binaire dispose d&#39;un build ARM. De nombreux outils Go/Rust en fournissent ; certains non.

<div id="32-bit-vs-64-bit">
  ### 32 bits vs 64 bits
</div>

**Utilisez toujours un système d’exploitation 64 bits.** Node.js et de nombreux outils modernes l’exigent. Vérifiez avec :

```bash
uname -m
# Devrait afficher : aarch64 (64 bits) et non armv7l (32 bits)
```

***

<div id="recommended-model-setup">
  ## Configuration recommandée des modèles
</div>

Comme le Pi ne fait tourner que le Gateway (les modèles s’exécutent dans le cloud), utilisez des modèles via api :

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-20250514",
        "fallbacks": ["openai/gpt-4o-mini"]
      }
    }
  }
}
```

**N’essaie pas de faire tourner des LLM locaux sur un Pi** — même les petits modèles sont beaucoup trop lents. Laisse Claude ou GPT faire le gros du travail.

***

<div id="auto-start-on-boot">
  ## Démarrage automatique au démarrage du système
</div>

L’assistant de configuration initiale met cela en place, mais pour vérifier :

```bash
# Vérifier que le service est activé
sudo systemctl is-enabled openclaw

# Activer si ce n'est pas le cas
sudo systemctl enable openclaw

# Démarrer au démarrage
sudo systemctl start openclaw
```

***

<div id="troubleshooting">
  ## Dépannage
</div>

<div id="out-of-memory-oom">
  ### Manque de mémoire (OOM)
</div>

```bash
# Vérifier la mémoire
free -h

# Ajouter davantage de swap (voir Étape 5)
# Ou réduire les services en cours d'exécution sur le Pi
```

<div id="slow-performance">
  ### Performances lentes
</div>

* Utilisez un SSD USB au lieu d&#39;une carte SD
* Désactivez les services inutilisés : `sudo systemctl disable cups bluetooth avahi-daemon`
* Vérifiez si le CPU est bridé : `vcgencmd get_throttled` (devrait renvoyer `0x0`)

<div id="service-wont-start">
  ### Le service ne démarre pas
</div>

```bash
# Check logs
journalctl -u openclaw --no-pager -n 100

# Correction courante : reconstruire
cd ~/openclaw  # si installation hackable
npm run build
sudo systemctl restart openclaw
```

<div id="arm-binary-issues">
  ### Problèmes de binaires ARM
</div>

Si une compétence échoue avec « exec format error » :

1. Vérifiez si le binaire dispose d’une version ARM64
2. Essayez de le compiler depuis les sources
3. Ou utilisez un conteneur Docker avec prise en charge d’ARM

<div id="wifi-drops">
  ### Coupures Wi‑Fi
</div>

Pour les Raspberry Pi sans écran connectés en Wi‑Fi :

```bash
# Désactiver la gestion de l'alimentation WiFi
sudo iwconfig wlan0 power off

# Make permanent
echo 'wireless-power off' | sudo tee -a /etc/network/interfaces
```

***

<div id="cost-comparison">
  ## Comparaison des coûts
</div>

| Configuration | Coût initial | Coût mensuel | Remarques |
|-------|---------------|--------------|-------|
| **Pi 4 (2GB)** | ~45 $ | 0 $ | + électricité (~5 $/an) |
| **Pi 4 (4GB)** | ~55 $ | 0 $ | Recommandé |
| **Pi 5 (4GB)** | ~60 $ | 0 $ | Meilleures performances |
| **Pi 5 (8GB)** | ~80 $ | 0 $ | Surdimensionné mais prêt pour l’avenir |
| DigitalOcean | 0 $ | 6 $/mois | 72 $/an |
| Hetzner | 0 $ | 3,79 €/mois | ~50 $/an |

**Seuil de rentabilité :** un Pi est amorti en ~6 à 12 mois par rapport à un VPS cloud.

***

<div id="see-also">
  ## Voir aussi
</div>

* [Guide Linux](/fr/platforms/linux) — configuration Linux générale
* [Guide DigitalOcean](/fr/platforms/digitalocean) — alternative cloud
* [Guide Hetzner](/fr/platforms/hetzner) — configuration Docker
* [Tailscale](/fr/gateway/tailscale) — accès à distance
* [Nœuds](/fr/nodes) — associez votre ordinateur portable ou votre téléphone au Gateway sur le Pi