---
title: Perplexity
summary: "Perplexity Sonar-Konfiguration für web_search"
read_when:
  - Du möchtest Perplexity Sonar für die Websuche verwenden
  - Du benötigst PERPLEXITY_API_KEY oder eine OpenRouter-Konfiguration
---

<div id="perplexity-sonar">
  # Perplexity Sonar
</div>

OpenClaw kann Perplexity Sonar für das Tool `web_search` verwenden. Du kannst dich
entweder direkt über die Perplexity-API oder über OpenRouter verbinden.

<div id="api-options">
  ## API-Optionen
</div>

<div id="perplexity-direct">
  ### Perplexity (direkt)
</div>

* Basis-URL: https://api.perplexity.ai
* Umgebungsvariable: `PERPLEXITY_API_KEY`

<div id="openrouter-alternative">
  ### OpenRouter (Alternative)
</div>

* Basis-URL: https://openrouter.ai/api/v1
* Umgebungsvariable: `OPENROUTER_API_KEY`
* Unterstützt Prepaid- und Krypto-Guthaben.

<div id="config-example">
  ## Beispielkonfiguration
</div>

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

<div id="switching-from-brave">
  ## Wechsel von Brave
</div>

```json5
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai"
        }
      }
    }
  }
}
```

Wenn sowohl `PERPLEXITY_API_KEY` als auch `OPENROUTER_API_KEY` gesetzt sind, setze
`tools.web.search.perplexity.baseUrl` (oder `tools.web.search.perplexity.apiKey`),
um die Zuordnung eindeutig festzulegen.

Wenn keine Base-URL gesetzt ist, wählt OpenClaw einen Standardwert basierend auf der Quelle des API-Schlüssels:

* `PERPLEXITY_API_KEY` oder `pplx-...` → direkte Perplexity-API (`https://api.perplexity.ai`)
* `OPENROUTER_API_KEY` oder `sk-or-...` → OpenRouter (`https://openrouter.ai/api/v1`)
* Unbekannte Schlüsselformate → OpenRouter (sichere Fallback-Option)

<div id="models">
  ## Modelle
</div>

* `perplexity/sonar` — schnelle Q&amp;A-Antworten mit Websuche
* `perplexity/sonar-pro` (Standard) — mehrstufiges Reasoning + Websuche
* `perplexity/sonar-reasoning-pro` — tiefgehende Recherche

Siehe [Web-Tools](/de/tools/web) für die vollständige web&#95;search-Konfiguration.