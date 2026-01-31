---
title: Routage des canaux
summary: "Règles de routage par canal (WhatsApp, Telegram, Discord, Slack) et contexte partagé"
read_when:
  - Modifier le routage des canaux ou le comportement de la boîte de réception
---

<div id="channels-routing">
  # Canaux et routage
</div>

OpenClaw achemine les réponses **vers le canal d&#39;où le message est issu**. Le
modèle ne choisit pas le canal ; le routage est déterministe et contrôlé par la
configuration de l&#39;hôte.

<div id="key-terms">
  ## Termes clés
</div>

* **Channel** : `whatsapp`, `telegram`, `discord`, `slack`, `signal`, `imessage`, `webchat`.
* **AccountId** : instance de compte spécifique à chaque Channel (lorsque pris en charge).
* **AgentId** : un espace de travail et un stockage de sessions isolés (« cerveau »).
* **SessionKey** : la clé de compartiment (bucket) utilisée pour stocker le contexte et contrôler la concurrence d’accès.

<div id="session-key-shapes-examples">
  ## Formats de clés de session (exemples)
</div>

Les messages directs sont rattachés à la session **principale** de l’agent :

* `agent:<agentId>:<mainKey>` (par défaut : `agent:main:main`)

Les groupes et canaux restent isolés par canal :

* Groupes : `agent:<agentId>:<channel>:group:<id>`
* Canaux/salons : `agent:<agentId>:<channel>:channel:<id>`

Fils de discussion :

* Les fils Slack/Discord ajoutent `:thread:<threadId>` à la clé de base.
* Les sujets de forum Telegram intègrent `:topic:<topicId>` dans la clé de groupe.

Exemples :

* `agent:main:telegram:group:-1001234567890:topic:42`
* `agent:main:discord:channel:123456:thread:987654`

<div id="routing-rules-how-an-agent-is-chosen">
  ## Règles de routage (comment un agent est sélectionné)
</div>

Le routage sélectionne **un seul agent** pour chaque message entrant :

1. **Correspondance exacte de pair** (`bindings` avec `peer.kind` + `peer.id`).
2. **Correspondance de guilde** (Discord) via `guildId`.
3. **Correspondance d&#39;équipe** (Slack) via `teamId`.
4. **Correspondance de compte** (`accountId` au niveau du canal).
5. **Correspondance de canal** (n&#39;importe quel compte sur ce canal).
6. **Agent par défaut** (`agents.list[].default`, sinon première entrée de la liste, ou à défaut `main`).

L&#39;agent correspondant détermine quel espace de travail et quel stockage de sessions sont utilisés.

<div id="broadcast-groups-run-multiple-agents">
  ## Groupes de diffusion (exécuter plusieurs agents)
</div>

Les groupes de diffusion vous permettent d’exécuter **plusieurs agents** pour le même interlocuteur **au moment où OpenClaw répondrait normalement** (par exemple : dans les groupes WhatsApp, après la mention/activation).

Config :

```json5
{
  broadcast: {
    strategy: "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"],
    "+15555550123": ["support", "logger"]
  }
}
```

Voir : [Groupes de diffusion](/fr/broadcast-groups).

<div id="config-overview">
  ## Aperçu de la configuration
</div>

* `agents.list` : définitions d&#39;agents nommés (espace de travail, modèle, etc.).
* `bindings` : associe les canaux, comptes et pairs entrants aux agents.

Exemple :

```json5
{
  agents: {
    list: [
      { id: "support", name: "Support", workspace: "~/.openclaw/workspace-support" }
    ]
  },
  bindings: [
    { match: { channel: "slack", teamId: "T123" }, agentId: "support" },
    { match: { channel: "telegram", peer: { kind: "group", id: "-100123" } }, agentId: "support" }
  ]
}
```

<div id="session-storage">
  ## Stockage des sessions
</div>

Les données de session sont stockées dans le répertoire d’état (par défaut `~/.openclaw`) :

* `~/.openclaw/agents/<agentId>/sessions/sessions.json`
* Les transcriptions JSONL sont stockées à côté du fichier de stockage

Vous pouvez surcharger le chemin de stockage via `session.store` et le modèle `{agentId}`.

<div id="webchat-behavior">
  ## Comportement de WebChat
</div>

WebChat se connecte à l’**agent sélectionné** et utilise par défaut la session
principale de cet agent. Grâce à cela, WebChat vous permet de voir le contexte
multicanal de cet agent en un seul endroit.

<div id="reply-context">
  ## Contexte de réponse
</div>

Les réponses entrantes incluent :

* `ReplyToId`, `ReplyToBody` et `ReplyToSender` lorsqu’ils sont disponibles.
* Le contexte cité est ajouté à `Body` sous la forme d’un bloc `[Replying to ...]`.

Ce comportement est cohérent sur l’ensemble des canaux.