---
title: Messages de groupe
summary: "Comportement et configuration de la gestion des messages de groupe WhatsApp (les mentionPatterns sont partagés entre les différentes interfaces)"
read_when:
  - Modification des règles de gestion des messages de groupe ou des mentions
---

<div id="group-messages-whatsapp-web-channel">
  # Messages de groupe (canal web WhatsApp)
</div>

Objectif : faire en sorte que Clawd reste dans des groupes WhatsApp, ne s’active que lorsqu’il est mentionné, et garde ce fil séparé de la session de messages privés (DM) personnelle.

Remarque : `agents.list[].groupChat.mentionPatterns` est désormais également utilisé par Telegram/Discord/Slack/iMessage ; ce document se concentre sur le comportement spécifique à WhatsApp. Pour les configurations multi-agents, définissez `agents.list[].groupChat.mentionPatterns` par agent (ou utilisez `messages.groupChat.mentionPatterns` comme configuration globale de repli).

<div id="whats-implemented-2025-12-03">
  ## Qu’est-ce qui est implémenté (2025-12-03)
</div>

- Modes d’activation : `mention` (par défaut) ou `always`. `mention` nécessite un ping (vraies @‑mentions WhatsApp via `mentionedJids`, expressions régulières/regex, ou l’E.164 du bot n’importe où dans le texte). `always` réveille l’agent à chaque message, mais il ne doit répondre que lorsqu’il peut apporter une valeur significative ; sinon il renvoie le jeton silencieux `NO_REPLY`. Les valeurs par défaut peuvent être définies dans la config (`channels.whatsapp.groups`) et remplacées par groupe via `/activation`. Lorsque `channels.whatsapp.groups` est défini, il sert aussi de liste d’autorisation de groupes (inclure `"*"` pour tout autoriser).
- Politique de groupe : `channels.whatsapp.groupPolicy` contrôle si les messages de groupe sont acceptés (`open|disabled|allowlist`). `allowlist` utilise `channels.whatsapp.groupAllowFrom` (solution de repli : `channels.whatsapp.allowFrom` explicite). La valeur par défaut est `allowlist` (bloqué tant que vous n’ajoutez pas d’expéditeurs).
- Sessions par groupe : les clés de session ressemblent à `agent:<agentId>:whatsapp:group:<jid>` afin que des commandes comme `/verbose on` ou `/think high` (envoyées comme messages indépendants) soient limitées à ce groupe ; l’état en message privé (DM) personnel n’est pas modifié. Les signaux de vie sont ignorés pour les fils de discussion de groupe.
- Injection de contexte : les messages de groupe **en attente uniquement** (50 par défaut) qui *n’ont pas* déclenché d’exécution sont préfixés sous `[Chat messages since your last reply - for context]`, avec la ligne déclenchante sous `[Current message - respond to this]`. Les messages déjà présents dans la session ne sont pas réinjectés.
- Identification de l’expéditeur : chaque lot de messages de groupe se termine désormais par `[from: Sender Name (+E164)]` afin que Pi sache qui parle.
- Éphémères/affichage unique : nous les déroulons avant d’extraire le texte/les mentions, de sorte que les pings à l’intérieur continuent de déclencher.
- Invite système de groupe : au premier tour d’une session de groupe (et chaque fois que `/activation` modifie le mode) nous injectons un court texte explicatif dans l’invite système, du type `You are replying inside the WhatsApp group "<subject>". Group members: Alice (+44...), Bob (+43...), … Activation: trigger-only … Address the specific sender noted in the message context.`. Si les métadonnées ne sont pas disponibles, nous indiquons quand même à l’agent qu’il s’agit d’un chat de groupe.

<div id="config-example-whatsapp">
  ## Exemple de configuration (WhatsApp)
</div>

Ajoutez un bloc `groupChat` dans `~/.openclaw/openclaw.json` pour que les mentions par nom d’affichage fonctionnent même lorsque WhatsApp retire l’« @ » visuel dans le corps du message :

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true }
      }
    }
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          historyLimit: 50,
          mentionPatterns: [
            "@?openclaw",
            "\\+?15555550123"
          ]
        }
      }
    ]
  }
}
```

Notes :

* Les expressions régulières sont insensibles à la casse ; elles couvrent une mention via le nom affiché, comme `@openclaw`, ainsi que le numéro brut avec ou sans `+`/espaces.
* WhatsApp envoie toujours des mentions canoniques via `mentionedJids` lorsque quelqu’un appuie sur le contact, donc la solution de repli basée sur le numéro est rarement nécessaire mais reste un filet de sécurité utile.


<div id="activation-command-owner-only">
  ### Commande d’activation (propriétaire uniquement)
</div>

Utilisez la commande dans le chat de groupe :

- `/activation mention`
- `/activation always`

Seul le numéro du propriétaire (défini dans `channels.whatsapp.allowFrom`, ou le numéro E.164 du bot si ce champ n’est pas renseigné) peut modifier ce paramètre. Envoyez `/status` comme message autonome dans le groupe pour afficher le mode d’activation actuel.

<div id="how-to-use">
  ## Comment l’utiliser
</div>

1) Ajoutez votre compte WhatsApp (celui qui fait tourner OpenClaw) au groupe.
2) Dites `@openclaw …` (ou incluez le numéro). Seuls les expéditeurs figurant dans la liste d’autorisation peuvent le déclencher, sauf si vous définissez `groupPolicy: "open"` (paramètre qui permet d’accepter les messages de n’importe quel utilisateur sans restriction).
3) Le prompt de l’agent inclura le contexte récent du groupe, ainsi que le marqueur final `[from: …]` afin qu’il puisse s’adresser à la bonne personne.
4) Les directives au niveau de la session (`/verbose on`, `/think high`, `/new` ou `/reset`, `/compact`) s’appliquent uniquement à la session de ce groupe ; envoyez-les sous forme de messages séparés pour qu’elles soient prises en compte. Votre session de messages privés (DM) reste indépendante.

<div id="testing-verification">
  ## Tests / vérification
</div>

- Smoke test manuel :
  - Envoyez un ping `@openclaw` dans le groupe et confirmez qu’une réponse fait référence au nom de l’expéditeur.
  - Envoyez un deuxième ping et vérifiez que le bloc d’historique est inclus, puis qu’il est effacé au tour suivant.
- Vérifiez les journaux du Gateway (en l’exécutant avec `--verbose`) pour voir des entrées `inbound web message` affichant `from: <groupJid>` et le suffixe `[from: …]`.

<div id="known-considerations">
  ## Considérations connues
</div>

- Les signaux de vie sont délibérément ignorés pour les groupes afin d’éviter des diffusions trop bruyantes.
- La suppression d’écho utilise la chaîne de batch combinée ; si vous envoyez deux fois le même texte sans mentions, seule la première recevra une réponse.
- Les entrées du stockage des sessions apparaîtront sous la forme `agent:<agentId>:whatsapp:group:<jid>` dans le stockage des sessions (`~/.openclaw/agents/<agentId>/sessions/sessions.json` par défaut) ; une entrée manquante signifie simplement que le groupe n’a pas encore déclenché d’exécution.
- Les indicateurs de saisie dans les groupes respectent `agents.defaults.typingMode` (valeur par défaut : `message` lorsqu’il n’y a pas de mention).