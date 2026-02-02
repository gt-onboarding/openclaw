---
title: Assistant d'onboarding
summary: "Assistant CLI d'onboarding : configuration guidée du Gateway, de l'espace de travail, des canaux et des compétences"
read_when:
  - Lorsque vous exécutez ou configurez l'assistant d'onboarding
  - Lors de la configuration d'une nouvelle machine
---

<div id="onboarding-wizard-cli">
  # Assistant d’onboarding (CLI)
</div>

L’assistant d’onboarding est la méthode **recommandée** pour installer et configurer OpenClaw sur macOS,
Linux ou Windows (via WSL2, fortement recommandé).
Il configure une instance Gateway locale ou une connexion à une instance Gateway distante, ainsi que les canaux, les compétences
et les valeurs par défaut de l’espace de travail, le tout au sein d’un parcours guidé unique.

Point d’entrée principal :

```bash
openclaw onboard
```

Chat initial le plus rapide : ouvrez le Control UI (aucune configuration de canal nécessaire). Exécutez
`openclaw dashboard` et discutez depuis le navigateur. Docs : [Dashboard](/fr/web/dashboard).

Reconfiguration ultérieure :

```bash
openclaw configure
```

Recommandé : configurez une clé API Brave Search afin que l’agent puisse utiliser `web_search`
(`web_fetch` fonctionne sans clé). Le moyen le plus simple : `openclaw configure --section web`
qui enregistre `tools.web.search.apiKey`. Docs : [Outils web](/fr/tools/web).

<div id="quickstart-vs-advanced">
  ## QuickStart vs Advanced
</div>

L’assistant démarre avec **QuickStart** (valeurs par défaut) ou **Advanced** (contrôle complet).

**QuickStart** garde les valeurs par défaut :

* Gateway locale (loopback)
* Espace de travail par défaut (ou espace de travail existant)
* Port de la Gateway : **18789**
* Authentification de la Gateway par **Token** (généré automatiquement, même sur loopback)
* Exposition Tailscale **Off**
* Les messages privés Telegram et WhatsApp utilisent par défaut une **liste d’autorisation** (vous devrez saisir votre numéro de téléphone)

**Advanced** expose chaque étape (mode, espace de travail, Gateway, canaux, démon, compétences).

<div id="what-the-wizard-does">
  ## Ce que fait l’assistant de configuration
</div>

Le **mode local (par défaut)** vous guide à travers les étapes suivantes :

* Modèle/authentification (abonnement OpenAI Code (Codex) via OAuth, clé API Anthropic (recommandée) ou setup-token (à coller), plus options MiniMax/GLM/Moonshot/AI Gateway)
* Emplacement de l’espace de travail + fichiers de bootstrap
* Paramètres du Gateway (port/bind/auth/tailscale)
* Fournisseurs (Telegram, WhatsApp, Discord, Google Chat, Mattermost (plugin), Signal)
* Installation du démon (LaunchAgent / unité utilisateur systemd)
* Vérification de l’état (health check)
* Compétences (recommandées)

Le **mode distant** ne configure que le client local pour se connecter à un Gateway distant.
Il **n’installe** ni ne modifie rien sur l’hôte distant.

Pour ajouter des agents plus isolés (espace de travail + sessions + auth séparés), utilisez :

```bash
openclaw agents add <name>
```

Astuce : `--json` n’implique **pas** le mode non interactif. Utilisez `--non-interactive` (et `--workspace`) pour les scripts.

<div id="flow-details-local">
  ## Détails du flux (local)
</div>

1. **Détection de configuration existante**
   * Si `~/.openclaw/openclaw.json` existe, choisissez **Conserver / Modifier / Réinitialiser**.
   * Relancer l’assistant **ne** supprime rien, sauf si vous choisissez explicitement **Réinitialiser**
     (ou passez `--reset`).
   * Si la configuration est invalide ou contient des clés obsolètes, l’assistant s’arrête et vous demande
     d’exécuter `openclaw doctor` avant de continuer.
   * La réinitialisation utilise `trash` (jamais `rm`) et propose des portées :
     * Config uniquement
     * Config + identifiants + sessions
     * Réinitialisation complète (supprime aussi l’espace de travail)

2. **Modèle/Auth**
   * **Clé API Anthropic (recommandée)** : utilise `ANTHROPIC_API_KEY` si elle est définie ou demande une clé, puis l’enregistre pour l’utilisation par le démon.
   * **Anthropic OAuth (Claude Code CLI)** : sur macOS, l’assistant vérifie l’élément de trousseau &quot;Claude Code-credentials&quot; (choisissez &quot;Toujours autoriser&quot; pour que les démarrages launchd ne soient pas bloqués) ; sur Linux/Windows, il réutilise `~/.claude/.credentials.json` si présent.
   * **Jeton Anthropic (coller setup-token)** : exécutez `claude setup-token` sur n’importe quelle machine, puis collez le jeton (vous pouvez le nommer ; vide = valeur par défaut).
   * **Abonnement OpenAI Code (Codex) (CLI Codex)** : si `~/.codex/auth.json` existe, l’assistant peut le réutiliser.
   * **Abonnement OpenAI Code (Codex) (OAuth)** : flux via le navigateur ; collez le `code#state`.
     * Définit `agents.defaults.model` sur `openai-codex/gpt-5.2` quand le modèle n’est pas défini ou qu’il vaut `openai/*`.
   * **Clé API OpenAI** : utilise `OPENAI_API_KEY` si elle est définie ou demande une clé, puis l’enregistre dans `~/.openclaw/.env` afin que launchd puisse la lire.
   * **OpenCode Zen (proxy multi‑modèles)** : demande `OPENCODE_API_KEY` (ou `OPENCODE_ZEN_API_KEY`, à obtenir sur https://opencode.ai/auth).
   * **Clé API** : enregistre la clé pour vous.
   * **Vercel AI Gateway (proxy multi‑modèles)** : demande `AI_GATEWAY_API_KEY`.
   * Plus de détails : [Vercel AI Gateway](/fr/providers/vercel-ai-gateway)
   * **MiniMax M2.1** : la configuration est générée automatiquement.
   * Plus de détails : [MiniMax](/fr/providers/minimax)
   * **Synthetic (compatible Anthropic)** : demande `SYNTHETIC_API_KEY`.
   * Plus de détails : [Synthetic](/fr/providers/synthetic)
   * **Moonshot (Kimi K2)** : la configuration est générée automatiquement.
   * **Kimi Code** : la configuration est générée automatiquement.
   * Plus de détails : [Moonshot AI (Kimi + Kimi Code)](/fr/providers/moonshot)
   * **Ignorer** : aucune authentification n’est configurée pour l’instant.
   * Choisissez un modèle par défaut parmi les options détectées (ou saisissez fournisseur/modèle manuellement).
   * L’assistant exécute une vérification du modèle et vous avertit si le modèle configuré est inconnu ou si l’authentification est manquante.

* Les identifiants OAuth sont stockés dans `~/.openclaw/credentials/oauth.json` ; les profils d’authentification sont stockés dans `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` (clés API + OAuth).
  * Plus de détails : [/concepts/oauth](/fr/concepts/oauth)

3. **Espace de travail**
   * `~/.openclaw/workspace` par défaut (paramétrable).
   * Initialise les fichiers d’espace de travail nécessaires pour le rituel de bootstrap de l’agent.
   * Disposition complète de l’espace de travail + guide de sauvegarde : [Agent workspace](/fr/concepts/agent-workspace)

4. **Gateway**
   * Port, liaison, mode d’authentification, exposition via Tailscale.
   * Recommandation d’auth : conserver **Token** même pour le loopback afin que les clients WS locaux doivent s’authentifier.
   * Ne désactivez l’authentification que si vous faites entièrement confiance à tous les processus locaux.
   * Les liaisons non‑loopback exigent toujours une authentification.

5. **Canaux**
   * [WhatsApp](/fr/channels/whatsapp) : connexion via QR facultative.
   * [Telegram](/fr/channels/telegram) : jeton de bot.
   * [Discord](/fr/channels/discord) : jeton de bot.
   * [Google Chat](/fr/channels/googlechat) : JSON de compte de service + audience du webhook.
   * [Mattermost](/fr/channels/mattermost) (plugin) : jeton de bot + URL de base.
   * [Signal](/fr/channels/signal) : installation facultative de `signal-cli` + configuration du compte.
   * [iMessage](/fr/channels/imessage) : chemin local du CLI `imsg` + accès à la base de données.
   * Sécurité des DM : par défaut, c’est l’appairage. Le premier DM envoie un code ; approuvez via `openclaw pairing approve <channel> <code>` ou utilisez des listes d’autorisation.

6. **Installation du démon**
   * macOS : LaunchAgent
     * Nécessite une session utilisateur ouverte ; pour un environnement sans écran (headless), utilisez un LaunchDaemon personnalisé (non fourni).
   * Linux (et Windows via WSL2) : unité systemd utilisateur
     * L’assistant tente d’activer le « lingering » via `loginctl enable-linger <user>` pour que le Gateway reste actif après la déconnexion.
     * Peut demander sudo (écrit dans `/var/lib/systemd/linger`) ; il essaie d’abord sans sudo.
   * **Sélection du runtime :** Node (recommandé ; requis pour WhatsApp/Telegram). Bun est **déconseillé**.

7. **Vérification d’état**
   * Démarre le Gateway (si nécessaire) et exécute `openclaw health`.
   * Astuce : `openclaw status --deep` ajoute des sondes d’état du Gateway à la sortie de la commande (nécessite un Gateway accessible).

8. **Compétences (recommandé)**
   * Lit les compétences disponibles et vérifie les prérequis.
   * Vous laisse choisir un gestionnaire de paquets : **npm / pnpm** (bun déconseillé).
   * Installe des dépendances facultatives (certaines utilisent Homebrew sur macOS).

9. **Fin**
   * Récapitulatif + prochaines étapes, y compris les applications iOS/Android/macOS pour des fonctionnalités supplémentaires.

* Si aucune interface graphique n’est détectée, l’assistant affiche des instructions de redirection de port SSH pour le Control UI au lieu d’ouvrir un navigateur.
  * Si les ressources du Control UI sont manquantes, l’assistant tente de les construire ; en repli, il utilise `pnpm ui:build` (installe automatiquement les dépendances de l’UI).

<div id="remote-mode">
  ## Mode distant
</div>

Le mode distant configure un client local pour se connecter à un Gateway hébergé ailleurs.

Ce que vous allez configurer :

* URL du Gateway distant (`ws://...`)
* Jeton d’accès si le Gateway distant requiert une authentification (recommandé)

Remarques :

* Aucune installation distante ni modification de service système (daemon) n’est effectuée.
* Si le Gateway est accessible uniquement en loopback, utilisez un tunnel SSH ou un tailnet.
* Indications pour la découverte :
  * macOS : Bonjour (`dns-sd`)
  * Linux : Avahi (`avahi-browse`)

<div id="add-another-agent">
  ## Ajouter un autre agent
</div>

Utilisez `openclaw agents add <name>` pour créer un agent distinct avec son propre espace de travail,
ses sessions et ses profils d’authentification. L’exécution sans `--workspace` lance l’assistant.

Ce que cela configure :

* `agents.list[].name`
* `agents.list[].workspace`
* `agents.list[].agentDir`

Remarques :

* Les espaces de travail par défaut suivent le schéma `~/.openclaw/workspace-<agentId>`.
* Ajoutez `bindings` pour router les messages entrants (l’assistant peut le faire).
* Options non interactives : `--model`, `--agent-dir`, `--bind`, `--non-interactive`.

<div id="noninteractive-mode">
  ## Mode non interactif
</div>

Utilisez `--non-interactive` pour automatiser ou intégrer l’onboarding dans des scripts :

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

Ajoutez `--json` pour obtenir un résumé interprétable par une machine.

Exemple avec Gemini :

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice gemini-api-key \
  --gemini-api-key "$GEMINI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Exemple Z.AI :

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice zai-api-key \
  --zai-api-key "$ZAI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Exemple avec Vercel AI Gateway :

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Exemple avec Moonshot :

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice moonshot-api-key \
  --moonshot-api-key "$MOONSHOT_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Exemple fictif :

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice synthetic-api-key \
  --synthetic-api-key "$SYNTHETIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Exemple OpenCode Zen :

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice opencode-zen \
  --opencode-zen-api-key "$OPENCODE_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Exemple (non interactif) d’ajout d’agent :

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

<div id="gateway-wizard-rpc">
  ## RPC de l&#39;assistant Gateway
</div>

Le Gateway expose le flux de l&#39;assistant via RPC (`wizard.start`, `wizard.next`, `wizard.cancel`, `wizard.status`).
Les clients (app macOS, Control UI) peuvent afficher les étapes sans avoir à réimplémenter la logique d&#39;onboarding.

<div id="signal-setup-signal-cli">
  ## Configuration de Signal (signal-cli)
</div>

L&#39;assistant peut installer `signal-cli` à partir des publications GitHub :

* Télécharge la ressource de publication (asset) appropriée.
* La stocke sous `~/.openclaw/tools/signal-cli/<version>/`.
* Écrit `channels.signal.cliPath` dans votre configuration.

Remarques :

* Les builds JVM nécessitent **Java 21**.
* Les builds natives sont utilisées lorsqu’elles sont disponibles.
* Sous Windows, WSL2 est utilisé ; l’installation de signal-cli suit la procédure Linux à l’intérieur de WSL.

<div id="what-the-wizard-writes">
  ## Ce que l’assistant d’onboarding écrit
</div>

Champs typiques dans `~/.openclaw/openclaw.json` :

* `agents.defaults.workspace`
* `agents.defaults.model` / `models.providers` (si Minimax est choisi)
* `gateway.*` (mode, bind, auth, tailscale)
* `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
* Listes d’autorisation de canaux (Slack/Discord/Matrix/Microsoft Teams) lorsque vous les activez pendant les questions interactives (les noms sont résolus en identifiants lorsque c’est possible).
* `skills.install.nodeManager`
* `wizard.lastRunAt`
* `wizard.lastRunVersion`
* `wizard.lastRunCommit`
* `wizard.lastRunCommand`
* `wizard.lastRunMode`

`openclaw agents add` écrit `agents.list[]` et éventuellement `bindings`.

Les informations d’identification WhatsApp sont stockées sous `~/.openclaw/credentials/whatsapp/<accountId>/`.
Les sessions sont stockées sous `~/.openclaw/agents/<agentId>/sessions/`.

Certains canaux sont fournis sous forme de plugins. Quand vous en choisissez un pendant l’onboarding, l’assistant
vous demandera de l’installer (npm ou un chemin local) avant de pouvoir le configurer.

<div id="related-docs">
  ## Documentation associée
</div>

* Onboarding de l’app macOS : [Onboarding](/fr/start/onboarding)
* Référence de configuration : [Configuration du Gateway](/fr/gateway/configuration)
* Fournisseurs : [WhatsApp](/fr/channels/whatsapp), [Telegram](/fr/channels/telegram), [Discord](/fr/channels/discord), [Google Chat](/fr/channels/googlechat), [Signal](/fr/channels/signal), [iMessage](/fr/channels/imessage)
* Compétences : [Compétences](/fr/tools/skills), [Configuration des compétences](/fr/tools/skills-config)