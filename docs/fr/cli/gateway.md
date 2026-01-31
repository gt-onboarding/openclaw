---
title: Gateway
summary: "CLI Gateway d’OpenClaw (`openclaw gateway`) — exécuter, interroger et découvrir des gateways"
read_when:
  - Lancement de Gateway depuis la CLI (dev ou serveurs)
  - Débogage de l’authentification, des modes de liaison (bind) et de la connectivité de Gateway
  - Découverte de gateways via Bonjour (LAN + tailnet)
---

<div id="gateway-cli">
  # CLI du Gateway
</div>

Le Gateway est le serveur WebSocket d’OpenClaw (canaux, nœuds, sessions, hooks).

Les sous-commandes décrites sur cette page s’utilisent via `openclaw gateway …`.

Documentation associée :

* [/gateway/bonjour](/fr/gateway/bonjour)
* [/gateway/discovery](/fr/gateway/discovery)
* [/gateway/configuration](/fr/gateway/configuration)

<div id="run-the-gateway">
  ## Lancer Gateway
</div>

Lancez un processus Gateway local :

```bash
openclaw gateway
```

Alias au premier plan :

```bash
openclaw gateway run
```

Notes :

* Par défaut, Gateway refuse de démarrer tant que `gateway.mode=local` n’est pas défini dans `~/.openclaw/openclaw.json`. Utilisez `--allow-unconfigured` pour des exécutions ad hoc / de développement.
* L’écoute au-delà de l’interface loopback sans authentification est bloquée (garde-fou de sécurité).
* `SIGUSR1` déclenche un redémarrage dans le processus si autorisé (activez `commands.restart` ou utilisez l’outil/config du Gateway pour apply/update).
* Les gestionnaires `SIGINT`/`SIGTERM` arrêtent le processus du Gateway, mais ils ne restaurent pas l’éventuel état personnalisé du terminal. Si vous encapsulez la CLI avec une TUI ou une entrée en mode raw, restaurez le terminal avant la sortie.

<div id="options">
  ### Options
</div>

* `--port <port>`: port WebSocket (valeur par défaut issue de la configuration/de l’environnement ; généralement `18789`).
* `--bind <loopback|lan|tailnet|auto|custom>`: mode de liaison du listener.
* `--auth <token|password>`: remplace le mode d’authentification.
* `--token <token>`: remplace le jeton (définit également `OPENCLAW_GATEWAY_TOKEN` pour le processus).
* `--password <password>`: remplace le mot de passe (définit également `OPENCLAW_GATEWAY_PASSWORD` pour le processus).
* `--tailscale <off|serve|funnel>`: expose le Gateway via Tailscale.
* `--tailscale-reset-on-exit`: réinitialise la configuration Tailscale serve/funnel à l’arrêt.
* `--allow-unconfigured`: autorise le démarrage du Gateway sans `gateway.mode=local` dans la configuration.
* `--dev`: crée une configuration de dev + espace de travail si manquants (ignore BOOTSTRAP.md).
* `--reset`: réinitialise la configuration de dev + identifiants + sessions + espace de travail (requiert `--dev`).
* `--force`: arrête tout listener existant sur le port sélectionné avant de démarrer.
* `--verbose`: journaux détaillés.
* `--claude-cli-logs`: affiche uniquement les journaux claude-cli dans la console (et active son stdout/stderr).
* `--ws-log <auto|full|compact>`: style de journalisation WebSocket (par défaut `auto`).
* `--compact`: alias de `--ws-log compact`.
* `--raw-stream`: journalise les événements bruts du flux du modèle en jsonl.
* `--raw-stream-path <path>`: chemin du fichier jsonl de flux brut.

<div id="query-a-running-gateway">
  ## Interroger un Gateway en cours d’exécution
</div>

Toutes les commandes de requête utilisent le RPC WebSocket.

Modes de sortie :

* Par défaut : lisible par un humain (coloré en TTY).
* `--json` : JSON lisible par une machine (sans mise en forme / indicateur de progression).
* `--no-color` (ou `NO_COLOR=1`) : désactiver l’ANSI tout en conservant un format lisible par un humain.

Options partagées (lorsqu’elles sont prises en charge) :

* `--url <url>` : URL WebSocket du Gateway.
* `--token <token>` : jeton du Gateway.
* `--password <password>` : mot de passe du Gateway.
* `--timeout <ms>` : délai/budget de temps (varie selon la commande).
* `--expect-final` : attendre une réponse « finale » (appels d’agent).

<div id="gateway-health">
  ### `gateway health`
</div>

```bash
openclaw gateway health --url ws://127.0.0.1:18789
```

<div id="gateway-status">
  ### `gateway status`
</div>

`gateway status` affiche le service Gateway (launchd/systemd/schtasks) ainsi qu’un test RPC facultatif.

```bash
openclaw gateway status
openclaw gateway status --json
```

Options :

* `--url <url>` : remplace l’URL utilisée pour la sonde.
* `--token <token>` : authentification par jeton pour la sonde.
* `--password <password>` : authentification par mot de passe pour la sonde.
* `--timeout <ms>` : délai d’expiration de la sonde (par défaut : `10000`).
* `--no-probe` : ignore la sonde RPC (vue limitée aux services).
* `--deep` : analyse également les services système.

<div id="gateway-probe">
  ### `gateway probe`
</div>

`gateway probe` est la commande de diagnostic global. Elle sonde toujours :

* votre Gateway distante configurée (si définie), et
* localhost (loopback) **même si une Gateway distante est configurée**.

Si plusieurs Gateways sont accessibles, elle les affiche toutes. Plusieurs Gateways sont prises en charge lorsque vous utilisez des profils ou ports isolés (par exemple un bot de secours), mais la plupart des installations ne font encore tourner qu&#39;une seule Gateway.

```bash
openclaw gateway probe
openclaw gateway probe --json
```

<div id="remote-over-ssh-mac-app-parity">
  #### Connexion distante via SSH (parité avec l’app Mac)
</div>

Le mode « Remote over SSH » de l’app macOS utilise une redirection de port locale afin que le Gateway distant (qui peut être lié uniquement à l’interface loopback) devienne accessible à `ws://127.0.0.1:<port>`.

Équivalent en CLI :

```bash
openclaw gateway probe --ssh user@gateway-host
```

Options :

* `--ssh <target>` : `user@host` ou `user@host:port` (le port par défaut est `22`).
* `--ssh-identity <path>` : fichier d&#39;identité SSH.
* `--ssh-auto` : sélectionne le premier hôte Gateway découvert comme cible SSH (LAN/WAB uniquement).

Configuration (facultative, utilisée comme valeurs par défaut) :

* `gateway.remote.sshTarget`
* `gateway.remote.sshIdentity`

<div id="gateway-call-method">
  ### `gateway call <method>`
</div>

Utilitaire RPC de bas niveau.

```bash
openclaw gateway call status
openclaw gateway call logs.tail --params '{"sinceMs": 60000}'
```

<div id="manage-the-gateway-service">
  ## Gérer le service Gateway
</div>

```bash
openclaw gateway install
openclaw gateway start
openclaw gateway stop
openclaw gateway restart
openclaw gateway uninstall
```

Remarques :

* `gateway install` accepte les options `--port`, `--runtime`, `--token`, `--force`, `--json`.
* Les commandes de cycle de vie acceptent l’option `--json` pour le scripting.

<div id="discover-gateways-bonjour">
  ## Découvrir des gateways (Bonjour)
</div>

`gateway discover` effectue un scan des balises Gateway (`_openclaw-gw._tcp`).

* Multicast DNS-SD : `local.`
* DNS-SD unicast (Wide-Area Bonjour) : choisissez un domaine (par exemple : `openclaw.internal.`) et configurez un split DNS + un serveur DNS ; voir [/gateway/bonjour](/fr/gateway/bonjour)

Seules les gateways avec la découverte Bonjour activée (par défaut) annoncent la balise.

Les enregistrements de découverte Wide-Area comprennent (TXT) :

* `role` (indication sur le rôle de la gateway)
* `transport` (indication sur le transport, par ex. `gateway`)
* `gatewayPort` (port WebSocket, généralement `18789`)
* `sshPort` (port SSH ; par défaut `22` s’il n’est pas présent)
* `tailnetDns` (nom d’hôte MagicDNS, lorsque disponible)
* `gatewayTls` / `gatewayTlsSha256` (TLS activé + empreinte du certificat)
* `cliPath` (indication facultative pour les installations distantes)

<div id="gateway-discover">
  ### `gateway discover`
</div>

```bash
openclaw gateway discover
```

Options :

* `--timeout <ms>` : délai d&#39;expiration par commande (browse/resolve) ; valeur par défaut `2000`.
* `--json` : sortie destinée aux machines (désactive également le style/le spinner).

Exemples :

```bash
openclaw gateway discover --timeout 4000
openclaw gateway discover --json | jq '.beacons[].wsUrl'
```
