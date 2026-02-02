---
title: Outils
summary: "Surface d'outils d'agent pour OpenClaw (navigateur, canvas, nœuds, messages, cron) remplaçant les anciennes compétences `openclaw-*`"
read_when:
  - Ajout ou modification d'outils d'agent
  - Retrait ou modification des compétences `openclaw-*`
---

<div id="tools-openclaw">
  # Outils (OpenClaw)
</div>

OpenClaw met à disposition des **outils d’agent de premier niveau** pour le navigateur, le canevas, les nœuds et cron.
Ils remplacent les anciennes compétences `openclaw-*` : les outils sont typés, sans appel au shell,
et l’agent doit s’appuyer directement dessus.

<div id="disabling-tools">
  ## Désactivation des outils
</div>

Vous pouvez autoriser ou interdire globalement des outils via `tools.allow` / `tools.deny` dans `openclaw.json`
(`deny` est prioritaire). Cela empêche les outils non autorisés d’être envoyés aux fournisseurs de modèles.

```json5
{
  tools: { deny: ["browser"] }
}
```

Remarques :

* La correspondance est insensible à la casse.
* Les caractères génériques `*` sont pris en charge (`"*"` signifie tous les outils).
* Si `tools.allow` ne fait référence qu&#39;à des noms d&#39;outils de plugin inconnus ou non chargés, OpenClaw consigne un avertissement dans les journaux et ignore la liste d’autorisation afin que les outils principaux restent disponibles.

<div id="tool-profiles-base-allowlist">
  ## Profils d’outils (liste d’autorisation de base)
</div>

`tools.profile` définit une **liste d’autorisation de base des outils** avant `tools.allow`/`tools.deny`.
Surcharge par agent : `agents.list[].tools.profile`.

Profils :

* `minimal` : uniquement `session_status`
* `coding` : `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
* `messaging` : `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
* `full` : aucune restriction (équivalent à une valeur non définie)

Exemple (par défaut uniquement messagerie, autoriser aussi les outils Slack + Discord) :

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"]
  }
}
```

Exemple (profil de programmation, mais interdire `exec/process` partout) :

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"]
  }
}
```

Exemple (profil de programmation global, agent de support dédié à la messagerie) :

```json5
{
  tools: { profile: "coding" },
  agents: {
    list: [
      {
        id: "support",
        tools: { profile: "messaging", allow: ["slack"] }
      }
    ]
  }
}
```

<div id="provider-specific-tool-policy">
  ## Stratégie d’outils spécifique au fournisseur
</div>

Utilisez `tools.byProvider` pour **restreindre encore davantage** les outils pour certains fournisseurs
(ou un seul `provider/model`) sans modifier vos paramètres globaux par défaut.
Surcharge par agent : `agents.list[].tools.byProvider`.

Ceci est appliqué **après** le profil d’outils de base et **avant** les listes allow/deny,
il ne peut donc que réduire l’ensemble des outils disponibles.
Les clés de fournisseur acceptent soit `provider` (par ex. `google-antigravity`), soit
`provider/model` (par ex. `openai/gpt-5.2`).

Exemple (conserver le profil global d’écriture de code, mais outils minimaux pour Google Antigravity) :

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" }
    }
  }
}
```

Exemple (liste d’autorisation spécifique au fournisseur/modèle pour un endpoint peu fiable) :

```json5
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] }
    }
  }
}
```

Exemple (surcharge propre à un agent pour un seul fournisseur) :

```json5
{
  agents: {
    list: [
      {
        id: "support",
        tools: {
          byProvider: {
            "google-antigravity": { allow: ["message", "sessions_list"] }
          }
        }
      }
    ]
  }
}
```

<div id="tool-groups-shorthands">
  ## Groupes d’outils (raccourcis)
</div>

Les politiques d’outils (globales, agent, sandbox) prennent en charge les entrées `group:*` qui s’étendent à plusieurs outils.
Utilisez-les dans `tools.allow` / `tools.deny`.

Groupes disponibles :

* `group:runtime`: `exec`, `bash`, `process`
* `group:fs`: `read`, `write`, `edit`, `apply_patch`
* `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory`: `memory_search`, `memory_get`
* `group:web`: `web_search`, `web_fetch`
* `group:ui`: `browser`, `canvas`
* `group:automation`: `cron`, `gateway`
* `group:messaging`: `message`
* `group:nodes`: `nodes`
* `group:openclaw`: tous les outils OpenClaw intégrés (hors plugins de fournisseurs)

Exemple (autoriser uniquement les outils de fichiers + l’outil `browser`) :

```json5
{
  tools: {
    allow: ["group:fs", "browser"]
  }
}
```

<div id="plugins-tools">
  ## Plugins + outils
</div>

Les plugins peuvent enregistrer **des outils supplémentaires** (et des commandes CLI) au-delà du jeu de base.
Voir [Plugins](/fr/plugin) pour l&#39;installation + la configuration, et [Skills](/fr/tools/skills) pour savoir comment
les instructions d&#39;utilisation des outils sont injectées dans les prompts. Certains plugins incluent leurs propres compétences
en plus des outils (par exemple, le plugin voice-call).

Outils de plugins optionnels :

* [Lobster](/fr/tools/lobster) : runtime de workflow typé avec approbations reprenables (nécessite le CLI Lobster sur l&#39;hôte du Gateway).
* [LLM Task](/fr/tools/llm-task) : étape LLM produisant uniquement du JSON pour une sortie de workflow structurée (validation de schéma optionnelle).

<div id="tool-inventory">
  ## Inventaire des outils
</div>

<div id="apply_patch">
  ### `apply_patch`
</div>

Applique des correctifs structurés à un ou plusieurs fichiers. À utiliser pour des modifications comportant plusieurs blocs.
Expérimental : à activer via `tools.exec.applyPatch.enabled` (modèles OpenAI uniquement).

<div id="exec">
  ### `exec`
</div>

Exécute des commandes shell dans l’espace de travail.

Paramètres principaux :

* `command` (obligatoire)
* `yieldMs` (mise en arrière-plan automatique après dépassement de délai, par défaut 10000)
* `background` (mise en arrière-plan immédiate)
* `timeout` (secondes ; tue le processus en cas de dépassement, par défaut 1800)
* `elevated` (bool ; s’exécute sur l’hôte si le mode élevé est activé/autorisé ; ne change le comportement que lorsque l’agent est exécuté dans un sandbox)
* `host` (`sandbox | gateway | node`)
* `security` (`deny | allowlist | full`)
* `ask` (`off | on-miss | always`)
* `node` (id/nom du nœud pour `host=node`)
* Besoin d’un vrai TTY ? Définissez `pty: true`.

Notes :

* Renvoie `status: "running"` avec un `sessionId` lorsqu’il est mis en arrière-plan.
* Utilisez `process` pour interroger/journaliser/écrire/tuer/effacer les sessions en arrière-plan.
* Si `process` est interdit, `exec` s’exécute de manière synchrone et ignore `yieldMs`/`background`.
* `elevated` est conditionné par `tools.elevated` plus toute surcharge `agents.list[].tools.elevated` (les deux doivent autoriser) et est un alias pour `host=gateway` + `security=full`.
* `elevated` ne change le comportement que lorsque l’agent est exécuté dans un sandbox (sinon, c’est un no-op).
* `host=node` peut cibler une application compagnon macOS ou un hôte de nœud headless (`openclaw node run`).
* Approbations Gateway/nœud et listes d’autorisation : [Exec approvals](/fr/tools/exec-approvals).

<div id="process">
  ### `process`
</div>

Gérer les sessions d&#39;exécution en arrière-plan.

Actions principales :

* `list`, `poll`, `log`, `write`, `kill`, `clear`, `remove`

Notes :

* `poll` renvoie la nouvelle sortie et le code de sortie une fois l&#39;exécution terminée.
* `log` prend en charge la pagination par lignes avec `offset`/`limit` (omettre `offset` pour récupérer les N dernières lignes).
* `process` est propre à chaque agent ; les sessions des autres agents ne sont pas visibles.

<div id="web_search">
  ### `web_search`
</div>

Effectue une recherche sur le web à l&#39;aide de l&#39;API Brave Search.

Paramètres principaux :

* `query` (obligatoire)
* `count` (1–10 ; valeur par défaut définie par `tools.web.search.maxResults`)

Remarques :

* Nécessite une clé d&#39;API Brave (recommandé : `openclaw configure --section web`, ou définir `BRAVE_API_KEY`).
* Activer via `tools.web.search.enabled`.
* Les réponses sont mises en cache (15 min par défaut).
* Voir [Outils web](/fr/tools/web) pour la configuration.

<div id="web_fetch">
  ### `web_fetch`
</div>

Récupérer et extraire le contenu lisible d’une URL (HTML → markdown/texte).

Paramètres principaux :

* `url` (obligatoire)
* `extractMode` (`markdown` | `text`)
* `maxChars` (tronquer les pages longues)

Remarques :

* À activer via `tools.web.fetch.enabled`.
* Les réponses sont mises en cache (15 min par défaut).
* Pour les sites très dépendants de JavaScript, privilégier l’outil `browser`.
* Voir [Web tools](/fr/tools/web) pour la configuration.
* Voir [Firecrawl](/fr/tools/firecrawl) pour le mécanisme de secours anti-bot facultatif.

<div id="browser">
  ### `browser`
</div>

Contrôle le navigateur dédié géré par OpenClaw.

Actions principales :

* `status`, `start`, `stop`, `tabs`, `open`, `focus`, `close`
* `snapshot` (aria/ai)
* `screenshot` (renvoie un bloc image + `MEDIA:<path>`)
* `act` (actions UI : click/type/press/hover/drag/select/fill/resize/wait/evaluate)
* `navigate`, `console`, `pdf`, `upload`, `dialog`

Gestion des profils :

* `profiles` — liste tous les profils navigateur avec leur statut
* `create-profile` — crée un nouveau profil avec port attribué automatiquement (ou `cdpUrl`)
* `delete-profile` — arrête le navigateur, supprime les données utilisateur, le retire de la config (local uniquement)
* `reset-profile` — tue le processus orphelin sur le port du profil (local uniquement)

Paramètres courants :

* `profile` (optionnel ; par défaut `browser.defaultProfile`)
* `target` (`sandbox` | `host` | `node`)
* `node` (optionnel ; choisit un id/nom de nœud spécifique)
  Notes :
* Nécessite `browser.enabled=true` (la valeur par défaut est `true` ; définissez `false` pour désactiver).
* Toutes les actions acceptent un paramètre `profile` optionnel pour la prise en charge multi‑instance.
* Quand `profile` est omis, utilise `browser.defaultProfile` (par défaut « chrome »).
* Noms de profil : alphanumérique en minuscules + tirets uniquement (64 caractères max).
* Plage de ports : 18800-18899 (~100 profils max).
* Les profils distants sont uniquement en mode attachement (pas de start/stop/reset).
* Si un nœud compatible navigateur est connecté, l’outil peut s’y router automatiquement (sauf si vous forcez `target`).
* `snapshot` utilise `ai` par défaut quand Playwright est installé ; utilisez `aria` pour l’arbre d’accessibilité.
* `snapshot` prend aussi en charge des options de capture par rôle (`interactive`, `compact`, `depth`, `selector`) qui renvoient des références de type `e12`.
* `act` requiert un `ref` issu de `snapshot` (numérique `12` pour les captures AI, ou `e12` pour les captures par rôle) ; utilisez `evaluate` pour les rares besoins en sélecteurs CSS.
* Évitez `act` → `wait` par défaut ; utilisez‑le uniquement dans des cas exceptionnels (aucun état de l’UI fiable sur lequel attendre).
* `upload` peut optionnellement passer un `ref` pour cliquer automatiquement après l’armement.
* `upload` prend aussi en charge `inputRef` (référence aria) ou `element` (sélecteur CSS) pour renseigner directement `<input type="file">`.

<div id="canvas">
  ### `canvas`
</div>

Pilote le Canvas du nœud (present, eval, snapshot, A2UI).

Actions principales :

* `present`, `hide`, `navigate`, `eval`
* `snapshot` (renvoie un bloc d’image + `MEDIA:<path>`)
* `a2ui_push`, `a2ui_reset`

Remarques :

* Utilise l’appel `node.invoke` du Gateway en interne.
* Si aucun `node` n’est fourni, l’outil en choisit un par défaut (nœud unique connecté ou nœud Mac local).
* A2UI n’est disponible qu’en v0.8 (pas de `createSurface`) ; la CLI rejette le JSONL v0.9 avec des erreurs de ligne.
* Test rapide : `openclaw nodes canvas a2ui push --node <id> --text "Hello from A2UI"`.

<div id="nodes">
  ### `nodes`
</div>

Découvrir et cibler les nœuds appairés ; envoyer des notifications ; capturer la caméra ou l’écran.

Actions principales :

* `status`, `describe`
* `pending`, `approve`, `reject` (appairage)
* `notify` (macOS `system.notify`)
* `run` (macOS `system.run`)
* `camera_snap`, `camera_clip`, `screen_record`
* `location_get`

Notes :

* Les commandes de caméra/écran nécessitent que l’app du nœud soit au premier plan.
* Les images renvoient des blocs d’image + `MEDIA:<path>`.
* Les vidéos renvoient `FILE:<path>` (mp4).
* La localisation renvoie un payload JSON (lat/lon/accuracy/timestamp).
* Paramètres de `run` : un tableau argv `command` ; `cwd` facultatif, `env` (`KEY=VAL`), `commandTimeoutMs`, `invokeTimeoutMs`, `needsScreenRecording`.

Exemple (`run`) :

```json
{
  "action": "run",
  "node": "office-mac",
  "command": ["echo", "Hello"],
  "env": ["FOO=bar"],
  "commandTimeoutMs": 12000,
  "invokeTimeoutMs": 45000,
  "needsScreenRecording": false
}
```

<div id="image">
  ### `image`
</div>

Analyser une image avec le modèle d’image configuré.

Paramètres principaux :

* `image` (chemin ou URL obligatoire)
* `prompt` (facultatif ; valeur par défaut : &quot;Describe the image.&quot;)
* `model` (remplacement facultatif)
* `maxBytesMb` (limite de taille facultative)

Remarques :

* Disponible uniquement lorsque `agents.defaults.imageModel` est configuré (modèle principal ou de secours), ou lorsqu’un modèle d’image implicite peut être déduit de votre modèle par défaut et de l’authentification configurée (appairage en mode « best-effort »).
* Utilise directement le modèle d’image (indépendamment du modèle de chat principal).

<div id="message">
  ### `message`
</div>

Envoyer des messages et des actions de canal sur Discord/Google Chat/Slack/Telegram/WhatsApp/Signal/iMessage/MS Teams.

Actions principales :

* `send` (texte + média optionnel ; MS Teams prend aussi en charge `card` pour les Adaptive Cards)
* `poll` (sondages WhatsApp/Discord/MS Teams)
* `react` / `reactions` / `read` / `edit` / `delete`
* `pin` / `unpin` / `list-pins`
* `permissions`
* `thread-create` / `thread-list` / `thread-reply`
* `search`
* `sticker`
* `member-info` / `role-info`
* `emoji-list` / `emoji-upload` / `sticker-upload`
* `role-add` / `role-remove`
* `channel-info` / `channel-list`
* `voice-status`
* `event-list` / `event-create`
* `timeout` / `kick` / `ban`

Remarques :

* `send` route WhatsApp via le Gateway ; les autres canaux sont envoyés directement.
* `poll` utilise le Gateway pour WhatsApp et MS Teams ; les sondages Discord passent directement.
* Lorsqu’un appel de l’outil `message` est lié à une session de chat active, les opérations `send` sont limitées à la cible de cette session afin d’éviter les fuites de contexte.

<div id="cron">
  ### `cron`
</div>

Gérer les tâches cron et les réveils du Gateway.

Actions principales :

* `status`, `list`
* `add`, `update`, `remove`, `run`, `runs`
* `wake` (mettre en file d’attente un événement système + signal de vie optionnel immédiat)

Notes :

* `add` attend un objet de tâche cron complet (même schéma que l’appel RPC `cron.add`).
* `update` utilise `{ id, patch }`.

<div id="gateway">
  ### `gateway`
</div>

Redémarrer le processus Gateway en cours d’exécution ou lui appliquer des mises à jour (redémarrage in-place).

Actions principales :

* `restart` (autorise l’opération + envoie `SIGUSR1` pour un redémarrage dans le même processus ; `openclaw gateway` redémarre in-place)
* `config.get` / `config.schema`
* `config.apply` (valider + écrire la configuration + redémarrer + réveiller)
* `config.patch` (fusionner une mise à jour partielle + redémarrer + réveiller)
* `update.run` (exécuter la mise à jour + redémarrer + réveiller)

Notes :

* Utilisez `delayMs` (2000 par défaut) pour éviter d’interrompre une réponse en cours.
* `restart` est désactivé par défaut ; activez-le avec `commands.restart: true`.

<div id="sessions_list-sessions_history-sessions_send-sessions_spawn-session_status">
  ### `sessions_list` / `sessions_history` / `sessions_send` / `sessions_spawn` / `session_status`
</div>

Lister les sessions, inspecter l’historique de transcription ou envoyer vers une autre session.

Paramètres principaux :

* `sessions_list` : `kinds?`, `limit?`, `activeMinutes?`, `messageLimit?` (0 = aucun)
* `sessions_history` : `sessionKey` (ou `sessionId`), `limit?`, `includeTools?`
* `sessions_send` : `sessionKey` (ou `sessionId`), `message`, `timeoutSeconds?` (0 = sans attente de réponse)
* `sessions_spawn` : `task`, `label?`, `agentId?`, `model?`, `runTimeoutSeconds?`, `cleanup?`
* `session_status` : `sessionKey?` (par défaut : session courante ; accepte `sessionId`), `model?` (`default` supprime la surcharge)

Remarques :

* `main` est la clé canonique de chat direct ; global/unknown sont masquées.
* `messageLimit > 0` récupère les N derniers messages par session (les messages d’outils sont filtrés).
* `sessions_send` attend la fin de l’exécution lorsque `timeoutSeconds > 0`.
* La livraison/l’annonce a lieu après la fin de l’exécution et est en mode best-effort ; `status: "ok"` confirme que l’exécution de l’agent est terminée, pas que l’annonce a été livrée.
* `sessions_spawn` démarre une exécution de sous-agent et publie une réponse d’annonce dans le chat du demandeur.
* `sessions_spawn` est non bloquant et retourne immédiatement `status: "accepted"`.
* `sessions_send` exécute un ping‑pong de réponses (répondre `REPLY_SKIP` pour arrêter ; nombre maximal d’allers-retours via `session.agentToAgent.maxPingPongTurns`, 0–5).
* Après le ping‑pong, l’agent cible exécute une **étape d’annonce** ; répondre `ANNOUNCE_SKIP` pour supprimer l’annonce.

<div id="agents_list">
  ### `agents_list`
</div>

Liste les identifiants d’agent que la session en cours peut cibler avec `sessions_spawn`.

Notes :

* Le résultat est restreint aux listes d’autorisation propres à chaque agent (`agents.list[].subagents.allowAgents`).
* Lorsque `["*"]` est configuré, l’outil inclut tous les agents configurés et définit `allowAny: true`.

<div id="parameters-common">
  ## Paramètres (communs)
</div>

Outils s’appuyant sur le Gateway (`canvas`, `nodes`, `cron`) :

* `gatewayUrl` (par défaut : `ws://127.0.0.1:18789`)
* `gatewayToken` (si l’authentification est activée)
* `timeoutMs`

Outil de navigateur :

* `profile` (optionnel ; par défaut : `browser.defaultProfile`)
* `target` (`sandbox` | `host` | `node`)
* `node` (optionnel ; verrouiller sur un identifiant/nom de nœud spécifique)

<div id="recommended-agent-flows">
  ## Flux d’agents recommandés
</div>

Automatisation du navigateur :

1. `browser` → `status` / `start`
2. `snapshot` (ai ou aria)
3. `act` (click/type/press)
4. `screenshot` si vous avez besoin d’une confirmation visuelle

Rendu du canvas :

1. `canvas` → `present`
2. `a2ui_push` (optionnel)
3. `snapshot`

Ciblage de nœuds :

1. `nodes` → `status`
2. `describe` sur le nœud choisi
3. `notify` / `run` / `camera_snap` / `screen_record`

<div id="safety">
  ## Sécurité
</div>

* Évitez d’utiliser directement `system.run` ; utilisez `nodes` → `run` uniquement avec le consentement explicite de l’utilisateur.
* Respectez le consentement de l’utilisateur pour la capture de la caméra ou de l’écran.
* Utilisez `status/describe` pour vérifier les autorisations avant d’invoquer des commandes multimédia.

<div id="how-tools-are-presented-to-the-agent">
  ## Comment les outils sont présentés à l&#39;agent
</div>

Les outils sont exposés dans deux canaux parallèles :

1. **Texte du prompt système** : une liste lisible par un humain + des instructions.
2. **Schéma d’outil** : les définitions de fonctions structurées envoyées à l’API du modèle.

Cela signifie que l’agent voit à la fois « quels outils existent » et « comment les appeler ». Si un outil
n’apparaît ni dans le prompt système ni dans le schéma, le modèle ne peut pas l’appeler.