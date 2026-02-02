---
title: Gcp
summary: "Exécuter OpenClaw Gateway 24/7 sur une VM GCP Compute Engine (Docker) avec état persistant"
read_when:
  - Vous souhaitez qu'OpenClaw s'exécute 24/7 sur GCP
  - Vous souhaitez une Gateway de niveau production, toujours active, sur votre propre VM
  - Vous souhaitez un contrôle total sur la persistance, les binaires et le comportement au redémarrage
---

<div id="openclaw-on-gcp-compute-engine-docker-production-vps-guide">
  # OpenClaw sur GCP Compute Engine (Docker, guide VPS pour la production)
</div>

<div id="goal">
  ## Objectif
</div>

Exécuter un Gateway OpenClaw persistant sur une VM GCP Compute Engine avec Docker, avec un état durable, des binaires préintégrés et un comportement de redémarrage sûr.

Si vous voulez « OpenClaw 24/7 pour ~5–12 $/mois », c’est une configuration fiable sur Google Cloud.
Le tarif varie selon le type de machine et la région ; choisissez la plus petite VM qui couvre votre charge de travail et augmentez la taille si vous rencontrez des erreurs OOM (mémoire insuffisante).

<div id="what-are-we-doing-simple-terms">
  ## Que faisons-nous (en termes simples) ?
</div>

* Créer un projet GCP et activer la facturation
* Créer une VM Compute Engine
* Installer Docker (environnement d’exécution applicatif isolé)
* Démarrer le Gateway OpenClaw dans Docker
* Rendre persistants `~/.openclaw` + `~/.openclaw/workspace` sur l’hôte (survit aux redémarrages/reconstructions)
* Accéder au Control UI depuis votre ordinateur portable via un tunnel SSH

Le Gateway est accessible via :

* Transfert de port SSH depuis votre ordinateur portable
* Exposition directe du port si vous gérez vous-même le pare-feu et les jetons

Ce guide utilise Debian sur GCP Compute Engine.
Ubuntu fonctionne également ; adaptez les paquets en conséquence.
Pour le flux Docker générique, voir [Docker](/fr/install/docker).

***

<div id="quick-path-experienced-operators">
  ## Parcours rapide (opérateurs expérimentés)
</div>

1. Créer un projet GCP + activer l’API Compute Engine
2. Créer une VM Compute Engine (e2-small, Debian 12, 20 Go)
3. Se connecter en SSH à la VM
4. Installer Docker
5. Cloner le dépôt OpenClaw
6. Créer des répertoires persistants sur l’hôte
7. Configurer `.env` et `docker-compose.yml`
8. Générer les binaires nécessaires, construire et lancer

***

<div id="what-you-need">
  ## Ce dont vous avez besoin
</div>

* Compte GCP (offre gratuite éligible à e2-micro)
* CLI gcloud installée (ou utilisez Cloud Console)
* Accès SSH depuis votre ordinateur portable
* Aisance minimale avec SSH et le copier-coller
* ~20-30 minutes
* Docker et Docker Compose
* Identifiants d’authentification pour les modèles
* Identifiants facultatifs de fournisseurs
  * QR code WhatsApp
  * Jeton de bot Telegram
  * OAuth Gmail

***

<div id="1-install-gcloud-cli-or-use-console">
  ## 1) Installer la CLI gcloud (ou utiliser la Console)
</div>

**Option A : CLI gcloud** (recommandée pour l&#39;automatisation)

Installez-la depuis https://cloud.google.com/sdk/docs/install

Initialisez-la et authentifiez-vous :

```bash
gcloud init
gcloud auth login
```

**Option B : Cloud Console**

Toutes les étapes peuvent être effectuées via l’interface Web à l’adresse https://console.cloud.google.com

***

<div id="2-create-a-gcp-project">
  ## 2) Créer un projet GCP
</div>

**CLI :**

```bash
gcloud projects create my-openclaw-project --name="OpenClaw Gateway"
gcloud config set project my-openclaw-project
```

Activez la facturation à l’adresse https://console.cloud.google.com/billing (obligatoire pour Compute Engine).

Activez l’API Compute Engine :

```bash
gcloud services enable compute.googleapis.com
```

**Console :**

1. Accédez à IAM &amp; Admin &gt; Create Project
2. Donnez-lui un nom et créez-le
3. Activez la facturation pour le projet
4. Accédez à APIs &amp; Services &gt; Enable APIs &gt; recherchez « Compute Engine API » &gt; Enable

***

<div id="3-create-the-vm">
  ## 3) Créer la VM
</div>

**Types de machines :**

| Type     | Caractéristiques           | Coût                        | Remarques                      |
| -------- | -------------------------- | --------------------------- | ------------------------------ |
| e2-small | 2 vCPU, 2GB RAM            | ~12 $/mois                  | Recommandé                     |
| e2-micro | 2 vCPU (partagés), 1GB RAM | Éligible à l’offre gratuite | Risque d’OOM sous forte charge |

**CLI :**

```bash
gcloud compute instances create openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small \
  --boot-disk-size=20GB \
  --image-family=debian-12 \
  --image-project=debian-cloud
```

**Console :**

1. Accédez à Compute Engine &gt; Instances de VM &gt; Créer une instance
2. Nom : `openclaw-gateway`
3. Région : `us-central1`, Zone : `us-central1-a`
4. Type de machine : `e2-small`
5. Disque de démarrage : Debian 12, 20 Go
6. Créer

***

<div id="4-ssh-into-the-vm">
  ## 4) Se connecter en SSH à la VM
</div>

**CLI :**

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

**Console :**

Cliquez sur le bouton « SSH » à côté de votre VM dans le tableau de bord Compute Engine.

Remarque : la propagation de la clé SSH peut prendre 1 à 2 minutes après la création de la VM. En cas de refus de connexion, patientez puis réessayez.

***

<div id="5-install-docker-on-the-vm">
  ## 5) Installez Docker (sur la VM)
</div>

```bash
sudo apt-get update
sudo apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
```

Déconnectez-vous puis reconnectez-vous afin que la modification de groupe soit prise en compte :

```bash
exit
```

Puis reconnectez-vous en SSH :

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

Vérifiez :

```bash
docker --version
docker compose version
```

***

<div id="6-clone-the-openclaw-repository">
  ## 6) Cloner le dépôt OpenClaw
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

Ce guide suppose que vous allez construire une image personnalisée pour garantir la persistance des binaires.

***

<div id="7-create-persistent-host-directories">
  ## 7) Créer des répertoires persistants sur l’hôte
</div>

Les conteneurs Docker sont éphémères.
Tout état persistant doit résider sur l’hôte.

```bash
mkdir -p ~/.openclaw
mkdir -p ~/.openclaw/workspace
```

***

<div id="8-configure-environment-variables">
  ## 8) Configurer les variables d&#39;environnement
</div>

Créez un fichier `.env` à la racine du dépôt.

```bash
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/home/$USER/.openclaw
OPENCLAW_WORKSPACE_DIR=/home/$USER/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

Générez des secrets forts :

```bash
openssl rand -hex 32
```

**Ne commitez pas ce fichier.**

***

<div id="9-docker-compose-configuration">
  ## 9) Configuration de Docker Compose
</div>

Créez ou mettez à jour `docker-compose.yml`.

```yaml
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE}
    build: .
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - HOME=/home/node
      - NODE_ENV=production
      - TERM=xterm-256color
      - OPENCLAW_GATEWAY_BIND=${OPENCLAW_GATEWAY_BIND}
      - OPENCLAW_GATEWAY_PORT=${OPENCLAW_GATEWAY_PORT}
      - OPENCLAW_GATEWAY_TOKEN=${OPENCLAW_GATEWAY_TOKEN}
      - GOG_KEYRING_PASSWORD=${GOG_KEYRING_PASSWORD}
      - XDG_CONFIG_HOME=${XDG_CONFIG_HOME}
      - PATH=/home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      # Recommended: keep the Gateway loopback-only on the VM; access via SSH tunnel.
      # To expose it publicly, remove the `127.0.0.1:` prefix and firewall accordingly.
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"

      # Optionnel : uniquement si vous exécutez des nœuds iOS/Android sur cette VM et avez besoin d'un hôte Canvas.
      # Si vous l'exposez publiquement, consultez /gateway/security et configurez le pare-feu en conséquence.
      # - "18793:18793"
    command:
      [
        "node",
        "dist/index.js",
        "gateway",
        "--bind",
        "${OPENCLAW_GATEWAY_BIND}",
        "--port",
        "${OPENCLAW_GATEWAY_PORT}"
      ]
```

***

<div id="10-bake-required-binaries-into-the-image-critical">
  ## 10) Intégrer les binaires requis dans l’image (critique)
</div>

Installer des binaires à l’intérieur d’un conteneur en cours d’exécution est un piège.
Tout ce qui est installé à l’exécution sera perdu au redémarrage.

Tous les binaires externes requis par les compétences doivent être installés pendant la construction de l’image.

Les exemples ci-dessous ne montrent que trois binaires courants :

* `gog` pour l’accès à Gmail
* `goplaces` pour Google Places
* `wacli` pour WhatsApp

Ce sont des exemples, pas une liste complète.
Vous pouvez installer autant de binaires que nécessaire en suivant le même modèle.

Si vous ajoutez plus tard de nouvelles compétences qui dépendent de binaires supplémentaires, vous devez :

1. Mettre à jour le Dockerfile
2. Reconstruire l’image
3. Redémarrer les conteneurs

**Exemple de Dockerfile**

```dockerfile
FROM node:22-bookworm

RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# Example binary 1: Gmail CLI
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# Example binary 2: Google Places CLI
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# Example binary 3: WhatsApp CLI
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

# Ajoutez d'autres binaires ci-dessous en suivant le même modèle

WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN corepack enable
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

***

<div id="11-build-and-launch">
  ## 11) Build et lancement
</div>

```bash
docker compose build
docker compose up -d openclaw-gateway
```

Vérifiez les binaires :

```bash
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli
```

Sortie attendue :

```
/usr/local/bin/gog
/usr/local/bin/goplaces
/usr/local/bin/wacli
```

***

<div id="12-verify-gateway">
  ## 12) Vérifier Gateway
</div>

```bash
docker compose logs -f openclaw-gateway
```

Succès :

```
[gateway] listening on ws://0.0.0.0:18789
```

***

<div id="13-access-from-your-laptop">
  ## 13) Accès depuis votre machine locale
</div>

Créez un tunnel SSH pour rediriger le port du Gateway :

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a -- -L 18789:127.0.0.1:18789
```

Ouvrez l&#39;adresse suivante dans votre navigateur :

`http://127.0.0.1:18789/`

Collez votre jeton Gateway.

***

<div id="what-persists-where-source-of-truth">
  ## Ce qui persiste où (source de vérité)
</div>

OpenClaw s’exécute dans Docker, mais Docker n’est pas la source de vérité.
Tout état de longue durée doit survivre aux redémarrages, reconstructions et redémarrages de la machine.

| Composant | Emplacement | Mécanisme de persistance | Notes |
|---|---|---|---|
| Configuration du Gateway | `/home/node/.openclaw/` | Montage de volume hôte | Inclut `openclaw.json`, jetons |
| Profils d’authentification de modèles | `/home/node/.openclaw/` | Montage de volume hôte | Jetons OAuth, clés API |
| Configurations de compétences | `/home/node/.openclaw/skills/` | Montage de volume hôte | État au niveau des compétences |
| Espace de travail de l’agent | `/home/node/.openclaw/workspace/` | Montage de volume hôte | Code et artefacts de l’agent |
| Session WhatsApp | `/home/node/.openclaw/` | Montage de volume hôte | Préserve la connexion par QR |
| Trousseau Gmail | `/home/node/.openclaw/` | Volume hôte + mot de passe | Nécessite `GOG_KEYRING_PASSWORD` |
| Binaires externes | `/usr/local/bin/` | Image Docker | Doivent être intégrés lors de la création de l’image |
| Environnement d’exécution Node | Système de fichiers du conteneur | Image Docker | Reconstruit à chaque build d’image |
| Paquets système | Système de fichiers du conteneur | Image Docker | Ne pas installer au moment de l’exécution |
| Conteneur Docker | Éphémère | Redémarrable | Peut être détruit sans risque |

***

<div id="updates">
  ## Mises à jour
</div>

Pour mettre à jour OpenClaw sur la VM :

```bash
cd ~/openclaw
git pull
docker compose build
docker compose up -d
```

***

<div id="troubleshooting">
  ## Dépannage
</div>

**Connexion SSH refusée**

La propagation de la clé SSH peut prendre 1 à 2 minutes après la création de la VM. Attendez, puis réessayez.

**Problèmes avec OS Login**

Vérifiez votre profil OS Login :

```bash
gcloud compute os-login describe-profile
```

Assurez-vous que votre compte dispose des autorisations IAM requises (`Compute OS Login` ou `Compute OS Admin Login`).

**Mémoire insuffisante (OOM)**

Si vous utilisez `e2-micro` et que vous êtes confronté à des erreurs de mémoire insuffisante (OOM), mettez à niveau vers `e2-small` ou `e2-medium` :

```bash
# Stop the VM first
gcloud compute instances stop openclaw-gateway --zone=us-central1-a

# Changez le type de machine
gcloud compute instances set-machine-type openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small

# Start the VM
gcloud compute instances start openclaw-gateway --zone=us-central1-a
```

***

<div id="service-accounts-security-best-practice">
  ## Comptes de service (bonne pratique de sécurité)
</div>

Pour un usage personnel, votre compte utilisateur par défaut convient.

Pour l’automatisation ou les pipelines CI/CD, créez un compte de service dédié avec des autorisations minimales :

1. Créez un compte de service :
   ```bash
   gcloud iam service-accounts create openclaw-deploy \
     --display-name="OpenClaw Deployment"
   ```

2. Attribuez le rôle Compute Instance Admin (ou un rôle personnalisé plus restreint) :
   ```bash
   gcloud projects add-iam-policy-binding my-openclaw-project \
     --member="serviceAccount:openclaw-deploy@my-openclaw-project.iam.gserviceaccount.com" \
     --role="roles/compute.instanceAdmin.v1"
   ```

Évitez d’utiliser le rôle Owner pour l’automatisation. Appliquez le principe du moindre privilège.

Consultez https://cloud.google.com/iam/docs/understanding-roles pour plus de détails sur les rôles IAM.

***

<div id="next-steps">
  ## Prochaines étapes
</div>

* Configurer les canaux de messagerie : [Channels](/fr/channels)
* Associer les appareils locaux en tant que nœuds : [Nodes](/fr/nodes)
* Configurer Gateway : [Gateway configuration](/fr/gateway/configuration)