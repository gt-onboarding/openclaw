---
title: Web
summary: "Web-Such- und Fetch-Tools (Brave-Search-API, Perplexity Direct/OpenRouter)"
read_when:
  - Du möchtest web_search oder web_fetch aktivieren
  - Du musst einen Brave-Search-API-Schlüssel einrichten
  - Du möchtest Perplexity Sonar für die Websuche verwenden
---

<div id="web-tools">
  # Web-Tools
</div>

OpenClaw stellt zwei schlanke Web-Tools bereit:

* `web_search` — Durchsuche das Web über die Brave Search API (Standard) oder Perplexity Sonar (direkt oder über OpenRouter).
* `web_fetch` — HTTP-Fetch + lesefreundliche Extraktion (HTML → Markdown/Text).

Das ist **keine** Browser-Automatisierung. Für JavaScript-lastige Seiten oder Logins verwende das
[Browser-Tool](/de/tools/browser).

<div id="how-it-works">
  ## Funktionsweise
</div>

* `web_search` ruft deinen konfigurierten Anbieter auf und gibt Ergebnisse zurück.
  * **Brave** (Standard): gibt strukturierte Ergebnisse zurück (Titel, URL, Ausschnitt).
  * **Perplexity**: gibt KI-synthetisierte Antworten mit Quellenangaben aus der Echtzeit-Websuche zurück.
* Ergebnisse werden pro Anfrage für 15 Minuten zwischengespeichert (konfigurierbar).
* `web_fetch` führt ein einfaches HTTP-GET aus und extrahiert lesbare Inhalte
  (HTML → Markdown/Text). Es führt **kein** JavaScript aus.
* `web_fetch` ist standardmäßig aktiviert (sofern nicht explizit deaktiviert).

<div id="choosing-a-search-provider">
  ## Auswahl eines Suchanbieters
</div>

| Anbieter             | Vorteile                                                         | Nachteile                                     | API-Schlüssel                                  |
| -------------------- | ---------------------------------------------------------------- | --------------------------------------------- | ---------------------------------------------- |
| **Brave** (Standard) | Schnelle, strukturierte Ergebnisse, kostenloses Kontingent       | Traditionelle Suchergebnisse                  | `BRAVE_API_KEY`                                |
| **Perplexity**       | KI-synthetisierte Antworten, Quellenangaben, Echtzeit-Ergebnisse | Erfordert Perplexity- oder OpenRouter-Zugriff | `OPENROUTER_API_KEY` oder `PERPLEXITY_API_KEY` |

Siehe [Brave Search Einrichtung](/de/brave-search) und [Perplexity Sonar](/de/perplexity) für anbieterspezifische Details.

Lege den Anbieter in der Konfiguration fest:

```json5
{
  tools: {
    web: {
      search: {
        provider: "brave"  // oder "perplexity"
      }
    }
  }
}
```

Beispiel: Auf Perplexity Sonar umstellen (direkte api):

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
  ## Brave-API-Schlüssel erhalten
</div>

1. Erstelle ein Brave-Search-API-Konto auf https://brave.com/search/api/
2. Wähle im Dashboard den **Data for Search**-Tarif (nicht „Data for AI“) und generiere einen API-Schlüssel.
3. Führe `openclaw configure --section web` aus, um den Schlüssel in der Konfiguration zu speichern (empfohlen), oder setze `BRAVE_API_KEY` als Umgebungsvariable.

Brave bietet einen kostenlosen Tarif sowie kostenpflichtige Pläne; informiere dich im Brave-API-Portal über die aktuellen Limits und Preise.

<div id="where-to-set-the-key-recommended">
  ### Wo du den Schlüssel setzt (empfohlen)
</div>

**Empfohlen:** Führe `openclaw configure --section web` aus. Der Schlüssel wird in
`~/.openclaw/openclaw.json` unter `tools.web.search.apiKey` gespeichert.

**Alternative über Umgebungsvariable:** Setze `BRAVE_API_KEY` in der Umgebung des Gateway-Prozesses. Für eine Gateway-Installation speicherst du ihn in `~/.openclaw/.env` (oder in deiner
Service-Umgebung). Siehe [Umgebungsvariablen](/de/help/faq#how-does-openclaw-load-environment-variables).

<div id="using-perplexity-direct-or-via-openrouter">
  ## Verwendung von Perplexity (direkt oder über OpenRouter)
</div>

Perplexity-Sonar-Modelle verfügen über integrierte Websuchfunktionen und liefern KI-synthetisierte Antworten mit Quellenangaben. Du kannst sie über OpenRouter nutzen (keine Kreditkarte erforderlich – unterstützt Krypto/Prepaid).

<div id="getting-an-openrouter-api-key">
  ### Abrufen eines OpenRouter-API-Schlüssels
</div>

1. Erstelle ein Konto unter https://openrouter.ai/
2. Lade Guthaben auf (unterstützt Kryptowährungen, Prepaid-Karten oder Kreditkarten)
3. Generiere einen API-Schlüssel in deinen Kontoeinstellungen

<div id="setting-up-perplexity-search">
  ### Perplexity-Suche einrichten
</div>

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "perplexity",
        perplexity: {
          // API-Schlüssel (optional, falls OPENROUTER_API_KEY oder PERPLEXITY_API_KEY gesetzt ist)
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

**Alternative über Umgebungsvariable:** setze `OPENROUTER_API_KEY` oder `PERPLEXITY_API_KEY` in der Gateway-Umgebung. Für eine Gateway-Installation speichere sie in `~/.openclaw/.env`.

Wenn keine Basis-URL gesetzt ist, wählt OpenClaw einen Standardwert auf Basis der Quelle des API-Schlüssels:

* `PERPLEXITY_API_KEY` oder `pplx-...` → `https://api.perplexity.ai`
* `OPENROUTER_API_KEY` oder `sk-or-...` → `https://openrouter.ai/api/v1`
* Unbekannte Schlüssel-Formate → OpenRouter (sichere Standardvorgabe)

<div id="available-perplexity-models">
  ### Verfügbare Perplexity-Modelle
</div>

| Modell | Beschreibung | Am besten geeignet für |
|-------|-------------|------------------------|
| `perplexity/sonar` | Schnelle Q&amp;A mit Websuche | Schnelles Nachschlagen |
| `perplexity/sonar-pro` (Standard) | Mehrschrittiges Schlussfolgern mit Websuche | Komplexe Fragen |
| `perplexity/sonar-reasoning-pro` | Chain-of-Thought-Analyse | Tiefgehende Recherche |

<div id="web_search">
  ## web_search
</div>

Durchsuche das Web mit deinem konfigurierten Anbieter.

<div id="requirements">
  ### Anforderungen
</div>

* `tools.web.search.enabled` darf nicht auf `false` gesetzt sein (Standard: aktiviert)
* API-Schlüssel für deinen gewählten Anbieter:
  * **Brave**: `BRAVE_API_KEY` oder `tools.web.search.apiKey`
  * **Perplexity**: `OPENROUTER_API_KEY`, `PERPLEXITY_API_KEY` oder `tools.web.search.perplexity.apiKey`

<div id="config">
  ### Konfiguration
</div>

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE", // optional, wenn BRAVE_API_KEY gesetzt ist
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15
      }
    }
  }
}
```

<div id="tool-parameters">
  ### Tool-Parameter
</div>

* `query` (erforderlich)
* `count` (1–10; Standardwert aus der Konfiguration)
* `country` (optional): 2-stelliger Ländercode für regionsspezifische Ergebnisse (z. B. &quot;DE&quot;, &quot;US&quot;, &quot;ALL&quot;). Wenn weggelassen, wählt Brave seine Standardregion.
* `search_lang` (optional): ISO-Sprachcode für Suchergebnisse (z. B. &quot;de&quot;, &quot;en&quot;, &quot;fr&quot;)
* `ui_lang` (optional): ISO-Sprachcode für UI-Elemente
* `freshness` (optional, nur Brave): Filter nach Zeitpunkt der Entdeckung (`pd`, `pw`, `pm`, `py` oder `YYYY-MM-DDtoYYYY-MM-DD`)

**Beispiele:**

```javascript
// Deutschlandspezifische Suche
await web_search({
  query: "TV online schauen",
  count: 10,
  country: "DE",
  search_lang: "de"
});

// Französische Suche mit französischer UI
await web_search({
  query: "actualités",
  country: "FR",
  search_lang: "fr",
  ui_lang: "fr"
});

// Aktuelle Ergebnisse (letzte Woche)
await web_search({
  query: "TMBG interview",
  freshness: "pw"
});
```

<div id="web_fetch">
  ## web_fetch
</div>

Ruft eine URL auf und extrahiert den lesbaren Inhalt.

<div id="requirements">
  ### Anforderungen
</div>

* `tools.web.fetch.enabled` darf nicht auf `false` gesetzt sein (Standard: aktiviert)
* Optionaler Firecrawl-Fallback: Setzen Sie `tools.web.fetch.firecrawl.apiKey` oder `FIRECRAWL_API_KEY`.

<div id="config">
  ### Konfiguration
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
          apiKey: "FIRECRAWL_API_KEY_HERE", // optional, wenn FIRECRAWL_API_KEY gesetzt ist
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
  ### Tool-Parameter
</div>

* `url` (erforderlich, nur http/https)
* `extractMode` (`markdown` | `text`)
* `maxChars` (lange Seiten kürzen)

Hinweise:

* `web_fetch` nutzt zuerst Readability (Extraktion des Hauptinhalts), dann Firecrawl (falls konfiguriert). Wenn beides fehlschlägt, gibt das Tool einen Fehler zurück.
* Firecrawl-Anfragen verwenden standardmäßig einen Bot-Umgehungsmodus und speichern Ergebnisse im Cache.
* `web_fetch` sendet standardmäßig einen Chrome-ähnlichen User-Agent und `Accept-Language`; überschreibe `userAgent` bei Bedarf.
* `web_fetch` blockiert private/interne Hostnamen und prüft Weiterleitungen erneut (begrenze mit `maxRedirects`).
* `web_fetch` ist eine „Best-Effort“-Extraktion; einige Websites benötigen das Browser-Tool.
* Siehe [Firecrawl](/de/tools/firecrawl) für Schlüsselkonfiguration und Details zum Dienst.
* Antworten werden zwischengespeichert (standardmäßig 15 Minuten), um wiederholte Abrufe zu reduzieren.
* Wenn du Tool-Profile/Allowlists verwendest, füge `web_search`/`web_fetch` oder `group:web` hinzu.
* Wenn der Brave-Schlüssel fehlt, gibt `web_search` einen kurzen Setup-Hinweis mit einem Link zur Doku zurück.