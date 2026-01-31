---
title: Web
summary: "Outils de recherche et de récupération sur le Web (Brave Search API, Perplexity direct/OpenRouter)"
read_when:
  - Vous souhaitez activer web_search ou web_fetch
  - Vous devez configurer une clé d’API Brave Search
  - Vous souhaitez utiliser Perplexity Sonar pour la recherche sur le Web
---

<div id="web-tools">
  # Outils web
</div>

OpenClaw est livré avec deux outils web légers :

* `web_search` — Recherche sur le web via l&#39;API Brave Search (par défaut) ou Perplexity Sonar (directement ou via OpenRouter).
* `web_fetch` — Requête HTTP + extraction lisible (HTML → markdown/texte).

Ce ne sont **pas** des outils d&#39;automatisation de navigateur. Pour les sites très dépendants de JS ou nécessitant une connexion, utilisez l&#39;[outil Browser](/fr/tools/browser).

<div id="how-it-works">
  ## Fonctionnement
</div>

* `web_search` appelle votre fournisseur configuré et renvoie des résultats.
  * **Brave** (par défaut) : renvoie des résultats structurés (titre, URL, extrait).
  * **Perplexity** : renvoie des réponses synthétisées par IA avec des citations issues d’une recherche sur le web en temps réel.
* Les résultats sont mis en cache, requête par requête, pendant 15 minutes (paramétrable).
* `web_fetch` effectue une requête HTTP GET simple et extrait le contenu lisible
  (HTML → markdown/texte). Il **n’exécute pas** JavaScript.
* `web_fetch` est activé par défaut (sauf si explicitement désactivé).

<div id="choosing-a-search-provider">
  ## Choisir un fournisseur de recherche
</div>

| Fournisseur            | Avantages                                           | Inconvénients                               | Clé API                                      |
| ---------------------- | --------------------------------------------------- | ------------------------------------------- | -------------------------------------------- |
| **Brave** (par défaut) | Résultats rapides et structurés, offre gratuite     | Résultats de recherche traditionnels        | `BRAVE_API_KEY`                              |
| **Perplexity**         | Réponses synthétisées par IA, citations, temps réel | Nécessite un accès Perplexity ou OpenRouter | `OPENROUTER_API_KEY` ou `PERPLEXITY_API_KEY` |

Voir [configuration de Brave Search](/fr/brave-search) et [Perplexity Sonar](/fr/perplexity) pour des détails propres à chaque fournisseur.

Définissez le fournisseur dans la configuration :

```json5
{
  tools: {
    web: {
      search: {
        provider: "brave"  // ou "perplexity"
      }
    }
  }
}
```

Exemple : passer à Perplexity Sonar (API directe) :

```json5
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai",
          model: "perplexity/sonar-pro"
        }
      }
    }
  }
}
```

<div id="getting-a-brave-api-key">
  ## Obtention d’une clé API Brave
</div>

1. Créez un compte pour l’API Brave Search sur https://brave.com/search/api/
2. Dans le tableau de bord, choisissez l’offre **Data for Search** (et non « Data for AI ») et générez une clé API.
3. Exécutez `openclaw configure --section web` pour stocker la clé dans la configuration (recommandé), ou définissez `BRAVE_API_KEY` dans vos variables d’environnement.

Brave propose un niveau gratuit ainsi que des offres payantes ; consultez le portail Brave API pour connaître les limites et tarifs actuels.

<div id="where-to-set-the-key-recommended">
  ### Où définir la clé (recommandé)
</div>

**Recommandé :** exécutez `openclaw configure --section web`. Cette commande enregistre la clé dans
`~/.openclaw/openclaw.json` sous `tools.web.search.apiKey`.

**Alternative via variable d’environnement :** définissez `BRAVE_API_KEY` dans l’environnement du processus Gateway. Pour une installation du Gateway, placez-la dans `~/.openclaw/.env` (ou dans l’environnement de votre service). Voir [Variables d’environnement](/fr/help/faq#how-does-openclaw-load-environment-variables).

<div id="using-perplexity-direct-or-via-openrouter">
  ## Utiliser Perplexity (directement ou via OpenRouter)
</div>

Les modèles Perplexity Sonar intègrent des capacités de recherche sur le Web et renvoient des réponses synthétisées par l’IA avec des citations. Vous pouvez les utiliser via OpenRouter (ne nécessite pas de carte bancaire — prend en charge la crypto et les paiements prépayés).

<div id="getting-an-openrouter-api-key">
  ### Obtenir une clé API OpenRouter
</div>

1. Créez un compte sur https://openrouter.ai/
2. Ajoutez des crédits (paiement par cryptomonnaie, carte prépayée ou carte bancaire)
3. Générez une clé API depuis les paramètres de votre compte

<div id="setting-up-perplexity-search">
  ### Configuration de la recherche avec Perplexity
</div>

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "perplexity",
        perplexity: {
          // Clé API (optionnelle si OPENROUTER_API_KEY ou PERPLEXITY_API_KEY est définie)
          apiKey: "sk-or-v1-...",
          // Base URL (key-aware default if omitted)
          baseUrl: "https://openrouter.ai/api/v1",
          // Model (defaults to perplexity/sonar-pro)
          model: "perplexity/sonar-pro"
        }
      }
    }
  }
}
```

**Autre méthode via variable d&#39;environnement :** définissez `OPENROUTER_API_KEY` ou `PERPLEXITY_API_KEY` dans l&#39;environnement du Gateway. Pour une installation du Gateway, placez-la dans `~/.openclaw/.env`.

Si aucune URL de base n&#39;est définie, OpenClaw choisit une valeur par défaut en fonction de la source de la clé API :

* `PERPLEXITY_API_KEY` ou `pplx-...` → `https://api.perplexity.ai`
* `OPENROUTER_API_KEY` ou `sk-or-...` → `https://openrouter.ai/api/v1`
* Formats de clés inconnus → OpenRouter (solution de repli sûre)

<div id="available-perplexity-models">
  ### Modèles Perplexity disponibles
</div>

| Modèle | Description | Idéal pour |
|--------|-------------|------------|
| `perplexity/sonar` | Questions-réponses rapides avec recherche sur le Web | Recherches rapides |
| `perplexity/sonar-pro` (par défaut) | Raisonnement en plusieurs étapes avec recherche sur le Web | Questions complexes |
| `perplexity/sonar-reasoning-pro` | Analyse du raisonnement pas à pas | Recherche approfondie |

<div id="web_search">
  ## web_search
</div>

Effectuez une recherche sur le Web avec le fournisseur que vous avez configuré.

<div id="requirements">
  ### Prérequis
</div>

* `tools.web.search.enabled` ne doit pas être défini sur `false` (valeur par défaut : activé)
* Clé API pour le fournisseur de votre choix :
  * **Brave** : `BRAVE_API_KEY` ou `tools.web.search.apiKey`
  * **Perplexity** : `OPENROUTER_API_KEY`, `PERPLEXITY_API_KEY` ou `tools.web.search.perplexity.apiKey`

<div id="config">
  ### Configuration
</div>

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE", // optionnel si BRAVE_API_KEY est définie
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15
      }
    }
  }
}
```

<div id="tool-parameters">
  ### Paramètres de l’outil
</div>

* `query` (requis)
* `count` (1–10 ; valeur par défaut définie dans la configuration)
* `country` (optionnel) : code de pays à 2 lettres pour des résultats spécifiques à une région (par ex. &quot;DE&quot;, &quot;US&quot;, &quot;ALL&quot;). S’il est omis, Brave choisit sa région par défaut.
* `search_lang` (optionnel) : code de langue ISO pour les résultats de recherche (par ex. &quot;de&quot;, &quot;en&quot;, &quot;fr&quot;)
* `ui_lang` (optionnel) : code de langue ISO pour les éléments de l’UI
* `freshness` (optionnel, Brave uniquement) : filtre selon la date de découverte (`pd`, `pw`, `pm`, `py`, ou `YYYY-MM-DDtoYYYY-MM-DD`)

**Exemples :**

```javascript
// German-specific search
await web_search({
  query: "TV online schauen",
  count: 10,
  country: "DE",
  search_lang: "de"
});

// Recherche française avec UI française
await web_search({
  query: "actualités",
  country: "FR",
  search_lang: "fr",
  ui_lang: "fr"
});

// Recent results (past week)
await web_search({
  query: "TMBG interview",
  freshness: "pw"
});
```

<div id="web_fetch">
  ## web_fetch
</div>

Récupère le contenu d’une URL et en extrait la partie lisible.

<div id="requirements">
  ### Prérequis
</div>

* `tools.web.fetch.enabled` ne doit pas être `false` (par défaut : activé)
* Repli optionnel sur Firecrawl : définissez `tools.web.fetch.firecrawl.apiKey` ou `FIRECRAWL_API_KEY`.

<div id="config">
  ### Configuration
</div>

```json5
{
  tools: {
    web: {
      fetch: {
        enabled: true,
        maxChars: 50000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        maxRedirects: 3,
        userAgent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 14_7_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36",
        readability: true,
        firecrawl: {
          enabled: true,
          apiKey: "FIRECRAWL_API_KEY_HERE", // optionnel si FIRECRAWL_API_KEY est définie
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 86400000, // ms (1 day)
          timeoutSeconds: 60
        }
      }
    }
  }
}
```

<div id="tool-parameters">
  ### Paramètres de l’outil
</div>

* `url` (obligatoire, http/https uniquement)
* `extractMode` (`markdown` | `text`)
* `maxChars` (tronque les pages longues)

Notes :

* `web_fetch` utilise d’abord Readability (extraction du contenu principal), puis Firecrawl (si configuré). Si les deux échouent, l’outil renvoie une erreur.
* Les requêtes Firecrawl utilisent un mode de contournement des bots et mettent les résultats en cache par défaut.
* `web_fetch` envoie par défaut un User-Agent de type Chrome et `Accept-Language` ; remplacez `userAgent` si nécessaire.
* `web_fetch` bloque les noms d’hôte privés/internes et revalide les redirections (limitez-les avec `maxRedirects`).
* `web_fetch` effectue une extraction « au mieux » (best effort) ; certains sites nécessiteront l’outil de navigation (navigateur).
* Voir [Firecrawl](/fr/tools/firecrawl) pour la configuration de la clé et les détails du service.
* Les réponses sont mises en cache (15 minutes par défaut) pour réduire les extractions répétées.
* Si vous utilisez des profils d’outils ou des listes d’autorisation, ajoutez `web_search`/`web_fetch` ou `group:web`.
* Si la clé Brave est manquante, `web_search` renvoie un court message d’aide à la configuration avec un lien vers la documentation.