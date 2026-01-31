---
title: Durcissement de cron.add
summary: "Renforcer le traitement des entrées de cron.add, aligner les schémas et améliorer l'UI cron et l'outillage des agents"
owner: "openclaw"
status: "complete"
last_updated: "2026-01-05"
---

<div id="cron-add-hardening-schema-alignment">
  # Durcissement de `cron add` et alignement du schéma
</div>

<div id="context">
  ## Contexte
</div>

Les journaux récents du Gateway montrent des échecs répétés de `cron.add` avec des paramètres non valides (`sessionTarget`, `wakeMode`, `payload` manquants et `schedule` mal formé). Cela indique qu’au moins un client (probablement le chemin d’appel d’outil de l’agent) envoie des payloads de tâches encapsulés ou partiellement spécifiés. Par ailleurs, il existe un décalage entre les enums du fournisseur Cron dans TypeScript, le schéma du Gateway, les options de la CLI et les types de formulaires de l’UI, ainsi qu’une incohérence dans l’UI pour `cron.status` (attend `jobCount` alors que le Gateway renvoie `jobs`).

<div id="goals">
  ## Objectifs
</div>

* Arrêter le spam `cron.add` INVALID&#95;REQUEST en normalisant les charges utiles des wrappers courants et en déduisant les champs `kind` manquants.
* Aligner les listes de fournisseurs cron entre le schéma du Gateway, les types cron, la documentation CLI et les formulaires de l’UI.
* Rendre le schéma de l’outil cron de l’agent explicite afin que le LLM produise des charges utiles de tâches correctes.
* Corriger l’affichage du nombre de tâches dans le statut cron de la Control UI.
* Ajouter des tests pour couvrir la normalisation et le comportement de l’outil.

<div id="non-goals">
  ## Hors périmètre
</div>

* Modifier la sémantique de planification cron ou le comportement d’exécution des jobs.
* Ajouter de nouveaux types de planification ou de nouveaux modes d’analyse des expressions cron.
* Refondre l’UI/UX de cron au-delà des corrections de champs strictement nécessaires.

<div id="findings-current-gaps">
  ## Constats (lacunes actuelles)
</div>

* `CronPayloadSchema` dans le Gateway exclut `signal` + `imessage`, tandis que les types TS les incluent.
* Le Control UI CronStatus attend `jobCount`, mais le Gateway renvoie `jobs`.
* Le schéma de l’outil cron de l’agent autorise des objets `job` arbitraires, ce qui permet des entrées mal formées.
* Le Gateway valide strictement `cron.add` sans normalisation, donc les payloads encapsulés échouent.

<div id="what-changed">
  ## Ce qui a changé
</div>

* `cron.add` et `cron.update` normalisent désormais les structures d’enveloppe les plus courantes et déduisent les champs `kind` manquants.
* Le schéma de l’outil cron des agents correspond au schéma du Gateway, ce qui réduit les charges utiles invalides.
* Les énumérations de fournisseurs sont alignées entre le Gateway, la CLI, l’UI et le sélecteur macOS.
* Le Control UI utilise le champ compteur `jobs` du Gateway pour le statut.

<div id="current-behavior">
  ## Comportement actuel
</div>

* **Normalisation :** les charges utiles `data`/`job` encapsulées sont désempaquetées ; `schedule.kind` et `payload.kind` sont déduits lorsque c’est sans risque.
* **Valeurs par défaut :** des valeurs sûres sont appliquées pour `wakeMode` et `sessionTarget` lorsqu’elles sont absentes.
* **Fournisseurs :** Discord/Slack/Signal/iMessage sont désormais exposés de manière cohérente dans la CLI et l’UI.

Consultez [Cron jobs](/fr/automation/cron-jobs) pour la structure normalisée et des exemples.

<div id="verification">
  ## Vérification
</div>

* Surveillez les logs du Gateway pour vérifier la diminution des erreurs `cron.add` INVALID&#95;REQUEST.
* Confirmez que le statut de cron dans le Control UI affiche le nombre de tâches après rafraîchissement.

<div id="optional-follow-ups">
  ## Suivis facultatifs
</div>

* Contrôle manuel (smoke) dans le Control UI : ajouter un job cron par fournisseur et vérifier le nombre de jobs de statut.

<div id="open-questions">
  ## Questions ouvertes
</div>

* `cron.add` doit-il accepter un `state` explicite de la part des clients (actuellement interdit par le schéma) ?
* Doit-on autoriser `webchat` comme fournisseur explicite de livraison (actuellement filtré lors de la résolution de la livraison) ?