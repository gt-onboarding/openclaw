---
title: Tâches Cron
summary: "Tâches Cron + réveils pour le planificateur du Gateway"
read_when:
  - Planifier des tâches d'arrière-plan ou des réveils
  - Mettre en place des automatisations devant s'exécuter avec ou en parallèle des signaux de vie
  - Choisir entre signal de vie et tâches Cron pour des tâches planifiées
---

<div id="cron-jobs-gateway-scheduler">
  # Tâches cron (planificateur du Gateway)
</div>

> **Cron ou signal de vie ?** Consultez la page [Cron vs Heartbeat](/fr/automation/cron-vs-heartbeat) pour savoir quand utiliser chaque mécanisme.

Cron est le planificateur intégré du Gateway. Il conserve les tâches, réveille l’agent au bon moment et peut, en option, renvoyer le résultat vers une discussion.

Si vous voulez *« exécuter ceci tous les matins »* ou *« déclencher l’agent dans 20 minutes »*,
cron est le mécanisme adapté.

<div id="tldr">
  ## TL;DR
</div>

* Cron s’exécute **à l’intérieur du Gateway** (et non à l’intérieur du modèle).
* Les tâches sont stockées sous `~/.openclaw/cron/`, de sorte que les redémarrages ne perdent pas les planifications.
* Deux modes d’exécution :
  * **Session principale** : met en file un événement système, puis s’exécute au prochain signal de vie.
  * **Isolée** : exécute un tour dédié d’agent dans `cron:<jobId>`, et livre éventuellement la sortie.
* Les réveils sont des primitives de premier ordre : une tâche peut demander un « réveil immédiat » plutôt qu’« au prochain signal de vie ».

<div id="beginner-friendly-overview">
  ## Aperçu pour débutants
</div>

Pense à un cron job comme à : **quand** l’exécuter + **quoi** faire.

1. **Choisir une planification**
   * Rappel ponctuel → `schedule.kind = "at"` (CLI : `--at`)
   * Tâche récurrente → `schedule.kind = "every"` ou `schedule.kind = "cron"`
   * Si ton horodatage ISO n’indique pas de fuseau horaire, il est interprété comme **UTC**.

2. **Choisir où il s’exécute**
   * `sessionTarget: "main"` → exécution lors du prochain signal de vie avec le contexte principal.
   * `sessionTarget: "isolated"` → exécute un tour d’agent dédié dans `cron:<jobId>`.

3. **Choisir le payload**
   * Session principale → `payload.kind = "systemEvent"`
   * Session isolée → `payload.kind = "agentTurn"`

Optionnel : `deleteAfterRun: true` supprime du stockage les tâches ponctuelles réussies.

<div id="concepts">
  ## Concepts clés
</div>

<div id="jobs">
  ### Tâches
</div>

Une tâche cron est un enregistrement persistant qui contient :

* une **planification** (quand elle doit s&#39;exécuter),
* une **charge utile** (ce qu&#39;elle doit faire),
* un **acheminement** optionnel (où la sortie doit être envoyée),
* un **rattachement d&#39;agent** optionnel (`agentId`) : exécuter la tâche sous un agent spécifique ; si
  ce rattachement est absent ou inconnu, le Gateway revient à l&#39;agent par défaut.

Les tâches sont identifiées par un `jobId` stable (utilisé par les API CLI/Gateway).
Dans les appels d’outils d’agent, `jobId` est canonique ; l’ancien `id` est accepté pour des raisons de compatibilité.
Les tâches peuvent être automatiquement supprimées après une exécution unique réussie via `deleteAfterRun: true`.

<div id="schedules">
  ### Planifications
</div>

Cron prend en charge trois types de planification :

* `at` : horodatage ponctuel (ms depuis l’époque Unix). Gateway accepte les horodatages ISO 8601 et les convertit en UTC.
* `every` : intervalle fixe (ms).
* `cron` : expression cron à 5 champs avec fuseau horaire IANA optionnel.

Les expressions cron utilisent `croner`. Si un fuseau horaire est omis, le fuseau
horaire local de l’hôte Gateway est utilisé.

<div id="main-vs-isolated-execution">
  ### Exécution principale ou isolée
</div>

<div id="main-session-jobs-system-events">
  #### Tâches de la session principale (événements système)
</div>

Les tâches principales placent un événement système dans la file d&#39;attente et peuvent éventuellement réveiller le processus de signal de vie.
Elles doivent utiliser `payload.kind = "systemEvent"`.

* `wakeMode: "next-heartbeat"` (par défaut) : l&#39;événement attend le prochain signal de vie planifié.
* `wakeMode: "now"` : l&#39;événement déclenche immédiatement une exécution du signal de vie.

C&#39;est l&#39;option la mieux adaptée lorsque vous souhaitez utiliser l&#39;invite de signal de vie normale, avec le contexte de la session principale.
Voir [Heartbeat](/fr/gateway/heartbeat).

<div id="isolated-jobs-dedicated-cron-sessions">
  #### Tâches isolées (sessions cron dédiées)
</div>

Les tâches isolées exécutent un tour d’agent dédié dans la session `cron:<jobId>`.

Comportements clés :

* Le prompt est préfixé par `[cron:<jobId> <job name>]` pour la traçabilité.
* Chaque exécution démarre avec un **nouvel identifiant de session** (aucun historique de conversation repris).
* Un récapitulatif est publié dans la session principale (préfixe `Cron`, configurable).
* `wakeMode: "now"` déclenche un signal de vie immédiat après la publication du récapitulatif.
* Si `payload.deliver: true`, la sortie est envoyée vers un canal ; sinon, elle reste interne.

Utilisez des tâches isolées pour les tâches bruyantes, fréquentes ou « de fond » qui ne doivent pas inonder
l’historique principal de votre chat.

<div id="payload-shapes-what-runs">
  ### Structures de payload (ce qui s’exécute)
</div>

Deux types de payload sont pris en charge :

* `systemEvent` : session principale uniquement, acheminé via le prompt de signal de vie.
* `agentTurn` : session isolée uniquement, exécute un tour d’agent dédié.

Champs courants de `agentTurn` :

* `message` : prompt textuel obligatoire.
* `model` / `thinking` : remplacements facultatifs (voir ci-dessous).
* `timeoutSeconds` : remplacement facultatif du délai d’expiration.
* `deliver` : `true` pour envoyer la sortie vers une cible de canal.
* `channel` : `last` ou un canal spécifique.
* `to` : cible spécifique au canal (téléphone/chat/ID de canal).
* `bestEffortDeliver` : éviter l’échec de la tâche si la remise échoue.

Options d’isolation (uniquement pour `session=isolated`) :

* `postToMainPrefix` (CLI : `--post-prefix`) : préfixe pour l’événement système dans la session principale.
* `postToMainMode` : `summary` (par défaut) ou `full`.
* `postToMainMaxChars` : nombre maximal de caractères lorsque `postToMainMode=full` (8000 par défaut).

<div id="model-and-thinking-overrides">
  ### Surcharges de modèle et de niveau de réflexion
</div>

Les tâches isolées (`agentTurn`) peuvent surcharger le modèle et le niveau de réflexion :

* `model` : chaîne fournisseur/modèle (par exemple, `anthropic/claude-sonnet-4-20250514`) ou alias (par exemple, `opus`)
* `thinking` : niveau de réflexion (`off`, `minimal`, `low`, `medium`, `high`, `xhigh` ; uniquement pour les modèles GPT-5.2 + Codex)

Remarque : vous pouvez aussi définir `model` sur les tâches de session principale, mais cela modifie le modèle principal de la session partagée. Nous recommandons de n’utiliser les surcharges de modèle que pour les tâches isolées afin d’éviter des changements de contexte inattendus.

Priorité de résolution :

1. Surcharge dans la charge utile de la tâche (priorité la plus élevée)
2. Valeurs par défaut spécifiques au hook (par exemple, `hooks.gmail.model`)
3. Valeur par défaut de la configuration d’agent

<div id="delivery-channel-target">
  ### Livraison (canal + cible)
</div>

Des jobs isolés peuvent envoyer la sortie vers un canal. Le payload du job peut spécifier :

* `channel`: `whatsapp` / `telegram` / `discord` / `slack` / `mattermost` (plugin) / `signal` / `imessage` / `last`
* `to`: destinataire spécifique au canal

Si `channel` ou `to` est omis, cron peut se replier sur la « dernière route » de la session principale
(le dernier endroit où l’agent a répondu).

Notes sur la livraison :

* Si `to` est défini, cron envoie automatiquement la sortie finale de l’agent même si `deliver` est omis.
* Utilisez `deliver: true` si vous voulez une livraison via la dernière route sans `to` explicite.
* Utilisez `deliver: false` pour garder la sortie interne même si un `to` est présent.

Rappels sur le format des cibles :

* Les cibles Slack/Discord/Mattermost (plugin) doivent utiliser des préfixes explicites (par ex. `channel:<id>`, `user:<id>`) pour éviter toute ambiguïté.
* Les topics Telegram doivent utiliser la forme `:topic:` (voir ci-dessous).

<div id="telegram-delivery-targets-topics-forum-threads">
  #### Cibles de livraison Telegram (sujets / fils de forum)
</div>

Telegram prend en charge les sujets de forum via `message_thread_id`. Pour les envois planifiés par cron, vous pouvez encoder
le sujet/fil dans le champ `to` :

* `-1001234567890` (ID du chat uniquement)
* `-1001234567890:topic:123` (recommandé : marqueur de sujet explicite)
* `-1001234567890:123` (forme abrégée : suffixe numérique)

Les cibles préfixées comme `telegram:...` / `telegram:group:...` sont également acceptées :

* `telegram:group:-1001234567890:topic:123`

<div id="storage-history">
  ## Stockage et historique
</div>

* Stockage des tâches : `~/.openclaw/cron/jobs.json` (JSON géré par le Gateway).
* Historique des exécutions : `~/.openclaw/cron/runs/<jobId>.jsonl` (JSONL, nettoyé automatiquement).
* Chemin de stockage personnalisé : `cron.store` dans la configuration.

<div id="configuration">
  ## Configuration
</div>

```json5
{
  cron: {
    enabled: true, // true par défaut
    store: "~/.openclaw/cron/jobs.json",
    maxConcurrentRuns: 1 // 1 par défaut
  }
}
```

Désactivez complètement cron :

* `cron.enabled: false` (config)
* `OPENCLAW_SKIP_CRON=1` (env)

<div id="cli-quickstart">
  ## Démarrage rapide en CLI
</div>

Rappel unique (UTC au format ISO, suppression automatique après réussite) :

```bash
openclaw cron add \
  --name "Send reminder" \
  --at "2026-01-12T18:00:00Z" \
  --session main \
  --system-event "Reminder: submit expense report." \
  --wake now \
  --delete-after-run
```

Rappel unique (session principale, réveil immédiat) :

```bash
openclaw cron add \
  --name "Calendar check" \
  --at "20m" \
  --session main \
  --system-event "Next heartbeat: check calendar." \
  --wake now
```

Tâche récurrente isolée (envoi vers WhatsApp) :

```bash
openclaw cron add \
  --name "Morning status" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize inbox + calendar for today." \
  --deliver \
  --channel whatsapp \
  --to "+15551234567"
```

Tâche isolée récurrente (envoi vers un topic Telegram) :

```bash
openclaw cron add \
  --name "Nightly summary (topic)" \
  --cron "0 22 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize today; send to the nightly topic." \
  --deliver \
  --channel telegram \
  --to "-1001234567890:topic:123"
```

Job isolé avec remplacement du modèle et du raisonnement :

````bash
openclaw cron add \
  --name "Analyse approfondie" \
  --cron "0 6 * * 1" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Analyse hebdomadaire approfondie de l'avancement du projet." \
  --model "opus" \
  --thinking high \
  --deliver \
  --channel whatsapp \
  --to "+15551234567"

Sélection d'agent (configurations multi-agents) :
```bash
# Épingler une tâche à l'agent "ops" (repli sur l'agent par défaut si cet agent est absent)
openclaw cron add --name "Balayage ops" --cron "0 6 * * *" --session isolated --message "Vérifier la file d'attente ops" --agent ops

# Changer ou supprimer l'agent sur une tâche existante
openclaw cron edit <jobId> --agent ops
openclaw cron edit <jobId> --clear-agent
````

````

Exécution manuelle (debug) :
```bash
openclaw cron run <jobId> --force
````

Modifier une tâche existante (mise à jour partielle des champs) :

```bash
openclaw cron edit <jobId> \
  --message "Updated prompt" \
  --model "opus" \
  --thinking low
```

Historique des exécutions :

```bash
openclaw cron runs --id <jobId> --limit 50
```

Événement système immédiat sans créer de tâche :

```bash
openclaw system event --mode now --text "Next heartbeat: check battery."
```

<div id="gateway-api-surface">
  ## Interface de l&#39;API du Gateway
</div>

* `cron.list`, `cron.status`, `cron.add`, `cron.update`, `cron.remove`
* `cron.run` (forcée ou arrivée à échéance), `cron.runs`
  Pour des événements système immédiats sans tâche planifiée, utilisez [`openclaw system event`](/fr/cli/system).

<div id="troubleshooting">
  ## Résolution des problèmes
</div>

<div id="nothing-runs">
  ### « Rien ne s’exécute »
</div>

* Vérifiez que cron est activé : `cron.enabled` et `OPENCLAW_SKIP_CRON`.
* Vérifiez que Gateway s’exécute en continu (les tâches cron s’exécutent à l’intérieur du processus Gateway).
* Pour les planifications `cron` : confirmez le fuseau horaire (`--tz`) par rapport au fuseau horaire de l’hôte.

<div id="telegram-delivers-to-the-wrong-place">
  ### Telegram envoie au mauvais endroit
</div>

* Pour les sujets de forum, utilisez `-100…:topic:<id>` afin que ce soit explicite et non ambigu.
* Si vous voyez des préfixes `telegram:...` dans les journaux ou dans les cibles « last route » enregistrées, c’est normal ; la livraison effectuée par cron les accepte et continue à analyser correctement les IDs de sujet.