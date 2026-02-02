---
title: Venice
summary: "Usa in OpenClaw i modelli Venice AI incentrati sulla privacy"
read_when:
  - Vuoi eseguire inferenze con particolare attenzione alla privacy in OpenClaw
  - Vuoi indicazioni per configurare Venice AI
---

<div id="venice-ai-venice-highlight">
  # Venice AI (Venice highlight)
</div>

**Venice** è la nostra configurazione Venice di punta per l'inferenza orientata alla privacy, con accesso opzionale e anonimizzato a modelli proprietari.

Venice AI offre inferenza IA incentrata sulla privacy con supporto per modelli non censurati e accesso ai principali modelli proprietari tramite un proxy anonimizzato. Tutta l'inferenza è privata per impostazione predefinita: nessun addestramento sui tuoi dati, nessuna registrazione dei log.

<div id="why-venice-in-openclaw">
  ## Perché Venice in OpenClaw
</div>

- **Inferenza privata** per modelli open-source (senza log).
- **Modelli non censurati** quando necessario.
- **Accesso anonimizzato** a modelli proprietari (Opus/GPT/Gemini) quando la qualità conta.
- Endpoint `/v1` compatibili con OpenAI.

<div id="privacy-modes">
  ## Modalità di privacy
</div>

Venice offre due livelli di privacy: comprenderli è fondamentale per scegliere il modello:

| Modalità | Descrizione | Modelli |
|------|-------------|--------|
| **Privata** | Completamente privata. I prompt e le risposte non vengono mai archiviati né registrati. Effimera. | Llama, Qwen, DeepSeek, Venice Uncensored, ecc. |
| **Anonimizzata** | Le richieste vengono proxyate tramite Venice con i metadati rimossi. Il provider sottostante (OpenAI, Anthropic) vede solo richieste anonimizzate. | Claude, GPT, Gemini, Grok, Kimi, MiniMax |

<div id="features">
  ## Funzionalità
</div>

- **Orientato alla privacy**: scegli tra le modalità "private" (completamente privata) e "anonymized" (con proxy)
- **Modelli non censurati**: accesso a modelli senza restrizioni sui contenuti
- **Accesso ai principali modelli**: usa Claude, GPT-5.2, Gemini, Grok tramite il proxy anonimizzato di Venice
- **API compatibile con OpenAI**: endpoint standard `/v1` per un'integrazione semplice
- **Streaming**: ✅ supportato su tutti i modelli
- **Function calling**: ✅ supportato su alcuni modelli (verifica le capacità del modello)
- **Vision**: ✅ supportato sui modelli con capacità di visione
- **Nessun hard rate limit**: può essere applicato un throttling basato sul fair use in caso di utilizzo estremo

<div id="setup">
  ## Configurazione
</div>

<div id="1-get-api-key">
  ### 1. Ottieni la chiave API
</div>

1. Registrati su [venice.ai](https://venice.ai)
2. Vai in **Settings → API Keys → Create new key**
3. Copia la tua chiave API (formato: `vapi_xxxxxxxxxxxx`)

<div id="2-configure-openclaw">
  ### 2. Configura OpenClaw
</div>

**Opzione A: Variabile d&#39;ambiente**

```bash
export VENICE_API_KEY="vapi_xxxxxxxxxxxx"
```

**Opzione B: Configurazione interattiva (consigliata)**

```bash
openclaw onboard --auth-choice venice-api-key
```

Questo:

1. Ti chiederà la tua chiave API (o utilizzerà la `VENICE_API_KEY` esistente)
2. Mostrerà tutti i modelli Venice disponibili
3. Ti permetterà di scegliere il tuo modello predefinito
4. Configurerà automaticamente il provider

**Opzione C: Non interattiva**

```bash
openclaw onboard --non-interactive \
  --auth-choice venice-api-key \
  --venice-api-key "vapi_xxxxxxxxxxxx"
```


<div id="3-verify-setup">
  ### 3. Verifica della configurazione
</div>

```bash
openclaw chat --model venice/llama-3.3-70b "Hello, are you working?"
```


<div id="model-selection">
  ## Selezione del modello
</div>

Dopo la configurazione, OpenClaw mostra tutti i modelli Venice disponibili. Scegline uno in base alle tue esigenze:

* **Predefinito (la nostra scelta)**: `venice/llama-3.3-70b` per prestazioni equilibrate e private.
* **Migliore qualità complessiva**: `venice/claude-opus-45` per i carichi più impegnativi (Opus resta il più potente).
* **Privacy**: scegli i modelli &quot;private&quot; per un&#39;inferenza completamente privata.
* **Capacità**: scegli i modelli &quot;anonymized&quot; per accedere a Claude, GPT e Gemini tramite il proxy Venice.

Puoi cambiare il modello predefinito in qualsiasi momento:

```bash
openclaw models set venice/claude-opus-45
openclaw models set venice/llama-3.3-70b
```

Elenca tutti i modelli disponibili:

```bash
openclaw models list | grep venice
```


<div id="configure-via-openclaw-configure">
  ## Configura tramite `openclaw configure`
</div>

1. Esegui il comando `openclaw configure`
2. Seleziona **Model/auth**
3. Scegli **Venice AI**

<div id="which-model-should-i-use">
  ## Quale modello dovrei usare?
</div>

| Caso d'uso | Modello consigliato | Motivo |
|----------|-------------------|-----|
| **Chat generale** | `llama-3.3-70b` | Buon modello generico, completamente privato |
| **Migliore qualità complessiva** | `claude-opus-45` | Opus resta il più forte per i compiti difficili |
| **Privacy + qualità Claude** | `claude-opus-45` | Miglior capacità di ragionamento tramite proxy anonimizzato |
| **Sviluppo (coding)** | `qwen3-coder-480b-a35b-instruct` | Ottimizzato per il codice, contesto da 262k token |
| **Attività di visione** | `qwen3-vl-235b-a22b` | Miglior modello di visione privato |
| **Senza filtri** | `venice-uncensored` | Nessuna restrizione sui contenuti |
| **Veloce + economico** | `qwen3-4b` | Leggero ma comunque valido |
| **Ragionamento complesso** | `deepseek-v3.2` | Ottime capacità di ragionamento, privato |

<div id="available-models-25-total">
  ## Modelli disponibili (25 totali)
</div>

<div id="private-models-15-fully-private-no-logging">
  ### Modelli privati (15) — Completamente privati, senza log
</div>

| Model ID | Nome | Contesto (token) | Funzionalità |
|----------|------|------------------|--------------|
| `llama-3.3-70b` | Llama 3.3 70B | 131k | Generale |
| `llama-3.2-3b` | Llama 3.2 3B | 131k | Veloce e leggero |
| `hermes-3-llama-3.1-405b` | Hermes 3 Llama 3.1 405B | 131k | Compiti complessi |
| `qwen3-235b-a22b-thinking-2507` | Qwen3 235B Thinking | 131k | Ragionamento |
| `qwen3-235b-a22b-instruct-2507` | Qwen3 235B Instruct | 131k | Generale |
| `qwen3-coder-480b-a35b-instruct` | Qwen3 Coder 480B | 262k | Codice |
| `qwen3-next-80b` | Qwen3 Next 80B | 262k | Generale |
| `qwen3-vl-235b-a22b` | Qwen3 VL 235B | 262k | Visione |
| `qwen3-4b` | Venice Small (Qwen3 4B) | 32k | Veloce, per il ragionamento |
| `deepseek-v3.2` | DeepSeek V3.2 | 163k | Ragionamento |
| `venice-uncensored` | Venice Uncensored | 32k | Non censurato |
| `mistral-31-24b` | Venice Medium (Mistral) | 131k | Visione |
| `google-gemma-3-27b-it` | Gemma 3 27B Instruct | 202k | Visione |
| `openai-gpt-oss-120b` | OpenAI GPT OSS 120B | 131k | Generale |
| `zai-org-glm-4.7` | GLM 4.7 | 202k | Ragionamento, multilingue |

<div id="anonymized-models-10-via-venice-proxy">
  ### Modelli anonimizzati (10) — tramite Venice Proxy
</div>

| ID modello | Originale | Contesto (token) | Funzionalità |
|-----------|-----------|------------------|--------------|
| `claude-opus-45` | Claude Opus 4.5 | 202k | Ragionamento, visione |
| `claude-sonnet-45` | Claude Sonnet 4.5 | 202k | Ragionamento, visione |
| `openai-gpt-52` | GPT-5.2 | 262k | Ragionamento |
| `openai-gpt-52-codex` | GPT-5.2 Codex | 262k | Ragionamento, visione |
| `gemini-3-pro-preview` | Gemini 3 Pro | 202k | Ragionamento, visione |
| `gemini-3-flash-preview` | Gemini 3 Flash | 262k | Ragionamento, visione |
| `grok-41-fast` | Grok 4.1 Fast | 262k | Ragionamento, visione |
| `grok-code-fast-1` | Grok Code Fast 1 | 262k | Ragionamento, codice |
| `kimi-k2-thinking` | Kimi K2 Thinking | 262k | Ragionamento |
| `minimax-m21` | MiniMax M2.1 | 202k | Ragionamento |

<div id="model-discovery">
  ## Scoperta dei modelli
</div>

OpenClaw rileva automaticamente i modelli tramite la Venice API quando `VENICE_API_KEY` è impostata. Se l'API non è raggiungibile, viene utilizzato un catalogo statico come fallback.

L'endpoint `/models` è pubblico (non è necessaria autenticazione per l'elenco), ma le richieste di inferenza richiedono una chiave API valida.

<div id="streaming-tool-support">
  ## Supporto per streaming e strumenti
</div>

| Funzionalità | Supporto |
|---------|---------|
| **Streaming** | ✅ Tutti i modelli |
| **Function calling** | ✅ La maggior parte dei modelli (verifica `supportsFunctionCalling` nell'API) |
| **Vision/Images** | ✅ Modelli contrassegnati con la funzionalità "Vision" |
| **Modalità JSON** | ✅ Supportata tramite `response_format` |

<div id="pricing">
  ## Prezzi
</div>

Venice utilizza un sistema basato su crediti. Consulta le tariffe attuali su [venice.ai/pricing](https://venice.ai/pricing):

- **Modelli privati**: Generalmente meno costosi
- **Modelli anonimizzati**: Prezzi simili all'API diretta + una piccola commissione di Venice

<div id="comparison-venice-vs-direct-api">
  ## Confronto: Venice vs API diretta
</div>

| Aspetto | Venice (anonimizzato) | API diretta |
|--------|------------------------|------------|
| **Privacy** | Metadati rimossi e anonimizzati | Associata al tuo account |
| **Latenza** | +10-50ms (proxy) | Diretta |
| **Funzionalità** | Supporta la maggior parte delle funzionalità | Funzionalità complete |
| **Fatturazione** | Crediti Venice | Fatturazione del provider |

<div id="usage-examples">
  ## Esempi d&#39;uso
</div>

```bash
# Use default private model
openclaw chat --model venice/llama-3.3-70b

# Usa Claude tramite Venice (anonimizzato)
openclaw chat --model venice/claude-opus-45

# Use uncensored model
openclaw chat --model venice/venice-uncensored

# Use vision model with image
openclaw chat --model venice/qwen3-vl-235b-a22b

# Use coding model
openclaw chat --model venice/qwen3-coder-480b-a35b-instruct
```


<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

<div id="api-key-not-recognized">
  ### Chiave API non riconosciuta
</div>

```bash
echo $VENICE_API_KEY
openclaw models list | grep venice
```

Assicurati che la chiave inizi con `vapi_`.


<div id="model-not-available">
  ### Modello non disponibile
</div>

Il catalogo dei modelli Venice viene aggiornato dinamicamente. Esegui il comando `openclaw models list` per visualizzare i modelli attualmente disponibili. Alcuni modelli potrebbero essere temporaneamente offline.

<div id="connection-issues">
  ### Problemi di connessione
</div>

L'endpoint dell'API Venice è `https://api.venice.ai/api/v1`. Assicurati che la rete consenta connessioni HTTPS.

<div id="config-file-example">
  ## Esempio di file di configurazione
</div>

```json5
{
  env: { VENICE_API_KEY: "vapi_..." },
  agents: { defaults: { model: { primary: "venice/llama-3.3-70b" } } },
  models: {
    mode: "merge",
    providers: {
      venice: {
        baseUrl: "https://api.venice.ai/api/v1",
        apiKey: "${VENICE_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "llama-3.3-70b",
            name: "Llama 3.3 70B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 131072,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```


<div id="links">
  ## Link
</div>

- [Venice AI](https://venice.ai)
- [Documentazione API](https://docs.venice.ai)
- [Prezzi](https://venice.ai/pricing)
- [Stato del servizio](https://status.venice.ai)