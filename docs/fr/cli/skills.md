---
title: Compétences
summary: "Référence CLI pour `openclaw skills` (list/info/check) et conditions d'éligibilité des compétences"
read_when:
  - Vous souhaitez voir quelles compétences sont disponibles et prêtes à être exécutées
  - Vous souhaitez déboguer les problèmes de binaires/environnement/configuration manquants pour les compétences
---

<div id="openclaw-skills">
  # `openclaw skills`
</div>

Examiner les compétences (intégrées + espace de travail + surcharges gérées) et identifier celles qui sont éligibles ou auxquelles il manque des prérequis.

Rubriques associées :

* Système de compétences : [Skills](/fr/tools/skills)
* Configuration des compétences : [Skills config](/fr/tools/skills-config)
* Installations depuis ClawHub : [ClawHub](/fr/tools/clawhub)

<div id="commands">
  ## Commandes
</div>

```bash
openclaw skills list
openclaw skills list --eligible
openclaw skills info <name>
openclaw skills check
```
