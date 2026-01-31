---
title: Configuration stricte
summary: "Validation stricte de la configuration + migrations exécutées uniquement via doctor"
read_when:
  - Concevoir ou implémenter le comportement de validation de la configuration
  - Travailler sur des migrations de configuration ou des workflows doctor
  - Gérer des schémas de configuration de plugins ou le contrôle du chargement des plugins
---

<div id="strict-config-validation-doctor-only-migrations">
  # Validation stricte de la configuration (migrations exécutées uniquement via doctor)
</div>

<div id="goals">
  ## Objectifs
</div>

- **Refuser toutes les clés de configuration inconnues** (racine et niveaux imbriqués).
- **Refuser toute configuration de plugin dépourvue de schéma** ; ne pas charger ce plugin.
- **Supprimer la logique de migration automatique héritée au chargement** ; n’exécuter les migrations que via doctor.
- **Lancer automatiquement doctor (dry-run) au démarrage** ; si la configuration est invalide, bloquer toutes les commandes sauf celles de diagnostic.

<div id="non-goals">
  ## Hors périmètre
</div>

- Rétrocompatibilité au chargement (les anciennes clés ne sont pas migrées automatiquement).
- Rejets silencieux des clés non reconnues.

<div id="strict-validation-rules">
  ## Règles de validation strictes
</div>

- La configuration doit correspondre exactement au schéma, et ce à chaque niveau.
- Les clés inconnues sont des erreurs de validation (aucune clé supplémentaire autorisée, ni à la racine ni dans les niveaux imbriqués).
- `plugins.entries.<id>.config` doit être validé par le schéma du plugin.
  - Si un plugin n’a pas de schéma, le chargement du plugin doit être **refusé** et une erreur explicite doit être renvoyée.
- Les clés `channels.<id>` inconnues sont des erreurs, sauf si un manifeste de plugin déclare l’identifiant du canal.
- Les manifestes de plugin (`openclaw.plugin.json`) sont requis pour tous les plugins.

<div id="plugin-schema-enforcement">
  ## Application stricte du schéma des plugins
</div>

- Chaque plugin fournit un schéma JSON strict pour sa configuration (intégré dans le manifeste).
- Processus de chargement du plugin :
  1) Résoudre le manifeste du plugin et son schéma (`openclaw.plugin.json`).
  2) Valider la configuration par rapport à ce schéma.
  3) En cas de schéma manquant ou de configuration invalide : bloquer le chargement du plugin, enregistrer l’erreur.
- Le message d’erreur inclut :
  - L’identifiant du plugin
  - La raison (schéma manquant / configuration invalide)
  - Le ou les chemins ayant échoué à la validation
- Les plugins désactivés conservent leur configuration, mais Doctor et les journaux signalent un avertissement.

<div id="doctor-flow">
  ## Flux Doctor
</div>

- Doctor s’exécute **à chaque fois** que la configuration est chargée (simulation `dry-run` par défaut).
- Si la configuration est invalide :
  - Affiche un résumé + des erreurs accompagnées d’actions à effectuer.
  - Indique : `openclaw doctor --fix`.
- `openclaw doctor --fix` :
  - Applique les migrations.
  - Supprime les clés inconnues.
  - Écrit la configuration mise à jour.

<div id="command-gating-when-config-is-invalid">
  ## Restriction des commandes (lorsque la configuration est invalide)
</div>

Autorisées (diagnostic uniquement) :

- `openclaw doctor`
- `openclaw logs`
- `openclaw health`
- `openclaw help`
- `openclaw status`
- `openclaw gateway status`

Toute autre commande doit échouer brutalement avec : « Config invalid. Run `openclaw doctor --fix`. »

<div id="error-ux-format">
  ## Format UX des erreurs
</div>

- En-tête de synthèse unique.
- Sections groupées :
  - Clés inconnues (chemins complets)
  - Clés obsolètes / migrations nécessaires
  - Échecs de chargement de plugin (identifiant du plugin + raison + chemin)

<div id="implementation-touchpoints">
  ## Points d’implémentation
</div>

- `src/config/zod-schema.ts` : supprimer le passthrough racine ; objets stricts partout.
- `src/config/zod-schema.providers.ts` : garantir des schémas de canaux stricts.
- `src/config/validation.ts` : lever une erreur en cas de clés inconnues ; ne pas appliquer les anciennes migrations.
- `src/config/io.ts` : supprimer les auto-migrations legacy ; toujours exécuter `doctor` en dry-run.
- `src/config/legacy*.ts` : limiter l’utilisation à `doctor` uniquement.
- `src/plugins/*` : ajouter un registre de schémas + mécanisme de gating.
- Gating des commandes CLI dans `src/cli`.

<div id="tests">
  ## Tests
</div>

- Rejet des clés inconnues (racine + imbriquées).
- Schéma de plugin manquant → chargement du plugin bloqué avec un message d’erreur explicite.
- Configuration invalide → démarrage du Gateway bloqué, sauf pour les commandes de diagnostic.
- Exécution à blanc automatique de `doctor` ; `doctor --fix` écrit la configuration corrigée.