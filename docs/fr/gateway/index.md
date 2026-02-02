---
title: Gateway
summary: "Runbook opérationnel du service Gateway : cycle de vie et exploitation"
read_when:
  - Lors de l’exécution ou du débogage du processus Gateway
---

<div id="gateway-service-runbook">
  # Runbook du service Gateway
</div>

Dernière mise à jour : 2025-12-09

<div id="what-it-is">
  ## Ce que c&#39;est
</div>

* Le processus toujours actif qui maintient l&#39;unique connexion Baileys/Telegram et le plan de contrôle/événements.
* Remplace l&#39;ancienne commande `gateway`. Point d&#39;entrée CLI : `openclaw gateway`.
* Tourne en continu jusqu&#39;à son arrêt ; se termine avec un code de sortie non nul en cas d&#39;erreur fatale afin que le superviseur le redémarre.

<div id="how-to-run-local">
  ## Exécuter en local
</div>

```bash
openclaw gateway --port 18789
# for full debug/trace logs in stdio:
openclaw gateway --port 18789 --verbose
# si le port est occupé, terminer les processus d'écoute puis démarrer :
openclaw gateway --force
# dev loop (auto-reload on TS changes):
pnpm gateway:watch
```

* Le rechargement à chaud de la configuration surveille `~/.openclaw/openclaw.json` (ou `OPENCLAW_CONFIG_PATH`).
  * Mode par défaut : `gateway.reload.mode="hybrid"` (applique à chaud les changements sûrs, redémarre en cas de changements critiques).
  * Le rechargement à chaud utilise un redémarrage en processus via **SIGUSR1** si nécessaire.
  * Désactiver avec `gateway.reload.mode="off"`.
* Lie le plan de contrôle WebSocket à `127.0.0.1:<port>` (18789 par défaut).
* Le même port sert aussi HTTP (Control UI, hooks, A2UI). Multiplexage sur un seul port.
  * OpenAI Chat Completions (HTTP) : [`/v1/chat/completions`](/fr/gateway/openai-http-api).
  * OpenResponses (HTTP) : [`/v1/responses`](/fr/gateway/openresponses-http-api).
  * Tools Invoke (HTTP) : [`/tools/invoke`](/fr/gateway/tools-invoke-http-api).
* Démarre par défaut un serveur de fichiers Canvas sur `canvasHost.port` (par défaut `18793`), servant `http://<gateway-host>:18793/__openclaw__/canvas/` à partir de `~/.openclaw/workspace/canvas`. Désactiver avec `canvasHost.enabled=false` ou `OPENCLAW_SKIP_CANVAS_HOST=1`.
* Consigne les journaux sur stdout ; utilisez launchd/systemd pour le maintenir en vie et faire la rotation des journaux.
* Passez `--verbose` pour dupliquer les journaux de débogage (handshakes, req/res, événements) du fichier de log vers stdio lors du diagnostic.
* `--force` utilise `lsof` pour trouver les processus à l’écoute sur le port choisi, envoie SIGTERM, consigne les processus qu’il a terminés, puis démarre le Gateway (échoue rapidement si `lsof` est indisponible).
* Si vous l’exécutez sous un superviseur (launchd/systemd/mode processus enfant de l’app mac), un arrêt/redémarrage envoie généralement **SIGTERM** ; des versions plus anciennes peuvent exposer cela comme un code de sortie `pnpm` `ELIFECYCLE` **143** (SIGTERM), ce qui correspond à un arrêt normal, pas à un crash.
* **SIGUSR1** déclenche un redémarrage en processus lorsqu’il est autorisé (application/mise à jour d’un outil/config du Gateway, ou activation de `commands.restart` pour des redémarrages manuels).
* L’authentification du Gateway est requise par défaut : définissez `gateway.auth.token` (ou `OPENCLAW_GATEWAY_TOKEN`) ou `gateway.auth.password`. Les clients doivent envoyer `connect.params.auth.token/password`, sauf en cas d’utilisation de l’identité Tailscale Serve.
* L’assistant génère désormais un jeton par défaut, même sur l’interface loopback.
* Priorité des ports : `--port` &gt; `OPENCLAW_GATEWAY_PORT` &gt; `gateway.port` &gt; par défaut `18789`.

<div id="remote-access">
  ## Accès distant
</div>

* Tailscale/VPN recommandé ; sinon, tunnel SSH :
  ```bash
  ssh -N -L 18789:127.0.0.1:18789 user@host
  ```
* Les clients se connectent ensuite à `ws://127.0.0.1:18789` via le tunnel.
* Si un jeton est configuré, les clients doivent l&#39;inclure dans `connect.params.auth.token`, même lorsqu&#39;ils passent par le tunnel.

<div id="multiple-gateways-same-host">
  ## Plusieurs Gateways (même hôte)
</div>

Généralement inutile : une seule instance Gateway peut servir plusieurs canaux de messagerie et agents. N’utilisez plusieurs Gateways que pour la redondance ou une isolation stricte (ex. : bot de secours).

C’est possible si vous isolez l’état et la configuration et utilisez des ports uniques. Guide complet : [Plusieurs gateways](/fr/gateway/multiple-gateways).

Les noms de service sont sensibles au profil :

* macOS : `bot.molt.<profile>` (l’ancienne forme `com.openclaw.*` peut encore exister)
* Linux : `openclaw-gateway-<profile>.service`
* Windows : `OpenClaw Gateway (<profile>)`

Les métadonnées d’installation sont intégrées dans la configuration du service :

* `OPENCLAW_SERVICE_MARKER=openclaw`
* `OPENCLAW_SERVICE_KIND=gateway`
* `OPENCLAW_SERVICE_VERSION=<version>`

Modèle « Rescue-Bot » : maintenez un deuxième Gateway isolé avec son propre profil, son répertoire d’état, son espace de travail et un décalage de ports de base dédié. Guide complet : [Guide Rescue-bot](/fr/gateway/multiple-gateways#rescue-bot-guide).

<div id="dev-profile-dev">
  ### Profil de développement (`--dev`)
</div>

Procédure rapide : lancer une instance de développement complètement isolée (config/état/espace de travail) sans impacter votre configuration principale.

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
# puis ciblez l'instance de dev :
openclaw --dev status
openclaw --dev health
```

Valeurs par défaut (peuvent être surchargées via env/flags/config) :

* `OPENCLAW_STATE_DIR=~/.openclaw-dev`
* `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
* `OPENCLAW_GATEWAY_PORT=19001` (Gateway WS + HTTP)
* port du service de contrôle du navigateur = `19003` (dérivé : `gateway.port+2`, interface loopback uniquement)
* `canvasHost.port=19005` (dérivé : `gateway.port+4`)
* la valeur par défaut de `agents.defaults.workspace` devient `~/.openclaw/workspace-dev` lorsque vous exécutez `setup`/`onboard` avec `--dev`.

Ports dérivés (règles générales) :

* Port de base = `gateway.port` (ou `OPENCLAW_GATEWAY_PORT` / `--port`)
* port du service de contrôle du navigateur = base + 2 (interface loopback uniquement)
* `canvasHost.port = base + 4` (ou `OPENCLAW_CANVAS_HOST_PORT` / surcharge via la configuration)
* Les ports CDP du profil navigateur sont attribués automatiquement à partir de `browser.controlPort + 9 .. + 108` (conservés par profil).

Liste de contrôle par instance :

* `gateway.port` unique
* `OPENCLAW_CONFIG_PATH` unique
* `OPENCLAW_STATE_DIR` unique
* `agents.defaults.workspace` unique
* numéros WhatsApp distincts (si vous utilisez WA)

Installation du service par profil :

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

Exemple :

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

<div id="protocol-operator-view">
  ## Protocole (vue opérateur)
</div>

* Documentation complète : [Gateway protocol](/fr/gateway/protocol) et [Bridge protocol (legacy)](/fr/gateway/bridge-protocol).
* Trame initiale obligatoire envoyée par le client : `req {type:"req", id, method:"connect", params:{minProtocol,maxProtocol,client:{id,displayName?,version,platform,deviceFamily?,modelIdentifier?,mode,instanceId?}, caps, auth?, locale?, userAgent? } }`.
* Le Gateway répond `res {type:"res", id, ok:true, payload:hello-ok }` (ou `ok:false` avec une erreur, puis ferme la connexion).
* Après la phase de handshake :
  * Requêtes : `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
  * Événements : `{type:"event", event, payload, seq?, stateVersion?}`
* Entrées de présence structurées : `{host, ip, version, platform?, deviceFamily?, modelIdentifier?, mode, lastInputSeconds?, ts, reason?, tags?[], instanceId? }` (pour les clients WS, `instanceId` provient de `connect.client.instanceId`).
* Les réponses `agent` se font en deux étapes : d’abord l’ACK `res` `{runId,status:"accepted"}`, puis une `res` finale `{runId,status:"ok"|"error",summary}` une fois l’exécution terminée ; la sortie en flux continu arrive sous la forme de `event:"agent"`.

<div id="methods-initial-set">
  ## Méthodes (ensemble initial)
</div>

* `health` — instantané complet de l’état de santé (même structure que `openclaw health --json`).
* `status` — bref résumé.
* `system-presence` — liste de présence actuelle.
* `system-event` — publier une note de présence/système (structurée).
* `send` — envoyer un message via le ou les canaux actifs.
* `agent` — exécuter un tour d’agent (renvoie les événements en streaming sur la même connexion).
* `node.list` — lister les nœuds appairés et actuellement connectés (inclut `caps`, `deviceFamily`, `modelIdentifier`, `paired`, `connected`, et les commandes annoncées (`commands`)).
* `node.describe` — décrire un nœud (capacités + commandes `node.invoke` prises en charge ; fonctionne pour les nœuds appairés et pour les nœuds non appairés actuellement connectés).
* `node.invoke` — invoquer une commande sur un nœud (par ex. `canvas.*`, `camera.*`).
* `node.pair.*` — cycle de vie d’appairage (`request`, `list`, `approve`, `reject`, `verify`).

Voir aussi : [Présence](/fr/concepts/presence) pour comprendre comment la présence est générée/dédupliquée et pourquoi un `client.instanceId` stable est important.

<div id="events">
  ## Événements
</div>

* `agent` — événements de diffusion d’outils/de sortie émis pendant l’exécution de l’agent (étiquetés par séquence).
* `presence` — mises à jour de présence (deltas avec stateVersion) diffusées à tous les clients connectés.
* `tick` — signal périodique de keepalive/no-op pour confirmer la vivacité.
* `shutdown` — le Gateway est en train de s’arrêter ; la charge utile contient `reason` et, éventuellement, `restartExpectedMs`. Les clients doivent se reconnecter.

<div id="webchat-integration">
  ## Intégration WebChat
</div>

* WebChat est une UI SwiftUI native qui communique directement avec le WebSocket du Gateway pour l’historique, l’envoi, les annulations et les événements.
* L’utilisation à distance passe par le même tunnel SSH/Tailscale ; si un jeton Gateway est configuré, le client l’inclut lors de `connect`.
* L’app macOS se connecte via un seul WS (connexion partagée) ; elle initialise l’état de présence à partir de l’instantané initial et écoute les événements `presence` pour mettre à jour l’UI.

<div id="typing-and-validation">
  ## Typage et validation
</div>

* Le serveur valide chaque trame entrante avec AJV par rapport au schéma JSON généré à partir des définitions de protocole.
* Les clients (TS/Swift) utilisent les types générés (TS directement ; Swift via le générateur du dépôt).
* Les définitions de protocole font foi ; régénérez le schéma et les modèles avec :
  * `pnpm protocol:gen`
  * `pnpm protocol:gen:swift`

<div id="connection-snapshot">
  ## Instantané de connexion
</div>

* `hello-ok` inclut un `snapshot` avec `presence`, `health`, `stateVersion` et `uptimeMs`, ainsi que `policy {maxPayload,maxBufferedBytes,tickIntervalMs}`, afin que les clients puissent l’afficher immédiatement sans requêtes supplémentaires.
* `health`/`system-presence` restent disponibles pour un rafraîchissement manuel, mais ne sont pas requis au moment de la connexion.

<div id="error-codes-reserror-shape">
  ## Codes d’erreur (structure de `res.error`)
</div>

* Les erreurs utilisent `{ code, message, details?, retryable?, retryAfterMs? }`.
* Codes standard :
  * `NOT_LINKED` — WhatsApp non authentifié.
  * `AGENT_TIMEOUT` — l’agent n’a pas répondu dans le délai configuré.
  * `INVALID_REQUEST` — la validation du schéma ou des paramètres a échoué.
  * `UNAVAILABLE` — le Gateway est en cours d’arrêt ou une dépendance est indisponible.

<div id="keepalive-behavior">
  ## Comportement de keepalive
</div>

* Les événements `tick` (ou les ping/pong WS) sont émis périodiquement afin que les clients sachent que Gateway est toujours actif, même en l’absence de trafic.
* Les accusés de réception de `send`/agent restent des réponses distinctes ; ne surchargez pas les ticks pour les envois.

<div id="replay-gaps">
  ## Relecture / écarts de séquence
</div>

* Les événements ne sont pas rejoués. Les clients détectent les écarts de séquence (`seq`) et doivent actualiser (`health` + `system-presence`) avant de continuer. Les clients WebChat et macOS procèdent désormais automatiquement à cette actualisation en cas d’écart.

<div id="supervision-macos-example">
  ## Supervision (exemple macOS)
</div>

* Utilisez launchd pour garder le service actif :
  * Program : chemin vers `openclaw`
  * Arguments : `gateway`
  * KeepAlive : true
  * StandardOut/Err : chemins de fichiers ou `syslog`
* En cas d’échec, launchd redémarre ; une erreur de configuration fatale doit continuer à provoquer une sortie immédiate afin que l’opérateur la remarque.
* Les LaunchAgents sont propres à chaque utilisateur et nécessitent une session utilisateur ouverte ; pour les configurations sans interface (headless), utilisez un LaunchDaemon personnalisé (non fourni).
  * `openclaw gateway install` écrit `~/Library/LaunchAgents/bot.molt.gateway.plist`
    (ou `bot.molt.<profile>.plist` ; l’ancien préfixe `com.openclaw.*` est supprimé automatiquement).
  * `openclaw doctor` audite la configuration du LaunchAgent et peut la mettre à jour en fonction des valeurs par défaut actuelles.

<div id="gateway-service-management-cli">
  ## Gestion du service Gateway (CLI)
</div>

Utilisez la CLI de Gateway pour installer, démarrer, arrêter, redémarrer et vérifier l&#39;état :

```bash
openclaw gateway status
openclaw gateway install
openclaw gateway stop
openclaw gateway restart
openclaw logs --follow
```

Notes :

* `gateway status` sonde le RPC du Gateway par défaut en utilisant le port/la config résolus du service (vous pouvez le remplacer avec `--url`).
* `gateway status --deep` ajoute des analyses au niveau système (LaunchDaemons/unités système).
* `gateway status --no-probe` ignore la sonde RPC (utile lorsque le réseau est indisponible).
* `gateway status --json` est stable pour les scripts.
* `gateway status` signale **le runtime du superviseur** (launchd/systemd en cours d’exécution) séparément de **l’accessibilité RPC** (connexion WS + RPC de statut).
* `gateway status` affiche le chemin de config ainsi que la cible de la sonde pour éviter la confusion « localhost vs liaison LAN » et les incohérences de profil.
* `gateway status` inclut la dernière ligne d’erreur du Gateway lorsque le service semble en cours d’exécution mais que le port est fermé.
* `logs` suit le fichier de log du Gateway via RPC (pas besoin de `tail`/`grep` manuels).
* Si d’autres services de type Gateway sont détectés, la CLI émet un avertissement, sauf s’il s’agit de services de profil OpenClaw.
  Nous recommandons toujours **un Gateway par machine** pour la plupart des configurations ; utilisez des profils/ports isolés pour la redondance ou un bot de secours. Voir [Multiple gateways](/fr/gateway/multiple-gateways).
  * Nettoyage : `openclaw gateway uninstall` (service actuel) et `openclaw doctor` (migrations héritées).
* `gateway install` est sans effet lorsqu’il est déjà installé ; utilisez `openclaw gateway install --force` pour réinstaller (modifications de profil/env/chemin).

App macOS intégrée :

* OpenClaw.app peut embarquer un relais de Gateway basé sur Node et installer un LaunchAgent par utilisateur, étiqueté
  `bot.molt.gateway` (ou `bot.molt.<profile>` ; les anciennes étiquettes `com.openclaw.*` se déchargent toujours proprement).
* Pour l’arrêter proprement, utilisez `openclaw gateway stop` (ou `launchctl bootout gui/$UID/bot.molt.gateway`).
* Pour le redémarrer, utilisez `openclaw gateway restart` (ou `launchctl kickstart -k gui/$UID/bot.molt.gateway`).
  * `launchctl` ne fonctionne que si le LaunchAgent est installé ; sinon, utilisez d’abord `openclaw gateway install`.
  * Remplacez l’étiquette par `bot.molt.<profile>` lorsque vous exécutez un profil nommé.

<div id="supervision-systemd-user-unit">
  ## Supervision (unité utilisateur systemd)
</div>

OpenClaw installe par défaut un **service utilisateur systemd** sur Linux/WSL2. Nous
recommandons les services utilisateur pour les machines mono-utilisateur (environnement plus simple, configuration par utilisateur).
Utilisez un **service système** pour les serveurs multi-utilisateurs ou toujours actifs (pas de lingering
nécessaire, supervision partagée).

`openclaw gateway install` écrit l’unité utilisateur. `openclaw doctor` vérifie l’unité
et peut la mettre à jour afin de la mettre en conformité avec les valeurs par défaut actuellement recommandées.

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
Environment=OPENCLAW_GATEWAY_TOKEN=
WorkingDirectory=/home/youruser

[Install]
WantedBy=default.target
```

Activez le mode persistant (nécessaire pour que le service utilisateur reste actif après une déconnexion ou une période d’inactivité) :

```
sudo loginctl enable-linger youruser
```

L’onboarding exécute ceci sous Linux/WSL2 (peut demander sudo et écrit dans `/var/lib/systemd/linger`).
Ensuite, activez le service :

```
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

**Alternative (service système)** - pour les serveurs toujours en fonctionnement ou multi-utilisateurs, vous pouvez
installer une unité **système** systemd au lieu d’une unité utilisateur (aucun « lingering » nécessaire).
Créez `/etc/systemd/system/openclaw-gateway[-<profile>].service` (copiez l’unité ci‑dessus,
remplacez par `WantedBy=multi-user.target`, définissez `User=` + `WorkingDirectory=`), puis :

```
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

<div id="windows-wsl2">
  ## Windows (WSL2)
</div>

Les installations sur Windows doivent utiliser **WSL2** et suivre la section Linux systemd ci-dessus.

<div id="operational-checks">
  ## Vérifications opérationnelles
</div>

* Vivacité : ouvrir une connexion WS et envoyer `req:connect` → s’attendre à recevoir une réponse `res` avec `payload.type="hello-ok"` (avec un instantané d’état).
* Préparation : appeler `health` → s’attendre à obtenir `ok: true` et un canal lié dans `linkChannel` (le cas échéant).
* Débogage : s’abonner aux événements `tick` et `presence` ; vérifier que `status` affiche l’ancienneté du lien/de l’authentification ; vérifier que les entrées de présence affichent l’hôte Gateway et les clients connectés.

<div id="safety-guarantees">
  ## Garanties de sécurité
</div>

* Par défaut, supposez une seule instance Gateway par hôte ; si vous exécutez plusieurs profils, isolez les ports et l&#39;état et ciblez la bonne instance.
* Aucun basculement vers des connexions Baileys directes ; si le Gateway est indisponible, les opérations d&#39;envoi échouent immédiatement.
* Les premières trames qui ne sont pas de type connect ou tout JSON mal formé sont rejetés et le socket est fermé.
* Arrêt en douceur : émission d&#39;un événement `shutdown` avant la fermeture ; les clients doivent gérer la fermeture et la reconnexion.

<div id="cli-helpers">
  ## Outils CLI
</div>

* `openclaw gateway health|status` — demande l&#39;état/la santé du Gateway via WS.
* `openclaw message send --target <num> --message "hi" [--media ...]` — envoie via le Gateway (idempotent pour WhatsApp).
* `openclaw agent --message "hi" --to <num>` — exécute un tour d’agent (attend le résultat final par défaut).
* `openclaw gateway call <method> --params '{"k":"v"}'` — appel brut de méthode pour le débogage.
* `openclaw gateway stop|restart` — arrête/redémarre le service Gateway supervisé (launchd/systemd).
* Les sous-commandes d&#39;aide de Gateway supposent un Gateway déjà en cours d&#39;exécution sur `--url` ; elles ne le lancent plus automatiquement.

<div id="migration-guidance">
  ## Guide de migration
</div>

* Supprimez les utilisations de `openclaw gateway` et du port de contrôle TCP hérité.
* Mettez à jour les clients pour qu&#39;ils utilisent le protocole WS avec connexion obligatoire et présence structurée.