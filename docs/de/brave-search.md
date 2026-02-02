---
title: Brave Search
summary: "Brave Search API-Einrichtung für web_search"
read_when:
  - Du möchtest Brave Search für web_search verwenden
  - Du benötigst einen BRAVE_API_KEY oder Tarifdetails
---

<div id="brave-search-api">
  # Brave Search API
</div>

OpenClaw verwendet Brave Search als Standard-Anbieter für `web_search`.

<div id="get-an-api-key">
  ## API-Schlüssel abrufen
</div>

1. Erstelle ein Brave Search API-Konto auf https://brave.com/search/api/
2. Wähle im Dashboard den **Data for Search**-Tarif aus und erzeugte einen API-Schlüssel.
3. Speichere den Schlüssel in der Konfiguration (empfohlen) oder setze `BRAVE_API_KEY` in der Gateway-Umgebung.

<div id="config-example">
  ## Konfigurationsbeispiel
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
  ## Hinweise
</div>

* Der „Data for AI“-Plan ist **nicht** mit `web_search` kompatibel.
* Brave bietet einen kostenlosen Tarif sowie kostenpflichtige Pläne; siehe das Brave-API-Portal für die aktuellen Limits.

Siehe [Web-Tools](/de/tools/web) für die vollständige `web_search`-Konfiguration.