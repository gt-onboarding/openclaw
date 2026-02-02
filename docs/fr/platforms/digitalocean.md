---
title: DigitalOcean
summary: "OpenClaw sur DigitalOcean (option VPS simple et payante)"
read_when:
  - Configuration d'OpenClaw sur DigitalOcean
  - Recherche d'un hébergement VPS peu coûteux pour OpenClaw
---

<div id="openclaw-on-digitalocean">
  # OpenClaw sur DigitalOcean
</div>

<div id="goal">
  ## Objectif
</div>

Exécuter un Gateway OpenClaw permanent sur DigitalOcean pour **6 $/mois** (ou 4 $/mois avec tarification réservée).

Si vous voulez une option à 0 $/mois et que l’architecture ARM + la configuration spécifique au fournisseur ne vous dérangent pas, consultez le [guide Oracle Cloud](/fr/platforms/oracle).

<div id="cost-comparison-2026">
  ## Comparaison des coûts (2026)
</div>

| Fournisseur | Offre | Spécifications | Prix/mois | Remarques |
|----------|------|-------|----------|-------|
| Oracle Cloud | Always Free ARM | jusqu’à 4 OCPU, 24 Go de RAM | 0 $ | ARM, capacité limitée / bizarreries à l’inscription |
| Hetzner | CX22 | 2 vCPU, 4 Go de RAM | 3,79 € (~4 $) | Option payante la moins chère |
| DigitalOcean | Basic | 1 vCPU, 1 Go de RAM | 6 $ | UI simple, bonne documentation |
| Vultr | Cloud Compute | 1 vCPU, 1 Go de RAM | 6 $ | Nombreuses régions disponibles |
| Linode | Nanode | 1 vCPU, 1 Go de RAM | 5 $ | Désormais intégré à Akamai |

**Choisir un fournisseur :**

* DigitalOcean : UX la plus simple + configuration prévisible (ce guide)
* Hetzner : bon rapport prix/performances (voir le [guide Hetzner](/fr/platforms/hetzner))
* Oracle Cloud : peut revenir à 0 $/mois, mais plus capricieux et uniquement ARM (voir le [guide Oracle](/fr/platforms/oracle))

***

<div id="prerequisites">
  ## Prérequis
</div>

* Compte DigitalOcean ([inscription avec 200 $ de crédit offert](https://m.do.co/c/signup))
* Paire de clés SSH (ou prêt à utiliser l’authentification par mot de passe)
* ~20 minutes

<div id="1-create-a-droplet">
  ## 1) Créer un Droplet
</div>

1. Connectez-vous à [DigitalOcean](https://cloud.digitalocean.com/)
2. Cliquez sur **Create → Droplets**
3. Choisissez :
   * **Region :** la plus proche de vous (ou de vos utilisateurs)
   * **Image :** Ubuntu 24.04 LTS
   * **Size :** Basic → Regular → **6 $/mois** (1 vCPU, 1 Go de RAM, 25 Go de SSD)
   * **Authentication :** clé SSH (recommandée) ou mot de passe
4. Cliquez sur **Create Droplet**
5. Notez l&#39;adresse IP

<div id="2-connect-via-ssh">
  ## 2) Connectez-vous en SSH
</div>

```bash
ssh root@YOUR_DROPLET_IP
```

<div id="3-install-openclaw">
  ## 3) Installer OpenClaw
</div>

```bash
# Update system
apt update && apt upgrade -y

# Installer Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs

# Install OpenClaw
curl -fsSL https://openclaw.bot/install.sh | bash

# Verify
openclaw --version
```

<div id="4-run-onboarding">
  ## 4) Lancer la procédure d’onboarding
</div>

```bash
openclaw onboard --install-daemon
```

L&#39;assistant de configuration vous guidera à travers les étapes suivantes :

* Authentification des modèles (clés API ou OAuth)
* Configuration des canaux (Telegram, WhatsApp, Discord, etc.)
* Jeton Gateway (généré automatiquement)
* Installation du démon (systemd)

<div id="5-verify-the-gateway">
  ## 5) Vérifier Gateway
</div>

```bash
# Vérifier l'état
openclaw status

# Vérifier le service
systemctl --user status openclaw-gateway.service

# Consulter les journaux
journalctl --user -u openclaw-gateway.service -f
```

<div id="6-access-the-dashboard">
  ## 6) Accéder au tableau de bord
</div>

Le Gateway écoute sur l’interface loopback par défaut. Pour accéder au Control UI :

**Option A : tunnel SSH (recommandé)**

```bash
# Depuis votre machine locale
ssh -L 18789:localhost:18789 root@YOUR_DROPLET_IP

# Puis ouvrir : http://localhost:18789
```

**Option B : Tailscale Serve (HTTPS, boucle locale uniquement)**

```bash
# On the droplet
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up

# Configurer Gateway pour utiliser Tailscale Serve
openclaw config set gateway.tailscale.mode serve
openclaw gateway restart
```

Ouvrez : `https://<magicdns>/`

Remarques :

* Serve maintient Gateway limité au loopback et s’authentifie via les en-têtes d&#39;identité Tailscale.
* Pour exiger plutôt un jeton ou mot de passe, définissez `gateway.auth.allowTailscale: false` ou utilisez `gateway.auth.mode: "password"`.

**Option C : liaison Tailnet (sans Serve)**

```bash
openclaw config set gateway.bind tailnet
openclaw gateway restart
```

Ouvrez l’URL : `http://<tailscale-ip>:18789` (jeton requis).

<div id="7-connect-your-channels">
  ## 7) Connectez vos canaux
</div>

<div id="telegram">
  ### Telegram
</div>

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

<div id="whatsapp">
  ### WhatsApp
</div>

```bash
openclaw channels login whatsapp
# Scannez le code QR
```

Voir [Channels](/fr/channels) pour d’autres fournisseurs.

***

<div id="optimizations-for-1gb-ram">
  ## Optimisations pour 1 Go de RAM
</div>

Le droplet à 6 $ ne dispose que de 1 Go de RAM. Pour que tout reste fluide :

<div id="add-swap-recommended">
  ### Ajouter du swap (recommandé)
</div>

```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

<div id="use-a-lighter-model">
  ### Utiliser un modèle plus léger
</div>

Si vous rencontrez des OOM (Out Of Memory), envisagez :

* D’utiliser des modèles basés sur une API (Claude, GPT) au lieu de modèles locaux
* De définir `agents.defaults.model.primary` sur un modèle plus petit

<div id="monitor-memory">
  ### Surveiller la mémoire
</div>

```bash
free -h
htop
```

***

<div id="persistence">
  ## Persistance
</div>

L’ensemble de l’état est stocké dans :

* `~/.openclaw/` — configuration, identifiants, données de session
* `~/.openclaw/workspace/` — espace de travail (SOUL.md, mémoire, etc.)

Ces données survivent aux redémarrages. Sauvegardez-les régulièrement :

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

***

<div id="oracle-cloud-free-alternative">
  ## Alternative gratuite avec Oracle Cloud
</div>

Oracle Cloud propose des instances ARM **Always Free** nettement plus puissantes que n&#39;importe quelle option payante ici — pour 0 $/mois.

| Ce que vous obtenez | Caractéristiques |
|--------------|-------|
| **4 OCPUs** | ARM Ampere A1 |
| **24 Go de RAM** | Largement suffisant |
| **200 Go de stockage** | Volume en mode bloc |
| **Gratuit à vie** | Aucune facturation sur carte bancaire |

**Mises en garde :**

* L&#39;inscription peut être capricieuse (réessayez en cas d&#39;échec)
* Architecture ARM — la plupart des usages fonctionnent, mais certains binaires nécessitent des versions compilées pour ARM

Pour le guide d&#39;installation complet, voir [Oracle Cloud](/fr/platforms/oracle). Pour des conseils d&#39;inscription et le dépannage du processus d&#39;inscription, consultez ce [guide communautaire](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd).

***

<div id="troubleshooting">
  ## Résolution des problèmes
</div>

<div id="gateway-wont-start">
  ### Gateway ne démarre pas
</div>

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl -u openclaw --no-pager -n 50
```

<div id="port-already-in-use">
  ### Port déjà occupé
</div>

```bash
lsof -i :18789
kill <PID>
```

<div id="out-of-memory">
  ### Mémoire insuffisante
</div>

```bash
# Vérifier la mémoire
free -h

# Ajouter plus de swap
# Ou passer à une droplet à 12 $/mois (2 Go de RAM)
```

***

<div id="see-also">
  ## Voir aussi
</div>

* [Guide Hetzner](/fr/platforms/hetzner) — moins cher, plus puissant
* [Installation Docker](/fr/install/docker) — installation conteneurisée
* [Tailscale](/fr/gateway/tailscale) — accès distant sécurisé
* [Configuration](/fr/gateway/configuration) — référence complète de la configuration