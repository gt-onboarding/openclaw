---
title: Cron vs signal de vie
summary: "Guide pour choisir entre le signal de vie et les tâches cron pour l'automatisation"
read_when:
  - Décider comment planifier l'exécution de tâches récurrentes
  - Configurer la surveillance ou les notifications en arrière-plan
  - Optimiser l'utilisation des tokens pour les vérifications périodiques
---

<div id="cron-vs-heartbeat-when-to-use-each">
  # Cron vs signal de vie : quand utiliser l’un ou l’autre
</div>

Les signaux de vie et les tâches cron vous permettent tous deux d’exécuter des tâches selon un horaire défini. Ce guide vous aide à choisir le bon mécanisme pour votre cas d’usage.

<div id="quick-decision-guide">
  ## Guide de décision rapide
</div>

| Cas d&#39;utilisation | Recommandé | Pourquoi |
|-------------------|------------|----------|
| Vérifier la boîte de réception toutes les 30 min | Heartbeat | Regroupé avec d&#39;autres vérifications, sensible au contexte |
| Envoyer le rapport quotidien à 9 h précises | Cron (isolated) | Nécessite une exécution à heure exacte |
| Surveiller le calendrier pour les événements à venir | Heartbeat | Idéal pour une surveillance périodique |
| Lancer une analyse hebdomadaire approfondie | Cron (isolated) | Tâche autonome, peut utiliser un autre modèle |
| Me rappeler dans 20 minutes | Cron (main, `--at`) | Exécution unique avec horaire précis |
| Vérification de l&#39;état de santé du projet en tâche de fond | Heartbeat | Se greffe sur le cycle existant |

<div id="heartbeat-periodic-awareness">
  ## Signal de vie : vigilance périodique
</div>

Les signaux de vie s&#39;exécutent dans la **session principale** à intervalles réguliers (par défaut : 30 min). Ils sont conçus pour que l&#39;agent surveille ce qui se passe et fasse remonter tout élément important.

<div id="when-to-use-heartbeat">
  ### Quand utiliser le signal de vie
</div>

* **Vérifications périodiques multiples** : au lieu de 5 tâches cron distinctes qui vérifient la boîte de réception, le calendrier, la météo, les notifications et l&#39;état des projets, un seul signal de vie peut regrouper toutes ces opérations.
* **Décisions tenant compte du contexte** : l&#39;agent dispose du contexte complet de la session principale, il peut donc prendre des décisions pertinentes sur ce qui est urgent par rapport à ce qui peut attendre.
* **Continuité de la conversation** : les exécutions du signal de vie partagent la même session, de sorte que l&#39;agent se souvient des conversations récentes et peut assurer un suivi naturel.
* **Surveillance à faible surcharge** : un signal de vie remplace de nombreuses petites tâches d&#39;interrogation.

<div id="heartbeat-advantages">
  ### Avantages du signal de vie
</div>

* **Regroupe plusieurs vérifications** : un tour d’agent peut passer en revue la boîte de réception, le calendrier et les notifications en une seule fois.
* **Réduit les appels API** : un seul signal de vie coûte moins cher que 5 tâches cron isolées.
* **Sensible au contexte** : l’agent sait sur quoi vous avez travaillé et peut établir les priorités en conséquence.
* **Suppression intelligente** : si rien ne nécessite d’attention, l’agent répond `HEARTBEAT_OK` et aucun message n’est envoyé.
* **Rythme naturel** : dérive légèrement en fonction de la charge de la file d’attente, ce qui convient à la plupart des besoins de supervision.

<div id="heartbeat-example-heartbeatmd-checklist">
  ### Exemple de signal de vie : checklist pour HEARTBEAT.md
</div>

```md
# Heartbeat checklist

- Check email for urgent messages
- Review calendar for events in next 2 hours
- If a background task finished, summarize results
- If idle for 8+ hours, send a brief check-in
```

L’agent lit ceci à chaque signal de vie et traite tous les éléments en un seul passage.

<div id="configuring-heartbeat">
  ### Configuration du signal de vie
</div>

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",        // interval
        target: "last",      // where to deliver alerts
        activeHours: { start: "08:00", end: "22:00" }  // facultatif
      }
    }
  }
}
```

Consultez [Heartbeat](/fr/gateway/heartbeat) pour la configuration complète.

<div id="cron-precise-scheduling">
  ## Cron : planification précise
</div>

Les tâches cron s’exécutent à des **moments précis** et peuvent s’exécuter dans des sessions isolées, sans affecter le contexte principal.

<div id="when-to-use-cron">
  ### Quand utiliser cron
</div>

* **Planification précise requise** : « Envoie ceci à 9 h 00 tous les lundis » (pas « à peu près vers 9 h »).
* **Tâches autonomes** : Tâches qui n&#39;ont pas besoin de contexte conversationnel.
* **Modèle/raisonnement différent** : Analyse lourde qui justifie l&#39;utilisation d&#39;un modèle plus puissant.
* **Rappels ponctuels** : « Rappelle-moi dans 20 minutes » avec `--at`.
* **Tâches bruyantes/fréquentes** : Tâches qui encombreraient l&#39;historique de la session principale.
* **Déclencheurs externes** : Tâches qui doivent s&#39;exécuter indépendamment du fait que l&#39;agent soit par ailleurs actif.

<div id="cron-advantages">
  ### Avantages de Cron
</div>

* **Horodatage précis** : expressions cron à 5 champs avec prise en charge des fuseaux horaires.
* **Isolation de session** : s’exécute dans `cron:<jobId>` sans polluer l’historique principal.
* **Surcharges de modèle** : utilise un modèle moins coûteux ou plus puissant par tâche.
* **Contrôle de la livraison** : peut envoyer directement vers un canal ; publie tout de même un récapitulatif sur le canal principal par défaut (paramétrable).
* **Aucun contexte d’agent requis** : s’exécute même si la session principale est inactive ou compactée.
* **Prise en charge des exécutions ponctuelles** : `--at` pour des horodatages futurs précis.

<div id="cron-example-daily-morning-briefing">
  ### Exemple de tâche cron : briefing matinal quotidien
</div>

```bash
openclaw cron add \
  --name "Morning briefing" \
  --cron "0 7 * * *" \
  --tz "America/New_York" \
  --session isolated \
  --message "Generate today's briefing: weather, calendar, top emails, news summary." \
  --model opus \
  --deliver \
  --channel whatsapp \
  --to "+15551234567"
```

Cette tâche s&#39;exécute exactement à 7h00, heure de New York, utilise Opus pour la qualité et envoie directement sur WhatsApp.

<div id="cron-example-one-shot-reminder">
  ### Exemple de cron : rappel ponctuel
</div>

```bash
openclaw cron add \
  --name "Meeting reminder" \
  --at "20m" \
  --session main \
  --system-event "Reminder: standup meeting starts in 10 minutes." \
  --wake now \
  --delete-after-run
```

Voir la page [Cron jobs](/fr/automation/cron-jobs) pour la référence complète de la CLI.

<div id="decision-flowchart">
  ## Arbre de décision
</div>

```
Does the task need to run at an EXACT time?
  YES -> Use cron
  NO  -> Continue...

Does the task need isolation from main session?
  YES -> Use cron (isolated)
  NO  -> Continue...

Can this task be batched with other periodic checks?
  YES -> Use heartbeat (add to HEARTBEAT.md)
  NO  -> Use cron

Is this a one-shot reminder?
  YES -> Use cron with --at
  NO  -> Continue...

Does it need a different model or thinking level?
  YES -> Use cron (isolated) with --model/--thinking
  NO  -> Use heartbeat
```

<div id="combining-both">
  ## Combiner les deux
</div>

La configuration la plus efficace utilise **les deux** :

1. Le **signal de vie** gère la surveillance courante (boîte de réception, calendrier, notifications) en un seul traitement groupé toutes les 30 minutes.
2. **Cron** gère les planifications précises (rapports quotidiens, revues hebdomadaires) et les rappels ponctuels.

<div id="example-efficient-automation-setup">
  ### Exemple : configuration d&#39;automatisation efficace
</div>

**HEARTBEAT.md** (vérifié toutes les 30 minutes) :

```md
# Heartbeat checklist
- Scan inbox for urgent emails
- Check calendar for events in next 2h
- Review any pending tasks
- Light check-in if quiet for 8+ hours
```

**Tâches cron** (planification précise) :

```bash
# Daily morning briefing at 7am
openclaw cron add --name "Morning brief" --cron "0 7 * * *" --session isolated --message "..." --deliver

# Revue de projet hebdomadaire le lundi à 9h
openclaw cron add --name "Weekly review" --cron "0 9 * * 1" --session isolated --message "..." --model opus

# One-shot reminder
openclaw cron add --name "Call back" --at "2h" --session main --system-event "Call back the client" --wake now
```

<div id="lobster-deterministic-workflows-with-approvals">
  ## Lobster : workflows déterministes avec approbations
</div>

Lobster est le moteur d’exécution de workflows pour les **pipelines d’outils multi‑étapes** qui nécessitent une exécution déterministe et des approbations explicites.
Utilisez‑le lorsque la tâche dépasse un seul tour d’agent et que vous avez besoin d’un workflow reprenable avec des points de contrôle humains.

<div id="when-lobster-fits">
  ### Quand utiliser Lobster
</div>

* **Automatisation en plusieurs étapes** : Vous avez besoin d’un pipeline fixe d’appels d’outils, pas d’un simple prompt ponctuel.
* **Étapes d’approbation** : Les effets de bord doivent se mettre en pause jusqu’à votre approbation, puis reprendre.
* **Exécutions reprenables** : Reprendre un workflow en pause sans réexécuter les étapes précédentes.

<div id="how-it-pairs-with-heartbeat-and-cron">
  ### Comment il s’intègre au signal de vie et à cron
</div>

* **Signal de vie/cron** déterminent *quand* une exécution a lieu.
* **Lobster** définit *quelles étapes* se produisent une fois que l’exécution démarre.

Pour les workflows planifiés, utilisez cron ou le signal de vie pour déclencher un tour d’agent qui appelle Lobster.
Pour les workflows ad hoc, appelez Lobster directement.

<div id="operational-notes-from-the-code">
  ### Remarques opérationnelles (d’après le code)
</div>

* Lobster s’exécute en tant que **sous-processus local** (CLI `lobster`) en mode outil et renvoie une **enveloppe JSON**.
* Si l’outil renvoie `needs_approval`, vous reprenez avec un `resumeToken` et l’indicateur `approve`.
* L’outil est un **plugin facultatif** ; activez-le en l’ajoutant via `tools.alsoAllow: ["lobster"]` (recommandé).
* Si vous passez `lobsterPath`, il doit s’agir d’un **chemin absolu**.

Voir [Lobster](/fr/tools/lobster) pour l’utilisation complète et des exemples.

<div id="main-session-vs-isolated-session">
  ## Session principale vs session isolée
</div>

Les mécanismes Heartbeat et cron peuvent tous deux interagir avec la session principale, mais de manière différente :

| | Heartbeat | Cron (session principale) | Cron (session isolée) |
|---|---|---|---|
| Session | Principale | Principale (via événement système) | `cron:<jobId>` |
| Historique | Partagé | Partagé | Réinitialisé à chaque exécution |
| Contexte | Complet | Complet | Aucun (démarre à vide) |
| Modèle | Modèle de la session principale | Modèle de la session principale | Peut être remplacé |
| Sortie | Transmise si différente de `HEARTBEAT_OK` | Invite Heartbeat + événement | Résumé publié dans la session principale |

<div id="when-to-use-main-session-cron">
  ### Quand utiliser le cron de la session principale
</div>

Utilisez `--session main` avec `--system-event` lorsque vous voulez :

* que le rappel/l’événement apparaisse dans le contexte de la session principale
* que l’agent le traite lors du prochain signal de vie avec tout le contexte
* éviter toute exécution isolée séparée

```bash
openclaw cron add \
  --name "Check project" \
  --every "4h" \
  --session main \
  --system-event "Time for a project health check" \
  --wake now
```

<div id="when-to-use-isolated-cron">
  ### Quand utiliser un cron isolé
</div>

Utilisez `--session isolated` lorsque vous voulez :

* Un point de départ vierge, sans contexte précédent
* Un modèle ou des paramètres de raisonnement différents
* Des résultats envoyés directement à un canal (un récapitulatif est toujours publié sur le canal principal par défaut)
* Un historique qui ne vient pas encombrer la session principale

```bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 0" \
  --session isolated \
  --message "Weekly codebase analysis..." \
  --model opus \
  --thinking high \
  --deliver
```

<div id="cost-considerations">
  ## Considérations de coût
</div>

| Mécanisme | Profil de coût |
|-----------|----------------|
| Signal de vie | Un échange toutes les N minutes ; évolue avec la taille de HEARTBEAT.md |
| Cron (principal) | Ajoute un événement au prochain signal de vie (aucun échange isolé) |
| Cron (isolé) | Interaction complète de l’agent par tâche ; peut utiliser un modèle moins coûteux |

**Conseils** :

* Gardez `HEARTBEAT.md` petit pour minimiser la surcharge en jetons.
* Regroupez des vérifications similaires dans le signal de vie plutôt que d’utiliser plusieurs tâches cron.
* Utilisez `target: "none"` sur le signal de vie si vous voulez uniquement un traitement interne.
* Utilisez un cron isolé avec un modèle moins coûteux pour les tâches de routine.

<div id="related">
  ## Contenu associé
</div>

* [Heartbeat](/fr/gateway/heartbeat) - configuration complète de Heartbeat (signal de vie)
* [Cron jobs](/fr/automation/cron-jobs) - référence complète de la CLI et de l’API cron
* [System](/fr/cli/system) - événements système + contrôles de Heartbeat (signal de vie)