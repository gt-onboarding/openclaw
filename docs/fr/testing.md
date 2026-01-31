---
title: Tests
summary: "Kit de tests : suites unitaires/e2e/live, exécuteurs Docker et couverture de chaque test"
read_when:
  - Exécuter des tests en local ou en CI
  - Ajouter des tests de régression pour des bugs de modèle/fournisseur
  - Déboguer le comportement du Gateway et des agents
---

<div id="testing">
  # Tests
</div>

OpenClaw inclut trois suites Vitest (unit/integration, e2e, live) et un petit ensemble de runners Docker.

Ce document est un guide « comment nous testons » :

* Ce que chaque suite couvre (et ce qu’elle choisit délibérément de *ne pas* couvrir)
* Quelles commandes exécuter pour les flux de travail courants (local, pré-push, débogage)
* Comment les tests live détectent les identifiants et sélectionnent les modèles/fournisseurs
* Comment ajouter des tests de régression pour des problèmes réels de modèles/fournisseurs

<div id="quick-start">
  ## Démarrage rapide
</div>

La plupart des jours :

* Validation complète (attendue avant un push) : `pnpm lint && pnpm build && pnpm test`

Quand vous modifiez des tests ou que vous voulez davantage de garanties :

* Validation de couverture : `pnpm test:coverage`
* Suite E2E : `pnpm test:e2e`

Pour déboguer de vrais fournisseurs/modèles (nécessite de vrais identifiants) :

* Suite live (sondes sur les modèles + outils/images du Gateway) : `pnpm test:live`

Astuce : lorsque vous n’avez besoin que d’un seul cas en échec, privilégiez le filtrage des tests live via les variables d’environnement allowlist décrites ci-dessous.

<div id="test-suites-what-runs-where">
  ## Suites de tests (ce qui s’exécute où)
</div>

Considérez les suites comme des niveaux de « réalisme croissant » (et d’instabilité/coût croissants) :

<div id="unit-integration-default">
  ### Unitaire / intégration (par défaut)
</div>

* Commande : `pnpm test`
* Config : `vitest.config.ts`
* Fichiers : `src/**/*.test.ts`
* Portée :
  * Tests unitaires purs
  * Tests d’intégration dans le même processus (Gateway auth, routing, tooling, parsing, config)
  * Régressions déterministes pour des bugs connus
* Comportement attendu :
  * S’exécute en CI
  * Aucune clé réelle requise
  * Doit être rapide et stable

<div id="e2e-gateway-smoke">
  ### E2E (smoke Gateway)
</div>

* Commande : `pnpm test:e2e`
* Config : `vitest.e2e.config.ts`
* Fichiers : `src/**/*.e2e.test.ts`
* Portée :
  * Comportement de bout en bout du Gateway en multi‑instances
  * Interfaces WebSocket/HTTP, appairage de nœuds et réseau plus lourd
* Comportement attendu :
  * S’exécute en CI (lorsqu’activé dans le pipeline)
  * Aucune clé réelle requise
  * Plus de composants en jeu que les tests unitaires (peut être plus lent)

<div id="live-real-providers-real-models">
  ### Live (fournisseurs réels + modèles réels)
</div>

* Commande : `pnpm test:live`
* Config : `vitest.live.config.ts`
* Fichiers : `src/**/*.live.test.ts`
* Par défaut : **activé** par `pnpm test:live` (définit `OPENCLAW_LIVE_TEST=1`)
* Portée :
  * « Est-ce que ce fournisseur/modèle fonctionne réellement *aujourd’hui* avec de vrais identifiants ? »
  * Permet de détecter les changements de format côté fournisseur, les bizarreries d’appel d’outils, les problèmes d’authentification et le comportement des limites de débit
* Attentes :
  * Instable en CI par conception (vrais réseaux, vraies politiques des fournisseurs, quotas, pannes)
  * Coûte de l’argent / consomme les limites de débit
  * Préférez exécuter des sous-ensembles restreints plutôt que « tout »
  * Les exécutions live vont lire `~/.profile` pour récupérer des clés API manquantes
  * Rotation de clé Anthropic : définissez `OPENCLAW_LIVE_ANTHROPIC_KEYS="sk-...,sk-..."` (ou `OPENCLAW_LIVE_ANTHROPIC_KEY=sk-...`) ou plusieurs variables `ANTHROPIC_API_KEY*` ; les tests réessaieront en cas de limites de débit

<div id="which-suite-should-i-run">
  ## Quelle suite dois-je exécuter ?
</div>

Utilise le tableau de décision suivant :

* Si tu modifies la logique ou les tests : exécute `pnpm test` (et `pnpm test:coverage` si tu as fait beaucoup de changements)
* Si tu touches au réseau du Gateway / au protocole WS / à l’appairage : ajoute `pnpm test:e2e`
* Pour déboguer « mon bot est en panne » / des échecs spécifiques à un fournisseur / des appels d’outils : exécute un `pnpm test:live` ciblé

<div id="live-model-smoke-profile-keys">
  ## Live : smoke test du modèle (clés de profil)
</div>

Les tests live sont répartis en deux couches afin que tu puisses isoler les défaillances :

* « Modèle direct » nous indique que le fournisseur/modèle peut au moins répondre avec la clé donnée.
* « Gateway smoke » nous indique que l’ensemble de la chaîne Gateway+agent fonctionne pour ce modèle (sessions, historique, outils, stratégie de sandbox, etc.).

<div id="layer-1-direct-model-completion-no-gateway">
  ### Couche 1 : complétion directe de modèle (sans Gateway)
</div>

* Test : `src/agents/models.profiles.live.test.ts`
* Objectif :
  * Énumérer les modèles découverts
  * Utiliser `getApiKeyForModel` pour sélectionner les modèles pour lesquels vous avez des identifiants
  * Lancer une petite complétion par modèle (et des régressions ciblées si nécessaire)
* Comment l’activer :
  * `pnpm test:live` (ou `OPENCLAW_LIVE_TEST=1` si vous invoquez Vitest directement)
* Définissez `OPENCLAW_LIVE_MODELS=modern` (ou `all`, alias de modern) pour réellement exécuter cette suite ; sinon, elle est ignorée afin de garder `pnpm test:live` centré sur les tests de base du Gateway
* Comment sélectionner les modèles :
  * `OPENCLAW_LIVE_MODELS=modern` pour exécuter la liste d’autorisation moderne (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.1, Grok 4)
  * `OPENCLAW_LIVE_MODELS=all` est un alias de la liste d’autorisation moderne
  * ou `OPENCLAW_LIVE_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-5,..."` (liste d’autorisation séparée par des virgules)
* Comment sélectionner les fournisseurs :
  * `OPENCLAW_LIVE_PROVIDERS="google,google-antigravity,google-gemini-cli"` (liste d’autorisation séparée par des virgules)
* D’où viennent les clés :
  * Par défaut : stockage de profils et variables d’environnement de repli
  * Définissez `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` pour imposer **uniquement** le **stockage de profils**
* Pourquoi cela existe :
  * Distingue « l’API du fournisseur est en panne / la clé est invalide » de « le pipeline d’agent du Gateway est cassé »
  * Contient de petites régressions isolées (exemple : relecture du raisonnement pour OpenAI Responses/Codex Responses + flux d’appels d’outils)

<div id="layer-2-gateway-dev-agent-smoke-what-openclaw-actually-does">
  ### Couche 2 : Gateway + smoke test agent de dev (ce que « @openclaw » fait réellement)
</div>

* Test : `src/gateway/gateway-models.profiles.live.test.ts`
* Objectif :
  * Démarrer un Gateway en processus intégré (in-process)
  * Créer/mettre à jour une session `agent:dev:*` (surcharge de modèle par exécution)
  * Parcourir les modèles disposant de clés et vérifier :
    * une réponse « significative » (aucun outil)
    * qu’un appel d’outil réel fonctionne (sonde read)
    * des sondes d’outils supplémentaires optionnelles (sonde exec+read)
    * que les chemins de régression OpenAI (tool-call-only → follow-up) continuent de fonctionner
* Détails des sondes (pour que vous puissiez expliquer les échecs rapidement) :
  * sonde `read` : le test écrit un fichier nonce dans l’espace de travail et demande à l’agent de le `read` et de renvoyer le nonce.
  * sonde `exec+read` : le test demande à l’agent d’écrire un nonce via `exec` dans un fichier temporaire, puis de le `read`.
  * sonde d’image : le test joint un PNG généré (chat + code aléatoire) et s’attend à ce que le modèle renvoie `cat <CODE>`.
  * Référence d’implémentation : `src/gateway/gateway-models.profiles.live.test.ts` et `src/gateway/live-image-probe.ts`.
* Comment l’activer :
  * `pnpm test:live` (ou `OPENCLAW_LIVE_TEST=1` si vous invoquez Vitest directement)
* Comment sélectionner les modèles :
  * Par défaut : liste d’autorisation moderne (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.1, Grok 4)
  * `OPENCLAW_LIVE_GATEWAY_MODELS=all` est un alias de la liste d’autorisation moderne
  * Ou définissez `OPENCLAW_LIVE_GATEWAY_MODELS="provider/model"` (ou une liste séparée par des virgules) pour restreindre
* Comment sélectionner les fournisseurs (éviter « OpenRouter everything ») :
  * `OPENCLAW_LIVE_GATEWAY_PROVIDERS="google,google-antigravity,google-gemini-cli,openai,anthropic,zai,minimax"` (liste d’autorisation séparée par des virgules)
* Les sondes d’outils + d’image sont toujours activées dans ce test live :
  * sonde `read` + sonde `exec+read` (test de charge des outils)
  * la sonde d’image s’exécute lorsque le modèle annonce la prise en charge de l’entrée image
  * Flux (vue d’ensemble) :
    * Le test génère un petit PNG avec « CAT » + code aléatoire (`src/gateway/live-image-probe.ts`)
    * L’envoie via `agent` `attachments: [{ mimeType: "image/png", content: "<base64>" }]`
    * Gateway analyse les pièces jointes en `images[]` (`src/gateway/server-methods/agent.ts` + `src/gateway/chat-attachments.ts`)
    * L’agent embarqué transmet un message utilisateur multimodal au modèle
    * Assertion : la réponse contient `cat` + le code (tolérance OCR : petites erreurs autorisées)

Astuce : pour voir ce que vous pouvez tester sur votre machine (et les identifiants exacts `provider/model`), exécutez :

```bash
openclaw models list
openclaw models list --json
```

<div id="live-anthropic-setup-token-smoke">
  ## Live : smoke test Anthropic setup-token
</div>

* Test : `src/agents/anthropic.setup-token.live.test.ts`
* Objectif : vérifier que la configuration du setup-token de Claude Code CLI (ou un profil setup-token collé) peut exécuter un prompt Anthropic jusqu’au bout.
* Activation :
  * `pnpm test:live` (ou `OPENCLAW_LIVE_TEST=1` si vous invoquez Vitest directement)
  * `OPENCLAW_LIVE_SETUP_TOKEN=1`
* Sources du jeton (choisissez-en une) :
  * Profil : `OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test`
  * Jeton brut : `OPENCLAW_LIVE_SETUP_TOKEN_VALUE=sk-ant-oat01-...`
* Remplacement du modèle (optionnel) :
  * `OPENCLAW_LIVE_SETUP_TOKEN_MODEL=anthropic/claude-opus-4-5`

Exemple de configuration :

```bash
openclaw models auth paste-token --provider anthropic --profile-id anthropic:setup-token-test
OPENCLAW_LIVE_SETUP_TOKEN=1 OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test pnpm test:live src/agents/anthropic.setup-token.live.test.ts
```

<div id="live-cli-backend-smoke-claude-code-cli-or-other-local-clis">
  ## Live : test de fumée du backend CLI (Claude Code CLI ou autres CLI locales)
</div>

* Test : `src/gateway/gateway-cli-backend.live.test.ts`
* Objectif : valider le pipeline Gateway + agent en utilisant un backend CLI local, sans modifier votre configuration par défaut.
* Activation :
  * `pnpm test:live` (ou `OPENCLAW_LIVE_TEST=1` si vous appelez directement Vitest)
  * `OPENCLAW_LIVE_CLI_BACKEND=1`
* Valeurs par défaut :
  * Modèle : `claude-cli/claude-sonnet-4-5`
  * Commande : `claude`
  * Arguments : `["-p","--output-format","json","--dangerously-skip-permissions"]`
* Surcharges (facultatif) :
  * `OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-opus-4-5"`
  * `OPENCLAW_LIVE_CLI_BACKEND_MODEL="codex-cli/gpt-5.2-codex"`
  * `OPENCLAW_LIVE_CLI_BACKEND_COMMAND="/full/path/to/claude"`
  * `OPENCLAW_LIVE_CLI_BACKEND_ARGS='["-p","--output-format","json","--permission-mode","bypassPermissions"]'`
  * `OPENCLAW_LIVE_CLI_BACKEND_CLEAR_ENV='["ANTHROPIC_API_KEY","ANTHROPIC_API_KEY_OLD"]'`
  * `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_PROBE=1` pour envoyer un véritable fichier image en pièce jointe (les chemins sont injectés dans le prompt).
  * `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_ARG="--image"` pour passer les chemins des fichiers image comme arguments de CLI au lieu d’une injection dans le prompt.
  * `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_MODE="repeat"` (ou `"list"`) pour contrôler la façon dont les arguments image sont passés lorsque `IMAGE_ARG` est défini.
  * `OPENCLAW_LIVE_CLI_BACKEND_RESUME_PROBE=1` pour envoyer un deuxième échange et valider le flux de reprise.
* `OPENCLAW_LIVE_CLI_BACKEND_DISABLE_MCP_CONFIG=0` pour conserver la configuration MCP de Claude Code CLI activée (par défaut, la config MCP est désactivée avec un fichier vide temporaire).

Exemple :

```bash
OPENCLAW_LIVE_CLI_BACKEND=1 \
  OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-sonnet-4-5" \
  pnpm test:live src/gateway/gateway-cli-backend.live.test.ts
```

<div id="recommended-live-recipes">
  ### Recettes de tests live recommandées
</div>

Des listes d’autorisation restreintes et explicites sont les plus rapides et les plus fiables :

* Modèle unique, direct (sans Gateway) :
  * `OPENCLAW_LIVE_MODELS="openai/gpt-5.2" pnpm test:live src/agents/models.profiles.live.test.ts`

* Modèle unique, smoke test via le Gateway :
  * `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

* Appels d’outils avec plusieurs fournisseurs :
  * `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-5,google/gemini-3-flash-preview,zai/glm-4.7,minimax/minimax-m2.1" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

* Focus Google (clé d’API Gemini + Antigravity) :
  * Gemini (clé d’API) : `OPENCLAW_LIVE_GATEWAY_MODELS="google/gemini-3-flash-preview" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
  * Antigravity (OAuth) : `OPENCLAW_LIVE_GATEWAY_MODELS="google-antigravity/claude-opus-4-5-thinking,google-antigravity/gemini-3-pro-high" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

Notes :

* `google/...` utilise l’API Gemini (clé d’API).
* `google-antigravity/...` utilise le pont OAuth Antigravity (endpoint d’agent de type Cloud Code Assist).
* `google-gemini-cli/...` utilise la CLI Gemini locale sur votre machine (authentification dédiée + particularités d’outillage).
* API Gemini vs CLI Gemini :
  * API : OpenClaw appelle l’API Gemini hébergée de Google via HTTP (clé d’API / profil d’authentification) ; c’est ce que la plupart des utilisateurs entendent par « Gemini ».
  * CLI : OpenClaw lance un binaire local `gemini` ; il a sa propre authentification et peut se comporter différemment (prise en charge du streaming / des outils / écart de versions).

<div id="live-model-matrix-what-we-cover">
  ## Live : matrice de modèles (ce que nous couvrons)
</div>

Il n’existe pas de « liste de modèles CI » fixe (le live est à activer explicitement), mais ce sont les modèles **recommandés** à tester régulièrement sur une machine de développement disposant de clés API.

<div id="modern-smoke-set-tool-calling-image">
  ### Jeu de tests de fumée moderne (appel d’outils + image)
</div>

C’est l’exécution des « modèles courants » que nous prévoyons de garder fonctionnels :

* OpenAI (non-Codex) : `openai/gpt-5.2` (facultatif : `openai/gpt-5.1`)
* OpenAI Codex : `openai-codex/gpt-5.2` (facultatif : `openai-codex/gpt-5.2-codex`)
* Anthropic : `anthropic/claude-opus-4-5` (ou `anthropic/claude-sonnet-4-5`)
* Google (API Gemini) : `google/gemini-3-pro-preview` et `google/gemini-3-flash-preview` (évitez les anciens modèles Gemini 2.x)
* Google (Antigravity) : `google-antigravity/claude-opus-4-5-thinking` et `google-antigravity/gemini-3-flash`
* Z.AI (GLM) : `zai/glm-4.7`
* MiniMax : `minimax/minimax-m2.1`

Lancez le test de fumée du Gateway avec outils + image :
`OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,openai-codex/gpt-5.2,anthropic/claude-opus-4-5,google/gemini-3-pro-preview,google/gemini-3-flash-preview,google-antigravity/claude-opus-4-5-thinking,google-antigravity/gemini-3-flash,zai/glm-4.7,minimax/minimax-m2.1" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

<div id="baseline-tool-calling-read-optional-exec">
  ### Configuration de base : appel d’outils (Read + Exec en option)
</div>

Choisissez au moins un modèle par famille de fournisseurs :

* OpenAI : `openai/gpt-5.2` (ou `openai/gpt-5-mini`)
* Anthropic : `anthropic/claude-opus-4-5` (ou `anthropic/claude-sonnet-4-5`)
* Google : `google/gemini-3-flash-preview` (ou `google/gemini-3-pro-preview`)
* Z.AI (GLM) : `zai/glm-4.7`
* MiniMax : `minimax/minimax-m2.1`

Couverture supplémentaire facultative (recommandée) :

* xAI : `xai/grok-4` (ou dernière version disponible)
* Mistral : `mistral/`… (choisissez un modèle compatible « tools » que vous avez activé)
* Cerebras : `cerebras/`… (si vous y avez accès)
* LM Studio : `lmstudio/`… (local ; l’appel d’outils dépend du mode api)

<div id="vision-image-send-attachment-multimodal-message">
  ### Vision : envoi d’image (pièce jointe → message multimodal)
</div>

Incluez au moins un modèle compatible avec les images dans `OPENCLAW_LIVE_GATEWAY_MODELS` (variantes de Claude/Gemini/OpenAI dotées de capacités de vision, etc.) afin de tester la sonde d’images.

<div id="aggregators-alternate-gateways">
  ### Agrégateurs / passerelles alternatives
</div>

Si vous avez des clés API activées, nous prenons également en charge les tests via :

* OpenRouter : `openrouter/...` (des centaines de modèles ; utilisez `openclaw models scan` pour trouver des candidats compatibles outils + images)
* OpenCode Zen : `opencode/...` (authentification via `OPENCODE_API_KEY` / `OPENCODE_ZEN_API_KEY`)

Vous pouvez inclure davantage de fournisseurs dans la matrice en temps réel (si vous avez les identifiants/la configuration) :

* Intégrés : `openai`, `openai-codex`, `anthropic`, `google`, `google-vertex`, `google-antigravity`, `google-gemini-cli`, `zai`, `openrouter`, `opencode`, `xai`, `groq`, `cerebras`, `mistral`, `github-copilot`
* Via `models.providers` (points de terminaison personnalisés) : `minimax` (cloud/API), plus tout proxy compatible OpenAI/Anthropic (LM Studio, vLLM, LiteLLM, etc.)

Conseil : n’essayez pas de coder en dur « tous les modèles » dans la documentation. La liste de référence est celle que `discoverModels(...)` renvoie sur votre machine, plus toutes les clés disponibles.

<div id="credentials-never-commit">
  ## Identifiants (ne jamais les committer)
</div>

Les tests en direct détectent les identifiants de la même manière que la CLI. Implications pratiques :

* Si la CLI fonctionne, les tests en direct devraient trouver les mêmes clés.

* Si un test en direct indique « no creds », déboguez de la même manière que vous débogueriez `openclaw models list` / la sélection de modèle.

* Stockage du profil : `~/.openclaw/credentials/` (préféré ; ce que « profile keys » signifie dans les tests)

* Configuration : `~/.openclaw/openclaw.json` (ou `OPENCLAW_CONFIG_PATH`)

Si vous voulez vous reposer sur des clés d’environnement (par exemple exportées dans votre `~/.profile`), exécutez les tests locaux après `source ~/.profile`, ou utilisez les runners Docker ci-dessous (ils peuvent monter `~/.profile` dans le conteneur).

<div id="deepgram-live-audio-transcription">
  ## Deepgram live (transcription audio en direct)
</div>

* Test : `src/media-understanding/providers/deepgram/audio.live.test.ts`
* Activer : `DEEPGRAM_API_KEY=... DEEPGRAM_LIVE_TEST=1 pnpm test:live src/media-understanding/providers/deepgram/audio.live.test.ts`

<div id="docker-runners-optional-works-in-linux-checks">
  ## Runners Docker (vérifications optionnelles « fonctionne sous Linux »)
</div>

Ces commandes exécutent `pnpm test:live` dans l’image Docker du dépôt, en montant votre répertoire de configuration local et votre espace de travail (et en chargeant `~/.profile` s’il est monté) :

* Modèles directs : `pnpm test:docker:live-models` (script : `scripts/test-live-models-docker.sh`)
* Gateway + agent de dev : `pnpm test:docker:live-gateway` (script : `scripts/test-live-gateway-models-docker.sh`)
* Assistant d’onboarding (TTY, scaffolding complet) : `pnpm test:docker:onboard` (script : `scripts/e2e/onboard-docker.sh`)
* Réseau du Gateway (deux conteneurs, auth WS + vérification d’état) : `pnpm test:docker:gateway-network` (script : `scripts/e2e/gateway-network-docker.sh`)
* Plugins (chargement d’extensions personnalisées + test de bon fonctionnement du registre) : `pnpm test:docker:plugins` (script : `scripts/e2e/plugins-docker.sh`)

Variables d’environnement utiles :

* `OPENCLAW_CONFIG_DIR=...` (par défaut : `~/.openclaw`) monté sur `/home/node/.openclaw`
* `OPENCLAW_WORKSPACE_DIR=...` (par défaut : `~/.openclaw/workspace`) monté sur `/home/node/.openclaw/workspace`
* `OPENCLAW_PROFILE_FILE=...` (par défaut : `~/.profile`) monté sur `/home/node/.profile` et chargé avant l’exécution des tests
* `OPENCLAW_LIVE_GATEWAY_MODELS=...` / `OPENCLAW_LIVE_MODELS=...` pour limiter l’exécution
* `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` pour garantir que les identifiants proviennent du stockage de profils (et non de l’environnement)

<div id="docs-sanity">
  ## Validation de la documentation
</div>

Lancez les vérifications de la documentation après chaque modification : `pnpm docs:list`.

<div id="offline-regression-ci-safe">
  ## Régression hors ligne (compatible CI)
</div>

Il s’agit de régressions de « pipeline réel » sans vrais fournisseurs :

* Appels d’outils du Gateway (OpenAI simulé, vraie boucle gateway + agent) : `src/gateway/gateway.tool-calling.mock-openai.test.ts`
* Assistant de configuration du Gateway (WS `wizard.start`/`wizard.next`, écrit la configuration + contrôles d’authentification appliqués) : `src/gateway/gateway.wizard.e2e.test.ts`

<div id="agent-reliability-evals-skills">
  ## Évaluations de fiabilité d’agent (compétences)
</div>

Nous avons déjà quelques tests sûrs pour la CI qui se comportent comme des « évaluations de fiabilité d’agent » :

* Appels d’outils simulés via la véritable boucle Gateway + agent (`src/gateway/gateway.tool-calling.mock-openai.test.ts`).
* Flux de wizard de bout en bout qui valident le câblage des sessions et les effets de configuration (`src/gateway/gateway.wizard.e2e.test.ts`).

Ce qui manque encore pour les compétences (voir [Skills](/fr/tools/skills)) :

* **Prise de décision :** quand les compétences sont listées dans le prompt, l’agent choisit‑il la bonne compétence (ou évite‑t‑il celles qui sont hors sujet) ?
* **Conformité :** l’agent lit‑il `SKILL.md` avant usage et suit‑il les étapes/arguments requis ?
* **Contrats de workflow :** scénarios multi‑tours qui vérifient l’ordre des outils, la persistance de l’historique de session et les limites du sandbox.

Les évaluations futures doivent rester déterministes en priorité :

* Un moteur de scénarios utilisant des fournisseurs fictifs pour vérifier les appels d’outils + leur ordre, les lectures de fichiers de compétence et le câblage des sessions.
* Une petite suite de scénarios centrés sur les compétences (utilisation vs évitement, contrôle d’accès, injection de prompt).
* Des évaluations « live » optionnelles (opt‑in, contrôlées par l’environnement) seulement une fois la suite sûre pour la CI en place.

<div id="adding-regressions-guidance">
  ## Ajouter des régressions (recommandations)
</div>

Quand vous corrigez un problème de fournisseur/modèle découvert en production :

* Ajoutez si possible une régression sûre pour la CI (fournisseur mock/stub, ou capture exacte de la transformation de la structure de requête)
* Si le problème n’est reproductible qu’en conditions réelles (limitations de débit, politiques d’authentification), gardez le test live ciblé et activable explicitement via des variables d’environnement
* Privilégiez la plus petite couche capable de capturer le bug :
  * bug de conversion/relecture de requête fournisseur → test direct des modèles
  * bug du pipeline session/historique/outils du Gateway → test de fumée live du Gateway ou test mock du Gateway sûr pour la CI