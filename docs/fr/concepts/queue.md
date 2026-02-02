---
title: File d'attente de commandes
summary: "Conception de la file d'attente de commandes qui sérialise les exécutions de réponses automatiques entrantes"
read_when:
  - Modification de l'exécution ou du niveau de concurrence des réponses automatiques
---

<div id="command-queue-2026-01-16">
  # File d'attente des commandes (2026-01-16)
</div>

Nous sérialisons les exécutions de réponses automatiques entrantes (tous canaux confondus) via une petite file d'attente en mémoire, dans le processus, afin d’empêcher plusieurs exécutions d’agent d’entrer en collision, tout en permettant un parallélisme sécurisé entre les sessions.

<div id="why">
  ## Pourquoi
</div>

- Les exécutions de réponses automatiques peuvent être coûteuses (appels LLM) et entrer en collision lorsque plusieurs messages entrants arrivent presque en même temps.
- La sérialisation évite la concurrence pour les ressources partagées (fichiers de session, journaux, stdin de la CLI) et réduit le risque de limites de débit imposées en amont.

<div id="how-it-works">
  ## Fonctionnement
</div>

- Une file d’attente FIFO « lane-aware » vide chaque voie (`lane`) avec un plafond de parallélisme configurable (par défaut 1 pour les voies non configurées ; `main` par défaut à 4, `subagent` à 8).
- `runEmbeddedPiAgent` met en file d’attente par **clé de session** (voie `session:<key>`) afin de garantir une seule exécution active par session.
- Chaque exécution de session est ensuite ajoutée à une **voie globale** (`main` par défaut), de sorte que le parallélisme global soit plafonné par `agents.defaults.maxConcurrent`.
- Quand la journalisation détaillée est activée, les exécutions en file d’attente émettent un court message si elles ont attendu plus d’environ 2 s avant de démarrer.
- Les indicateurs de saisie sont toujours déclenchés immédiatement lors de la mise en file d’attente (lorsque le canal les prend en charge), de sorte que l’expérience utilisateur reste inchangée pendant l’attente de son tour.

<div id="queue-modes-per-channel">
  ## Modes de file d&#39;attente (par canal)
</div>

Les messages entrants peuvent orienter l&#39;exécution en cours, attendre un tour de suivi ou faire les deux :

* `steer` : injecte immédiatement dans l&#39;exécution en cours (annule les appels d&#39;outils en attente après la prochaine frontière d&#39;outil). Si le streaming est désactivé, revient à `followup`.
* `followup` : met en file d&#39;attente pour le prochain tour de l&#39;agent après la fin de l&#39;exécution en cours.
* `collect` : fusionne tous les messages en file d&#39;attente en **un seul** tour de suivi (valeur par défaut). Si les messages ciblent différents canaux/fils, ils sont vidés individuellement pour préserver le routage.
* `steer-backlog` (alias `steer+backlog`) : oriente maintenant **et** préserve le message pour un tour de suivi.
* `interrupt` (legacy) : interrompt l&#39;exécution active pour cette session, puis exécute le message le plus récent.
* `queue` (alias legacy) : identique à `steer`.

`steer-backlog` signifie que vous pouvez obtenir une réponse de suivi après l&#39;exécution dirigée, de sorte que
les surfaces de streaming peuvent sembler dupliquées. Préférez `collect`/`steer` si vous voulez
une seule réponse par message entrant.
Envoyez `/queue collect` comme commande autonome (par session) ou définissez `messages.queue.byChannel.discord: "collect"`.

Valeurs par défaut (lorsqu&#39;aucune valeur n&#39;est définie dans la configuration) :

* Toutes les surfaces → `collect`

Configurez globalement ou par canal via `messages.queue` :

```json5
{
  messages: {
    queue: {
      mode: "collect",
      debounceMs: 1000,
      cap: 20,
      drop: "summarize",
      byChannel: { discord: "collect" }
    }
  }
}
```


<div id="queue-options">
  ## Options de file d'attente
</div>

Les options s'appliquent à `followup`, `collect` et `steer-backlog` (et à `steer` lorsqu'il bascule en mode `followup`) :

- `debounceMs` : attendre un moment de silence avant de démarrer un tour de `followup` (évite « continue, continue »).
- `cap` : nombre maximal de messages en file d'attente par session.
- `drop` : politique en cas de dépassement (`old`, `new`, `summarize`).

`summarize` conserve une courte liste à puces des messages abandonnés et l'injecte comme invite de `followup` synthétique.
Valeurs par défaut : `debounceMs: 1000`, `cap: 20`, `drop: summarize`.

<div id="per-session-overrides">
  ## Surcharges par session
</div>

- Envoyez `/queue <mode>` en tant que commande autonome pour enregistrer le mode pour la session en cours.
- Les options peuvent être combinées : `/queue collect debounce:2s cap:25 drop:summarize`
- `/queue default` ou `/queue reset` supprime la surcharge de session.

<div id="scope-and-guarantees">
  ## Portée et garanties
</div>

- S’applique aux exécutions de l’agent de réponse automatique sur tous les canaux entrants qui utilisent le pipeline de réponse du Gateway (WhatsApp web, Telegram, Slack, Discord, Signal, iMessage, webchat, etc.).
- La voie par défaut (`main`) est globale au processus pour les traitements entrants et les signaux de vie principaux ; définissez `agents.defaults.maxConcurrent` pour autoriser plusieurs sessions en parallèle.
- Des voies supplémentaires peuvent exister (par ex. `cron`, `subagent`) afin que les tâches en arrière-plan puissent s’exécuter en parallèle sans bloquer les réponses entrantes.
- Les voies par session garantissent qu’une seule exécution d’agent modifie une session donnée à la fois.
- Aucune dépendance externe ni thread worker en arrière-plan ; TypeScript pur + promises.

<div id="troubleshooting">
  ## Dépannage
</div>

- Si des commandes semblent bloquées, activez les journaux détaillés et recherchez les lignes « queued for …ms » pour confirmer que la file d’attente se vide.
- Si vous avez besoin de la profondeur de la file d’attente, activez les journaux détaillés et surveillez les lignes indiquant le minutage de la file d’attente.