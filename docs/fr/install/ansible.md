---
title: Ansible
summary: "Installation automatisée et sécurisée d'OpenClaw avec Ansible, le VPN Tailscale et une isolation par pare-feu"
read_when:
  - Vous voulez un déploiement automatisé de serveurs avec renforcement de la sécurité
  - Vous avez besoin d'une configuration isolée par pare-feu avec accès VPN
  - Vous déployez sur des serveurs Debian/Ubuntu distants
---

<div id="ansible-installation">
  # Installation avec Ansible
</div>

La méthode recommandée pour déployer OpenClaw sur des serveurs de production consiste à utiliser **[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** — un installateur automatisé dont l’architecture privilégie la sécurité.

<div id="quick-start">
  ## Démarrage rapide
</div>

Installation en une seule commande :

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

> **📦 Guide complet : [github.com/openclaw/openclaw-ansible](https://github.com/openclaw/openclaw-ansible)**
>
> Le dépôt openclaw-ansible est la référence officielle pour le déploiement Ansible. Cette page est un aperçu rapide.

<div id="what-you-get">
  ## Ce que vous obtenez
</div>

* 🔒 **Sécurité axée sur le pare-feu** : UFW + isolation Docker (seuls SSH + Tailscale sont accessibles)
* 🔐 **VPN Tailscale** : accès distant sécurisé sans exposition publique des services
* 🐳 **Docker** : conteneurs sandbox isolés, liaisons limitées à localhost
* 🛡️ **Défense en profondeur** : architecture de sécurité en 4 couches
* 🚀 **Mise en place en une commande** : déploiement complet en quelques minutes
* 🔧 **Intégration systemd** : démarrage automatique au démarrage du système avec durcissement

<div id="requirements">
  ## Prérequis
</div>

* **OS** : Debian 11+ ou Ubuntu 20.04+
* **Accès** : privilèges root ou sudo
* **Réseau** : connexion Internet pour installer les paquets
* **Ansible** : 2.14+ (installé automatiquement par le script de démarrage rapide)

<div id="what-gets-installed">
  ## Ce qui est installé
</div>

Le playbook Ansible installe et configure :

1. **Tailscale** (VPN maillé pour un accès distant sécurisé)
2. **Pare-feu UFW** (ports SSH + Tailscale uniquement)
3. **Docker CE + Compose V2** (pour les sandboxes d’agents)
4. **Node.js 22.x + pnpm** (dépendances d’exécution)
5. **OpenClaw** (exécuté sur l’hôte, non conteneurisé)
6. **Service systemd** (démarrage automatique avec renforcement de la sécurité)

Remarque : Gateway s’exécute **directement sur l’hôte** (et non dans Docker), mais les sandboxes d’agents utilisent Docker pour l’isolation. Voir [Sandboxing](/fr/gateway/sandboxing) pour plus de détails.

<div id="post-install-setup">
  ## Configuration après l&#39;installation
</div>

Une fois l&#39;installation terminée, passez à l&#39;utilisateur openclaw :

```bash
sudo -i -u openclaw
```

Le script de post-installation vous guidera dans les étapes suivantes :

1. **Assistant de configuration initiale** : configurer les paramètres OpenClaw
2. **Connexion aux fournisseurs** : connecter WhatsApp/Telegram/Discord/Signal
3. **Test de Gateway** : vérifier l’installation
4. **Configuration de Tailscale** : se connecter à votre maillage VPN

<div id="quick-commands">
  ### Commandes rapides
</div>

```bash
# Vérifier le statut du service
sudo systemctl status openclaw

# Consulter les journaux en direct
sudo journalctl -u openclaw -f

# Redémarrer le Gateway
sudo systemctl restart openclaw

# Connexion au fournisseur (exécuter en tant qu'utilisateur openclaw)
sudo -i -u openclaw
openclaw channels login
```

<div id="security-architecture">
  ## Architecture de sécurité
</div>

<div id="4-layer-defense">
  ### Défense en 4 couches
</div>

1. **Pare-feu (UFW)** : seuls SSH (22) et Tailscale (41641/udp) sont exposés sur Internet
2. **VPN (Tailscale)** : Gateway accessible uniquement via le maillage VPN
3. **Isolation Docker** : la chaîne iptables DOCKER-USER empêche l’exposition de ports vers l’extérieur
4. **Renforcement systemd** : NoNewPrivileges, PrivateTmp, utilisateur non privilégié

<div id="verification">
  ### Vérification
</div>

Procédez au test de la surface d&#39;attaque externe :

```bash
nmap -p- YOUR_SERVER_IP
```

Ne doit afficher que le **port 22** (SSH) comme ouvert. Tous les autres services (Gateway, Docker) doivent être verrouillés.

<div id="docker-availability">
  ### Disponibilité de Docker
</div>

Docker est installé pour les **sandbox d&#39;agent** (exécution isolée des outils), et non pour exécuter le Gateway lui-même. Le Gateway se lie uniquement à localhost et est accessible via le VPN Tailscale.

Voir [Multi-Agent Sandbox &amp; Tools](/fr/multi-agent-sandbox-tools) pour la configuration des sandbox.

<div id="manual-installation">
  ## Installation manuelle
</div>

Si vous préférez un contrôle manuel plutôt que l’automatisation :

```bash
# 1. Install prerequisites
sudo apt update && sudo apt install -y ansible git

# 2. Clone repository
git clone https://github.com/openclaw/openclaw-ansible.git
cd openclaw-ansible

# 3. Install Ansible collections
ansible-galaxy collection install -r requirements.yml

# 4. Run playbook
./run-playbook.sh

# Ou exécuter directement (puis exécuter manuellement /tmp/openclaw-setup.sh ensuite)
# ansible-playbook playbook.yml --ask-become-pass
```

<div id="updating-openclaw">
  ## Mise à jour d&#39;OpenClaw
</div>

Le programme d&#39;installation Ansible configure OpenClaw pour des mises à jour manuelles. Consultez la section [Mise à jour](/fr/install/updating) pour connaître la procédure de mise à jour standard.

Pour relancer le playbook Ansible (par exemple pour des modifications de configuration) :

```bash
cd openclaw-ansible
./run-playbook.sh
```

Remarque : cette opération est idempotente et peut être exécutée plusieurs fois en toute sécurité.

<div id="troubleshooting">
  ## Dépannage
</div>

<div id="firewall-blocks-my-connection">
  ### Le pare-feu bloque ma connexion
</div>

Si vous avez perdu l&#39;accès :

* Assurez-vous d&#39;abord de pouvoir vous connecter via le VPN Tailscale
* L&#39;accès SSH (port 22) est toujours autorisé
* Le Gateway est, par conception, **uniquement** accessible via Tailscale

<div id="service-wont-start">
  ### Le service ne se lance pas
</div>

```bash
# Vérifier les logs
sudo journalctl -u openclaw -n 100

# Vérifier les permissions
sudo ls -la /opt/openclaw

# Tester le démarrage manuel
sudo -i -u openclaw
cd ~/openclaw
pnpm start
```

<div id="docker-sandbox-issues">
  ### Problèmes liés à la sandbox Docker
</div>

```bash
# Verify Docker is running
sudo systemctl status docker

# Check sandbox image
sudo docker images | grep openclaw-sandbox

# Construire l'image sandbox si manquante
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

<div id="provider-login-fails">
  ### La connexion au fournisseur échoue
</div>

Assurez-vous d’exécuter la commande en tant qu’utilisateur `openclaw` :

```bash
sudo -i -u openclaw
openclaw channels login
```

<div id="advanced-configuration">
  ## Configuration avancée
</div>

Pour des informations détaillées sur l’architecture de sécurité et la résolution des problèmes :

* [Architecture de sécurité](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
* [Détails techniques](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
* [Guide de résolution des problèmes](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)

<div id="related">
  ## Ressources associées
</div>

* [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — guide de déploiement complet
* [Docker](/fr/install/docker) — configuration du Gateway dans un conteneur
* [Sandboxing](/fr/gateway/sandboxing) — configuration du sandbox pour les agents
* [Multi-Agent Sandbox &amp; Tools](/fr/multi-agent-sandbox-tools) — isolation par agent