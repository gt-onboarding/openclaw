---
title: Oracle
summary: "OpenClaw sur Oracle Cloud (Always Free ARM)"
read_when:
  - Configurer OpenClaw sur Oracle Cloud
  - Rechercher un hébergement VPS peu coûteux pour OpenClaw
  - Faire tourner OpenClaw 24h/24 et 7j/7 sur un petit serveur
---

<div id="openclaw-on-oracle-cloud-oci">
  # OpenClaw sur Oracle Cloud (OCI)
</div>

<div id="goal">
  ## Objectif
</div>

Exécuter un Gateway OpenClaw persistant sur le niveau ARM **Always Free** d’Oracle Cloud.

Le niveau gratuit d’Oracle peut très bien convenir à OpenClaw (surtout si vous avez déjà un compte OCI), mais il implique certains compromis :

* Architecture ARM (la plupart des composants fonctionnent, mais certains binaires peuvent être uniquement x86)
* La capacité disponible et le processus d’inscription peuvent être capricieux

<div id="cost-comparison-2026">
  ## Comparaison des coûts (2026)
</div>

| Fournisseur | Offre | Caractéristiques | Prix/mois | Notes |
|------------|-------|------------------|----------|-------|
| Oracle Cloud | Always Free ARM | jusqu&#39;à 4 OCPU, 24 Go de RAM | $0 | ARM, capacité limitée |
| Hetzner | CX22 | 2 vCPU, 4 Go de RAM | ~ $4 | Option payante la moins chère |
| DigitalOcean | Basic | 1 vCPU, 1 Go de RAM | $6 | UI intuitive, bonne documentation |
| Vultr | Cloud Compute | 1 vCPU, 1 Go de RAM | $6 | Nombreux emplacements géographiques |
| Linode | Nanode | 1 vCPU, 1 Go de RAM | $5 | Désormais intégré à Akamai |

***

<div id="prerequisites">
  ## Prérequis
</div>

* Compte Oracle Cloud ([inscription](https://www.oracle.com/cloud/free/)) — consultez le [guide communautaire d&#39;inscription](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd) si vous rencontrez des problèmes
* Compte Tailscale (gratuit sur [tailscale.com](https://tailscale.com))
* ~30 minutes

<div id="1-create-an-oci-instance">
  ## 1) Créer une instance OCI
</div>

1. Connectez-vous à la [console Oracle Cloud](https://cloud.oracle.com/)
2. Accédez à **Compute → Instances → Create Instance**
3. Configurez :
   * **Name :** `openclaw`
   * **Image :** Ubuntu 24.04 (aarch64)
   * **Shape :** `VM.Standard.A1.Flex` (Ampere ARM)
   * **OCPUs :** 2 (ou jusqu’à 4)
   * **Memory :** 12 GB (ou jusqu’à 24 GB)
   * **Boot volume :** 50 GB (jusqu’à 200 GB disponibles)
   * **SSH key :** Ajoutez votre clé publique
4. Cliquez sur **Create**
5. Notez l’adresse IP publique

**Conseil :** Si la création de l’instance échoue avec « Out of capacity », essayez un autre domaine de disponibilité ou réessayez plus tard. La capacité du Free Tier est limitée.

<div id="2-connect-and-update">
  ## 2) Connectez-vous et mettez à jour
</div>

```bash
# Connexion via l'IP publique
ssh ubuntu@YOUR_PUBLIC_IP

# Mise à jour du système
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential
```

**Remarque :** `build-essential` est nécessaire pour la compilation ARM de certaines dépendances.

<div id="3-configure-user-and-hostname">
  ## 3) Configurer l’utilisateur et le nom d’hôte
</div>

```bash
# Définir le nom d'hôte
sudo hostnamectl set-hostname openclaw

# Définir le mot de passe de l'utilisateur ubuntu
sudo passwd ubuntu

# Activer le lingering (maintient les services utilisateur en cours d'exécution après la déconnexion)
sudo loginctl enable-linger ubuntu
```

<div id="4-install-tailscale">
  ## 4) Installer Tailscale
</div>

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --ssh --hostname=openclaw
```

Cela active Tailscale SSH, ce qui vous permet de vous connecter via `ssh openclaw` depuis n&#39;importe quel appareil sur votre tailnet — aucune adresse IP publique n&#39;est nécessaire.

Vérifiez :

```bash
tailscale status
```

**Désormais, connectez-vous via Tailscale :** `ssh ubuntu@openclaw` (ou utilisez l’adresse IP Tailscale).

<div id="5-install-openclaw">
  ## 5) Installer OpenClaw
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
source ~/.bashrc
```

Lorsque l’invite &quot;How do you want to hatch your bot?&quot; s’affiche, sélectionnez **&quot;Do this later&quot;**.

> Remarque : Si vous rencontrez des problèmes de compilation native sur ARM, commencez par les paquets système (par exemple `sudo apt install -y build-essential`) avant de vous tourner vers Homebrew.

<div id="6-configure-gateway-loopback-token-auth-and-enable-tailscale-serve">
  ## 6) Configurer Gateway (loopback + authentification par jeton) et activer Tailscale Serve
</div>

Utilisez l’authentification par jeton comme mode par défaut. Elle est prévisible et vous évite d’avoir à activer des options « insecure auth » dans la Control UI.

```bash
# Keep the Gateway private on the VM
openclaw config set gateway.bind loopback

# Require auth for the Gateway + Control UI
openclaw config set gateway.auth.mode token
openclaw doctor --generate-gateway-token

# Exposer via Tailscale Serve (HTTPS + accès tailnet)
openclaw config set gateway.tailscale.mode serve
openclaw config set gateway.trustedProxies '["127.0.0.1"]'

systemctl --user restart openclaw-gateway
```

<div id="7-verify">
  ## 7) Vérifier
</div>

```bash
# Check version
openclaw --version

# Vérifier le statut du daemon
systemctl --user status openclaw-gateway

# Check Tailscale Serve
tailscale serve status

# Test local response
curl http://localhost:18789
```

<div id="8-lock-down-vcn-security">
  ## 8) Verrouiller la sécurité du VCN
</div>

Maintenant que tout fonctionne, verrouillez le VCN pour bloquer tout le trafic, à l’exception de Tailscale. Le Virtual Cloud Network d’OCI agit comme un pare-feu en bordure de réseau : le trafic est bloqué avant d’atteindre votre instance.

1. Allez dans **Networking → Virtual Cloud Networks** dans la console OCI
2. Cliquez sur votre VCN → **Security Lists** → Default Security List
3. **Supprimez** toutes les règles d’ingress, sauf :
   * `0.0.0.0/0 UDP 41641` (Tailscale)
4. Conservez les règles d’egress par défaut (autoriser tout le trafic sortant)

Cela bloque SSH sur le port 22, HTTP, HTTPS et tout le reste au niveau de la bordure du réseau. À partir de maintenant, vous ne pouvez plus vous connecter que via Tailscale.

***

<div id="access-the-control-ui">
  ## Accéder à Control UI
</div>

Depuis n&#39;importe quel appareil connecté à votre réseau Tailscale :

```
https://openclaw.<tailnet-name>.ts.net/
```

Remplacez `<tailnet-name>` par le nom de votre tailnet (visible dans `tailscale status`).

Aucun tunnel SSH n’est nécessaire. Tailscale fournit :

* Chiffrement HTTPS (certificats automatiques)
* Authentification via l’identité Tailscale
* Accès depuis n’importe quel appareil de votre tailnet (ordinateur portable, téléphone, etc.)

***

<div id="security-vcn-tailscale-recommended-baseline">
  ## Sécurité : VCN + Tailscale (configuration de base recommandée)
</div>

Avec le VCN verrouillé (seul l’UDP 41641 est ouvert) et le Gateway lié à l’interface loopback, vous obtenez une défense en profondeur solide : le trafic public est bloqué au niveau du périmètre réseau, et l’accès administrateur se fait via votre tailnet.

Cette configuration supprime souvent la nécessité de règles de pare-feu supplémentaires au niveau de l’hôte uniquement pour empêcher les attaques par force brute SSH à l’échelle d’Internet — mais vous devez tout de même maintenir le système d’exploitation à jour, exécuter `openclaw security audit` et vérifier que vous n’avez pas, par inadvertance, de services à l’écoute sur des interfaces publiques.

<div id="whats-already-protected">
  ### Ce qui est déjà protégé
</div>

| Étape traditionnelle | Nécessaire ? | Pourquoi |
|----------------------|-------------|----------|
| Pare-feu UFW | Non | Le VCN bloque le trafic avant qu’il n’atteigne l’instance |
| fail2ban | Non | Pas d’attaques par force brute si le port 22 est bloqué au niveau du VCN |
| Durcissement de sshd | Non | Tailscale SSH n’utilise pas sshd |
| Désactivation de la connexion root | Non | Tailscale utilise l’identité Tailscale, pas les utilisateurs système |
| Authentification SSH par clé uniquement | Non | Tailscale authentifie via votre tailnet |
| Durcissement IPv6 | Généralement non | Dépend de la configuration de votre VCN/sous-réseau ; vérifiez ce qui est réellement assigné/exposé |

<div id="still-recommended">
  ### Toujours recommandé
</div>

* **Permissions des identifiants :** `chmod 700 ~/.openclaw`
* **Audit de sécurité :** `openclaw security audit`
* **Mises à jour du système :** exécutez régulièrement `sudo apt update && sudo apt upgrade`
* **Surveillance de Tailscale :** vérifiez les appareils dans la [console d’administration Tailscale](https://login.tailscale.com/admin)

<div id="verify-security-posture">
  ### Vérifier la posture de sécurité
</div>

```bash
# Confirmer qu'aucun port public n'est à l'écoute
sudo ss -tlnp | grep -v '127.0.0.1\|::1'

# Vérifier que Tailscale SSH est actif
tailscale status | grep -q 'offers: ssh' && echo "Tailscale SSH active"

# Optionnel : désactiver complètement sshd
sudo systemctl disable --now ssh
```

***

<div id="fallback-ssh-tunnel">
  ## Solution de repli : tunnel SSH
</div>

Si Tailscale Serve ne fonctionne pas, utilisez un tunnel SSH :

```bash
# Depuis votre machine locale (via Tailscale)
ssh -L 18789:127.0.0.1:18789 ubuntu@openclaw
```

Puis ouvrez `http://localhost:18789`.

***

<div id="troubleshooting">
  ## Résolution des problèmes
</div>

<div id="instance-creation-fails-out-of-capacity">
  ### Échec de la création de l’instance (« Out of capacity »)
</div>

Les instances ARM de l’offre gratuite sont populaires. Essayez :

* Un autre domaine de disponibilité
* De réessayer en heures creuses (tôt le matin)
* D’utiliser le filtre « Always Free » lors de la sélection de la forme

<div id="tailscale-wont-connect">
  ### Tailscale n&#39;arrive pas à se connecter
</div>

```bash
# Vérifier l'état
sudo tailscale status

# Se ré-authentifier
sudo tailscale up --ssh --hostname=openclaw --reset
```

<div id="gateway-wont-start">
  ### Gateway ne démarre pas
</div>

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl --user -u openclaw-gateway -n 50
```

<div id="cant-reach-control-ui">
  ### Impossible d&#39;accéder à Control UI
</div>

```bash
# Vérifier que Tailscale Serve est en cours d'exécution
tailscale serve status

# Check gateway is listening
curl http://localhost:18789

# Restart if needed
systemctl --user restart openclaw-gateway
```

<div id="arm-binary-issues">
  ### Problèmes avec les binaires ARM
</div>

Certains outils peuvent ne pas proposer de builds ARM. Vérifiez :

```bash
uname -m  # Devrait afficher aarch64
```

La plupart des packages npm fonctionnent sans problème. Pour les binaires, recherchez des versions `linux-arm64` ou `aarch64`.

***

<div id="persistence">
  ## Persistance
</div>

L’ensemble de l’état est stocké dans :

* `~/.openclaw/` — configuration, identifiants, données de session
* `~/.openclaw/workspace/` — espace de travail (SOUL.md, mémoire, artefacts)

Effectuez des sauvegardes régulières :

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

***

<div id="see-also">
  ## Voir aussi
</div>

* [Accès distant au Gateway](/fr/gateway/remote) — autres modes d&#39;accès à distance
* [Intégration Tailscale](/fr/gateway/tailscale) — documentation complète de Tailscale
* [Configuration du Gateway](/fr/gateway/configuration) — toutes les options de configuration
* [Guide DigitalOcean](/fr/platforms/digitalocean) — si vous voulez une solution payante avec une inscription plus simple
* [Guide Hetzner](/fr/platforms/hetzner) — alternative basée sur Docker