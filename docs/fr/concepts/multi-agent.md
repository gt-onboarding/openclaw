---
summary: "Routage multi-agents : agents isolés, comptes de canaux et liaisons"
title: Routage multi-agents
read_when: "Vous avez besoin de plusieurs agents isolés (espaces de travail + authentification) au sein d’un seul processus Gateway."
status: active
---

<div id="multi-agent-routing">
  # Routage multi-agents
</div>

Objectif : plusieurs agents *isolés* (espace de travail distinct + `agentDir` + sessions), plus plusieurs comptes de canaux (par ex. deux WhatsApp) dans un Gateway unique en cours d’exécution. Les messages entrants sont acheminés vers un agent via des liaisons.

<div id="what-is-one-agent">
  ## Qu’est-ce qu’« un agent » ?
</div>

Un **agent** est un cerveau entièrement autonome, avec son propre :

* **Espace de travail** (fichiers, AGENTS.md/SOUL.md/USER.md, notes locales, règles de persona).
* **Répertoire d’état** (`agentDir`) pour les profils d’authentification, le registre de modèles et la configuration propre à l’agent.
* **Stockage des sessions** (historique de discussion + état de routage) sous `~/.openclaw/agents/<agentId>/sessions`.

Les profils d’authentification sont **par agent**. Chaque agent lit à partir de son propre :

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Les identifiants de l’agent principal ne sont **pas** partagés automatiquement. Ne réutilisez jamais `agentDir`
entre plusieurs agents (cela provoque des collisions d’auth/session). Si vous voulez partager des identifiants,
copiez `auth-profiles.json` dans le `agentDir` de l’autre agent.

Les compétences sont propres à chaque agent via le dossier `skills/` de chaque espace de travail, avec des compétences partagées
disponibles dans `~/.openclaw/skills`. Voir [Compétences : par agent vs partagées](/fr/tools/skills#per-agent-vs-shared-skills).

Le Gateway peut héberger **un agent** (par défaut) ou **plusieurs agents** côte à côte.

**Remarque sur l’espace de travail :** l’espace de travail de chaque agent est le **cwd par défaut**, pas un
sandbox strict. Les chemins relatifs se résolvent à l’intérieur de l’espace de travail, mais les chemins absolus peuvent
atteindre d’autres emplacements de l’hôte, sauf si le sandboxing est activé. Voir
[Sandboxing](/fr/gateway/sandboxing).

<div id="paths-quick-map">
  ## Chemins (vue d&#39;ensemble rapide)
</div>

* Configuration : `~/.openclaw/openclaw.json` (ou `OPENCLAW_CONFIG_PATH`)
* Répertoire d’état : `~/.openclaw` (ou `OPENCLAW_STATE_DIR`)
* Espace de travail : `~/.openclaw/workspace` (ou `~/.openclaw/workspace-<agentId>`)
* Répertoire de l’agent : `~/.openclaw/agents/<agentId>/agent` (ou `agents.list[].agentDir`)
* Sessions : `~/.openclaw/agents/<agentId>/sessions`

<div id="single-agent-mode-default">
  ### Mode agent unique (par défaut)
</div>

Si vous ne faites rien, OpenClaw exécute un seul agent :

* `agentId` a pour valeur par défaut **`main`**.
* Les sessions sont identifiées par `agent:main:<mainKey>`.
* L’espace de travail par défaut est `~/.openclaw/workspace` (ou `~/.openclaw/workspace-<profile>` lorsque `OPENCLAW_PROFILE` est défini).
* L’état par défaut est stocké dans `~/.openclaw/agents/main/agent`.

<div id="agent-helper">
  ## Assistant de configuration d’agent
</div>

Utilisez l’assistant de configuration d’agent pour ajouter un nouvel agent isolé :

```bash
openclaw agents add work
```

Ajoutez ensuite des `bindings` (ou laissez l’assistant le faire) pour acheminer les messages entrants.

Vérifiez avec :

```bash
openclaw agents list --bindings
```

<div id="multiple-agents-multiple-people-multiple-personalities">
  ## Plusieurs agents = plusieurs personnes, plusieurs personnalités
</div>

Avec **plusieurs agents**, chaque `agentId` devient une **personnalité entièrement isolée** :

* **Différents numéros de téléphone/comptes** (par `accountId` de canal).
* **Différentes personnalités** (fichiers d’espace de travail propres à chaque agent comme `AGENTS.md` et `SOUL.md`).
* **Authentification et sessions séparées** (aucune interaction croisée sauf si elle est explicitement activée).

Cela permet à **plusieurs personnes** de partager un serveur Gateway tout en gardant leurs « cerveaux » IA et leurs données isolés.

<div id="one-whatsapp-number-multiple-people-dm-split">
  ## Un numéro WhatsApp, plusieurs personnes (DM séparés)
</div>

Vous pouvez rediriger **différents DMs WhatsApp** vers différents agents tout en restant sur **un seul compte WhatsApp**. Faites correspondre l’expéditeur au format E.164 (par exemple `+15551234567`) avec `peer.kind: "dm"`. Les réponses proviennent toujours du même numéro WhatsApp (pas d’identité d’expéditeur propre à chaque agent).

Détail important : les discussions directes sont regroupées sous la **clé de session principale** de l’agent, donc une véritable isolation nécessite **un agent par personne**.

Exemple :

```json5
{
  agents: {
    list: [
      { id: "alex", workspace: "~/.openclaw/workspace-alex" },
      { id: "mia", workspace: "~/.openclaw/workspace-mia" }
    ]
  },
  bindings: [
    { agentId: "alex", match: { channel: "whatsapp", peer: { kind: "dm", id: "+15551230001" } } },
    { agentId: "mia",  match: { channel: "whatsapp", peer: { kind: "dm", id: "+15551230002" } } }
  ],
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551230001", "+15551230002"]
    }
  }
}
```

Notes :

* Le contrôle d’accès en DM est **global par compte WhatsApp** (appairage/liste d’autorisation), et non par agent.
* Pour les groupes partagés, associez le groupe à un seul agent ou utilisez les [Broadcast groups](/fr/broadcast-groups).

<div id="routing-rules-how-messages-pick-an-agent">
  ## Règles de routage (comment les messages choisissent un agent)
</div>

Les associations sont **déterministes** et **la règle la plus spécifique l’emporte** :

1. correspondance sur `peer` (ID exact de MP/groupe/canal)
2. `guildId` (Discord)
3. `teamId` (Slack)
4. correspondance sur `accountId` pour un canal
5. correspondance au niveau du canal (`accountId: "*"`)
6. repli sur l’agent par défaut (`agents.list[].default`, sinon première entrée de la liste, par défaut : `main`)

<div id="multiple-accounts-phone-numbers">
  ## Plusieurs comptes / numéros de téléphone
</div>

Les canaux qui prennent en charge **plusieurs comptes** (par exemple WhatsApp) utilisent `accountId` pour identifier chaque compte. Chaque `accountId` peut être routé vers un agent différent, ce qui permet à un seul serveur d’héberger plusieurs numéros de téléphone sans mélanger les sessions.

<div id="concepts">
  ## Concepts
</div>

* `agentId` : un « cerveau » (espace de travail, authentification propre à l’agent, stockage de session propre à l’agent).
* `accountId` : une instance de compte de canal (p. ex. compte WhatsApp `"personal"` vs `"biz"`).
* `binding` : achemine les messages entrants vers un `agentId` en fonction de `(channel, accountId, peer)` et éventuellement d’identifiants de guild/team.
* Les conversations directes sont regroupées sous `agent:<agentId>:<mainKey>` (clé « principale » propre à l’agent ; `session.mainKey`).

<div id="example-two-whatsapps-two-agents">
  ## Exemple : deux comptes WhatsApp → deux agents
</div>

`~/.openclaw/openclaw.json` (JSON5) :

```js
{
  agents: {
    list: [
      {
        id: "home",
        default: true,
        name: "Home",
        workspace: "~/.openclaw/workspace-home",
        agentDir: "~/.openclaw/agents/home/agent",
      },
      {
        id: "work",
        name: "Work",
        workspace: "~/.openclaw/workspace-work",
        agentDir: "~/.openclaw/agents/work/agent",
      },
    ],
  },

  // Deterministic routing: first match wins (most-specific first).
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },

    // Optional per-peer override (example: send a specific group to work agent).
    {
      agentId: "work",
      match: {
        channel: "whatsapp",
        accountId: "personal",
        peer: { kind: "group", id: "1203630...@g.us" },
      },
    },
  ],

  // Désactivé par défaut : la messagerie agent-à-agent doit être explicitement activée et ajoutée à la liste d'autorisation.
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },

  channels: {
    whatsapp: {
      accounts: {
        personal: {
          // Optional override. Default: ~/.openclaw/credentials/whatsapp/personal
          // authDir: "~/.openclaw/credentials/whatsapp/personal",
        },
        biz: {
          // Optional override. Default: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

<div id="example-whatsapp-daily-chat-telegram-deep-work">
  ## Exemple : discussion quotidienne sur WhatsApp + travail approfondi sur Telegram
</div>

Répartissez par canal : acheminez WhatsApp vers un agent rapide pour l’usage quotidien et Telegram vers un agent Opus.

```json5
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5"
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-5"
      }
    ]
  },
  bindings: [
    { agentId: "chat", match: { channel: "whatsapp" } },
    { agentId: "opus", match: { channel: "telegram" } }
  ]
}
```

Notes :

* Si vous avez plusieurs comptes pour un même canal, ajoutez `accountId` au binding (par exemple `{ channel: "whatsapp", accountId: "personal" }`).
* Pour router un seul DM/groupe vers Opus tout en laissant le reste sur chat, ajoutez un binding `match.peer` pour ce peer ; les correspondances de peer priment toujours sur les règles globales du canal.

<div id="example-same-channel-one-peer-to-opus">
  ## Exemple : même canal, un seul correspondant vers Opus
</div>

Laisse WhatsApp sur l’agent rapide, mais redirige un DM vers Opus :

```json5
{
  agents: {
    list: [
      { id: "chat", name: "Everyday", workspace: "~/.openclaw/workspace-chat", model: "anthropic/claude-sonnet-4-5" },
      { id: "opus", name: "Deep Work", workspace: "~/.openclaw/workspace-opus", model: "anthropic/claude-opus-4-5" }
    ]
  },
  bindings: [
    { agentId: "opus", match: { channel: "whatsapp", peer: { kind: "dm", id: "+15551234567" } } },
    { agentId: "chat", match: { channel: "whatsapp" } }
  ]
}
```

Les liaisons propres à un pair l’emportent toujours ; gardez-les donc au-dessus de la règle globale du canal.

<div id="family-agent-bound-to-a-whatsapp-group">
  ## Agent familial associé à un groupe WhatsApp
</div>

Associez un agent familial dédié à un seul groupe WhatsApp, avec activation par mention
et une politique d’outils plus stricte :

```json5
{
  agents: {
    list: [
      {
        id: "family",
        name: "Family",
        workspace: "~/.openclaw/workspace-family",
        identity: { name: "Family Bot" },
        groupChat: {
          mentionPatterns: ["@family", "@familybot", "@Family Bot"]
        },
        sandbox: {
          mode: "all",
          scope: "agent"
        },
        tools: {
          allow: ["exec", "read", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
          deny: ["write", "edit", "apply_patch", "browser", "canvas", "nodes", "cron"]
        }
      }
    ]
  },
  bindings: [
    {
      agentId: "family",
      match: {
        channel: "whatsapp",
        peer: { kind: "group", id: "120363999999999999@g.us" }
      }
    }
  ]
}
```

Remarques :

* Les listes d’autorisation/de refus d’outils sont des **tools**, pas des compétences. Si une compétence doit exécuter un binaire, assurez-vous que `exec` est autorisé et que le binaire existe dans le sandbox.
* Pour un filtrage plus strict, configurez `agents.list[].groupChat.mentionPatterns` et laissez les listes d’autorisation de groupe activées pour le canal.

<div id="per-agent-sandbox-and-tool-configuration">
  ## Configuration du sandbox et des outils par agent
</div>

À partir de la v2026.1.6, chaque agent peut avoir son propre sandbox et ses propres restrictions d’outils :

```js
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: {
          mode: "off",  // No sandbox for personal agent
        },
        // No tool restrictions - all tools available
      },
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",     // Always sandboxed
          scope: "agent",  // One container per agent
          docker: {
            // Optional one-time setup after container creation
            setupCommand: "apt-get update && apt-get install -y git curl",
          },
        },
        tools: {
          allow: ["read"],                    // Only read tool
          deny: ["exec", "write", "edit", "apply_patch"],    // Refuser les autres
        },
      },
    ],
  },
}
```

Remarque : `setupCommand` se trouve sous `sandbox.docker` et s’exécute une fois lors de la création du conteneur.
Les remplacements `sandbox.docker.*` par agent sont ignorés lorsque la portée résolue est `"shared"`.

**Avantages :**

* **Isolation de sécurité** : restreindre les outils pour les agents non approuvés
* **Contrôle des ressources** : exécuter certains agents dans un sandbox tout en laissant les autres sur l’hôte
* **Politiques flexibles** : permissions différentes par agent

Remarque : `tools.elevated` est **global** et dépend de l’expéditeur ; il n’est pas configurable par agent.
Si vous avez besoin de limites par agent, utilisez `agents.list[].tools` pour refuser l’accès à `exec`.
Pour le ciblage de groupe, utilisez `agents.list[].groupChat.mentionPatterns` afin que les @mentions correspondent clairement à l’agent visé.

Voir [Multi-Agent Sandbox &amp; Tools](/fr/multi-agent-sandbox-tools) pour des exemples détaillés.
