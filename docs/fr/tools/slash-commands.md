---
title: Commandes slash
summary: "Commandes slash : texte vs natif, configuration et commandes disponibles"
read_when:
  - Utilisation ou configuration des commandes de chat
  - Débogage du routage ou des autorisations des commandes
---

<div id="slash-commands">
  # Commandes slash
</div>

Les commandes sont gérées par le Gateway. La plupart des commandes doivent être envoyées sous forme de message **indépendant** qui commence par `/`.
La commande de chat bash réservée à l’hôte utilise `! <cmd>` (avec `/bash <cmd>` comme alias).

Il existe deux systèmes associés :

* **Commandes** : messages `/...` indépendants.
* **Directives** : `/think`, `/verbose`, `/reasoning`, `/elevated`, `/exec`, `/model`, `/queue`.
  * Les directives sont retirées du message avant que le modèle ne le voie.
  * Dans les messages de chat normaux (non composés uniquement de directives), elles sont traitées comme des « indications intégrées » et **ne** conservent pas les paramètres de session.
  * Dans les messages composés uniquement de directives (le message ne contient que des directives), elles sont conservées dans la session et une réponse d’accusé de réception est renvoyée.
  * Les directives ne sont appliquées que pour les **expéditeurs autorisés** (liste d’autorisation/appairage de canal plus `commands.useAccessGroups`).
    Les expéditeurs non autorisés voient les directives traitées comme du texte brut.

Il existe aussi quelques **raccourcis en ligne** (uniquement pour les expéditeurs autorisés/présents dans une liste d’autorisation) : `/help`, `/commands`, `/status`, `/whoami` (`/id`).
Ils s’exécutent immédiatement, sont retirés avant que le modèle ne voie le message, et le texte restant suit ensuite le flux normal.

<div id="config">
  ## Configuration
</div>

```json5
{
  commands: {
    native: "auto",
    nativeSkills: "auto",
    text: true,
    bash: false,
    bashForegroundMs: 2000,
    config: false,
    debug: false,
    restart: false,
    useAccessGroups: true
  }
}
```

* `commands.text` (par défaut `true`) active l’analyse de `/...` dans les messages de chat.
  * Sur les interfaces sans commandes natives (WhatsApp/WebChat/Signal/iMessage/Google Chat/MS Teams), les commandes textuelles continuent de fonctionner même si vous définissez cette option sur `false`.
* `commands.native` (par défaut `"auto"`) enregistre les commandes natives.
  * Auto : activé pour Discord/Telegram ; désactivé pour Slack (jusqu’à ce que vous ajoutiez des commandes slash) ; ignoré pour les fournisseurs sans prise en charge native.
  * Définissez `channels.discord.commands.native`, `channels.telegram.commands.native` ou `channels.slack.commands.native` pour remplacer ce comportement par fournisseur (booléen ou `"auto"`).
  * `false` supprime les commandes précédemment enregistrées sur Discord/Telegram au démarrage. Les commandes Slack sont gérées dans l’app Slack et ne sont pas supprimées automatiquement.
* `commands.nativeSkills` (par défaut `"auto"`) enregistre les commandes de **compétence** nativement lorsqu’elles sont prises en charge.
  * Auto : activé pour Discord/Telegram ; désactivé pour Slack (Slack exige la création d’une commande slash par compétence).
  * Définissez `channels.discord.commands.nativeSkills`, `channels.telegram.commands.nativeSkills` ou `channels.slack.commands.nativeSkills` pour remplacer ce comportement par fournisseur (booléen ou `"auto"`).
* `commands.bash` (par défaut `false`) active `! <cmd>` pour exécuter des commandes shell de l’hôte (`/bash <cmd>` est un alias ; nécessite des listes d’autorisation `tools.elevated`).
* `commands.bashForegroundMs` (par défaut `2000`) contrôle la durée pendant laquelle bash attend avant de basculer en mode arrière-plan (`0` passe immédiatement en arrière-plan).
* `commands.config` (par défaut `false`) active `/config` (lecture/écriture de `openclaw.json`).
* `commands.debug` (par défaut `false`) active `/debug` (surcharges valables uniquement pendant l’exécution).
* `commands.useAccessGroups` (par défaut `true`) applique les listes d’autorisation et les politiques associées aux commandes.

<div id="command-list">
  ## Liste des commandes
</div>

Texte + natif (lorsqu’activé) :

* `/help`
* `/commands`
* `/skill <name> [input]` (exécuter une skill par nom)
* `/status` (afficher l’état actuel ; inclut l’utilisation/le quota du fournisseur de modèles actuel lorsque disponible)
* `/allowlist` (lister/ajouter/supprimer des entrées de liste d’autorisation)
* `/approve <id> allow-once|allow-always|deny` (résoudre les demandes d’approbation d’exécution)
* `/context [list|detail|json]` (expliquer le « context » ; `detail` affiche la taille par fichier + par outil + par skill + du prompt système)
* `/whoami` (afficher votre identifiant d’expéditeur ; alias : `/id`)
* `/subagents list|stop|log|info|send` (inspecter, arrêter, journaliser ou envoyer des messages aux exécutions de sous-agents pour la session en cours)
* `/config show|get|set|unset` (enregistrer la configuration sur le disque, propriétaire uniquement ; nécessite `commands.config: true`)
* `/debug show|set|unset|reset` (surcharges à l’exécution, propriétaire uniquement ; nécessite `commands.debug: true`)
* `/usage off|tokens|full|cost` (pied de page d’utilisation par réponse ou résumé local des coûts)
* `/tts off|always|inbound|tagged|status|provider|limit|summary|audio` (contrôler le TTS ; voir [/tts](/fr/tts))
  * Discord : la commande native est `/voice` (Discord réserve `/tts`) ; la commande texte `/tts` fonctionne toujours.
* `/stop`
* `/restart`
* `/dock-telegram` (alias : `/dock_telegram`) (basculer les réponses vers Telegram)
* `/dock-discord` (alias : `/dock_discord`) (basculer les réponses vers Discord)
* `/dock-slack` (alias : `/dock_slack`) (basculer les réponses vers Slack)
* `/activation mention|always` (groupes uniquement)
* `/send on|off|inherit` (propriétaire uniquement)
* `/reset` ou `/new [model]` (indication de modèle optionnelle ; le reste est transmis tel quel)
* `/think <off|minimal|low|medium|high|xhigh>` (choix dynamiques selon le modèle/fournisseur ; alias : `/thinking`, `/t`)
* `/verbose on|full|off` (alias : `/v`)
* `/reasoning on|off|stream` (alias : `/reason` ; lorsqu’activé, envoie un message séparé préfixé par `Reasoning:` ; `stream` = brouillon Telegram uniquement)
* `/elevated on|off|ask|full` (alias : `/elev` ; `full` ignore les approbations d’exécution)
* `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>` (envoyer `/exec` pour afficher la configuration actuelle)
* `/model <name>` (alias : `/models` ; ou `/<alias>` depuis `agents.defaults.models.*.alias`)
* `/queue <mode>` (plus des options comme `debounce:2s cap:25 drop:summarize` ; envoyer `/queue` pour voir les paramètres actuels)
* `/bash <command>` (hôte uniquement ; alias de `! <command>` ; nécessite `commands.bash: true` + listes d’autorisation `tools.elevated`)

Texte uniquement :

* `/compact [instructions]` (voir [/concepts/compaction](/fr/concepts/compaction))
* `! <command>` (hôte uniquement ; une à la fois ; utiliser `!poll` + `!stop` pour les tâches de longue durée)
* `!poll` (vérifier la sortie/l’état ; accepte un `sessionId` optionnel ; `/bash poll` fonctionne aussi)
* `!stop` (arrêter la tâche bash en cours ; accepte un `sessionId` optionnel ; `/bash stop` fonctionne aussi)

Notes :

* Les commandes acceptent un `:` facultatif entre la commande et les arguments (par ex. `/think: high`, `/send: on`, `/help:`).
* `/new <model>` accepte un alias de modèle, `provider/model`, ou un nom de fournisseur (correspondance approximative) ; en l’absence de correspondance, le texte est traité comme le corps du message.
* Pour une répartition complète de l’utilisation par fournisseur, utilisez `openclaw status --usage`.
* `/allowlist add|remove` nécessite `commands.config=true` et respecte la configuration `configWrites` du canal.
* `/usage` contrôle le pied de page d’utilisation par réponse ; `/usage cost` affiche un récapitulatif local des coûts à partir des journaux de session OpenClaw.
* `/restart` est désactivé par défaut ; définissez `commands.restart: true` pour l’activer.
* `/verbose` est destiné au débogage et à une visibilité accrue ; laissez-le **désactivé** dans un usage normal.
* `/reasoning` (et `/verbose`) sont risqués dans les contextes de groupe : ils peuvent révéler un raisonnement interne ou la sortie d’outils que vous ne souhaitiez pas exposer. Il est préférable de les laisser désactivés, en particulier dans les conversations de groupe.
* **Voie rapide :** les messages ne contenant qu’une commande provenant d’expéditeurs figurant sur la liste d’autorisation sont traités immédiatement (contournement de la file d’attente + du modèle).
* **Contrôle par mention en groupe :** les messages ne contenant qu’une commande provenant d’expéditeurs figurant sur la liste d’autorisation contournent les exigences de mention.
* **Raccourcis en ligne (expéditeurs figurant sur la liste d’autorisation uniquement) :** certaines commandes fonctionnent également lorsqu’elles sont intégrées dans un message normal et sont retirées avant que le modèle voie le texte restant.
  * Exemple : `hey /status` déclenche une réponse d’état, et le texte restant continue via le flux normal.
* Actuellement : `/help`, `/commands`, `/status`, `/whoami` (`/id`).
* Les messages non autorisés ne contenant qu’une commande sont ignorés silencieusement, et les tokens `/...` en ligne sont traités comme du texte brut.
* **Commandes de compétences :** les compétences `user-invocable` sont exposées en tant que commandes slash. Les noms sont normalisés en `a-z0-9_` (32 caractères max) ; les collisions reçoivent un suffixe numérique (par ex. `_2`).
  * `/skill <name> [input]` exécute une compétence par son nom (utile lorsque les limites de commandes natives empêchent d’avoir une commande par compétence).
  * Par défaut, les commandes de compétences sont transmises au modèle comme une requête normale.
  * Les compétences peuvent, en option, déclarer `command-dispatch: tool` pour router la commande directement vers un outil (déterministe, sans modèle).
  * Exemple : `/prose` (plugin OpenProse) — voir [OpenProse](/fr/prose).
* **Arguments des commandes natives :** Discord utilise l’auto-complétion pour les options dynamiques (et des menus de boutons lorsque vous omettez des arguments obligatoires). Telegram et Slack affichent un menu de boutons lorsqu’une commande prend en charge des choix et que vous omettez l’argument.

<div id="usage-surfaces-what-shows-where">
  ## Surfaces d’usage (ce qui s’affiche où)
</div>

* **Utilisation/quota du fournisseur** (exemple : « Claude 80% left ») apparaît dans `/status` pour le fournisseur de modèle actuel lorsque le suivi de l’utilisation est activé.
* **Jetons/coût par réponse** sont contrôlés par `/usage off|tokens|full` (ajouté aux réponses normales).
* `/model status` concerne les **modèles/auth/endpoints**, pas l’utilisation.

<div id="model-selection-model">
  ## Sélection du modèle (`/model`)
</div>

`/model` est implémenté comme une directive.

Exemples :

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model opus@anthropic:default
/model status
```

Notes :

* `/model` et `/model list` affichent un sélecteur compact et numéroté (famille de modèles + fournisseurs disponibles).
* `/model <#>` sélectionne un élément à partir de ce sélecteur (et privilégie le fournisseur actuel si possible).
* `/model status` affiche la vue détaillée, y compris le point de terminaison (`baseUrl`) du fournisseur configuré et le mode API (`api`) lorsqu’ils sont disponibles.

<div id="debug-overrides">
  ## Substitutions de débogage
</div>

`/debug` vous permet de définir des substitutions de configuration **uniquement à l’exécution** (en mémoire, pas sur disque). Réservé au propriétaire. Désactivé par défaut ; activez-le avec `commands.debug: true`.

Exemples :

```
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug set channels.whatsapp.allowFrom=["+1555","+4477"]
/debug unset messages.responsePrefix
/debug reset
```

Remarques :

* Les surcharges s’appliquent immédiatement aux nouvelles lectures de configuration, mais **n’écrivent pas** dans `openclaw.json`.
* Utilisez `/debug reset` pour effacer toutes les surcharges et revenir à la configuration sur le disque.

<div id="config-updates">
  ## Mises à jour de configuration
</div>

`/config` met à jour votre fichier de configuration sur le disque (`openclaw.json`). Commande réservée au propriétaire. Désactivée par défaut ; activez-la avec `commands.config: true`.

Exemples :

```
/config show
/config show messages.responsePrefix
/config get messages.responsePrefix
/config set messages.responsePrefix="[openclaw]"
/config unset messages.responsePrefix
```

Notes :

* La configuration est validée avant d’être enregistrée ; les modifications invalides sont rejetées.
* Les mises à jour de `/config` sont conservées après les redémarrages.

<div id="surface-notes">
  ## Notes d’interface
</div>

* Les **commandes texte** s’exécutent dans la session de chat normale (les DMs partagent `main`, les groupes ont chacun leur propre session).
* Les **commandes natives** utilisent des sessions isolées :
  * Discord : `agent:<agentId>:discord:slash:<userId>`
  * Slack : `agent:<agentId>:slack:slash:<userId>` (préfixe configurable via `channels.slack.slashCommand.sessionPrefix`)
  * Telegram : `telegram:slash:<userId>` (cible la session de chat via `CommandTargetSessionKey`)
* **`/stop`** cible la session de chat active afin de pouvoir interrompre l’exécution en cours.
* **Slack :** `channels.slack.slashCommand` est toujours pris en charge pour une commande unique de type `/openclaw`. Si vous activez `commands.native`, vous devez créer une commande slash Slack par commande intégrée (mêmes noms que `/help`). Les menus d’arguments de commande pour Slack sont fournis sous forme de boutons Block Kit éphémères.