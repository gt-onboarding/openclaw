---
title: Hooks
summary: "Hooks : automatisation événementielle pour les commandes et les événements de cycle de vie"
read_when:
  - Vous avez besoin d'une automatisation événementielle pour /new, /reset, /stop et les événements de cycle de vie des agents
  - Vous cherchez à créer, installer ou déboguer des hooks
---

<div id="hooks">
  # Hooks
</div>

Les hooks fournissent un système événementiel extensible pour automatiser des actions en réponse aux commandes et événements des agents. Les hooks sont automatiquement détectés dans des répertoires et peuvent être gérés via des commandes CLI, de manière similaire au fonctionnement des compétences dans OpenClaw.

<div id="getting-oriented">
  ## Premiers repères
</div>

Les hooks sont de petits scripts qui s’exécutent lorsqu’il se passe quelque chose. Il en existe deux types :

* **Hooks** (cette page) : s’exécutent au sein du Gateway lorsque des événements d’agent se produisent, comme `/new`, `/reset`, `/stop`, ou d’autres événements de cycle de vie.
* **Webhooks** : webhooks HTTP externes qui permettent à d’autres systèmes de déclencher des traitements dans OpenClaw. Voir [Webhook Hooks](/fr/automation/webhook) ou utiliser `openclaw webhooks` pour les commandes d’assistance Gmail.

Les hooks peuvent aussi être regroupés dans des plugins ; voir [Plugins](/fr/plugin#plugin-hooks).

Cas d’usage courants :

* Enregistrer un instantané de la mémoire lorsque vous réinitialisez une session
* Conserver une piste d’audit des commandes pour le dépannage ou la conformité
* Déclencher une automatisation de suivi lorsqu’une session démarre ou se termine
* Écrire des fichiers dans l’espace de travail de l’agent ou appeler des API externes lorsque des événements se déclenchent

Si vous pouvez écrire une petite fonction TypeScript, vous pouvez écrire un hook. Les hooks sont détectés automatiquement, et vous les activez ou les désactivez via la CLI.

<div id="overview">
  ## Vue d’ensemble
</div>

Le système de hooks vous permet de :

* Sauvegarder le contexte de session en mémoire lorsque `/new` est exécuté
* Journaliser toutes les commandes à des fins d’audit
* Déclencher des automatisations personnalisées sur les événements du cycle de vie des agents
* Étendre le comportement d’OpenClaw sans modifier le cœur du code

<div id="getting-started">
  ## Premiers pas
</div>

<div id="bundled-hooks">
  ### Hooks fournis
</div>

OpenClaw est livré avec quatre hooks intégrés qui sont découverts automatiquement :

* **💾 session-memory** : Enregistre le contexte de la session dans l&#39;espace de travail de votre agent (par défaut `~/.openclaw/workspace/memory/`) lorsque vous exécutez `/new`
* **📝 command-logger** : Journalise tous les événements de commande dans `~/.openclaw/logs/commands.log`
* **🚀 boot-md** : Exécute `BOOT.md` au démarrage du Gateway (nécessite l&#39;activation des hooks internes)
* **😈 soul-evil** : Remplace le contenu injecté de `SOUL.md` par `SOUL_EVIL.md` pendant une fenêtre de purge ou de manière aléatoire

Listez les hooks disponibles :

```bash
openclaw hooks list
```

Activer un hook :

```bash
openclaw hooks enable session-memory
```

Vérifiez l&#39;état du hook :

```bash
openclaw hooks check
```

Obtenir des informations détaillées :

```bash
openclaw hooks info session-memory
```

<div id="onboarding">
  ### Mise en route
</div>

Lors de la mise en route (`openclaw onboard`), l&#39;assistant vous proposera d&#39;activer les hooks recommandés. Il détecte automatiquement les hooks pouvant être activés et vous les propose à la sélection.

<div id="hook-discovery">
  ## Découverte des Hooks
</div>

Les hooks sont automatiquement découverts à partir de trois répertoires (par ordre de priorité) :

1. **Hooks de l’espace de travail** : `<workspace>/hooks/` (par agent, priorité la plus élevée)
2. **Hooks gérés** : `~/.openclaw/hooks/` (installés par l’utilisateur, partagés entre les espaces de travail)
3. **Hooks intégrés** : `<openclaw>/dist/hooks/bundled/` (fournis avec OpenClaw)

Les répertoires de hooks gérés peuvent être soit un **hook unique**, soit un **pack de hooks** (répertoire de package).

Chaque hook est un répertoire contenant :

```
my-hook/
├── HOOK.md          # Métadonnées + documentation
└── handler.ts       # Implémentation du gestionnaire
```

<div id="hook-packs-npmarchives">
  ## Packs de hooks (npm/archives)
</div>

Les packs de hooks sont des packages npm standard qui exportent un ou plusieurs hooks via `openclaw.hooks` dans
`package.json`. Installez-les avec :

```bash
openclaw hooks install <path-or-spec>
```

Exemple de fichier `package.json` :

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "openclaw": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

Chaque entrée pointe vers un répertoire de hook contenant `HOOK.md` et `handler.ts` (ou `index.ts`).
Les packs de hooks peuvent inclure des dépendances, qui seront installées dans `~/.openclaw/hooks/<id>`.

<div id="hook-structure">
  ## Structure d’un hook
</div>

<div id="hookmd-format">
  ### Format de HOOK.md
</div>

Le fichier `HOOK.md` contient des métadonnées au format frontmatter YAML ainsi que de la documentation Markdown :

```markdown
---
name: my-hook
description: "Short description of what this hook does"
homepage: https://docs.openclaw.ai/hooks#my-hook
metadata: {"openclaw":{"emoji":"🔗","events":["command:new"],"requires":{"bins":["node"]}}}
---

# My Hook

Detailed documentation goes here...

## What It Does

- Listens for `/new` commands
- Performs some action
- Logs the result

## Requirements

- Node.js must be installed

<div id="configuration">
  ## Configuration
</div>

No configuration needed.
```

<div id="metadata-fields">
  ### Champs de métadonnées
</div>

L&#39;objet `metadata.openclaw` prend en charge :

* **`emoji`** : Emoji affiché dans la CLI (par exemple, `"💾"`)
* **`events`** : Tableau d&#39;événements à écouter (par exemple, `["command:new", "command:reset"]`)
* **`export`** : Export nommé à utiliser (par défaut `"default"`)
* **`homepage`** : URL de la documentation
* **`requires`** : Prérequis facultatifs
  * **`bins`** : Binaires requis dans le PATH (par exemple, `["git", "node"]`)
  * **`anyBins`** : Au moins un de ces binaires doit être présent
  * **`env`** : Variables d&#39;environnement requises
  * **`config`** : Chemins de configuration requis (par exemple, `["workspace.dir"]`)
  * **`os`** : Plateformes requises (par exemple, `["darwin", "linux"]`)
* **`always`** : Contourne les vérifications d&#39;éligibilité (booléen)
* **`install`** : Méthodes d&#39;installation (pour les hooks intégrés : `[{"id":"bundled","kind":"bundled"}]`)

<div id="handler-implementation">
  ### Implémentation du gestionnaire
</div>

Le fichier `handler.ts` exporte une fonction `HookHandler` :

```typescript
import type { HookHandler } from '../../src/hooks/hooks.js';

const myHandler: HookHandler = async (event) => {
  // Se déclenche uniquement sur la commande 'new'
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  console.log(`[my-hook] Commande new déclenchée`);
  console.log(`  Session : ${event.sessionKey}`);
  console.log(`  Horodatage : ${event.timestamp.toISOString()}`);

  // Votre logique personnalisée ici

  // Envoie optionnel d'un message à l'utilisateur
  event.messages.push('✨ Mon hook a été exécuté !');
};

export default myHandler;
```

<div id="event-context">
  #### Contexte de l&#39;événement
</div>

Chaque événement contient :

```typescript
{
  type: 'command' | 'session' | 'agent' | 'gateway',
  action: string,              // e.g., 'new', 'reset', 'stop'
  sessionKey: string,          // Session identifier
  timestamp: Date,             // When the event occurred
  messages: string[],          // Ajouter les messages ici pour les envoyer à l'utilisateur
  context: {
    sessionEntry?: SessionEntry,
    sessionId?: string,
    sessionFile?: string,
    commandSource?: string,    // e.g., 'whatsapp', 'telegram'
    senderId?: string,
    workspaceDir?: string,
    bootstrapFiles?: WorkspaceBootstrapFile[],
    cfg?: OpenClawConfig
  }
}
```

<div id="event-types">
  ## Types d’événements
</div>

<div id="command-events">
  ### Événements de commande
</div>

Déclenchés lorsque des commandes d’agent sont exécutées :

* **`command`** : Tous les événements de commande (écouteur générique)
* **`command:new`** : Lorsque la commande `/new` est exécutée
* **`command:reset`** : Lorsque la commande `/reset` est exécutée
* **`command:stop`** : Lorsque la commande `/stop` est exécutée

<div id="agent-events">
  ### Événements d&#39;agent
</div>

* **`agent:bootstrap`** : Avant l’injection des fichiers de bootstrap de l’espace de travail (les hooks peuvent modifier `context.bootstrapFiles`)

<div id="gateway-events">
  ### Événements du Gateway
</div>

Déclenchés lorsque Gateway démarre :

* **`gateway:startup`** : Après le démarrage des canaux et le chargement des hooks

<div id="tool-result-hooks-plugin-api">
  ### Hooks de résultats d’outil (API de plugin)
</div>

Ces hooks ne sont pas des écouteurs de flux d’événements ; ils permettent aux plugins d’ajuster de manière synchrone les résultats d’outils avant leur enregistrement par OpenClaw.

* **`tool_result_persist`** : transformer les résultats d’outils avant qu’ils ne soient écrits dans la transcription de la session. Doit être synchrone ; renvoyer le payload de résultat d’outil mis à jour ou `undefined` pour le laisser inchangé. Voir [Agent Loop](/fr/concepts/agent-loop).

<div id="future-events">
  ### Événements futurs
</div>

Types d&#39;événements prévus :

* **`session:start`** : Lorsqu&#39;une nouvelle session démarre
* **`session:end`** : Lorsqu&#39;une session se termine
* **`agent:error`** : Lorsqu&#39;un agent rencontre une erreur
* **`message:sent`** : Lorsqu&#39;un message est envoyé
* **`message:received`** : Lorsqu&#39;un message est reçu

<div id="creating-custom-hooks">
  ## Créer des hooks personnalisés
</div>

<div id="1-choose-location">
  ### 1. Choisir l’emplacement
</div>

* **Hooks d’espace de travail** (`<workspace>/hooks/`) : Spécifiques à chaque agent, priorité la plus élevée
* **Hooks gérés** (`~/.openclaw/hooks/`) : Partagés entre les espaces de travail

<div id="2-create-directory-structure">
  ### 2. Créer l’arborescence de répertoires
</div>

```bash
mkdir -p ~/.openclaw/hooks/my-hook
cd ~/.openclaw/hooks/my-hook
```

<div id="3-create-hookmd">
  ### 3. Créez HOOK.md
</div>

```markdown
---
name: my-hook
description: "Does something useful"
metadata: {"openclaw":{"emoji":"🎯","events":["command:new"]}}
---

# My Custom Hook

This hook does something useful when you issue `/new`.
```

<div id="4-create-handlerts">
  ### 4. Créer le fichier handler.ts
</div>

```typescript
import type { HookHandler } from '../../src/hooks/hooks.js';

const handler: HookHandler = async (event) => {
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  console.log('[my-hook] Running!');
  // Votre logique ici
};

export default handler;
```

<div id="5-enable-and-test">
  ### 5. Activer et tester
</div>

```bash
# Vérifier que le hook est découvert
openclaw hooks list

# L'activer
openclaw hooks enable my-hook

# Redémarrer votre processus Gateway (redémarrage de l'application dans la barre de menus sur macOS, ou redémarrage de votre processus de développement)

# Déclencher l'événement
# Envoyer /new via votre canal de messagerie
```

<div id="configuration">
  ## Configuration
</div>

<div id="new-config-format-recommended">
  ### Nouveau format de configuration (recommandé)
</div>

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "session-memory": { "enabled": true },
        "command-logger": { "enabled": false }
      }
    }
  }
}
```

<div id="per-hook-configuration">
  ### Configuration propre à chaque hook
</div>

Chaque hook peut disposer de sa propre configuration :

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "my-hook": {
          "enabled": true,
          "env": {
            "MY_CUSTOM_VAR": "value"
          }
        }
      }
    }
  }
}
```

<div id="extra-directories">
  ### Répertoires supplémentaires
</div>

Chargez des hooks à partir d’autres répertoires :

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "load": {
        "extraDirs": ["/path/to/more/hooks"]
      }
    }
  }
}
```

<div id="legacy-config-format-still-supported">
  ### Ancien format de configuration (encore pris en charge)
</div>

L’ancien format de configuration fonctionne toujours pour des raisons de compatibilité ascendante :

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts",
          "export": "default"
        }
      ]
    }
  }
}
```

**Migration** : Utilisez le nouveau système de découverte pour les nouveaux hooks. Les gestionnaires hérités sont chargés après les hooks basés sur des répertoires.

<div id="cli-commands">
  ## Commandes CLI
</div>

<div id="list-hooks">
  ### Lister les hooks
</div>

```bash
# List all hooks
openclaw hooks list

# Show only eligible hooks
openclaw hooks list --eligible

# Sortie détaillée (afficher les exigences manquantes)
openclaw hooks list --verbose

# JSON output
openclaw hooks list --json
```

<div id="hook-information">
  ### Informations sur le hook
</div>

```bash
# Afficher des informations détaillées sur un hook
openclaw hooks info session-memory

# Sortie JSON
openclaw hooks info session-memory --json
```

<div id="check-eligibility">
  ### Vérifier l’éligibilité
</div>

```bash
# Afficher le résumé d'éligibilité
openclaw hooks check

# JSON output
openclaw hooks check --json
```

<div id="enabledisable">
  ### Activer/désactiver
</div>

```bash
# Enable a hook
openclaw hooks enable session-memory

# Désactiver un hook
openclaw hooks disable command-logger
```

## Hooks fournis

<div id="session-memory">
  ### session-memory
</div>

Enregistre le contexte de la session en mémoire lorsque vous exécutez `/new`.

**Événements** : `command:new`

**Prérequis** : `workspace.dir` doit être configuré

**Sortie** : `<workspace>/memory/YYYY-MM-DD-slug.md` (par défaut `~/.openclaw/workspace`)

**Ce que cela fait** :

1. Utilise l’entrée de session avant réinitialisation pour localiser la transcription appropriée
2. Extrait les 15 dernières lignes de la conversation
3. Utilise un LLM pour générer un slug de nom de fichier descriptif
4. Enregistre les métadonnées de la session dans un fichier mémoire daté

**Exemple de sortie** :

```markdown
# Session : 2026-01-16 14:30:00 UTC

- **Clé de session** : agent:main:main
- **ID de session** : abc123def456
- **Source** : telegram
```

**Exemples de noms de fichiers** :

* `2026-01-16-vendor-pitch.md`
* `2026-01-16-api-design.md`
* `2026-01-16-1430.md` (horodatage de repli si la génération du slug échoue)

**Activer** :

```bash
openclaw hooks enable session-memory
```

<div id="command-logger">
  ### command-logger
</div>

Enregistre tous les événements liés aux commandes dans un fichier d&#39;audit centralisé.

**Événements** : `command`

**Prérequis** : Aucun

**Sortie** : `~/.openclaw/logs/commands.log`

**Ce qu&#39;il fait** :

1. Capture les détails de l&#39;événement (action de commande, horodatage, clé de session, ID de l&#39;expéditeur, source)
2. Ajoute les entrées au fichier de journal au format JSONL
3. S&#39;exécute silencieusement en arrière-plan

**Exemples d&#39;entrées de journal** :

```jsonl
{"timestamp":"2026-01-16T14:30:00.000Z","action":"new","sessionKey":"agent:main:main","senderId":"+1234567890","source":"telegram"}
{"timestamp":"2026-01-16T15:45:22.000Z","action":"stop","sessionKey":"agent:main:main","senderId":"user@example.com","source":"whatsapp"}
```

**Consulter les journaux** :

```bash
# Voir les commandes récentes
tail -n 20 ~/.openclaw/logs/commands.log

# Pretty-print with jq
cat ~/.openclaw/logs/commands.log | jq .

# Filter by action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**Activation** :

```bash
openclaw hooks enable command-logger
```

<div id="soul-evil">
  ### soul-evil
</div>

Remplace le contenu injecté de `SOUL.md` par `SOUL_EVIL.md` pendant une fenêtre de purge ou au hasard.

**Événements** : `agent:bootstrap`

**Docs** : [SOUL Evil Hook](/fr/hooks/soul-evil)

**Sortie** : aucun fichier n’est écrit ; les remplacements se produisent uniquement en mémoire.

**Activation** :

```bash
openclaw hooks enable soul-evil
```

**Config** :

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 0.1,
          "purge": { "at": "21:00", "duration": "15m" }
        }
      }
    }
  }
}
```

<div id="boot-md">
  ### boot-md
</div>

Exécute `BOOT.md` au démarrage du Gateway (après le démarrage des canaux).
Les hooks internes doivent être activés pour que cela fonctionne.

**Événements** : `gateway:startup`

**Prérequis** : `workspace.dir` doit être configuré

**Ce que cela fait** :

1. Lit `BOOT.md` à partir de votre espace de travail
2. Exécute les instructions via l’agent runner
3. Envoie tous les messages sortants demandés via l’outil de messagerie

**Activation** :

```bash
openclaw hooks enable boot-md
```

<div id="best-practices">
  ## Bonnes pratiques
</div>

<div id="keep-handlers-fast">
  ### Veillez à ce que les handlers restent rapides
</div>

Les hooks sont exécutés pendant le traitement des commandes. Gardez-les légers :

```typescript
// ✓ Bon - travail asynchrone, retourne immédiatement
const handler: HookHandler = async (event) => {
  void processInBackground(event); // Lance et oublie
};

// ✗ Mauvais - bloque le traitement des commandes
const handler: HookHandler = async (event) => {
  await slowDatabaseQuery(event);
  await evenSlowerAPICall(event);
};
```

<div id="handle-errors-gracefully">
  ### Gérer les erreurs proprement
</div>

Encapsulez toujours les opérations risquées :

```typescript
const handler: HookHandler = async (event) => {
  try {
    await riskyOperation(event);
  } catch (err) {
    console.error('[my-handler] Failed:', err instanceof Error ? err.message : String(err));
    // Ne pas lever d'exception - laisser les autres gestionnaires s'exécuter
  }
};
```

<div id="filter-events-early">
  ### Filtrer les événements en amont
</div>

Retournez immédiatement si l’événement n’est pas pertinent :

```typescript
const handler: HookHandler = async (event) => {
  // Ne traiter que les commandes 'new'
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  // Votre logique ici
};
```

<div id="use-specific-event-keys">
  ### Utilisez des clés d’événement spécifiques
</div>

Précisez les événements exacts dans les métadonnées lorsque possible :

```yaml
metadata: {"openclaw":{"events":["command:new"]}}  # Spécifique
```

Plutôt que :

```yaml
metadata: {"openclaw":{"events":["command"]}}      # Général - plus de charge
```

<div id="debugging">
  ## Débogage
</div>

<div id="enable-hook-logging">
  ### Activer la journalisation des hooks
</div>

Gateway journalise le chargement des hooks au démarrage :

```
Registered hook: session-memory -> command:new
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

<div id="check-discovery">
  ### Vérifier la détection
</div>

Répertorier tous les hooks détectés :

```bash
openclaw hooks list --verbose
```

<div id="check-registration">
  ### Vérifier l’enregistrement
</div>

Dans votre gestionnaire, consignez dans les logs le moment où il est appelé :

```typescript
const handler: HookHandler = async (event) => {
  console.log('[my-handler] Triggered:', event.type, event.action);
  // Votre logique ici
};
```

<div id="verify-eligibility">
  ### Vérifiez l’éligibilité
</div>

Vérifiez pourquoi un hook n’est pas éligible :

```bash
openclaw hooks info my-hook
```

Recherchez d’éventuelles exigences manquantes dans la sortie.

<div id="testing">
  ## Tests
</div>

<div id="gateway-logs">
  ### Journaux du Gateway
</div>

Surveillez les journaux du Gateway pour suivre l&#39;exécution des hooks :

```bash
# macOS
./scripts/clawlog.sh -f

# Autres plateformes
tail -f ~/.openclaw/gateway.log
```

<div id="test-hooks-directly">
  ### Tester les hooks directement
</div>

Testez vos gestionnaires de hooks isolément :

```typescript
import { test } from 'vitest';
import { createHookEvent } from './src/hooks/hooks.js';
import myHandler from './hooks/my-hook/handler.js';

test('my handler works', async () => {
  const event = createHookEvent('command', 'new', 'test-session', {
    foo: 'bar'
  });

  await myHandler(event);

  // Assert side effects
});
```

<div id="architecture">
  ## Architecture
</div>

<div id="core-components">
  ### Composants principaux
</div>

* **`src/hooks/types.ts`**: Définitions de types
* **`src/hooks/workspace.ts`**: Analyse et chargement de répertoires
* **`src/hooks/frontmatter.ts`**: Analyse des métadonnées HOOK.md
* **`src/hooks/config.ts`**: Vérification de l’éligibilité
* **`src/hooks/hooks-status.ts`**: Rapport de statut
* **`src/hooks/loader.ts`**: Chargeur de modules dynamiques
* **`src/cli/hooks-cli.ts`**: Commandes CLI
* **`src/gateway/server-startup.ts`**: Charge les hooks au démarrage du Gateway
* **`src/auto-reply/reply/commands-core.ts`**: Déclenche les événements liés aux commandes

<div id="discovery-flow">
  ### Flux de découverte
</div>

```
Gateway startup
    ↓
Scan directories (workspace → managed → bundled)
    ↓
Parse HOOK.md files
    ↓
Check eligibility (bins, env, config, os)
    ↓
Load handlers from eligible hooks
    ↓
Register handlers for events
```

<div id="event-flow">
  ### Flux d’événements
</div>

```
User sends /new
    ↓
Command validation
    ↓
Create hook event
    ↓
Trigger hook (all registered handlers)
    ↓
Command processing continues
    ↓
Session reset
```

<div id="troubleshooting">
  ## Résolution des problèmes
</div>

<div id="hook-not-discovered">
  ### Hook non détecté
</div>

1. Vérifiez la structure du répertoire :
   ```bash
   ls -la ~/.openclaw/hooks/my-hook/
   # Vous devriez voir : HOOK.md, handler.ts
   ```

2. Vérifiez le format de HOOK.md :
   ```bash
   cat ~/.openclaw/hooks/my-hook/HOOK.md
   # Doit contenir un front matter YAML avec le champ name et les métadonnées
   ```

3. Répertoriez tous les hooks détectés :
   ```bash
   openclaw hooks list
   ```

<div id="hook-not-eligible">
  ### Hook non éligible
</div>

Vérifiez les prérequis :

```bash
openclaw hooks info my-hook
```

Vérifiez l&#39;absence des éléments suivants :

* Binaires (vérifiez `PATH`)
* Variables d&#39;environnement
* Valeurs de configuration
* Compatibilité avec le système d&#39;exploitation

<div id="hook-not-executing">
  ### Le hook ne s&#39;exécute pas
</div>

1. Vérifiez que le hook est bien activé :
   ```bash
   openclaw hooks list
   # Doit afficher un ✓ à côté des hooks activés
   ```

2. Redémarrez le processus Gateway pour recharger les hooks.

3. Consultez les journaux de Gateway pour détecter d’éventuelles erreurs :
   ```bash
   ./scripts/clawlog.sh | grep hook
   ```

<div id="handler-errors">
  ### Erreurs du gestionnaire
</div>

Vérifiez les erreurs de TypeScript ou d’import :

```bash
# Tester l'importation directement
node -e "import('./path/to/handler.ts').then(console.log)"
```

<div id="migration-guide">
  ## Guide de migration
</div>

<div id="from-legacy-config-to-discovery">
  ### De l’ancienne configuration à Discovery
</div>

**Avant** :

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts"
        }
      ]
    }
  }
}
```

**Après** :

1. Créer le répertoire du hook :
   ```bash
   mkdir -p ~/.openclaw/hooks/my-hook
   mv ./hooks/handlers/my-handler.ts ~/.openclaw/hooks/my-hook/handler.ts
   ```

2. Créer HOOK.md :
   ```markdown
   ---
   name: my-hook
   description: "Mon hook personnalisé"
   metadata: {"openclaw":{"emoji":"🎯","events":["command:new"]}}
   ---

   # Mon hook

   Fait quelque chose d'utile.
   ```

3. Mettre à jour la configuration :
   ```json
   {
     "hooks": {
       "internal": {
         "enabled": true,
         "entries": {
           "my-hook": { "enabled": true }
         }
       }
     }
   }
   ```

4. Vérifier et redémarrer le processus de votre Gateway :
   ```bash
   openclaw hooks list
   # Doit afficher : 🎯 my-hook ✓
   ```

**Avantages de la migration** :

* Découverte automatique
* Gestion via la CLI
* Contrôle de l&#39;éligibilité
* Meilleure documentation
* Structure cohérente

<div id="see-also">
  ## Voir aussi
</div>

* [Référence de la CLI : hooks](/fr/cli/hooks)
* [README des hooks intégrés](https://github.com/openclaw/openclaw/tree/main/src/hooks/bundled)
* [Hooks de webhook](/fr/automation/webhook)
* [Configuration](/fr/gateway/configuration#hooks)