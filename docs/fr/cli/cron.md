---
title: Cron
summary: "Référence CLI pour `openclaw cron` (planifier et exécuter des tâches en arrière-plan)"
read_when:
  - Vous souhaitez configurer des tâches planifiées et leurs déclenchements
  - Vous déboguez l'exécution et les journaux de cron
---

<div id="openclaw-cron">
  # `openclaw cron`
</div>

Gérer les tâches cron pour le planificateur du Gateway.

Liens associés :

* Tâches cron : [Tâches cron](/fr/automation/cron-jobs)

Astuce : exécutez `openclaw cron --help` pour afficher l’ensemble des options disponibles de la commande.

<div id="common-edits">
  ## Modifications courantes
</div>

Mettre à jour les paramètres d’envoi sans modifier le message :

```bash
openclaw cron edit <job-id> --deliver --channel telegram --to "123456789"
```

Désactiver l’envoi pour une tâche isolée :

```bash
openclaw cron edit <job-id> --no-deliver
```
