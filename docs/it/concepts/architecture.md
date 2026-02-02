---
title: Architettura
summary: "Architettura del Gateway WS, componenti e flussi dei client"
read_when:
  - Stai lavorando al protocollo del Gateway, ai client o ai trasporti
---

<div id="gateway-architecture">
  # Architettura del Gateway
</div>

Ultimo aggiornamento: 2026-01-22

<div id="overview">
  ## Panoramica
</div>

* Un singolo **Gateway** di lunga durata gestisce tutti i canali di messaggistica (WhatsApp tramite
  Baileys, Telegram tramite grammY, Slack, Discord, Signal, iMessage, WebChat).
* I client del piano di controllo (app macOS, CLI, UI web, automazioni) si connettono al
  Gateway tramite **WebSocket** sull&#39;host di binding configurato (predefinito
  `127.0.0.1:18789`).
* I **nodi** (macOS/iOS/Android/headless) si connettono anch&#39;essi tramite **WebSocket**, ma
  dichiarano `role: node` con capacità/comandi espliciti.
* Un Gateway per host; è l&#39;unico punto in cui viene aperta una sessione WhatsApp.
* Un **canvas host** (predefinito `18793`) espone HTML modificabile dagli agenti e A2UI.

<div id="components-and-flows">
  ## Componenti e flussi
</div>

<div id="gateway-daemon">
  ### Gateway (demone)
</div>

* Mantiene le connessioni con i provider.
* Espone un&#39;API WS tipizzata (richieste, risposte, eventi server‑push).
* Valida i frame in ingresso rispetto a uno schema JSON.
* Emette eventi come `agent`, `chat`, `presence`, `health`, `heartbeat`, `cron`.

<div id="clients-mac-app-cli-web-admin">
  ### Client (app macOS / CLI / web admin)
</div>

* Una connessione WS per ogni client.
* Invia richieste (`health`, `status`, `send`, `agent`, `system-presence`).
* Sottoscrivi agli eventi (`tick`, `agent`, `presence`, `shutdown`).

<div id="nodes-macos-ios-android-headless">
  ### Nodi (macOS / iOS / Android / headless)
</div>

* Si connettono allo **stesso server WS** con `role: node`.
* Forniscono un&#39;identità del dispositivo in `connect`; l&#39;abbinamento è **per dispositivo** (ruolo `node`) e
  l&#39;approvazione viene memorizzata nello store di abbinamento del dispositivo.
* Espongono comandi come `canvas.*`, `camera.*`, `screen.record`, `location.get`.

Dettagli del protocollo:

* [Gateway protocol](/it/gateway/protocol)

<div id="webchat">
  ### WebChat
</div>

* UI statica che utilizza la API WS del Gateway per la cronologia delle chat e le operazioni di invio.
* Nelle configurazioni remote, si connette tramite lo stesso tunnel SSH/Tailscale degli altri client.

<div id="connection-lifecycle-single-client">
  ## Ciclo di vita della connessione (singolo client)
</div>

```
Client                    Gateway
  |                          |
  |---- req:connect -------->|
  |<------ res (ok) ---------|   (or res error + close)
  |   (payload=hello-ok carries snapshot: presence + health)
  |                          |
  |<------ event:presence ---|
  |<------ event:tick -------|
  |                          |
  |------- req:agent ------->|
  |<------ res:agent --------|   (ack: {runId,status:"accepted"})
  |<------ event:agent ------|   (streaming)
  |<------ res:agent --------|   (final: {runId,status,summary})
  |                          |
```

<div id="wire-protocol-summary">
  ## Protocollo wire (riepilogo)
</div>

* Trasporto: WebSocket, frame di testo con payload JSON.
* Il primo frame **deve** essere `connect`.
* Dopo l’handshake:
  * Richieste: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
  * Eventi: `{type:"event", event, payload, seq?, stateVersion?}`
* Se `OPENCLAW_GATEWAY_TOKEN` (o `--token`) è impostato, `connect.params.auth.token`
  deve corrispondere, in caso contrario il socket viene chiuso.
* Le chiavi di idempotenza sono obbligatorie per i metodi che producono effetti collaterali (`send`, `agent`) per
  poter ritentare in sicurezza; il server mantiene una cache di deduplica di breve durata.
* I nodi devono includere `role: "node"` più capacità/comandi/autorizzazioni in `connect`.

<div id="pairing-local-trust">
  ## Abbinamento e fiducia locale
</div>

* Tutti i client WS (operatori + nodi) includono un’**identità del dispositivo** in fase di `connect`.
* I nuovi ID di dispositivo richiedono l’approvazione di abbinamento; il Gateway emette un **token del dispositivo**
  per le connessioni successive.
* Le connessioni **locali** (loopback o l’indirizzo tailnet dell’host del Gateway stesso) possono essere
  approvate automaticamente per mantenere fluida la UX sullo stesso host.
* Le connessioni **non locali** devono firmare il nonce `connect.challenge` e richiedono
  un’approvazione esplicita.
* L’autenticazione del Gateway (`gateway.auth.*`) si applica comunque a **tutte** le connessioni, locali o
  remote.

Dettagli: [Gateway protocol](/it/gateway/protocol), [Pairing](/it/start/pairing),
[Security](/it/gateway/security).

<div id="protocol-typing-and-codegen">
  ## Tipizzazione del protocollo e generazione del codice
</div>

* Gli schemi TypeBox definiscono il protocollo.
* Lo schema JSON viene generato da tali schemi.
* I modelli Swift vengono generati dallo schema JSON.

<div id="remote-access">
  ## Accesso remoto
</div>

* Consigliato: Tailscale o VPN.
* Alternativa: tunnel SSH
  ```bash
  ssh -N -L 18789:127.0.0.1:18789 user@host
  ```
* Lo stesso handshake e lo stesso token di autenticazione si applicano anche attraverso il tunnel.
* TLS e il pinning opzionale possono essere abilitati per WS nelle configurazioni remote.

<div id="operations-snapshot">
  ## Panoramica operativa
</div>

* Avvio: `openclaw gateway` (in foreground, scrive i log su stdout).
* Stato: `health` via WS (incluso anche in `hello-ok`).
* Supervisione: launchd/systemd per il riavvio automatico.

<div id="invariants">
  ## Invarianti
</div>

* Esattamente un Gateway controlla una singola sessione Baileys per ogni host.
* L&#39;handshake è obbligatorio; qualsiasi primo frame non JSON o che non sia di tipo connect comporta una chiusura immediata.
* Gli eventi non vengono rigiocati; i client devono aggiornare in caso di gap.