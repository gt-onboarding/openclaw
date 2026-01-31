---
title: Compétences
summary: "Compétences : gérées vs espace de travail, règles de filtrage et liaison config/env"
read_when:
  - Ajout ou modification de compétences
  - Modification des règles de filtrage ou de chargement des compétences
---

<div id="skills-openclaw">
  # Compétences (OpenClaw)
</div>

OpenClaw utilise des dossiers de compétences **compatibles [AgentSkills](https://agentskills.io)** pour apprendre à l’agent à utiliser des outils. Chaque compétence est un répertoire contenant un `SKILL.md` avec un en-tête YAML et des instructions. OpenClaw charge les **compétences intégrées** ainsi que d’éventuelles surcharges locales, et les filtre au chargement en fonction de l’environnement, de la configuration et de la présence de binaires.

<div id="locations-and-precedence">
  ## Emplacements et ordre de priorité
</div>

Les compétences sont chargées depuis **trois** emplacements :

1. **Compétences intégrées** : fournies avec l’installation (package npm ou OpenClaw.app)
2. **Compétences gérées/locales** : `~/.openclaw/skills`
3. **Compétences d’espace de travail** : `<workspace>/skills`

Si plusieurs compétences portent le même nom, l’ordre de priorité est le suivant :

`<workspace>/skills` (priorité la plus élevée) → `~/.openclaw/skills` → compétences intégrées (priorité la plus basse)

De plus, vous pouvez configurer des dossiers de compétences supplémentaires (priorité la plus basse) via
`skills.load.extraDirs` dans `~/.openclaw/openclaw.json`.

<div id="per-agent-vs-shared-skills">
  ## Compétences par agent vs partagées
</div>

Dans les configurations **multi-agents**, chaque agent possède son propre espace de travail. Cela signifie :

* Les **compétences par agent** résident dans `<workspace>/skills` pour cet agent uniquement.
* Les **compétences partagées** résident dans `~/.openclaw/skills` (gérées/en local) et sont visibles
  par **tous les agents** sur la même machine.
* Des **dossiers partagés** peuvent également être ajoutés via `skills.load.extraDirs` (priorité
  la plus basse) si vous voulez un pack de compétences commun utilisé par plusieurs agents.

Si le même nom de compétence existe à plusieurs endroits, la règle de priorité habituelle
s’applique : l’espace de travail l’emporte, puis les compétences gérées/locales, puis celles intégrées au binaire.

<div id="plugins-skills">
  ## Plugins + compétences
</div>

Les plugins peuvent intégrer leurs propres compétences en référençant des répertoires `skills` dans `openclaw.plugin.json` (chemins relatifs à la racine du plugin). Les compétences fournies par un plugin se chargent lorsque le plugin est activé et suivent les règles normales de priorité des compétences. Vous pouvez les conditionner via `metadata.openclaw.requires.config` sur l’entrée de configuration du plugin. Voir [Plugins](/fr/plugin) pour la découverte et la configuration, et [Tools](/fr/tools) pour la surface d’outils que ces compétences exposent.

<div id="clawhub-install-sync">
  ## ClawHub (installation + synchronisation)
</div>

ClawHub est le registre public de compétences pour OpenClaw. Accédez-y sur
https://clawhub.com. Utilisez-le pour découvrir, installer, mettre à jour et sauvegarder des compétences.
Guide complet : [ClawHub](/fr/tools/clawhub).

Parcours typiques :

* Installer une compétence dans votre espace de travail :
  * `clawhub install <skill-slug>`
* Mettre à jour toutes les compétences installées :
  * `clawhub update --all`
* Synchroniser (analyser + publier les mises à jour) :
  * `clawhub sync --all`

Par défaut, `clawhub` installe dans `./skills` sous votre répertoire de travail
actuel (ou se rabat sur l’espace de travail OpenClaw configuré). OpenClaw le prend
en compte comme `<workspace>/skills` lors de la session suivante.

<div id="security-notes">
  ## Remarques de sécurité
</div>

* Considérez les compétences tierces comme du **code digne de confiance**. Lisez-les avant de les activer.
* Privilégiez l’exécution en sandbox pour les entrées non fiables et les outils risqués. Voir [Sandboxing](/fr/gateway/sandboxing).
* `skills.entries.*.env` et `skills.entries.*.apiKey` injectent des secrets dans le processus **hôte**
  pour ce tour d’agent (et non dans le sandbox). Gardez les secrets en dehors des prompts et des journaux.
* Pour un modèle de menace plus complet et des listes de contrôle, voir [Security](/fr/gateway/security).

<div id="format-agentskills-pi-compatible">
  ## Format (AgentSkills + compatible avec Pi)
</div>

`SKILL.md` doit inclure au minimum :

```markdown
---
name: nano-banana-pro
description: Générer ou modifier des images via Gemini 3 Pro Image
---
```

Notes :

* Nous suivons la spécification AgentSkills pour la structure et l’intention.
* Le parseur utilisé par l’agent embarqué ne prend en charge que les clés de frontmatter **sur une seule ligne**.
* `metadata` doit être un **objet JSON sur une seule ligne**.
* Utilisez `{baseDir}` dans les instructions pour faire référence au chemin du dossier de la compétence.
* Clés de frontmatter optionnelles :
  * `homepage` — URL affichée comme « Website » dans la UI Compétences de macOS (également pris en charge via `metadata.openclaw.homepage`).
  * `user-invocable` — `true|false` (par défaut : `true`). Lorsque `true`, la compétence est exposée comme commande slash utilisateur.
  * `disable-model-invocation` — `true|false` (par défaut : `false`). Lorsque `true`, la compétence est exclue du prompt du modèle (toujours disponible via invocation utilisateur).
  * `command-dispatch` — `tool` (optionnel). Lorsque défini sur `tool`, la commande slash contourne le modèle et est envoyée directement à un outil.
  * `command-tool` — nom de l’outil à invoquer lorsque `command-dispatch: tool` est défini.
  * `command-arg-mode` — `raw` (par défaut). Pour l’acheminement vers un outil, transmet la chaîne d’arguments bruts à l’outil (aucune analyse par le cœur du système).

    L’outil est invoqué avec les paramètres :
    `{ command: "<raw args>", commandName: "<slash command>", skillName: "<skill name>" }`.

<div id="gating-load-time-filters">
  ## Gating (filtres au chargement)
</div>

OpenClaw **filtre les compétences au chargement** à l&#39;aide de `metadata` (JSON sur une seule ligne) :

```markdown
---
name: nano-banana-pro
description: Générer ou modifier des images via Gemini 3 Pro Image
metadata: {"openclaw":{"requires":{"bins":["uv"],"env":["GEMINI_API_KEY"],"config":["browser.enabled"]},"primaryEnv":"GEMINI_API_KEY"}}
---
```

Champs sous `metadata.openclaw` :

* `always: true` — toujours inclure la compétence (ignorer les autres conditions).
* `emoji` — emoji facultatif utilisé par l&#39;UI Skills de macOS.
* `homepage` — URL facultative affichée comme « Website » dans l&#39;UI Skills de macOS.
* `os` — liste facultative de plates-formes (`darwin`, `linux`, `win32`). Si défini, la compétence n&#39;est éligible que sur ces OS.
* `requires.bins` — liste ; chacun doit exister dans le `PATH`.
* `requires.anyBins` — liste ; au moins un doit exister dans le `PATH`.
* `requires.env` — liste ; la variable d&#39;environnement doit exister **ou** être fournie dans la configuration.
* `requires.config` — liste de chemins `openclaw.json` qui doivent s&#39;évaluer à vrai.
* `primaryEnv` — nom de variable d&#39;environnement associé à `skills.entries.<name>.apiKey`.
* `install` — tableau facultatif de spécifications d&#39;installateurs utilisé par l&#39;UI Skills de macOS (brew/node/go/uv/download).

Remarque concernant le sandbox :

* `requires.bins` est vérifié sur l&#39;**hôte** au moment du chargement de la compétence.
* Si un agent est exécuté dans un sandbox, le binaire doit aussi exister **dans le conteneur**.
  Installez-le via `agents.defaults.sandbox.docker.setupCommand` (ou une image personnalisée).
  `setupCommand` s&#39;exécute une fois après la création du conteneur.
  Les installations de paquets nécessitent également un accès sortant au réseau, un système de fichiers racine accessible en écriture et un utilisateur root dans le sandbox.
  Exemple : la compétence `summarize` (`skills/summarize/SKILL.md`) a besoin du CLI `summarize`
  dans le conteneur du sandbox pour pouvoir s&#39;y exécuter.

Exemple d&#39;installateur :

```markdown
---
name: gemini
description: Use Gemini CLI for coding assistance and Google search lookups.
metadata: {"openclaw":{"emoji":"♊️","requires":{"bins":["gemini"]},"install":[{"id":"brew","kind":"brew","formula":"gemini-cli","bins":["gemini"],"label":"Install Gemini CLI (brew)"}]}}
---
```

Remarques :

* Si plusieurs installateurs sont listés, le Gateway choisit **une seule** option préférée (brew lorsqu’il est disponible, sinon node).
* Si tous les installateurs sont `download`, OpenClaw répertorie chaque entrée afin que vous puissiez voir les artéfacts disponibles.
* Les spécifications d’installateur peuvent inclure `os: ["darwin"|"linux"|"win32"]` pour filtrer les options par plateforme.
* Les installations Node respectent `skills.install.nodeManager` dans `openclaw.json` (valeur par défaut : npm ; options : npm/pnpm/yarn/bun).
  Cela affecte uniquement les **installations de compétences** ; le runtime du Gateway doit toujours être Node
  (Bun n’est pas recommandé pour WhatsApp/Telegram).
* Installations Go : si `go` est absent et que `brew` est disponible, le Gateway installe d’abord Go via Homebrew et définit `GOBIN` sur le `bin` de Homebrew lorsque c’est possible.
* Installations par téléchargement : `url` (obligatoire), `archive` (`tar.gz` | `tar.bz2` | `zip`), `extract` (par défaut : automatique lorsqu’une archive est détectée), `stripComponents`, `targetDir` (par défaut : `~/.openclaw/tools/<skillKey>`).

Si aucun `metadata.openclaw` n’est présent, la compétence est toujours éligible (sauf si elle est désactivée dans la configuration ou bloquée par `skills.allowBundled` pour les compétences intégrées).

<div id="config-overrides-openclawopenclawjson">
  ## Surcharges de configuration (`~/.openclaw/openclaw.json`)
</div>

Les compétences intégrées/gérées peuvent être activées ou désactivées et recevoir des variables d&#39;environnement :

```json5
{
  skills: {
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE"
        },
        config: {
          endpoint: "https://example.invalid",
          model: "nano-pro"
        }
      },
      peekaboo: { enabled: true },
      sag: { enabled: false }
    }
  }
}
```

Remarque : si le nom de la compétence contient des tirets, mettez la clé entre guillemets (JSON5 autorise les clés entre guillemets).

Les clés de configuration correspondent par défaut au **nom de la compétence**. Si une compétence définit
`metadata.openclaw.skillKey`, utilisez cette clé sous `skills.entries`.

Règles :

* `enabled: false` désactive la compétence même si elle est empaquetée/installée.
* `env` : sera injectée **uniquement si** la variable n’est pas déjà définie dans le processus.
* `apiKey` : raccourci pour les compétences qui déclarent `metadata.openclaw.primaryEnv`.
* `config` : conteneur facultatif pour les champs personnalisés propres à chaque compétence ; les clés personnalisées doivent se trouver ici.
* `allowBundled` : liste d’autorisation facultative pour les compétences **empaquetées** uniquement. Si ce champ est défini, seules
  les compétences empaquetées figurant dans la liste sont éligibles (compétences gérées/de l’espace de travail non affectées).

<div id="environment-injection-per-agent-run">
  ## Injection d&#39;environnement (par exécution d&#39;agent)
</div>

Lorsqu&#39;une exécution d&#39;agent démarre, OpenClaw :

1. Lit les métadonnées des compétences.
2. Applique tout `skills.entries.<key>.env` ou `skills.entries.<key>.apiKey` à
   `process.env`.
3. Construit le prompt système avec les compétences **éligibles**.
4. Restaure l&#39;environnement d&#39;origine une fois l&#39;exécution terminée.

Cette opération est **limitée à l&#39;exécution de l&#39;agent**, et non à un environnement de shell global.

<div id="session-snapshot-performance">
  ## Instantané de session (performance)
</div>

OpenClaw prend un instantané des compétences éligibles **au démarrage d&#39;une session** et réutilise cette liste pour les tours suivants dans la même session. Les modifications des compétences ou de la configuration prennent effet à la prochaine session.

Les compétences peuvent également être actualisées en cours de session lorsque le watcher de compétences est activé ou lorsqu&#39;un nouveau nœud distant éligible apparaît (voir ci-dessous). Considérez cela comme un **rechargement à chaud** : la liste actualisée est prise en compte au prochain tour de l&#39;agent.

<div id="remote-macos-nodes-linux-gateway">
  ## Nœuds macOS distants (Gateway Linux)
</div>

Si le Gateway s’exécute sous Linux mais qu’un **nœud macOS** est connecté **avec `system.run` autorisé** (les paramètres de sécurité d’approbation d’exécution ne sont pas définis sur `deny`), OpenClaw peut considérer les compétences spécifiques à macOS comme utilisables lorsque les binaires requis sont présents sur ce nœud. L’agent doit exécuter ces compétences via l’outil `nodes` (généralement `nodes.run`).

Cela repose sur le fait que le nœud signale la prise en charge de ses commandes et sur une détection des binaires via `system.run`. Si le nœud macOS se déconnecte ultérieurement, les compétences restent visibles ; les appels peuvent échouer jusqu’à ce que le nœud se reconnecte.

<div id="skills-watcher-auto-refresh">
  ## Surveillance des compétences (rafraîchissement automatique)
</div>

Par défaut, OpenClaw surveille les dossiers de compétences et met à jour l’instantané des compétences lorsque les fichiers `SKILL.md` changent. Vous pouvez configurer ce comportement sous `skills.load` :

```json5
{
  skills: {
    load: {
      watch: true,
      watchDebounceMs: 250
    }
  }
}
```

<div id="token-impact-skills-list">
  ## Impact sur les jetons (liste des compétences)
</div>

Lorsque des compétences sont éligibles, OpenClaw injecte une liste XML compacte des compétences disponibles dans le prompt système (via `formatSkillsForPrompt` dans `pi-coding-agent`). Le coût en jetons est déterministe :

* **Surcharge de base (uniquement lorsqu’il y a ≥1 compétence)** : 195 caractères.
* **Par compétence :** 97 caractères + la longueur des valeurs de `<name>`, `<description>` et `<location>` après échappement XML.

Formule (caractères) :

```
total = 195 + Σ (97 + len(name_escaped) + len(description_escaped) + len(location_escaped))
```

Remarques :

* L’échappement XML transforme `& < > " '` en entités (`&amp;`, `&lt;`, etc.), ce qui augmente la longueur.
* Le nombre de tokens varie selon le tokenizer du modèle. Une estimation approximative de type OpenAI est d’environ 4 caractères par token, donc **97 caractères ≈ 24 tokens** par skill, en plus des longueurs réelles de vos champs.

<div id="managed-skills-lifecycle">
  ## Cycle de vie des compétences gérées
</div>

OpenClaw est livré avec un ensemble de compétences de base en tant que **compétences intégrées** dans le cadre de
l&#39;installation (package npm ou OpenClaw.app). `~/.openclaw/skills` existe pour les
remplacements locaux (par exemple, figer/corriger une compétence sans modifier la copie intégrée). Les compétences d&#39;espace de travail appartiennent à l&#39;utilisateur et ont priorité sur les deux en cas de conflit de nom.

<div id="config-reference">
  ## Référence de configuration
</div>

Consultez [la configuration des compétences](/fr/tools/skills-config) pour le schéma de configuration complet.

<div id="looking-for-more-skills">
  ## Vous cherchez plus de compétences ?
</div>

Parcourez https://clawhub.com.

***