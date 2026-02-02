---
title: Firecrawl
summary: "Fallback Firecrawl per web_fetch (estrazione anti-bot + basata su cache)"
read_when:
  - Vuoi usare l'estrazione web basata su Firecrawl
  - Hai bisogno di una chiave api Firecrawl
  - Vuoi un'estrazione anti-bot per web_fetch
---

<div id="firecrawl">
  # Firecrawl
</div>

OpenClaw può usare **Firecrawl** come estrattore di fallback per `web_fetch`. È un servizio
di estrazione dei contenuti hostato che supporta l&#39;elusione dei bot e la memorizzazione in cache, il che aiuta
con siti molto basati su JS o pagine che bloccano le normali richieste HTTP.

<div id="get-an-api-key">
  ## Ottieni una chiave API
</div>

1. Crea un account Firecrawl e genera una chiave API.
2. Salvala nella configurazione oppure imposta `FIRECRAWL_API_KEY` come variabile d&#39;ambiente del Gateway.

<div id="configure-firecrawl">
  ## Configura Firecrawl
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

Note:

* `firecrawl.enabled` viene impostato su `true` per impostazione predefinita quando è presente una chiave API.
* `maxAgeMs` definisce l’età massima dei risultati memorizzati in cache (ms). Il valore predefinito è 2 giorni.

<div id="stealth-bot-circumvention">
  ## Elusione / aggiramento dei bot
</div>

Firecrawl espone un parametro **proxy mode** per aggirare i bot (`basic`, `stealth` o `auto`).
OpenClaw usa sempre `proxy: "auto"` più `storeInCache: true` per le richieste a Firecrawl.
Se il proxy viene omesso, Firecrawl usa `auto` come valore predefinito. Con `auto`, in caso di fallimento di un tentativo basic, il sistema ripete il tentativo usando proxy stealth, il che può consumare più crediti rispetto a uno scraping solo basic.

<div id="how-web_fetch-uses-firecrawl">
  ## Come `web_fetch` utilizza Firecrawl
</div>

Ordine di estrazione di `web_fetch`:

1. Readability (locale)
2. Firecrawl (se configurato)
3. Pulizia HTML di base (come ultima risorsa)

Consulta [Strumenti web](/it/tools/web) per la configurazione completa degli strumenti web.