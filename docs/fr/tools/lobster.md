---
title: Lobster
summary: "Moteur d'exécution de workflows typés pour OpenClaw avec étapes d'approbation reprenables."
description: Moteur d'exécution de workflows typés pour OpenClaw — pipelines composables avec étapes d'approbation.
read_when:
  - Vous voulez des workflows déterministes en plusieurs étapes avec des validations explicites
  - Vous devez pouvoir reprendre un workflow sans réexécuter les étapes précédentes
---

<div id="lobster">
  # Lobster
</div>

Lobster est un shell de workflow qui permet à OpenClaw d’exécuter des enchaînements d’outils en plusieurs étapes comme une seule opération déterministe, avec des points d’approbation explicites.

<div id="hook">
  ## Hook
</div>

Votre assistant peut construire les outils qui gèrent son propre fonctionnement. Demandez un workflow et, 30 minutes plus tard, vous disposez d’une CLI et de pipelines qui s’exécutent en un seul appel. Lobster est la pièce manquante : des pipelines déterministes, des validations explicites et un état reprenable.

<div id="why">
  ## Pourquoi
</div>

Aujourd&#39;hui, les workflows complexes nécessitent de nombreux appels d&#39;outils avec allers-retours. Chaque appel coûte des jetons, et le LLM doit orchestrer chaque étape. Lobster déplace cette orchestration dans un runtime typé :

* **Un seul appel au lieu de plusieurs** : OpenClaw exécute un appel d&#39;outil Lobster et obtient un résultat structuré.
* **Approbations intégrées** : Les effets de bord (envoyer un e-mail, publier un commentaire) interrompent le workflow jusqu&#39;à approbation explicite.
* **Reprise possible** : Les workflows interrompus renvoient un jeton ; approuvez et reprenez sans tout réexécuter.

<div id="why-a-dsl-instead-of-plain-programs">
  ## Pourquoi un DSL plutôt que des programmes classiques ?
</div>

Lobster est volontairement minimal. L’objectif n’est pas « un nouveau langage », mais une spécification de pipeline prévisible, adaptée à l’IA, avec des approbations et des jetons de reprise de première classe.

* **Les mécanismes d’approbation et de reprise sont intégrés** : un programme classique peut solliciter un humain, mais il ne peut pas *se mettre en pause puis reprendre* avec un jeton durable sans que vous n’inventiez vous-même ce runtime.
* **Déterminisme + auditabilité** : les pipelines sont des données, donc faciles à consigner, comparer, rejouer et examiner.
* **Surface contrainte pour l’IA** : une grammaire minuscule + du chaînage JSON réduisent les chemins de code « créatifs » et rendent la validation réaliste.
* **Politique de sécurité intégrée** : timeouts, plafonds de sortie, vérifications de sandbox et listes d’autorisation sont appliqués par le runtime, pas par chaque script.
* **Toujours programmable** : chaque étape peut appeler n’importe quel CLI ou script. Si vous voulez du JS/TS, générez des fichiers `.lobster` à partir du code.

<div id="how-it-works">
  ## Fonctionnement
</div>

OpenClaw lance la CLI locale `lobster` en **mode outil** et lit une enveloppe JSON depuis stdout.
Si le pipeline se met en pause pour approbation, l’outil renvoie un `resumeToken` afin que vous puissiez reprendre plus tard.

<div id="pattern-small-cli-json-pipes-approvals">
  ## Modèle : petit outil CLI + pipes JSON + approbations
</div>

Crée de toutes petites commandes qui parlent JSON, puis enchaîne-les dans un seul appel Lobster. (Exemples de noms de commandes ci-dessous — remplace-les par les tiens.)

```bash
inbox list --json
inbox categorize --json
inbox apply --json
```

```json
{
  "action": "run",
  "pipeline": "exec --json --shell 'inbox list --json' | exec --stdin json --shell 'inbox categorize --json' | exec --stdin json --shell 'inbox apply --json' | approve --preview-from-stdin --limit 5 --prompt 'Apply changes?'",
  "timeoutMs": 30000
}
```

Si le pipeline demande une approbation, reprenez en fournissant le jeton :

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

L’IA déclenche le workflow, Lobster exécute les étapes. Les étapes d’approbation rendent les effets de bord explicites et auditables.

Exemple : faire correspondre des éléments en entrée à des appels d’outils :

```bash
gog.gmail.search --query 'newer_than:1d' \
  | openclaw.invoke --tool message --action send --each --item-key message --args-json '{"provider":"telegram","to":"..."}'
```

<div id="json-only-llm-steps-llm-task">
  ## Étapes LLM uniquement en JSON (llm-task)
</div>

Pour les workflows qui ont besoin d’une **étape LLM structurée**, activez l’outil plugin facultatif
`llm-task` et appelez-le depuis Lobster. Cela garde le workflow
déterministe tout en vous permettant de classifier/résumer/rédiger avec un modèle.

Activez l’outil :

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  },
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```

Utilisez-le dans un pipeline :

```lobster
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Given the input email, return intent and draft.",
  "input": { "subject": "Hello", "body": "Can you help?" },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```

Voir [LLM Task](/fr/tools/llm-task) pour plus de détails et des options de configuration.

<div id="workflow-files-lobster">
  ## Fichiers de workflow (.lobster)
</div>

Lobster peut exécuter des fichiers de workflow au format YAML/JSON avec les champs `name`, `args`, `steps`, `env`, `condition` et `approval`. Dans les appels d’outils OpenClaw, définissez `pipeline` sur le chemin du fichier.

```yaml
name: inbox-triage
args:
  tag:
    default: "family"
steps:
  - id: collect
    command: inbox list --json
  - id: categorize
    command: inbox categorize --json
    stdin: $collect.stdout
  - id: approve
    command: inbox apply --approve
    stdin: $categorize.stdout
    approval: required
  - id: execute
    command: inbox apply --execute
    stdin: $categorize.stdout
    condition: $approve.approved
```

Remarques :

* `stdin: $step.stdout` et `stdin: $step.json` transmettent la sortie de l’étape précédente.
* `condition` (ou `when`) peut servir à exécuter ou non une étape en fonction de `$step.approved`.

<div id="install-lobster">
  ## Installer Lobster
</div>

Installe la CLI Lobster sur le **même hôte** que celui qui exécute le Gateway OpenClaw (voir le [dépôt Lobster](https://github.com/openclaw/lobster)) et assurez-vous que `lobster` est dans le `PATH`.
Si vous souhaitez utiliser un emplacement binaire personnalisé, fournissez un `lobsterPath` **absolu** dans l’appel de l’outil.

<div id="enable-the-tool">
  ## Activer l’outil
</div>

Lobster est un outil plugin **optionnel** (désactivé par défaut).

Recommandé (supplémentaire, sans risque) :

```json
{
  "tools": {
    "alsoAllow": ["lobster"]
  }
}
```

Ou pour chaque agent :

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": {
          "alsoAllow": ["lobster"]
        }
      }
    ]
  }
}
```

Évite d’utiliser `tools.allow: ["lobster"]` sauf si tu as l’intention de l’exécuter en mode liste d’autorisation restrictif.

Remarque : les listes d’autorisation sont facultatives et doivent être activées explicitement pour les plugins optionnels. Si ta liste d’autorisation ne nomme que des outils de plugins (comme `lobster`), OpenClaw laisse les outils cœur activés. Pour restreindre également les outils cœur, ajoute à la liste d’autorisation les outils ou groupes cœur que tu souhaites.

<div id="example-email-triage">
  ## Exemple : triage des e-mails
</div>

Sans Lobster :

```
Utilisateur : "Vérifie mes emails et rédige des réponses"
→ openclaw appelle gmail.list
→ LLM résume
→ Utilisateur : "rédige des réponses pour #2 et #5"
→ LLM rédige
→ Utilisateur : "envoie #2"
→ openclaw appelle gmail.send
(répété quotidiennement, aucune mémoire de ce qui a été trié)
```

Avec Lobster :

```json
{
  "action": "run",
  "pipeline": "email.triage --limit 20",
  "timeoutMs": 30000
}
```

Renvoie une enveloppe JSON tronquée :

```json
{
  "ok": true,
  "status": "needs_approval",
  "output": [{ "summary": "5 need replies, 2 need action" }],
  "requiresApproval": {
    "type": "approval_request",
    "prompt": "Envoyer les 2 brouillons de réponse ?",
    "items": [],
    "resumeToken": "..."
  }
}
```

Approbation de l’utilisateur → reprise :

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

Un seul workflow. Déterministe. Sécurisé.

<div id="tool-parameters">
  ## Paramètres de l’outil
</div>

<div id="run">
  ### `run`
</div>

Exécute un pipeline en mode tool.

```json
{
  "action": "run",
  "pipeline": "gog.gmail.search --query 'newer_than:1d' | email.triage",
  "cwd": "/path/to/workspace",
  "timeoutMs": 30000,
  "maxStdoutBytes": 512000
}
```

Exécuter un fichier de workflow avec des arguments :

```json
{
  "action": "run",
  "pipeline": "/path/to/inbox-triage.lobster",
  "argsJson": "{\"tag\":\"family\"}"
}
```

<div id="resume">
  ### `resume`
</div>

Reprendre l’exécution d’un workflow interrompu après approbation.

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

<div id="optional-inputs">
  ### Entrées facultatives
</div>

* `lobsterPath` : Chemin absolu du binaire Lobster (laissez vide pour utiliser `PATH`).
* `cwd` : Répertoire de travail pour le pipeline (par défaut : répertoire de travail du processus en cours).
* `timeoutMs` : Interrompt le sous-processus s’il dépasse cette durée (par défaut : 20000).
* `maxStdoutBytes` : Interrompt le sous-processus si la sortie standard dépasse cette taille (par défaut : 512000).
* `argsJson` : Chaîne JSON transmise à `lobster run --args-json` (uniquement pour les fichiers de workflow).

<div id="output-envelope">
  ## Enveloppe de sortie
</div>

Lobster renvoie une enveloppe JSON avec l’un des trois statuts suivants :

* `ok` → terminé avec succès
* `needs_approval` → en attente ; `requiresApproval.resumeToken` est requis pour reprendre
* `cancelled` → explicitement refusé ou annulé

L’outil met l’enveloppe à disposition à la fois dans `content` (JSON lisible) et `details` (objet brut).

<div id="approvals">
  ## Approbations
</div>

Si `requiresApproval` est présent, inspecte le prompt et décide :

* `approve: true` → reprendre et continuer les effets secondaires
* `approve: false` → annuler et finaliser le workflow

Utilise `approve --preview-from-stdin --limit N` pour ajouter un aperçu JSON aux demandes d’approbation sans avoir besoin de code jq/heredoc « glue » personnalisé. Les jetons de reprise sont désormais compacts : Lobster stocke l’état de reprise du workflow dans son répertoire d’état et renvoie une petite clé de jeton.

<div id="openprose">
  ## OpenProse
</div>

OpenProse se combine bien avec Lobster : utilise `/prose` pour orchestrer la préparation multi‑agents, puis exécute un pipeline Lobster pour des approbations déterministes. Si un programme Prose a besoin de Lobster, autorise l’outil `lobster` pour les sous-agents via `tools.subagents.tools`. Voir [OpenProse](/fr/prose).

<div id="safety">
  ## Sécurité
</div>

* **Sous-processus locaux uniquement** — aucun appel réseau émis par le plugin lui-même.
* **Aucun secret** — Lobster ne gère pas OAuth ; il appelle des outils OpenClaw qui le font.
* **Compatible avec le sandbox** — désactivé lorsque le contexte de l’outil est dans un sandbox.
* **Renforcé** — `lobsterPath` doit être un chemin absolu s’il est spécifié ; des délais d’expiration et des limites de sortie sont appliqués.

<div id="troubleshooting">
  ## Dépannage
</div>

* **`lobster subprocess timed out`** → augmentez `timeoutMs`, ou divisez un long pipeline.
* **`lobster output exceeded maxStdoutBytes`** → augmentez `maxStdoutBytes` ou réduisez la taille de la sortie.
* **`lobster returned invalid JSON`** → assurez-vous que le pipeline s’exécute en mode outil et n’imprime que du JSON.
* **`lobster failed (code …)`** → exécutez le même pipeline dans un terminal pour inspecter stderr.

<div id="learn-more">
  ## En savoir plus
</div>

* [Plugins](/fr/plugin)
* [Développement d&#39;outils de plugin](/fr/plugins/agent-tools)

<div id="case-study-community-workflows">
  ## Étude de cas : workflows communautaires
</div>

Un exemple public : un « second cerveau » sous forme de CLI + des pipelines Lobster qui gèrent trois coffres-forts Markdown (personnel, partenaire, partagé). La CLI émet du JSON pour les statistiques, les listes de boîte de réception et les analyses d’éléments obsolètes ; Lobster enchaîne ces commandes dans des workflows comme `weekly-review`, `inbox-triage`, `memory-consolidation` et `shared-task-sync`, chacun avec des étapes d’approbation. L’IA prend en charge l’arbitrage (catégorisation) lorsqu’elle est disponible et se rabat sur des règles déterministes dans le cas contraire.

* Thread : https://x.com/plattenschieber/status/2014508656335770033
* Repo : https://github.com/bloomedai/brain-cli