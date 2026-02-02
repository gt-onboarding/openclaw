---
title: Modelli locali
summary: "Esegui OpenClaw su LLM locali (LM Studio, vLLM, LiteLLM, endpoint OpenAI personalizzati)"
read_when:
  - Vuoi servire modelli dalla tua macchina con GPU
  - Stai integrando LM Studio o un proxy compatibile con OpenAI
  - Hai bisogno delle linee guida più sicure sui modelli locali
---

<div id="local-models">
  # Modelli locali
</div>

L&#39;esecuzione locale è fattibile, ma OpenClaw richiede contesti molto ampi e difese robuste contro la prompt injection. Schede/GPU con poca memoria troncano il contesto e compromettono la sicurezza. Punta in alto: **≥2 Mac Studio configurati al massimo o un sistema GPU equivalente (~30.000 $+)**. Una singola GPU da **24 GB** è utilizzabile solo per prompt più leggeri, con latenza più elevata. Usa **la variante di modello più grande / full-size che riesci a eseguire**; checkpoint fortemente quantizzati o “small” aumentano il rischio di prompt injection (vedi [Security](/it/gateway/security)).

<div id="recommended-lm-studio-minimax-m21-responses-api-full-size">
  ## Consigliato: LM Studio + MiniMax M2.1 (Responses API, full-size)
</div>

Il miglior stack locale al momento. Carica MiniMax M2.1 in LM Studio, abilita il server locale (predefinito `http://127.0.0.1:1234`) e utilizza la Responses API per tenere separata la fase di ragionamento dal testo finale.

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" }
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

**Checklist di configurazione**

* Installa LM Studio: https://lmstudio.ai
* In LM Studio, scarica la **build MiniMax M2.1 più grande disponibile** (evita le varianti “small”/fortemente quantizzate), avvia il server e conferma che `http://127.0.0.1:1234/v1/models` la elenchi.
* Mantieni il modello caricato; un cold-load aggiunge latenza di avvio.
* Regola `contextWindow`/`maxTokens` se la tua build di LM Studio è diversa.
* Per WhatsApp, usa la Responses API in modo che venga inviato solo il testo finale.

Mantieni i modelli hosted configurati anche quando esegui quelli locali; usa `models.mode: "merge"` in modo che i fallback restino disponibili.

<div id="hybrid-config-hosted-primary-local-fallback">
  ### Configurazione ibrida: primario in hosting, fallback locale
</div>

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["lmstudio/minimax-m2.1-gs32", "anthropic/claude-opus-4-5"]
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "lmstudio/minimax-m2.1-gs32": { alias: "MiniMax Local" },
        "anthropic/claude-opus-4-5": { alias: "Opus" }
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

<div id="local-first-with-hosted-safety-net">
  ### Prima il locale con rete di sicurezza ospitata
</div>

Inverti l&#39;ordine tra primario e fallback; mantieni lo stesso blocco di provider e `models.mode: "merge"` in modo da poter ripiegare su Sonnet o Opus quando la macchina locale non è disponibile.

<div id="regional-hosting-data-routing">
  ### Hosting regionale / instradamento dei dati
</div>

* Su OpenRouter esistono anche varianti MiniMax/Kimi/GLM con endpoint vincolati a una regione (ad es. ospitati negli Stati Uniti). Seleziona lì la variante regionale per mantenere il traffico nella giurisdizione scelta, continuando comunque a usare `models.mode: "merge"` per i fallback Anthropic/OpenAI.
* L’esecuzione solo in locale rimane l’opzione più solida in termini di privacy; l’instradamento regionale ospitato è una soluzione intermedia quando ti servono le funzionalità del provider ma vuoi mantenere il controllo sul flusso dei dati.

<div id="other-openai-compatible-local-proxies">
  ## Altri proxy locali compatibili con OpenAI
</div>

vLLM, LiteLLM, OAI-proxy o gateway personalizzati funzionano se espongono un endpoint `/v1` compatibile con OpenAI. Sostituisci il blocco del provider sopra con il tuo endpoint e l&#39;ID del modello:

```json5
{
  models: {
    mode: "merge",
    providers: {
      local: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "sk-local",
        api: "openai-responses",
        models: [
          {
            id: "my-local-model",
            name: "Local Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 120000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

Mantieni `models.mode: "merge"` in modo che i modelli hosted restino disponibili come fallback.

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

* Il Gateway riesce a raggiungere il proxy? `curl http://127.0.0.1:1234/v1/models`.
* Modello LM Studio non più caricato? Ricaricalo; un cold start è una causa comune di “stalli”.
* Errori di contesto? Riduci `contextWindow` o aumenta il limite del tuo server.
* Sicurezza: i modelli locali bypassano i filtri lato provider; mantieni gli agenti con ambito ristretto e la compattazione attiva per limitare il raggio d’azione degli attacchi di prompt injection.