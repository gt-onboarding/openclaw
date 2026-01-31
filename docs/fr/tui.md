---
title: TUI
summary: "UI de terminal (TUI) : connectez-vous au Gateway depuis n'importe quelle machine"
read_when:
  - Vous voulez un guide pas à pas pour débuter avec la TUI
  - Vous avez besoin de la liste complète des fonctionnalités, commandes et raccourcis de la TUI
---

<div id="tui-terminal-ui">
  # TUI (UI en mode terminal)
</div>

<div id="quick-start">
  ## Démarrage rapide
</div>

1. Démarrez le Gateway.

```bash
openclaw gateway
```

2. Ouvrez l’interface TUI.

```bash
openclaw tui
```

3. Tapez un message et appuyez sur Entrée.

Gateway distant :

```bash
openclaw tui --url ws://<host>:<port> --token <gateway-token>
```

Utilisez `--password` si votre Gateway utilise l’authentification par mot de passe.

<div id="what-you-see">
  ## Ce que vous voyez
</div>

* En-tête : URL de connexion, agent actuel, session actuelle.
* Journal de chat : messages de l’utilisateur, réponses de l’assistant, notifications système, cartes d’outils.
* Ligne d’état : état de connexion/exécution (connexion, en cours d’exécution, streaming, inactif, erreur).
* Pied de page : état de connexion + agent + session + modèle + think/verbose/reasoning + compteurs de tokens + deliver.
* Saisie : éditeur de texte avec autocomplétion.

<div id="mental-model-agents-sessions">
  ## Modèle mental : agents + sessions
</div>

* Les agents sont des identifiants uniques (slugs) (par ex. `main`, `research`). Le Gateway expose la liste.
* Les sessions appartiennent à l’agent actuel.
* Les clés de session sont stockées sous la forme `agent:<agentId>:<sessionKey>`.
  * Si vous saisissez `/session main`, la TUI le développe en `agent:<currentAgent>:main`.
  * Si vous saisissez `/session agent:other:main`, vous basculez explicitement vers cette session d’agent.
* Portée des sessions :
  * `per-sender` (par défaut) : chaque agent possède plusieurs sessions.
  * `global` : la TUI utilise toujours la session `global` (le sélecteur peut être vide).
* L’agent et la session actuels sont toujours visibles dans le pied de page.

<div id="sending-delivery">
  ## Envoi + acheminement
</div>

* Les messages sont envoyés au Gateway ; l’acheminement vers les fournisseurs est désactivé par défaut.
* Activer l’acheminement :
  * `/deliver on`
  * ou via le panneau Settings
  * ou démarrer avec `openclaw tui --deliver`

<div id="pickers-overlays">
  ## Sélecteurs + superpositions
</div>

* Sélecteur de modèle : liste les modèles disponibles et définit le remplacement pour la session.
* Sélecteur d’agent : permet de choisir un autre agent.
* Sélecteur de session : affiche uniquement les sessions de l’agent actuel.
* Paramètres : active/désactive l’envoi, l’affichage détaillé de la sortie des outils et l’affichage du raisonnement.

<div id="keyboard-shortcuts">
  ## Raccourcis clavier
</div>

* Enter : envoyer le message
* Esc : interrompre l&#39;exécution en cours
* Ctrl+C : effacer la saisie (appuyez deux fois pour quitter)
* Ctrl+D : quitter
* Ctrl+L : sélecteur de modèle
* Ctrl+G : sélecteur d&#39;agent
* Ctrl+P : sélecteur de session
* Ctrl+O : activer/désactiver l&#39;affichage détaillé de la sortie des outils
* Ctrl+T : afficher/masquer le raisonnement (recharge l&#39;historique)

<div id="slash-commands">
  ## Commandes slash
</div>

Commandes de base :

* `/help`
* `/status`
* `/agent <id>` (ou `/agents`)
* `/session <key>` (ou `/sessions`)
* `/model <provider/model>` (ou `/models`)

Contrôles de session :

* `/think <off|minimal|low|medium|high>`
* `/verbose <on|full|off>`
* `/reasoning <on|off|stream>`
* `/usage <off|tokens|full>`
* `/elevated <on|off|ask|full>` (alias : `/elev`)
* `/activation <mention|always>`
* `/deliver <on|off>`

Cycle de vie de la session :

* `/new` ou `/reset` (réinitialise la session)
* `/abort` (interrompt l&#39;exécution en cours)
* `/settings`
* `/exit`

Les autres commandes slash du Gateway (par exemple, `/context`) sont relayées au Gateway et affichées en sortie système. Voir [Commandes slash](/fr/tools/slash-commands).

<div id="local-shell-commands">
  ## Commandes shell locales
</div>

* Préfixez une ligne par `!` pour exécuter une commande shell locale sur l’hôte TUI.
* La TUI vous invite une fois par session à autoriser l’exécution locale ; si vous refusez, `!` reste désactivé pour la session.
* Les commandes s’exécutent dans un nouveau shell non interactif, dans le répertoire de travail de la TUI (aucun `cd`/env persistant).
* Un `!` seul est envoyé comme message normal ; les espaces en début de ligne ne déclenchent pas l’exécution locale.

<div id="tool-output">
  ## Sortie des outils
</div>

* Les appels d’outils s’affichent sous forme de cartes contenant les arguments et les résultats.
* Ctrl+O bascule entre la vue réduite et la vue développée.
* Pendant l’exécution d’un outil, les mises à jour partielles sont diffusées en continu dans la même carte.

<div id="history-streaming">
  ## Historique + streaming
</div>

* Lors de la connexion, la TUI charge l&#39;historique le plus récent (200 messages par défaut).
* Les réponses en streaming sont mises à jour en place jusqu&#39;à leur finalisation.
* La TUI écoute également les événements d&#39;outils de l&#39;agent pour des fiches d&#39;outils plus riches.

<div id="connection-details">
  ## Détails de connexion
</div>

* La TUI s’enregistre auprès du Gateway avec `mode: "tui"`.
* Lors d’une reconnexion, un message système s’affiche ; les interruptions dans le flux d’événements sont signalées dans le journal.

<div id="options">
  ## Options
</div>

* `--url <url>`: URL WS du Gateway (par défaut selon la configuration ou `ws://127.0.0.1:<port>`)
* `--token <token>`: Jeton du Gateway (si nécessaire)
* `--password <password>`: Mot de passe du Gateway (si nécessaire)
* `--session <key>`: Clé de session (par défaut : `main`, ou `global` lorsque la portée est globale)
* `--deliver`: Acheminer les réponses de l&#39;assistant vers le fournisseur (désactivé par défaut)
* `--thinking <level>`: Remplacer le niveau de réflexion pour `send`
* `--timeout-ms <ms>`: Délai d&#39;expiration de l’agent en ms (par défaut : `agents.defaults.timeoutSeconds`)

<div id="troubleshooting">
  ## Dépannage
</div>

Aucune réponse après l’envoi d’un message :

* Exécutez `/status` dans la TUI pour confirmer que le Gateway est connecté et inactif/occupé.
* Vérifiez les journaux du Gateway : `openclaw logs --follow`.
* Confirmez que l’agent peut s’exécuter : `openclaw status` et `openclaw models status`.
* Si vous vous attendez à recevoir des messages dans un canal de discussion, activez la remise (`/deliver on` ou `--deliver`).
* `--history-limit <n>` : nombre d’entrées d’historique à charger (200 par défaut)

<div id="troubleshooting">
  ## Dépannage
</div>

* `disconnected` : assurez-vous que le service Gateway est en cours d’exécution et que vos options `--url/--token/--password` sont correctes.
* Aucun agent dans le sélecteur : vérifiez `openclaw agents list` et votre configuration de routage.
* Sélecteur de session vide : vous êtes peut-être dans la portée globale ou vous n’avez pas encore de sessions.