---
title: Fly.io
description: Déployer OpenClaw sur Fly.io
---

<div id="flyio-deployment">
  # Déploiement sur Fly.io
</div>

**Objectif :** faire tourner Gateway OpenClaw sur une machine [Fly.io](https://fly.io) avec stockage persistant, HTTPS automatique et accès à Discord et aux canaux.

<div id="what-you-need">
  ## Ce dont vous avez besoin
</div>

- [CLI flyctl](https://fly.io/docs/hands-on/install-flyctl/) installée
- Compte Fly.io (l’offre gratuite suffit)
- Authentification de modèle : clé API Anthropic (ou autres clés de fournisseurs)
- Identifiants de canal : jeton de bot Discord, jeton Telegram, etc.

<div id="beginner-quick-path">
  ## Parcours rapide pour débuter
</div>

1. Clonez le dépôt → personnalisez `fly.toml`
2. Créez l’application + le volume → définissez les secrets
3. Déployez avec `fly deploy`
4. Ouvrez une session SSH pour créer la configuration ou utilisez Control UI

<div id="1-create-the-fly-app">
  ## 1) Créer l’application Fly
</div>

```bash
# Cloner le dépôt
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# Créer une nouvelle application Fly (choisissez votre propre nom)
fly apps create my-openclaw

# Créer un volume persistant (1 Go suffit généralement)
fly volumes create openclaw_data --size 1 --region iad
```

**Conseil :** Choisissez une région proche de vous. Options courantes : `lhr` (Londres), `iad` (Virginie), `sjc` (San José).


<div id="2-configure-flytoml">
  ## 2) Configurer fly.toml
</div>

Modifiez `fly.toml` pour qu&#39;il corresponde au nom et aux besoins de votre application.

**Note de sécurité :** La configuration par défaut expose une URL publique. Pour un déploiement renforcé sans IP publique, consultez [Déploiement privé](#private-deployment-hardened) ou utilisez `fly.private.toml`.

```toml
app = "my-openclaw"  # Nom de votre application
primary_region = "iad"

[build]
  dockerfile = "Dockerfile"

[env]
  NODE_ENV = "production"
  OPENCLAW_PREFER_PNPM = "1"
  OPENCLAW_STATE_DIR = "/data"
  NODE_OPTIONS = "--max-old-space-size=1536"

[processes]
  app = "node dist/index.js gateway --allow-unconfigured --port 3000 --bind lan"

[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = false
  auto_start_machines = true
  min_machines_running = 1
  processes = ["app"]

[[vm]]
  size = "shared-cpu-2x"
  memory = "2048mb"

[mounts]
  source = "openclaw_data"
  destination = "/data"
```

**Paramètres clés :**

| Paramètre                      | Pourquoi                                                                                     |
| ------------------------------ | -------------------------------------------------------------------------------------------- |
| `--bind lan`                   | Lie l’écoute à `0.0.0.0` pour que le proxy Fly puisse atteindre le Gateway                   |
| `--allow-unconfigured`         | Démarre sans fichier de configuration (vous en créerez un ensuite)                           |
| `internal_port = 3000`         | Doit correspondre à `--port 3000` (ou `OPENCLAW_GATEWAY_PORT`) pour les contrôles d’état Fly |
| `memory = "2048mb"`            | 512 MB est insuffisant ; 2 Go recommandés                                                    |
| `OPENCLAW_STATE_DIR = "/data"` | Conserve l’état sur le volume                                                                |


<div id="3-set-secrets">
  ## 3) Configurer les secrets
</div>

```bash
# Requis : jeton Gateway (pour liaison non-loopback)
fly secrets set OPENCLAW_GATEWAY_TOKEN=$(openssl rand -hex 32)

# Model provider API keys
fly secrets set ANTHROPIC_API_KEY=sk-ant-...

# Optional: Other providers
fly secrets set OPENAI_API_KEY=sk-...
fly secrets set GOOGLE_API_KEY=...

# Channel tokens
fly secrets set DISCORD_BOT_TOKEN=MTQ...
```

**Remarques :**

* Les liaisons non-loopback (`--bind lan`) nécessitent `OPENCLAW_GATEWAY_TOKEN` pour des raisons de sécurité.
* Traitez ces jetons comme des mots de passe.
* **Préférez les variables d’environnement au fichier de configuration** pour toutes les clés d’API et tous les jetons. Cela permet de garder les secrets en dehors de `openclaw.json`, où ils pourraient être exposés ou consignés dans des journaux par erreur.


<div id="4-deploy">
  ## 4) Déployer
</div>

```bash
fly deploy
```

Le premier déploiement génère l&#39;image Docker (~2-3 minutes). Les déploiements suivants sont plus rapides.

Après le déploiement, vérifiez les points suivants :

```bash
fly status
fly logs
```

Vous devriez voir s’afficher :

```
[gateway] listening on ws://0.0.0.0:3000 (PID xxx)
[discord] logged in to discord as xxx
```


<div id="5-create-config-file">
  ## 5) Créer le fichier de configuration
</div>

Connectez-vous en SSH sur la machine pour créer un fichier de configuration adéquat :

```bash
fly ssh console
```

Créez le répertoire de configuration et le fichier :

```bash
mkdir -p /data
cat > /data/openclaw.json << 'EOF'
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-5",
        "fallbacks": ["anthropic/claude-sonnet-4-5", "openai/gpt-4o"]
      },
      "maxConcurrent": 4
    },
    "list": [
      {
        "id": "main",
        "default": true
      }
    ]
  },
  "auth": {
    "profiles": {
      "anthropic:default": { "mode": "token", "provider": "anthropic" },
      "openai:default": { "mode": "token", "provider": "openai" }
    }
  },
  "bindings": [
    {
      "agentId": "main",
      "match": { "channel": "discord" }
    }
  ],
  "channels": {
    "discord": {
      "enabled": true,
      "groupPolicy": "allowlist",
      "guilds": {
        "YOUR_GUILD_ID": {
          "channels": { "general": { "allow": true } },
          "requireMention": false
        }
      }
    }
  },
  "gateway": {
    "mode": "local",
    "bind": "auto"
  },
  "meta": {
    "lastTouchedVersion": "2026.1.29"
  }
}
EOF
```

**Remarque :** Avec `OPENCLAW_STATE_DIR=/data`, le chemin de configuration est `/data/openclaw.json`.

**Remarque :** Le jeton Discord peut provenir de l’une des sources suivantes :

* Variable d&#39;environnement : `DISCORD_BOT_TOKEN` (recommandée pour les secrets)
* Fichier de configuration : `channels.discord.token`

Si vous utilisez la variable d&#39;environnement, il n&#39;est pas nécessaire d&#39;ajouter le jeton dans la configuration. Le Gateway lit automatiquement `DISCORD_BOT_TOKEN`.

Redémarrez pour appliquer :

```bash
exit
fly machine restart <machine-id>
```


<div id="6-access-the-gateway">
  ## 6) Accéder au Gateway
</div>

<div id="control-ui">
  ### Control UI
</div>

Ouvrir dans le navigateur :

```bash
fly open
```

Ou accédez à `https://my-openclaw.fly.dev/`

Collez votre jeton du Gateway (la valeur de `OPENCLAW_GATEWAY_TOKEN`) pour vous authentifier.


<div id="logs">
  ### Logs
</div>

```bash
fly logs              # Journaux en direct
fly logs --no-tail    # Journaux récents
```


<div id="ssh-console">
  ### Console SSH
</div>

```bash
fly ssh console
```


<div id="troubleshooting">
  ## Résolution des problèmes
</div>

<div id="app-is-not-listening-on-expected-address">
  ### « L’application n’écoute pas à l’adresse attendue »
</div>

Gateway écoute sur `127.0.0.1` au lieu de `0.0.0.0`.

**Correctif :** ajoutez `--bind lan` à votre commande de processus dans `fly.toml`.

<div id="health-checks-failing-connection-refused">
  ### Les vérifications d'état échouent / connexion refusée
</div>

Fly ne parvient pas à atteindre le Gateway sur le port configuré.

**Correction :** Assurez-vous que `internal_port` correspond au port utilisé par le Gateway (définissez `--port 3000` ou `OPENCLAW_GATEWAY_PORT=3000`).

<div id="oom-memory-issues">
  ### Problèmes de mémoire / OOM
</div>

Le conteneur redémarre en boucle ou est tué. Symptômes : `SIGABRT`, `v8::internal::Runtime_AllocateInYoungGeneration` ou redémarrages silencieux.

**Solution :** Augmentez la mémoire dans `fly.toml` :

```toml
[[vm]]
  memory = "2048mb"
```

Ou bien mettez à jour une machine existante :

```bash
fly machine update <machine-id> --vm-memory 2048 -y
```

**Remarque :** 512 Mo est insuffisant. 1 Go peut fonctionner mais risque un OOM (manque de mémoire) sous charge ou avec une journalisation verbeuse. **2 Go sont recommandés.**


<div id="gateway-lock-issues">
  ### Problèmes de verrouillage de Gateway
</div>

Gateway refuse de démarrer et affiche des erreurs « already running ».

Cela se produit lorsque le conteneur redémarre mais que le fichier de verrouillage de PID persiste sur le volume.

**Correctif :** supprimez le fichier de verrouillage :

```bash
fly ssh console --command "rm -f /data/gateway.*.lock"
fly machine restart <machine-id>
```

Le fichier de verrouillage se trouve à l&#39;emplacement `/data/gateway.*.lock` (et non dans un sous-répertoire).


<div id="config-not-being-read">
  ### La configuration n’est pas prise en compte
</div>

Si vous utilisez `--allow-unconfigured`, le Gateway crée une configuration minimale. Votre configuration personnalisée dans `/data/openclaw.json` doit être prise en compte au redémarrage.

Vérifiez que la configuration existe :

```bash
fly ssh console --command "cat /data/openclaw.json"
```


<div id="writing-config-via-ssh">
  ### Écriture de la configuration via SSH
</div>

La commande `fly ssh console -C` ne prend pas en charge la redirection du shell. Pour écrire un fichier de configuration :

```bash
# Utiliser echo + tee (pipe du local vers le distant)
echo '{"your":"config"}' | fly ssh console -C "tee /data/openclaw.json"

# Or use sftp
fly sftp shell
> put /local/path/config.json /data/openclaw.json
```

**Remarque :** `fly sftp` peut échouer si le fichier existe déjà. Supprimez-le d’abord :

```bash
fly ssh console --command "rm /data/openclaw.json"
```


<div id="state-not-persisting">
  ### L'état ne persiste pas
</div>

Si vous perdez des identifiants ou des sessions après un redémarrage, cela signifie que le répertoire d'état est situé sur le système de fichiers du conteneur.

**Correction :** Assurez-vous que `OPENCLAW_STATE_DIR=/data` est défini dans `fly.toml`, puis redéployez.

<div id="updates">
  ## Mises à jour
</div>

```bash
# Récupérer les dernières modifications
git pull

# Redéployer
fly deploy

# Vérifier l'état
fly status
fly logs
```


<div id="updating-machine-command">
  ### Mise à jour de la commande de la machine
</div>

Si vous devez modifier la commande de démarrage sans procéder à un redéploiement complet :

```bash
# Get machine ID
fly machines list

# Update command
fly machine update <machine-id> --command "node dist/index.js gateway --port 3000 --bind lan" -y

# Ou avec augmentation de mémoire
fly machine update <machine-id> --vm-memory 2048 --command "node dist/index.js gateway --port 3000 --bind lan" -y
```

**Remarque :** Après `fly deploy`, la commande de la machine peut être réinitialisée à ce qui est défini dans `fly.toml`. Si vous avez effectué des modifications manuelles, réappliquez-les après le déploiement.


<div id="private-deployment-hardened">
  ## Déploiement privé (renforcé)
</div>

Par défaut, Fly alloue des adresses IP publiques, ce qui rend votre Gateway accessible à `https://your-app.fly.dev`. C’est pratique, mais cela signifie que votre déploiement est découvrable par des scanners Internet (Shodan, Censys, etc.).

Pour un déploiement renforcé sans **aucune exposition publique**, utilisez le template privé.

<div id="when-to-use-private-deployment">
  ### Quand utiliser un déploiement privé
</div>

- Vous ne faites que des appels/messages **sortants** (aucun webhook entrant)
- Vous utilisez des tunnels **ngrok ou Tailscale** pour les callbacks webhook
- Vous accédez au Gateway via **SSH, un proxy ou WireGuard** plutôt que via un navigateur
- Vous voulez que le déploiement soit **caché des scanners Internet**

<div id="setup">
  ### Configuration
</div>

Utilisez `fly.private.toml` au lieu du fichier de configuration standard :

```bash
# Déployer avec la config privée
fly deploy -c fly.private.toml
```

Ou convertissez un déploiement existant :

```bash
# Lister les IP actuelles
fly ips list -a my-openclaw

# Libérer les IP publiques
fly ips release <public-ipv4> -a my-openclaw
fly ips release <public-ipv6> -a my-openclaw

# Basculer vers la configuration privée pour éviter la réallocation d'IP publiques lors des futurs déploiements
# (supprimer [http_service] ou déployer avec le modèle privé)
fly deploy -c fly.private.toml

# Allouer une IPv6 privée uniquement
fly ips allocate-v6 --private -a my-openclaw
```

Ensuite, `fly ips list` ne devrait plus afficher qu’une adresse IP de type `private` :

```
VERSION  IP                   TYPE             REGION
v6       fdaa:x:x:x:x::x      private          global
```


<div id="accessing-a-private-deployment">
  ### Accéder à un déploiement privé
</div>

Puisqu&#39;il n&#39;y a pas d&#39;URL publique, utilisez l&#39;une des méthodes suivantes :

**Option 1 : proxy local (la plus simple)**

```bash
# Transférer le port local 3000 vers l'application
fly proxy 3000:3000 -a my-openclaw

# Puis ouvrir http://localhost:3000 dans le navigateur
```

**Option 2 : VPN WireGuard**

```bash
# Créer la configuration WireGuard (une seule fois)
fly wireguard create

# Importer dans le client WireGuard, puis accéder via l'IPv6 interne
# Exemple : http://[fdaa:x:x:x:x::x]:3000
```

**Option 3 : SSH uniquement**

```bash
fly ssh console -a my-openclaw
```


<div id="webhooks-with-private-deployment">
  ### Webhooks avec déploiement privé
</div>

Si vous avez besoin de callbacks webhook (Twilio, Telnyx, etc.) sans exposition publique :

1. **Tunnel ngrok** - Exécutez ngrok dans le conteneur ou en tant que sidecar
2. **Tailscale Funnel** - Exposez des chemins spécifiques via Tailscale
3. **Sortant uniquement** - Certains fournisseurs (Twilio) fonctionnent correctement pour les appels sortants sans webhooks

Exemple de configuration d’appel vocal avec ngrok :

```json
{
  "plugins": {
    "entries": {
      "voice-call": {
        "enabled": true,
        "config": {
          "provider": "twilio",
          "tunnel": { "provider": "ngrok" }
        }
      }
    }
  }
}
```

Le tunnel ngrok s’exécute à l’intérieur du conteneur et fournit une URL de webhook publique sans exposer l’application Fly elle-même.


<div id="security-benefits">
  ### Avantages en matière de sécurité
</div>

| Aspect | Public | Privé |
|--------|--------|---------|
| Scanners Internet | Détectable | Masqué |
| Attaques directes | Possibles | Bloquées |
| Accès au Control UI | Navigateur | Proxy/VPN |
| Remise des webhooks | Directe | Via un tunnel |

<div id="notes">
  ## Notes
</div>

- Fly.io utilise l’**architecture x86** (et non ARM)
- Le Dockerfile est compatible avec les deux architectures
- Pour l’onboarding WhatsApp/Telegram, utilisez `fly ssh console`
- Les données persistantes se trouvent sur le volume `/data`
- Signal nécessite Java + signal-cli ; utilisez une image personnalisée et configurez la mémoire à au moins 2 Go.

<div id="cost">
  ## Coût
</div>

Avec la configuration recommandée (`shared-cpu-2x`, 2 Go de RAM) :

- ~10 à 15 $/mois selon l’usage
- Le palier gratuit inclut un certain quota

Voir la [tarification Fly.io](https://fly.io/docs/about/pricing/) pour plus de détails.