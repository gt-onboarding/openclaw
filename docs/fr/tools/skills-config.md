---
title: Configuration des compétences
summary: "Schéma et exemples de configuration des compétences"
read_when:
  - Ajout ou modification de la configuration des compétences
  - Ajustement de la liste d’autorisation intégrée ou du comportement d’installation
---

<div id="skills-config">
  # Configuration des compétences
</div>

Toute la configuration relative aux compétences se trouve dans `skills` dans `~/.openclaw/openclaw.json`.

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: [
        "~/Projects/agent-scripts/skills",
        "~/Projects/oss/some-skill-pack/skills"
      ],
      watch: true,
      watchDebounceMs: 250
    },
    install: {
      preferBrew: true,
      nodeManager: "npm" // npm | pnpm | yarn | bun (le runtime du Gateway reste Node ; bun non recommandé)
    },
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE"
        }
      },
      peekaboo: { enabled: true },
      sag: { enabled: false }
    }
  }
}
```


<div id="fields">
  ## Champs
</div>

- `allowBundled` : liste d’autorisation optionnelle pour les compétences **intégrées** uniquement. Lorsqu’elle est définie, seules les compétences intégrées de la liste sont éligibles (les compétences gérées/espace de travail ne sont pas affectées).
- `load.extraDirs` : répertoires de compétences supplémentaires à analyser (priorité la plus faible).
- `load.watch` : surveille les dossiers de compétences et actualise l’instantané des compétences (valeur par défaut : true).
- `load.watchDebounceMs` : délai de temporisation pour les événements du mécanisme de surveillance des compétences, en millisecondes (valeur par défaut : 250).
- `install.preferBrew` : privilégie les installateurs brew lorsqu’ils sont disponibles (valeur par défaut : true).
- `install.nodeManager` : préférence de gestionnaire d’installation Node (`npm` | `pnpm` | `yarn` | `bun`, valeur par défaut : npm).
  Cela affecte uniquement les **installations de compétences** ; le runtime du Gateway doit toujours être Node
  (Bun n’est pas recommandé pour WhatsApp/Telegram).
- `entries.<skillKey>` : remplacements par compétence.

Champs par compétence :

- `enabled` : définissez `false` pour désactiver une compétence même si elle est intégrée/installée.
- `env` : variables d’environnement injectées pour l’exécution de l’agent (uniquement si elles ne sont pas déjà définies).
- `apiKey` : option pratique facultative pour les compétences qui déclarent une variable d’environnement principale.

<div id="notes">
  ## Remarques
</div>

- Les clés sous `entries` correspondent par défaut au nom de la compétence. Si une compétence définit
  `metadata.openclaw.skillKey`, utilisez plutôt cette clé.
- Les modifications apportées aux compétences sont prises en compte au prochain tour de l’agent lorsque le watcher est activé.

<div id="sandboxed-skills-env-vars">
  ### Compétences en sandbox + variables d'environnement
</div>

Lorsqu'une session est exécutée en **sandbox**, les processus des compétences s'exécutent dans Docker. La sandbox
n'hérite **pas** du `process.env` de l'hôte.

Utilisez l'une des options suivantes :

- `agents.defaults.sandbox.docker.env` (ou par agent `agents.list[].sandbox.docker.env`)
- intégrer les variables d'environnement dans votre image de sandbox personnalisée

Les `env` globales et `skills.entries.<skill>.env/apiKey` s'appliquent uniquement aux exécutions sur l'**hôte**.