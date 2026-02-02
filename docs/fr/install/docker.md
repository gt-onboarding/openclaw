---
title: Docker
summary: "Configuration et mise en route optionnelles d’OpenClaw avec Docker"
read_when:
  - Vous voulez exécuter le Gateway dans un conteneur plutôt qu’avec une installation locale
  - Vous validez le workflow Docker
---

<div id="docker-optional">
  # Docker (optionnel)
</div>

Docker est **optionnel**. Utilisez-le uniquement si vous voulez exécuter le Gateway dans un conteneur ou valider le workflow Docker.

<div id="is-docker-right-for-me">
  ## Docker est-il adapté à votre cas ?
</div>

* **Oui** : vous voulez un environnement Gateway isolé et jetable, ou exécuter OpenClaw sur une machine hôte sans installations locales.
* **Non** : vous exécutez OpenClaw sur votre propre machine et vous voulez simplement la boucle de développement la plus rapide. Utilisez plutôt la procédure d’installation normale.
* **Remarque concernant la sandbox** : la sandbox des agents utilise aussi Docker, mais elle **n’exige pas** que le Gateway complet s’exécute dans Docker. Voir [Sandboxing](/fr/gateway/sandboxing).

Ce guide couvre :

* Gateway conteneurisé (OpenClaw complet dans Docker)
* Sandbox d’agent par session (Gateway sur l’hôte + outils d’agent isolés par Docker)

Détails sur la sandbox : [Sandboxing](/fr/gateway/sandboxing)

<div id="requirements">
  ## Prérequis
</div>

* Docker Desktop (ou Docker Engine) + Docker Compose v2
* Espace disque suffisant pour les images + logs

<div id="containerized-gateway-docker-compose">
  ## Gateway en conteneur (Docker Compose)
</div>

<div id="quick-start-recommended">
  ### Démarrage rapide (recommandé)
</div>

À partir de la racine du dépôt :

```bash
./docker-setup.sh
```

Ce script :

* construit l&#39;image du Gateway
* exécute l&#39;assistant d&#39;onboarding
* affiche des indications facultatives de configuration de fournisseur
* démarre le Gateway via Docker Compose
* génère un jeton du Gateway et l&#39;écrit dans `.env`

Variables d&#39;environnement optionnelles :

* `OPENCLAW_DOCKER_APT_PACKAGES` — installer des paquets apt supplémentaires pendant le build
* `OPENCLAW_EXTRA_MOUNTS` — ajouter des bind mounts supplémentaires depuis l&#39;hôte
* `OPENCLAW_HOME_VOLUME` — conserver `/home/node` dans un volume nommé

Une fois terminé :

* Ouvrez `http://127.0.0.1:18789/` dans votre navigateur.
* Collez le jeton dans le Control UI (Settings → token).

Il écrit la configuration et l&#39;espace de travail sur l&#39;hôte :

* `~/.openclaw/`
* `~/.openclaw/workspace`

Vous utilisez un VPS ? Voir [Hetzner (Docker VPS)](/fr/platforms/hetzner).

<div id="manual-flow-compose">
  ### Flux manuel (Docker Compose)
</div>

```bash
docker build -t openclaw:local -f Dockerfile .
docker compose run --rm openclaw-cli onboard
docker compose up -d openclaw-gateway
```

<div id="extra-mounts-optional">
  ### Points de montage supplémentaires (optionnel)
</div>

Si vous souhaitez monter des répertoires hôtes supplémentaires dans les conteneurs, définissez
`OPENCLAW_EXTRA_MOUNTS` avant d&#39;exécuter `docker-setup.sh`. Cette variable accepte une
liste de montages bind Docker, séparés par des virgules, et les applique à la fois à
`openclaw-gateway` et `openclaw-cli` en générant `docker-compose.extra.yml`.

Exemple :

```bash
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

Remarques :

* Les chemins doivent être partagés dans Docker Desktop sur macOS/Windows.
* Si vous modifiez `OPENCLAW_EXTRA_MOUNTS`, relancez `docker-setup.sh` pour régénérer
  le fichier Compose supplémentaire.
* `docker-compose.extra.yml` est généré automatiquement. Ne le modifiez pas manuellement.

<div id="persist-the-entire-container-home-optional">
  ### Rendre persistant l’intégralité du home du conteneur (optionnel)
</div>

Si vous voulez que `/home/node` persiste entre les recréations de conteneurs, définissez un volume nommé via `OPENCLAW_HOME_VOLUME`. Cela crée un volume Docker et le monte sur `/home/node`, tout en conservant les montages standards de configuration/espace de travail en bind. Utilisez ici un volume nommé (et non un chemin bind) ; pour les montages bind, utilisez
`OPENCLAW_EXTRA_MOUNTS`.

Exemple :

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

Vous pouvez le combiner avec des montages supplémentaires :

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

Notes :

* Si vous modifiez `OPENCLAW_HOME_VOLUME`, relancez `docker-setup.sh` pour régénérer
  le fichier Compose supplémentaire.
* Le volume nommé persiste jusqu’à sa suppression avec `docker volume rm &lt;name&gt;`.

<div id="install-extra-apt-packages-optional">
  ### Installer des paquets apt supplémentaires (optionnel)
</div>

Si vous avez besoin de paquets système à l’intérieur de l’image (par exemple des outils de compilation ou des bibliothèques multimédia), définissez `OPENCLAW_DOCKER_APT_PACKAGES` avant d’exécuter `docker-setup.sh`.
Les paquets seront installés pendant le build de l’image, afin qu’ils persistent même si le conteneur est supprimé.

Exemple :

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="ffmpeg build-essential"
./docker-setup.sh
```

Remarques :

* Ce paramètre accepte une liste de noms de paquets apt séparés par des espaces.
* Si vous modifiez `OPENCLAW_DOCKER_APT_PACKAGES`, relancez `docker-setup.sh` pour reconstruire l’image.

<div id="faster-rebuilds-recommended">
  ### Reconstructions plus rapides (recommandé)
</div>

Pour accélérer les reconstructions, organisez votre Dockerfile de façon à ce que les couches de dépendances soient mises en cache.
Cela évite de relancer `pnpm install` sauf si les fichiers de verrouillage changent :

```dockerfile
FROM node:22-bookworm

# Installer Bun (requis pour les scripts de compilation)
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:${PATH}"

RUN corepack enable

WORKDIR /app

# Mettre en cache les dépendances sauf si les métadonnées du package changent
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

<div id="channel-setup-optional">
  ### Configuration des canaux (facultatif)
</div>

Utilisez le conteneur CLI pour configurer les canaux, puis redémarrez le Gateway si nécessaire.

WhatsApp (QR) :

```bash
docker compose run --rm openclaw-cli channels login
```

Telegram (token du bot) :

```bash
docker compose run --rm openclaw-cli channels add --channel telegram --token "<token>"
```

Discord (token du bot) :

```bash
docker compose run --rm openclaw-cli channels add --channel discord --token "<token>"
```

Documentation : [WhatsApp](/fr/channels/whatsapp), [Telegram](/fr/channels/telegram), [Discord](/fr/channels/discord)

<div id="health-check">
  ### Vérification de l’état de santé
</div>

```bash
docker compose exec openclaw-gateway node dist/index.js health --token "$OPENCLAW_GATEWAY_TOKEN"
```

<div id="e2e-smoke-test-docker">
  ### Smoke test E2E (Docker)
</div>

```bash
scripts/e2e/onboard-docker.sh
```

<div id="qr-import-smoke-test-docker">
  ### Test de bon fonctionnement de l’import via QR (Docker)
</div>

```bash
pnpm test:docker:qr
```

<div id="notes">
  ### Remarques
</div>

* Par défaut, le bind de Gateway est `lan` pour une utilisation en conteneur.
* Le conteneur Gateway est la source de vérité pour les sessions (`~/.openclaw/agents/&lt;agentId&gt;/sessions/`).

<div id="agent-sandbox-host-gateway-docker-tools">
  ## Sandbox d’agent (Gateway hôte + outils Docker)
</div>

Analyse détaillée : [Sandboxing](/fr/gateway/sandboxing)

<div id="what-it-does">
  ### Fonctionnement
</div>

Quand `agents.defaults.sandbox` est activé, les **sessions non principales** exécutent les outils dans un
conteneur Docker. Le Gateway reste sur votre hôte, mais l’exécution des outils est isolée :

* scope : `"agent"` par défaut (un conteneur + un espace de travail par agent)
* scope : `"session"` pour une isolation par session
* dossier d’espace de travail par portée monté sur `/workspace`
* accès optionnel à l’espace de travail de l’agent (`agents.defaults.sandbox.workspaceAccess`)
* stratégie d’autorisation/interdiction des outils (l’interdiction l’emporte)
* les médias entrants sont copiés dans l’espace de travail de la sandbox active (`media/inbound/*`) pour que les outils puissent les read (avec `workspaceAccess: "rw"`, cela se retrouve dans l’espace de travail de l’agent)

Avertissement : `scope: "shared"` désactive l’isolation entre sessions. Toutes les sessions partagent
un seul conteneur et un seul espace de travail.

<div id="per-agent-sandbox-profiles-multi-agent">
  ### Profils de sandbox par agent (multi-agent)
</div>

Si vous utilisez le routage multi-agent, chaque agent peut redéfinir les paramètres de sandbox et d’outils :
`agents.list[].sandbox` et `agents.list[].tools` (plus `agents.list[].tools.sandbox.tools`). Cela vous permet d’exécuter
des niveaux d’accès mixtes au sein d’un même Gateway :

* Accès complet (agent personnel)
* Outils en lecture seule + espace de travail en lecture seule (agent familial/professionnel)
* Aucun outil de système de fichiers ni shell (agent public)

Consultez [Multi-Agent Sandbox &amp; Tools](/fr/multi-agent-sandbox-tools) pour des exemples,
l’ordre de priorité et le dépannage.

<div id="default-behavior">
  ### Comportement par défaut
</div>

* Image : `openclaw-sandbox:bookworm-slim`
* Un conteneur par agent
* Accès à l’espace de travail de l’agent : `workspaceAccess: "none"` (par défaut) utilise `~/.openclaw/sandboxes`
  * `"ro"` conserve l’espace de travail de la sandbox à `/workspace` et monte l’espace de travail de l’agent en lecture seule à `/agent` (désactive `write`/`edit`/`apply_patch`)
  * `"rw"` monte l’espace de travail de l’agent en lecture/écriture à `/workspace`
* Nettoyage automatique : inactivité &gt; 24 h OU durée de vie &gt; 7 j
* Réseau : `none` par défaut (activation explicite requise si vous avez besoin de trafic sortant)
* Autorisations par défaut : `exec`, `process`, `read`, `write`, `edit`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* Interdictions par défaut : `browser`, `canvas`, `nodes`, `cron`, `discord`, `gateway`

<div id="enable-sandboxing">
  ### Activer le sandboxing
</div>

Si vous prévoyez d’installer des paquets dans `setupCommand`, notez les points suivants :

* La valeur par défaut de `docker.network` est `"none"` (aucune sortie réseau).
* `readOnlyRoot: true` empêche l’installation de paquets.
* `user` doit être root pour `apt-get` (omettre `user` ou définir `user: "0:0"`).
  OpenClaw recrée automatiquement les conteneurs lorsque `setupCommand` (ou la configuration Docker) change,
  sauf si le conteneur a été **utilisé récemment** (dans les ~5 dernières minutes). Les conteneurs « chauds »
  journalisent un avertissement avec la commande exacte `openclaw sandbox recreate ...`.

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent est la valeur par défaut)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"]
        },
        prune: {
          idleHours: 24, // 0 disables idle pruning
          maxAgeDays: 7  // 0 disables max-age pruning
        }
      }
    }
  },
  tools: {
    sandbox: {
      tools: {
        allow: ["exec", "process", "read", "write", "edit", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"]
      }
    }
  }
}
```

Les paramètres de durcissement se trouvent sous `agents.defaults.sandbox.docker` :
`network`, `user`, `pidsLimit`, `memory`, `memorySwap`, `cpus`, `ulimits`,
`seccompProfile`, `apparmorProfile`, `dns`, `extraHosts`.

Multi-agents : remplace `agents.defaults.sandbox.{docker,browser,prune}.*` pour chaque agent via `agents.list[].sandbox.{docker,browser,prune}.*`
(ces paramètres sont ignorés lorsque `agents.defaults.sandbox.scope` / `agents.list[].sandbox.scope` est défini sur `"shared"`).

<div id="build-the-default-sandbox-image">
  ### Construire l&#39;image de sandbox par défaut
</div>

```bash
scripts/sandbox-setup.sh
```

Cela construit l’image `openclaw-sandbox:bookworm-slim` à partir de `Dockerfile.sandbox`.

<div id="sandbox-common-image-optional">
  ### Image sandbox commune (optionnelle)
</div>

Si vous voulez une image sandbox avec des outils de build courants (Node, Go, Rust, etc.), générez l&#39;image commune :

```bash
scripts/sandbox-common-setup.sh
```

Cela construit l’image Docker `openclaw-sandbox-common:bookworm-slim`. Pour l’utiliser :

```json5
{
  agents: { defaults: { sandbox: { docker: { image: "openclaw-sandbox-common:bookworm-slim" } } } }
}
```

<div id="sandbox-browser-image">
  ### Image de navigateur pour la sandbox
</div>

Pour exécuter l’outil navigateur dans la sandbox, construisez l’image de navigateur :

```bash
scripts/sandbox-browser-setup.sh
```

Cette commande construit `openclaw-sandbox-browser:bookworm-slim` en utilisant
`Dockerfile.sandbox-browser`. Le conteneur exécute Chromium avec CDP activé et
un observateur noVNC optionnel (mode avec interface graphique via Xvfb).

Remarques :

* Le mode avec interface graphique (Xvfb) réduit les blocages anti-bot par rapport au mode sans interface graphique.
* Le mode sans interface graphique peut toujours être utilisé en définissant `agents.defaults.sandbox.browser.headless=true`.
* Aucun environnement de bureau complet (GNOME) n’est nécessaire ; Xvfb fournit l’affichage.

Utilisez la configuration suivante :

```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: { enabled: true }
      }
    }
  }
}
```

Image personnalisée du navigateur :

```json5
{
  agents: {
    defaults: {
      sandbox: { browser: { image: "my-openclaw-browser" } }
    }
  }
}
```

Lorsque cette option est activée, l’agent reçoit :

* une URL de contrôle de navigateur du sandbox (pour l’outil `browser`)
* une URL noVNC (si activé et headless=false)

N’oubliez pas : si vous utilisez une liste d’autorisation pour les outils, ajoutez `browser` (et retirez‑le de deny), sinon l’outil restera bloqué.
Les règles de purge (`agents.defaults.sandbox.prune`) s’appliquent également aux conteneurs de navigateur.

<div id="custom-sandbox-image">
  ### Image de sandbox personnalisée
</div>

Créez votre propre image et référencez-la dans la configuration :

```bash
docker build -t my-openclaw-sbx -f Dockerfile.sandbox .
```

```json5
{
  agents: {
    defaults: {
      sandbox: { docker: { image: "my-openclaw-sbx" } }
    }
  }
}
```

<div id="tool-policy-allowdeny">
  ### Politique des outils (allow/deny)
</div>

* `deny` a priorité sur `allow`.
* Si `allow` est vide : tous les outils (sauf ceux figurant dans `deny`) sont disponibles.
* Si `allow` n’est pas vide : seuls les outils mentionnés dans `allow` sont disponibles (à l’exception de ceux figurant dans `deny`).

<div id="pruning-strategy">
  ### Stratégie de purge
</div>

Deux paramètres :

* `prune.idleHours` : supprimer les conteneurs non utilisés depuis X heures (0 = désactivé)
* `prune.maxAgeDays` : supprimer les conteneurs âgés de plus de X jours (0 = désactivé)

Exemple :

* Conserver les sessions actives mais limiter leur durée de vie :
  `idleHours: 24`, `maxAgeDays: 7`
* Ne jamais purger :
  `idleHours: 0`, `maxAgeDays: 0`

<div id="security-notes">
  ### Notes de sécurité
</div>

* La barrière stricte s’applique uniquement aux **outils** (exec/read/write/edit/apply&#95;patch).
* Les outils réservés à l’hôte, comme browser/camera/canvas, sont bloqués par défaut.
* Autoriser `browser` dans la sandbox **compromet l’isolation** (le navigateur s’exécute sur l’hôte).

<div id="troubleshooting">
  ## Dépannage
</div>

* Image manquante : générez-la avec [`scripts/sandbox-setup.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/sandbox-setup.sh) ou définissez `agents.defaults.sandbox.docker.image`.
* Conteneur non démarré : il sera créé automatiquement à la demande, par session.
* Erreurs de permission dans le sandbox : définissez `docker.user` sur un UID:GID correspondant au propriétaire de votre
  espace de travail monté (ou exécutez un chown sur le dossier de l’espace de travail).
* Outils personnalisés introuvables : OpenClaw exécute les commandes avec `sh -lc` (shell de connexion), qui
  charge `/etc/profile` et peut réinitialiser PATH. Définissez `docker.env.PATH` pour ajouter en préfixe vos
  chemins d’outils personnalisés (par ex. `/custom/bin:/usr/local/share/npm-global/bin`), ou ajoutez
  un script sous `/etc/profile.d/` dans votre Dockerfile.