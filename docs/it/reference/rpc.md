---
title: Rpc
summary: "Adattatori RPC per CLI esterne (signal-cli, imsg) e pattern del Gateway"
read_when:
  - Per aggiungere o modificare integrazioni con CLI esterne
  - Per effettuare il debug degli adattatori RPC (signal-cli, imsg)
---

<div id="rpc-adapters">
  # Adattatori RPC
</div>

OpenClaw integra CLI esterne tramite JSON-RPC. Attualmente sono utilizzati due pattern.

<div id="pattern-a-http-daemon-signal-cli">
  ## Pattern A: demone HTTP (signal-cli)
</div>

* `signal-cli` viene eseguito come demone con JSON-RPC su HTTP.
* Lo stream di eventi è SSE (`/api/v1/events`).
* Probe di health check: `/api/v1/check`.
* OpenClaw gestisce il ciclo di vita quando `channels.signal.autoStart=true`.

Vedi [Signal](/it/channels/signal) per la configurazione e gli endpoint.

<div id="pattern-b-stdio-child-process-imsg">
  ## Pattern B: processo figlio stdio (imsg)
</div>

* OpenClaw avvia `imsg rpc` come processo figlio.
* JSON-RPC è separato per righe su stdin/stdout (un oggetto JSON per riga).
* Nessuna porta TCP, non è richiesto alcun demone.

Metodi principali utilizzati:

* `watch.subscribe` → notifiche (`method: "message"`)
* `watch.unsubscribe`
* `send`
* `chats.list` (verifica/diagnostica)

Vedi [iMessage](/it/channels/imessage) per la configurazione e l&#39;indirizzamento (`chat_id` preferito).

<div id="adapter-guidelines">
  ## Linee guida per gli adapter
</div>

* Il Gateway controlla il processo (avvio/arresto legati al ciclo di vita del provider).
* Mantieni i client RPC resilienti: timeout, riavvio alla terminazione.
* Preferisci ID stabili (ad es. `chat_id`) rispetto alle stringhe mostrate all’utente.