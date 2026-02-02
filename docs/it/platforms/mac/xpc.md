---
title: Xpc
summary: "Architettura IPC di macOS per l'app OpenClaw, il trasporto del nodo Gateway e PeekabooBridge"
read_when:
  - Modifica dei contratti IPC o dell'IPC dell'app nella barra dei menu
---

<div id="openclaw-macos-ipc-architecture">
  # Architettura IPC di OpenClaw per macOS
</div>

**Modello attuale:** un socket Unix locale collega il **servizio host del nodo** all&#39;**app macOS** per le approvazioni di esecuzione e `system.run`. Esiste una CLI di debug `openclaw-mac` per i controlli di discovery/connessione; le azioni degli agenti continuano comunque a passare attraverso il WebSocket del Gateway e `node.invoke`. L&#39;automazione della UI utilizza PeekabooBridge.

<div id="goals">
  ## Obiettivi
</div>

* Un&#39;unica istanza dell&#39;app con interfaccia grafica (GUI) che gestisce tutte le attività che interagiscono con TCC (notifiche, registrazione dello schermo, microfono, riconoscimento vocale, AppleScript).
* Una superficie ridotta per l&#39;automazione: comandi del Gateway + nodo, e PeekabooBridge per l&#39;automazione della UI.
* Permessi prevedibili: sempre lo stesso bundle ID firmato, avviata da launchd, così le autorizzazioni TCC restano valide.

<div id="how-it-works">
  ## Come funziona
</div>

<div id="gateway-node-transport">
  ### Gateway + trasporto nodo
</div>

* L&#39;app esegue il Gateway (modalità locale) e vi si connette come nodo.
* Le azioni dell&#39;agente vengono eseguite tramite `node.invoke` (ad es. `system.run`, `system.notify`, `canvas.*`).

<div id="node-service-app-ipc">
  ### Servizio del nodo + IPC dell&#39;app
</div>

* Un servizio del nodo headless si connette al WebSocket del Gateway.
* Le richieste `system.run` vengono inoltrate all&#39;app macOS tramite una socket Unix locale.
* L&#39;app esegue il comando nel contesto della UI, chiede conferma se necessario e restituisce l&#39;output.

Diagramma (SCI):

```
Agent -> Gateway -> Node Service (WS)
                      |  IPC (UDS + token + HMAC + TTL)
                      v
                  Mac App (UI + TCC + system.run)
```

<div id="peekaboobridge-ui-automation">
  ### PeekabooBridge (automazione UI)
</div>

* L&#39;automazione UI utilizza un socket UNIX separato chiamato `bridge.sock` e il protocollo JSON di PeekabooBridge.
* Ordine di preferenza dell&#39;host (lato client): Peekaboo.app → Claude.app → OpenClaw.app → esecuzione locale.
* Sicurezza: gli host del bridge richiedono un TeamID autorizzato; il meccanismo di emergenza solo in DEBUG per lo stesso UID è controllato da `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` (convenzione Peekaboo).
* Vedi: [utilizzo di PeekabooBridge](/it/platforms/mac/peekaboo) per maggiori dettagli.

<div id="operational-flows">
  ## Flussi operativi
</div>

* Riavvio/ricostruzione: `SIGN_IDENTITY="Apple Development: <Developer Name> (<TEAMID>)" scripts/restart-mac.sh`
  * Termina le istanze esistenti
  * Compila Swift + crea il pacchetto
  * Scrive/inizializza/avvia il LaunchAgent
* Istanza singola: l&#39;applicazione termina subito se è già in esecuzione un&#39;altra istanza con lo stesso bundle ID.

<div id="hardening-notes">
  ## Note di hardening
</div>

* È preferibile richiedere una corrispondenza del TeamID per tutte le superfici privilegiate.
* PeekabooBridge: `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` (solo DEBUG) può consentire client con lo stesso UID per lo sviluppo locale.
* Tutte le comunicazioni rimangono locali; nessun socket di rete viene esposto.
* I prompt TCC provengono esclusivamente dal bundle dell&#39;app GUI; mantieni stabile l&#39;ID del bundle firmato tra una ricompilazione e l&#39;altra.
* Hardening IPC: modalità socket `0600`, token, verifiche del peer UID, challenge/response HMAC, TTL breve.