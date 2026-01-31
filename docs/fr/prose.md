---
title: Prose
summary: "OpenProse : workflows .prose, commandes slash et état dans OpenClaw"
read_when:
  - Vous souhaitez exécuter ou écrire des workflows .prose
  - Vous souhaitez activer le plugin OpenProse
  - Vous avez besoin de comprendre le stockage de l’état
---

<div id="openprose">
  # OpenProse
</div>

OpenProse est un format de workflow portable, centré sur le markdown, pour orchestrer des sessions d’IA. Dans OpenClaw, il est fourni sous forme de plugin qui installe un pack de compétences OpenProse ainsi qu’une commande slash `/prose`. Les programmes sont définis dans des fichiers `.prose` et peuvent lancer plusieurs sous-agents avec un contrôle de flux explicite.

Site officiel : https://www.prose.md

<div id="what-it-can-do">
  ## Ce qu’il peut faire
</div>

* Recherche et synthèse multi‑agents avec parallélisme explicite.
* Workflows reproductibles et sûrs du point de vue des validations/approbations (revue de code, triage d’incident, pipelines de contenu).
* Programmes `.prose` réutilisables que vous pouvez exécuter sur les runtimes d’agents pris en charge.

<div id="install-enable">
  ## Installation + activation
</div>

Les plugins intégrés sont désactivés par défaut. Activez OpenProse :

```bash
openclaw plugins enable open-prose
```

Redémarrez Gateway après avoir activé le plugin.

Checkout dev/local : `openclaw plugins install ./extensions/open-prose`

Documentation associée : [Plugins](/fr/plugin), [Manifeste de plugin](/fr/plugins/manifest), [Compétences](/fr/tools/skills).

<div id="slash-command">
  ## Commande slash
</div>

OpenProse enregistre `/prose` comme commande de skill invocable par l’utilisateur. Elle redirige vers les instructions de la VM OpenProse et utilise en interne les outils OpenClaw.

Commandes courantes :

```
/prose help
/prose run <file.prose>
/prose run <handle/slug>
/prose run <https://example.com/file.prose>
/prose compile <file.prose>
/prose examples
/prose update
```

<div id="example-a-simple-prose-file">
  ## Exemple : fichier `.prose` simple
</div>

```prose
# Research + synthesis with two agents running in parallel.

input topic: "What should we research?"

agent researcher:
  model: sonnet
  prompt: "You research thoroughly and cite sources."

agent writer:
  model: opus
  prompt: "You write a concise summary."

parallel:
  findings = session: researcher
    prompt: "Research {topic}."
  draft = session: writer
    prompt: "Summarize {topic}."

session "Merge the findings + draft into a final answer."
context: { findings, draft }
```

<div id="file-locations">
  ## Emplacements des fichiers
</div>

OpenProse stocke son état dans le sous-répertoire `.prose/` de votre espace de travail :

```
.prose/
├── .env
├── runs/
│   └── {YYYYMMDD}-{HHMMSS}-{random}/
│       ├── program.prose
│       ├── state.md
│       ├── bindings/
│       └── agents/
└── agents/
```

Les agents persistants au niveau utilisateur sont stockés dans :

```
~/.prose/agents/
```

<div id="state-modes">
  ## Modes d&#39;état
</div>

OpenProse prend en charge plusieurs backends de stockage d&#39;état :

* **filesystem** (par défaut) : `.prose/runs/...`
* **in-context** : transitoire, pour les petits programmes
* **sqlite** (expérimental) : nécessite le binaire `sqlite3`
* **postgres** (expérimental) : nécessite `psql` et une chaîne de connexion

Remarques :

* sqlite/postgres sont optionnels et expérimentaux.
* Les identifiants postgres sont propagés dans les journaux des sous-agents ; utilisez une base de données dédiée avec les privilèges les plus faibles possibles.

<div id="remote-programs">
  ## Programmes distants
</div>

`/prose run <handle/slug>` correspond à `https://p.prose.md/<handle>/<slug>`.
Les URL directes sont appelées telles quelles. Cette opération utilise l’outil `web_fetch` (ou `exec` pour POST).

<div id="openclaw-runtime-mapping">
  ## Correspondance avec l’environnement d’exécution OpenClaw
</div>

Les programmes OpenProse se correspondent avec les primitives OpenClaw :

| Concept OpenProse | Outil OpenClaw |
| --- | --- |
| Création de session / outil de tâche | `sessions_spawn` |
| Lecture/écriture de fichier | `read` / `write` |
| Récupération Web | `web_fetch` |

Si votre liste d’autorisation d’outils bloque ces outils, les programmes OpenProse échoueront. Voir la [configuration des compétences](/fr/tools/skills-config).

<div id="security-approvals">
  ## Sécurité + validations
</div>

Traitez les fichiers `.prose` comme du code. Relisez-les avant de les exécuter. Utilisez les listes d’autorisation d’outils OpenClaw et les étapes de validation pour contrôler les effets de bord.

Pour des workflows déterministes avec validations obligatoires, reportez-vous à [Lobster](/fr/tools/lobster).