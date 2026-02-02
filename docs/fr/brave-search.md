---
title: Brave Search
summary: "Configuration de l’API Brave Search pour web_search"
read_when:
  - Vous souhaitez utiliser Brave Search pour web_search
  - Vous avez besoin d’une BRAVE_API_KEY ou de détails sur l’offre
---

<div id="brave-search-api">
  # API Brave Search
</div>

OpenClaw utilise Brave Search comme fournisseur par défaut de `web_search`.

<div id="get-an-api-key">
  ## Obtenir une clé API
</div>

1. Créez un compte Brave Search API sur https://brave.com/search/api/
2. Dans le tableau de bord, choisissez l’offre **Data for Search** et générez une clé API.
3. Stockez la clé dans la configuration (recommandé) ou définissez `BRAVE_API_KEY` dans l’environnement du Gateway.

<div id="config-example">
  ## Exemple de configuration
</div>

```json5
{
  tools: {
    web: {
      search: {
        provider: "brave",
        apiKey: "BRAVE_API_KEY_HERE",
        maxResults: 5,
        timeoutSeconds: 30
      }
    }
  }
}
```

<div id="notes">
  ## Remarques
</div>

* L’offre Data for AI n’est **pas** compatible avec `web_search`.
* Brave propose une formule gratuite ainsi que des offres payantes ; consultez le portail API Brave pour connaître les limites actuelles.

Consultez la section [Outils web](/fr/tools/web) pour la configuration complète de `web_search`.