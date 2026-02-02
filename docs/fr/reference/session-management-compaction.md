---
title: Gestion et compactage des sessions
summary: "Analyse approfondie : magasin de sessions + transcriptions, cycle de vie et fonctionnement interne du compactage (automatique)"
read_when:
  - Vous devez déboguer des ID de session, le JSONL des transcriptions ou les champs de sessions.json
  - Vous modifiez le comportement de compactage automatique ou ajoutez des tâches de maintenance de « pré-compactage »
  - Vous voulez implémenter des vidages de mémoire ou des tours système silencieux
---

<div id="session-management-compaction-deep-dive">
  # Gestion des sessions et compactage (approfondi)
</div>

Ce document explique comment OpenClaw gère les sessions de bout en bout :

* **Routage des sessions** (comment les messages entrants sont associés à une `sessionKey`)
* **Stockage des sessions** (`sessions.json`) et ce qu’il suit et conserve
* **Persistance des transcriptions** (`*.jsonl`) et sa structure
* **Hygiène des transcriptions** (corrections spécifiques au fournisseur avant les exécutions)
* **Limites de contexte** (fenêtre de contexte vs jetons suivis)
* **Compactage** (compactage manuel + automatique) et où brancher le travail pré‑compactage
* **Maintenance silencieuse** (par ex. écritures en mémoire qui ne doivent pas produire de sortie visible pour l’utilisateur)

Si vous voulez d’abord une vue d’ensemble plus globale, commencez par :

* [/concepts/session](/fr/concepts/session)
* [/concepts/compaction](/fr/concepts/compaction)
* [/concepts/session-pruning](/fr/concepts/session-pruning)
* [/reference/transcript-hygiene](/fr/reference/transcript-hygiene)

***

<div id="source-of-truth-the-gateway">
  ## Source de vérité : le Gateway
</div>

OpenClaw est conçu autour d’un unique **processus Gateway** qui détient l’état des sessions.

* Les UI (application macOS, Control UI web, TUI) doivent interroger le Gateway pour obtenir les listes de sessions et les nombres de jetons.
* En mode distant, les fichiers de session se trouvent sur l’hôte distant ; « vérifier les fichiers locaux de votre Mac » ne reflètera pas ce que le Gateway utilise réellement.

***

<div id="two-persistence-layers">
  ## Deux couches de persistance
</div>

OpenClaw stocke les sessions dans deux couches :

1. **Session store (`sessions.json`)**
   * Mappage clé/valeur : `sessionKey -> SessionEntry`
   * Petit, mutable, sans risque à modifier (ou à supprimer des entrées)
   * Suit les métadonnées de session (identifiant de session actuel, dernière activité, options, compteurs de jetons, etc.)

2. **Transcript (`<sessionId>.jsonl`)**
   * Journal en ajout uniquement avec structure en arbre (les entrées ont `id` + `parentId`)
   * Stocke la conversation réelle + les appels d’outils + les résumés de compaction
   * Sert à reconstruire le contexte du modèle pour les échanges futurs

***

<div id="on-disk-locations">
  ## Emplacements sur disque
</div>

Par agent, sur l’hôte du Gateway :

* Stockage : `~/.openclaw/agents/<agentId>/sessions/sessions.json`
* Transcriptions : `~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl`
  * Sessions par sujet Telegram : `.../<sessionId>-topic-<threadId>.jsonl`

OpenClaw détermine ces chemins via `src/config/sessions.ts`.

***

<div id="session-keys-sessionkey">
  ## Clés de session (`sessionKey`)
</div>

Un `sessionKey` identifie *dans quel espace de conversation* vous vous trouvez (routage + isolation).

Schémas courants :

* Discussion principale/directe (par agent) : `agent:<agentId>:<mainKey>` (par défaut `main`)
* Groupe : `agent:<agentId>:<channel>:group:<id>`
* Salon/canal (Discord/Slack) : `agent:<agentId>:<channel>:channel:<id>` ou `...:room:<id>`
* Cron : `cron:<job.id>`
* Webhook : `hook:<uuid>` (sauf si surchargé)

Les règles canoniques sont documentées dans [/concepts/session](/fr/concepts/session).

***

<div id="session-ids-sessionid">
  ## Identifiants de session (`sessionId`)
</div>

Chaque `sessionKey` pointe vers un `sessionId` courant (le fichier de transcription qui prolonge la conversation).

Règles générales :

* **Réinitialisation** (`/new`, `/reset`) crée un nouveau `sessionId` pour ce `sessionKey`.
* **Réinitialisation quotidienne** (par défaut à 4:00 du matin, heure locale de la machine hébergeant le Gateway) crée un nouveau `sessionId` au prochain message après la limite de réinitialisation.
* **Expiration d&#39;inactivité** (`session.reset.idleMinutes` ou ancien `session.idleMinutes`) crée un nouveau `sessionId` lorsqu&#39;un message arrive après la fenêtre d&#39;inactivité. Quand la réinitialisation quotidienne et l&#39;expiration d&#39;inactivité sont toutes deux configurées, la première à expirer l&#39;emporte.

Détail d’implémentation : la décision se fait dans `initSessionState()` dans `src/auto-reply/reply/session.ts`.

***

<div id="session-store-schema-sessionsjson">
  ## Schéma du magasin de sessions (`sessions.json`)
</div>

Le type de valeur du magasin est `SessionEntry` dans `src/config/sessions.ts`.

Champs clés (liste non exhaustive) :

* `sessionId` : identifiant de la transcription en cours (le nom de fichier est dérivé de celui-ci sauf si `sessionFile` est défini)
* `updatedAt` : horodatage de la dernière activité
* `sessionFile` : chemin de transcription explicite optionnel (surcharge)
* `chatType` : `direct | group | room` (aide les UI et la politique d’envoi)
* `provider`, `subject`, `room`, `space`, `displayName` : métadonnées pour l’étiquetage des groupes/canaux
* Commutateurs :
  * `thinkingLevel`, `verboseLevel`, `reasoningLevel`, `elevatedLevel`
  * `sendPolicy` (surcharge au niveau de la session)
* Sélection du modèle :
  * `providerOverride`, `modelOverride`, `authProfileOverride`
* Compteurs de tokens (au mieux / dépendants du fournisseur) :
  * `inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`
* `compactionCount` : nombre de fois où la compaction automatique s’est terminée pour cette clé de session
* `memoryFlushAt` : horodatage du dernier vidage de mémoire avant compaction
* `memoryFlushCompactionCount` : compteur de compaction au moment du dernier vidage

Le magasin peut être modifié sans risque, mais le Gateway fait autorité : il peut réécrire ou réhydrater des entrées au fur et à mesure de l’exécution des sessions.

***

<div id="transcript-structure-jsonl">
  ## Structure du transcript (`*.jsonl`)
</div>

Les transcripts sont gérés par le `SessionManager` de `@mariozechner/pi-coding-agent`.

Le fichier est au format JSONL :

* Première ligne : en-tête de session (`type: "session"`, inclut `id`, `cwd`, `timestamp`, éventuellement `parentSession`)
* Ensuite : entrées de session avec `id` + `parentId` (arborescence)

Types d’entrées importants :

* `message` : messages user/assistant/toolResult
* `custom_message` : messages injectés par des extensions qui *entrent* dans le contexte du modèle (peuvent être masqués dans l’UI)
* `custom` : état d’extension qui *n’entre pas* dans le contexte du modèle
* `compaction` : résumé de compaction persistant avec `firstKeptEntryId` et `tokensBefore`
* `branch_summary` : résumé persistant lors de la navigation dans une branche de l’arbre

OpenClaw ne **corrige** volontairement pas les transcripts ; le Gateway utilise `SessionManager` pour les lire/écrire.

***

<div id="context-windows-vs-tracked-tokens">
  ## Fenêtres de contexte vs jetons suivis
</div>

Deux concepts différents sont importants :

1. **Fenêtre de contexte du modèle** : limite maximale stricte par modèle (jetons visibles par le modèle)
2. **Compteurs du store de sessions** : statistiques glissantes enregistrées dans `sessions.json` (utilisées pour /status et les tableaux de bord)

Si vous ajustez les limites :

* La fenêtre de contexte provient du catalogue de modèles (et peut être surchargée via la configuration).
* `contextTokens` dans le store est une valeur d’estimation/de reporting à l’exécution ; ne la considérez pas comme une garantie stricte.

Pour plus de détails, voir [/token-use](/fr/token-use).

***

<div id="compaction-what-it-is">
  ## Compaction : en quoi ça consiste
</div>

La compaction résume les parties plus anciennes de la conversation dans une entrée `compaction` persistée dans le journal de conversation et conserve les messages récents intacts.

Après la compaction, les échanges suivants voient :

* Le résumé de compaction
* Les messages après `firstKeptEntryId`

La compaction est **persistante** (contrairement à l&#39;élagage de session). Voir [/concepts/session-pruning](/fr/concepts/session-pruning).

***

<div id="when-auto-compaction-happens-pi-runtime">
  ## Quand la compaction automatique a lieu (runtime Pi)
</div>

Dans l’agent Pi embarqué, la compaction automatique se déclenche dans deux cas :

1. **Récupération en cas de dépassement** : le modèle renvoie une erreur de dépassement de contexte → compacter → réessayer.
2. **Maintien du seuil** : après un échange réussi, lorsque :

`contextTokens > contextWindow - reserveTokens`

Où :

* `contextWindow` est la fenêtre de contexte du modèle
* `reserveTokens` est la marge réservée pour les prompts + la prochaine sortie du modèle

Ce sont la sémantique du runtime Pi (OpenClaw consomme les événements, mais Pi décide quand compacter).

***

<div id="compaction-settings-reservetokens-keeprecenttokens">
  ## Paramètres de compactage (`reserveTokens`, `keepRecentTokens`)
</div>

Les paramètres de compactage de Pi se configurent dans les paramètres de Pi :

```json5
{
  compaction: {
    enabled: true,
    reserveTokens: 16384,
    keepRecentTokens: 20000
  }
}
```

OpenClaw applique également un seuil minimal de sécurité pour les exécutions embarquées :

* Si `compaction.reserveTokens < reserveTokensFloor`, OpenClaw l’augmente.
* Le seuil minimal par défaut est de `20000` jetons.
* Définis `agents.defaults.compaction.reserveTokensFloor: 0` pour désactiver ce seuil.
* S’il est déjà plus élevé, OpenClaw le laisse tel quel.

Pourquoi : conserver suffisamment de marge pour la « gestion courante » à plusieurs tours (comme les écritures en mémoire) avant que la compaction ne devienne inévitable.

Implémentation : `ensurePiCompactionReserveTokens()` dans `src/agents/pi-settings.ts`
(appelé depuis `src/agents/pi-embedded-runner.ts`).

***

<div id="user-visible-surfaces">
  ## Surfaces visibles pour l’utilisateur
</div>

Vous pouvez observer la compaction et l’état de la session via :

* `/status` (dans n’importe quelle session de chat)
* `openclaw status` (CLI)
* `openclaw sessions` / `sessions --json`
* Mode détaillé : `🧹 Auto-compaction complete` + nombre de compactages

***

<div id="silent-housekeeping-no_reply">
  ## Maintenance silencieuse (`NO_REPLY`)
</div>

OpenClaw prend en charge des tours « silencieux » pour les tâches en arrière-plan, lorsque l’utilisateur ne doit pas voir les sorties intermédiaires.

Convention :

* L’assistant commence sa sortie par `NO_REPLY` pour indiquer « ne pas envoyer de réponse à l’utilisateur ».
* OpenClaw retire/supprime ceci dans la couche de distribution.

Depuis la version `2026.1.10`, OpenClaw supprime également le **streaming du brouillon/de la saisie** lorsqu’un fragment partiel commence par `NO_REPLY`, afin que les opérations silencieuses ne laissent pas fuiter de sortie partielle en cours de tour.

***

<div id="pre-compaction-memory-flush-implemented">
  ## Vidage de « mémoire » pré-compaction (implémenté)
</div>

Objectif : avant que la compaction automatique n’ait lieu, exécuter un tour d’agent silencieux qui écrit un état
persistant sur le disque (par ex. `memory/YYYY-MM-DD.md` dans l’espace de travail de l’agent) afin que la compaction ne puisse pas
effacer de contexte critique.

OpenClaw utilise l’approche de **vidage avant seuil** :

1. Surveiller l’utilisation du contexte de session.
2. Lorsqu’elle dépasse un « seuil souple » (en dessous du seuil de compaction de Pi), exécuter une directive silencieuse
   « write memory now » vers l’agent.
3. Utiliser `NO_REPLY` pour que l’utilisateur ne voie rien.

Config (`agents.defaults.compaction.memoryFlush`) :

* `enabled` (valeur par défaut : `true`)
* `softThresholdTokens` (valeur par défaut : `4000`)
* `prompt` (message utilisateur pour le tour de vidage)
* `systemPrompt` (prompt système supplémentaire ajouté pour le tour de vidage)

Notes :

* Le prompt et le prompt système par défaut incluent un indice `NO_REPLY` pour empêcher l’envoi de toute réponse à l’utilisateur.
* Le vidage s’exécute une fois par cycle de compaction (suivi dans `sessions.json`).
* Le vidage ne s’exécute que pour les sessions Pi intégrées (les backends CLI le sautent).
* Le vidage est ignoré lorsque l’espace de travail de la session est en lecture seule (`workspaceAccess: "ro"` ou `"none"`).
* Voir [Memory](/fr/concepts/memory) pour la structure des fichiers de l’espace de travail et les schémas d’écriture.

Pi expose également un hook `session_before_compact` dans l’API d’extension, mais la logique de vidage d’OpenClaw
se trouve aujourd’hui côté Gateway.

<div id="troubleshooting-checklist">
  ## Liste de vérification pour le dépannage
</div>

* Clé de session incorrecte ? Commencez par [/concepts/session](/fr/concepts/session) et confirmez la `sessionKey` dans `/status`.
* Décalage entre store et transcript ? Confirmez l’hôte du Gateway et le chemin du store à partir de `openclaw status`.
* Compactage trop fréquent ? Vérifiez :
  * la fenêtre de contexte du modèle (trop petite)
  * les paramètres de compactage (`reserveTokens` trop élevé par rapport à la fenêtre du modèle peut provoquer un compactage plus précoce)
  * le gonflement des résultats d’outils : activez/ajustez l’élagage des sessions
* Fuites d’échanges silencieux ? Confirmez que la réponse commence par `NO_REPLY` (jeton exact) et que vous utilisez un build incluant la correction de suppression du streaming.