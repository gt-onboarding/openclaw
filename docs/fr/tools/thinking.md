---
title: Réflexion
summary: "Syntaxe des directives /think et /verbose et leur impact sur le raisonnement du modèle"
read_when:
  - Ajustement de l’analyse ou des valeurs par défaut des directives /think ou /verbose
---

<div id="thinking-levels-think-directives">
  # Niveaux de réflexion (/think directives)
</div>

<div id="what-it-does">
  ## Ce que cela fait
</div>

* Directive en ligne dans tout corps entrant : `/t <level>`, `/think:<level>`, ou `/thinking <level>`.
* Niveaux (alias) : `off | minimal | low | medium | high | xhigh` (modèles GPT-5.2 + Codex uniquement)
  * minimal → « think »
  * low → « think hard »
  * medium → « think harder »
  * high → « ultrathink » (budget max)
  * xhigh → « ultrathink+ » (modèles GPT-5.2 + Codex uniquement)
  * `highest`, `max` correspondent à `high`.
* Notes concernant les fournisseurs :
  * Z.AI (`zai/*`) ne prend en charge que la réflexion binaire (`on`/`off`). Tout niveau différent de `off` est traité comme `on` (assimilé à `low`).

<div id="resolution-order">
  ## Ordre de résolution
</div>

1. Directive inline dans le message (ne s’applique qu’à ce message).
2. Surcharge au niveau de la session (définie en envoyant un message contenant uniquement la directive).
3. Valeur par défaut globale (`agents.defaults.thinkingDefault` dans la configuration).
4. Valeur de repli : `low` pour les modèles capables de raisonnement ; `off` sinon.

<div id="setting-a-session-default">
  ## Définir une valeur par défaut de session
</div>

* Envoyer un message qui contient **uniquement** la directive (espaces autorisés), par exemple `/think:medium` ou `/t high`.
* Cela reste actif pour la session en cours (par expéditeur, par défaut) ; réinitialisé par `/think:off` ou à l’expiration de la session pour cause d’inactivité.
* Une réponse de confirmation est envoyée (`Thinking level set to high.` / `Thinking disabled.`). Si le niveau est invalide (par ex. `/thinking big`), la commande est rejetée avec un message d’aide et l’état de la session reste inchangé.
* Envoyer `/think` (ou `/think:`) sans argument pour afficher le niveau de réflexion actuel.

<div id="application-by-agent">
  ## Application par agent
</div>

* **Pi intégré** : le niveau résolu est transmis au runtime de l’Agent Pi s’exécutant dans le même processus.

<div id="verbose-directives-verbose-or-v">
  ## Directives de verbosité (/verbose ou /v)
</div>

* Niveaux : `on` (minimal) | `full` | `off` (valeur par défaut).
* Un message contenant uniquement la directive bascule le mode verbeux de la session et répond `Verbose logging enabled.` / `Verbose logging disabled.` ; des niveaux invalides renvoient une indication sans changer l’état.
* `/verbose off` enregistre un remplacement explicite pour la session ; supprimez-le via l’UI Sessions en choisissant `inherit`.
* Une directive en ligne n’affecte que ce message ; les valeurs par défaut de session/globales s’appliquent par ailleurs.
* Envoyez `/verbose` (ou `/verbose:`) sans argument pour voir le niveau de verbosité actuel.
* Lorsque le mode verbeux est activé, les agents qui émettent des résultats d’outils structurés (Pi, autres agents JSON) envoient chaque appel d’outil en retour sous forme de message ne contenant que des métadonnées, préfixé par `<emoji> <tool-name>: <arg>` lorsque disponible (chemin/commande). Ces résumés d’outils sont envoyés dès que chaque outil démarre (bulles séparées), et non sous forme de flux de deltas.
* Lorsque le mode verbeux est `full`, les sorties d’outils sont également retransmises après leur achèvement (bulle séparée, tronquée à une longueur sûre). Si vous basculez `/verbose on|full|off` pendant qu’une exécution est en cours, les bulles d’outils suivantes respectent le nouveau paramètre.

<div id="reasoning-visibility-reasoning">
  ## Visibilité du raisonnement (/reasoning)
</div>

* Niveaux : `on|off|stream`.
* Un message ne contenant que la directive permet d’activer ou de désactiver l’affichage des blocs de raisonnement dans les réponses.
* Quand il est activé, le raisonnement est envoyé comme un **message séparé** préfixé par `Reasoning:`.
* `stream` (Telegram uniquement) : diffuse le raisonnement dans la bulle de brouillon Telegram pendant la génération de la réponse, puis envoie la réponse finale sans raisonnement.
* Alias : `/reason`.
* Envoyez `/reasoning` (ou `/reasoning:`) sans argument pour voir le niveau de raisonnement actuel.

<div id="related">
  ## Contenu associé
</div>

* La documentation du mode Élevé est disponible dans [Mode Élevé](/fr/tools/elevated).

<div id="heartbeats">
  ## Signaux de vie
</div>

* Le corps de la sonde de signal de vie correspond à l&#39;invite de signal de vie configurée (par défaut : `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`). Les directives en ligne dans un message de signal de vie s&#39;appliquent comme d&#39;habitude (mais évitez de modifier les paramètres de session par défaut depuis les signaux de vie).
* L&#39;envoi des signaux de vie se fait par défaut uniquement via la charge utile finale. Pour envoyer également le message séparé `Reasoning:` (lorsqu&#39;il est disponible), définissez `agents.defaults.heartbeat.includeReasoning: true` ou, par agent, `agents.list[].heartbeat.includeReasoning: true`.

<div id="web-chat-ui">
  ## UI de chat web
</div>

* Le sélecteur de niveau de réflexion de l&#39;UI web reflète le niveau de session enregistré dans le store/config de session entrant lors du chargement de la page.
* Choisir un autre niveau s&#39;applique uniquement au prochain message (`thinkingOnce`) ; après l&#39;envoi, le sélecteur revient au niveau de session enregistré.
* Pour modifier la valeur par défaut de la session, envoyez une directive `/think:<level>` (comme auparavant) ; le sélecteur la reflétera après le prochain rechargement de la page.