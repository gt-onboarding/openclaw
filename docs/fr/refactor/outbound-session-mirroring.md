---
title: Refactorisation de la mise en miroir des sessions sortantes (Issue #1520)
description: Suivi des notes, décisions, tests et points en suspens liés à la refactorisation de la mise en miroir des sessions sortantes.
---

<div id="outbound-session-mirroring-refactor-issue-1520">
  # Refactorisation de la mise en miroir des sessions sortantes (Issue #1520)
</div>

<div id="status">
  ## Statut
</div>

- En cours.
- Routage des canaux Core + plugin mis à jour pour la mise en miroir sortante.
- Gateway send déduit désormais la session cible lorsque `sessionKey` n’est pas fourni.

<div id="context">
  ## Contexte
</div>

Les envois sortants étaient recopiés dans la session d’agent *en cours* (clé de session d’outil) plutôt que dans la session du canal cible. Le routage entrant utilise les clés de session canal/peer, donc les réponses sortantes arrivaient dans la mauvaise session et les cibles lors du premier contact se retrouvaient souvent sans entrée de session.

<div id="goals">
  ## Objectifs
</div>

- Répliquer les messages sortants dans la clé de session du canal cible.
- Créer des entrées de session en sortie lorsqu’elles n’existent pas.
- Garder la portée des fils/sujets alignée avec les clés de session entrantes.
- Couvrir les canaux principaux ainsi que les extensions intégrées.

<div id="implementation-summary">
  ## Résumé de l’implémentation
</div>

- Nouvel utilitaire de routage de session sortante :
  - `src/infra/outbound/outbound-session.ts`
  - `resolveOutboundSessionRoute` construit la sessionKey cible en utilisant `buildAgentSessionKey` (dmScope + identityLinks).
  - `ensureOutboundSessionEntry` écrit un `MsgContext` minimal via `recordSessionMetaFromInbound`.
- `runMessageAction` (Envoyer) dérive la sessionKey cible et la transmet à `executeSendAction` pour la mise en miroir.
- `message-tool` ne fait plus de mise en miroir directement ; il se contente de résoudre `agentId` à partir de la sessionKey actuelle.
- Le chemin d’envoi du plugin effectue la mise en miroir via `appendAssistantMessageToSessionTranscript` en utilisant la sessionKey dérivée.
- Lors d’un envoi via le Gateway, une sessionKey cible est dérivée lorsqu’aucune n’est fournie (agent par défaut), et l’existence d’une entrée de session est assurée.

<div id="threadtopic-handling">
  ## Gestion des fils/sujets
</div>

- Slack : replyTo/threadId -> `resolveThreadSessionKeys` (suffixe).
- Discord : threadId/replyTo -> `resolveThreadSessionKeys` avec `useSuffix=false` pour correspondre aux messages entrants (lʼID du canal de fil délimite déjà la session).
- Telegram : les identifiants de sujet sont associés à `chatId:topic:<id>` via `buildTelegramGroupPeerId`.

<div id="extensions-covered">
  ## Extensions couvertes
</div>

- Matrix, MS Teams, Mattermost, BlueBubbles, Nextcloud Talk, Zalo, Zalo Personal, Nostr, Tlon.
- Remarques :
  - Les cibles Mattermost suppriment désormais `@` pour le routage des clés de session de DM.
  - Zalo Personal utilise le type de pair de DM pour les cibles 1:1 (groupe uniquement lorsque `group:` est présent).
  - Les cibles de groupe BlueBubbles suppriment les préfixes `chat_*` pour faire correspondre les clés de session entrantes.
  - La mise en miroir automatique des fils Slack compare les identifiants de canal sans tenir compte de la casse.
  - La méthode Gateway `send` met en minuscules les clés de session fournies avant la mise en miroir.

<div id="decisions">
  ## Décisions
</div>

- **Dérivation de session pour l’envoi via le Gateway** : si `sessionKey` est fourni, l’utiliser. S’il est absent, dériver un `sessionKey` à partir de la cible + de l’agent par défaut et y répliquer la session.
- **Création d’entrée de session** : toujours utiliser `recordSessionMetaFromInbound` avec `Provider/From/To/ChatType/AccountId/Originating*` alignés sur les formats entrants.
- **Normalisation de la cible** : le routage sortant utilise les cibles résolues (après `resolveChannelTarget`) lorsqu’elles sont disponibles.
- **Casse des clés de session** : normaliser les clés de session en minuscules lors de l’écriture et pendant les migrations.

<div id="tests-addedupdated">
  ## Tests ajoutés/mis à jour
</div>

- `src/infra/outbound/outbound-session.test.ts`
  - Clé de session de thread Slack.
  - Clé de session de sujet Telegram.
  - dmScope identityLinks avec Discord.
- `src/agents/tools/message-tool.test.ts`
  - Déduit agentId à partir de la clé de session (aucun sessionKey transmis).
- `src/gateway/server-methods/send.test.ts`
  - Déduit la clé de session lorsqu’elle est omise et crée une entrée de session.

<div id="open-items-follow-ups">
  ## Points ouverts / Suivis
</div>

- Le plugin d'appels vocaux utilise des clés de session personnalisées `voice:<phone>`. Le mappage sortant n'est pas standardisé ici ; si `message-tool` doit prendre en charge l’envoi d’appels vocaux, ajoutez un mappage explicite.
- Confirmer si un plugin externe utilise des formats `From/To` non standard au‑delà de l’ensemble fourni.

<div id="files-touched">
  ## Fichiers concernés
</div>

- `src/infra/outbound/outbound-session.ts`
- `src/infra/outbound/outbound-send-service.ts`
- `src/infra/outbound/message-action-runner.ts`
- `src/agents/tools/message-tool.ts`
- `src/gateway/server-methods/send.ts`
- Tests :
  - `src/infra/outbound/outbound-session.test.ts`
  - `src/agents/tools/message-tool.test.ts`
  - `src/gateway/server-methods/send.test.ts`