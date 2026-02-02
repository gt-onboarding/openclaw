---
title: Brave Search
summary: "Configurazione della Brave Search API per web_search"
read_when:
  - Vuoi usare Brave Search per web_search
  - Ti serve una BRAVE_API_KEY o informazioni sul piano
---

<div id="brave-search-api">
  # Brave Search API
</div>

OpenClaw utilizza Brave Search come provider predefinito per `web_search`.

<div id="get-an-api-key">
  ## Ottieni una chiave API
</div>

1. Crea un account Brave Search API su https://brave.com/search/api/
2. Dalla dashboard, scegli il piano **Data for Search** e genera una chiave API.
3. Salva la chiave nella configurazione (consigliato) oppure imposta `BRAVE_API_KEY` nell&#39;ambiente del Gateway.

<div id="config-example">
  ## Esempio di configurazione
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
  ## Note
</div>

* Il piano Data for AI **non** è compatibile con `web_search`.
* Brave offre un piano gratuito più piani a pagamento; consulta il portale API di Brave per i limiti aggiornati.

Vedi [Strumenti web](/it/tools/web) per la configurazione completa di `web_search`.