---
title: MiniMax
summary: "Usa MiniMax M2.1 in OpenClaw"
read_when:
  - Vuoi usare i modelli MiniMax in OpenClaw
  - Ti serve una guida per configurare MiniMax
---

<div id="minimax">
  # MiniMax
</div>

MiniMax è un&#39;azienda di intelligenza artificiale che sviluppa la famiglia di modelli **M2/M2.1**. L&#39;attuale
versione orientata al coding è **MiniMax M2.1** (23 dicembre 2025), progettata per
attività complesse nel mondo reale.

Fonte: [Nota di rilascio di MiniMax M2.1](https://www.minimax.io/news/minimax-m21)

<div id="model-overview-m21">
  ## Panoramica del modello (M2.1)
</div>

MiniMax evidenzia i seguenti miglioramenti in M2.1:

* **Programmazione multilingua** più potente (Rust, Java, Go, C++, Kotlin, Objective-C, TS/JS).
* Migliore **sviluppo web/app** e qualità estetica dell&#39;output (incluso mobile nativo).
* Gestione migliorata delle **istruzioni composite** per flussi di lavoro in stile office, basata su
  ragionamento interlacciato ed esecuzione integrata dei vincoli.
* **Risposte più concise** con minore utilizzo di token e cicli di iterazione più rapidi.
* Maggiore compatibilità con il **framework per strumenti/agenti** e migliore gestione del contesto (Claude Code,
  Droid/Factory AI, Cline, Kilo Code, Roo Code, BlackBox).
* Output di **dialogo e scrittura tecnica** di qualità superiore.

<div id="minimax-m21-vs-minimax-m21-lightning">
  ## MiniMax M2.1 vs MiniMax M2.1 Lightning
</div>

* **Velocità:** Lightning è la variante “veloce” nella documentazione dei prezzi di MiniMax.
* **Costo:** I prezzi indicano lo stesso costo di input, ma Lightning ha un costo di output più alto.
* **Instradamento nel piano coding:** Il backend Lightning non è direttamente disponibile
  nel piano coding di MiniMax. MiniMax instrada automaticamente la maggior parte delle
  richieste verso Lightning, ma effettua il fallback al backend M2.1 standard durante i picchi di traffico.

<div id="choose-a-setup">
  ## Scegli una configurazione
</div>

<div id="minimax-m21-recommended">
  ### MiniMax M2.1 — consigliato
</div>

**Ideale per:** MiniMax gestito con API compatibile Anthropic.

Configura tramite CLI:

* Esegui `openclaw configure`
* Seleziona **Model/auth**
* Scegli **MiniMax M2.1**

```json5
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "minimax/MiniMax-M2.1" } } },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

<div id="minimax-m21-as-fallback-opus-primary">
  ### MiniMax M2.1 come fallback (Opus principale)
</div>

**Ideale per:** mantenere Opus 4.5 come modello principale ed eseguire il failover su MiniMax M2.1.

```json5
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" }
      },
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: ["minimax/MiniMax-M2.1"]
      }
    }
  }
}
```

<div id="optional-local-via-lm-studio-manual">
  ### Opzionale: locale tramite LM Studio (manuale)
</div>

**Ideale per:** inferenza locale con LM Studio.
Abbiamo riscontrato ottimi risultati con MiniMax M2.1 su hardware potente (ad es. un
desktop/server) utilizzando il server locale di LM Studio.

Configura manualmente tramite il file `openclaw.json`:

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: { "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" } }
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

<div id="configure-via-openclaw-configure">
  ## Configura tramite `openclaw configure`
</div>

Usa la procedura guidata di configurazione interattiva per configurare MiniMax senza modificare il JSON:

1. Esegui `openclaw configure`.
2. Seleziona **Model/auth**.
3. Scegli **MiniMax M2.1**.
4. Seleziona il modello predefinito quando richiesto.

<div id="configuration-options">
  ## Opzioni di configurazione
</div>

* `models.providers.minimax.baseUrl`: usa preferibilmente `https://api.minimax.io/anthropic` (compatibile con Anthropic); `https://api.minimax.io/v1` è opzionale per payload compatibili con OpenAI.
* `models.providers.minimax.api`: usa preferibilmente `anthropic-messages`; `openai-completions` è opzionale per payload compatibili con OpenAI.
* `models.providers.minimax.apiKey`: chiave API MiniMax (`MINIMAX_API_KEY`).
* `models.providers.minimax.models`: definisci `id`, `name`, `reasoning`, `contextWindow`, `maxTokens`, `cost`.
* `agents.defaults.models`: definisci alias per i modelli che vuoi includere nella lista di autorizzati.
* `models.mode`: lascia `merge` se vuoi aggiungere MiniMax in aggiunta ai modelli integrati.

<div id="notes">
  ## Note
</div>

* I riferimenti ai modelli sono `minimax/<model>`.
* API per l&#39;utilizzo del Coding Plan: `https://api.minimaxi.com/v1/api/openplatform/coding_plan/remains` (richiede una chiave del Coding Plan).
* Aggiorna i valori dei prezzi in `models.json` se hai bisogno di un tracciamento preciso dei costi.
* Link di referral per MiniMax Coding Plan (10% di sconto): https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&amp;source=link
* Consulta [/concepts/model-providers](/it/concepts/model-providers) per le regole sui provider.
* Usa `openclaw models list` e `openclaw models set minimax/MiniMax-M2.1` per cambiare modello.

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

<div id="unknown-model-minimaxminimax-m21">
  ### “Unknown model: minimax/MiniMax-M2.1”
</div>

Di solito significa che il **provider MiniMax non è configurato** (nessuna voce
di provider e nessun profilo di autenticazione/chiave di ambiente MiniMax trovato). Una correzione per questa rilevazione è inclusa in
**2026.1.12** (non ancora rilasciata al momento della stesura). Risolvi così:

* Aggiorna a **2026.1.12** (o esegui da sorgente il branch `main`), quindi riavvia il Gateway.
* Esegui `openclaw configure` e seleziona **MiniMax M2.1**, oppure
* Aggiungi manualmente il blocco `models.providers.minimax`, oppure
* Imposta `MINIMAX_API_KEY` (o un profilo di autenticazione MiniMax) in modo che il provider possa essere iniettato.

Assicurati che l’ID del modello sia **case‑sensitive**:

* `minimax/MiniMax-M2.1`
* `minimax/MiniMax-M2.1-lightning`

Quindi ricontrolla con:

```bash
openclaw models list
```
