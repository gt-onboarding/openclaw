---
title: Log
summary: "Riferimento CLI per `openclaw logs` (visualizzazione in tempo reale dei log del Gateway tramite RPC)"
read_when:
  - Hai bisogno di seguire in tempo reale i log del Gateway da remoto (senza SSH)
  - Vuoi righe di log in formato JSON per strumenti e automazione
---

<div id="openclaw-logs">
  # `openclaw logs`
</div>

Mostra in streaming (tail) i file di log del Gateway tramite RPC (funziona in modalità remota).

Correlati:

* Panoramica del logging: [Logging](/it/logging)

<div id="examples">
  ## Esempi
</div>

```bash
openclaw logs
openclaw logs --follow
openclaw logs --json
openclaw logs --limit 500
```
