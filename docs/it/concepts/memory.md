---
title: Memoria
summary: "Come funziona la memoria di OpenClaw (file dello spazio di lavoro + flush automatico della memoria)"
read_when:
  - Vuoi conoscere l'organizzazione dei file di memoria e il relativo flusso di lavoro
  - Vuoi regolare il flush automatico della memoria prima della compattazione
---

<div id="memory">
  # Memoria
</div>

La memoria di OpenClaw è **semplice Markdown nello spazio di lavoro dell&#39;agente**. I file sono la
fonte di verità; il modello &quot;ricorda&quot; solo ciò che viene scritto su disco.

Gli strumenti di ricerca della memoria sono forniti dal plugin di memoria attivo (predefinito:
`memory-core`). Disattiva i plugin di memoria con `plugins.slots.memory = "none"`.

<div id="memory-files-markdown">
  ## File di memoria (Markdown)
</div>

Il layout predefinito dello spazio di lavoro utilizza due livelli di memoria:

* `memory/YYYY-MM-DD.md`
  * Registro giornaliero (solo in append).
  * Leggi i file di oggi e di ieri all&#39;avvio della sessione.
* `MEMORY.md` (opzionale)
  * Memoria a lungo termine curata manualmente.
  * **Carica solo nella sessione principale, privata** (mai nei contesti di gruppo).

Questi file si trovano all&#39;interno dello spazio di lavoro (`agents.defaults.workspace`, predefinito
`~/.openclaw/workspace`). Vedi [Spazio di lavoro dell&#39;agente](/it/concepts/agent-workspace) per il layout completo.

<div id="when-to-write-memory">
  ## Quando scrivere in memoria
</div>

* Decisioni, preferenze e informazioni stabili vanno in `MEMORY.md`.
* Note quotidiane e contesto corrente vanno in `memory/YYYY-MM-DD.md`.
* Se qualcuno dice &quot;ricorda questo&quot;, scrivilo (non tenerlo in RAM).
* Questa parte è ancora in evoluzione. È utile ricordare al modello di salvare le informazioni in memoria; saprà cosa fare.
* Se vuoi che qualcosa rimanga, **chiedi al bot di scriverlo** in memoria.

<div id="automatic-memory-flush-pre-compaction-ping">
  ## Svuotamento automatico della memoria (ping pre-compattazione)
</div>

Quando una sessione è **vicina all&#39;auto-compattazione**, OpenClaw attiva un **turno agentico silenzioso** che ricorda al modello di scrivere nella memoria persistente **prima** che il contesto venga compattato. I prompt predefiniti dicono esplicitamente che il modello *può rispondere*, ma di solito `NO_REPLY` è la risposta corretta, così l&#39;utente non vede mai questo turno.

Questo è controllato da `agents.defaults.compaction.memoryFlush`:

```json5
{
  agents: {
    defaults: {
      compaction: {
        reserveTokensFloor: 20000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 4000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Scrivi eventuali note permanenti in memory/YYYY-MM-DD.md; rispondi con NO_REPLY se non c'è nulla da memorizzare."
        }
      }
    }
  }
}
```

Dettagli:

* **Soglia soft**: il flush viene attivato quando la stima dei token della sessione supera
  `contextWindow - reserveTokensFloor - softThresholdTokens`.
* **Silenzioso** per impostazione predefinita: i prompt includono `NO_REPLY`, quindi non viene inviato nulla.
* **Due prompt**: un prompt utente più un prompt di sistema che aggiunge il promemoria.
* **Un solo flush per ciclo di compattazione** (tracciato in `sessions.json`).
* **Lo spazio di lavoro deve essere scrivibile**: se la sessione viene eseguita in sandbox con
  `workspaceAccess: "ro"` o `"none"`, il flush viene saltato.

Per l&#39;intero ciclo di vita della compattazione, vedi
[Gestione delle sessioni + compattazione](/it/reference/session-management-compaction).

<div id="vector-memory-search">
  ## Ricerca nella memoria vettoriale
</div>

OpenClaw può costruire un piccolo indice vettoriale su `MEMORY.md` e `memory/*.md` (più
eventuali directory o file aggiuntivi che decidi di includere), in modo che le query semantiche possano trovare note correlate
anche quando la formulazione è diversa.

Valori predefiniti:

* Abilitata per impostazione predefinita.
* Monitora i file di memoria per rilevare modifiche (con debounce).
* Usa embedding remoti per impostazione predefinita. Se `memorySearch.provider` non è impostato, OpenClaw seleziona automaticamente:
  1. `local` se è configurato un `memorySearch.local.modelPath` e il file esiste.
  2. `openai` se è disponibile una chiave OpenAI.
  3. `gemini` se è disponibile una chiave Gemini.
  4. In caso contrario, la ricerca nella memoria rimane disabilitata finché non viene configurata.
* La modalità locale usa node-llama-cpp e può richiedere `pnpm approve-builds`.
* Usa sqlite-vec (quando disponibile) per accelerare la ricerca vettoriale all&#39;interno di SQLite.

Gli embedding remoti **richiedono** una chiave API per il provider di embedding. OpenClaw
ricava le chiavi dai profili di autenticazione, da `models.providers.*.apiKey` o dalle variabili
d&#39;ambiente. L’OAuth di Codex copre solo chat/completions e **non** fornisce
embedding per la ricerca nella memoria. Per Gemini, usa `GEMINI_API_KEY` o
`models.providers.google.apiKey`. Quando utilizzi un endpoint personalizzato compatibile con OpenAI,
imposta `memorySearch.remote.apiKey` (e opzionalmente `memorySearch.remote.headers`).

<div id="additional-memory-paths">
  ### Percorsi di memoria aggiuntivi
</div>

Se vuoi indicizzare file Markdown al di fuori della struttura predefinita dello spazio di lavoro, aggiungi
percorsi espliciti:

```json5
agents: {
  defaults: {
    memorySearch: {
      extraPaths: ["../team-docs", "/srv/shared-notes/overview.md"]
    }
  }
}
```

Note:

* I percorsi possono essere assoluti o relativi allo spazio di lavoro.
* Le directory vengono scansionate ricorsivamente alla ricerca di file `.md`.
* Vengono indicizzati solo i file Markdown.
* I collegamenti simbolici (file o directory) vengono ignorati.

<div id="gemini-embeddings-native">
  ### Gemini embeddings (native)
</div>

Imposta il provider su `gemini` per usare direttamente le API di embedding Gemini:

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "gemini",
      model: "gemini-embedding-001",
      remote: {
        apiKey: "YOUR_GEMINI_API_KEY"
      }
    }
  }
}
```

Note:

* `remote.baseUrl` è opzionale (per impostazione predefinita usa l’URL di base dell’API Gemini).
* `remote.headers` ti permette di aggiungere intestazioni HTTP aggiuntive se necessario.
* Modello predefinito: `gemini-embedding-001`.

Se desideri usare un **endpoint personalizzato compatibile con OpenAI** (OpenRouter, vLLM o un proxy),
puoi usare la configurazione `remote` con il provider OpenAI:

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      remote: {
        baseUrl: "https://api.example.com/v1/",
        apiKey: "YOUR_OPENAI_COMPAT_API_KEY",
        headers: { "X-Custom-Header": "value" }
      }
    }
  }
}
```

Se non vuoi impostare una chiave API, usa `memorySearch.provider = "local"` oppure imposta
`memorySearch.fallback = "none"`.

Fallback:

* `memorySearch.fallback` può essere `openai`, `gemini`, `local` o `none`.
* Il provider di fallback viene utilizzato solo quando il provider di embedding primario ha esito negativo.

Indicizzazione in batch (OpenAI + Gemini):

* Abilitata per impostazione predefinita per gli embedding OpenAI e Gemini. Imposta `agents.defaults.memorySearch.remote.batch.enabled = false` per disabilitare.
* Il comportamento predefinito attende il completamento del batch; regola `remote.batch.wait`, `remote.batch.pollIntervalMs` e `remote.batch.timeoutMinutes` se necessario.
* Imposta `remote.batch.concurrency` per controllare quanti job di batch vengono inviati in parallelo (valore predefinito: 2).
* La modalità batch si applica quando `memorySearch.provider = "openai"` o `"gemini"` e utilizza la corrispondente chiave API.
* I job di batch Gemini usano l&#39;endpoint asincrono per gli embedding batch e richiedono la disponibilità della Gemini Batch API.

Perché il batch OpenAI è veloce ed economico:

* Per grandi operazioni di backfill, OpenAI è in genere l&#39;opzione più veloce che supportiamo perché possiamo inviare molte richieste di embedding in un singolo job di batch e lasciare che OpenAI le elabori in modo asincrono.
* OpenAI offre prezzi scontati per i carichi di lavoro Batch API, quindi le esecuzioni di indicizzazione di grandi dimensioni sono di solito più economiche rispetto all&#39;invio delle stesse richieste in modo sincrono.
* Consulta la documentazione e il listino prezzi della OpenAI Batch API per i dettagli:
  * https://platform.openai.com/docs/api-reference/batch
  * https://platform.openai.com/pricing

Esempio di configurazione:

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      fallback: "openai",
      remote: {
        batch: { enabled: true, concurrency: 2 }
      },
      sync: { watch: true }
    }
  }
}
```

Strumenti:

* `memory_search` — restituisce frammenti con file + intervalli di righe.
* `memory_get` — legge il contenuto di un file di memoria in base al percorso.

Modalità locale:

* Imposta `agents.defaults.memorySearch.provider = "local"`.
* Fornisci `agents.defaults.memorySearch.local.modelPath` (GGUF o URI `hf:`).
* Opzionale: imposta `agents.defaults.memorySearch.fallback = "none"` per evitare il fallback remoto.

<div id="how-the-memory-tools-work">
  ### Come funzionano gli strumenti di memoria
</div>

* `memory_search` esegue una ricerca semantica su blocchi Markdown (obiettivo di ~400 token, sovrapposizione di 80 token) tratti da `MEMORY.md` + `memory/**/*.md`. Restituisce il testo dello snippet (limitato a ~700 caratteri), il percorso del file, l&#39;intervallo di righe, il punteggio, il provider/modello e se è stato eseguito un fallback da embedding locali → remoti. Non viene restituito il payload completo del file.
* `memory_get` legge uno specifico file Markdown di memoria (relativo allo spazio di lavoro), opzionalmente a partire da una riga iniziale e per N righe. I percorsi al di fuori di `MEMORY.md` / `memory/` sono consentiti solo quando esplicitamente elencati in `memorySearch.extraPaths`.
* Entrambi gli strumenti sono abilitati solo quando `memorySearch.enabled` risulta true per l&#39;agente.

<div id="what-gets-indexed-and-when">
  ### Cosa viene indicizzato (e quando)
</div>

* Tipo di file: solo Markdown (`MEMORY.md`, `memory/**/*.md`, più qualsiasi file `.md` in `memorySearch.extraPaths`).
* Archiviazione dell&#39;indice: SQLite per agente in `~/.openclaw/memory/<agentId>.sqlite` (configurabile tramite `agents.defaults.memorySearch.store.path`, supporta il token `{agentId}`).
* Aggiornamento: un watcher su `MEMORY.md`, `memory/` e `memorySearch.extraPaths` contrassegna l&#39;indice come sporco (con debounce di 1,5 s). La sincronizzazione viene pianificata all&#39;avvio della sessione, durante la ricerca o a intervalli regolari e viene eseguita in modo asincrono. Le trascrizioni di sessione utilizzano soglie di delta per attivare la sincronizzazione in background.
* Trigger di reindicizzazione: l&#39;indice memorizza il **provider/modello di embedding + fingerprint dell&#39;endpoint + parametri di suddivisione in chunk**. Se uno qualsiasi di questi cambia, OpenClaw reimposta e reindicizza automaticamente l&#39;intero archivio.

<div id="hybrid-search-bm25-vector">
  ### Ricerca ibrida (BM25 + vettoriale)
</div>

Quando è attivata, OpenClaw combina:

* **Similarità vettoriale** (corrispondenza semantica, la formulazione può differire)
* **Rilevanza delle parole chiave BM25** (token esatti come ID, variabili d&#39;ambiente, simboli di codice)

Se la ricerca full-text non è disponibile sulla tua piattaforma, OpenClaw passa automaticamente alla sola ricerca vettoriale.

<div id="why-hybrid">
  #### Perché ibrido?
</div>

La ricerca vettoriale è ottima per “questo significa la stessa cosa”:

* “Mac Studio gateway host” vs “the machine running the gateway”
* “debounce file updates” vs “avoid indexing on every write”

Ma può essere debole sui token esatti e ad alto segnale:

* ID (`a828e60`, `b3b9895a…`)
* simboli di codice (`memorySearch.query.hybrid`)
* stringhe di errore (“sqlite-vec unavailable”)

BM25 (full-text) è l’opposto: forte sui token esatti, più debole sulle parafrasi.
La ricerca ibrida è il compromesso pragmatico: **usa entrambi i segnali di retrieval** così da ottenere
buoni risultati sia per query in “linguaggio naturale” sia per query “ago in un pagliaio”.

<div id="how-we-merge-results-the-current-design">
  #### Come uniamo i risultati (design attuale)
</div>

Schema di implementazione:

1. Recupera un pool di candidati da entrambe le parti:

* **Vector**: primi `maxResults * candidateMultiplier` per similarità del coseno.
* **BM25**: primi `maxResults * candidateMultiplier` per rank FTS5 BM25 (più basso è meglio).

2. Converti il rank BM25 in uno score approssimativamente compreso tra 0 e 1:

* `textScore = 1 / (1 + max(0, bm25Rank))`

3. Esegui l’unione dei candidati per chunk id e calcola uno score pesato:

* `finalScore = vectorWeight * vectorScore + textWeight * textScore`

Note:

* `vectorWeight` + `textWeight` è normalizzato a 1.0 nella risoluzione della configurazione, quindi i pesi si comportano come percentuali.
* Se gli embedding non sono disponibili (o il provider restituisce un vettore nullo), eseguiamo comunque BM25 e restituiamo i match per parola chiave.
* Se FTS5 non può essere creato, manteniamo la ricerca solo vettoriale (nessun errore bloccante).

Questo non è “perfetto secondo la teoria dell’IR”, ma è semplice, veloce e tende a migliorare recall/precisione su note reali.
Se in futuro vorremo essere più sofisticati, i passi successivi comuni sono Reciprocal Rank Fusion (RRF) o la normalizzazione degli score
(min/max o z-score) prima di combinarli.

Config:

```json5
agents: {
  defaults: {
    memorySearch: {
      query: {
        hybrid: {
          enabled: true,
          vectorWeight: 0.7,
          textWeight: 0.3,
          candidateMultiplier: 4
        }
      }
    }
  }
}
```

<div id="embedding-cache">
  ### Cache degli embedding
</div>

OpenClaw può memorizzare nella cache gli **embedding dei chunk** in SQLite, così che reindicizzazioni e aggiornamenti frequenti (in particolare le trascrizioni delle sessioni) non richiedano di ricalcolare gli embedding per il testo invariato.

Configurazione:

```json5
agents: {
  defaults: {
    memorySearch: {
      cache: {
        enabled: true,
        maxEntries: 50000
      }
    }
  }
}
```

<div id="session-memory-search-experimental">
  ### Ricerca nella memoria di sessione (sperimentale)
</div>

Puoi opzionalmente indicizzare le **trascrizioni di sessione** e renderle disponibili tramite `memory_search`.
Questa funzionalità è attivabile tramite un flag sperimentale.

```json5
agents: {
  defaults: {
    memorySearch: {
      experimental: { sessionMemory: true },
      sources: ["memory", "sessions"]
    }
  }
}
```

Note:

* L’indicizzazione delle sessioni è **facoltativa** (disattivata per impostazione predefinita).
* Gli aggiornamenti della sessione vengono sottoposti a debounce e **indicizzati in modo asincrono** una volta superate le soglie di delta (best-effort).
* `memory_search` non si blocca mai in attesa dell’indicizzazione; i risultati possono essere leggermente obsoleti finché la sincronizzazione in background non è completata.
* I risultati includono comunque solo frammenti (snippet); `memory_get` resta limitato ai file di memoria.
* L’indicizzazione delle sessioni è isolata per agente (vengono indicizzati solo i log di sessione di quell’agente).
* I log di sessione risiedono su disco (`~/.openclaw/agents/<agentId>/sessions/*.jsonl`). Qualsiasi processo/utente con accesso al filesystem può leggerli, quindi considera l’accesso al disco come il confine di fiducia. Per un isolamento più rigoroso, esegui gli agenti come utenti di sistema o host separati.

Soglie di delta (valori predefiniti indicati):

```json5
agents: {
  defaults: {
    memorySearch: {
      sync: {
        sessions: {
          deltaBytes: 100000,   // ~100 KB
          deltaMessages: 50     // righe JSONL
        }
      }
    }
  }
}
```

<div id="sqlite-vector-acceleration-sqlite-vec">
  ### Accelerazione vettoriale SQLite (sqlite-vec)
</div>

Quando l&#39;estensione sqlite-vec è disponibile, OpenClaw memorizza gli embedding in
una tabella virtuale SQLite (`vec0`) ed esegue query di distanza vettoriale nel
database. In questo modo la ricerca rimane rapida senza caricare ogni embedding in JS.

Configurazione (opzionale):

```json5
agents: {
  defaults: {
    memorySearch: {
      store: {
        vector: {
          enabled: true,
          extensionPath: "/path/to/sqlite-vec"
        }
      }
    }
  }
}
```

Note:

* `enabled` è true per impostazione predefinita; quando è disabilitato, la ricerca passa alla similarità del coseno in-process sulle embedding memorizzate.
* Se l&#39;estensione sqlite-vec manca o non riesce a essere caricata, OpenClaw registra l&#39;errore e continua con il fallback JS (nessuna tabella vettoriale).
* `extensionPath` sovrascrive il percorso sqlite-vec fornito nel pacchetto (utile per build personalizzate o percorsi di installazione non standard).

<div id="local-embedding-auto-download">
  ### Download automatico delle embedding locali
</div>

* Modello di embedding locale predefinito: `hf:ggml-org/embeddinggemma-300M-GGUF/embeddinggemma-300M-Q8_0.gguf` (~0,6 GB).
* Quando `memorySearch.provider = "local"`, `node-llama-cpp` risolve `modelPath`; se il file GGUF manca, viene **scaricato automaticamente** nella cache (o in `local.modelCacheDir` se impostato), quindi caricato. I download riprendono al nuovo tentativo.
* Requisito per la build nativa: esegui `pnpm approve-builds`, seleziona `node-llama-cpp`, quindi `pnpm rebuild node-llama-cpp`.
* Fallback: se la configurazione locale non va a buon fine e `memorySearch.fallback = "openai"`, si passa automaticamente alle embedding remote (`openai/text-embedding-3-small`, a meno che non venga sovrascritto) e viene registrato il motivo.

<div id="custom-openai-compatible-endpoint-example">
  ### Esempio di endpoint personalizzato compatibile con OpenAI
</div>

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      remote: {
        baseUrl: "https://api.example.com/v1/",
        apiKey: "YOUR_REMOTE_API_KEY",
        headers: {
          "X-Organization": "org-id",
          "X-Project": "project-id"
        }
      }
    }
  }
}
```

Note:

* `remote.*` ha la precedenza su `models.providers.openai.*`.
* `remote.headers` si unisce agli header di OpenAI; in caso di conflitto di chiavi prevale `remote`. Ometti `remote.headers` per utilizzare i valori predefiniti di OpenAI.
