---
title: Firecrawl
summary: "Firecrawl-Fallback für web_fetch (Anti-Bot + zwischengespeicherte Extraktion)"
read_when:
  - Du möchtest Firecrawl-gestützte Web-Extraktion
  - Du benötigst einen Firecrawl-API-Schlüssel
  - Du möchtest Anti-Bot-Extraktion für web_fetch
---

<div id="firecrawl">
  # Firecrawl
</div>

OpenClaw kann **Firecrawl** als Fallback-Extractor für `web_fetch` verwenden. Es ist ein gehosteter Dienst zur Inhaltsextraktion, der die Umgehung von Bot-Sperren und Caching unterstützt, was bei JS-lastigen Seiten oder Seiten hilft, die normale HTTP-Fetch-Aufrufe blockieren.

<div id="get-an-api-key">
  ## API-Schlüssel erhalten
</div>

1. Lege ein Firecrawl-Konto an und erstelle einen API-Schlüssel.
2. Speichere ihn in der Konfiguration oder setze `FIRECRAWL_API_KEY` in der Gateway-Umgebung.

<div id="configure-firecrawl">
  ## Firecrawl konfigurieren
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

Hinweise:

* `firecrawl.enabled` ist standardmäßig auf true gesetzt, wenn ein API-Schlüssel konfiguriert ist.
* `maxAgeMs` legt fest, wie alt zwischengespeicherte Ergebnisse maximal sein dürfen (in ms). Der Standardwert beträgt 2 Tage.

<div id="stealth-bot-circumvention">
  ## Stealth-/Bot-Umgehung
</div>

Firecrawl stellt einen **Proxy-Modus**-Parameter zur Bot-Umgehung bereit (`basic`, `stealth` oder `auto`).
OpenClaw verwendet für Firecrawl-Anfragen immer `proxy: "auto"` plus `storeInCache: true`.
Wenn `proxy` weggelassen wird, verwendet Firecrawl standardmäßig `auto`. `auto` versucht bei einem Fehlschlag mit `basic` erneut einen Abruf mit Stealth-Proxys, was mehr Credits
als reines `basic`-Scraping verbrauchen kann.

<div id="how-web_fetch-uses-firecrawl">
  ## Wie `web_fetch` Firecrawl verwendet
</div>

Extraktionsreihenfolge von `web_fetch`:

1. Readability (lokal)
2. Firecrawl (falls konfiguriert)
3. Grundlegende HTML-Bereinigung (letzter Fallback)

Siehe [Web-Tools](/de/tools/web) für die vollständige Web-Tool-Einrichtung.