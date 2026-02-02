---
title: Boucle de l'agent
summary: "Cycle de vie de la boucle de l'agent, flux et sémantique d'attente"
read_when:
  - Vous avez besoin d'une description précise de la boucle de l'agent ou des événements de son cycle de vie
---

<div id="agent-loop-openclaw">
  # Boucle d’agent (OpenClaw)
</div>

Une boucle agentique est l’exécution « réelle » complète d’un agent : intake → assemblage du contexte → inférence du modèle → exécution des outils → réponses diffusées en continu → persistance. C’est le chemin de référence qui transforme un message en actions et en réponse finale, tout en maintenant l’état de la session cohérent.

Dans OpenClaw, une boucle est une exécution unique et sérialisée par session qui émet des événements de cycle de vie et de flux au fur et à mesure que le modèle raisonne, appelle des outils et diffuse la sortie. Ce document explique comment cette boucle réelle est orchestrée de bout en bout.

<div id="entry-points">
  ## Points d&#39;entrée
</div>

* RPC du Gateway : `agent` et `agent.wait`.
* CLI : commande `agent`.

<div id="how-it-works-high-level">
  ## Fonctionnement (vue d’ensemble)
</div>

1. L’RPC `agent` valide les paramètres, résout la session (sessionKey/sessionId), persiste les métadonnées de la session, renvoie immédiatement `{ runId, acceptedAt }`.
2. `agentCommand` exécute l’agent :
   * résout le modèle + les valeurs par défaut pour thinking/verbose
   * charge un instantané des compétences
   * appelle `runEmbeddedPiAgent` (runtime pi-agent-core)
   * émet **lifecycle end/error** si la boucle embarquée n’en émet pas
3. `runEmbeddedPiAgent` :
   * sérialise les exécutions via des files d’attente par session + globales
   * résout le modèle + le profil d’authentification et construit la session pi
   * s’abonne aux événements pi et diffuse en continu les deltas assistant/tool
   * applique un délai d’expiration -&gt; annule l’exécution si celui-ci est dépassé
   * renvoie les payloads + les métadonnées d’usage
4. `subscribeEmbeddedPiSession` fait la liaison entre les événements pi-agent-core et le flux `agent` d’OpenClaw :
   * événements d’outils =&gt; `stream: "tool"`
   * deltas assistant =&gt; `stream: "assistant"`
   * événements de cycle de vie =&gt; `stream: "lifecycle"` (`phase: "start" | "end" | "error"`)
5. `agent.wait` utilise `waitForAgentJob` :
   * attend **lifecycle end/error** pour `runId`
   * renvoie `{ status: ok|error|timeout, startedAt, endedAt, error? }`

<div id="queueing-concurrency">
  ## Mise en file d&#39;attente + concurrence
</div>

* Les exécutions sont sérialisées par clé de session (session lane) et, en option, via une voie globale.
* Cela empêche les conditions de concurrence entre outils et sessions et maintient un historique de session cohérent.
* Les canaux de messagerie peuvent choisir des modes de mise en file d&#39;attente (collect/steer/followup) qui alimentent ce système de voies.
  Voir [Command Queue](/fr/concepts/queue).

<div id="session-workspace-preparation">
  ## Préparation de la session et de l’espace de travail
</div>

* L’espace de travail est déterminé et créé ; les exécutions en sandbox peuvent être redirigées vers une racine d’espace de travail de la sandbox.
* Les compétences sont chargées (ou réutilisées à partir d’un snapshot) et injectées dans `env` et dans le prompt.
* Les fichiers de bootstrap/contexte sont résolus et injectés dans le rapport du prompt système.
* Un verrou d’écriture sur la session est acquis ; `SessionManager` est ouvert et préparé avant le début du streaming.

<div id="prompt-assembly-system-prompt">
  ## Construction du prompt + prompt système
</div>

* Le prompt système est construit à partir du prompt de base d’OpenClaw, du prompt des compétences, du contexte de bootstrap et des surcharges propres à chaque exécution.
* Les limites spécifiques au modèle et les jetons réservés pour le compactage sont appliqués.
* Voir [Prompt système](/fr/concepts/system-prompt) pour ce que le modèle reçoit.

<div id="hook-points-where-you-can-intercept">
  ## Points de hook (où vous pouvez intercepter)
</div>

OpenClaw dispose de deux systèmes de hooks :

* **Hooks internes** (hooks du Gateway) : scripts déclenchés par des événements pour les commandes et le cycle de vie.
* **Hooks de plugin** : points d’extension dans le cycle de vie des agents/outils et dans le pipeline du Gateway.

<div id="internal-hooks-gateway-hooks">
  ### Hooks internes (hooks Gateway)
</div>

* **`agent:bootstrap`** : s’exécute pendant la génération des fichiers bootstrap avant la finalisation du prompt système.
  Utilisez ce hook pour ajouter ou supprimer des fichiers de contexte bootstrap.
* **Hooks de commande** : événements `/new`, `/reset`, `/stop` et autres commandes (voir la doc sur les hooks).

Voir [Hooks](/fr/hooks) pour la configuration et des exemples.

<div id="plugin-hooks-agent-gateway-lifecycle">
  ### Hooks de plugin (cycle de vie agent + Gateway)
</div>

Ces hooks s’exécutent à l’intérieur de la boucle de l’agent ou du pipeline du Gateway :

* **`before_agent_start`** : injecter du contexte ou remplacer le prompt système avant le début de l’exécution.
* **`agent_end`** : inspecter la liste finale des messages et les métadonnées d’exécution après la fin.
* **`before_compaction` / `after_compaction`** : observer ou annoter les cycles de compactage.
* **`before_tool_call` / `after_tool_call`** : intercepter les paramètres/résultats d’outil.
* **`tool_result_persist`** : transformer de manière synchrone les résultats d’outil avant qu’ils ne soient écrits dans la transcription de la session.
* **`message_received` / `message_sending` / `message_sent`** : hooks pour les messages entrants et sortants.
* **`session_start` / `session_end`** : délimitation du cycle de vie de la session.
* **`gateway_start` / `gateway_stop`** : événements de cycle de vie du Gateway.

Voir [Plugins](/fr/plugin#plugin-hooks) pour l’API des hooks et les détails d’enregistrement.

<div id="streaming-partial-replies">
  ## Streaming + réponses partielles
</div>

* Les deltas de l’assistant sont diffusés en continu depuis pi-agent-core et émis sous forme d’événements `assistant`.
* Le streaming par blocs peut produire des réponses partielles soit sur `text_end`, soit sur `message_end`.
* Le streaming du raisonnement peut être produit comme un flux séparé ou comme des réponses par blocs.
* Voir [Streaming](/fr/concepts/streaming) pour le comportement de découpage en segments et de réponses par blocs.

<div id="tool-execution-messaging-tools">
  ## Exécution des outils + outils de messagerie
</div>

* Les événements de début, de mise à jour et de fin d’outil sont émis sur le flux `tool`.
* Les résultats des outils sont nettoyés/normalisés en termes de taille et de contenu image avant journalisation/émission.
* Les envois d’outils de messagerie sont suivis afin d’éviter les confirmations en double de l’assistant.

<div id="reply-shaping-suppression">
  ## Mise en forme et suppression des réponses
</div>

* Les charges utiles finales sont assemblées à partir des éléments suivants :
  * le texte de l’assistant (et, éventuellement, son raisonnement)
  * les résumés d’outils en ligne (en mode verbeux + autorisés)
  * le texte d’erreur de l’assistant lorsque le modèle produit une erreur
* `NO_REPLY` est traité comme un jeton silencieux et filtré des charges utiles sortantes.
* Les doublons d’outils de messagerie sont supprimés de la liste finale des charges utiles.
* S’il ne reste aucune charge utile affichable et qu’un outil a échoué, une réponse de secours signalant l’erreur d’outil est émise
  (sauf si un outil de messagerie a déjà envoyé une réponse visible par l’utilisateur).

<div id="compaction-retries">
  ## Compactage et réessais
</div>

* Le compactage automatique émet des événements de flux `compaction` et peut déclencher un réessai.
* Lors d’un réessai, les tampons en mémoire et les résumés des outils sont réinitialisés pour éviter les doublons dans la sortie.
* Consultez [Compactage](/fr/concepts/compaction) pour le pipeline de compactage.

<div id="event-streams-today">
  ## Flux d’événements (aujourd’hui)
</div>

* `lifecycle` : émis par `subscribeEmbeddedPiSession` (et, en solution de repli, par `agentCommand`)
* `assistant` : deltas diffusés en continu depuis pi-agent-core
* `tool` : événements liés aux outils diffusés en continu depuis pi-agent-core

<div id="chat-channel-handling">
  ## Gestion du canal de chat
</div>

* Les deltas de l&#39;assistant sont mis en mémoire tampon sous forme de messages de chat `delta`.
* Un message de chat `final` est émis à la **fin du cycle de vie ou en cas d’erreur**.

<div id="timeouts">
  ## Délais d&#39;expiration
</div>

* Valeur par défaut de `agent.wait` : 30s (uniquement le temps d&#39;attente). Le paramètre `timeoutMs` le remplace.
* Runtime de l’Agent : `agents.defaults.timeoutSeconds` a une valeur par défaut de 600s ; appliqué dans le minuteur d&#39;annulation de `runEmbeddedPiAgent`.

<div id="where-things-can-end-early">
  ## Cas de fin anticipée
</div>

* Expiration du délai de l’Agent (abandon)
* AbortSignal (annulation)
* Déconnexion du Gateway ou expiration du délai RPC
* Expiration du délai `agent.wait` (attente uniquement, n’arrête pas l’agent)