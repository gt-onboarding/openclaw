---
title: Session
summary: "Règles de gestion des sessions, clés et persistance pour les conversations"
read_when:
  - Modifier la gestion ou le stockage des sessions
---

<div id="session-management">
  # Gestion des sessions
</div>

OpenClaw considère **une session de discussion directe par agent** comme principale. Les discussions directes s’agrègent dans `agent:<agentId>:<mainKey>` (par défaut `main`), tandis que les discussions de groupe/de canal ont leurs propres clés. `session.mainKey` est pris en compte.

Utilisez `session.dmScope` pour contrôler la façon dont les **messages directs** sont regroupés :

* `main` (par défaut) : tous les messages directs partagent la session principale pour la continuité.
* `per-peer` : isolation par identifiant d’expéditeur, tous canaux confondus.
* `per-channel-peer` : isolation par canal + expéditeur (recommandé pour les boîtes de réception multi-utilisateurs).
* `per-account-channel-peer` : isolation par compte + canal + expéditeur (recommandé pour les boîtes de réception multi-comptes).
  Utilisez `session.identityLinks` pour faire correspondre les identifiants d’interlocuteurs préfixés par le fournisseur à une identité canonique, afin que la même personne partage une session de messages directs à travers les canaux lorsque vous utilisez `per-peer`, `per-channel-peer` ou `per-account-channel-peer`.

<div id="gateway-is-the-source-of-truth">
  ## Gateway est la source de vérité
</div>

Tout l’état de session est **détenu par le Gateway** (le Gateway OpenClaw principal). Les clients UI (app macOS, WebChat, etc.) doivent interroger le Gateway pour obtenir les listes de sessions et les comptages de jetons plutôt que de lire des fichiers locaux.

* En **remote mode**, le stockage des sessions qui vous intéresse se trouve sur l’hôte Gateway distant, pas sur votre Mac.
* Les comptages de jetons affichés dans les UI proviennent des champs de stockage du Gateway (`inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`). Les clients n’analysent pas les transcriptions JSONL pour « corriger » les totaux.

<div id="where-state-lives">
  ## Où réside l’état
</div>

* Sur l’**hôte du Gateway** :
  * Fichier de stockage : `~/.openclaw/agents/<agentId>/sessions/sessions.json` (par agent).
* Transcriptions : `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl` (les sessions de topic Telegram utilisent `.../<SessionId>-topic-<threadId>.jsonl`).
* Le store est une map `sessionKey -> { sessionId, updatedAt, ... }`. La suppression d’entrées est sans risque ; elles sont recréées à la demande.
* Les entrées de groupe peuvent inclure `displayName`, `channel`, `subject`, `room` et `space` pour étiqueter les sessions dans les UI.
* Les entrées de session incluent des métadonnées `origin` (libellé + indications de routage) afin que les UI puissent expliquer d’où provient une session.
* OpenClaw ne **lit pas** les anciens dossiers de session Pi/Tau.

<div id="session-pruning">
  ## Purge des sessions
</div>

OpenClaw supprime par défaut les **anciens résultats d’outils** du contexte en mémoire juste avant les appels au LLM.
Cela **ne** réécrit **pas** l’historique JSONL. Voir [/concepts/session-pruning](/fr/concepts/session-pruning).

<div id="pre-compaction-memory-flush">
  ## Vidage de la mémoire avant compactage
</div>

Lorsqu&#39;une session se rapproche de l’auto-compactage, OpenClaw peut exécuter un **vidage silencieux de la mémoire**
qui rappelle au modèle d’écrire des notes durables sur le disque. Cette opération ne s’exécute que si
l’espace de travail est accessible en écriture. Voir [Mémoire](/fr/concepts/memory) et
[Compactage](/fr/concepts/compaction).

<div id="mapping-transports-session-keys">
  ## Mappage des transports → clés de session
</div>

* Les conversations directes suivent `session.dmScope` (valeur par défaut `main`).
  * `main` : `agent:<agentId>:<mainKey>` (continuité entre appareils/canaux).
    * Plusieurs numéros de téléphone et canaux peuvent être associés à la même clé principale d&#39;agent ; ils servent de transports vers une seule conversation.
  * `per-peer` : `agent:<agentId>:dm:<peerId>`.
  * `per-channel-peer` : `agent:<agentId>:<channel>:dm:<peerId>`.
  * `per-account-channel-peer` : `agent:<agentId>:<channel>:<accountId>:dm:<peerId>` (`accountId` vaut `default` par défaut).
  * Si `session.identityLinks` correspond à un identifiant de participant préfixé par un fournisseur (par exemple `telegram:123`), la clé canonique remplace `<peerId>` afin que la même personne partage une session sur plusieurs canaux.
* Les conversations de groupe disposent d’un état isolé : `agent:<agentId>:<channel>:group:<id>` (les salons/canaux utilisent `agent:<agentId>:<channel>:channel:<id>`).
  * Les sujets de forum Telegram ajoutent `:topic:<threadId>` à l&#39;identifiant de groupe pour l&#39;isolation.
  * Les anciennes clés `group:<id>` sont toujours reconnues pour la migration.
* Les contextes entrants peuvent encore utiliser `group:<id>` ; le canal est déduit à partir de `Provider` et normalisé sous la forme canonique `agent:<agentId>:<channel>:group:<id>`.
* Autres sources :
  * Tâches cron : `cron:<job.id>`
  * Webhooks : `hook:<uuid>` (sauf si défini explicitement par le hook)
  * Exécutions de nœud : `node-<nodeId>`

<div id="lifecycle">
  ## Cycle de vie
</div>

* Politique de réinitialisation : les sessions sont réutilisées jusqu’à leur expiration, et l’expiration est évaluée au prochain message entrant.
* Réinitialisation quotidienne : par défaut à **4:00 du matin, heure locale de l’hôte du Gateway**. Une session est considérée comme obsolète lorsque sa dernière mise à jour est antérieure à l’heure de la réinitialisation quotidienne la plus récente.
* Réinitialisation d’inactivité (optionnelle) : `idleMinutes` ajoute une fenêtre d’inactivité glissante. Lorsque les réinitialisations quotidienne et d’inactivité sont toutes deux configurées, **la première qui expire** force une nouvelle session.
* Ancien mode « inactivité seule » : si vous définissez `session.idleMinutes` sans aucune configuration `session.reset`/`resetByType`, OpenClaw reste en mode uniquement basé sur l’inactivité pour des raisons de compatibilité ascendante.
* Surcharges par type (optionnelles) : `resetByType` vous permet de surcharger la politique pour les sessions `dm`, `group` et `thread` (thread = fils Slack/Discord, sujets Telegram, fils Matrix lorsque fournis par le connecteur).
* Surcharges par canal (optionnelles) : `resetByChannel` surcharge la politique de réinitialisation pour un canal (s’applique à tous les types de session pour ce canal et a priorité sur `reset`/`resetByType`).
* Déclencheurs de réinitialisation : les commandes exactes `/new` ou `/reset` (plus toute commande supplémentaire définie dans `resetTriggers`) démarrent un nouvel identifiant de session et transmettent le reste du message. `/new <model>` accepte un alias de modèle, `provider/model` ou un nom de fournisseur (correspondance approximative) pour définir le modèle de la nouvelle session. Si `/new` ou `/reset` est envoyé seul, OpenClaw exécute un court échange de salutation (« hello ») pour confirmer la réinitialisation.
* Réinitialisation manuelle : supprimez des clés spécifiques du store ou retirez la transcription JSONL ; le message suivant les recrée.
* Les tâches cron isolées génèrent toujours un nouveau `sessionId` à chaque exécution (aucune réutilisation basée sur l’inactivité).

<div id="send-policy-optional">
  ## Stratégie d&#39;envoi (facultatif)
</div>

Bloquez l&#39;envoi pour certains types de session sans répertorier chaque ID individuellement.

```json5
{
  session: {
    sendPolicy: {
      rules: [
        { action: "deny", match: { channel: "discord", chatType: "group" } },
        { action: "deny", match: { keyPrefix: "cron:" } }
      ],
      default: "allow"
    }
  }
}
```

Surcharge à l&#39;exécution (propriétaire uniquement) :

* `/send on` → autoriser pour cette session
* `/send off` → refuser pour cette session
* `/send inherit` → annuler la surcharge et utiliser les règles de configuration
  Envoie-les comme messages séparés afin qu&#39;elles soient prises en compte.

<div id="configuration-optional-rename-example">
  ## Configuration (exemple facultatif de renommage)
</div>

```json5
// ~/.openclaw/openclaw.json
{
  session: {
    scope: "per-sender",      // maintenir les clés de groupe séparées
    dmScope: "main",          // continuité des DM (définir per-channel-peer/per-account-channel-peer pour les boîtes de réception partagées)
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"]
    },
    reset: {
      // Valeurs par défaut : mode=daily, atHour=4 (heure locale de l'hôte Gateway).
      // Si vous définissez également idleMinutes, le premier à expirer l'emporte.
      mode: "daily",
      atHour: 4,
      idleMinutes: 120
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      dm: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 }
    },
    resetByChannel: {
      discord: { mode: "idle", idleMinutes: 10080 }
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    mainKey: "main",
  }
}
```

<div id="inspecting">
  ## Inspection
</div>

* `openclaw status` — affiche le chemin du store et les sessions récentes.
* `openclaw sessions --json` — affiche toutes les entrées (à filtrer avec `--active <minutes>`).
* `openclaw gateway call sessions.list --params '{}'` — récupère les sessions depuis le Gateway en cours d’exécution (utilisez `--url`/`--token` pour accéder à un Gateway distant).
* Envoyez `/status` comme message autonome dans le chat pour voir si l’agent est joignable, quelle part du contexte de session est utilisée, l’état actuel des modes réflexion/verbeux, et quand vos identifiants WhatsApp Web ont été actualisés pour la dernière fois (aide à repérer les besoins de reconnexion).
* Envoyez `/context list` ou `/context detail` pour voir ce qui se trouve dans le system prompt et les fichiers d’espace de travail injectés (et les principaux contributeurs au contexte).
* Envoyez `/stop` comme message autonome pour interrompre l’exécution en cours, vider les relances en file d’attente pour cette session, et arrêter toutes les exécutions de sous-agents lancées à partir de celle-ci (la réponse inclut le nombre d’exécutions arrêtées).
* Envoyez `/compact` (instructions facultatives) comme message autonome pour résumer l’ancien contexte et libérer de l’espace dans la fenêtre. Voir [/concepts/compaction](/fr/concepts/compaction).
* Les transcriptions JSONL peuvent être ouvertes directement pour examiner l’intégralité des échanges.

<div id="tips">
  ## Conseils
</div>

* Réserve la clé principale au trafic 1:1 ; laisse les groupes utiliser leurs propres clés.
* Lors de l&#39;automatisation du nettoyage, supprime les clés individuellement plutôt que de vider entièrement le store afin de préserver le contexte ailleurs.

<div id="session-origin-metadata">
  ## Métadonnées d’origine de session
</div>

Chaque entrée de session enregistre sa provenance (dans la mesure du possible) dans `origin` :

* `label` : libellé lisible par un humain (résolu à partir du libellé de conversation + sujet/groupe/canal)
* `provider` : identifiant de canal normalisé (y compris les extensions)
* `from`/`to` : identifiants de routage bruts provenant de l’enveloppe entrante
* `accountId` : identifiant de compte fournisseur (en cas de multi‑compte)
* `threadId` : identifiant de fil/sujet lorsque le canal le prend en charge

Les champs d’origine sont renseignés pour les messages directs, les canaux et les groupes. Si un
connecteur ne met à jour que le routage de la livraison (par exemple pour garder une session principale de message direct fraîche),
il doit tout de même fournir un contexte entrant afin que la session conserve ses
métadonnées d’origine. Les extensions peuvent le faire en envoyant `ConversationLabel`,
`GroupSubject`, `GroupChannel`, `GroupSpace` et `SenderName` dans le contexte entrant
et en appelant `recordSessionMetaFromInbound` (ou en passant le même contexte
à `updateLastRoute`).