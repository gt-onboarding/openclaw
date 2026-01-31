---
title: CLI
summary: "Référence de la CLI OpenClaw pour les commandes, sous-commandes et options `openclaw`"
read_when:
  - Ajout ou modification de commandes ou d'options de la CLI
  - Documentation de nouveaux points d'entrée de commande
---

<div id="cli-reference">
  # Référence de la CLI
</div>

Ce document décrit le comportement actuel de la CLI. Si les commandes évoluent, mettez ce document à jour.

<div id="command-pages">
  ## Pages de commandes
</div>

* [`setup`](/fr/cli/setup)
* [`onboard`](/fr/cli/onboard)
* [`configure`](/fr/cli/configure)
* [`config`](/fr/cli/config)
* [`doctor`](/fr/cli/doctor)
* [`dashboard`](/fr/cli/dashboard)
* [`reset`](/fr/cli/reset)
* [`uninstall`](/fr/cli/uninstall)
* [`update`](/fr/cli/update)
* [`message`](/fr/cli/message)
* [`agent`](/fr/cli/agent)
* [`agents`](/fr/cli/agents)
* [`acp`](/fr/cli/acp)
* [`status`](/fr/cli/status)
* [`health`](/fr/cli/health)
* [`sessions`](/fr/cli/sessions)
* [`gateway`](/fr/cli/gateway)
* [`logs`](/fr/cli/logs)
* [`system`](/fr/cli/system)
* [`models`](/fr/cli/models)
* [`memory`](/fr/cli/memory)
* [`nodes`](/fr/cli/nodes)
* [`devices`](/fr/cli/devices)
* [`node`](/fr/cli/node)
* [`approvals`](/fr/cli/approvals)
* [`sandbox`](/fr/cli/sandbox)
* [`tui`](/fr/cli/tui)
* [`browser`](/fr/cli/browser)
* [`cron`](/fr/cli/cron)
* [`dns`](/fr/cli/dns)
* [`docs`](/fr/cli/docs)
* [`hooks`](/fr/cli/hooks)
* [`webhooks`](/fr/cli/webhooks)
* [`pairing`](/fr/cli/pairing)
* [`plugins`](/fr/cli/plugins) (commandes de plugin)
* [`channels`](/fr/cli/channels)
* [`security`](/fr/cli/security)
* [`skills`](/fr/cli/skills)
* [`voicecall`](/fr/cli/voicecall) (plugin ; si installé)

<div id="global-flags">
  ## Options globales
</div>

* `--dev` : isole l’état dans `~/.openclaw-dev` et modifie les ports par défaut.
* `--profile <name>` : isole l’état dans `~/.openclaw-<name>`.
* `--no-color` : désactive les couleurs ANSI.
* `--update` : raccourci pour `openclaw update` (installations depuis les sources uniquement).
* `-V`, `--version`, `-v` : affiche la version puis se termine.

<div id="output-styling">
  ## Style de sortie
</div>

* Les couleurs ANSI et les indicateurs de progression ne s’affichent que dans les sessions TTY.
* Les hyperliens OSC-8 s’affichent comme liens cliquables dans les terminaux compatibles ; sinon, une simple URL est utilisée en solution de repli.
* `--json` (et `--plain` lorsqu’elle est prise en charge) désactive le style pour une sortie épurée.
* `--no-color` désactive le style ANSI ; `NO_COLOR=1` est également pris en compte.
* Les commandes de longue durée affichent un indicateur de progression (OSC 9;4 lorsqu’il est pris en charge).

<div id="color-palette">
  ## Palette de couleurs
</div>

OpenClaw utilise une palette « lobster » pour la sortie de la CLI.

* `accent` (#FF5A2D) : titres, libellés, mises en évidence principales.
* `accentBright` (#FF7A3D) : noms de commandes, emphase.
* `accentDim` (#D14A22) : texte de mise en évidence secondaire.
* `info` (#FF8A5B) : valeurs d’information.
* `success` (#2FBF71) : états de réussite.
* `warn` (#FFB020) : avertissements, solutions de repli, éléments à surveiller.
* `error` (#E23D2D) : erreurs, échecs.
* `muted` (#8B7F77) : éléments atténués, métadonnées.

Fichier de référence pour la palette : `src/terminal/palette.ts` (alias « lobster seam »).

<div id="command-tree">
  ## Arborescence des commandes
</div>

```
openclaw [--dev] [--profile <name>] <command>
  setup
  onboard
  configure
  config
    get
    set
    unset
  doctor
  security
    audit
  reset
  uninstall
  update
  channels
    list
    status
    logs
    add
    remove
    login
    logout
  skills
    list
    info
    check
  plugins
    list
    info
    install
    enable
    disable
    doctor
  memory
    status
    index
    search
  message
  agent
  agents
    list
    add
    delete
  acp
  status
  health
  sessions
  gateway
    call
    health
    status
    probe
    discover
    install
    uninstall
    start
    stop
    restart
    run
  logs
  system
    event
    heartbeat last|enable|disable
    presence
  models
    list
    status
    set
    set-image
    aliases list|add|remove
    fallbacks list|add|remove|clear
    image-fallbacks list|add|remove|clear
    scan
    auth add|setup-token|paste-token
    auth order get|set|clear
  sandbox
    list
    recreate
    explain
  cron
    status
    list
    add
    edit
    rm
    enable
    disable
    runs
    run
  nodes
  devices
  node
    run
    status
    install
    uninstall
    start
    stop
    restart
  approvals
    get
    set
    allowlist add|remove
  browser
    status
    start
    stop
    reset-profile
    tabs
    open
    focus
    close
    profiles
    create-profile
    delete-profile
    screenshot
    snapshot
    navigate
    resize
    click
    type
    press
    hover
    drag
    select
    upload
    fill
    dialog
    wait
    evaluate
    console
    pdf
  hooks
    list
    info
    check
    enable
    disable
    install
    update
  webhooks
    gmail setup|run
  pairing
    list
    approve
  docs
  dns
    setup
  tui
```

Remarque : les plugins peuvent ajouter des commandes principales supplémentaires (par exemple `openclaw voicecall`).

<div id="security">
  ## Sécurité
</div>

* `openclaw security audit` — analyse la configuration et l’état local pour détecter les pièges de sécurité courants.
* `openclaw security audit --deep` — réalise une analyse en direct du Gateway dans la mesure du possible.
* `openclaw security audit --fix` — renforce les paramètres sûrs par défaut et ajuste les permissions (chmod) sur l’état et la configuration.

<div id="plugins">
  ## Plugins
</div>

Gérer les extensions et leur configuration :

* `openclaw plugins list` — découvrir les plugins (utiliser `--json` pour une sortie lisible par une machine).
* `openclaw plugins info <id>` — afficher les détails d&#39;un plugin.
* `openclaw plugins install <path|.tgz|npm-spec>` — installer un plugin (ou ajouter un chemin de plugin à `plugins.load.paths`).
* `openclaw plugins enable <id>` / `disable <id>` — activer/désactiver `plugins.entries.<id>.enabled`.
* `openclaw plugins doctor` — signaler les erreurs de chargement des plugins.

La plupart des modifications relatives aux plugins nécessitent un redémarrage du Gateway. Voir [/plugin](/fr/plugin).

<div id="memory">
  ## Mémoire
</div>

Recherche vectorielle dans `MEMORY.md` et `memory/*.md` :

* `openclaw memory status` — afficher les statistiques de l’index.
* `openclaw memory index` — réindexer les fichiers de mémoire.
* `openclaw memory search "<query>"` — effectuer une recherche sémantique dans la mémoire.

<div id="chat-slash-commands">
  ## Commandes slash du chat
</div>

Les messages de chat prennent en charge les commandes `/...` (textuelles et natives). Voir [/tools/slash-commands](/fr/tools/slash-commands).

Points importants :

* `/status` pour des diagnostics rapides.
* `/config` pour des modifications de configuration persistantes.
* `/debug` pour des remplacements de configuration valables uniquement à l’exécution (en mémoire, pas sur le disque ; nécessite `commands.debug: true`).

<div id="setup-onboarding">
  ## Installation et prise en main
</div>

<div id="setup">
  ### `setup`
</div>

Initialiser la configuration et l’espace de travail.

Options :

* `--workspace <dir>` : chemin de l’espace de travail de l’agent (par défaut `~/.openclaw/workspace`).
* `--wizard` : exécuter l’assistant d’onboarding.
* `--non-interactive` : exécuter l’assistant sans questions interactives.
* `--mode <local|remote>` : mode de l’assistant.
* `--remote-url <url>` : URL du Gateway distant.
* `--remote-token <token>` : jeton du Gateway distant.

L’assistant se lance automatiquement dès qu’une des options suivantes est utilisée (`--non-interactive`, `--mode`, `--remote-url`, `--remote-token`).

<div id="onboard">
  ### `onboard`
</div>

Assistant interactif pour configurer le Gateway, l’espace de travail et les compétences.

Options :

* `--workspace <dir>`
* `--reset` (réinitialise la configuration + les identifiants + les sessions + l’espace de travail avant l’assistant)
* `--non-interactive`
* `--mode <local|remote>`
* `--flow <quickstart|advanced|manual>` (`manual` est un alias de `advanced`)
* `--auth-choice <setup-token|token|chutes|openai-codex|openai-api-key|openrouter-api-key|ai-gateway-api-key|moonshot-api-key|kimi-code-api-key|synthetic-api-key|venice-api-key|gemini-api-key|zai-api-key|apiKey|minimax-api|minimax-api-lightning|opencode-zen|skip>`
* `--token-provider <id>` (non interactif ; utilisé avec `--auth-choice token`)
* `--token <token>` (non interactif ; utilisé avec `--auth-choice token`)
* `--token-profile-id <id>` (non interactif ; valeur par défaut : `<provider>:manual`)
* `--token-expires-in <duration>` (non interactif ; p. ex. `365d`, `12h`)
* `--anthropic-api-key <key>`
* `--openai-api-key <key>`
* `--openrouter-api-key <key>`
* `--ai-gateway-api-key <key>`
* `--moonshot-api-key <key>`
* `--kimi-code-api-key <key>`
* `--gemini-api-key <key>`
* `--zai-api-key <key>`
* `--minimax-api-key <key>`
* `--opencode-zen-api-key <key>`
* `--gateway-port <port>`
* `--gateway-bind <loopback|lan|tailnet|auto|custom>`
* `--gateway-auth <token|password>`
* `--gateway-token <token>`
* `--gateway-password <password>`
* `--remote-url <url>`
* `--remote-token <token>`
* `--tailscale <off|serve|funnel>`
* `--tailscale-reset-on-exit`
* `--install-daemon`
* `--no-install-daemon` (alias : `--skip-daemon`)
* `--daemon-runtime <node|bun>`
* `--skip-channels`
* `--skip-skills`
* `--skip-health`
* `--skip-ui`
* `--node-manager <npm|pnpm|bun>` (pnpm recommandé ; bun non recommandé pour l’exécution du Gateway)
* `--json`

<div id="configure">
  ### `configure`
</div>

Assistant de configuration interactif pour les modèles, canaux, compétences et le Gateway.

<div id="config">
  ### `config`
</div>

Utilitaires de configuration non interactifs (get/set/unset). L&#39;exécution de `openclaw config` sans
sous-commande lance l&#39;assistant.

Sous-commandes :

* `config get <path>` : affiche une valeur de configuration (chemin en notation par points/crochets).
* `config set <path> <value>` : définit une valeur (JSON5 ou chaîne de caractères brute).
* `config unset <path>` : supprime une valeur.

<div id="doctor">
  ### `doctor`
</div>

Vérifications d’intégrité + correctifs rapides (config + Gateway + services hérités).

Options :

* `--no-workspace-suggestions` : désactive les suggestions liées à la mémoire de l’espace de travail.
* `--yes` : accepte les valeurs par défaut sans confirmation (mode sans interface, headless).
* `--non-interactive` : ignore les confirmations ; applique uniquement les migrations sans risque.
* `--deep` : analyse les services système pour détecter des installations supplémentaires de Gateway.

<div id="channel-helpers">
  ## Utilitaires pour les canaux
</div>

<div id="channels">
  ### `channels`
</div>

Gérer les comptes de canaux de discussion (WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (plugin)/Signal/iMessage/MS Teams).

Sous-commandes :

* `channels list` : afficher les canaux configurés et les profils d’authentification.
* `channels status` : vérifier l’accessibilité du Gateway et l’état de santé des canaux (`--probe` exécute des vérifications supplémentaires ; utilisez `openclaw health` ou `openclaw status --deep` pour des vérifications de santé du Gateway).
* Astuce : `channels status` affiche des avertissements avec des corrections suggérées lorsqu’il peut détecter des erreurs de configuration courantes (puis vous renvoie vers `openclaw doctor`).
* `channels logs` : afficher les journaux récents des canaux à partir du fichier de journaux du Gateway.
* `channels add` : configuration via un assistant interactif (wizard) lorsqu’aucune option n’est fournie ; les options activent le mode non interactif.
* `channels remove` : désactive le canal par défaut ; passez `--delete` pour supprimer les entrées de configuration sans confirmation.
* `channels login` : connexion interactive au canal (WhatsApp Web uniquement).
* `channels logout` : se déconnecter d’une session de canal (si pris en charge).

Options communes :

* `--channel <name>` : `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams`
* `--account <id>` : identifiant du compte de canal (par défaut `default`)
* `--name <label>` : nom d’affichage pour le compte

Options de `channels login` :

* `--channel <channel>` (par défaut `whatsapp` ; prend en charge `whatsapp`/`web`)
* `--account <id>`
* `--verbose`

Options de `channels logout` :

* `--channel <channel>` (par défaut `whatsapp`)
* `--account <id>`

Options de `channels list` :

* `--no-usage` : ignorer les instantanés d’utilisation/de quota des fournisseurs de modèles (uniquement pour les connexions OAuth/API).
* `--json` : sortie JSON (inclut l’utilisation sauf si `--no-usage` est défini).

Options de `channels logs` :

* `--channel <name|all>` (par défaut `all`)
* `--lines <n>` (par défaut `200`)
* `--json`

Plus de détails : [/concepts/oauth](/fr/concepts/oauth)

Exemples :

```bash
openclaw channels add --channel telegram --account alerts --name "Alerts Bot" --token $TELEGRAM_BOT_TOKEN
openclaw channels add --channel discord --account work --name "Work Bot" --token $DISCORD_BOT_TOKEN
openclaw channels remove --channel discord --account work --delete
openclaw channels status --probe
openclaw status --deep
```

<div id="skills">
  ### `skills`
</div>

Lister et inspecter les compétences disponibles ainsi que leur état de préparation.

Sous-commandes :

* `skills list` : lister les compétences (comportement par défaut sans sous-commande).
* `skills info <name>` : afficher les détails d’une compétence.
* `skills check` : récapitulatif des prérequis satisfaits ou manquants.

Options :

* `--eligible` : afficher uniquement les compétences prêtes.
* `--json` : afficher du JSON (sans mise en forme).
* `-v`, `--verbose` : inclure le détail des prérequis manquants.

Conseil : utilisez `npx clawhub` pour rechercher, installer et synchroniser des compétences.

<div id="pairing">
  ### `pairing`
</div>

Approuver les demandes d&#39;appairage en messages privés sur tous les canaux.

Subcommands :

* `pairing list <channel> [--json]`
* `pairing approve <channel> <code> [--notify]`

<div id="webhooks-gmail">
  ### `webhooks gmail`
</div>

Configuration et exécution du hook Pub/Sub Gmail. Voir [/automation/gmail-pubsub](/fr/automation/gmail-pubsub).

Sous-commandes :

* `webhooks gmail setup` (nécessite `--account <email>` ; prend en charge `--project`, `--topic`, `--subscription`, `--label`, `--hook-url`, `--hook-token`, `--push-token`, `--bind`, `--port`, `--path`, `--include-body`, `--max-bytes`, `--renew-minutes`, `--tailscale`, `--tailscale-path`, `--tailscale-target`, `--push-endpoint`, `--json`)
* `webhooks gmail run` (paramètres d’exécution qui surchargent les mêmes options)

<div id="dns-setup">
  ### `dns setup`
</div>

Utilitaire DNS de découverte sur réseau étendu (CoreDNS + Tailscale). Voir [/gateway/discovery](/fr/gateway/discovery).

Options :

* `--apply` : installer/mettre à jour la configuration CoreDNS (nécessite sudo ; macOS uniquement).

<div id="messaging-agent">
  ## Messagerie + agent
</div>

<div id="message">
  ### `message`
</div>

Messagerie sortante unifiée et actions de canal.

Voir : [/cli/message](/fr/cli/message)

Sous-commandes :

* `message send|poll|react|reactions|read|edit|delete|pin|unpin|pins|permissions|search|timeout|kick|ban`
* `message thread <create|list|reply>`
* `message emoji <list|upload>`
* `message sticker <send|upload>`
* `message role <info|add|remove>`
* `message channel <info|list>`
* `message member info`
* `message voice status`
* `message event <list|create>`

Exemples :

* `openclaw message send --target +15555550123 --message "Hi"`
* `openclaw message poll --channel discord --target channel:123 --poll-question "Snack?" --poll-option Pizza --poll-option Sushi`

<div id="agent">
  ### `agent`
</div>

Exécuter un tour d’agent via le Gateway (ou en mode intégré avec `--local`).

Obligatoire :

* `--message <text>`

Options :

* `--to <dest>` (pour la clé de session et, éventuellement, la remise)
* `--session-id <id>`
* `--thinking <off|minimal|low|medium|high|xhigh>` (modèles GPT-5.2 + Codex uniquement)
* `--verbose <on|full|off>`
* `--channel <whatsapp|telegram|discord|slack|mattermost|signal|imessage|msteams>`
* `--local`
* `--deliver`
* `--json`
* `--timeout <seconds>`

<div id="agents">
  ### `agents`
</div>

Gérer des agents isolés (espaces de travail + authentification + routage).

<div id="agents-list">
  #### `agents list`
</div>

Affiche la liste des agents configurés.

Options :

* `--json`
* `--bindings`

<div id="agents-add-name">
  #### `agents add [name]`
</div>

Ajoute un nouvel agent isolé. Exécute l’assistant de configuration guidé sauf si des options (ou `--non-interactive`) sont passées ; `--workspace` est requis en mode non interactif.

Options :

* `--workspace <dir>`
* `--model <id>`
* `--agent-dir <dir>`
* `--bind <channel[:accountId]>` (répétable)
* `--non-interactive`
* `--json`

Les spécifications de liaison utilisent le format `channel[:accountId]`. Lorsque `accountId` est omis pour WhatsApp, l’identifiant de compte par défaut est utilisé.

<div id="agents-delete-id">
  #### `agents delete <id>`
</div>

Supprimer un agent et purger son espace de travail et son état.

Options :

* `--force`
* `--json`

<div id="acp">
  ### `acp`
</div>

Exécute le pont ACP qui connecte les IDE au Gateway.

Voir [`acp`](/fr/cli/acp) pour toutes les options et des exemples.

<div id="status">
  ### `status`
</div>

Afficher l’état de santé de la session liée et les destinataires récents.

Options :

* `--json`
* `--all` (diagnostic complet ; en lecture seule, prêt à être collé)
* `--deep` (sonder les canaux)
* `--usage` (afficher l’utilisation/le quota du fournisseur de modèles)
* `--timeout <ms>`
* `--verbose`
* `--debug` (alias de `--verbose`)

Remarques :

* La vue d’ensemble inclut l’état du Gateway et du service hôte du nœud lorsqu’il est disponible.

<div id="usage-tracking">
  ### Suivi d&#39;utilisation
</div>

OpenClaw peut afficher l&#39;utilisation et le quota des fournisseurs lorsque des identifiants OAuth/API sont disponibles.

Emplacements d&#39;affichage :

* `/status` (ajoute une courte ligne d&#39;utilisation du fournisseur lorsque disponible)
* `openclaw status --usage` (affiche la répartition complète par fournisseur)
* barre des menus macOS (section Usage sous Context)

Remarques :

* Les données proviennent directement des API d&#39;utilisation des fournisseurs (aucune estimation).
* Fournisseurs : Anthropic, GitHub Copilot, OpenAI Codex OAuth, ainsi que Gemini CLI/Antigravity lorsque ces plugins de fournisseurs sont activés.
* Si aucun identifiant approprié n&#39;existe, l&#39;utilisation est masquée.
* Détails : voir [Suivi d&#39;utilisation](/fr/concepts/usage-tracking).

<div id="health">
  ### `health`
</div>

Récupérer l&#39;état de santé du Gateway en cours d&#39;exécution.

Options :

* `--json`
* `--timeout <ms>`
* `--verbose`

<div id="sessions">
  ### `sessions`
</div>

Répertorie les sessions de conversation enregistrées.

Options :

* `--json`
* `--verbose`
* `--store <path>`
* `--active <minutes>`

<div id="reset-uninstall">
  ## Réinitialisation / Désinstallation
</div>

<div id="reset">
  ### `reset`
</div>

Réinitialise la configuration et l’état locaux (conserve la CLI installée).

Options :

* `--scope <config|config+creds+sessions|full>`
* `--yes`
* `--non-interactive`
* `--dry-run`

Remarques :

* `--non-interactive` nécessite `--scope` et `--yes`.

<div id="uninstall">
  ### `uninstall`
</div>

Désinstalle le service Gateway et les données locales (la CLI reste installée).

Options :

* `--service`
* `--state`
* `--workspace`
* `--app`
* `--all`
* `--yes`
* `--non-interactive`
* `--dry-run`

Remarques :

* `--non-interactive` nécessite `--yes` et des portées explicites (ou `--all`).

## Gateway

<div id="gateway">
  ### `gateway`
</div>

Lance le Gateway WebSocket.

Options :

* `--port <port>`
* `--bind <loopback|tailnet|lan|auto|custom>`
* `--token <token>`
* `--auth <token|password>`
* `--password <password>`
* `--tailscale <off|serve|funnel>`
* `--tailscale-reset-on-exit`
* `--allow-unconfigured`
* `--dev`
* `--reset` (réinitialise la configuration de développement, les identifiants, les sessions et l’espace de travail)
* `--force` (force l’arrêt de l’écouteur existant sur le port)
* `--verbose`
* `--claude-cli-logs`
* `--ws-log <auto|full|compact>`
* `--compact` (alias de `--ws-log compact`)
* `--raw-stream`
* `--raw-stream-path <chemin>`

<div id="gateway-service">
  ### `gateway service`
</div>

Gérer le service Gateway (launchd/systemd/schtasks).

Sous-commandes :

* `gateway status` (sonde par défaut le RPC du Gateway)
* `gateway install` (installation du service)
* `gateway uninstall`
* `gateway start`
* `gateway stop`
* `gateway restart`

Notes :

* `gateway status` sonde par défaut le RPC du Gateway en utilisant le port et la configuration résolus du service (pouvant être surchargé via `--url/--token/--password`).
* `gateway status` prend en charge `--no-probe`, `--deep` et `--json` pour les scripts.
* `gateway status` fait aussi remonter les services Gateway supplémentaires ou hérités lorsqu’il peut les détecter (`--deep` ajoute des analyses au niveau système). Les services OpenClaw portant un nom de profil sont traités comme des services de première classe et ne sont pas signalés comme « extra ».
* `gateway status` affiche le chemin de configuration utilisé par la CLI, ainsi que la configuration que le service utilise probablement (environnement du service), plus l’URL cible de la sonde résolue.
* `gateway install|uninstall|start|stop|restart` prennent en charge `--json` pour les scripts (la sortie par défaut reste lisible pour les humains).
* `gateway install` utilise par défaut le runtime Node ; bun est **non recommandé** (bogues WhatsApp/Telegram).
* Options de `gateway install` : `--port`, `--runtime`, `--token`, `--force`, `--json`.

<div id="logs">
  ### `logs`
</div>

Diffuse en continu les journaux de fichiers du Gateway via RPC.

Notes :

* Les sessions TTY affichent une vue structurée et colorisée ; en l’absence de TTY, la sortie revient au texte brut.
* `--json` émet du JSON délimité par ligne (un événement de log par ligne).

Exemples :

```bash
openclaw logs --follow
openclaw logs --limit 200
openclaw logs --plain
openclaw logs --json
openclaw logs --no-color
```

<div id="gateway-subcommand">
  ### `gateway <subcommand>`
</div>

Utilitaires CLI pour Gateway (utilisez `--url`, `--token`, `--password`, `--timeout`, `--expect-final` pour les sous-commandes RPC).

Sous-commandes :

* `gateway call <method> [--params <json>]`
* `gateway health`
* `gateway status`
* `gateway probe`
* `gateway discover`
* `gateway install|uninstall|start|stop|restart`
* `gateway run`

Appels RPC courants :

* `config.apply` (valider + écrire la configuration + redémarrer + réactiver)
* `config.patch` (fusionner une mise à jour partielle + redémarrer + réactiver)
* `update.run` (exécuter la mise à jour + redémarrer + réactiver)

Astuce : lorsque vous appelez directement `config.set`/`config.apply`/`config.patch`, transmettez le `baseHash` renvoyé par
`config.get` si une configuration existe déjà.

<div id="models">
  ## Modèles
</div>

Voir [/concepts/models](/fr/concepts/models) pour le comportement de repli et la stratégie d&#39;analyse.

Authentification Anthropic recommandée (setup-token) :

```bash
claude setup-token
openclaw models auth setup-token --provider anthropic
openclaw models status
```

<div id="models-root">
  ### `models` (racine)
</div>

`openclaw models` est un alias pour `models status`.

Options de niveau racine :

* `--status-json` (alias pour `models status --json`)
* `--status-plain` (alias pour `models status --plain`)

<div id="models-list">
  ### `models list`
</div>

Options :

* `--all`
* `--local`
* `--provider <nom>`
* `--json`
* `--plain`

<div id="models-status">
  ### `models status`
</div>

Options :

* `--json`
* `--plain`
* `--check` (code de sortie 1 = expiré/manquant, 2 = en cours d&#39;expiration)
* `--probe` (sonde en temps réel des profils d&#39;authentification configurés)
* `--probe-provider <name>`
* `--probe-profile <id>` (répéter ou séparer par des virgules)
* `--probe-timeout <ms>`
* `--probe-concurrency <n>`
* `--probe-max-tokens <n>`

Inclut toujours l’aperçu de l’authentification et l’état d’expiration OAuth pour les profils du store d’authentification.
`--probe` exécute des requêtes en temps réel (peut consommer des jetons et déclencher des limites de débit).

<div id="models-set-model">
  ### `models set <model>`
</div>

Définit `agents.defaults.model.primary`.

<div id="models-set-image-model">
  ### `models set-image <model>`
</div>

Définit `agents.defaults.imageModel.primary`.

<div id="models-aliases-listaddremove">
  ### `models aliases list|add|remove`
</div>

Options :

* `list`: `--json`, `--plain`
* `add <alias> <model>`
* `remove <alias>`

<div id="models-fallbacks-listaddremoveclear">
  ### `models fallbacks list|add|remove|clear`
</div>

Options :

* `list`: `--json`, `--plain`
* `add <model>`
* `remove <model>`
* `clear`

<div id="models-image-fallbacks-listaddremoveclear">
  ### `models image-fallbacks list|add|remove|clear`
</div>

Options :

* `list`: `--json`, `--plain`
* `add <model>`
* `remove <model>`
* `clear`

<div id="models-scan">
  ### `models scan`
</div>

Options :

* `--min-params <b>`
* `--max-age-days <jours>`
* `--provider <nom>`
* `--max-candidates <n>`
* `--timeout <ms>`
* `--concurrency <n>`
* `--no-probe`
* `--yes`
* `--no-input`
* `--set-default`
* `--set-image`
* `--json`

<div id="models-auth-addsetup-tokenpaste-token">
  ### `models auth add|setup-token|paste-token`
</div>

Options :

* `add` : assistant d’authentification interactif
* `setup-token` : `--provider <name>` (par défaut : `anthropic`), `--yes`
* `paste-token` : `--provider <name>`, `--profile-id <id>`, `--expires-in <duration>`

<div id="models-auth-order-getsetclear">
  ### `models auth order get|set|clear`
</div>

Options :

* `get` : `--provider <name>`, `--agent <id>`, `--json`
* `set` : `--provider <name>`, `--agent <id>`, `<profileIds...>`
* `clear` : `--provider <name>`, `--agent <id>`

<div id="system">
  ## Système
</div>

<div id="system-event">
  ### `system event`
</div>

Mettre en file d&#39;attente un événement système et, éventuellement, déclencher un signal de vie (Gateway RPC).

Requis :

* `--text <text>`

Options :

* `--mode <now|next-heartbeat>`
* `--json`
* `--url`, `--token`, `--timeout`, `--expect-final`

<div id="system-heartbeat-lastenabledisable">
  ### `system heartbeat last|enable|disable`
</div>

Contrôle du signal de vie (Gateway RPC).

Options :

* `--json`
* `--url`, `--token`, `--timeout`, `--expect-final`

<div id="system-presence">
  ### `system presence`
</div>

Liste les entrées de présence du système (RPC du Gateway).

Options :

* `--json`
* `--url`, `--token`, `--timeout`, `--expect-final`

<div id="cron">
  ## Cron
</div>

Gérer les tâches planifiées (Gateway RPC). Voir [/automation/cron-jobs](/fr/automation/cron-jobs).

Sous-commandes :

* `cron status [--json]`
* `cron list [--all] [--json]` (sortie sous forme de tableau par défaut ; utilisez `--json` pour la sortie brute)
* `cron add` (alias : `create` ; nécessite `--name` et exactement un de `--at` | `--every` | `--cron`, et exactement un type de charge utile parmi `--system-event` | `--message`)
* `cron edit <id>` (modifie les champs)
* `cron rm <id>` (alias : `remove`, `delete`)
* `cron enable <id>`
* `cron disable <id>`
* `cron runs --id <id> [--limit <n>]`
* `cron run <id> [--force]`

Toutes les commandes `cron` acceptent `--url`, `--token`, `--timeout`, `--expect-final`.

<div id="node-host">
  ## Hôte de nœud
</div>

`node` exécute un **hôte de nœud sans interface graphique** ou le gère en tant que service en tâche de fond. Voir
[`openclaw node`](/fr/cli/node).

Sous-commandes :

* `node run --host <gateway-host> --port 18789`
* `node status`
* `node install [--host <gateway-host>] [--port <port>] [--tls] [--tls-fingerprint <sha256>] [--node-id <id>] [--display-name <name>] [--runtime <node|bun>] [--force]`
* `node uninstall`
* `node stop`
* `node restart`

<div id="nodes">
  ## Nœuds
</div>

`nodes` communique avec le Gateway et cible les nœuds appairés. Voir [/nodes](/fr/nodes).

Options courantes :

* `--url`, `--token`, `--timeout`, `--json`

Sous-commandes :

* `nodes status [--connected] [--last-connected <duration>]`
* `nodes describe --node <id|name|ip>`
* `nodes list [--connected] [--last-connected <duration>]`
* `nodes pending`
* `nodes approve <requestId>`
* `nodes reject <requestId>`
* `nodes rename --node <id|name|ip> --name <displayName>`
* `nodes invoke --node <id|name|ip> --command <command> [--params <json>] [--invoke-timeout <ms>] [--idempotency-key <key>]`
* `nodes run --node <id|name|ip> [--cwd <path>] [--env KEY=VAL] [--command-timeout <ms>] [--needs-screen-recording] [--invoke-timeout <ms>] <command...>` (nœud macOS ou hôte de nœud sans interface graphique)
* `nodes notify --node <id|name|ip> [--title <text>] [--body <text>] [--sound <name>] [--priority <passive|active|timeSensitive>] [--delivery <system|overlay|auto>] [--invoke-timeout <ms>]` (mac uniquement)

Caméra :

* `nodes camera list --node <id|name|ip>`
* `nodes camera snap --node <id|name|ip> [--facing front|back|both] [--device-id <id>] [--max-width <px>] [--quality <0-1>] [--delay-ms <ms>] [--invoke-timeout <ms>]`
* `nodes camera clip --node <id|name|ip> [--facing front|back] [--device-id <id>] [--duration <ms|10s|1m>] [--no-audio] [--invoke-timeout <ms>]`

Canvas + écran :

* `nodes canvas snapshot --node <id|name|ip> [--format png|jpg|jpeg] [--max-width <px>] [--quality <0-1>] [--invoke-timeout <ms>]`
* `nodes canvas present --node <id|name|ip> [--target <urlOrPath>] [--x <px>] [--y <px>] [--width <px>] [--height <px>] [--invoke-timeout <ms>]`
* `nodes canvas hide --node <id|name|ip> [--invoke-timeout <ms>]`
* `nodes canvas navigate <url> --node <id|name|ip> [--invoke-timeout <ms>]`
* `nodes canvas eval [<js>] --node <id|name|ip> [--js <code>] [--invoke-timeout <ms>]`
* `nodes canvas a2ui push --node <id|name|ip> (--jsonl <path> | --text <text>) [--invoke-timeout <ms>]`
* `nodes canvas a2ui reset --node <id|name|ip> [--invoke-timeout <ms>]`
* `nodes screen record --node <id|name|ip> [--screen <index>] [--duration <ms|10s>] [--fps <n>] [--no-audio] [--out <path>] [--invoke-timeout <ms>]`

Localisation :

* `nodes location get --node <id|name|ip> [--max-age <ms>] [--accuracy <coarse|balanced|precise>] [--location-timeout <ms>] [--invoke-timeout <ms>]`

<div id="browser">
  ## Navigateur
</div>

CLI de contrôle du navigateur (Chrome/Brave/Edge/Chromium dédiés). Voir [`openclaw browser`](/fr/cli/browser) et l’[outil Browser](/fr/tools/browser).

Options courantes :

* `--url`, `--token`, `--timeout`, `--json`
* `--browser-profile <name>`

Gérer :

* `browser status`
* `browser start`
* `browser stop`
* `browser reset-profile`
* `browser tabs`
* `browser open <url>`
* `browser focus <targetId>`
* `browser close [targetId]`
* `browser profiles`
* `browser create-profile --name <name> [--color <hex>] [--cdp-url <url>]`
* `browser delete-profile --name <name>`

Inspecter :

* `browser screenshot [targetId] [--full-page] [--ref <ref>] [--element <selector>] [--type png|jpeg]`
* `browser snapshot [--format aria|ai] [--target-id <id>] [--limit <n>] [--interactive] [--compact] [--depth <n>] [--selector <sel>] [--out <path>]`

Actions :

* `browser navigate <url> [--target-id <id>]`
* `browser resize <width> <height> [--target-id <id>]`
* `browser click <ref> [--double] [--button <left|right|middle>] [--modifiers <csv>] [--target-id <id>]`
* `browser type <ref> <text> [--submit] [--slowly] [--target-id <id>]`
* `browser press <key> [--target-id <id>]`
* `browser hover <ref> [--target-id <id>]`
* `browser drag <startRef> <endRef> [--target-id <id>]`
* `browser select <ref> <values...> [--target-id <id>]`
* `browser upload <paths...> [--ref <ref>] [--input-ref <ref>] [--element <selector>] [--target-id <id>] [--timeout-ms <ms>]`
* `browser fill [--fields <json>] [--fields-file <path>] [--target-id <id>]`
* `browser dialog --accept|--dismiss [--prompt <text>] [--target-id <id>] [--timeout-ms <ms>]`
* `browser wait [--time <ms>] [--text <value>] [--text-gone <value>] [--target-id <id>]`
* `browser evaluate --fn <code> [--ref <ref>] [--target-id <id>]`
* `browser console [--level <error|warn|info>] [--target-id <id>]`
* `browser pdf [--target-id <id>]`

<div id="docs-search">
  ## Recherche dans la documentation
</div>

<div id="docs-query">
  ### `docs [query...]`
</div>

Rechercher dans l’index de la documentation en ligne.

## TUI

<div id="tui">
  ### `tui`
</div>

Ouvre l&#39;UI en mode terminal connectée au Gateway.

Options :

* `--url <url>`
* `--token <token>`
* `--password <password>`
* `--session <key>`
* `--deliver`
* `--thinking <level>`
* `--message <text>`
* `--timeout-ms <ms>` (par défaut : `agents.defaults.timeoutSeconds`)
* `--history-limit <n>`