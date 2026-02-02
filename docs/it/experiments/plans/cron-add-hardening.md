---
title: Hardening di Cron Add
summary: "Rinforzare la gestione degli input di cron.add, allineare gli schemi e migliorare la UI e gli strumenti degli agenti per cron"
owner: "openclaw"
status: "complete"
last_updated: "2026-01-05"
---

<div id="cron-add-hardening-schema-alignment">
  # Hardening di cron add e allineamento dello schema
</div>

<div id="context">
  ## Contesto
</div>

I log più recenti del Gateway mostrano ripetuti errori di `cron.add` con parametri non validi (`sessionTarget`, `wakeMode`, `payload` mancanti e `schedule` malformato). Questo indica che almeno un client (probabilmente il percorso di chiamata dello strumento dell&#39;agente) sta inviando payload di job incapsulati o solo parzialmente specificati. Separatamente, c&#39;è una divergenza tra gli enum del provider cron in TypeScript, lo schema del Gateway, i flag della CLI e i tipi dei moduli della UI, oltre a una discrepanza nella UI per `cron.status` (prevede `jobCount` mentre il Gateway restituisce `jobs`).

<div id="goals">
  ## Obiettivi
</div>

* Eliminare lo spam di `cron.add` INVALID&#95;REQUEST normalizzando i payload wrapper più comuni e deducendo i campi `kind` mancanti.
* Uniformare gli elenchi dei provider cron tra lo schema del Gateway, i tipi cron, la documentazione della CLI e i form della UI.
* Rendere esplicito lo schema dello strumento cron dell&#39;agente, così che l&#39;LLM produca payload di job corretti.
* Correggere la visualizzazione del numero di job nello stato cron della Control UI.
* Aggiungere test per coprire la normalizzazione e il comportamento dello strumento.

<div id="non-goals">
  ## Non-obiettivi
</div>

* Modificare la semantica di pianificazione di cron o il comportamento di esecuzione dei job.
* Aggiungere nuovi tipi di pianificazione o l’analisi delle espressioni cron.
* Riprogettare la UI/UX per cron oltre le correzioni ai campi strettamente necessarie.

<div id="findings-current-gaps">
  ## Osservazioni (lacune attuali)
</div>

* `CronPayloadSchema` nel Gateway esclude `signal` + `imessage`, mentre i tipi TS li includono.
* Control UI CronStatus prevede `jobCount`, ma il Gateway restituisce `jobs`.
* Lo schema dello strumento cron dell&#39;agente consente oggetti `job` arbitrari, permettendo input non validi.
* Il Gateway convalida rigorosamente `cron.add` senza alcuna normalizzazione, quindi i payload incapsulati non vanno a buon fine.

<div id="what-changed">
  ## Cosa è cambiato
</div>

* `cron.add` e `cron.update` ora normalizzano le strutture wrapper comuni e inferiscono i campi `kind` mancanti.
* Lo schema dello strumento cron dell&#39;agente corrisponde allo schema del Gateway, il che riduce i payload non validi.
* Gli enum dei provider sono allineati tra Gateway, CLI, UI e selettore macOS.
* La Control UI utilizza il campo contatore `jobs` del Gateway per lo stato.

<div id="current-behavior">
  ## Comportamento attuale
</div>

* **Normalizzazione:** i payload `data`/`job` incapsulati vengono estratti; `schedule.kind` e `payload.kind` vengono inferiti quando è sicuro farlo.
* **Valori predefiniti:** vengono applicati valori predefiniti sicuri per `wakeMode` e `sessionTarget` quando non specificati.
* **Provider:** Discord/Slack/Signal/iMessage ora sono esposti in modo coerente in CLI/UI.

Consulta [Cron jobs](/it/automation/cron-jobs) per la struttura normalizzata e gli esempi.

<div id="verification">
  ## Verifica
</div>

* Monitora i log del Gateway per verificare una riduzione degli errori `cron.add` INVALID&#95;REQUEST.
* Verifica che lo stato del cron nella Control UI mostri il numero di job dopo l&#39;aggiornamento.

<div id="optional-follow-ups">
  ## Follow-up opzionali
</div>

* Smoke test manuale nel Control UI: aggiungi un job cron per ogni provider e verifica il numero di job di stato.

<div id="open-questions">
  ## Questioni aperte
</div>

* `cron.add` dovrebbe accettare uno `state` esplicito dai client (attualmente non consentito dallo schema)?
* Dovremmo consentire `webchat` come provider di recapito esplicito (attualmente filtrato nella risoluzione del recapito)?