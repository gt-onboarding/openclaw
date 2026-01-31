---
title: Sous-agents
summary: "Sous-agents : lancer des exécutions d'agents isolées qui renvoient leurs résultats au chat demandeur"
read_when:
  - Vous voulez du travail en tâche de fond ou en parallèle via l'agent
  - Vous modifiez la configuration sessions_spawn ou la politique des outils des sous-agents
---

<div id="sub-agents">
  # Sous-agents
</div>

Les sous-agents sont des exécutions d’agent en arrière-plan déclenchées à partir d’une exécution d’agent existante. Ils s’exécutent dans leur propre session (`agent:<agentId>:subagent:<uuid>`) et, une fois terminés, **annoncent** leur résultat au canal de discussion ayant émis la requête.

<div id="slash-command">
  ## Commande slash
</div>

Utilisez `/subagents` pour inspecter ou contrôler les exécutions des sous-agents pour la **session actuelle** :

- `/subagents list`
- `/subagents stop <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` affiche les métadonnées d’exécution (statut, horodatages, id de session, chemin du journal de conversation, nettoyage).

Objectifs principaux :

- Paralléliser le travail de « recherche / tâche longue / outil lent » sans bloquer l’exécution principale.
- Garder les sous-agents isolés par défaut (séparation de session + sandbox optionnelle).
- Garder la surface de l’outil difficile à mal utiliser : les sous-agents **n’obtiennent pas** les outils de session par défaut.
- Éviter le fan-out imbriqué : les sous-agents ne peuvent pas lancer d’autres sous-agents.

Remarque sur le coût : chaque sous-agent a **son propre** contexte et sa propre consommation de jetons. Pour des tâches lourdes ou répétitives, configurez un modèle moins coûteux pour les sous-agents et conservez un modèle de meilleure qualité pour votre agent principal.
Vous pouvez configurer cela via `agents.defaults.subagents.model` ou via des remplacements spécifiques à chaque agent.

<div id="tool">
  ## Outil
</div>

Utilisez `sessions_spawn` :

- Démarre une exécution de sous-agent (`deliver: false`, voie globale : `subagent`)
- Puis exécute une étape d’annonce et publie la réponse d’annonce sur le canal de discussion du demandeur
- Modèle par défaut : hérite de l’appelant, sauf si vous définissez `agents.defaults.subagents.model` (ou, par agent, `agents.list[].subagents.model`) ; un `sessions_spawn.model` explicite reste prioritaire.

Paramètres de l’outil :

- `task` (obligatoire)
- `label?` (facultatif)
- `agentId?` (facultatif ; lance le sous-agent sous un autre identifiant d’agent si autorisé)
- `model?` (facultatif ; remplace le modèle du sous-agent ; les valeurs non valides sont ignorées et le sous-agent s’exécute avec le modèle par défaut, avec un avertissement dans le résultat de l’outil)
- `thinking?` (facultatif ; remplace le niveau de réflexion pour l’exécution du sous-agent)
- `runTimeoutSeconds?` (par défaut `0` ; lorsqu’il est défini, l’exécution du sous-agent est interrompue après N secondes)
- `cleanup?` (`delete|keep`, par défaut `keep`)

Liste d’autorisation :

- `agents.list[].subagents.allowAgents` : liste des identifiants d’agent qui peuvent être ciblés via `agentId` (`["*"]` pour tout autoriser). Par défaut : uniquement l’agent demandeur.

Découverte :

- Utilisez `agents_list` pour voir quels identifiants d’agent sont actuellement autorisés pour `sessions_spawn`.

Archivage automatique :

- Les sessions de sous-agent sont automatiquement archivées après `agents.defaults.subagents.archiveAfterMinutes` (par défaut : 60).
- L’archivage utilise `sessions.delete` et renomme la transcription en `*.deleted.<timestamp>` (même dossier).
- `cleanup: "delete"` archive immédiatement après l’annonce (conserve tout de même la transcription via renommage).
- L’archivage automatique est en mode best-effort ; les minuteries en attente sont perdues si le Gateway est redémarré.
- `runTimeoutSeconds` n’**archive pas** automatiquement ; il se contente d’arrêter l’exécution. La session reste jusqu’à l’archivage automatique.

<div id="authentication">
  ## Authentification
</div>

L’authentification des sous-agents est résolue par **ID d’agent**, et non par type de session :

- La clé de session du sous-agent est `agent:<agentId>:subagent:<uuid>`.
- Le magasin d’authentification est chargé depuis le `agentDir` de cet agent.
- Les profils d’authentification de l’agent principal sont fusionnés en tant que **solution de repli** ; les profils d’agent remplacent les profils principaux en cas de conflit.

Remarque : la fusion est additive, donc les profils principaux restent toujours disponibles comme solution de repli. Une authentification totalement isolée par agent n’est pas encore prise en charge.

<div id="announce">
  ## Annonce
</div>

Les sous-agents rendent compte via une étape d'annonce :

- L'étape d'annonce s'exécute dans la session du sous-agent (et non dans la session du demandeur).
- Si le sous-agent répond exactement `ANNOUNCE_SKIP`, rien n'est publié.
- Sinon, la réponse d'annonce est publiée sur le canal de conversation du demandeur via un appel `agent` de suivi (`deliver=true`).
- Les réponses d'annonce conservent le routage par fil/sujet lorsque c'est disponible (fils Slack, sujets Telegram, fils Matrix).
- Les messages d'annonce sont normalisés selon un gabarit stable :
  - `Status:` dérivé du résultat de l'exécution (`success`, `error`, `timeout` ou `unknown`).
  - `Result:` le contenu récapitulatif provenant de l'étape d'annonce (ou `(not available)` s'il est absent).
  - `Notes:` les détails d'erreur et autres éléments de contexte utiles.
- `Status` n'est pas déduit de la sortie du modèle ; il provient des signaux de résultat au runtime.

Les payloads d'annonce incluent une ligne de statistiques à la fin (même lorsqu'ils sont encapsulés) :

- Temps d'exécution (par exemple, `runtime 5m12s`)
- Utilisation de jetons (entrée/sortie/total)
- Coût estimé lorsque la tarification du modèle est configurée (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` et chemin de transcription (afin que l'agent principal puisse récupérer l'historique via `sessions_history` ou inspecter le fichier sur le disque)

<div id="tool-policy-sub-agent-tools">
  ## Politique d’outils (outils des sous-agents)
</div>

Par défaut, les sous-agents disposent de **tous les outils sauf les outils de session** :

* `sessions_list`
* `sessions_history`
* `sessions_send`
* `sessions_spawn`

Surchargez ce comportement via la configuration :

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1
      }
    }
  },
  tools: {
    subagents: {
      tools: {
        // deny a la priorité
        deny: ["gateway", "cron"],
        // si allow est défini, cela devient allow-only (deny a toujours la priorité)
        // allow: ["read", "exec", "process"]
      }
    }
  }
}
```


<div id="concurrency">
  ## Concurrence
</div>

Les sous-agents utilisent une file d’attente dédiée au sein du processus :

- Nom de la file : `subagent`
- Niveau de concurrence : `agents.defaults.subagents.maxConcurrent` (valeur par défaut : `8`)

<div id="stopping">
  ## Arrêt
</div>

- Envoyer la commande `/stop` dans le chat du demandeur interrompt la session correspondante et arrête toutes les exécutions de sous-agents en cours qui en dépendent.

<div id="limitations">
  ## Limitations
</div>

- L’annonce du sous-agent est effectuée en mode **best-effort**. Si le Gateway redémarre, tout travail d’« announce back » en attente est perdu.
- Les sous-agents partagent toujours les mêmes ressources de processus du Gateway ; considérez `maxConcurrent` comme une soupape de sécurité.
- `sessions_spawn` est toujours non bloquant : il renvoie immédiatement `{ status: "accepted", runId, childSessionKey }`.
- Le contexte du sous-agent n’injecte que `AGENTS.md` + `TOOLS.md` (pas de `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` ou `BOOTSTRAP.md`).