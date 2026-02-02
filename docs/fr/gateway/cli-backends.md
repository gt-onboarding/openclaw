---
title: Backends CLI
summary: "Backends CLI : solution de secours en mode texte uniquement via des CLI d’IA locales"
read_when:
  - Vous voulez une solution de secours fiable lorsque des fournisseurs d’API sont en panne
  - Vous faites tourner Claude Code CLI ou d'autres CLI d’IA locales et souhaitez les réutiliser
  - Vous avez besoin d'un mode purement texte, sans outils, qui prenne malgré tout en charge les sessions et les images
---

<div id="cli-backends-fallback-runtime">
  # Backends CLI (exécution de secours)
</div>

OpenClaw peut exécuter des **CLI d’IA locales** comme **solution de secours en mode texte uniquement** lorsque les fournisseurs d’API sont indisponibles,
soumis à des limitations de débit ou se comportent temporairement de manière anormale. Cette approche est volontairement conservatrice :

- **Les outils sont désactivés** (aucun appel d’outil).
- **Texte en entrée → texte en sortie** (fiable).
- **Les sessions sont prises en charge** (les échanges suivants restent donc cohérents).
- **Les images peuvent être transmises** si la CLI accepte des chemins d’images.

Ce mécanisme est conçu comme un **filet de sécurité** plutôt que comme chemin principal. Utilisez-le lorsque vous
voulez des réponses textuelles qui « fonctionnent toujours » sans dépendre d’API externes.

<div id="beginner-friendly-quick-start">
  ## Démarrage rapide pour les débutants
</div>

Vous pouvez utiliser la CLI Claude Code **sans aucune configuration préalable** (OpenClaw inclut une configuration par défaut intégrée) :

```bash
openclaw agent --message "hi" --model claude-cli/opus-4.5
```

Codex CLI fonctionne lui aussi prêt à l&#39;emploi :

```bash
openclaw agent --message "hi" --model codex-cli/gpt-5.2-codex
```

Si votre Gateway s&#39;exécute sous launchd/systemd et que PATH est minimal, ajoutez simplement le chemin complet de la commande :

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude"
        }
      }
    }
  }
}
```

C’est tout. Aucune clé et aucune configuration d’authentification supplémentaire ne sont nécessaires au-delà de la CLI elle-même.


<div id="using-it-as-a-fallback">
  ## L&#39;utiliser comme solution de secours
</div>

Ajoutez un backend CLI à votre liste de secours afin qu&#39;il ne soit utilisé que lorsque les modèles principaux échouent :

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: [
          "claude-cli/opus-4.5"
        ]
      },
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "claude-cli/opus-4.5": {}
      }
    }
  }
}
```

Remarques :

* Si vous utilisez `agents.defaults.models` (liste d’autorisation), vous devez inclure `claude-cli/...`.
* Si le fournisseur principal échoue (problème d’authentification, limites de débit, dépassements de délai), OpenClaw essaiera ensuite d’utiliser le backend CLI.


<div id="configuration-overview">
  ## Aperçu de la configuration
</div>

Tous les backends de la CLI se trouvent sous :

```
agents.defaults.cliBackends
```

Chaque entrée est indexée par un **ID de fournisseur** (par ex. `claude-cli`, `my-cli`).
L’ID de fournisseur devient la partie gauche de la référence à votre modèle :

```
<provider>/<model>
```


<div id="example-configuration">
  ### Exemple de configuration
</div>

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude"
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          input: "arg",
          modelArg: "--model",
          modelAliases: {
            "claude-opus-4-5": "opus",
            "claude-sonnet-4-5": "sonnet"
          },
          sessionArg: "--session",
          sessionMode: "existing",
          sessionIdFields: ["session_id", "conversation_id"],
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
          serialize: true
        }
      }
    }
  }
}
```


<div id="how-it-works">
  ## Comment ça fonctionne
</div>

1) **Sélectionne un backend** en fonction du préfixe du fournisseur (`claude-cli/...`).
2) **Construit un prompt système** en utilisant le même prompt OpenClaw + le contexte d’espace de travail.
3) **Exécute la CLI** avec un identifiant de session (si supporté) afin que l’historique reste cohérent.
4) **Analyse la sortie** (JSON ou texte brut) et renvoie le texte final.
5) **Conserve les identifiants de session** par backend, afin que les requêtes suivantes réutilisent la même session CLI.

<div id="sessions">
  ## Sessions
</div>

- Si la CLI prend en charge les sessions, définissez `sessionArg` (par ex. `--session-id`) ou
  `sessionArgs` (avec le placeholder `{sessionId}`) lorsque l’ID doit être inséré
  dans plusieurs options.
- Si la CLI utilise une **sous-commande de reprise (resume)** avec des options différentes, définissez
  `resumeArgs` (remplace `args` lors de la reprise) et éventuellement `resumeOutput`
  (pour les reprises non-JSON).
- `sessionMode` :
  - `always` : toujours envoyer un ID de session (nouvel UUID si aucun n’est stocké).
  - `existing` : n’envoyer un ID de session que si un a déjà été stocké auparavant.
  - `none` : ne jamais envoyer d’ID de session.

<div id="images-pass-through">
  ## Images (transmises telles quelles)
</div>

Si votre CLI accepte des chemins d’accès aux images, définissez `imageArg` :

```json5
imageArg: "--image",
imageMode: "repeat"
```

OpenClaw écrit les images encodées en base64 dans des fichiers temporaires. Si `imageArg` est défini, ces
chemins sont transmis comme arguments de la CLI. Si `imageArg` est absent, OpenClaw ajoute les
chemins de fichier au prompt (injection de chemin), ce qui suffit pour les CLI qui chargent
automatiquement les fichiers locaux à partir de chemins bruts (comportement de la CLI Claude Code).


<div id="inputs-outputs">
  ## Entrées / sorties
</div>

- `output: "json"` (par défaut) tente d’analyser le JSON et d’extraire le texte + l’ID de session.
- `output: "jsonl"` analyse les flux JSONL (Codex CLI `--json`) et extrait le
  dernier message de l’agent ainsi que `thread_id` lorsqu’il est présent.
- `output: "text"` traite stdout comme la réponse finale.

Modes d’entrée :

- `input: "arg"` (par défaut) passe le prompt en tant que dernier argument de la CLI.
- `input: "stdin"` envoie le prompt via stdin.
- Si le prompt est très long et que `maxPromptArgChars` est défini, stdin est utilisé.

<div id="defaults-built-in">
  ## Valeurs par défaut (intégrées)
</div>

OpenClaw inclut une configuration par défaut pour `claude-cli` :

- `command: "claude"`
- `args: ["-p", "--output-format", "json", "--dangerously-skip-permissions"]`
- `resumeArgs: ["-p", "--output-format", "json", "--dangerously-skip-permissions", "--resume", "{sessionId}"]`
- `modelArg: "--model"`
- `systemPromptArg: "--append-system-prompt"`
- `sessionArg: "--session-id"`
- `systemPromptWhen: "first"`
- `sessionMode: "always"`

OpenClaw inclut également une configuration par défaut pour `codex-cli` :

- `command: "codex"`
- `args: ["exec","--json","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
- `resumeArgs: ["exec","resume","{sessionId}","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
- `output: "jsonl"`
- `resumeOutput: "text"`
- `modelArg: "--model"`
- `imageArg: "--image"`
- `sessionMode: "existing"`

Ne remplacez ces valeurs que si nécessaire (cas courant : chemin absolu pour `command`).

<div id="limitations">
  ## Limitations
</div>

- **Pas d’outils OpenClaw** (le backend CLI ne reçoit jamais d’appels d’outils). Certaines CLI
  peuvent néanmoins exécuter leurs propres outils d’agent.
- **Pas de streaming** (la sortie de la CLI est collectée puis renvoyée).
- **Les sorties structurées** dépendent du format JSON de la CLI.
- **Les sessions Codex CLI** reprennent via la sortie textuelle (sans JSONL), qui est moins
  structurée que l’exécution initiale avec `--json`. Les sessions OpenClaw continuent de fonctionner
  normalement.

<div id="troubleshooting">
  ## Dépannage
</div>

- **CLI introuvable** : définissez `command` avec un chemin complet.
- **Nom de modèle incorrect** : utilisez `modelAliases` pour faire correspondre `provider/model` → modèle CLI.
- **Aucune continuité de session** : assurez-vous que `sessionArg` est défini et que `sessionMode` n'est pas
  `none` (la CLI Codex ne peut actuellement pas reprendre avec une sortie JSON).
- **Images ignorées** : définissez `imageArg` (et vérifiez que la CLI prend en charge les chemins de fichiers).