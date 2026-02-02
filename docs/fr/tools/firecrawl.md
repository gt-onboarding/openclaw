---
title: Firecrawl
summary: "Solution de repli Firecrawl pour web_fetch (anti-bot + extraction mise en cache)"
read_when:
  - Vous souhaitez une extraction web basée sur Firecrawl
  - Vous avez besoin d'une clé api Firecrawl
  - Vous souhaitez une extraction anti-bot pour web_fetch
---

<div id="firecrawl">
  # Firecrawl
</div>

OpenClaw peut utiliser **Firecrawl** comme extracteur de secours pour `web_fetch`. Il s’agit d’un service hébergé d’extraction de contenu qui prend en charge le contournement des protections anti‑bots et la mise en cache, ce qui aide pour les sites fortement basés sur JavaScript ou les pages qui bloquent les requêtes HTTP classiques.

<div id="get-an-api-key">
  ## Obtenir une clé API
</div>

1. Créez un compte Firecrawl et générez une clé API.
2. Stockez-la dans la configuration ou définissez `FIRECRAWL_API_KEY` dans l&#39;environnement de votre Gateway.

<div id="configure-firecrawl">
  ## Configurer Firecrawl
</div>

```json5
{
  tools: {
    web: {
      fetch: {
        firecrawl: {
          apiKey: "FIRECRAWL_API_KEY_HERE",
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 172800000,
          timeoutSeconds: 60
        }
      }
    }
  }
}
```

Notes :

* `firecrawl.enabled` est activé par défaut lorsqu’une clé d’API est présente.
* `maxAgeMs` détermine l’ancienneté maximale autorisée des résultats mis en cache (en ms). La valeur par défaut est de 2 jours.

<div id="stealth-bot-circumvention">
  ## Discrétion / contournement des bots
</div>

Firecrawl propose un paramètre de **mode proxy** pour le contournement des bots (`basic`, `stealth` ou `auto`).
OpenClaw utilise toujours `proxy: "auto"` ainsi que `storeInCache: true` pour les requêtes Firecrawl.
Si le proxy est omis, Firecrawl utilise `auto` par défaut. `auto` réessaie avec des proxies furtifs si une tentative en mode basic échoue, ce qui peut consommer plus de crédits
que le scraping uniquement en mode basic.

<div id="how-web_fetch-uses-firecrawl">
  ## Comment `web_fetch` utilise Firecrawl
</div>

Ordre d&#39;extraction utilisé par `web_fetch` :

1. Readability (local)
2. Firecrawl (si configuré)
3. Nettoyage HTML de base (dernier recours)

Consultez la page [Outils web](/fr/tools/web) pour la configuration complète des outils web.