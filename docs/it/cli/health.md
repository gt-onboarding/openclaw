---
title: Stato di salute
summary: "Riferimento CLI per `openclaw health` (endpoint di stato del Gateway via RPC)"
read_when:
  - Vuoi verificare rapidamente lo stato del Gateway in esecuzione
---

<div id="openclaw-health">
  # `openclaw health`
</div>

Recupera lo stato di salute del Gateway in esecuzione.

```bash
openclaw health
openclaw health --json
openclaw health --verbose
```

Note:

* `--verbose` esegue controlli live e stampa i tempi per ogni account quando sono configurati più account.
* L&#39;output include gli archivi di sessione per ogni agente quando sono configurati più agenti.
