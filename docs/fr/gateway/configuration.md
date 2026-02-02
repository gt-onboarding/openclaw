---
title: Configuration
summary: "Toutes les options de configuration pour ~/.openclaw/openclaw.json avec des exemples"
read_when:
  - Pour ajouter ou modifier des champs de configuration
---

<div id="configuration">
  # Configuration 🔧
</div>

OpenClaw lit une configuration **JSON5** optionnelle depuis `~/.openclaw/openclaw.json` (commentaires + virgules finales autorisés).

Si le fichier est absent, OpenClaw utilise des valeurs par défaut relativement sûres (agent Pi intégré + sessions par expéditeur + espace de travail `~/.openclaw/workspace`). Vous n&#39;avez généralement besoin d&#39;un fichier de configuration que pour :

* restreindre qui peut déclencher le bot (`channels.whatsapp.allowFrom`, `channels.telegram.allowFrom`, etc.)
* contrôler les listes d’autorisation de groupes + le comportement des mentions (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.discord.guilds`, `agents.list[].groupChat`)
* personnaliser les préfixes de messages (`messages`)
* définir l&#39;espace de travail de l&#39;agent (`agents.defaults.workspace` ou `agents.list[].workspace`)
* ajuster les valeurs par défaut de l&#39;agent intégré (`agents.defaults`) et le comportement de session (`session`)
* définir l&#39;identité de chaque agent (`agents.list[].identity`)

> **Vous débutez avec la configuration ?** Consultez le guide [Exemples de configuration](/fr/gateway/configuration-examples) pour des exemples complets avec des explications détaillées !

<div id="strict-config-validation">
  ## Validation stricte de la configuration
</div>

OpenClaw n&#39;accepte que les configurations qui correspondent entièrement au schéma.
Les clés inconnues, les types mal formés ou les valeurs invalides font que le Gateway **refuse de démarrer** pour des raisons de sécurité.

Lorsque la validation échoue :

* Le Gateway ne démarre pas.
* Seules les commandes de diagnostic sont autorisées (par exemple : `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`).
* Lancez `openclaw doctor` pour voir les problèmes exacts.
* Lancez `openclaw doctor --fix` (ou `--yes`) pour appliquer les migrations/réparations.

`doctor` n’écrit jamais de modifications à moins que vous n’activiez explicitement `--fix`/`--yes`.

<div id="schema-ui-hints">
  ## Indications de schéma + d’UI
</div>

Le Gateway expose une représentation JSON Schema de la configuration via `config.schema` pour les éditeurs d’UI.
Le Control UI génère un formulaire à partir de ce schéma, avec un éditeur **Raw JSON** comme solution de repli.

Les plugins de canal et les extensions peuvent enregistrer des indications de schéma + d’UI pour leur configuration, afin que les paramètres de canal
restent pilotés par le schéma sur l’ensemble des applications, sans formulaires codés en dur.

Les indications (libellés, groupements, champs sensibles) sont fournies avec le schéma pour que les clients puissent générer
de meilleurs formulaires sans coder en dur la logique de configuration.

<div id="apply-restart-rpc">
  ## Appliquer + redémarrer (RPC)
</div>

Utilisez `config.apply` pour valider et écrire l’intégralité de la configuration, puis redémarrer le Gateway en une seule étape.
Cette commande écrit un « restart sentinel » et envoie un ping à la dernière session active après le redémarrage du Gateway.

Avertissement : `config.apply` remplace **toute la configuration**. Si vous voulez modifier seulement quelques clés,
utilisez `config.patch` ou `openclaw config set`. Conservez une sauvegarde de `~/.openclaw/openclaw.json`.

Paramètres :

* `raw` (string) — charge utile JSON5 pour l’ensemble de la configuration
* `baseHash` (optional) — hachage de configuration renvoyé par `config.get` (requis lorsqu’une configuration existe déjà)
* `sessionKey` (optional) — clé de la dernière session active pour le ping de réveil
* `note` (optional) — note à inclure dans le « restart sentinel »
* `restartDelayMs` (optional) — délai avant le redémarrage (par défaut 2000)

Exemple (via `gateway call`) :

```bash
openclaw gateway call config.get --params '{}' # capturer payload.hash
openclaw gateway call config.apply --params '{
  "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

<div id="partial-updates-rpc">
  ## Mises à jour partielles (RPC)
</div>

Utilisez `config.patch` pour fusionner une mise à jour partielle dans la configuration existante sans écraser
les clés non concernées. Il applique la sémantique « JSON merge patch » :

* les objets sont fusionnés récursivement
* `null` supprime une clé
* les tableaux remplacent la valeur existante
  Comme `config.apply`, il valide, écrit la configuration, enregistre une sentinelle de redémarrage et planifie
  le redémarrage du Gateway (avec un réveil optionnel lorsque `sessionKey` est fourni).

Paramètres :

* `raw` (string) — payload JSON5 contenant uniquement les clés à modifier
* `baseHash` (required) — hachage de configuration renvoyé par `config.get`
* `sessionKey` (optional) — clé de la dernière session active pour le ping de réveil
* `note` (optional) — note à inclure dans la sentinelle de redémarrage
* `restartDelayMs` (optional) — délai avant le redémarrage (2000 par défaut)

Exemple :

```bash
openclaw gateway call config.get --params '{}' # capturer payload.hash
openclaw gateway call config.patch --params '{
  "raw": "{\\n  channels: { telegram: { groups: { \\"*\\": { requireMention: false } } } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

<div id="minimal-config-recommended-starting-point">
  ## Configuration minimale (recommandée comme point de départ)
</div>

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } }
}
```

Construisez l’image par défaut une fois avec :

```bash
scripts/sandbox-setup.sh
```

<div id="self-chat-mode-recommended-for-group-control">
  ## mode conversation privée (recommandé pour la gestion des groupes)
</div>

Pour empêcher le bot de répondre aux @-mentions WhatsApp dans les groupes (et ne répondre qu’à des déclencheurs textuels spécifiques) :

```json5
{
  agents: {
    defaults: { workspace: "~/.openclaw/workspace" },
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["@openclaw", "reisponde"] }
      }
    ]
  },
  channels: {
    whatsapp: {
      // La liste d'autorisation ne concerne que les messages directs ; inclure votre propre numéro active le mode self-chat.
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } }
    }
  }
}
```

<div id="config-includes-include">
  ## Inclusions de configuration (`$include`)
</div>

Divisez votre configuration en plusieurs fichiers à l&#39;aide de la directive `$include`. Cette approche est utile pour :

* Structurer des configurations volumineuses (par exemple, définitions d&#39;agents par client)
* Partager des paramètres communs entre différents environnements
* Garder les configurations sensibles séparées

<div id="basic-usage">
  ### Utilisation de base
</div>

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  
  // Include a single file (replaces the key's value)
  agents: { "$include": "./agents.json5" },
  
  // Inclure plusieurs fichiers (fusion profonde dans l'ordre)
  broadcast: { 
    "$include": [
      "./clients/mueller.json5",
      "./clients/schmidt.json5"
    ]
  }
}
```

```json5
// ~/.openclaw/agents.json5
{
  defaults: { sandbox: { mode: "all", scope: "session" } },
  list: [
    { id: "main", workspace: "~/.openclaw/workspace" }
  ]
}
```

<div id="merge-behavior">
  ### Comportement de fusion
</div>

* **Fichier unique** : Remplace l’objet contenant `$include`
* **Tableau de fichiers** : Fusion profonde des fichiers dans l’ordre (les fichiers suivants écrasent les précédents)
* **Avec des clés au même niveau** : Ces clés sont fusionnées après les includes (elles écrasent les valeurs incluses)
* **Clés au même niveau + tableaux/primitifs** : Non pris en charge (le contenu inclus doit être un objet)

```json5
// Sibling keys override included values
{
  "$include": "./base.json5",   // { a: 1, b: 2 }
  b: 99                          // Résultat : { a: 1, b: 99 }
}
```

<div id="nested-includes">
  ### Inclusions imbriquées
</div>

Les fichiers inclus peuvent eux-mêmes contenir des directives `$include` (jusqu&#39;à 10 niveaux d’imbrication) :

```json5
// clients/mueller.json5
{
  agents: { "$include": "./mueller/agents.json5" },
  broadcast: { "$include": "./mueller/broadcast.json5" }
}
```

<div id="path-resolution">
  ### Résolution des chemins
</div>

* **Chemins relatifs** : Résolus par rapport au fichier qui les inclut
* **Chemins absolus** : Utilisés tels quels
* **Répertoires parents** : Les références `../` fonctionnent comme prévu

```json5
{ "$include": "./sub/config.json5" }      // relative
{ "$include": "/etc/openclaw/base.json5" } // absolute
{ "$include": "../shared/common.json5" }   // répertoire parent
```

<div id="error-handling">
  ### Gestion des erreurs
</div>

* **Fichier manquant** : erreur explicite avec le chemin résolu
* **Erreur d&#39;analyse** : indique quel fichier inclus a provoqué l&#39;erreur
* **Inclusions circulaires** : détectées et signalées avec la chaîne d&#39;inclusions

<div id="example-multi-client-legal-setup">
  ### Exemple : configuration juridique pour plusieurs clients
</div>

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789, auth: { token: "secret" } },
  
  // Common agent defaults
  agents: {
    defaults: {
      sandbox: { mode: "all", scope: "session" }
    },
    // Fusionner les listes d'agents de tous les clients
    list: { "$include": [
      "./clients/mueller/agents.json5",
      "./clients/schmidt/agents.json5"
    ]}
  },
  
  // Merge broadcast configs
  broadcast: { "$include": [
    "./clients/mueller/broadcast.json5",
    "./clients/schmidt/broadcast.json5"
  ]},
  
  channels: { whatsapp: { groupPolicy: "allowlist" } }
}
```

```json5
// ~/.openclaw/clients/mueller/agents.json5
[
  { id: "mueller-transcribe", workspace: "~/clients/mueller/transcribe" },
  { id: "mueller-docs", workspace: "~/clients/mueller/docs" }
]
```

```json5
// ~/.openclaw/clients/mueller/broadcast.json5
{
  "120363403215116621@g.us": ["mueller-transcribe", "mueller-docs"]
}
```

<div id="common-options">
  ## Options communes
</div>

<div id="env-vars-env">
  ### Variables d&#39;environnement + `.env`
</div>

OpenClaw lit les variables d&#39;environnement du processus parent (shell, launchd/systemd, CI, etc.).

Il charge également :

* `.env` depuis le répertoire de travail courant (s&#39;il existe)
* un fichier `.env` global de repli depuis `~/.openclaw/.env` (aussi appelé `$OPENCLAW_STATE_DIR/.env`)

Aucun des fichiers `.env` n&#39;écrase les variables d&#39;environnement déjà existantes.

Vous pouvez également définir des variables d&#39;environnement directement dans la configuration. Elles ne sont appliquées que si
la variable d&#39;environnement du processus ne définit pas encore cette clé (même règle de non-écrasement) :

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-..."
    }
  }
}
```

Voir [/environment](/fr/environment) pour l’ordre de priorité complet et les différentes sources.

<div id="envshellenv-optional">
  ### `env.shellEnv` (optionnel)
</div>

Option pratique à activer explicitement : si elle est activée et qu’aucune des clés attendues n’est encore définie, OpenClaw exécute votre shell de connexion et importe uniquement les clés attendues manquantes (sans jamais les écraser).
Cela revient essentiellement à « sourcer » votre profil de shell.

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000
    }
  }
}
```

Équivalent via variables d&#39;environnement :

* `OPENCLAW_LOAD_SHELL_ENV=1`
* `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

<div id="env-var-substitution-in-config">
  ### Substitution des variables d&#39;environnement dans la configuration
</div>

Vous pouvez faire référence à des variables d&#39;environnement directement dans n&#39;importe quelle valeur de chaîne de la configuration en utilisant la syntaxe
`${VAR_NAME}`. Les variables sont substituées au moment du chargement de la configuration, avant la validation.

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}"
      }
    }
  },
  gateway: {
    auth: {
      token: "${OPENCLAW_GATEWAY_TOKEN}"
    }
  }
}
```

**Règles :**

* Seuls les noms de variables d’environnement en majuscules sont pris en compte : `[A-Z_][A-Z0-9_]*`
* Les variables d’environnement manquantes ou vides provoquent une erreur au chargement de la configuration
* Échapper avec `$${VAR}` pour produire littéralement `${VAR}`
* Fonctionne avec `$include` (les fichiers inclus bénéficient également de la substitution)

**Substitution en ligne :**

```json5
{
  models: {
    providers: {
      custom: {
        baseUrl: "${CUSTOM_API_BASE}/v1"  // → "https://api.example.com/v1"
      }
    }
  }
}
```

<div id="auth-storage-oauth-api-keys">
  ### Stockage de l’authentification (OAuth + clés API)
</div>

OpenClaw stocke les profils d’authentification **par agent** (OAuth + clés API) dans :

* `<agentDir>/auth-profiles.json` (par défaut : `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`)

Voir aussi : [/concepts/oauth](/fr/concepts/oauth)

Imports OAuth hérités :

* `~/.openclaw/credentials/oauth.json` (ou `$OPENCLAW_STATE_DIR/credentials/oauth.json`)

L’agent Pi embarqué maintient un cache d’exécution dans :

* `<agentDir>/auth.json` (géré automatiquement ; ne pas modifier manuellement)

Répertoire d’agent hérité (avant le multi-agent) :

* `~/.openclaw/agent/*` (migré par `openclaw doctor` vers `~/.openclaw/agents/<defaultAgentId>/agent/*`)

Surcharges :

* Répertoire OAuth (import hérité uniquement) : `OPENCLAW_OAUTH_DIR`
* Répertoire d’agent (remplacement de la racine de l’agent par défaut) : `OPENCLAW_AGENT_DIR` (préféré), `PI_CODING_AGENT_DIR` (hérité)

Lors de la première utilisation, OpenClaw importe les entrées de `oauth.json` dans `auth-profiles.json`.

<div id="auth">
  ### `auth`
</div>

Métadonnées facultatives pour les profils d’authentification. Ces métadonnées **ne** stockent **pas** de secrets ; elles associent des IDs de profil à un fournisseur + mode (et éventuellement une adresse e-mail) et définissent l’ordre de rotation des fournisseurs utilisé pour le basculement en cas de défaillance.

```json5
{
  auth: {
    profiles: {
      "anthropic:me@example.com": { provider: "anthropic", mode: "oauth", email: "me@example.com" },
      "anthropic:work": { provider: "anthropic", mode: "api_key" }
    },
    order: {
      anthropic: ["anthropic:me@example.com", "anthropic:work"]
    }
  }
}
```

<div id="agentslistidentity">
  ### `agents.list[].identity`
</div>

Identité optionnelle propre à chaque agent, utilisée pour les valeurs par défaut et l’expérience utilisateur (UX). Celle-ci est définie par l’assistant d’onboarding macOS.

Si elle est définie, OpenClaw en déduit des valeurs par défaut (uniquement lorsque vous ne les avez pas définies explicitement) :

* `messages.ackReaction` à partir de `identity.emoji` de l’**agent actif** (avec valeur de repli 👀)
* `agents.list[].groupChat.mentionPatterns` à partir de `identity.name`/`identity.emoji` de l’agent (afin que « @Samantha » fonctionne dans les groupes sur Telegram/Slack/Discord/Google Chat/iMessage/WhatsApp)
* `identity.avatar` accepte un chemin d’image relatif à l’espace de travail ou une URL distante ou de type data URL. Les fichiers locaux doivent se trouver à l’intérieur de l’espace de travail de l’agent.

`identity.avatar` accepte :

* Chemin relatif à l’espace de travail (doit rester dans l’espace de travail de l’agent)
* URL `http(s)`
* URI `data:`

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Samantha",
          theme: "paresseux utile",
          emoji: "🦥",
          avatar: "avatars/samantha.png"
        }
      }
    ]
  }
}
```

<div id="wizard">
  ### `wizard`
</div>

Métadonnées générées par les assistants de la CLI (`onboard`, `configure`, `doctor`).

```json5
{
  wizard: {
    lastRunAt: "2026-01-01T00:00:00.000Z",
    lastRunVersion: "2026.1.4",
    lastRunCommit: "abc1234",
    lastRunCommand: "configure",
    lastRunMode: "local"
  }
}
```

<div id="logging">
  ### `logging`
</div>

* Fichier journal par défaut : `/tmp/openclaw/openclaw-YYYY-MM-DD.log`
* Si vous voulez un chemin stable, définissez `logging.file` sur `/tmp/openclaw/openclaw.log`.
* La sortie console peut être configurée séparément via :
  * `logging.consoleLevel` (valeur par défaut : `info`, passe à `debug` avec `--verbose`)
  * `logging.consoleStyle` (`pretty` | `compact` | `json`)
* Les résumés d’outils peuvent être expurgés pour éviter les fuites de secrets :
  * `logging.redactSensitive` (`off` | `tools`, par défaut : `tools`)
  * `logging.redactPatterns` (tableau de chaînes regex ; remplace les valeurs par défaut)

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty",
    redactSensitive: "tools",
    redactPatterns: [
      // Exemple : remplacer les valeurs par défaut par vos propres règles.
      "\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1",
      "/\\bsk-[A-Za-z0-9_-]{8,}\\b/gi"
    ]
  }
}
```

<div id="channelswhatsappdmpolicy">
  ### `channels.whatsapp.dmPolicy`
</div>

Contrôle la façon dont les conversations privées WhatsApp (DM) sont gérées :

* `"pairing"` (valeur par défaut) : les expéditeurs inconnus reçoivent un code d’appairage ; le propriétaire doit approuver
* `"allowlist"` : n’autorise que les expéditeurs figurant dans `channels.whatsapp.allowFrom` (ou dans un store d’autorisations appairé)
* `"open"` : permet tous les DM entrants (le paramètre `open` autorise l’acceptation sans restriction des messages de tout utilisateur, **nécessite** que `channels.whatsapp.allowFrom` inclue `"*"`)
* `"disabled"` : ignore tous les DM entrants

Les codes d’appairage expirent après 1 heure ; le bot n’envoie un code d’appairage que lorsqu’une nouvelle demande est créée. Les demandes d’appairage de DM en attente sont limitées à **3 par canal** par défaut.

Approbation d’appairage :

* `openclaw pairing list whatsapp`
* `openclaw pairing approve whatsapp <code>`

<div id="channelswhatsappallowfrom">
  ### `channels.whatsapp.allowFrom`
</div>

Liste d’autorisation de numéros de téléphone E.164 pouvant déclencher des réponses automatiques WhatsApp (**messages privés uniquement**).
Si cette liste est vide et que `channels.whatsapp.dmPolicy="pairing"`, les expéditeurs inconnus recevront un code d’appairage.
Pour les groupes, utilisez `channels.whatsapp.groupPolicy` + `channels.whatsapp.groupAllowFrom`.

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000, // optional outbound chunk size (chars)
      chunkMode: "length", // mode de découpage optionnel (length | newline)
      mediaMaxMb: 50 // optional inbound media cap (MB)
    }
  }
}
```

<div id="channelswhatsappsendreadreceipts">
  ### `channels.whatsapp.sendReadReceipts`
</div>

Détermine si les messages WhatsApp entrants sont marqués comme lus (double coche bleue). Valeur par défaut : `true`.

Le mode self-chat ignore toujours les accusés de lecture, même lorsqu’ils sont activés.

Remplacement par compte : `channels.whatsapp.accounts.<id>.sendReadReceipts`.

```json5
{
  channels: {
    whatsapp: { sendReadReceipts: false }
  }
}
```

<div id="channelswhatsappaccounts-multi-account">
  ### `channels.whatsapp.accounts` (multi-compte)
</div>

Utilisez plusieurs comptes WhatsApp dans un seul Gateway :

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {}, // optional; keeps the default id stable
        personal: {},
        biz: {
          // Remplacement optionnel. Par défaut : ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        }
      }
    }
  }
}
```

Notes :

* Les commandes sortantes utilisent par défaut le compte `default` s’il est présent ; sinon, c’est l’identifiant du premier compte configuré (par ordre de tri) qui est utilisé.
* L’ancien répertoire d’authentification Baileys pour compte unique est migré par `openclaw doctor` vers `whatsapp/default`.

<div id="channelstelegramaccounts-channelsdiscordaccounts-channelsgooglechataccounts-channelsslackaccounts-channelsmattermostaccounts-channelssignalaccounts-channelsimessageaccounts">
  ### `channels.telegram.accounts` / `channels.discord.accounts` / `channels.googlechat.accounts` / `channels.slack.accounts` / `channels.mattermost.accounts` / `channels.signal.accounts` / `channels.imessage.accounts`
</div>

Utilisez plusieurs comptes par canal (chaque compte possède son propre `accountId` et un `name` optionnel) :

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC..."
        },
        alerts: {
          name: "Alerts bot",
          botToken: "987654:XYZ..."
        }
      }
    }
  }
}
```

Remarques :

* `default` est utilisé lorsque `accountId` est omis (CLI + routage).
* Les jetons d’environnement ne s’appliquent qu’au compte **default**.
* Les paramètres de base du canal (politique de groupe, filtrage par mention, etc.) s’appliquent à tous les comptes, sauf s’ils sont redéfinis par compte.
* Utilisez `bindings[].match.accountId` pour router chaque compte vers un `agents.defaults` différent.

<div id="group-chat-mention-gating-agentslistgroupchat-messagesgroupchat">
  ### Contrôle d’accès par mention en discussion de groupe (`agents.list[].groupChat` + `messages.groupChat`)
</div>

Par défaut, les messages de groupe **exigent une mention** (soit une mention via métadonnées, soit via expressions régulières). S’applique aux discussions de groupe sur WhatsApp, Telegram, Discord, Google Chat et iMessage.

**Types de mention :**

* **Mentions via métadonnées** : @‑mentions natives de la plateforme (par exemple, appui pour @‑mentionner sur WhatsApp). Ignorées dans le mode de discussion avec soi-même de WhatsApp (voir `channels.whatsapp.allowFrom`).
* **Motifs textuels** : expressions régulières définies dans `agents.list[].groupChat.mentionPatterns`. Toujours vérifiées, quel que soit le mode de discussion avec soi-même.
* Le contrôle d’accès par mention n’est appliqué que lorsque la détection de mention est possible (mentions natives ou au moins un `mentionPattern`).

```json5
{
  messages: {
    groupChat: { historyLimit: 50 }
  },
  agents: {
    list: [
      { id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }
    ]
  }
}
```

`messages.groupChat.historyLimit` définit la valeur par défaut globale pour le contexte d’historique des conversations de groupe. Les canaux peuvent la redéfinir avec `channels.<channel>.historyLimit` (ou `channels.<channel>.accounts.*.historyLimit` pour les configurations multi‑compte). Définissez `0` pour désactiver la troncature automatique de l’historique.

<div id="dm-history-limits">
  #### Limites de l’historique des DM
</div>

Les conversations en DM utilisent un historique par session, géré par l’agent. Vous pouvez limiter le nombre d’échanges utilisateur conservés par session de DM :

```json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30,  // limit DM sessions to 30 user turns
      dms: {
        "123456789": { historyLimit: 50 }  // remplacement par utilisateur (ID utilisateur)
      }
    }
  }
}
```

Ordre de résolution :

1. Surcharge par DM : `channels.<provider>.dms[userId].historyLimit`
2. Valeur par défaut du fournisseur : `channels.<provider>.dmHistoryLimit`
3. Aucune limite (tout l’historique est conservé)

Fournisseurs pris en charge : `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

Surcharge par agent (prioritaire lorsqu’elle est définie, même `[]`) :

```json5
{
  agents: {
    list: [
      { id: "work", groupChat: { mentionPatterns: ["@workbot", "\\+15555550123"] } },
      { id: "personal", groupChat: { mentionPatterns: ["@homebot", "\\+15555550999"] } }
    ]
  }
}
```

Les valeurs par défaut de la restriction par mention s’appliquent par canal (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.imessage.groups`, `channels.discord.guilds`). Lorsque `*.groups` est défini, il fonctionne également comme liste d’autorisation de groupes ; incluez `"*"` pour autoriser tous les groupes.

Pour répondre **uniquement** à des déclencheurs textuels spécifiques (en ignorant les @‑mentions natives) :

```json5
{
  channels: {
    whatsapp: {
      // Incluez votre propre numéro pour activer le mode auto-discussion (ignore les @-mentions natives).
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } }
    }
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          // Only these text patterns will trigger responses
          mentionPatterns: ["reisponde", "@openclaw"]
        }
      }
    ]
  }
}
```

<div id="group-policy-per-channel">
  ### Politique de groupe (par canal)
</div>

Utilisez `channels.*.groupPolicy` pour contrôler si les messages de groupe/salon sont acceptés ou non :

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"]
    },
    telegram: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["tg:123456789", "@alice"]
    },
    signal: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"]
    },
    imessage: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["chat_id:123"]
    },
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"]
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "GUILD_ID": {
          channels: { help: { allow: true } }
        }
      }
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } }
    }
  }
}
```

Notes :

* `"open"` : les groupes contournent la liste d’autorisation ; le filtrage par mention reste appliqué ; ce paramètre permet d’accepter sans restriction les messages de n’importe quel utilisateur.
* `"disabled"` : bloque tous les messages de groupe/salon.
* `"allowlist"` : autorise uniquement les groupes/salons correspondant à la liste d’autorisation configurée.
* `channels.defaults.groupPolicy` définit la valeur par défaut quand le `groupPolicy` d’un fournisseur n’est pas défini.
* WhatsApp/Telegram/Signal/iMessage/Microsoft Teams utilisent `groupAllowFrom` (valeur de repli : `allowFrom` explicite).
* Discord/Slack utilisent des listes d’autorisation de canaux (`channels.discord.guilds.*.channels`, `channels.slack.channels`).
* Les messages privés de groupe (Discord/Slack) restent contrôlés par `dm.groupEnabled` + `dm.groupChannels`.
* Par défaut, `groupPolicy: "allowlist"` (sauf si remplacé par `channels.defaults.groupPolicy`) ; si aucune liste d’autorisation n’est configurée, les messages de groupe sont bloqués.

<div id="multi-agent-routing-agentslist-bindings">
  ### Routage multi-agent (`agents.list` + `bindings`)
</div>

Exécuter plusieurs agents isolés (espace de travail séparé, `agentDir`, sessions) au sein d’un seul Gateway.
Les messages entrants sont routés vers un agent via des bindings.

* `agents.list[]` : surcharges par agent.
  * `id` : identifiant d’agent stable (obligatoire).
  * `default` : optionnel ; s’il y en a plusieurs, le premier l’emporte et un avertissement est consigné dans les logs.
    Si aucun n’est défini, la **première entrée** de la liste est l’agent par défaut.
  * `name` : nom d’affichage pour l’agent.
  * `workspace` : valeur par défaut `~/.openclaw/workspace-<agentId>` (pour `main`, revient à `agents.defaults.workspace`).
  * `agentDir` : valeur par défaut `~/.openclaw/agents/<agentId>/agent`.
  * `model` : modèle par défaut pour l’agent, remplace `agents.defaults.model` pour cet agent.
    * forme chaîne : `"provider/model"`, remplace uniquement `agents.defaults.model.primary`
    * forme objet : `{ primary, fallbacks }` (les fallbacks remplacent `agents.defaults.model.fallbacks` ; `[]` désactive les fallbacks globaux pour cet agent)
  * `identity` : nom/thème/emoji par agent (utilisé pour les motifs de mention + réactions d’accusé de réception).
  * `groupChat` : filtrage par mention par agent (`mentionPatterns`).
  * `sandbox` : configuration de sandbox par agent (remplace `agents.defaults.sandbox`).
    * `mode` : `"off"` | `"non-main"` | `"all"`
    * `workspaceAccess` : `"none"` | `"ro"` | `"rw"`
    * `scope` : `"session"` | `"agent"` | `"shared"`
    * `workspaceRoot` : racine personnalisée de l’espace de travail du sandbox
    * `docker` : surcharges Docker par agent (par ex. `image`, `network`, `env`, `setupCommand`, limites ; ignoré lorsque `scope: "shared"`)
    * `browser` : surcharges de navigateur en sandbox par agent (ignoré lorsque `scope: "shared"`)
    * `prune` : surcharges de nettoyage du sandbox par agent (ignoré lorsque `scope: "shared"`)
  * `subagents` : valeurs par défaut de sous-agents par agent.
    * `allowAgents` : liste d’autorisation d’identifiants d’agent pour `sessions_spawn` depuis cet agent (`["*"]` = autoriser n’importe lequel ; par défaut : uniquement le même agent)
  * `tools` : restrictions d’outils par agent (appliquées avant la politique d’outils du sandbox).
    * `profile` : profil d’outils de base (appliqué avant allow/deny)
    * `allow` : tableau de noms d’outils autorisés
    * `deny` : tableau de noms d’outils refusés (deny l’emporte)
* `agents.defaults` : valeurs par défaut partagées pour les agents (modèle, espace de travail, sandbox, etc.).
* `bindings[]` : routent les messages entrants vers un `agentId`.
  * `match.channel` (obligatoire)
  * `match.accountId` (optionnel ; `*` = n’importe quel compte ; omis = compte par défaut)
  * `match.peer` (optionnel ; `{ kind: dm|group|channel, id }`)
  * `match.guildId` / `match.teamId` (optionnel ; spécifique au canal)

Ordre de correspondance déterministe :

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (exact, sans peer/guild/team)
5. `match.accountId: "*"` (au niveau du canal, sans peer/guild/team)
6. agent par défaut (`agents.list[].default`, sinon première entrée de la liste, sinon `"main"`)

À l’intérieur de chaque niveau de correspondance, la première entrée correspondante dans `bindings` l’emporte.

<div id="per-agent-access-profiles-multi-agent">
  #### Profils d&#39;accès par agent (multi-agents)
</div>

Chaque agent peut avoir sa propre sandbox et sa propre politique d’outils. Utilisez cela pour mélanger les niveaux d&#39;accès dans un même Gateway :

* **Accès complet** (agent personnel)
* Outils en lecture seule + espace de travail
* **Aucun accès au système de fichiers** (outils de messagerie/de session uniquement)

Voir [Sandbox et outils multi-agents](/fr/multi-agent-sandbox-tools) pour l&#39;ordre de priorité et
des exemples supplémentaires.

Accès complet (sans sandbox) :

```json5
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: { mode: "off" }
      }
    ]
  }
}
```

Outils en lecture seule + espace de travail en lecture seule :

```json5
{
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "ro"
        },
        tools: {
          allow: ["read", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
          deny: ["write", "edit", "apply_patch", "exec", "process", "browser"]
        }
      }
    ]
  }
}
```

Aucun accès au système de fichiers (outils de messagerie et de session activés) :

```json5
{
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "none"
        },
        tools: {
          allow: ["sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status", "whatsapp", "telegram", "slack", "discord", "gateway"],
          deny: ["read", "write", "edit", "apply_patch", "exec", "process", "browser", "canvas", "nodes", "cron", "gateway", "image"]
        }
      }
    ]
  }
}
```

Exemple : deux comptes WhatsApp → deux agents

```json5
{
  agents: {
    list: [
      { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
      { id: "work", workspace: "~/.openclaw/workspace-work" }
    ]
  },
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } }
  ],
  channels: {
    whatsapp: {
      accounts: {
        personal: {},
        biz: {},
      }
    }
  }
}
```

<div id="toolsagenttoagent-optional">
  ### `tools.agentToAgent` (optionnel)
</div>

La messagerie entre agents nécessite une activation explicite :

```json5
{
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"]
    }
  }
}
```

<div id="messagesqueue">
  ### `messages.queue`
</div>

Contrôle le comportement des messages entrants lorsqu&#39;une exécution d&#39;agent est déjà en cours.

```json5
{
  messages: {
    queue: {
      mode: "collect", // steer | followup | collect | steer-backlog (steer+backlog ok) | interrupt (queue=steer ancien)
      debounceMs: 1000,
      cap: 20,
      drop: "summarize", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
        discord: "collect",
        imessage: "collect",
        webchat: "collect"
      }
    }
  }
}
```

<div id="messagesinbound">
  ### `messages.inbound`
</div>

Regroupe les messages entrants rapides provenant du **même expéditeur** afin que plusieurs messages successifs deviennent un seul tour d’agent. Ce regroupement est défini par couple canal + conversation et utilise le message le plus récent pour le chaînage des réponses et les identifiants.

```json5
{
  messages: {
    inbound: {
      debounceMs: 2000, // 0 pour désactiver
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500
      }
    }
  }
}
```

Notes :

* Applique un regroupement avec délai uniquement aux messages **texte** ; les médias/pièces jointes sont envoyés immédiatement.
* Les commandes de contrôle (par ex. `/queue`, `/new`) ignorent ce délai afin de rester indépendantes.

<div id="commands-chat-command-handling">
  ### `commands` (gestion des commandes de chat)
</div>

Définit la manière dont les commandes de chat sont activées sur les différents connecteurs.

```json5
{
  commands: {
    native: "auto",         // register native commands when supported (auto)
    text: true,             // parse slash commands in chat messages
    bash: false,            // autoriser ! (alias : /bash) (hôte uniquement ; nécessite des listes d'autorisation tools.elevated)
    bashForegroundMs: 2000, // bash foreground window (0 backgrounds immediately)
    config: false,          // allow /config (writes to disk)
    debug: false,           // allow /debug (runtime-only overrides)
    restart: false,         // allow /restart + gateway restart tool
    useAccessGroups: true   // enforce access-group allowlists/policies for commands
  }
}
```

Notes :

* Les commandes textuelles doivent être envoyées dans un message **isolé** et utiliser le préfixe `/` (aucun alias en texte brut).
* `commands.text: false` désactive l’analyse des messages de discussion pour y détecter des commandes.
* `commands.native: "auto"` (par défaut) active les commandes natives pour Discord/Telegram et les laisse désactivées pour Slack ; les canaux non pris en charge restent limités au texte.
* Définissez `commands.native: true|false` pour tout activer/désactiver globalement, ou surchargez par canal avec `channels.discord.commands.native`, `channels.telegram.commands.native`, `channels.slack.commands.native` (booléen ou `"auto"`). `false` efface les commandes précédemment enregistrées sur Discord/Telegram au démarrage ; les commandes Slack sont gérées dans l’application Slack.
* `channels.telegram.customCommands` ajoute des entrées supplémentaires au menu du bot Telegram. Les noms sont normalisés ; les conflits avec les commandes natives sont ignorés.
* `commands.bash: true` active `! <cmd>` pour exécuter des commandes shell sur l’hôte (`/bash <cmd>` fonctionne aussi comme alias). Nécessite `tools.elevated.enabled` et l’ajout de l’expéditeur à la liste d’autorisation dans `tools.elevated.allowFrom.<channel>`.
* `commands.bashForegroundMs` contrôle la durée pendant laquelle bash attend avant de passer en arrière-plan. Tant qu’une tâche bash est en cours, les nouvelles requêtes `! <cmd>` sont rejetées (une à la fois).
* `commands.config: true` active `/config` (lit/écrit `openclaw.json`).
* `channels.<provider>.configWrites` régit les modifications de configuration initiées par ce canal (par défaut : true). Cela s’applique à `/config set|unset` ainsi qu’aux migrations automatiques spécifiques au fournisseur (changements d’ID de supergroupe Telegram, changements d’ID de canal Slack).
* `commands.debug: true` active `/debug` (surcharges limitées au runtime).
* `commands.restart: true` active `/restart` et l’action de redémarrage de l’outil Gateway.
* `commands.useAccessGroups: false` permet aux commandes de contourner les listes d’autorisation et politiques des groupes d’accès.
* Les commandes slash et directives ne sont prises en compte que pour les **expéditeurs autorisés**. L’autorisation est dérivée des listes d’autorisation/de l’appairage du canal, plus `commands.useAccessGroups`.

<div id="web-whatsapp-web-channel-runtime">
  ### `web` (runtime du canal web WhatsApp)
</div>

WhatsApp s’exécute via le canal web du Gateway (Baileys Web). Il démarre automatiquement lorsqu’il existe une session liée.
Définissez `web.enabled: false` pour le laisser désactivé par défaut.

```json5
{
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0
    }
  }
}
```

<div id="channelstelegram-bot-transport">
  ### `channels.telegram` (transport de bot)
</div>

OpenClaw ne démarre Telegram que lorsqu&#39;une section de configuration `channels.telegram` est présente. Le jeton du bot est récupéré à partir de `channels.telegram.botToken` (ou `channels.telegram.tokenFile`), avec `TELEGRAM_BOT_TOKEN` comme solution de repli pour le compte par défaut.
Définissez `channels.telegram.enabled: false` pour désactiver le démarrage automatique.
La prise en charge multi‑compte se trouve sous `channels.telegram.accounts` (voir la section multi‑compte ci‑dessus). Les jetons fournis via des variables d&#39;environnement ne s&#39;appliquent qu&#39;au compte par défaut.
Définissez `channels.telegram.configWrites: false` pour bloquer les écritures de configuration initiées par Telegram (y compris les migrations d&#39;ID de supergroupe et `/config set|unset`).

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing",                 // pairing | allowlist | open | disabled
      allowFrom: ["tg:123456789"],         // optional; "open" requires ["*"]
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Keep answers brief.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Stay on topic."
            }
          }
        }
      },
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" }
      ],
      historyLimit: 50,                     // include last N group messages as context (0 disables)
      replyToMode: "first",                 // off | first | all
      linkPreview: true,                   // toggle outbound link previews
      streamMode: "partial",               // off | partial | block (streaming de brouillon ; distinct du streaming par blocs)
      draftChunk: {                        // optional; only for streamMode=block
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph"       // paragraph | newline | sentence
      },
      actions: { reactions: true, sendMessage: true }, // tool action gates (false disables)
      reactionNotifications: "own",   // off | own | all
      mediaMaxMb: 5,
      retry: {                             // outbound retry policy
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1
      },
      network: {                           // transport overrides
        autoSelectFamily: false
      },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook"
    }
  }
}
```

Notes sur le streaming de brouillons :

* Utilise `sendMessageDraft` de Telegram (bulle de brouillon, pas un vrai message).
* Nécessite des **sujets de discussion privés** (`message_thread_id` en DM/messages privés ; le bot a les sujets activés).
* `/reasoning stream` diffuse le raisonnement dans le brouillon, puis envoie la réponse finale.
  Les valeurs par défaut et le comportement de la stratégie de réessai sont documentés dans [Stratégie de réessai](/fr/concepts/retry).

<div id="channelsdiscord-bot-transport">
  ### `channels.discord` (transport du bot)
</div>

Configure le bot Discord en définissant le jeton du bot et, éventuellement, les mécanismes de contrôle d’accès :
La prise en charge multi-compte se trouve sous `channels.discord.accounts` (voir la section multi-compte ci-dessus). Les jetons définis via les variables d’environnement ne s’appliquent qu’au compte par défaut.

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8,                          // clamp inbound media size
      allowBots: false,                       // allow bot-authored messages
      actions: {                              // tool action gates (false disables)
        reactions: true,
        stickers: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        voiceStatus: true,
        events: true,
        moderation: false
      },
      replyToMode: "off",                     // off | first | all
      dm: {
        enabled: true,                        // disable all DMs when false
        policy: "pairing",                    // pairing | allowlist | open | disabled
        allowFrom: ["1234567890", "steipete"], // optional DM allowlist ("open" requires ["*"])
        groupEnabled: false,                 // enable group DMs
        groupChannels: ["openclaw-dm"]          // optional group DM allowlist
      },
      guilds: {
        "123456789012345678": {               // guild id (preferred) or slug
          slug: "friends-of-openclaw",
          requireMention: false,              // per-guild default
          reactionNotifications: "own",       // off | own | all | allowlist
          users: ["987654321098765432"],      // optional per-guild user allowlist
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Short answers only."
            }
          }
        }
      },
      historyLimit: 20,                       // include last N guild messages as context
      textChunkLimit: 2000,                   // optional outbound text chunk size (chars)
      chunkMode: "length",                    // optional chunking mode (length | newline)
      maxLinesPerMessage: 17,                 // limite souple de lignes par message (troncature UI Discord)
      retry: {                                // outbound retry policy
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1
      }
    }
  }
}
```

OpenClaw ne démarre Discord que lorsqu’une section de configuration `channels.discord` existe. Le jeton est obtenu à partir de `channels.discord.token`, avec `DISCORD_BOT_TOKEN` comme valeur de repli pour le compte par défaut (sauf si `channels.discord.enabled` vaut `false`). Utilisez `user:&lt;id&gt;` (DM) ou `channel:&lt;id&gt;` (salon de guilde) lorsque vous spécifiez des cibles de diffusion pour les commandes cron/CLI ; les ID numériques seuls sont ambigus et rejetés.
Les slugs de guildes sont en minuscules avec les espaces remplacés par `-` ; les clés de salons utilisent le nom du salon transformé en slug (sans `#` initial). Privilégiez les ID de guildes comme clés pour éviter les ambiguïtés en cas de renommage.
Les messages émis par des bots sont ignorés par défaut. Activez-les avec `channels.discord.allowBots` (les propres messages du bot restent filtrés pour empêcher les boucles d’auto-réponse).
Modes de notification pour les réactions :

* `off` : aucun événement de réaction.
* `own` : réactions sur les propres messages du bot (par défaut).
* `all` : toutes les réactions sur tous les messages.
* `allowlist` : réactions depuis `guilds.&lt;id&gt;.users` sur tous les messages (une liste vide désactive ce mode).
  Le texte sortant est découpé en segments par `channels.discord.textChunkLimit` (2000 par défaut). Définissez `channels.discord.chunkMode="newline"` pour fractionner sur les lignes vides (limites de paragraphes) avant le découpage par longueur. Les clients Discord peuvent tronquer les messages très longs, donc `channels.discord.maxLinesPerMessage` (17 par défaut) scinde les longues réponses multi‑lignes même en dessous de 2000 caractères.
  Les valeurs par défaut et le comportement de la stratégie de réessai sont documentés dans [Retry policy](/fr/concepts/retry).

<div id="channelsgooglechat-chat-api-webhook">
  ### `channels.googlechat` (webhook de l&#39;API Chat)
</div>

Google Chat fonctionne via des webhooks HTTP avec une authentification au niveau de l&#39;application (compte de service).
La prise en charge de plusieurs comptes se trouve sous `channels.googlechat.accounts` (voir la section multi‑compte ci‑dessus). Les variables d&#39;environnement ne s&#39;appliquent qu&#39;au compte par défaut.

```json5
{
  channels: {
    "googlechat": {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url",             // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890",        // optional; improves mention detection
      dm: {
        enabled: true,
        policy: "pairing",                // pairing (appairage) | allowlist (liste d'autorisation) | open (ouvert à tous) | disabled (désactivé)
        allowFrom: ["users/1234567890"]   // optional; "open" requires ["*"]
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true }
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20
    }
  }
}
```

Notes :

* Le JSON du compte de service peut être fourni en ligne (`serviceAccount`) ou via un fichier (`serviceAccountFile`).
* Variables d’environnement de repli pour le compte par défaut : `GOOGLE_CHAT_SERVICE_ACCOUNT` ou `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
* `audienceType` + `audience` doivent correspondre à la configuration d’authentification du webhook de l’application Chat.
* Utilisez `spaces/&lt;spaceId&gt;` ou `users/&lt;userId|email&gt;` pour définir les cibles d’envoi.

<div id="channelsslack-socket-mode">
  ### `channels.slack` (Socket Mode)
</div>

Slack fonctionne en Socket Mode et nécessite à la fois un jeton de bot et un jeton d&#39;application :

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["U123", "U456", "*"], // optional; "open" requires ["*"]
        groupEnabled: false,
        groupChannels: ["G123"]
      },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "Short answers only."
        }
      },
      historyLimit: 50,          // inclure les N derniers messages du canal/groupe comme contexte (0 désactive)
      allowBots: false,
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["U123"],
      replyToMode: "off",           // off | first | all
      thread: {
        historyScope: "thread",     // thread | channel
        inheritParent: false
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true
      },
      textChunkLimit: 4000,
      chunkMode: "length",
      mediaMaxMb: 20
    }
  }
}
```

La prise en charge multi-comptes se trouve sous `channels.slack.accounts` (voir la section multi-comptes ci-dessus). Les jetons d’environnement ne s’appliquent qu’au compte par défaut.

OpenClaw démarre Slack lorsque le fournisseur est activé et que les deux jetons sont définis (via la config ou `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN`). Utilisez `user:<id>` (DM) ou `channel:<id>` pour spécifier les cibles de livraison des commandes cron/CLI.
Définissez `channels.slack.configWrites: false` pour bloquer les écritures de configuration initiées par Slack (y compris les migrations d’ID de canal et `/config set|unset`).

Les messages écrits par le bot sont ignorés par défaut. Activez-les avec `channels.slack.allowBots` ou `channels.slack.channels.<id>.allowBots`.

Modes de notification des réactions :

* `off` : aucun événement de réaction.
* `own` : réactions sur les propres messages du bot (par défaut).
* `all` : toutes les réactions sur tous les messages.
* `allowlist` : réactions provenant de `channels.slack.reactionAllowlist` sur tous les messages (une liste vide désactive cette option).

Isolation des sessions de fil de discussion :

* `channels.slack.thread.historyScope` contrôle si l’historique du fil est propre à chaque fil (`thread`, par défaut) ou partagé à l’échelle du canal (`channel`).
* `channels.slack.thread.inheritParent` contrôle si les nouvelles sessions de fil héritent de la transcription du canal parent (par défaut : false).

Groupes d’actions Slack (contrôlent les actions de l’outil `slack`) :

| Groupe d’actions | Valeur par défaut | Notes                           |
| ---------------- | ----------------- | ------------------------------- |
| reactions        | enabled           | Réagir + lister les réactions   |
| messages         | enabled           | Lire/envoyer/modifier/supprimer |
| pins             | enabled           | Épingler/désépingler/lister     |
| memberInfo       | enabled           | Informations sur les membres    |
| emojiList        | enabled           | Liste des émojis personnalisés  |

<div id="channelsmattermost-bot-token">
  ### `channels.mattermost` (jeton de bot)
</div>

Mattermost est fourni sous forme de plugin et n&#39;est pas inclus dans l&#39;installation de base.
Installez-le d&#39;abord : `openclaw plugins install @openclaw/mattermost` (ou `./extensions/mattermost` depuis un clone Git).

Mattermost nécessite un jeton de bot ainsi que l&#39;URL de base de votre serveur :

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      textChunkLimit: 4000,
      chunkMode: "length"
    }
  }
}
```

OpenClaw démarre Mattermost lorsque le compte est configuré (bot token + URL de base) et activé. Le token et l’URL de base sont résolus à partir de `channels.mattermost.botToken` et `channels.mattermost.baseUrl` ou de `MATTERMOST_BOT_TOKEN` et `MATTERMOST_URL` pour le compte par défaut (sauf si `channels.mattermost.enabled` est à `false`).

Modes de chat :

* `oncall` (par défaut) : répondre aux messages du canal uniquement lorsqu’il y a une @mention.
* `onmessage` : répondre à chaque message du canal.
* `onchar` : répondre lorsqu’un message commence par un préfixe de déclenchement (`channels.mattermost.oncharPrefixes`, par défaut `[">", "!"]`).

Contrôle d’accès :

* DMs par défaut : `channels.mattermost.dmPolicy="pairing"` (les expéditeurs inconnus reçoivent un code d’appairage).
* DMs publics : `channels.mattermost.dmPolicy="open"` plus `channels.mattermost.allowFrom=["*"]`.
* Groupes : `channels.mattermost.groupPolicy="allowlist"` par défaut (géré par mention). Utilisez `channels.mattermost.groupAllowFrom` pour restreindre les expéditeurs.

La gestion multi-comptes se trouve sous `channels.mattermost.accounts` (voir la section multi-comptes ci-dessus). Les variables d’environnement ne s’appliquent qu’au compte par défaut.
Utilisez `channel:<id>` ou `user:<id>` (ou `@username`) pour spécifier les cibles de livraison ; les identifiants simples sont traités comme des identifiants de canal.

<div id="channelssignal-signal-cli">
  ### `channels.signal` (signal-cli)
</div>

Les réactions dans Signal peuvent émettre des événements système (outils partagés pour les réactions) :

```json5
{
  channels: {
    signal: {
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50 // inclut les N derniers messages de groupe comme contexte (0 pour désactiver)
    }
  }
}
```

Modes de notification des réactions :

* `off` : aucun événement de réaction.
* `own` : réactions sur les propres messages du bot (par défaut).
* `all` : toutes les réactions sur tous les messages.
* `allowlist` : réactions provenant de `channels.signal.reactionAllowlist` sur tous les messages (liste vide = désactivation).

<div id="channelsimessage-imsg-cli">
  ### `channels.imessage` (CLI imsg)
</div>

OpenClaw démarre `imsg rpc` (JSON-RPC sur l’entrée/sortie standard, stdio). Aucun démon ni port requis.

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host", // SCP pour les pièces jointes distantes lors de l'utilisation d'un wrapper SSH
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50,    // include last N group messages as context (0 disables)
      includeAttachments: false,
      mediaMaxMb: 16,
      service: "auto",
      region: "US"
    }
  }
}
```

La prise en charge de plusieurs comptes se trouve sous `channels.imessage.accounts` (voir la section multi‑compte ci‑dessus).

Notes :

* Nécessite l’accès complet au disque pour la base de données Messages.
* Le premier envoi affichera une demande d’autorisation d’automatisation Messages.
* Privilégiez les cibles `chat_id:<id>`. Utilisez `imsg chats --limit 20` pour lister les discussions.
* `channels.imessage.cliPath` peut pointer vers un script wrapper (par exemple `ssh` vers un autre Mac qui exécute `imsg rpc`) ; utilisez des clés SSH pour éviter les demandes de mot de passe.
* Pour les wrappers SSH distants, définissez `channels.imessage.remoteHost` pour récupérer les pièces jointes via SCP lorsque `includeAttachments` est activé.

Exemple de wrapper :

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

<div id="agentsdefaultsworkspace">
  ### `agents.defaults.workspace`
</div>

Définit le **répertoire global unique de l’espace de travail** utilisé par l’agent pour les opérations sur les fichiers.

Valeur par défaut : `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } }
}
```

Si `agents.defaults.sandbox` est activé, les sessions non principales peuvent surcharger ce paramètre avec leurs propres espaces de travail par portée sous `agents.defaults.sandbox.workspaceRoot`.

<div id="agentsdefaultsreporoot">
  ### `agents.defaults.repoRoot`
</div>

Racine du dépôt facultative à afficher dans la ligne Runtime de l’invite système. Si elle n’est pas définie, OpenClaw
essaie de détecter un répertoire `.git` en remontant l’arborescence à partir de l’espace de travail (et du répertoire de travail
actuel). Le chemin doit exister pour pouvoir être utilisé.

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } }
}
```

<div id="agentsdefaultsskipbootstrap">
  ### `agents.defaults.skipBootstrap`
</div>

Désactive la création automatique des fichiers de bootstrap de l’espace de travail (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md` et `BOOTSTRAP.md`).

Utilisez cette option pour les déploiements préconfigurés où les fichiers de votre espace de travail proviennent d’un dépôt.

```json5
{
  agents: { defaults: { skipBootstrap: true } }
}
```

<div id="agentsdefaultsbootstrapmaxchars">
  ### `agents.defaults.bootstrapMaxChars`
</div>

Nombre maximal de caractères de chaque fichier de bootstrap de l’espace de travail injectés dans l’invite système
avant troncature. Valeur par défaut : `20000`.

Lorsqu’un fichier dépasse cette limite, OpenClaw consigne un avertissement dans les journaux et injecte une
version tronquée, composée du début et de la fin du fichier, avec un marqueur.

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } }
}
```

<div id="agentsdefaultsusertimezone">
  ### `agents.defaults.userTimezone`
</div>

Définit le fuseau horaire de l’utilisateur pour le **contexte du prompt système** (et non pour les horodatages des enveloppes de message). Si ce paramètre n’est pas défini, OpenClaw utilise le fuseau horaire de l’hôte au moment de l’exécution.

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } }
}
```

<div id="agentsdefaultstimeformat">
  ### `agents.defaults.timeFormat`
</div>

Contrôle le **format de l’heure** affiché dans la section « Current Date &amp; Time » de l’invite système.
Valeur par défaut : `auto` (préférence du système d’exploitation).

```json5
{
  agents: { defaults: { timeFormat: "auto" } } // auto | 12 | 24
}
```

<div id="messages">
  ### `messages`
</div>

Contrôle les préfixes d’entrée/sortie et les réactions d’accusé de réception (ack) facultatives.
Voir [Messages](/fr/concepts/messages) pour tout ce qui concerne la mise en file d’attente, les sessions et le streaming.

```json5
{
  messages: {
    responsePrefix: "🦞", // ou "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions",
    removeAckAfterReply: false
  }
}
```

`responsePrefix` est appliqué à **toutes les réponses sortantes** (résumés d’outils, diffusion en blocs, réponses finales) sur l’ensemble des canaux, sauf s’il est déjà présent.

Si `messages.responsePrefix` n’est pas défini, aucun préfixe n’est appliqué par défaut. Les réponses dans une conversation WhatsApp avec vous‑même constituent l’exception : elles utilisent par défaut `[{identity.name}]` lorsqu’il est défini, sinon
`[openclaw]`, afin que les conversations sur le même téléphone restent lisibles.
Réglez‑le sur `"auto"` pour dériver `[{identity.name}]` pour l’agent routé (lorsqu’il est défini).

<div id="template-variables">
  #### Variables de modèle
</div>

La chaîne `responsePrefix` peut inclure des variables de modèle qui sont résolues dynamiquement :

| Variable          | Description                   | Exemple                      |
| ----------------- | ----------------------------- | ---------------------------- |
| `{model}`         | Nom court du modèle           | `claude-opus-4-5`, `gpt-4o`  |
| `{modelFull}`     | Identifiant complet du modèle | `anthropic/claude-opus-4-5`  |
| `{provider}`      | Nom du fournisseur            | `anthropic`, `openai`        |
| `{thinkingLevel}` | Niveau de réflexion actuel    | `high`, `low`, `off`         |
| `{identity.name}` | Nom d’identité de l’Agent     | (identique au mode `"auto"`) |

Les variables ne sont pas sensibles à la casse (`{MODEL}` = `{model}`). `{think}` est un alias pour `{thinkingLevel}`.
Les variables non résolues restent telles quelles dans le texte.

```json5
{
  messages: {
    responsePrefix: "[{model} | think:{thinkingLevel}]"
  }
}
```

Exemple de sortie : `[claude-opus-4-5 | think:high] Here's my response...`

Le préfixe des messages entrants WhatsApp se configure via `channels.whatsapp.messagePrefix` (obsolète :
`messages.messagePrefix`). La valeur par défaut reste **inchangée** : `"[openclaw]"` lorsque
`channels.whatsapp.allowFrom` est vide, sinon `""` (aucun préfixe). Lorsque vous utilisez
`"[openclaw]"`, OpenClaw utilisera à la place `[{identity.name}]` lorsque l’agent vers lequel le message est routé
a `identity.name` défini.

`ackReaction` envoie, dans la mesure du possible, une réaction emoji pour accuser réception des messages entrants
sur les canaux qui prennent en charge les réactions (Slack/Discord/Telegram/Google Chat). Par défaut, elle utilise
`identity.emoji` de l’agent actif lorsqu’elle est définie, sinon `"👀"`. Définissez-la à `""` pour la désactiver.

`ackReactionScope` contrôle le moment où les réactions sont déclenchées :

* `group-mentions` (par défaut) : uniquement lorsqu’un groupe/salon nécessite des mentions **et** que le bot a été mentionné
* `group-all` : tous les messages de groupe/salon
* `direct` : uniquement les messages directs
* `all` : tous les messages

`removeAckAfterReply` supprime la réaction d’accusé de réception du bot après l’envoi d’une réponse
(Slack/Discord/Telegram/Google Chat uniquement). Valeur par défaut : `false`.

<div id="messagestts">
  #### `messages.tts`
</div>

Active la synthèse vocale (text-to-speech) pour les réponses sortantes. Lorsqu’elle est activée, OpenClaw génère de l’audio à l’aide d’ElevenLabs ou d’OpenAI et le joint aux réponses. Telegram utilise des messages vocaux Opus ; les autres canaux envoient de l’audio MP3.

```json5
{
  messages: {
    tts: {
      auto: "always", // off | always | inbound | tagged
      mode: "final", // final | all (inclure les réponses d'outils/blocs)
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true
      },
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0
        }
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy"
      }
    }
  }
}
```

Notes :

* `messages.tts.auto` contrôle le TTS automatique (`off`, `always`, `inbound`, `tagged`).
* `/tts off|always|inbound|tagged` définit le mode automatique par session (outrepasse la configuration).
* `messages.tts.enabled` est obsolète ; `doctor` le migre vers `messages.tts.auto`.
* `prefsPath` stocke les surcharges locales (fournisseur/limit/summarize).
* `maxTextLength` est une limite stricte pour l’entrée TTS ; les résumés sont tronqués pour rester dans cette limite.
* `summaryModel` remplace `agents.defaults.model.primary` pour le résumé automatique.
  * Accepte `provider/model` ou un alias de `agents.defaults.models`.
* `modelOverrides` active les surcharges pilotées par modèle comme les balises `[[tts:...]]` (activé par défaut).
* `/tts limit` et `/tts summary` contrôlent les paramètres de résumé par utilisateur.
* Les valeurs `apiKey` utilisent par défaut `ELEVENLABS_API_KEY`/`XI_API_KEY` et `OPENAI_API_KEY` si elles ne sont pas définies.
* `elevenlabs.baseUrl` remplace l’URL de base de l’API ElevenLabs.
* `elevenlabs.voiceSettings` prend en charge `stability`/`similarityBoost`/`style` (0..1),
  `useSpeakerBoost` et `speed` (0,5..2,0).

<div id="talk">
  ### `talk`
</div>

Valeurs par défaut pour le mode Talk (macOS/iOS/Android). Les identifiants de voix se rabattent sur `ELEVENLABS_VOICE_ID` ou `SAG_VOICE_ID` lorsqu’ils ne sont pas définis.
`apiKey` se rabat sur `ELEVENLABS_API_KEY` (ou le profil shell du Gateway) lorsqu’il n’est pas défini.
`voiceAliases` permet aux directives Talk d’utiliser des noms plus parlants (par ex. `"voice":"Clawd"`).

```json5
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    voiceAliases: {
      Clawd: "EXAVITQu4vr4xnSDxMaL",
      Roger: "CwhRBWXzGAHq8TQ4Fs17"
    },
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true
  }
}
```

<div id="agentsdefaults">
  ### `agents.defaults`
</div>

Contrôle l’environnement d’exécution intégré de l’agent (modèle/raisonnement/verbeux/timeouts).
`agents.defaults.models` définit le catalogue de modèles configurés (et sert de liste d’autorisation pour `/model`).
`agents.defaults.model.primary` définit le modèle par défaut ; `agents.defaults.model.fallbacks` sont les solutions de repli globales.
`agents.defaults.imageModel` est facultatif et est **utilisé uniquement si le modèle principal ne prend pas en charge l’entrée d’image**.
Chaque entrée de `agents.defaults.models` peut inclure :

* `alias` (raccourci de modèle facultatif, par ex. `/opus`).
* `params` (paramètres d’API spécifiques au fournisseur, facultatifs, transmis tels quels dans la requête de modèle).

`params` est également appliqué aux exécutions en streaming (agent intégré + compaction). Clés actuellement prises en charge : `temperature`, `maxTokens`. Celles‑ci sont fusionnées avec les options fournies à l’appel ; les valeurs fournies par l’appelant prévalent. `temperature` est un réglage avancé : laissez‑le non défini, sauf si vous connaissez les valeurs par défaut du modèle et avez besoin de les modifier.

Exemple :

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-sonnet-4-5-20250929": {
          params: { temperature: 0.6 }
        },
        "openai/gpt-5.2": {
          params: { maxTokens: 8192 }
        }
      }
    }
  }
}
```

Les modèles Z.AI GLM-4.x activent automatiquement le thinking mode, sauf si vous :

* définissez `--thinking off`, ou
* définissez vous‑même `agents.defaults.models["zai/<model>"].params.thinking`.

OpenClaw fournit également quelques alias abrégés intégrés. Les valeurs par défaut ne s’appliquent que lorsque le modèle
est déjà présent dans `agents.defaults.models` :

* `opus` -&gt; `anthropic/claude-opus-4-5`
* `sonnet` -&gt; `anthropic/claude-sonnet-4-5`
* `gpt` -&gt; `openai/gpt-5.2`
* `gpt-mini` -&gt; `openai/gpt-5-mini`
* `gemini` -&gt; `google/gemini-3-pro-preview`
* `gemini-flash` -&gt; `google/gemini-3-flash-preview`

Si vous configurez vous‑même un alias portant le même nom (insensible à la casse), votre valeur a priorité (les valeurs par défaut ne remplacent jamais).

Exemple : Opus 4.5 comme modèle principal avec MiniMax M2.1 en repli (MiniMax hébergé) :

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" }
      },
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: ["minimax/MiniMax-M2.1"]
      }
    }
  }
}
```

Authentification MiniMax : définissez la variable d’environnement `MINIMAX_API_KEY` ou configurez `models.providers.minimax`.

<div id="agentsdefaultsclibackends-cli-fallback">
  #### `agents.defaults.cliBackends` (repli via CLI)
</div>

Backends CLI optionnels pour des exécutions de secours en mode texte uniquement (sans appels d’outils). Ils sont utiles comme
chemin de secours lorsque les fournisseurs d’API échouent. La transmission d’images en transparence est prise en charge lorsque vous configurez
un `imageArg` qui accepte des chemins de fichiers.

Notes :

* Les backends CLI sont **orientés texte en priorité** ; les outils sont toujours désactivés.
* Les sessions sont prises en charge lorsque `sessionArg` est défini ; les identifiants de session sont conservés par backend.
* Pour `claude-cli`, des valeurs par défaut sont déjà intégrées. Redéfinissez le chemin de la commande si `PATH` est minimal
  (launchd/systemd).

Exemple :

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude"
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          modelArg: "--model",
          sessionArg: "--session",
          sessionMode: "existing",
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat"
        }
      }
    }
  }
}
```

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "anthropic/claude-sonnet-4-1": { alias: "Sonnet" },
        "openrouter/deepseek/deepseek-r1:free": {},
        "zai/glm-4.7": {
          alias: "GLM",
          params: {
            thinking: {
              type: "enabled",
              clear_thinking: false
            }
          }
        }
      },
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: [
          "openrouter/deepseek/deepseek-r1:free",
          "openrouter/meta-llama/llama-3.3-70b-instruct:free"
        ]
      },
      imageModel: {
        primary: "openrouter/qwen/qwen-2.5-vl-72b-instruct:free",
        fallbacks: [
          "openrouter/google/gemini-2.0-flash-vision:free"
        ]
      },
      thinkingDefault: "low",
      verboseDefault: "off",
      elevatedDefault: "on",
      timeoutSeconds: 600,
      mediaMaxMb: 5,
      heartbeat: {
        every: "30m",
        target: "last"
      },
      maxConcurrent: 3,
      subagents: {
        model: "minimax/MiniMax-M2.1",
        maxConcurrent: 1,
        archiveAfterMinutes: 60
      },
      exec: {
        backgroundMs: 10000,
        timeoutSec: 1800,
        cleanupMs: 1800000
      },
      contextTokens: 200000
    }
  }
}
```

<div id="agentsdefaultscontextpruning-tool-result-pruning">
  #### `agents.defaults.contextPruning` (élagage des résultats d’outils)
</div>

`agents.defaults.contextPruning` supprime les **anciens résultats d’outils** du contexte en mémoire juste avant qu’une requête ne soit envoyée au LLM.
Il **ne** modifie **pas** l’historique de session sur disque (les fichiers `*.jsonl` restent complets).

L’objectif est de réduire la consommation de jetons pour les agents bavards qui accumulent de gros résultats d’outils au fil du temps.

Vue d’ensemble :

* Ne touche jamais aux messages user/assistant.
* Protège les `keepLastAssistants` derniers messages assistant (aucun résultat d’outil après ce point n’est élagué).
* Protège le préfixe de bootstrap (rien avant le premier message user n’est élagué).
* Modes :
  * `adaptive` : tronque légèrement les résultats d’outils surdimensionnés (garde début/fin) lorsque le ratio de contexte estimé dépasse `softTrimRatio`.
    Puis efface complètement les plus anciens résultats d’outils éligibles lorsque le ratio de contexte estimé dépasse `hardClearRatio` **et**
    qu’il y a suffisamment de volume de résultats d’outils élagables (`minPrunableToolChars`).
  * `aggressive` : remplace toujours les résultats d’outils éligibles avant le seuil de coupure par `hardClear.placeholder` (aucun contrôle de ratio).

Élagage « soft » vs « hard » (ce qui change dans le contexte envoyé au LLM) :

* **Soft-trim** : uniquement pour les résultats d’outils *surdimensionnés*. Garde le début + la fin et insère `...` au milieu.
  * Avant : `toolResult("…very long output…")`
  * Après : `toolResult("HEAD…\n...\n…TAIL\n\n[Tool result trimmed: …]")`
* **Hard-clear** : remplace l’intégralité du résultat d’outil par le placeholder.
  * Avant : `toolResult("…very long output…")`
  * Après : `toolResult("[Old tool result content cleared]")`

Remarques / limitations actuelles :

* Les résultats d’outils contenant des **blocs d’images sont ignorés** (jamais tronqués/effacés) pour l’instant.
* Le « ratio de contexte » estimé est basé sur le **nombre de caractères** (approximation), pas sur les jetons exacts.
* Si la session ne contient pas encore au moins `keepLastAssistants` messages assistant, l’élagage est ignoré.
* En mode `aggressive`, `hardClear.enabled` est ignoré (les résultats d’outils éligibles sont toujours remplacés par `hardClear.placeholder`).

Par défaut : (`adaptive`) :

```json5
{
  agents: { defaults: { contextPruning: { mode: "adaptive" } } }
}
```

Pour désactiver :

```json5
{
  agents: { defaults: { contextPruning: { mode: "off" } } }
}
```

Valeurs par défaut (lorsque `mode` est `"adaptive"` ou `"aggressive"`) :

* `keepLastAssistants`: `3`
* `softTrimRatio`: `0.3` (mode `"adaptive"` uniquement)
* `hardClearRatio`: `0.5` (mode `"adaptive"` uniquement)
* `minPrunableToolChars`: `50000` (mode `"adaptive"` uniquement)
* `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }` (mode `"adaptive"` uniquement)
* `hardClear`: `{ enabled: true, placeholder: "[Old tool result content cleared]" }`

Exemple (mode &quot;aggressive&quot;, configuration minimale) :

```json5
{
  agents: { defaults: { contextPruning: { mode: "aggressive" } } }
}
```

Exemple (réglage adaptatif) :

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "adaptive",
        keepLastAssistants: 3,
        softTrimRatio: 0.3,
        hardClearRatio: 0.5,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { enabled: true, placeholder: "[Old tool result content cleared]" },
        // Optionnel : restreindre l'élagage à des outils spécifiques (deny l'emporte ; prend en charge les jokers "*")
        tools: { deny: ["browser", "canvas"] },
      }
    }
  }
}
```

Voir [/concepts/session-pruning](/fr/concepts/session-pruning) pour plus de détails sur le comportement.

<div id="agentsdefaultscompaction-reserve-headroom-memory-flush">
  #### `agents.defaults.compaction` (réserver de la marge de sécurité + vidage de la mémoire)
</div>

`agents.defaults.compaction.mode` sélectionne la stratégie de synthèse pour la compaction. La valeur par défaut est `default` ; définissez `safeguard` pour activer la synthèse par blocs pour des historiques très longs. Voir [/concepts/compaction](/fr/concepts/compaction).

`agents.defaults.compaction.reserveTokensFloor` impose une valeur minimale de `reserveTokens`
pour la compaction Pi (par défaut : `20000`). Définissez-la à `0` pour désactiver ce plancher.

`agents.defaults.compaction.memoryFlush` exécute un tour d’agent **silencieux** avant
l’auto-compaction, en demandant au modèle de stocker des mémoires persistantes sur le disque (par exemple
`memory/YYYY-MM-DD.md`). Il se déclenche lorsque l’estimation de jetons de la session franchit un
seuil souple situé en dessous de la limite de compaction.

Anciens paramètres par défaut :

* `memoryFlush.enabled` : `true`
* `memoryFlush.softThresholdTokens` : `4000`
* `memoryFlush.prompt` / `memoryFlush.systemPrompt` : valeurs par défaut intégrées avec `NO_REPLY`
* Remarque : le vidage de la mémoire est ignoré lorsque l’espace de travail de la session est en lecture seule
  (`agents.defaults.sandbox.workspaceAccess: "ro"` ou `"none"`).

Exemple (ajusté) :

```json5
{
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard",
        reserveTokensFloor: 24000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 6000,
          systemPrompt: "Session proche de la compaction. Enregistrez les mémoires durables maintenant.",
          prompt: "Écrivez toutes les notes durables dans memory/YYYY-MM-DD.md ; répondez avec NO_REPLY s'il n'y a rien à enregistrer."
        }
      }
    }
  }
}
```

Block streaming :

* `agents.defaults.blockStreamingDefault` : `"on"`/`"off"` (désactivé par défaut).
* Surcharges spécifiques aux canaux : `*.blockStreaming` (et variantes par compte) pour forcer l’activation ou la désactivation du block streaming.
  Les canaux autres que Telegram nécessitent un `*.blockStreaming: true` explicite pour activer les réponses en blocs.
* `agents.defaults.blockStreamingBreak` : `"text_end"` ou `"message_end"` (par défaut : `text_end`).
* `agents.defaults.blockStreamingChunk` : découpage souple pour les blocs diffusés en streaming. Par défaut entre
  800 et 1200 caractères, privilégie les sauts de paragraphe (`\n\n`), puis les retours à la ligne, puis les phrases.
  Exemple :
  ```json5
  {
    agents: { defaults: { blockStreamingChunk: { minChars: 800, maxChars: 1200 } } }
  }
  ```
* `agents.defaults.blockStreamingCoalesce` : fusionne les blocs diffusés avant envoi.
  Par défaut `{ idleMs: 1000 }` et hérite `minChars` depuis `blockStreamingChunk`,
  avec `maxChars` plafonné à la limite de texte du canal. Signal/Slack/Discord/Google Chat utilisent par défaut
  `minChars: 1500`, sauf surcharge explicite.
  Surcharges spécifiques aux canaux : `channels.whatsapp.blockStreamingCoalesce`, `channels.telegram.blockStreamingCoalesce`,
  `channels.discord.blockStreamingCoalesce`, `channels.slack.blockStreamingCoalesce`, `channels.mattermost.blockStreamingCoalesce`,
  `channels.signal.blockStreamingCoalesce`, `channels.imessage.blockStreamingCoalesce`, `channels.msteams.blockStreamingCoalesce`,
  `channels.googlechat.blockStreamingCoalesce`
  (et variantes par compte).
* `agents.defaults.humanDelay` : pause aléatoire entre les **réponses en blocs** après la première.
  Modes : `off` (par défaut), `natural` (800–2500 ms), `custom` (utilise `minMs`/`maxMs`).
  Surcharge par agent : `agents.list[].humanDelay`.
  Exemple :
  ```json5
  {
    agents: { defaults: { humanDelay: { mode: "natural" } } }
  }
  ```

Voir [/concepts/streaming](/fr/concepts/streaming) pour les détails de comportement et de découpage.

Indicateurs de saisie :

* `agents.defaults.typingMode` : `"never" | "instant" | "thinking" | "message"`. Par défaut :
  `instant` pour les conversations directes / mentions et `message` pour les conversations de groupe sans mention.
* `session.typingMode` : surcharge par session pour le mode.
* `agents.defaults.typingIntervalSeconds` : intervalle de rafraîchissement du signal de saisie (par défaut : 6 s).
* `session.typingIntervalSeconds` : surcharge par session pour l’intervalle de rafraîchissement.
  Voir [/concepts/typing-indicators](/fr/concepts/typing-indicators) pour les détails de comportement.

`agents.defaults.model.primary` doit être défini sous la forme `provider/model` (par ex. `anthropic/claude-opus-4-5`).
Les alias proviennent de `agents.defaults.models.*.alias` (par ex. `Opus`).
Si vous omettez le fournisseur, OpenClaw suppose actuellement `anthropic` comme mécanisme
de repli temporaire pour la dépréciation.
Les modèles Z.AI sont disponibles sous la forme `zai/<model>` (par ex. `zai/glm-4.7`) et nécessitent
`ZAI_API_KEY` (ou l’ancienne variable `Z_AI_API_KEY`) dans l’environnement.

`agents.defaults.heartbeat` configure les exécutions périodiques du signal de vie :

* `every` : chaîne de durée (`ms`, `s`, `m`, `h`) ; unité par défaut : minutes. Valeur par défaut :
  `30m`. Définissez `0m` pour désactiver.
* `model` : modèle de remplacement optionnel pour les exécutions de signal de vie (`provider/model`).
* `includeReasoning` : lorsque `true`, les signaux de vie délivrent également le message `Reasoning:` séparé lorsqu’il est disponible (même forme que `/reasoning on`). Valeur par défaut : `false`.
* `session` : clé de session optionnelle pour contrôler dans quelle session le signal de vie s’exécute. Valeur par défaut : `main`.
* `to` : remplacement optionnel du destinataire (id spécifique au canal, par ex. E.164 pour WhatsApp, identifiant de chat pour Telegram).
* `target` : canal de livraison optionnel (`last`, `whatsapp`, `telegram`, `discord`, `slack`, `msteams`, `signal`, `imessage`, `none`). Valeur par défaut : `last`.
* `prompt` : remplacement optionnel pour le corps du signal de vie (valeur par défaut : `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`). Les remplacements sont envoyés tels quels ; incluez une ligne `Read HEARTBEAT.md` si vous voulez toujours que le fichier soit lu.
* `ackMaxChars` : nombre maximal de caractères autorisés après `HEARTBEAT_OK` avant la livraison (valeur par défaut : 300).

Signaux de vie par agent :

* Définissez `agents.list[].heartbeat` pour activer ou remplacer les paramètres de signal de vie pour un agent spécifique.
* Si une entrée d’agent définit `heartbeat`, **seuls ces agents** exécutent des signaux de vie ; les valeurs par défaut
  deviennent la base commune pour ces agents.

Les signaux de vie exécutent des tours d’agent complets. Des intervalles plus courts consomment plus de jetons ; faites attention
à `every`, gardez `HEARTBEAT.md` très petit et/ou choisissez un `model` moins coûteux.

`tools.exec` configure les valeurs par défaut d’exécution en arrière-plan :

* `backgroundMs` : délai avant passage automatique en arrière-plan (ms, valeur par défaut : 10000)
* `timeoutSec` : arrêt automatique après ce temps d’exécution (secondes, valeur par défaut : 1800)
* `cleanupMs` : durée de conservation des sessions terminées en mémoire (ms, valeur par défaut : 1800000)
* `notifyOnExit` : met en file d’attente un événement système + demande un signal de vie lorsqu’une exécution en arrière-plan se termine (valeur par défaut : true)
* `applyPatch.enabled` : active `apply_patch` expérimental (OpenAI/OpenAI Codex uniquement ; valeur par défaut : false)
* `applyPatch.allowModels` : liste d’autorisation optionnelle d’ids de modèles (par ex. `gpt-5.2` ou `openai/gpt-5.2`)
  Remarque : `applyPatch` ne se trouve que sous `tools.exec`.

`tools.web` configure les outils de recherche web + fetch :

* `tools.web.search.enabled` (valeur par défaut : true lorsque la clé est présente)
* `tools.web.search.apiKey` (recommandé : à définir via `openclaw configure --section web`, ou utiliser la variable d’environnement `BRAVE_API_KEY`)
* `tools.web.search.maxResults` (1–10, valeur par défaut : 5)
* `tools.web.search.timeoutSeconds` (valeur par défaut : 30)
* `tools.web.search.cacheTtlMinutes` (valeur par défaut : 15)
* `tools.web.fetch.enabled` (valeur par défaut : true)
* `tools.web.fetch.maxChars` (valeur par défaut : 50000)
* `tools.web.fetch.timeoutSeconds` (valeur par défaut : 30)
* `tools.web.fetch.cacheTtlMinutes` (valeur par défaut : 15)
* `tools.web.fetch.userAgent` (remplacement optionnel)
* `tools.web.fetch.readability` (valeur par défaut : true ; désactivez pour utiliser uniquement un nettoyage HTML basique)
* `tools.web.fetch.firecrawl.enabled` (valeur par défaut : true lorsqu’une clé d’API est définie)
* `tools.web.fetch.firecrawl.apiKey` (optionnel ; valeur par défaut : `FIRECRAWL_API_KEY`)
* `tools.web.fetch.firecrawl.baseUrl` (valeur par défaut : https://api.firecrawl.dev)
* `tools.web.fetch.firecrawl.onlyMainContent` (valeur par défaut : true)
* `tools.web.fetch.firecrawl.maxAgeMs` (optionnel)
* `tools.web.fetch.firecrawl.timeoutSeconds` (optionnel)

`tools.media` configure l’analyse des médias entrants (image/audio/vidéo) :

* `tools.media.models` : liste de modèles partagés (étiquetés par capacité ; utilisés après les listes spécifiques).
* `tools.media.concurrency` : nombre maximal d’exécutions de capacités en parallèle (par défaut 2).
* `tools.media.image` / `tools.media.audio` / `tools.media.video` :
  * `enabled` : option de désactivation (activé par défaut lorsque des modèles sont configurés).
  * `prompt` : remplacement optionnel du prompt (image/vidéo ajoutent automatiquement un indice `maxChars`).
  * `maxChars` : nombre maximal de caractères de sortie (par défaut 500 pour image/vidéo ; non défini pour l’audio).
  * `maxBytes` : taille maximale du média à envoyer (par défaut : image 10 Mo, audio 20 Mo, vidéo 50 Mo).
  * `timeoutSeconds` : délai d’expiration de la requête (par défaut : image 60 s, audio 60 s, vidéo 120 s).
  * `language` : indication audio optionnelle.
  * `attachments` : stratégie de pièces jointes (`mode`, `maxAttachments`, `prefer`).
  * `scope` : filtrage optionnel (la première correspondance l’emporte) avec `match.channel`, `match.chatType` ou `match.keyPrefix`.
  * `models` : liste ordonnée d’entrées de modèle ; en cas d’échec ou de média trop volumineux, bascule vers l’entrée suivante.
* Chaque entrée `models[]` :
  * Entrée de type fournisseur (`type: "provider"` ou omis) :
    * `provider` : identifiant du fournisseur d’API (`openai`, `anthropic`, `google`/`gemini`, `groq`, etc.).
    * `model` : surcharge de l’identifiant du modèle (obligatoire pour l’image ; par défaut `gpt-4o-mini-transcribe`/`whisper-large-v3-turbo` pour les fournisseurs audio, et `gemini-3-flash-preview` pour la vidéo).
    * `profile` / `preferredProfile` : sélection du profil d’authentification.
  * Entrée CLI (`type: "cli"`) :
    * `command` : exécutable à lancer.
    * `args` : arguments paramétrables (prend en charge `{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}`, etc.).
  * `capabilities` : liste optionnelle (`image`, `audio`, `video`) pour restreindre une entrée partagée. Valeurs par défaut si omis : `openai`/`anthropic`/`minimax` → image, `google` → image+audio+video, `groq` → audio.
  * `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language` peuvent être surchargés par entrée.

Si aucun modèle n’est configuré (ou `enabled: false`), la compréhension est ignorée ; le modèle reçoit tout de même les pièces jointes d’origine.

L’authentification des fournisseurs suit l’ordre standard d’authentification des modèles (profils d’auth, variables d’environnement comme `OPENAI_API_KEY`/`GROQ_API_KEY`/`GEMINI_API_KEY`, ou `models.providers.*.apiKey`).

Exemple :

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        scope: {
          default: "deny",
          rules: [{ action: "allow", match: { chatType: "direct" } }]
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          { type: "cli", command: "whisper", args: ["--model", "base", "{{MediaPath}}"] }
        ]
      },
      video: {
        enabled: true,
        maxBytes: 52428800,
        models: [{ provider: "google", model: "gemini-3-flash-preview" }]
      }
    }
  }
}
```

`agents.defaults.subagents` paramètre les valeurs par défaut des sous-agents :

* `model` : modèle par défaut pour les sous-agents créés (chaîne ou `{ primary, fallbacks }`). S’il est omis, les sous-agents héritent du modèle de l’appelant, sauf si une valeur est surchargée par agent ou par appel.
* `maxConcurrent` : nombre maximal d’exécutions de sous-agents concurrentes (par défaut : 1)
* `archiveAfterMinutes` : archivage automatique des sessions de sous-agents après N minutes (par défaut : 60 ; définir `0` pour désactiver)
* Politique d’outils par sous-agent : `tools.subagents.tools.allow` / `tools.subagents.tools.deny` (`deny` prévaut)

`tools.profile` définit une **liste d’autorisation de base pour les outils** avant `tools.allow`/`tools.deny` :

* `minimal` : `session_status` uniquement
* `coding` : `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
* `messaging` : `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
* `full` : aucune restriction (équivaut à non défini)

Surcharge par agent : `agents.list[].tools.profile`.

Exemple (messagerie uniquement par défaut, autorise aussi les outils Slack + Discord) :

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"]
  }
}
```

Exemple (profil d’exécution de code, mais interdire `exec`/`process` partout) :

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"]
  }
}
```

`tools.byProvider` vous permet de **restreindre encore davantage** les outils pour des fournisseurs spécifiques (ou un seul couple `provider/model`).
Surcharge par agent : `agents.list[].tools.byProvider`.

Ordre : profil de base → profil du fournisseur → politiques allow/deny.
Les clés de fournisseur acceptent soit `provider` (par ex. `google-antigravity`), soit `provider/model`
(par ex. `openai/gpt-5.2`).

Exemple (conserver le profil de codage global, mais n’activer que le minimum d’outils pour Google Antigravity) :

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" }
    }
  }
}
```

Exemple (liste d’autorisation spécifique au fournisseur/modèle) :

```json5
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] }
    }
  }
}
```

`tools.allow` / `tools.deny` configurent une politique globale d’autorisation/interdiction des outils (**deny** est prioritaire).
La correspondance n’est pas sensible à la casse et prend en charge le caractère générique `*` (`"*"` signifie tous les outils).
Cette règle s’applique même lorsque le sandbox Docker est **off**.

Exemple (désactiver `browser`/`canvas` partout) :

```json5
{
  tools: { deny: ["browser", "canvas"] }
}
```

Les groupes d’outils (abréviations) fonctionnent dans les politiques d’outils **globales** et **par agent** :

* `group:runtime` : `exec`, `bash`, `process`
* `group:fs` : `read`, `write`, `edit`, `apply_patch`
* `group:sessions` : `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory` : `memory_search`, `memory_get`
* `group:web` : `web_search`, `web_fetch`
* `group:ui` : `browser`, `canvas`
* `group:automation` : `cron`, `gateway`
* `group:messaging` : `message`
* `group:nodes` : `nodes`
* `group:openclaw` : tous les outils OpenClaw intégrés (exclut les plugins de fournisseurs)

`tools.elevated` contrôle l’accès privilégié (hôte) à `exec` :

* `enabled` : autoriser le mode privilégié (true par défaut)
* `allowFrom` : listes d’autorisation par canal (vide = désactivé)
  * `whatsapp` : numéros E.164
  * `telegram` : identifiants de chat ou noms d’utilisateur
  * `discord` : identifiants utilisateur ou noms d’utilisateur (se rabat sur `channels.discord.dm.allowFrom` si omis)
  * `signal` : numéros E.164
  * `imessage` : handles/identifiants de chat
  * `webchat` : identifiants de session ou noms d’utilisateur

Exemple :

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15555550123"],
        discord: ["steipete", "1234567890123"]
      }
    }
  }
}
```

Surcharge par agent (restriction supplémentaire) :

```json5
{
  agents: {
    list: [
      {
        id: "family",
        tools: {
          elevated: { enabled: false }
        }
      }
    ]
  }
}
```

Notes :

* `tools.elevated` est la valeur de base globale. `agents.list[].tools.elevated` ne peut que restreindre davantage (les deux doivent autoriser).
* `/elevated on|off|ask|full` stocke l’état par clé de session ; les directives inline s’appliquent à un seul message.
* L’exécution élevée de `exec` s’effectue sur l’hôte et contourne le sandboxing.
* La politique d’utilisation des outils reste applicable ; si `exec` est refusé, le mode élevé ne peut pas être utilisé.

`agents.defaults.maxConcurrent` définit le nombre maximal d’exécutions d’agents imbriqués pouvant
s’exécuter en parallèle sur l’ensemble des sessions. Chaque session reste sérialisée (une exécution
par clé de session à la fois). Valeur par défaut : 1.

<div id="agentsdefaultssandbox">
  ### `agents.defaults.sandbox`
</div>

**Sandbox Docker** optionnel pour l&#39;agent embarqué. Prévu pour les sessions
non principales afin qu&#39;elles ne puissent pas accéder à votre système hôte.

Détails : [Sandboxing](/fr/gateway/sandboxing)

Valeurs par défaut (si activé) :

* scope : `"agent"` (un conteneur + espace de travail par agent)
* image basée sur Debian bookworm-slim
* accès de l&#39;agent à l&#39;espace de travail : `workspaceAccess: "none"` (par défaut)
  * `"none"` : utiliser un espace de travail de sandbox par portée sous `~/.openclaw/sandboxes`
* `"ro"` : conserver l&#39;espace de travail de la sandbox à `/workspace` et monter l&#39;espace de travail de l&#39;agent en lecture seule à `/agent` (désactive `write`/`edit`/`apply_patch`)
  * `"rw"` : monter l&#39;espace de travail de l&#39;agent en lecture/écriture à `/workspace`
* nettoyage automatique : inactif &gt; 24h OU âge &gt; 7j
* stratégie d’outils : autoriser uniquement `exec`, `process`, `read`, `write`, `edit`, `apply_patch`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` (la règle de refus l’emporte)
  * configuration via `tools.sandbox.tools`, surcharge par agent via `agents.list[].tools.sandbox.tools`
  * abréviations de groupes d’outils prises en charge dans la stratégie de sandbox : `group:runtime`, `group:fs`, `group:sessions`, `group:memory` (voir [Sandbox vs Tool Policy vs Elevated](/fr/gateway/sandbox-vs-tool-policy-vs-elevated#tool-groups-shorthands))
* navigateur optionnel isolé dans une sandbox (Chromium + CDP, observateur noVNC)
* paramètres de durcissement de la sécurité : `network`, `user`, `pidsLimit`, `memory`, `cpus`, `ulimits`, `seccompProfile`, `apparmorProfile`

Avertissement : `scope: "shared"` signifie un conteneur partagé et un espace de travail partagé. Aucune isolation entre sessions. Utilisez `scope: "session"` pour une isolation par session.

Compatibilité héritée : `perSession` est toujours pris en charge (`true` → `scope: "session"`,
`false` → `scope: "shared"`).

`setupCommand` s’exécute **une seule fois** après la création du conteneur (à l’intérieur du conteneur via `sh -lc`).
Pour les installations de paquets, assurez-vous de disposer d’un accès réseau sortant, d’un système de fichiers racine inscriptible et d’un utilisateur root.

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent is default)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          containerPrefix: "openclaw-sbx-",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          // Remplacement par agent (multi-agent) : agents.list[].sandbox.docker.*
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
          extraHosts: ["internal.service:10.0.0.5"],
          binds: ["/var/run/docker.sock:/var/run/docker.sock", "/home/user/source:/source:rw"]
        },
        browser: {
          enabled: false,
          image: "openclaw-sandbox-browser:bookworm-slim",
          containerPrefix: "openclaw-sbx-browser-",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          allowedControlUrls: ["http://10.0.0.42:18791"],
          allowedControlHosts: ["browser.lab.local", "10.0.0.42"],
          allowedControlPorts: [18791],
          autoStart: true,
          autoStartTimeoutMs: 12000
        },
        prune: {
          idleHours: 24,  // 0 disables idle pruning
          maxAgeDays: 7   // 0 disables max-age pruning
        }
      }
    }
  },
  tools: {
    sandbox: {
      tools: {
        allow: ["exec", "process", "read", "write", "edit", "apply_patch", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"]
      }
    }
  }
}
```

Construisez l’image sandbox par défaut une fois avec :

```bash
scripts/sandbox-setup.sh
```

Remarque : par défaut, les conteneurs de sandbox utilisent `network: "none"` ; définissez `agents.defaults.sandbox.docker.network`
sur `"bridge"` (ou votre réseau personnalisé) si l’agent a besoin d’un accès sortant.

Remarque : les pièces jointes entrantes sont mises en attente dans l’espace de travail actif sous `media/inbound/*`. Avec `workspaceAccess: "rw"`, cela signifie que les fichiers sont écrits dans l’espace de travail de l’agent.

Remarque : `docker.binds` monte des répertoires hôtes supplémentaires ; les montages globaux et spécifiques à chaque agent sont fusionnés.

Générez l’image de navigateur facultative avec :

```bash
scripts/sandbox-browser-setup.sh
```

Lorsque `agents.defaults.sandbox.browser.enabled=true`, l’outil de navigation utilise une instance Chromium en sandbox (CDP). Si noVNC est activé (valeur par défaut lorsque headless=false),
l’URL noVNC est injectée dans l’invite système afin que l’agent puisse s’y référer.
Cela ne nécessite pas `browser.enabled` dans la configuration principale ; l’URL de contrôle de la sandbox
est injectée par session.

`agents.defaults.sandbox.browser.allowHostControl` (par défaut : false) permet
aux sessions en sandbox de cibler explicitement le serveur de contrôle du navigateur de l’**hôte**
via l’outil de navigation (`target: "host"`). Laissez cette option désactivée si vous voulez une isolation
de sandbox stricte.

Listes d’autorisation pour le contrôle à distance :

* `allowedControlUrls` : URL de contrôle exactes autorisées pour `target: "custom"`.
* `allowedControlHosts` : noms d’hôte autorisés (nom d’hôte uniquement, sans port).
* `allowedControlPorts` : ports autorisés (par défaut : http=80, https=443).
  Valeurs par défaut : toutes les listes d’autorisation sont non définies (aucune restriction). `allowHostControl` est false par défaut.

<div id="models-custom-providers-base-urls">
  ### `models` (fournisseurs personnalisés + URL de base)
</div>

OpenClaw utilise le catalogue de modèles **pi-coding-agent**. Vous pouvez ajouter des fournisseurs personnalisés
(LiteLLM, serveurs locaux compatibles OpenAI, proxies Anthropic, etc.) en créant ou en modifiant
`~/.openclaw/agents/<agentId>/agent/models.json` ou en définissant le même schéma dans votre
configuration OpenClaw sous `models.providers`.
Vue d’ensemble par fournisseur + exemples : [/concepts/model-providers](/fr/concepts/model-providers).

Lorsque `models.providers` est présent, OpenClaw écrit ou fusionne un `models.json` dans
`~/.openclaw/agents/<agentId>/agent/` au démarrage :

* comportement par défaut : **fusion** (conserve les fournisseurs existants, écrase ceux portant le même nom)
* définissez `models.mode: "replace"` pour écraser le contenu du fichier

Sélectionnez le modèle via `agents.defaults.model.primary` (fournisseur/modèle).

```json5
{
  agents: {
    defaults: {
      model: { primary: "custom-proxy/llama-3.1-8b" },
      models: {
        "custom-proxy/llama-3.1-8b": {}
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions",
        models: [
          {
            id: "llama-3.1-8b",
            name: "Llama 3.1 8B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 32000
          }
        ]
      }
    }
  }
}
```

<div id="opencode-zen-multi-model-proxy">
  ### OpenCode Zen (proxy multi-modèles)
</div>

OpenCode Zen est un Gateway multi-modèles avec des endpoints dédiés par modèle. OpenClaw utilise
le fournisseur `opencode` intégré de pi-ai ; définissez `OPENCODE_API_KEY` (ou
`OPENCODE_ZEN_API_KEY`) à partir de https://opencode.ai/auth.

Notes :

* Les références de modèles utilisent `opencode/<modelId>` (exemple : `opencode/claude-opus-4-5`).
* Si vous activez une liste d’autorisation via `agents.defaults.models`, ajoutez chaque modèle que vous prévoyez d’utiliser.
* Raccourci : `openclaw onboard --auth-choice opencode-zen`.

```json5
{
  agents: {
    defaults: {
      model: { primary: "opencode/claude-opus-4-5" },
      models: { "opencode/claude-opus-4-5": { alias: "Opus" } }
    }
  }
}
```

<div id="zai-glm-47-provider-alias-support">
  ### Z.AI (GLM-4.7) — prise en charge des alias de fournisseur
</div>

Les modèles Z.AI sont disponibles via le fournisseur intégré `zai`. Définissez `ZAI_API_KEY`
dans votre environnement et référencez le modèle par fournisseur/modèle.

Raccourci : `openclaw onboard --auth-choice zai-api-key`.

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} }
    }
  }
}
```

Remarques :

* `z.ai/*` et `z-ai/*` sont des alias acceptés et sont normalisés en `zai/*`.
* Si `ZAI_API_KEY` est absent, les requêtes vers `zai/*` échoueront avec une erreur d’authentification à l’exécution.
* Exemple d’erreur : `No API key found for provider "zai".`
* Le endpoint API général de Z.AI est `https://api.z.ai/api/paas/v4`. Les requêtes
  GLM coding utilisent le endpoint dédié « Coding » `https://api.z.ai/api/coding/paas/v4`.
  Le fournisseur intégré `zai` utilise le endpoint Coding. Si vous avez besoin du
  endpoint général, définissez un fournisseur personnalisé dans `models.providers`
  avec un remplacement de l’URL de base (voir la section sur les fournisseurs
  personnalisés ci-dessus).
* Utilisez une valeur factice dans la documentation/les configurations ; ne
  validez jamais de vraies clés API.

<div id="moonshot-ai-kimi">
  ### Moonshot AI (Kimi)
</div>

Utilisez le point de terminaison Moonshot compatible avec OpenAI :

```json5
{
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: { "moonshot/kimi-k2.5": { alias: "Kimi K2.5" } }
    }
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

Notes :

* Définissez `MOONSHOT_API_KEY` dans l’environnement ou utilisez `openclaw onboard --auth-choice moonshot-api-key`.
* Référence du modèle : `moonshot/kimi-k2.5`.
* Utilisez `https://api.moonshot.cn/v1` si vous avez besoin du point de terminaison pour la Chine.

<div id="kimi-code">
  ### Kimi Code
</div>

Utilisez l’endpoint spécifique de Kimi Code, compatible avec OpenAI (séparé de Moonshot) :

```json5
{
  env: { KIMICODE_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-code/kimi-for-coding" },
      models: { "kimi-code/kimi-for-coding": { alias: "Kimi Code" } }
    }
  },
  models: {
    mode: "merge",
    providers: {
      "kimi-code": {
        baseUrl: "https://api.kimi.com/coding/v1",
        apiKey: "${KIMICODE_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-for-coding",
            name: "Kimi For Coding",
            reasoning: true,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 262144,
            maxTokens: 32768,
            headers: { "User-Agent": "KimiCLI/0.77" },
            compat: { supportsDeveloperRole: false }
          }
        ]
      }
    }
  }
}
```

Notes :

* Définissez la variable d’environnement `KIMICODE_API_KEY` ou exécutez `openclaw onboard --auth-choice kimi-code-api-key`.
* Référence du modèle : `kimi-code/kimi-for-coding`.

<div id="synthetic-anthropic-compatible">
  ### Synthetic (compatible avec Anthropic)
</div>

Utilisez l&#39;endpoint de Synthetic compatible avec Anthropic :

```json5
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.1": { alias: "MiniMax M2.1" } }
    }
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536
          }
        ]
      }
    }
  }
}
```

Notes :

* Configurez `SYNTHETIC_API_KEY` ou utilisez `openclaw onboard --auth-choice synthetic-api-key`.
* Référence du modèle : `synthetic/hf:MiniMaxAI/MiniMax-M2.1`.
* L’URL de base doit omettre `/v1`, car le client Anthropic l’ajoute.

<div id="local-models-lm-studio-recommended-setup">
  ### Modèles locaux (LM Studio) — configuration recommandée
</div>

Voir [/gateway/local-models](/fr/gateway/local-models) pour les recommandations actuelles concernant les modèles locaux. En résumé : exécute MiniMax M2.1 via l’API Responses de LM Studio sur une machine suffisamment puissante ; garde les modèles hébergés activés en parallèle comme solution de repli.

<div id="minimax-m21">
  ### MiniMax M2.1
</div>

Utilisez MiniMax M2.1 directement, sans LM Studio :

```json5
{
  agent: {
    model: { primary: "minimax/MiniMax-M2.1" },
    models: {
      "anthropic/claude-opus-4-5": { alias: "Opus" },
      "minimax/MiniMax-M2.1": { alias: "Minimax" }
    }
  },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            // Tarification : mettez à jour models.json si vous avez besoin d'un suivi précis des coûts.
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

Remarques :

* Définissez la variable d&#39;environnement `MINIMAX_API_KEY` ou utilisez `openclaw onboard --auth-choice minimax-api`.
* Modèle disponible : `MiniMax-M2.1` (par défaut).
* Mettez à jour la tarification dans `models.json` si vous avez besoin d&#39;un suivi précis des coûts.

<div id="cerebras-glm-46-47">
  ### Cerebras (GLM 4.6 / 4.7)
</div>

Utilisez Cerebras via son endpoint compatible avec l’API OpenAI :

```json5
{
  env: { CEREBRAS_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: {
        primary: "cerebras/zai-glm-4.7",
        fallbacks: ["cerebras/zai-glm-4.6"]
      },
      models: {
        "cerebras/zai-glm-4.7": { alias: "GLM 4.7 (Cerebras)" },
        "cerebras/zai-glm-4.6": { alias: "GLM 4.6 (Cerebras)" }
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      cerebras: {
        baseUrl: "https://api.cerebras.ai/v1",
        apiKey: "${CEREBRAS_API_KEY}",
        api: "openai-completions",
        models: [
          { id: "zai-glm-4.7", name: "GLM 4.7 (Cerebras)" },
          { id: "zai-glm-4.6", name: "GLM 4.6 (Cerebras)" }
        ]
      }
    }
  }
}
```

Remarques :

* Utilisez `cerebras/zai-glm-4.7` pour Cerebras ; utilisez `zai/glm-4.7` pour un accès direct à Z.AI.
* Définissez `CEREBRAS_API_KEY` dans l’environnement ou la configuration.

Remarques :

* API prises en charge : `openai-completions`, `openai-responses`, `anthropic-messages`,
  `google-generative-ai`
* Utilisez `authHeader: true` + `headers` pour des besoins d’authentification personnalisés.
* Remplacez la racine de configuration de l’agent par `OPENCLAW_AGENT_DIR` (ou `PI_CODING_AGENT_DIR`)
  si vous voulez que `models.json` soit stocké ailleurs (valeur par défaut : `~/.openclaw/agents/main/agent`).

<div id="session">
  ### `session`
</div>

Contrôle le périmètre de la session, la stratégie de réinitialisation, les déclencheurs de réinitialisation et le lieu de stockage du magasin de session.

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main",
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"]
    },
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 60
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      dm: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 }
    },
    resetTriggers: ["/new", "/reset"],
    // Par défaut déjà défini par agent sous ~/.openclaw/agents/<agentId>/sessions/sessions.json
    // Vous pouvez remplacer en utilisant le template {agentId} :
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    // Direct chats collapse to agent:<agentId>:<mainKey> (default: "main").
    mainKey: "main",
    agentToAgent: {
      // Max ping-pong reply turns between requester/target (0–5).
      maxPingPongTurns: 5
    },
    sendPolicy: {
      rules: [
        { action: "deny", match: { channel: "discord", chatType: "group" } }
      ],
      default: "allow"
    }
  }
}
```

Fields:

* `mainKey`: clé de compartiment de conversation directe (par défaut : `"main"`). Utile lorsque tu veux « renommer » le fil de MP principal sans changer `agentId`.
  * Remarque sandbox : `agents.defaults.sandbox.mode: "non-main"` utilise cette clé pour détecter la session principale. Toute clé de session qui ne correspond pas à `mainKey` (groupes/canaux) est placée dans un sandbox.
* `dmScope`: manière dont les sessions de MP sont regroupées (par défaut : `"main"`).
  * `main`: tous les MP partagent la session principale pour assurer la continuité.
  * `per-peer`: isole les MP par identifiant d’expéditeur sur l’ensemble des canaux.
  * `per-channel-peer`: isole les MP par canal + expéditeur (recommandé pour les boîtes de réception multi‑utilisateurs).
  * `per-account-channel-peer`: isole les MP par compte + canal + expéditeur (recommandé pour les boîtes de réception multi‑comptes).
* `identityLinks`: associe des identifiants canoniques à des pairs préfixés par le fournisseur afin que la même personne partage une session de MP sur l’ensemble des canaux lorsque tu utilises `per-peer`, `per-channel-peer` ou `per-account-channel-peer`.
  * Exemple : `alice: ["telegram:123456789", "discord:987654321012345678"]`.
* `reset`: stratégie de réinitialisation principale. Par défaut, réinitialisation quotidienne à 4h00, heure locale de l’hôte du Gateway.
  * `mode`: `daily` ou `idle` (par défaut : `daily` lorsque `reset` est présent).
  * `atHour`: heure locale (0-23) de la limite de réinitialisation quotidienne.
  * `idleMinutes`: fenêtre glissante d’inactivité en minutes. Lorsque `daily` et `idle` sont tous deux configurés, celui qui expire en premier l’emporte.
* `resetByType`: remplacements par type de session pour `dm`, `group` et `thread`.
  * Si tu ne définis que l’ancien `session.idleMinutes` sans aucun `reset`/`resetByType`, OpenClaw reste en mode inactivité seule pour la compatibilité ascendante.
* `heartbeatIdleMinutes`: surcharge optionnelle d’inactivité pour les vérifications de signal de vie (la réinitialisation quotidienne s’applique toujours lorsqu’elle est activée).
* `agentToAgent.maxPingPongTurns`: nombre maximal d’allers‑retours de réponses entre le demandeur et la cible (0–5, par défaut 5).
* `sendPolicy.default`: `allow` ou `deny` utilisé en dernier recours lorsqu’aucune règle ne correspond.
* `sendPolicy.rules[]`: fait correspondre en fonction de `channel`, `chatType` (`direct|group|room`) ou `keyPrefix` (par ex. `cron:`). Le premier `deny` l’emporte ; sinon, `allow`.

<div id="skills-skills-config">
  ### `skills` (configuration des compétences)
</div>

Contrôle la liste d’autorisation intégrée, les préférences d’installation, les dossiers de compétences supplémentaires et les remplacements de configuration par compétence. S’applique aux compétences **intégrées** et à `~/.openclaw/skills` (les compétences de l’espace de travail restent prioritaires en cas de conflit de nom).

Champs :

* `allowBundled` : liste d’autorisation facultative pour les compétences **intégrées** uniquement. Si elle est définie, seules ces compétences intégrées sont éligibles (les compétences gérées/de l’espace de travail ne sont pas affectées).
* `load.extraDirs` : répertoires de compétences supplémentaires à parcourir (plus faible priorité).
* `install.preferBrew` : privilégier les installations via brew lorsqu’elles sont disponibles (par défaut : true).
* `install.nodeManager` : préférence pour le gestionnaire de paquets (`npm` | `pnpm` | `yarn`, par défaut : npm).
* `entries.<skillKey>` : remplacements de configuration par compétence.

Champs par compétence :

* `enabled` : définir sur `false` pour désactiver une compétence même si elle est intégrée/installée.
* `env` : variables d’environnement injectées pour l’exécution de l’agent (uniquement si elles ne sont pas déjà définies).
* `apiKey` : raccourci pratique facultatif pour les compétences qui déclarent une variable d’environnement principale (par ex. `nano-banana-pro` → `GEMINI_API_KEY`).

Exemple :

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: [
        "~/Projects/agent-scripts/skills",
        "~/Projects/oss/some-skill-pack/skills"
      ]
    },
    install: {
      preferBrew: true,
      nodeManager: "npm"
    },
    entries: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE"
        }
      },
      peekaboo: { enabled: true },
      sag: { enabled: false }
    }
  }
}
```

<div id="plugins-extensions">
  ### `plugins` (extensions)
</div>

Contrôle la découverte des plugins, les règles d’autorisation/interdiction et la configuration par plugin. Les plugins sont chargés
depuis `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions`, ainsi que depuis toutes les entrées
`plugins.load.paths`. **Les modifications de configuration nécessitent un redémarrage de Gateway.**
Voir [/plugin](/fr/plugin) pour une description détaillée de l’utilisation.

Champs :

* `enabled` : interrupteur global pour le chargement des plugins (par défaut : true).
* `allow` : liste d’autorisation optionnelle d’identifiants de plugins ; lorsqu’elle est définie, seuls les plugins listés sont chargés.
* `deny` : liste de blocage optionnelle d’identifiants de plugins (l’interdiction a priorité).
* `load.paths` : fichiers ou répertoires de plugins supplémentaires à charger (absolus ou `~`).
* `entries.<pluginId>` : remplacements par plugin.
  * `enabled` : mettre à `false` pour désactiver.
  * `config` : objet de configuration spécifique au plugin (validé par le plugin si fourni).

Exemple :

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    load: {
      paths: ["~/Projects/oss/voice-call-extension"]
    },
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio"
        }
      }
    }
  }
}
```

<div id="browser-openclaw-managed-browser">
  ### `browser` (navigateur géré par openclaw)
</div>

OpenClaw peut démarrer une instance Chrome/Brave/Edge/Chromium **dédiée et isolée** pour openclaw et exposer un petit service de contrôle sur l’interface loopback locale.
Les profils peuvent cibler un navigateur **distant** basé sur Chromium via `profiles.<name>.cdpUrl`. Les profils distants ne permettent que l’attachement (les opérations start/stop/reset sont désactivées).

`browser.cdpUrl` reste utilisé pour les anciennes configurations à profil unique et comme schéma/hôte de base pour les profils qui ne définissent que `cdpPort`.

Valeurs par défaut :

* enabled : `true`
* evaluateEnabled : `true` (mettre `false` pour désactiver `act:evaluate` et `wait --fn`)
* control service : loopback uniquement (port dérivé de `gateway.port`, `18791` par défaut)
* CDP URL : `http://127.0.0.1:18792` (control service + 1, ancien mode profil unique)
* profile color : `#FF4500` (orange homard)
* Remarque : le serveur de contrôle est démarré par le Gateway en cours d’exécution (barre de menus OpenClaw.app ou `openclaw gateway`).
* Ordre d’auto‑détection : navigateur par défaut s’il est basé sur Chromium ; sinon Chrome → Brave → Edge → Chromium → Chrome Canary.

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: true,
    // cdpUrl: "http://127.0.0.1:18792", // legacy single-profile override
    defaultProfile: "chrome",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" }
    },
    color: "#FF4500",
    // Advanced:
    // headless: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    // attachOnly: false, // définir à true lors du tunnel d'un CDP distant vers localhost
  }
}
```

<div id="ui-appearance">
  ### `ui` (Apparence)
</div>

Couleur d’accentuation optionnelle utilisée par les applications natives pour l’habillage de l’UI (par exemple, la teinte de la bulle en Talk Mode).

Si ce paramètre n’est pas défini, les clients utilisent par défaut un bleu clair atténué.

```json5
{
  ui: {
    seamColor: "#FF4500", // hex (RRGGBB or #RRGGBB)
    // Optionnel : remplacement de l'identité de l'assistant du Control UI.
    // Si non défini, le Control UI utilise l'identité de l'agent actif (config ou IDENTITY.md).
    assistant: {
      name: "OpenClaw",
      avatar: "CB" // emoji, short text, or image URL/data URI
    }
  }
}
```

<div id="gateway-gateway-server-mode-bind">
  ### `gateway` (mode serveur Gateway + liaison)
</div>

Utilisez `gateway.mode` pour déclarer explicitement si cette machine doit exécuter Gateway.

Valeurs par défaut :

* mode : **non défini** (interprété comme « ne pas démarrer automatiquement »)
* bind : `loopback`
* port : `18789` (port unique pour WS + HTTP)

```json5
{
  gateway: {
    mode: "local", // or "remote"
    port: 18789, // WS + HTTP multiplex
    bind: "loopback",
    // controlUi: { enabled: true, basePath: "/openclaw" }
    // auth: { mode: "token", token: "your-token" } // le token contrôle l'accès WS + Control UI
    // tailscale: { mode: "off" | "serve" | "funnel" }
  }
}
```

Chemin de base de la Control UI :

* `gateway.controlUi.basePath` définit le préfixe d’URL sous lequel la Control UI est servie.
* Exemples : `"/ui"`, `"/openclaw"`, `"/apps/openclaw"`.
* Valeur par défaut : racine (`/`) (inchangée).
* `gateway.controlUi.allowInsecureAuth` autorise une authentification par jeton uniquement pour la Control UI lorsque
  l’identité de l’appareil est omise (généralement via HTTP). Valeur par défaut : `false`. Préférez HTTPS
  (Tailscale Serve) ou `127.0.0.1`.
* `gateway.controlUi.dangerouslyDisableDeviceAuth` désactive les vérifications d’identité de l’appareil pour la
  Control UI (jeton/mot de passe uniquement). Valeur par défaut : `false`. À utiliser uniquement en dernier recours.

Documentation associée :

* [Control UI](/fr/web/control-ui)
* [Vue d’ensemble Web](/fr/web)
* [Tailscale](/fr/gateway/tailscale)
* [Accès distant](/fr/gateway/remote)

Proxies de confiance :

* `gateway.trustedProxies` : liste des IP de reverse proxies qui terminent TLS devant le Gateway.
* Lorsqu’une connexion provient de l’une de ces IP, OpenClaw utilise `x-forwarded-for` (ou `x-real-ip`) pour déterminer l’IP cliente pour les vérifications d’appairage local et les vérifications d’authentification HTTP/locales.
* N’énumérez que des proxies que vous contrôlez entièrement, et assurez-vous qu’ils **écrasent** les en-têtes `x-forwarded-for` entrants.

Remarques :

* `openclaw gateway` refuse de démarrer tant que `gateway.mode` n’est pas défini sur `local` (ou que vous ne fournissez pas l’option de contournement).
* `gateway.port` contrôle l’unique port multiplexé utilisé pour WebSocket + HTTP (Control UI, hooks, A2UI).
* Endpoint OpenAI Chat Completions : **désactivé par défaut** ; activez-le avec `gateway.http.endpoints.chatCompletions.enabled: true`.
* Priorité : `--port` &gt; `OPENCLAW_GATEWAY_PORT` &gt; `gateway.port` &gt; valeur par défaut `18789`.
* L’authentification du Gateway est requise par défaut (jeton/mot de passe ou identité Tailscale Serve). Les liaisons non-loopback nécessitent un jeton/mot de passe partagé.
* L’assistant d’onboarding génère un jeton de Gateway par défaut (même sur loopback).
* `gateway.remote.token` est **uniquement** destiné aux appels CLI distants ; il n’active pas l’authentification locale du Gateway. `gateway.token` est ignoré.

Authentification et Tailscale :

* `gateway.auth.mode` définit les exigences de handshake (`token` ou `password`). S’il n’est pas défini, l’authentification par jeton est supposée.
* `gateway.auth.token` stocke le jeton partagé pour l’authentification par jeton (utilisé par la CLI sur la même machine).
* Lorsque `gateway.auth.mode` est défini, seule cette méthode est acceptée (plus les en-têtes Tailscale optionnels).
* `gateway.auth.password` peut être défini ici ou via `OPENCLAW_GATEWAY_PASSWORD` (recommandé).
* `gateway.auth.allowTailscale` autorise les en-têtes d’identité Tailscale Serve
  (`tailscale-user-login`) à satisfaire l’authentification lorsque la requête arrive sur loopback
  avec `x-forwarded-for`, `x-forwarded-proto` et `x-forwarded-host`. OpenClaw
  vérifie l’identité en résolvant l’adresse `x-forwarded-for` via
  `tailscale whois` avant de l’accepter. Lorsque `true`, les requêtes Serve n’ont pas besoin
  de jeton/mot de passe ; définissez `false` pour exiger des identifiants explicites. La valeur par défaut est
  `true` lorsque `tailscale.mode = "serve"` et que le mode d’authentification n’est pas `password`.
* `gateway.tailscale.mode: "serve"` utilise Tailscale Serve (tailnet uniquement, bind sur loopback).
* `gateway.tailscale.mode: "funnel"` expose le tableau de bord publiquement ; nécessite une authentification.
* `gateway.tailscale.resetOnExit` réinitialise la configuration Serve/Funnel à l’arrêt.

Valeurs par défaut pour les clients distants (CLI) :

* `gateway.remote.url` définit l’URL WebSocket par défaut du Gateway pour les appels CLI lorsque `gateway.mode = "remote"`.
* `gateway.remote.transport` sélectionne le transport distant macOS (`ssh` par défaut, `direct` pour ws/wss). Lorsque la valeur est `direct`, `gateway.remote.url` doit utiliser `ws://` ou `wss://`. `ws://host` utilise par défaut le port `18789`.
* `gateway.remote.token` fournit le jeton pour les appels distants (laissez ce champ non défini pour désactiver l’authentification).
* `gateway.remote.password` fournit le mot de passe pour les appels distants (laissez ce champ non défini pour désactiver l’authentification).

Comportement de l’app macOS :

* OpenClaw.app surveille `~/.openclaw/openclaw.json` et bascule de mode en temps réel lorsque `gateway.mode` ou `gateway.remote.url` est modifié.
* Si `gateway.mode` n’est pas défini mais que `gateway.remote.url` l’est, l’app macOS considère qu’il s’agit du mode distant.
* Lorsque vous changez le mode de connexion dans l’app macOS, elle écrit `gateway.mode` (et `gateway.remote.url` + `gateway.remote.transport` en mode distant) dans le fichier de configuration.

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      password: "your-password"
    }
  }
}
```

Exemple de transport direct (app macOS) :

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      transport: "direct",
      url: "wss://gateway.example.ts.net",
      token: "your-token"
    }
  }
}
```

<div id="gatewayreload-config-hot-reload">
  ### `gateway.reload` (rechargement à chaud de la configuration)
</div>

Le Gateway surveille `~/.openclaw/openclaw.json` (ou `OPENCLAW_CONFIG_PATH`) et applique automatiquement les modifications.

Modes :

* `hybrid` (par défaut) : applique à chaud les modifications sûres ; redémarre le Gateway pour les changements critiques.
* `hot` : applique uniquement les modifications sûres à chaud ; consigne dans les logs lorsqu’un redémarrage est nécessaire.
* `restart` : redémarre le Gateway à chaque modification de configuration.
* `off` : désactive le rechargement à chaud.

```json5
{
  gateway: {
    reload: {
      mode: "hybrid",
      debounceMs: 300
    }
  }
}
```

<div id="hot-reload-matrix-files-impact">
  #### Matrice de rechargement à chaud (fichiers + impact)
</div>

Fichiers surveillés :

* `~/.openclaw/openclaw.json` (ou `OPENCLAW_CONFIG_PATH`)

Appliqué à chaud (sans redémarrage complet du Gateway) :

* `hooks` (authentification webhook/chemin/mappages) + `hooks.gmail` (surveillance Gmail redémarrée)
* `browser` (redémarrage du serveur de contrôle du navigateur)
* `cron` (redémarrage du service cron + mise à jour du parallélisme)
* `agents.defaults.heartbeat` (redémarrage du runner de signal de vie)
* `web` (redémarrage du canal WhatsApp Web)
* `telegram`, `discord`, `signal`, `imessage` (redémarrages des canaux)
* `agent`, `models`, `routing`, `messages`, `session`, `whatsapp`, `logging`, `skills`, `ui`, `talk`, `identity`, `wizard` (lectures dynamiques)

Nécessite un redémarrage complet du Gateway :

* `gateway` (port/bind/auth/Control UI/Tailscale)
* `bridge` (ancien, legacy)
* `discovery`
* `canvasHost`
* `plugins`
* Tout chemin de configuration inconnu/non pris en charge (par sécurité, redémarrage par défaut)

<div id="multi-instance-isolation">
  ### Isolation multi‑instance
</div>

Pour exécuter plusieurs instances Gateway sur un même hôte (pour la redondance ou un bot de secours), isolez l’état et la configuration par instance et utilisez des ports uniques :

* `OPENCLAW_CONFIG_PATH` (configuration par instance)
* `OPENCLAW_STATE_DIR` (sessions/identifiants)
* `agents.defaults.workspace` (mémoires)
* `gateway.port` (unique par instance)

Options pratiques (CLI) :

* `openclaw --dev …` → utilise `~/.openclaw-dev` et décale les ports à partir de la base `19001`
* `openclaw --profile <name> …` → utilise `~/.openclaw-<name>` (port via config/env/flags)

Consultez le [Gateway runbook](/fr/gateway) pour le mappage de ports dérivé (gateway/navigateur/canvas).
Consultez [Multiple gateways](/fr/gateway/multiple-gateways) pour les détails d’isolation des ports navigateur/CDP.

Exemple :

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

<div id="hooks-gateway-webhooks">
  ### `hooks` (webhooks du Gateway)
</div>

Active un simple point de terminaison webhook HTTP sur le serveur HTTP du Gateway.

Valeurs par défaut :

* enabled : `false`
* path : `/hooks`
* maxBodyBytes : `262144` (256 KB)

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    presets: ["gmail"],
    transformsDir: "~/.openclaw/hooks",
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate:
          "From: {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}",
        deliver: true,
        channel: "last",
        model: "openai/gpt-5.2-mini",
      },
    ],
  }
}
```

Les requêtes doivent inclure le jeton de hook :

* `Authorization: Bearer <token>` **ou**
* `x-openclaw-token: <token>` **ou**
* `?token=<token>`

Points de terminaison :

* `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
* `POST /hooks/agent` → `{ message, name?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }`
* `POST /hooks/<name>` → résolu via `hooks.mappings`

`/hooks/agent` publie toujours un résumé dans la session principale (et peut éventuellement déclencher immédiatement un signal de vie via `wakeMode: "now"`).

Notes de mappage :

* `match.path` fait correspondre le sous-chemin après `/hooks` (par ex. `/hooks/gmail` → `gmail`).
* `match.source` fait correspondre un champ du payload (par ex. `{ source: "gmail" }`), ce qui vous permet d’utiliser un chemin générique `/hooks/ingest`.
* Les modèles tels que `{{messages[0].subject}}` lisent à partir du payload.
* `transform` peut pointer vers un module JS/TS qui renvoie une action de hook.
* `deliver: true` achemine la réponse finale vers un canal ; `channel` vaut par défaut `last` (se rabat sur WhatsApp).
* S’il n’existe aucune route de livraison préalable, définissez explicitement `channel` + `to` (requis pour Telegram/Discord/Google Chat/Slack/Signal/iMessage/MS Teams).
* `model` remplace le LLM pour cette exécution de hook (`provider/model` ou alias ; doit être autorisé si `agents.defaults.models` est défini).

Configuration de l’utilitaire Gmail (utilisée par `openclaw webhooks gmail setup` / `run`) :

```json5
{
  hooks: {
    gmail: {
      account: "openclaw@gmail.com",
      topic: "projects/<project-id>/topics/gog-gmail-watch",
      subscription: "gog-gmail-watch-push",
      pushToken: "shared-push-token",
      hookUrl: "http://127.0.0.1:18789/hooks/gmail",
      includeBody: true,
      maxBytes: 20000,
      renewEveryMinutes: 720,
      serve: { bind: "127.0.0.1", port: 8788, path: "/" },
      tailscale: { mode: "funnel", path: "/gmail-pubsub" },

      // Optionnel : utiliser un modèle moins coûteux pour le traitement des hooks Gmail
      // Repli sur agents.defaults.model.fallbacks, puis primary, en cas d'échec d'auth/rate-limit/timeout
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      // Optionnel : niveau de réflexion par défaut pour les hooks Gmail
      thinking: "off",
    }
  }
}
```

Surcharge de modèle pour les hooks Gmail :

* `hooks.gmail.model` spécifie un modèle à utiliser pour le traitement des hooks Gmail (par défaut, celui de la session principale).
* Accepte des références `provider/model` ou des alias provenant de `agents.defaults.models`.
* Bascule vers `agents.defaults.model.fallbacks`, puis vers `agents.defaults.model.primary`, en cas d’échec d’authentification, de limitation de débit ou de dépassement de délai.
* Si `agents.defaults.models` est défini, incluez le modèle des hooks dans la liste d’autorisation.
* Au démarrage, affiche un avertissement si le modèle configuré ne figure pas dans le catalogue de modèles ou la liste d’autorisation.
* `hooks.gmail.thinking` définit le niveau de réflexion par défaut pour les hooks Gmail et est remplacé par la valeur `thinking` propre à chaque hook.

Démarrage automatique du Gateway :

* Si `hooks.enabled=true` et que `hooks.gmail.account` est défini, le Gateway démarre
  `gog gmail watch serve` au démarrage et renouvelle automatiquement la surveillance.
* Définissez `OPENCLAW_SKIP_GMAIL_WATCHER=1` pour désactiver le démarrage automatique (pour des exécutions manuelles).
* Évitez d&#39;exécuter un processus `gog gmail watch serve` séparé en parallèle du Gateway ; il
  échouera avec `listen tcp 127.0.0.1:8788: bind: address already in use`.

Remarque : lorsque `tailscale.mode` est activé, OpenClaw définit par défaut `serve.path` sur `/` pour que
Tailscale puisse assurer le proxy de `/gmail-pubsub` correctement (il supprime le préfixe de chemin configuré).
Si vous avez besoin que le backend reçoive le chemin préfixé, définissez
`hooks.gmail.tailscale.target` sur une URL complète (et ajustez `serve.path` en conséquence).

<div id="canvashost-lantailnet-canvas-file-server-live-reload">
  ### `canvasHost` (serveur de fichiers Canvas sur LAN/tailnet + rechargement en direct)
</div>

Le Gateway sert un répertoire de fichiers HTML/CSS/JS via HTTP afin que les nœuds iOS/Android puissent simplement faire un `canvas.navigate` vers celui-ci.

Racine par défaut : `~/.openclaw/workspace/canvas`
Port par défaut : `18793` (choisi pour éviter le port CDP du navigateur openclaw `18792`)
Le serveur écoute sur l’**hôte de liaison du Gateway** (LAN ou Tailnet) afin que les nœuds puissent y accéder.

Le serveur :

* sert les fichiers sous `canvasHost.root`
* injecte un petit client de rechargement en direct dans le HTML servi
* surveille le répertoire et diffuse les rechargements via un endpoint WebSocket à `/__openclaw__/ws`
* crée automatiquement un `index.html` de démarrage lorsque le répertoire est vide (pour que vous voyiez immédiatement quelque chose)
* sert aussi A2UI à `/__openclaw__/a2ui/` et est exposé aux nœuds sous le nom `canvasHostUrl`
  (toujours utilisé par les nœuds pour Canvas/A2UI)

Désactivez le rechargement en direct (et la surveillance de fichiers) si le répertoire est volumineux ou si vous rencontrez `EMFILE` :

* config : `canvasHost: { liveReload: false }`

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    port: 18793,
    liveReload: true
  }
}
```

Les modifications de `canvasHost.*` nécessitent un redémarrage du Gateway (le rechargement de la configuration entraînera un redémarrage).

Désactivez avec :

* config : `canvasHost: { enabled: false }`
* env : `OPENCLAW_SKIP_CANVAS_HOST=1`

<div id="bridge-legacy-tcp-bridge-removed">
  ### `bridge` (pont TCP hérité, supprimé)
</div>

Les versions actuelles n’incluent plus l’écouteur de pont TCP ; les clés de configuration `bridge.*` sont ignorées.
Les nœuds se connectent via le WebSocket du Gateway. Cette section est conservée à des fins de référence historique.

Comportement historique :

* Le Gateway pouvait exposer un simple pont TCP pour les nœuds (iOS/Android), généralement sur le port `18790`.

Valeurs par défaut :

* enabled : `true`
* port : `18790`
* bind : `lan` (écoute sur `0.0.0.0`)

Modes de bind :

* `lan` : `0.0.0.0` (joignable sur n’importe quelle interface, y compris LAN/Wi‑Fi et Tailscale)
* `tailnet` : écoute uniquement sur l’adresse IP Tailscale de la machine (recommandé pour Vienne ⇄ Londres)
* `loopback` : `127.0.0.1` (local uniquement)
* `auto` : privilégie l’IP tailnet si présente, sinon `lan`

TLS :

* `bridge.tls.enabled` : active TLS pour les connexions du pont (TLS uniquement lorsqu’il est activé).
* `bridge.tls.autoGenerate` : génère un certificat auto-signé lorsqu’aucun certificat/clé n’est présent (par défaut : true).
* `bridge.tls.certPath` / `bridge.tls.keyPath` : chemins PEM pour le certificat du pont + la clé privée.
* `bridge.tls.caPath` : fichier PEM d’AC facultatif (racines personnalisées ou futur mTLS).

Lorsque TLS est activé, le Gateway annonce `bridgeTls=1` et `bridgeTlsSha256` dans les enregistrements
TXT de découverte pour que les nœuds puissent épingler le certificat. Les connexions manuelles
utilisent le modèle « trust-on-first-use » si aucune empreinte n’est encore stockée.
Les certificats générés automatiquement nécessitent `openssl` dans le PATH ; si la génération échoue, le pont ne démarrera pas.

```json5
{
  bridge: {
    enabled: true,
    port: 18790,
    bind: "tailnet",
    tls: {
      enabled: true,
      // Utilise ~/.openclaw/bridge/tls/bridge-{cert,key}.pem par défaut.
      // certPath: "~/.openclaw/bridge/tls/bridge-cert.pem",
      // keyPath: "~/.openclaw/bridge/tls/bridge-key.pem"
    }
  }
}
```

<div id="discoverymdns-bonjour-mdns-broadcast-mode">
  ### `discovery.mdns` (mode de diffusion Bonjour / mDNS)
</div>

Contrôle les annonces de découverte mDNS sur le LAN (`_openclaw-gw._tcp`).

* `minimal` (par défaut) : omet `cliPath` + `sshPort` dans les enregistrements TXT
* `full` : inclut `cliPath` + `sshPort` dans les enregistrements TXT
* `off` : désactive complètement les annonces mDNS
* Hostname : par défaut, `openclaw` (annonce `openclaw.local`). Peut être remplacé via `OPENCLAW_MDNS_HOSTNAME`.

```json5
{
  discovery: { mdns: { mode: "minimal" } }
}
```

<div id="discoverywidearea-wide-area-bonjour-unicast-dnssd">
  ### `discovery.wideArea` (Bonjour étendu / DNS‑SD unicast)
</div>

Lorsqu’il est activé, le Gateway écrit une zone DNS-SD unicast pour `_openclaw-gw._tcp` sous `~/.openclaw/dns/` en utilisant le domaine de découverte configuré (exemple : `openclaw.internal.`).

Pour permettre à iOS/Android de découvrir des services à travers plusieurs réseaux (Vienne ⇄ Londres), combine-la avec :

* un serveur DNS sur la machine hôte du Gateway servant le domaine de votre choix (CoreDNS est recommandé)
* le **split DNS** Tailscale pour que les clients résolvent ce domaine via le serveur DNS du Gateway

Assistant de configuration à exécuter une seule fois (hôte du Gateway) :

```bash
openclaw dns setup --apply
```

```json5
{
  discovery: { wideArea: { enabled: true } }
}
```

## Variables de template

Les espaces réservés de template sont développés dans `tools.media.*.models[].args` et `tools.media.models[].args` (ainsi que dans tout futur champ d’arguments utilisant un template).

| Variable | Description |
|----------|-------------|
| `{{Body}}` | Corps complet du message entrant |
| `{{RawBody}}` | Corps brut du message entrant (sans enveloppes d’historique/expéditeur ; idéal pour l’analyse de commandes) |
| `{{BodyStripped}}` | Corps avec les mentions de groupe supprimées (meilleur choix par défaut pour les agents) |
| `{{From}}` | Identifiant de l’expéditeur (E.164 pour WhatsApp ; peut varier selon le canal) |
| `{{To}}` | Identifiant de destination |
| `{{MessageSid}}` | ID du message du canal (lorsqu’il est disponible) |
| `{{SessionId}}` | UUID de la session en cours |
| `{{IsNewSession}}` | `"true"` lorsqu’une nouvelle session a été créée |
| `{{MediaUrl}}` | Pseudo-URL du média entrant (si présent) |
| `{{MediaPath}}` | Chemin local du média (s’il a été téléchargé) |
| `{{MediaType}}` | Type de média (image/audio/document/…) |
| `{{Transcript}}` | Transcription audio (lorsqu’elle est activée) |
| `{{Prompt}}` | Prompt média déterminé pour les entrées CLI |
| `{{MaxChars}}` | Nombre maximal de caractères de sortie déterminé pour les entrées CLI |
| `{{ChatType}}` | `"direct"` ou `"group"` |
| `{{GroupSubject}}` | Sujet du groupe (dans la mesure du possible) |
| `{{GroupMembers}}` | Aperçu des membres du groupe (dans la mesure du possible) |
| `{{SenderName}}` | Nom d’affichage de l’expéditeur (dans la mesure du possible) |
| `{{SenderE164}}` | Numéro de téléphone de l’expéditeur (dans la mesure du possible) |
| `{{Provider}}` | Indication de fournisseur (whatsapp|telegram|discord|googlechat|slack|signal|imessage|msteams|webchat|…) |

<div id="cron-gateway-scheduler">
  ## Cron (planificateur du Gateway)
</div>

Cron est un planificateur appartenant au Gateway pour les réveils et les tâches planifiées. Consulte la page [Tâches Cron](/fr/automation/cron-jobs) pour une vue d’ensemble de la fonctionnalité et des exemples de CLI.

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2
  }
}
```

***

*Suivant : [Environnement d’exécution de l’Agent](/fr/concepts/agent)* 🦞
