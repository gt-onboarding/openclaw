---
title: Outil de session
summary: "Outils de session d’Agent pour lister les sessions, récupérer l’historique et envoyer des messages entre sessions"
read_when:
  - Ajout ou modification des outils de session
---

<div id="session-tools">
  # Outils de session
</div>

Objectif : un petit ensemble d’outils peu sujets aux erreurs, pour permettre aux agents de lister les sessions, de consulter l’historique et d’envoyer vers une autre session.

<div id="tool-names">
  ## Noms des outils
</div>

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

<div id="key-model">
  ## Modèle de clé
</div>

- Le bucket principal de discussion directe est toujours la clé littérale `"main"` (résolue vers la clé principale de l’agent actuel).
- Les discussions de groupe utilisent `agent:<agentId>:<channel>:group:<id>` ou `agent:<agentId>:<channel>:channel:<id>` (transmettez la clé complète).
- Les tâches Cron utilisent `cron:<job.id>`.
- Les hooks utilisent `hook:<uuid>` sauf si elles sont définies explicitement.
- Les sessions de nœud utilisent `node-<nodeId>` sauf si elles sont définies explicitement.

`global` et `unknown` sont des valeurs réservées et ne sont jamais répertoriées. Si `session.scope = "global"`, nous le faisons correspondre à `main` pour tous les outils afin que les appelants ne voient jamais `global`.

<div id="sessions_list">
  ## sessions_list
</div>

Répertorie les sessions sous forme de tableau de lignes.

Paramètres :

- `kinds?: string[]` filtre : n’importe lequel parmi `"main" | "group" | "cron" | "hook" | "node" | "other"`
- `limit?: number` nombre maximal de lignes (par défaut : valeur par défaut du serveur, limité p. ex. à 200)
- `activeMinutes?: number` uniquement les sessions mises à jour dans les N dernières minutes
- `messageLimit?: number` 0 = aucun message (par défaut 0) ; >0 = inclure les N derniers messages

Comportement :

- `messageLimit > 0` récupère `chat.history` par session et inclut les N derniers messages.
- Les résultats des outils sont exclus de la sortie de la liste ; utilisez `sessions_history` pour les messages d’outils.
- Lors de l’exécution dans une session d’agent en **sandbox**, les outils de session ont par défaut une **visibilité limitée aux sessions enfant uniquement** (voir ci-dessous).

Structure de ligne (JSON) :

- `key` : clé de session (string)
- `kind` : `main | group | cron | hook | node | other`
- `channel` : `whatsapp | telegram | discord | signal | imessage | webchat | internal | unknown`
- `displayName` (étiquette d’affichage de groupe si disponible)
- `updatedAt` (ms)
- `sessionId`
- `model`, `contextTokens`, `totalTokens`
- `thinkingLevel`, `verboseLevel`, `systemSent`, `abortedLastRun`
- `sendPolicy` (surcharge au niveau de la session si définie)
- `lastChannel`, `lastTo`
- `deliveryContext` (`{ channel, to, accountId }` normalisé lorsque disponible)
- `transcriptPath` (chemin calculé au mieux à partir du répertoire de stockage + sessionId)
- `messages?` (uniquement lorsque `messageLimit > 0`)

<div id="sessions_history">
  ## sessions_history
</div>

Récupère la transcription pour une session.

Paramètres :

- `sessionKey` (obligatoire ; accepte une clé de session ou un `sessionId` provenant de `sessions_list`)
- `limit?: number` nombre maximal de messages (limite appliquée côté serveur)
- `includeTools?: boolean` (false par défaut)

Comportement :

- `includeTools=false` filtre les messages `role: "toolResult"`.
- Renvoie un tableau de messages au format brut de transcription.
- Lorsqu’un `sessionId` est fourni, OpenClaw le fait correspondre à la clé de session associée (erreur en cas d’identifiant manquant).

<div id="sessions_send">
  ## sessions_send
</div>

Envoyer un message dans une autre session.

Paramètres :

- `sessionKey` (obligatoire ; accepte une clé de session ou un `sessionId` provenant de `sessions_list`)
- `message` (obligatoire)
- `timeoutSeconds?: number` (valeur par défaut > 0 ; 0 = envoi « fire‑and‑forget », sans attendre de réponse)

Comportement :

- `timeoutSeconds = 0` : met en file d’attente et renvoie `{ runId, status: "accepted" }`.
- `timeoutSeconds > 0` : attend jusqu’à N secondes la fin de l’exécution, puis renvoie `{ runId, status: "ok", reply }`.
- Si le délai d’attente est dépassé : `{ runId, status: "timeout", error }`. L’exécution continue ; appelez `sessions_history` plus tard.
- Si l’exécution échoue : `{ runId, status: "error", error }`.
- Les exécutions d’annonce de livraison sont déclenchées après la fin de l’exécution principale et sont en mode best‑effort ; `status: "ok"` ne garantit pas que l’annonce a été livrée.
- Attend via la méthode `agent.wait` du Gateway (côté serveur), de sorte que les reconnexions ne coupent pas l’attente.
- Le contexte de message agent‑à‑agent est injecté pour l’exécution principale.
- Après la fin de l’exécution principale, OpenClaw exécute une **boucle de renvoi de réponse** :
  - À partir du deuxième tour, l’échange alterne entre l’agent demandeur et l’agent cible.
  - Répondez exactement `REPLY_SKIP` pour arrêter le ping‑pong.
  - Le nombre maximal d’itérations est `session.agentToAgent.maxPingPongTurns` (0–5, 5 par défaut).
- Une fois la boucle terminée, OpenClaw exécute l’**étape d’annonce agent‑à‑agent** (agent cible uniquement) :
  - Répondez exactement `ANNOUNCE_SKIP` pour rester silencieux.
  - Toute autre réponse est envoyée au canal cible.
  - L’étape d’annonce inclut la demande initiale + la réponse du premier tour + la dernière réponse de ping‑pong.

<div id="channel-field">
  ## Champ `channel`
</div>

- Pour les groupes, `channel` est le canal enregistré dans l’entrée de session.
- Pour les conversations directes, `channel` est renseigné à partir de `lastChannel`.
- Pour cron/hook/nœud, `channel` est `internal`.
- S’il est absent, `channel` est `unknown`.

<div id="security-send-policy">
  ## Sécurité / Politique d’envoi
</div>

Blocage fondé sur des politiques, appliqué par type de canal/chat (et non par identifiant de session).

```json
{
  "session": {
    "sendPolicy": {
      "rules": [
        {
          "match": { "channel": "discord", "chatType": "group" },
          "action": "deny"
        }
      ],
      "default": "allow"
    }
  }
}
```

Surcharge au runtime (par entrée de session) :

* `sendPolicy: "allow" | "deny"` (non défini = hérite de la configuration)
* Paramétrable via `sessions.patch` ou `/send on|off|inherit` réservé au propriétaire (message autonome).

Points d’application :

* `chat.send` / `agent` (Gateway)
* logique de livraison des réponses automatiques


<div id="sessions_spawn">
  ## sessions_spawn
</div>

Lance l’exécution d’un sous-agent dans une session isolée et annonce le résultat sur le canal de discussion du demandeur.

Paramètres :

- `task` (obligatoire)
- `label?` (optionnel ; utilisé pour les logs/UI)
- `agentId?` (optionnel ; lance sous un autre id d’agent si autorisé)
- `model?` (optionnel ; remplace le modèle du sous-agent ; valeurs invalides → erreur)
- `runTimeoutSeconds?` (0 par défaut ; si défini, interrompt l’exécution du sous-agent après N secondes)
- `cleanup?` (`delete|keep`, `keep` par défaut)

Liste d’autorisation :

- `agents.list[].subagents.allowAgents` : liste d’ids d’agent autorisés via `agentId` (`["*"]` pour autoriser n’importe lequel). Par défaut : uniquement l’agent demandeur.

Découverte :

- Utilisez `agents_list` pour découvrir quels ids d’agent sont autorisés pour `sessions_spawn`.

Comportement :

- Démarre une nouvelle session `agent:<agentId>:subagent:<uuid>` avec `deliver: false`.
- Les sous-agents utilisent par défaut l’ensemble complet d’outils **moins les outils de session** (configurable via `tools.subagents.tools`).
- Les sous-agents ne sont pas autorisés à appeler `sessions_spawn` (pas de sous-agent → sous-agent).
- Toujours non bloquant : renvoie immédiatement `{ status: "accepted", runId, childSessionKey }`.
- Une fois l’exécution terminée, OpenClaw exécute une **étape d’annonce** du sous-agent et publie le résultat sur le canal de discussion du demandeur.
- Répondez exactement `ANNOUNCE_SKIP` pendant l’étape d’annonce pour rester silencieux.
- Les réponses d’annonce sont normalisées en `Status`/`Result`/`Notes` ; `Status` provient du résultat d’exécution (et non du texte du modèle).
- Les sessions de sous-agent sont automatiquement archivées après `agents.defaults.subagents.archiveAfterMinutes` (60 par défaut).
- Les réponses d’annonce incluent une ligne de statistiques (durée d’exécution, jetons, sessionKey/sessionId, chemin de la transcription et coût optionnel).

<div id="sandbox-session-visibility">
  ## Visibilité des sessions en sandbox
</div>

Les sessions en sandbox peuvent utiliser les outils de session, mais, par défaut, elles ne voient que les sessions qu’elles ont créées via `sessions_spawn`.

Configuration :

```json5
{
  agents: {
    defaults: {
      sandbox: {
        // default: "spawned"
        sessionToolsVisibility: "spawned" // ou "all"
      }
    }
  }
}
```
