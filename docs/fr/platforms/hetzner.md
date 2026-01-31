---
title: Hetzner
summary: "Exécuter le Gateway OpenClaw 24h/24 et 7j/7 sur un VPS Hetzner bon marché (Docker), avec un état persistant et des binaires intégrés"
read_when:
  - Vous voulez exécuter OpenClaw 24h/24 et 7j/7 sur un VPS dans le cloud (pas sur votre ordinateur portable)
  - Vous voulez un Gateway de niveau production, toujours actif, sur votre propre VPS
  - Vous voulez un contrôle total sur la persistance, les binaires et le comportement de redémarrage
  - Vous exécutez OpenClaw dans Docker sur Hetzner ou un fournisseur similaire
---

<div id="openclaw-on-hetzner-docker-production-vps-guide">
  # OpenClaw sur Hetzner (Docker, guide de mise en production sur VPS)
</div>

<div id="goal">
  ## Objectif
</div>

Exécuter en continu un Gateway OpenClaw sur un VPS Hetzner avec Docker, avec un état persistant, des binaires intégrés et un comportement de redémarrage sûr.

Si vous voulez « OpenClaw 24/7 pour environ 5 $ », c’est la configuration fiable la plus simple.
Les tarifs Hetzner évoluent ; choisissez le plus petit VPS Debian/Ubuntu et montez en gamme si vous rencontrez des OOM (Out Of Memory).

<div id="what-are-we-doing-simple-terms">
  ## Qu’allons-nous faire, en termes simples ?
</div>

* Louer un petit serveur Linux (VPS Hetzner)
* Installer Docker (runtime d’application isolé)
* Démarrer le Gateway OpenClaw dans Docker
* Rendre persistants `~/.openclaw` + `~/.openclaw/workspace` sur l’hôte (survivent aux redémarrages/reconstructions)
* Accéder au Control UI depuis votre ordinateur portable via un tunnel SSH

Le Gateway est accessible via :

* Redirection de port SSH depuis votre ordinateur portable
* Exposition directe des ports si vous gérez vous‑même le pare-feu et les jetons

Ce guide suppose Ubuntu ou Debian sur Hetzner.\
Si vous utilisez un autre VPS Linux, adaptez les paquets en conséquence.
Pour la procédure Docker générique, voir [Docker](/fr/install/docker).

***

<div id="quick-path-experienced-operators">
  ## Parcours rapide (opérateurs expérimentés)
</div>

1. Provisionner un VPS Hetzner
2. Installer Docker
3. Cloner le dépôt OpenClaw
4. Créer des répertoires persistants sur l’hôte
5. Configurer `.env` et `docker-compose.yml`
6. Intégrer les binaires nécessaires dans l’image
7. `docker compose up -d`
8. Vérifier la persistance des données et l’accès au Gateway

***

<div id="what-you-need">
  ## Ce dont vous avez besoin
</div>

* VPS Hetzner avec accès root
* Accès SSH depuis votre ordinateur portable
* Être à l’aise avec SSH et le copier-coller de base
* Environ 20 minutes
* Docker et Docker Compose
* Identifiants d’authentification pour les modèles
* Identifiants de fournisseurs facultatifs
  * QR WhatsApp
  * Jeton de bot Telegram
  * OAuth Gmail

***

<div id="1-provision-the-vps">
  ## 1) Approvisionner le VPS
</div>

Créez un VPS Ubuntu ou Debian chez Hetzner.

Connectez-vous en tant que root :

```bash
ssh root@YOUR_VPS_IP
```

Ce guide suppose que le VPS est persistant (stateful).
Ne le traitez pas comme une infrastructure jetable.

***

<div id="2-install-docker-on-the-vps">
  ## 2) Installer Docker (sur votre VPS)
</div>

```bash
apt-get update
apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sh
```

Vérifiez :

```bash
docker --version
docker compose version
```

***

<div id="3-clone-the-openclaw-repository">
  ## 3) Cloner le dépôt OpenClaw
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

Ce guide part du principe que vous allez créer une image personnalisée afin de garantir la persistance des binaires.

***

<div id="4-create-persistent-host-directories">
  ## 4) Créer des répertoires persistants sur l&#39;hôte
</div>

Les conteneurs Docker sont éphémères.
Tout état persistant doit être conservé sur l&#39;hôte.

```bash
mkdir -p /root/.openclaw
mkdir -p /root/.openclaw/workspace

# Définir la propriété à l'utilisateur du conteneur (uid 1000) :
chown -R 1000:1000 /root/.openclaw
chown -R 1000:1000 /root/.openclaw/workspace
```

***

<div id="5-configure-environment-variables">
  ## 5) Configurer les variables d&#39;environnement
</div>

Créez le fichier `.env` à la racine du dépôt.

```bash
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/root/.openclaw
OPENCLAW_WORKSPACE_DIR=/root/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

Générez des secrets forts :

```bash
openssl rand -hex 32
```

**Ne validez pas ce fichier dans le dépôt.**

***

<div id="6-docker-compose-configuration">
  ## 6) Configuration de Docker Compose
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
      # Recommended: keep the Gateway loopback-only on the VPS; access via SSH tunnel.
      # To expose it publicly, remove the `127.0.0.1:` prefix and firewall accordingly.
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"

      # Optionnel : uniquement si vous exécutez des nœuds iOS/Android sur ce VPS et avez besoin d'un hôte Canvas.
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

<div id="7-bake-required-binaries-into-the-image-critical">
  ## 7) Intégrer les binaires requis dans l’image (critique)
</div>

Installer des binaires à l’intérieur d’un conteneur en cours d’exécution est un piège.
Tout ce qui est installé au moment de l’exécution sera perdu au redémarrage.

Tous les binaires externes requis par les compétences doivent être installés lors de la construction de l’image.

Les exemples ci-dessous ne montrent que trois binaires courants :

* `gog` pour l’accès à Gmail
* `goplaces` pour Google Places
* `wacli` pour WhatsApp

Ce sont des exemples, pas une liste complète.
Vous pouvez installer autant de binaires que nécessaire en suivant le même modèle.

Si vous ajoutez plus tard de nouvelles compétences qui dépendent de binaires supplémentaires, vous devez :

1. Mettre à jour le Dockerfile
2. Recréer l’image
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

# Ajoutez d'autres binaires ci-dessous selon le même modèle

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

<div id="8-build-and-launch">
  ## 8) Build et démarrage
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

<div id="9-verify-gateway">
  ## 9) Vérifier le Gateway
</div>

```bash
docker compose logs -f openclaw-gateway
```

Succès :

```
[gateway] listening on ws://0.0.0.0:18789
```

Depuis votre ordinateur portable :

```bash
ssh -N -L 18789:127.0.0.1:18789 root@YOUR_VPS_IP
```

Ouvrez :

`http://127.0.0.1:18789/`

Collez votre jeton Gateway.

***

<div id="what-persists-where-source-of-truth">
  ## Ce qui persiste où (source de vérité)
</div>

OpenClaw s&#39;exécute dans Docker, mais Docker n&#39;est pas la source de vérité.
Tout état persistant doit survivre aux redémarrages, reconstructions et reboots.

| Composant | Emplacement | Mécanisme de persistance | Notes |
|---|---|---|---|
| Configuration du Gateway | `/home/node/.openclaw/` | Montage de volume hôte | Inclut `openclaw.json`, jetons |
| Profils d&#39;authentification des modèles | `/home/node/.openclaw/` | Montage de volume hôte | Jetons OAuth, clés API |
| Configurations des compétences | `/home/node/.openclaw/skills/` | Montage de volume hôte | État au niveau des compétences |
| Espace de travail de l&#39;agent | `/home/node/.openclaw/workspace/` | Montage de volume hôte | Code et artefacts de l&#39;agent |
| Session WhatsApp | `/home/node/.openclaw/` | Montage de volume hôte | Préserve la connexion via QR |
| Trousseau Gmail | `/home/node/.openclaw/` | Volume hôte + mot de passe | Nécessite `GOG_KEYRING_PASSWORD` |
| Binaires externes | `/usr/local/bin/` | Image Docker | Doivent être intégrés au moment du build |
| Environnement d&#39;exécution Node.js | Système de fichiers du conteneur | Image Docker | Reconstruit à chaque build d&#39;image |
| Paquets du système d&#39;exploitation | Système de fichiers du conteneur | Image Docker | Ne pas installer à l&#39;exécution |
| Conteneur Docker | Éphémère | Redémarrable | Peut être détruit sans risque |