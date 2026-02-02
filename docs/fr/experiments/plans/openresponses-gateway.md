---
title: Gateway OpenResponses
summary: "Plan : ajouter l'endpoint /v1/responses d'OpenResponses et rendre obsolètes proprement les chat completions"
owner: "openclaw"
status: "draft"
last_updated: "2026-01-19"
---

<div id="openresponses-gateway-integration-plan">
  # Plan d’intégration de la Gateway OpenResponses
</div>

<div id="context">
  ## Contexte
</div>

Le Gateway OpenClaw expose actuellement un endpoint minimal compatible avec l’API OpenAI Chat Completions à
`/v1/chat/completions` (voir [OpenAI Chat Completions](/fr/gateway/openai-http-api)).

Open Responses est un standard d’inférence ouvert basé sur l’API OpenAI Responses. Il est conçu
pour des workflows orientés agents et utilise des entrées par éléments ainsi que des événements de streaming sémantique. La spécification OpenResponses définit `/v1/responses`, et non `/v1/chat/completions`.

<div id="goals">
  ## Objectifs
</div>

* Ajouter un endpoint `/v1/responses` qui respecte la sémantique d’OpenResponses.
* Conserver Chat Completions comme couche de compatibilité, facile à désactiver puis à supprimer.
* Standardiser la validation et l’analyse avec des schémas isolés et réutilisables.

<div id="non-goals">
  ## Hors périmètre
</div>

* Atteindre une parité complète des fonctionnalités avec OpenResponses dès la première itération (images, fichiers, outils hébergés).
* Remplacer la logique interne d&#39;exécution des agents ou l&#39;orchestration des outils.
* Modifier le comportement actuel de `/v1/chat/completions` pendant la première phase.

<div id="research-summary">
  ## Synthèse de la recherche
</div>

Sources : OpenResponses OpenAPI, site de spécification OpenResponses et article de blog Hugging Face.

Points clés extraits :

* `POST /v1/responses` accepte les champs `CreateResponseBody` tels que `model`, `input` (chaîne de caractères ou
  `ItemParam[]`), `instructions`, `tools`, `tool_choice`, `stream`, `max_output_tokens` et
  `max_tool_calls`.
* `ItemParam` est une union discriminée de :
  * éléments `message` avec les rôles `system`, `developer`, `user`, `assistant`
  * `function_call` et `function_call_output`
  * `reasoning`
  * `item_reference`
* Les réponses réussies retournent un `ResponseResource` avec `object: "response"`, `status` et des
  éléments `output`.
* Le streaming utilise des événements sémantiques tels que :
  * `response.created`, `response.in_progress`, `response.completed`, `response.failed`
  * `response.output_item.added`, `response.output_item.done`
  * `response.content_part.added`, `response.content_part.done`
  * `response.output_text.delta`, `response.output_text.done`
* La spécification exige :
  * `Content-Type: text/event-stream`
  * `event:` doit correspondre au champ JSON `type`
  * l&#39;événement terminal doit être littéralement `[DONE]`
* Les éléments `reasoning` peuvent exposer `content`, `encrypted_content` et `summary`.
* Les exemples HF incluent `OpenResponses-Version: latest` dans les requêtes (en-tête optionnel).

<div id="proposed-architecture">
  ## Architecture proposée
</div>

* Ajouter `src/gateway/open-responses.schema.ts` ne contenant que des schémas Zod (aucun import du Gateway).
* Ajouter `src/gateway/openresponses-http.ts` (ou `open-responses-http.ts`) pour `/v1/responses`.
* Conserver `src/gateway/openai-http.ts` tel quel en tant qu’adaptateur de compatibilité legacy.
* Ajouter la configuration `gateway.http.endpoints.responses.enabled` (valeur par défaut `false`).
* Garder `gateway.http.endpoints.chatCompletions.enabled` indépendant ; permettre d’activer ou de désactiver chacun des deux endpoints séparément.
* Émettre un avertissement au démarrage lorsque Chat Completions est activé afin de signaler son statut obsolète.

<div id="deprecation-path-for-chat-completions">
  ## Stratégie de dépréciation pour Chat Completions
</div>

* Maintenir des limites de module strictes : aucun type de schéma partagé entre Responses et Chat Completions.
* Rendre Chat Completions activable explicitement via la configuration afin qu’il puisse être désactivé sans changement de code.
* Mettre à jour la documentation pour marquer Chat Completions comme obsolète une fois que `/v1/responses` est stable.
* Étape future optionnelle : rediriger les requêtes Chat Completions vers le gestionnaire Responses pour un chemin de retrait plus simple.

<div id="phase-1-support-subset">
  ## Sous-ensemble pris en charge pour la phase 1
</div>

* Accepter `input` comme chaîne de caractères ou `ItemParam[]` avec des rôles de message et un `function_call_output`.
* Extraire les messages système et développeur dans `extraSystemPrompt`.
* Utiliser le `user` ou `function_call_output` le plus récent comme message courant pour les exécutions d’agent.
* Rejeter les parties de contenu non prises en charge (image/fichier) avec `invalid_request_error`.
* Retourner un seul message d’assistant avec un contenu `output_text`.
* Retourner `usage` avec des valeurs nulles tant que la comptabilisation des tokens n’est pas encore implémentée.

<div id="validation-strategy-no-sdk">
  ## Stratégie de validation (sans SDK)
</div>

* Implémenter des schémas Zod pour le sous-ensemble pris en charge de :
  * `CreateResponseBody`
  * `ItemParam` + unions de parties de contenu de message
  * `ResponseResource`
  * Structures d&#39;événements de streaming utilisées par le Gateway
* Conserver les schémas dans un module unique et isolé pour éviter les divergences et permettre une génération de code ultérieure.

<div id="streaming-implementation-phase-1">
  ## Implémentation du streaming (Phase 1)
</div>

* Lignes SSE avec à la fois `event:` et `data:`.
* Séquence requise (minimum viable) :
  * `response.created`
  * `response.output_item.added`
  * `response.content_part.added`
  * `response.output_text.delta` (répété si nécessaire)
  * `response.output_text.done`
  * `response.content_part.done`
  * `response.completed`
  * `[DONE]`

<div id="tests-and-verification-plan">
  ## Plan de tests et de vérification
</div>

* Ajouter une couverture de tests e2e pour `/v1/responses` :
  * Authentification requise
  * Structure de la réponse en mode non-stream
  * Ordonnancement des événements de stream et `[DONE]`
  * Routage de session avec en-têtes et `user`
* Ne pas modifier `src/gateway/openai-http.e2e.test.ts`.
* Manuel : exécuter curl vers `/v1/responses` avec `stream: true` et vérifier l’ordonnancement des événements et le marqueur terminal
  `[DONE]`.

<div id="doc-updates-follow-up">
  ## Mises à jour de la documentation (suivi)
</div>

* Ajouter une nouvelle page de documentation décrivant l’utilisation de `/v1/responses` et fournissant des exemples.
* Mettre à jour `/gateway/openai-http-api` avec une note de dépréciation et un renvoi vers `/v1/responses`.