---
title: Potatura delle sessioni
summary: "Potatura delle sessioni: riduzione dei risultati degli strumenti per contenere l’esplosione del contesto"
read_when:
  - Vuoi ridurre la crescita del contesto LLM dovuta agli output degli strumenti
  - Stai configurando agents.defaults.contextPruning
---

<div id="session-pruning">
  # Pulizia delle sessioni
</div>

La pulizia delle sessioni rimuove i **risultati degli strumenti più vecchi** dal contesto in memoria immediatamente prima di ogni chiamata all&#39;LLM. **Non** riscrive la cronologia della sessione memorizzata su disco (`*.jsonl`).

<div id="when-it-runs">
  ## Quando viene eseguita
</div>

* Quando `mode: "cache-ttl"` è abilitata e l&#39;ultima chiamata Anthropic per la sessione risale a più di `ttl` fa.
* Influisce solo sui messaggi inviati al modello per quella richiesta.
* È attiva solo per chiamate API Anthropic (e modelli Anthropic di OpenRouter).
* Per risultati ottimali, allinea `ttl` al `cacheControlTtl` del tuo modello.
* Dopo un&#39;operazione di pruning, la finestra di TTL viene reimpostata, quindi le richieste successive mantengono la cache finché `ttl` non scade di nuovo.

<div id="smart-defaults-anthropic">
  ## Impostazioni predefinite intelligenti (Anthropic)
</div>

* Profili **OAuth o setup-token**: abilita il pruning di `cache-ttl` e imposta l&#39;heartbeat a `1h`.
* Profili **API key**: abilita il pruning di `cache-ttl`, imposta l&#39;heartbeat a `30m` e imposta `cacheControlTtl` predefinito a `1h` sui modelli Anthropic.
* Se imposti esplicitamente uno qualsiasi di questi valori, OpenClaw **non** li sovrascrive.

<div id="what-this-improves-cost-cache-behavior">
  ## Cosa migliora (costo + comportamento della cache)
</div>

* **Perché effettuare il pruning:** la cache del prompt di Anthropic si applica solo all&#39;interno del TTL. Se una sessione resta inattiva oltre il TTL, la richiesta successiva rimette in cache l&#39;intero prompt, a meno che tu non lo riduca prima.
* **Cosa diventa più economico:** il pruning riduce la dimensione di **cacheWrite** per quella prima richiesta dopo la scadenza del TTL.
* **Perché il reset del TTL è importante:** una volta eseguito il pruning, la finestra della cache viene reimpostata, quindi le richieste successive possono riutilizzare il prompt appena messo in cache invece di mettere nuovamente in cache l&#39;intera cronologia.
* **Cosa non fa:** il pruning non aggiunge token né “raddoppia” i costi; cambia solo ciò che viene messo in cache in quella prima richiesta dopo il TTL.

<div id="what-can-be-pruned">
  ## Cosa può essere sottoposto a pruning
</div>

* Solo i messaggi `toolResult`.
* I messaggi di utente e assistente **non** vengono mai modificati.
* Gli ultimi messaggi di assistente indicati da `keepLastAssistants` sono protetti; i risultati degli strumenti successivi a quella soglia non vengono sottoposti a pruning.
* Se non ci sono abbastanza messaggi dell’assistente per stabilire la soglia, il pruning non viene eseguito.
* I risultati degli strumenti che contengono **blocchi immagine** vengono ignorati (mai troncati/azzerati).

<div id="context-window-estimation">
  ## Stima della finestra di contesto
</div>

Il pruning utilizza una finestra di contesto stimata (caratteri ≈ token × 4). La dimensione della finestra viene determinata in questo ordine:

1. `contextWindow` definita nel modello (dal registro dei modelli).
2. sovrascrittura `models.providers.*.models[].contextWindow`.
3. `agents.defaults.contextTokens`.
4. Valore predefinito di `200000` token.

<div id="mode">
  ## Modalità
</div>

<div id="cache-ttl">
  ### cache-ttl
</div>

* Il pruning viene eseguito solo se l&#39;ultima chiamata ad Anthropic risale a più di `ttl` fa (predefinito `5m`).
* Quando viene eseguito: stesso comportamento di soft-trim + hard-clear di prima.

<div id="soft-vs-hard-pruning">
  ## Pruning soft vs hard
</div>

* **Soft-trim**: solo per risultati degli strumenti sovradimensionati.
  * Mantiene inizio e fine, inserisce `...` e aggiunge una nota con la dimensione originale.
  * Ignora i risultati con blocchi immagine.
* **Hard-clear**: sostituisce l&#39;intero risultato dello strumento con `hardClear.placeholder`.

<div id="tool-selection">
  ## Selezione degli strumenti
</div>

* `tools.allow` / `tools.deny` supportano il carattere jolly `*`.
* Le regole di `deny` hanno la precedenza.
* Il confronto non fa distinzione tra maiuscole e minuscole.
* Elenco `allow` vuoto =&gt; tutti gli strumenti sono consentiti.

<div id="interaction-with-other-limits">
  ## Interazione con altri limiti
</div>

* Gli strumenti integrati troncano già il proprio output; il pruning delle sessioni è un ulteriore livello che impedisce alle chat di lunga durata di accumulare troppo output degli strumenti nel contesto del modello.
* La compattazione è distinta: la compattazione riassume e rende persistenti i dati, il pruning è transitorio e avviene per singola richiesta. Vedi [/concepts/compaction](/it/concepts/compaction).

<div id="defaults-when-enabled">
  ## Valori predefiniti (quando abilitati)
</div>

* `ttl`: `"5m"`
* `keepLastAssistants`: `3`
* `softTrimRatio`: `0.3`
* `hardClearRatio`: `0.5`
* `minPrunableToolChars`: `50000`
* `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }`
* `hardClear`: `{ enabled: true, placeholder: "[Contenuto del risultato precedente dello strumento rimosso]" }`

<div id="examples">
  ## Esempi
</div>

Impostazione predefinita (disattivata):

```json5
{
  agent: {
    contextPruning: { mode: "off" }
  }
}
```

Abilita il pruning sensibile al TTL:

```json5
{
  agent: {
    contextPruning: { mode: "cache-ttl", ttl: "5m" }
  }
}
```

Limitare il pruning a strumenti specifici:

```json5
{
  agent: {
    contextPruning: {
      mode: "cache-ttl",
      tools: { allow: ["exec", "read"], deny: ["*image*"] }
    }
  }
}
```

Consulta il riferimento alla configurazione: [Configurazione del Gateway](/it/gateway/configuration)
