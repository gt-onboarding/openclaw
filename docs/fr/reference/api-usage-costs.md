---
title: Coûts d'utilisation des API
summary: "Auditer ce qui peut générer des dépenses, quelles clés sont utilisées et comment consulter l'utilisation"
read_when:
  - Vous voulez comprendre quelles fonctionnalités peuvent appeler des API payantes
  - Vous devez auditer les clés, les coûts et la visibilité sur l'utilisation
  - Vous expliquez le reporting des coûts via /status ou /usage
---

<div id="api-usage-costs">
  # Utilisation de l’API et coûts
</div>

Ce document répertorie les **fonctionnalités susceptibles d’utiliser des clés d’API** et indique où leurs coûts sont imputés. Il se concentre sur les fonctionnalités d’OpenClaw qui peuvent générer de la consommation auprès d’un fournisseur ou des appels d’API payants.

<div id="where-costs-show-up-chat-cli">
  ## Où s’affichent les coûts (chat + CLI)
</div>

**Instantané des coûts par session**

* `/status` affiche le modèle de session actuel, l’utilisation du contexte et les derniers jetons de réponse.
* Si le modèle utilise une **authentification par clé API**, `/status` affiche également le **coût estimé** de la dernière réponse.

**Pied de message avec coût par message**

* `/usage full` ajoute un pied de message d’utilisation à chaque réponse, incluant le **coût estimé** (clé API uniquement).
* `/usage tokens` affiche uniquement les jetons ; les flux OAuth masquent le coût en dollars.

**Fenêtres d’utilisation dans la CLI (quotas du fournisseur)**

* `openclaw status --usage` et `openclaw channels list` affichent les **fenêtres d’utilisation** du fournisseur
  (instantanés de quota, et non coûts par message).

Voir [Utilisation des tokens &amp; coûts](/fr/token-use) pour des détails et des exemples.

<div id="how-keys-are-discovered">
  ## Comment les clés sont détectées
</div>

OpenClaw peut récupérer des identifiants à partir de :

* **Profils d&#39;authentification** (par agent, stockés dans `auth-profiles.json`).
* **Variables d&#39;environnement** (par ex. `OPENAI_API_KEY`, `BRAVE_API_KEY`, `FIRECRAWL_API_KEY`).
* **Config** (`models.providers.*.apiKey`, `tools.web.search.*`, `tools.web.fetch.firecrawl.*`,
  `memorySearch.*`, `talk.apiKey`).
* **Compétences** (`skills.entries.<name>.apiKey`) qui peuvent exporter des clés vers l&#39;environnement du processus de la compétence.

<div id="features-that-can-spend-keys">
  ## Fonctionnalités pouvant consommer des clés
</div>

<div id="1-core-model-responses-chat-tools">
  ### 1) Réponses du modèle principal (chat + outils)
</div>

Chaque réponse ou appel d&#39;outil utilise le **fournisseur de modèles actuel** (OpenAI, Anthropic, etc.). C&#39;est la principale source de consommation et de coûts.

Voir [Modèles](/fr/providers/models) pour la configuration des tarifs et [Utilisation et coûts des jetons](/fr/token-use) pour leur affichage.

<div id="2-media-understanding-audioimagevideo">
  ### 2) Compréhension des médias (audio/image/vidéo)
</div>

Les médias entrants peuvent être résumés/transcrits avant que la réponse ne soit générée. Pour cela, des API de modèles/fournisseurs sont utilisées.

* Audio : OpenAI / Groq / Deepgram (désormais **activé automatiquement** lorsque des clés existent).
* Image : OpenAI / Anthropic / Google.
* Vidéo : Google.

Voir [Compréhension des médias](/fr/nodes/media-understanding).

<div id="3-memory-embeddings-semantic-search">
  ### 3) Embeddings de mémoire + recherche sémantique
</div>

La recherche sémantique en mémoire utilise des **API d&#39;embeddings** lorsqu&#39;elle est configurée pour des fournisseurs distants :

* `memorySearch.provider = "openai"` → embeddings OpenAI
* `memorySearch.provider = "gemini"` → embeddings Gemini
* Basculement facultatif vers OpenAI si les embeddings locaux échouent

Vous pouvez rester en local avec `memorySearch.provider = "local"` (aucune utilisation d&#39;API).

Voir [Mémoire](/fr/concepts/memory).

<div id="4-web-search-tool-brave-perplexity-via-openrouter">
  ### 4) Outil de recherche Web (Brave / Perplexity via OpenRouter)
</div>

`web_search` utilise des clés API et peut engendrer des frais d’utilisation :

* **Brave Search API** : `BRAVE_API_KEY` ou `tools.web.search.apiKey`
* **Perplexity** (via OpenRouter) : `PERPLEXITY_API_KEY` ou `OPENROUTER_API_KEY`

**Offre gratuite Brave (très généreuse) :**

* **2 000 requêtes/mois**
* **1 requête/seconde**
* **Carte bancaire requise** pour la vérification (aucun débit tant que vous ne passez pas à une offre supérieure)

Voir [Outils web](/fr/tools/web).

<div id="5-web-fetch-tool-firecrawl">
  ### 5) Outil de récupération web (Firecrawl)
</div>

`web_fetch` peut appeler **Firecrawl** lorsqu’une clé API est présente :

* `FIRECRAWL_API_KEY` ou `tools.web.fetch.firecrawl.apiKey`

Si Firecrawl n’est pas configuré, l’outil bascule sur une récupération directe + Readability (aucune API payante).

Voir [Outils web](/fr/tools/web).

<div id="6-provider-usage-snapshots-statushealth">
  ### 6) Instantanés d&#39;utilisation des fournisseurs (statut/état de santé)
</div>

Certaines commandes de statut appellent des **points de terminaison d&#39;utilisation des fournisseurs** pour afficher les fenêtres de quota ou l&#39;état de santé de l&#39;authentification.
Ce sont généralement des appels peu nombreux, mais qui atteignent tout de même les API des fournisseurs :

* `openclaw status --usage`
* `openclaw models status --json`

Voir [Models CLI](/fr/cli/models).

<div id="7-compaction-safeguard-summarization">
  ### 7) Résumé par la protection de compactage
</div>

La protection de compactage peut résumer l’historique de session en utilisant le **modèle actuel**, ce qui entraîne des appels aux API du fournisseur lors de son exécution.

Voir [Gestion des sessions et compactage](/fr/reference/session-management-compaction).

<div id="8-model-scan-probe">
  ### 8) Analyse / exploration des modèles
</div>

`openclaw models scan` peut explorer les modèles OpenRouter et utilise `OPENROUTER_API_KEY` lorsque
l’exploration est activée.

Voir [CLI des modèles](/fr/cli/models).

<div id="9-talk-speech">
  ### 9) Talk (voix)
</div>

Le mode Talk peut faire appel à **ElevenLabs** lorsqu&#39;il est configuré :

* `ELEVENLABS_API_KEY` ou `talk.apiKey`

Voir le [mode Talk](/fr/nodes/talk).

<div id="10-skills-third-party-apis">
  ### 10) Compétences (API tierces)
</div>

Les compétences peuvent stocker `apiKey` dans `skills.entries.<name>.apiKey`. Si une compétence utilise cette clé pour des API externes, elle peut engendrer des coûts en fonction du fournisseur de la compétence.

Voir [Compétences](/fr/tools/skills).