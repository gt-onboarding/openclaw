---
title: Costi di utilizzo delle API
summary: "Monitorare cosa può generare costi, quali chiavi vengono usate e come visualizzare l'utilizzo"
read_when:
  - Vuoi capire quali funzionalità possono chiamare API a pagamento
  - Devi verificare chiavi, costi e modalità di visualizzazione dell'utilizzo
  - Stai spiegando la reportistica dei costi di /status o /usage
---

<div id="api-usage-costs">
  # Utilizzo delle API e costi
</div>

Questo documento elenca le **funzionalità che possono utilizzare chiavi API** e indica dove vengono contabilizzati i relativi costi. Si concentra sulle funzionalità di OpenClaw che possono generare consumo del provider o chiamate API a pagamento.

<div id="where-costs-show-up-chat-cli">
  ## Dove compaiono i costi (chat + CLI)
</div>

**Istantanea del costo per sessione**

* `/status` mostra il modello della sessione corrente, l&#39;utilizzo del contesto e gli ultimi token di risposta.
* Se il modello usa **API-key auth**, `/status` mostra anche il **costo stimato** per l&#39;ultima risposta.

**Piè di pagina dei costi per messaggio**

* `/usage full` aggiunge un piè di pagina di utilizzo a ogni risposta, incluso il **costo stimato** (solo API-key).
* `/usage tokens` mostra solo i token; i flussi OAuth nascondono il costo in dollari.

**Finestre di utilizzo nella CLI (quote del provider)**

* `openclaw status --usage` e `openclaw channels list` mostrano le **finestre di utilizzo** del provider
  (istantanee delle quote, non costi per singolo messaggio).

Consulta [Uso dei token e costi](/it/token-use) per dettagli ed esempi.

<div id="how-keys-are-discovered">
  ## Come vengono individuate le chiavi
</div>

OpenClaw può recuperare le credenziali da:

* **Profili di autenticazione** (per singolo agente, memorizzati in `auth-profiles.json`).
* **Variabili d&#39;ambiente** (ad es. `OPENAI_API_KEY`, `BRAVE_API_KEY`, `FIRECRAWL_API_KEY`).
* **Configurazione** (`models.providers.*.apiKey`, `tools.web.search.*`, `tools.web.fetch.firecrawl.*`,
  `memorySearch.*`, `talk.apiKey`).
* **Abilità** (`skills.entries.<name>.apiKey`) che possono esportare chiavi nell&#39;ambiente del processo dell&#39;abilità.

<div id="features-that-can-spend-keys">
  ## Funzionalità che possono utilizzare le chiavi
</div>

<div id="1-core-model-responses-chat-tools">
  ### 1) Risposte del modello core (chat + strumenti)
</div>

Ogni risposta o chiamata a uno strumento usa il **provider di modello corrente** (OpenAI, Anthropic, ecc.). Questa è la
fonte principale di utilizzo e costi.

Consulta [Modelli](/it/providers/models) per la configurazione dei prezzi e [Uso dei token e costi](/it/token-use) per i dettagli sulla visualizzazione.

<div id="2-media-understanding-audioimagevideo">
  ### 2) Comprensione dei media (audio/immagine/video)
</div>

I media in ingresso possono essere riassunti/trascritti prima che venga generata la risposta. Per farlo vengono utilizzate le API dei modelli/provider.

* Audio: OpenAI / Groq / Deepgram (ora **abilitato automaticamente** quando sono presenti le chiavi).
* Immagine: OpenAI / Anthropic / Google.
* Video: Google.

Vedi [Comprensione dei media](/it/nodes/media-understanding).

<div id="3-memory-embeddings-semantic-search">
  ### 3) Embedding della memoria + ricerca semantica
</div>

La ricerca semantica sulla memoria utilizza **API di embedding** quando è configurata per l&#39;uso con provider remoti:

* `memorySearch.provider = "openai"` → embedding OpenAI
* `memorySearch.provider = "gemini"` → embedding Gemini
* Fallback opzionale a OpenAI se gli embedding locali falliscono

Puoi mantenerla locale con `memorySearch.provider = "local"` (nessun utilizzo di API).

Vedi [Memory](/it/concepts/memory).

<div id="4-web-search-tool-brave-perplexity-via-openrouter">
  ### 4) Strumento di ricerca web (Brave / Perplexity tramite OpenRouter)
</div>

`web_search` utilizza chiavi API e può generare costi di utilizzo:

* **Brave Search API**: `BRAVE_API_KEY` oppure `tools.web.search.apiKey`
* **Perplexity** (tramite OpenRouter): `PERPLEXITY_API_KEY` oppure `OPENROUTER_API_KEY`

**Piano gratuito Brave (molto generoso):**

* **2.000 richieste/mese**
* **1 richiesta/secondo**
* **Carta di credito richiesta** per la verifica (nessun addebito salvo passaggio a un piano superiore)

Vedi [Strumenti web](/it/tools/web).

<div id="5-web-fetch-tool-firecrawl">
  ### 5) Strumento di web fetch (Firecrawl)
</div>

`web_fetch` può chiamare **Firecrawl** quando è presente una chiave API:

* `FIRECRAWL_API_KEY` oppure `tools.web.fetch.firecrawl.apiKey`

Se Firecrawl non è configurato, lo strumento ricorre al fetch diretto + readability (nessuna API a pagamento).

Consulta [Strumenti web](/it/tools/web).

<div id="6-provider-usage-snapshots-statushealth">
  ### 6) Snapshot sull’utilizzo dei provider (stato/integrità)
</div>

Alcuni comandi di stato chiamano **endpoint di utilizzo dei provider** per mostrare le finestre di quota o lo stato di integrità dell’autenticazione.
Si tratta in genere di chiamate a basso volume che comunque invocano le API dei provider:

* `openclaw status --usage`
* `openclaw models status --json`

Vedi [Models CLI](/it/cli/models).

<div id="7-compaction-safeguard-summarization">
  ### 7) Riepilogo della salvaguardia di compattazione
</div>

La salvaguardia di compattazione può riepilogare la cronologia della sessione usando il **modello corrente**, il che comporta chiamate alle API del provider durante l&#39;esecuzione.

Consulta [Gestione delle sessioni + compattazione](/it/reference/session-management-compaction).

<div id="8-model-scan-probe">
  ### 8) Scansione / sondaggio dei modelli
</div>

`openclaw models scan` può sondare i modelli OpenRouter e utilizza `OPENROUTER_API_KEY` quando
la funzionalità di sondaggio è abilitata.

Vedi [Models CLI](/it/cli/models).

<div id="9-talk-speech">
  ### 9) Talk (voce)
</div>

La modalità Talk può invocare **ElevenLabs** se configurata:

* `ELEVENLABS_API_KEY` o `talk.apiKey`

Vedi la [modalità Talk](/it/nodes/talk).

<div id="10-skills-third-party-apis">
  ### 10) Abilità (API di terze parti)
</div>

Le abilità possono salvare `apiKey` in `skills.entries.<name>.apiKey`. Se un&#39;abilità utilizza quella chiave per API esterne,
potrebbe generare costi a seconda del provider dell&#39;abilità.

Consulta [Abilità](/it/tools/skills).