---
title: Manifeste
summary: "Manifeste de plugin + exigences relatives au schéma JSON (validation stricte de la configuration)"
read_when:
  - Vous développez un plugin OpenClaw
  - Vous devez fournir un schéma de configuration de plugin ou déboguer des erreurs de validation de plugin
---

<div id="plugin-manifest-openclawpluginjson">
  # Manifeste de plugin (openclaw.plugin.json)
</div>

Chaque plugin **doit** inclure un fichier `openclaw.plugin.json` à la **racine du plugin**.
OpenClaw utilise ce manifeste pour valider la configuration **sans exécuter le code du plugin**. Les manifestes manquants ou invalides sont traités comme des erreurs de plugin et bloquent la validation de la configuration.

Consulte le guide complet du système de plugins : [Plugins](/fr/plugin).

<div id="required-fields">
  ## Champs requis
</div>

```json
{
  "id": "voice-call",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {}
  }
}
```

Clés requises :

* `id` (string): identifiant canonique du plugin.
* `configSchema` (object): schéma JSON de configuration du plugin (intégré).

Clés optionnelles :

* `kind` (string): type de plugin (par exemple : `"memory"`).
* `channels` (array): identifiants de canaux enregistrés par ce plugin (par exemple : `["matrix"]`).
* `providers` (array): identifiants de fournisseurs enregistrés par ce plugin.
* `skills` (array): répertoires de compétences à charger (relatifs à la racine du plugin).
* `name` (string): nom d’affichage du plugin.
* `description` (string): résumé succinct du plugin.
* `uiHints` (object): libellés/champs de saisie/indicateurs de sensibilité pour le rendu dans l’UI.
* `version` (string): version du plugin (à titre informatif).

<div id="json-schema-requirements">
  ## Exigences relatives au schéma JSON
</div>

* **Chaque plugin doit fournir un schéma JSON**, même s’il n’accepte aucune configuration.
* Un schéma vide est acceptable (par exemple, `{ "type": "object", "additionalProperties": false }`).
* Les schémas sont validés au moment de la lecture et de l’écriture de la configuration, et non à l’exécution.

<div id="validation-behavior">
  ## Comportement de validation
</div>

* Les clés `channels.*` inconnues sont des **erreurs**, sauf si l’ID de canal est
  déclaré par un manifeste de plugin.
* `plugins.entries.<id>`, `plugins.allow`, `plugins.deny` et `plugins.slots.*`
  doivent référencer des IDs de plugin **découvrables**. Les IDs inconnus sont
  des **erreurs**.
* Si un plugin est installé mais que son manifeste ou son schéma est corrompu ou
  manquant, la validation échoue et Doctor signale l’erreur de plugin.
* Si une configuration de plugin existe mais que le plugin est **désactivé**, la
  configuration est conservée et un **avertissement** est remonté dans Doctor +
  les journaux.

<div id="notes">
  ## Remarques
</div>

* Le manifeste est **obligatoire pour tous les plugins**, y compris les chargements à partir du système de fichiers local.
* Le runtime charge toujours le module du plugin séparément ; le manifeste ne sert qu&#39;à
  la découverte + validation.
* Si votre plugin dépend de modules natifs, documentez les étapes de construction et toute
  exigence de liste d’autorisation du gestionnaire de paquets (par exemple, pnpm `allow-build-scripts`
  * `pnpm rebuild <package>`).