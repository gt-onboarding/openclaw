---
title: Test
summary: "Comment exécuter les tests en local (vitest) et quand utiliser les modes force/coverage"
read_when:
  - Lorsque vous exécutez ou corrigez des tests
---

<div id="tests">
  # Tests
</div>

* Kit de tests complet (suites, live, Docker) : [Testing](/fr/testing)

* `pnpm test:force` : tue tout processus Gateway persistant qui occupe le port de contrôle par défaut, puis exécute la suite Vitest complète avec un port Gateway isolé, afin que les tests serveur n’entrent pas en collision avec une instance déjà en cours d’exécution. Utilisez cette commande lorsqu’une exécution précédente du Gateway a laissé le port 18789 occupé.

* `pnpm test:coverage` : exécute Vitest avec la couverture V8. Les seuils globaux sont de 70 % pour les lignes/branches/fonctions/instructions. La couverture exclut les points d’entrée fortement orientés intégration (câblage de la CLI, passerelles Gateway/Telegram, serveur statique webchat) afin de garder l’objectif centré sur la logique testable par des tests unitaires.

* `pnpm test:e2e` : exécute les tests de fumée de bout en bout du Gateway (WS/HTTP/appairage de nœuds multi‑instances).

* `pnpm test:live` : exécute les tests live des fournisseurs (minimax/zai). Nécessite des clés API et `LIVE=1` (ou `*_LIVE_TEST=1` spécifique au fournisseur) pour que les tests ne soient pas ignorés.

<div id="model-latency-bench-local-keys">
  ## Banc de test de latence des modèles (clés locales)
</div>

Script : [`scripts/bench-model.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-model.ts)

Utilisation :

* `source ~/.profile && pnpm tsx scripts/bench-model.ts --runs 10`
* Variables d&#39;environnement optionnelles : `MINIMAX_API_KEY`, `MINIMAX_BASE_URL`, `MINIMAX_MODEL`, `ANTHROPIC_API_KEY`
* Prompt par défaut : « Réponds avec un seul mot : ok. Sans ponctuation ni texte supplémentaire. »

Dernière exécution (2025-12-31, 20 exécutions) :

* minimax latence médiane 1279 ms (min 1114 ms, max 2431 ms)
* opus latence médiane 2454 ms (min 1224 ms, max 3170 ms)

<div id="onboarding-e2e-docker">
  ## Onboarding E2E (Docker)
</div>

Docker est facultatif ; ce n&#39;est nécessaire que pour les tests de bon fonctionnement d&#39;onboarding en conteneur.

Flux complet de démarrage à froid dans un conteneur Linux vierge :

```bash
scripts/e2e/onboard-docker.sh
```

Ce script pilote l’assistant interactif via un pseudo-tty, vérifie les fichiers de configuration, d’espace de travail et de session, puis démarre le Gateway et exécute `openclaw health`.

<div id="qr-import-smoke-docker">
  ## Test rapide d&#39;import QR (Docker)
</div>

Vérifie que `qrcode-terminal` se charge correctement avec Node.js 22+ dans Docker :

```bash
pnpm test:docker:qr
```
