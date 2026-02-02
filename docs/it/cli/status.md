---
title: Stato
summary: "Riferimento CLI per `openclaw status` (diagnostica, controlli, snapshot di utilizzo)"
read_when:
  - Vuoi una diagnosi rapida dello stato dei canali e dei destinatari delle sessioni più recenti
  - Vuoi uno stato “all” da copiare e incollare per il debugging
---

<div id="openclaw-status">
  # `openclaw status`
</div>

Diagnostica di canali e sessioni.

```bash
openclaw status
openclaw status --all
openclaw status --deep
openclaw status --usage
```

Note:

* `--deep` esegue controlli live (WhatsApp Web + Telegram + Discord + Google Chat + Slack + Signal).
* L&#39;output include gli archivi di sessione per agente quando sono configurati più agenti.
* La panoramica include lo stato di installazione/runtime del servizio host del Gateway + nodo quando disponibile.
* La panoramica include il canale di aggiornamento + git SHA (per i checkout dal codice sorgente).
* Le informazioni sugli aggiornamenti compaiono nella panoramica; se è disponibile un aggiornamento, `status` visualizza un suggerimento per eseguire `openclaw update` (vedi [Aggiornamento](/it/install/updating)).
