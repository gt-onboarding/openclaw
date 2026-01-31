---
title: Scripts
summary: "Scripts du dépôt : objectif, portée et consignes de sécurité"
read_when:
  - Exécuter des scripts à partir du dépôt
  - Ajouter ou modifier des scripts sous ./scripts
---

<div id="scripts">
  # Scripts
</div>

Le répertoire `scripts/` contient des scripts auxiliaires pour les workflows locaux et les tâches d’opérations.
Utilisez-les lorsqu’une tâche est clairement liée à un script ; sinon, privilégiez la CLI.

<div id="conventions">
  ## Conventions
</div>

* Les scripts sont **facultatifs** sauf s’ils sont mentionnés dans la documentation ou les checklists de publication.
* Privilégier les interfaces CLI lorsqu’elles existent (exemple : la supervision de l’authentification utilise `openclaw models status --check`).
* Considérer que les scripts sont propres à chaque hôte ; les lire avant de les exécuter sur une nouvelle machine.

<div id="git-hooks">
  ## Hooks Git
</div>

* `scripts/setup-git-hooks.js` : configuration « best effort » de `core.hooksPath` lorsque vous êtes dans un dépôt Git.
* `scripts/format-staged.js` : outil de formatage pré-commit pour les fichiers `src/` et `test/` indexés (`staged`).

<div id="auth-monitoring-scripts">
  ## Scripts de surveillance de l&#39;authentification
</div>

Ces scripts sont documentés ici :
[/automation/auth-monitoring](/fr/automation/auth-monitoring)

<div id="when-adding-scripts">
  ## Lors de l’ajout de scripts
</div>

* Gardez les scripts ciblés et bien documentés.
* Ajoutez une brève entrée dans la documentation correspondante (ou créez-la si elle n’existe pas encore).