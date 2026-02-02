---
title: Ollama
summary: "Esegui OpenClaw con Ollama (runtime locale per LLM)"
read_when:
  - Vuoi eseguire OpenClaw con modelli locali usando Ollama
  - Hai bisogno di assistenza per l'installazione e la configurazione di Ollama
---

<div id="ollama">
  # Ollama
</div>

Ollama è un runtime LLM locale che semplifica l&#39;esecuzione di modelli open source sulla tua macchina. OpenClaw si integra con l&#39;API compatibile con OpenAI di Ollama e può **rilevare automaticamente i modelli con supporto agli strumenti** quando configuri `OLLAMA_API_KEY` (o un profilo di autenticazione) e non definisci una voce esplicita in `models.providers.ollama`.

<div id="quick-start">
  ## Guida rapida
</div>

1. Installa Ollama: https://ollama.ai

2. Scarica un modello:

```bash
ollama pull llama3.3
# oppure
ollama pull qwen2.5-coder:32b
# oppure
ollama pull deepseek-r1:32b
```

3. Abilita Ollama per OpenClaw (qualunque valore va bene; Ollama non richiede una chiave effettiva):

```bash
# Imposta variabile d'ambiente
export OLLAMA_API_KEY="ollama-local"

# Oppure configura nel file di configurazione
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4. Utilizza i modelli di Ollama:

```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/llama3.3" }
    }
  }
}
```

<div id="model-discovery-implicit-provider">
  ## Individuazione dei modelli (provider implicito)
</div>

Quando imposti `OLLAMA_API_KEY` (o un profilo di autenticazione) e **non** definisci `models.providers.ollama`, OpenClaw rileva automaticamente i modelli dall&#39;istanza locale di Ollama all&#39;indirizzo `http://127.0.0.1:11434`:

* Interroga `/api/tags` e `/api/show`
* Mantiene solo i modelli che riportano la funzionalità `tools`
* Imposta `reasoning` quando il modello riporta `thinking`
* Legge `contextWindow` da `model_info["<arch>.context_length"]` quando disponibile
* Imposta `maxTokens` a 10× la finestra di contesto
* Imposta tutti i costi a `0`

Questo evita la definizione manuale dei modelli, mantenendo il catalogo allineato alle capacità di Ollama.

Per vedere quali modelli sono disponibili:

```bash
ollama list
openclaw models list
```

Per aggiungere un nuovo modello, è sufficiente eseguirne il pull con Ollama:

```bash
ollama pull mistral
```

Il nuovo modello verrà rilevato automaticamente e sarà disponibile per l&#39;uso.

Se imposti esplicitamente `models.providers.ollama`, l&#39;individuazione automatica viene disattivata e dovrai definire i modelli manualmente (vedi sotto).

<div id="configuration">
  ## Configurazione
</div>

<div id="basic-setup-implicit-discovery">
  ### Configurazione di base (rilevamento implicito)
</div>

Il modo più semplice per abilitare Ollama è tramite una variabile d&#39;ambiente:

```bash
export OLLAMA_API_KEY="ollama-local"
```

<div id="explicit-setup-manual-models">
  ### Configurazione esplicita (modelli configurati manualmente)
</div>

Usa una configurazione esplicita quando:

* Ollama è in esecuzione su un altro host o su un’altra porta.
* Vuoi forzare finestre di contesto specifiche o elenchi di modelli.
* Vuoi includere modelli che non dichiarano il supporto per gli strumenti.

```json5
{
  models: {
    providers: {
      ollama: {
        // Usa un host che include /v1 per le API compatibili con OpenAI
        baseUrl: "http://ollama-host:11434/v1",
        apiKey: "ollama-local",
        api: "openai-completions",
        models: [
          {
            id: "llama3.3",
            name: "Llama 3.3",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 8192 * 10
          }
        ]
      }
    }
  }
}
```

Se `OLLAMA_API_KEY` è impostata, puoi omettere `apiKey` nella voce del provider e OpenClaw la imposterà automaticamente per i controlli di disponibilità.

<div id="custom-base-url-explicit-config">
  ### URL di base personalizzato (configurazione esplicita)
</div>

Se Ollama è in esecuzione su un host o una porta diversi (la configurazione esplicita disabilita il rilevamento automatico, quindi devi definire i modelli manualmente):

```json5
{
  models: {
    providers: {
      ollama: {
        apiKey: "ollama-local",
        baseUrl: "http://ollama-host:11434/v1"
      }
    }
  }
}
```

<div id="model-selection">
  ### Selezione del modello
</div>

Una volta completata la configurazione, tutti i tuoi modelli Ollama saranno disponibili:

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/llama3.3",
        fallback: ["ollama/qwen2.5-coder:32b"]
      }
    }
  }
}
```

<div id="advanced">
  ## Avanzate
</div>

<div id="reasoning-models">
  ### Modelli di ragionamento
</div>

OpenClaw contrassegna i modelli come dotati di capacità di ragionamento quando Ollama riporta `thinking` in `/api/show`:

```bash
ollama pull deepseek-r1:32b
```

<div id="model-costs">
  ### Costi dei modelli
</div>

Ollama è gratuito e gira in locale, quindi tutti i costi dei modelli sono fissati a 0 $.

<div id="context-windows">
  ### Finestre di contesto
</div>

Per i modelli rilevati automaticamente, OpenClaw usa la finestra di contesto riportata da Ollama quando è disponibile; in caso contrario, usa il valore predefinito `8192`. Puoi sovrascrivere `contextWindow` e `maxTokens` nella configurazione esplicita del provider.

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

<div id="ollama-not-detected">
  ### Ollama non rilevato
</div>

Assicurati che Ollama sia in esecuzione, di aver impostato `OLLAMA_API_KEY` (o un profilo di autenticazione) e di **non** aver definito una voce `models.providers.ollama` esplicita:

```bash
ollama serve
```

E che l&#39;api sia raggiungibile:

```bash
curl http://localhost:11434/api/tags
```

<div id="no-models-available">
  ### Nessun modello disponibile
</div>

OpenClaw rileva automaticamente solo i modelli che dichiarano il supporto agli strumenti (tools). Se il tuo modello non è elencato, puoi:

* Recuperare (`pull`) un modello con supporto agli strumenti, oppure
* Definire esplicitamente il modello in `models.providers.ollama`.

Per aggiungere modelli:

```bash
ollama list  # See what's installed
ollama pull llama3.3  # Scarica un modello
```

<div id="connection-refused">
  ### Connessione rifiutata
</div>

Verifica che Ollama sia in esecuzione sulla porta corretta:

```bash
# Verifica se Ollama è in esecuzione
ps aux | grep ollama

# Oppure riavvia Ollama
ollama serve
```

<div id="see-also">
  ## Vedi anche
</div>

* [Model Providers](/it/concepts/model-providers) - Panoramica di tutti i provider disponibili
* [Model Selection](/it/concepts/models) - Come scegliere i modelli
* [Configuration](/it/gateway/configuration) - Riferimento completo alla configurazione