---
title: Web
summary: "Strumenti di ricerca e fetch sul web (Brave Search API, Perplexity direct/OpenRouter)"
read_when:
  - Vuoi abilitare web_search o web_fetch
  - Hai bisogno di configurare la chiave API di Brave Search
  - Vuoi usare Perplexity Sonar per la ricerca sul web
---

<div id="web-tools">
  # Strumenti web
</div>

OpenClaw fornisce due strumenti web leggeri:

* `web_search` — Cerca sul web tramite Brave Search API (predefinito) o Perplexity Sonar (diretto o via OpenRouter).
* `web_fetch` — HTTP fetch + estrazione in formato leggibile (HTML → markdown/testo).

Questi **non** sono strumenti di automazione del browser. Per siti fortemente basati su JS o che richiedono login, usa lo
[strumento Browser](/it/tools/browser).

<div id="how-it-works">
  ## Come funziona
</div>

* `web_search` chiama il provider che hai configurato e restituisce i risultati.
  * **Brave** (predefinito): restituisce risultati strutturati (titolo, URL, snippet).
  * **Perplexity**: restituisce risposte elaborate dall&#39;AI con citazioni da ricerche web in tempo reale.
* I risultati vengono memorizzati in cache per ogni query per 15 minuti (configurabile).
* `web_fetch` esegue una semplice richiesta HTTP GET ed estrae il contenuto leggibile
  (HTML → markdown/testo). **Non** esegue JavaScript.
* `web_fetch` è abilitato per impostazione predefinita (a meno che non sia esplicitamente disabilitato).

<div id="choosing-a-search-provider">
  ## Scelta di un provider di ricerca
</div>

| Provider                | Vantaggi                                                 | Contro                                     | API Key                                     |
| ----------------------- | -------------------------------------------------------- | ------------------------------------------ | ------------------------------------------- |
| **Brave** (predefinito) | Risultati veloci e strutturati, piano gratuito           | Risultati di ricerca tradizionali          | `BRAVE_API_KEY`                             |
| **Perplexity**          | Risposte generate dall&#39;AI, citazioni, in tempo reale | Richiede accesso a Perplexity o OpenRouter | `OPENROUTER_API_KEY` o `PERPLEXITY_API_KEY` |

Consulta [Brave Search setup](/it/brave-search) e [Perplexity Sonar](/it/perplexity) per dettagli specifici del provider.

Imposta il provider nella configurazione:

```json5
{
  tools: {
    web: {
      search: {
        provider: "brave"  // oppure "perplexity"
      }
    }
  }
}
```

Esempio: passa a Perplexity Sonar (api diretta):

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
  ## Ottenere una chiave API Brave
</div>

1. Crea un account Brave Search API su https://brave.com/search/api/
2. Nella dashboard, scegli il piano **Data for Search** (non “Data for AI”) e genera una chiave API.
3. Esegui `openclaw configure --section web` per salvare la chiave nella configurazione (consigliato), oppure imposta `BRAVE_API_KEY` nel tuo ambiente.

Brave offre un piano gratuito e piani a pagamento; controlla il portale Brave API per i limiti e i prezzi aggiornati.

<div id="where-to-set-the-key-recommended">
  ### Dove impostare la chiave (consigliato)
</div>

**Consigliato:** esegui `openclaw configure --section web`. Salva la chiave in
`~/.openclaw/openclaw.json` sotto `tools.web.search.apiKey`.

**Alternativa tramite variabile d&#39;ambiente:** imposta `BRAVE_API_KEY` nell&#39;ambiente
del processo Gateway. Per un&#39;installazione del Gateway, inseriscila in `~/.openclaw/.env`
(o nell&#39;ambiente del tuo servizio). Consulta [Variabili d&#39;ambiente](/it/help/faq#how-does-openclaw-load-environment-variables).

<div id="using-perplexity-direct-or-via-openrouter">
  ## Utilizzo di Perplexity (direttamente o tramite OpenRouter)
</div>

I modelli Perplexity Sonar hanno funzionalità di ricerca web integrate e restituiscono risposte generate dall&#39;IA con citazioni. Puoi utilizzarli tramite OpenRouter (non è richiesta una carta di credito - supporta pagamenti in crypto/prepagati).

<div id="getting-an-openrouter-api-key">
  ### Ottenere una chiave API di OpenRouter
</div>

1. Crea un account su https://openrouter.ai/
2. Aggiungi dei crediti (supporta criptovalute, carte prepagate o carte di credito)
3. Genera una chiave API nelle impostazioni del tuo account

<div id="setting-up-perplexity-search">
  ### Configurazione della ricerca Perplexity
</div>

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "perplexity",
        perplexity: {
          // Chiave API (opzionale se è impostata OPENROUTER_API_KEY o PERPLEXITY_API_KEY)
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

**Alternativa tramite variabile d&#39;ambiente:** imposta `OPENROUTER_API_KEY` o `PERPLEXITY_API_KEY`
nell&#39;ambiente del Gateway. Per un&#39;installazione del Gateway, impostala in `~/.openclaw/.env`.

Se non è impostata alcuna URL di base, OpenClaw sceglie un valore predefinito in base
all&#39;origine della chiave API:

* `PERPLEXITY_API_KEY` o `pplx-...` → `https://api.perplexity.ai`
* `OPENROUTER_API_KEY` o `sk-or-...` → `https://openrouter.ai/api/v1`
* Formati di chiave sconosciuti → OpenRouter (fallback sicuro)

<div id="available-perplexity-models">
  ### Modelli Perplexity disponibili
</div>

| Model | Description | Best for |
|-------|-------------|----------|
| `perplexity/sonar` | Domande e risposte rapide con ricerca sul web | Ricerche rapide |
| `perplexity/sonar-pro` (default) | Ragionamento a più passaggi con ricerca sul web | Domande complesse |
| `perplexity/sonar-reasoning-pro` | Analisi del ragionamento passo-passo | Ricerca approfondita |

<div id="web_search">
  ## web_search
</div>

Cerca sul web utilizzando il provider configurato.

<div id="requirements">
  ### Requisiti
</div>

* `tools.web.search.enabled` non deve essere `false` (impostazione predefinita: abilitato)
* Chiave API per il provider scelto:
  * **Brave**: `BRAVE_API_KEY` oppure `tools.web.search.apiKey`
  * **Perplexity**: `OPENROUTER_API_KEY`, `PERPLEXITY_API_KEY` oppure `tools.web.search.perplexity.apiKey`

<div id="config">
  ### Configurazione
</div>

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE", // opzionale se BRAVE_API_KEY è impostato
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15
      }
    }
  }
}
```

<div id="tool-parameters">
  ### Parametri dello strumento
</div>

* `query` (obbligatorio)
* `count` (1–10; valore predefinito dalla configurazione)
* `country` (opzionale): codice paese di 2 lettere per risultati specifici per regione (ad es. &quot;DE&quot;, &quot;US&quot;, &quot;ALL&quot;). Se omesso, Brave seleziona la sua regione predefinita.
* `search_lang` (opzionale): codice lingua ISO per i risultati di ricerca (ad es. &quot;de&quot;, &quot;en&quot;, &quot;fr&quot;)
* `ui_lang` (opzionale): codice lingua ISO per gli elementi della UI
* `freshness` (opzionale, solo Brave): filtra per tempo di scoperta (`pd`, `pw`, `pm`, `py` o `YYYY-MM-DDtoYYYY-MM-DD`)

**Esempi:**

```javascript
// German-specific search
await web_search({
  query: "TV online schauen",
  count: 10,
  country: "DE",
  search_lang: "de"
});

// French search with French UI
await web_search({
  query: "actualités",
  country: "FR",
  search_lang: "fr",
  ui_lang: "fr"
});

// Risultati recenti (ultima settimana)
await web_search({
  query: "TMBG interview",
  freshness: "pw"
});
```

<div id="web_fetch">
  ## web_fetch
</div>

Recupera un URL e ne estrae il contenuto leggibile.

<div id="requirements">
  ### Requisiti
</div>

* L&#39;impostazione `tools.web.fetch.enabled` non deve essere `false` (valore predefinito: abilitato)
* Fallback opzionale con Firecrawl: configura `tools.web.fetch.firecrawl.apiKey` oppure `FIRECRAWL_API_KEY`.

<div id="config">
  ### Configurazione
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
          apiKey: "FIRECRAWL_API_KEY_HERE", // opzionale se è impostato FIRECRAWL_API_KEY
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
  ### Parametri dello strumento
</div>

* `url` (obbligatorio, solo http/https)
* `extractMode` (`markdown` | `text`)
* `maxChars` (tronca le pagine lunghe)

Note:

* `web_fetch` utilizza prima Readability (estrazione del contenuto principale), poi Firecrawl (se configurato). Se entrambi falliscono, lo strumento restituisce un errore.
* Le richieste Firecrawl usano la modalità di elusione dei bot e memorizzano i risultati in cache per impostazione predefinita.
* `web_fetch` invia per impostazione predefinita uno User-Agent simile a Chrome e `Accept-Language`; sovrascrivi `userAgent` se necessario.
* `web_fetch` blocca nomi host privati/interni e ricontrolla i redirect (limita con `maxRedirects`).
* `web_fetch` effettua un&#39;estrazione in modalità best effort; alcuni siti richiederanno lo strumento browser.
* Vedi [Firecrawl](/it/tools/firecrawl) per la configurazione delle chiavi e i dettagli del servizio.
* Le risposte sono messe in cache (15 minuti per impostazione predefinita) per ridurre i fetch ripetuti.
* Se usi profili/lista di autorizzati per gli strumenti, aggiungi `web_search`/`web_fetch` o `group:web`.
* Se manca la chiave Brave, `web_search` restituisce un breve suggerimento di configurazione con un link alla documentazione.