---
title: Élagage des sessions
summary: "Élagage des sessions : élagage des résultats d’outils pour limiter l’encombrement du contexte"
read_when:
  - Vous voulez réduire la croissance du contexte du LLM liée aux résultats d’outils
  - Vous êtes en train d’ajuster agents.defaults.contextPruning
---

<div id="session-pruning">
  # Élagage des sessions
</div>

L&#39;élagage des sessions supprime les **anciens résultats d&#39;outils** du contexte en mémoire juste avant chaque appel au LLM. Il ne réécrit **pas** l&#39;historique de la session sur le disque (`*.jsonl`).

<div id="when-it-runs">
  ## Moment d&#39;exécution
</div>

* Lorsque `mode: "cache-ttl"` est activé et que le dernier appel Anthropic pour la session est antérieur à `ttl`.
* Ne concerne que les messages envoyés au modèle pour cette requête.
* Actif uniquement pour les appels à l&#39;API Anthropic (et les modèles Anthropic via OpenRouter).
* Pour de meilleurs résultats, alignez `ttl` sur le `cacheControlTtl` de votre modèle.
* Après une purge, la fenêtre de TTL est réinitialisée afin que les requêtes suivantes réutilisent le cache jusqu&#39;à la nouvelle expiration de `ttl`.

<div id="smart-defaults-anthropic">
  ## Paramètres par défaut optimisés (Anthropic)
</div>

* Profils **OAuth ou setup-token** : activez le nettoyage `cache-ttl` et définissez le signal de vie sur `1h`.
* Profils **API key** : activez le nettoyage `cache-ttl`, définissez le signal de vie sur `30m` et définissez `cacheControlTtl` par défaut à `1h` sur les modèles Anthropic.
* Si vous définissez explicitement l&#39;une de ces valeurs, OpenClaw ne les remplace **pas**.

<div id="what-this-improves-cost-cache-behavior">
  ## Ce que cela améliore (coût + comportement du cache)
</div>

* **Pourquoi élaguer :** le cache de prompts Anthropic ne s’applique que pendant le TTL. Si une session reste inactive au‑delà du TTL, la requête suivante remet en cache le prompt complet, sauf si vous le réduisez d’abord.
* **Ce qui devient moins cher :** l’élagage réduit la taille de **cacheWrite** pour cette première requête après l’expiration du TTL.
* **Pourquoi la réinitialisation du TTL est importante :** une fois l’élagage exécuté, la fenêtre de cache est réinitialisée, donc les requêtes suivantes peuvent réutiliser le prompt fraîchement mis en cache au lieu de remettre en cache l’intégralité de l’historique.
* **Ce que cela ne fait pas :** l’élagage n’ajoute pas de jetons et ne « double » pas les coûts ; il modifie uniquement ce qui est mis en cache lors de cette première requête après expiration du TTL.

<div id="what-can-be-pruned">
  ## Ce qui peut être élagué
</div>

* Uniquement les messages `toolResult`.
* Les messages utilisateur + assistant ne sont **jamais** modifiés.
* Les derniers messages de l’assistant définis par `keepLastAssistants` sont protégés ; les résultats d’outils après ce seuil ne sont pas élagués.
* S’il n’y a pas assez de messages de l’assistant pour établir ce seuil, l’élagage est ignoré.
* Les résultats d’outils contenant des **blocs d’image** sont ignorés (jamais tronqués ni effacés).

<div id="context-window-estimation">
  ## Estimation de la fenêtre de contexte
</div>

L’élagage utilise une fenêtre de contexte estimée (caractères ≈ tokens × 4). La taille de la fenêtre est déterminée dans cet ordre :

1. Définition du modèle `contextWindow` (provenant du registre de modèles).
2. Remplacement `models.providers.*.models[].contextWindow`.
3. `agents.defaults.contextTokens`.
4. Valeur par défaut : `200000` tokens.

<div id="mode">
  ## Mode
</div>

<div id="cache-ttl">
  ### cache-ttl
</div>

* L&#39;élagage ne s&#39;exécute que si le dernier appel à Anthropic date de plus de `ttl` (par défaut `5m`).
* Lorsqu&#39;il s&#39;exécute : il applique le même comportement de `soft-trim` + `hard-clear` qu&#39;auparavant.

<div id="soft-vs-hard-pruning">
  ## Élagage souple vs strict
</div>

* **Soft-trim** : uniquement pour les résultats d’outil surdimensionnés.
  * Conserve le début et la fin, insère `...` et ajoute une note avec la taille d’origine.
  * Ignore les résultats avec des blocs d’image.
* **Hard-clear** : remplace l’intégralité du résultat d’outil par `hardClear.placeholder`.

<div id="tool-selection">
  ## Sélection des outils
</div>

* `tools.allow` / `tools.deny` prennent en charge les caractères génériques `*`.
* Les règles de refus priment.
* La correspondance n’est pas sensible à la casse.
* Liste d’autorisation vide ⇒ tous les outils sont autorisés.

<div id="interaction-with-other-limits">
  ## Interaction avec d&#39;autres limites
</div>

* Les outils intégrés tronquent déjà leurs propres sorties ; l&#39;élagage de session est une couche supplémentaire qui empêche les conversations de longue durée d&#39;accumuler trop de sorties d&#39;outils dans le contexte du modèle.
* La compaction est distincte : la compaction résume et conserve, tandis que l&#39;élagage est transitoire et appliqué à chaque requête. Voir [/concepts/compaction](/fr/concepts/compaction).

<div id="defaults-when-enabled">
  ## Valeurs par défaut (lorsqu’elles sont activées)
</div>

* `ttl`: `"5m"`
* `keepLastAssistants`: `3`
* `softTrimRatio`: `0.3`
* `hardClearRatio`: `0.5`
* `minPrunableToolChars`: `50000`
* `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }`
* `hardClear`: `{ enabled: true, placeholder: "[Ancien contenu de résultat d’outil effacé]" }`

<div id="examples">
  ## Exemples
</div>

Par défaut (désactivé) :

```json5
{
  agent: {
    contextPruning: { mode: "off" }
  }
}
```

Activer la purge avec prise en compte du TTL :

```json5
{
  agent: {
    contextPruning: { mode: "cache-ttl", ttl: "5m" }
  }
}
```

Limiter la purge à certains outils spécifiques :

```json5
{
  agent: {
    contextPruning: {
      mode: "cache-ttl",
      tools: { allow: ["exec", "read"], deny: ["*image*"] }
    }
  }
}
```

Consultez la référence de configuration : [Configuration du Gateway](/fr/gateway/configuration)
