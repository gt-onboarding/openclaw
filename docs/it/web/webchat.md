---
title: WebChat
summary: "Host statico WebChat in loopback e utilizzo del WS del Gateway per la UI di chat"
read_when:
  - Durante il debug o la configurazione dell'accesso a WebChat
---

<div id="webchat-gateway-websocket-ui">
  # WebChat (Gateway WebSocket UI)
</div>

Stato: la UI di chat SwiftUI per macOS/iOS comunica direttamente con il WebSocket del Gateway.

<div id="what-it-is">
  ## Di cosa si tratta
</div>

* Una UI di chat nativa per il Gateway (nessun browser integrato né server statico locale).
* Utilizza le stesse sessioni e le stesse regole di instradamento degli altri canali.
* Instradamento deterministico: le risposte vengono sempre recapitate a WebChat.

<div id="quick-start">
  ## Avvio rapido
</div>

1. Avvia il Gateway.
2. Apri la UI WebChat (app macOS/iOS) oppure la scheda chat del Control UI.
3. Assicurati che l&#39;autenticazione del Gateway sia configurata correttamente (attiva per impostazione predefinita, anche sull&#39;interfaccia di loopback).

<div id="how-it-works-behavior">
  ## Come funziona (comportamento)
</div>

* La UI si connette al WebSocket del Gateway e usa `chat.history`, `chat.send` e `chat.inject`.
* `chat.inject` aggiunge una nota dell&#39;assistente direttamente alla trascrizione e la trasmette alla UI (nessuna esecuzione dell&#39;agente).
* La cronologia viene sempre recuperata dal Gateway (nessun monitoraggio di file locali).
* Se il Gateway non è raggiungibile, WebChat è in sola lettura.

<div id="remote-use">
  ## Uso remoto
</div>

* La modalità remota instrada il WebSocket del Gateway tramite SSH/Tailscale.
* Non è necessario eseguire un server WebChat separato.

<div id="configuration-reference-webchat">
  ## Riferimento di configurazione (WebChat)
</div>

Configurazione completa: [Configuration](/it/gateway/configuration)

Opzioni del canale:

* Nessun blocco `webchat.*` dedicato. WebChat utilizza l&#39;endpoint del Gateway + le impostazioni di autenticazione riportate di seguito.

Opzioni globali correlate:

* `gateway.port`, `gateway.bind`: host/porta WebSocket.
* `gateway.auth.mode`, `gateway.auth.token`, `gateway.auth.password`: autenticazione WebSocket.
* `gateway.remote.url`, `gateway.remote.token`, `gateway.remote.password`: destinazione del Gateway remoto.
* `session.*`: archiviazione delle sessioni e valori predefiniti delle chiavi principali di sessione.