---
title: Memoria
summary: "Riferimento alla CLI per `openclaw memory` (status/index/search)"
read_when:
  - Vuoi indicizzare o cercare nella memoria semantica
  - Stai effettuando il debug della disponibilità o dell'indicizzazione della memoria
---

<div id="openclaw-memory">
  # `openclaw memory`
</div>

Gestisci l&#39;indicizzazione e la ricerca della memoria semantica.
Fornito dal plugin di memoria attivo (predefinito: `memory-core`; imposta `plugins.slots.memory = "none"` per disabilitarlo).

Voci correlate:

* Concetto di memoria: [Memory](/it/concepts/memory)
* Plugin: [Plugins](/it/plugins)

<div id="examples">
  ## Esempi
</div>

```bash
openclaw memory status
openclaw memory status --deep
openclaw memory status --deep --index
openclaw memory status --deep --index --verbose
openclaw memory index
openclaw memory index --verbose
openclaw memory search "release checklist"
openclaw memory status --agent main
openclaw memory index --agent main --verbose
```

<div id="options">
  ## Opzioni
</div>

Comuni:

* `--agent <id>`: limita a un singolo agente (predefinito: tutti gli agenti configurati).
* `--verbose`: genera log dettagliati durante i controlli e l&#39;indicizzazione.

Note:

* `memory status --deep` verifica la disponibilità di vettori ed embedding.
* `memory status --deep --index` esegue una reindicizzazione se lo store è marcato come dirty.
* `memory index --verbose` stampa i dettagli per fase (provider, modello, sorgenti, attività dei batch).
* `memory status` include eventuali percorsi aggiuntivi configurati tramite `memorySearch.extraPaths`.