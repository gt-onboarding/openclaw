---
title: Cron
summary: "Riferimento CLI per `openclaw cron` (pianificare ed eseguire attività in background)"
read_when:
  - Vuoi job pianificati e riattivazioni
  - Stai eseguendo il debug dell'esecuzione e dei log di cron
---

<div id="openclaw-cron">
  # `openclaw cron`
</div>

Gestisci i job cron per lo scheduler del Gateway.

Correlato:

* Job cron: [Cron jobs](/it/automation/cron-jobs)

Suggerimento: esegui `openclaw cron --help` per l&#39;elenco completo dei comandi disponibili.

<div id="common-edits">
  ## Modifiche comuni
</div>

Aggiorna le impostazioni di recapito senza modificare il messaggio:

```bash
openclaw cron edit <job-id> --deliver --channel telegram --to "123456789"
```

Disabilita l’invio per un job isolato:

```bash
openclaw cron edit <job-id> --no-deliver
```
