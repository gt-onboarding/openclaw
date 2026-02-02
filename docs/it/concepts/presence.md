---
title: Presenza
summary: "Come vengono create, unite e visualizzate le voci di presenza in OpenClaw"
read_when:
  - Debug della scheda Instances
  - Analisi di righe di istanza duplicate o obsolete
  - Modifica dei beacon di connessione WS del Gateway o dei beacon di eventi di sistema
---

<div id="presence">
  # Presenza
</div>

La “presenza” in OpenClaw è una vista leggera, best‑effort, di:

- del **Gateway** stesso, e
- dei **client connessi al Gateway** (app macOS, WebChat, CLI, ecc.)

La presenza viene utilizzata principalmente per popolare la scheda **Instances** dell'app macOS e per
fornire all'operatore una visibilità immediata.

<div id="presence-fields-what-shows-up">
  ## Campi di presenza (cosa viene visualizzato)
</div>

Le voci di presenza sono oggetti strutturati con campi come:

- `instanceId` (opzionale ma fortemente consigliato): identità stabile del client (di solito `connect.client.instanceId`)
- `host`: nome host leggibile
- `ip`: miglior indirizzo IP disponibile (best‑effort)
- `version`: stringa di versione del client
- `deviceFamily` / `modelIdentifier`: indicazioni sull’hardware
- `mode`: `ui`, `webchat`, `cli`, `backend`, `probe`, `test`, `node`, ...
- `lastInputSeconds`: “secondi dall’ultimo input dell’utente” (se noto)
- `reason`: `self`, `connect`, `node-connected`, `periodic`, ...
- `ts`: timestamp dell’ultimo aggiornamento (ms dall’epoca Unix)

<div id="producers-where-presence-comes-from">
  ## Produttori (da cui proviene la presence)
</div>

Le voci di presence sono prodotte da più sorgenti e **unificate**.

<div id="1-gateway-self-entry">
  ### 1) Gateway self entry
</div>

Il Gateway inserisce sempre una voce "self" all'avvio, in modo che le UI mostrino l'host del Gateway
anche prima che qualsiasi client si connetta.

<div id="2-websocket-connect">
  ### 2) Connessione WebSocket
</div>

Ogni client WS inizia inviando una richiesta `connect`. Al completamento dell’handshake il
Gateway inserisce o aggiorna una voce di presence relativa a quella connessione.

<div id="why-oneoff-cli-commands-dont-show-up">
  #### Perché i comandi CLI una tantum non vengono visualizzati
</div>

La CLI spesso si connette per comandi brevi, eseguiti una sola volta. Per evitare di riempire
la Instances list, `client.mode === "cli"` **non** viene registrato come voce di presenza.

<div id="3-system-event-beacons">
  ### 3) beacon `system-event`
</div>

I client possono inviare beacon periodici più dettagliati tramite il metodo `system-event`. L'app macOS la utilizza per segnalare il nome host, l'indirizzo IP e `lastInputSeconds`.

<div id="4-node-connects-role-node">
  ### 4) Connessione del nodo (ruolo: nodo)
</div>

Quando un nodo si connette al Gateway via WebSocket con `role: node`, il Gateway
inserisce o aggiorna una voce di presenza per quel nodo (stesso flusso degli altri client WS).

<div id="merge-dedupe-rules-why-instanceid-matters">
  ## Regole di merge e deduplicazione (perché `instanceId` è importante)
</div>

Le voci di presenza sono archiviate in un'unica mappa in memoria:

- Le voci sono indicizzate da una **chiave di presenza**.
- La chiave migliore è un `instanceId` stabile (da `connect.client.instanceId`) che sopravvive ai riavvii.
- Le chiavi non sono sensibili alle maiuscole/minuscole.

Se un client si riconnette senza un `instanceId` stabile, può comparire come una riga
**duplicata**.

<div id="ttl-and-bounded-size">
  ## TTL e dimensione limitata
</div>

La presence è volutamente effimera:

- **TTL:** le voci più vecchie di 5 minuti vengono eliminate
- **Numero massimo di voci:** 200 (le più vecchie vengono rimosse per prime)

Questo mantiene l'elenco aggiornato ed evita una crescita incontrollata della memoria.

<div id="remotetunnel-caveat-loopback-ips">
  ## Avvertenza per connessioni remote/tunnel (IP di loopback)
</div>

Quando un client si connette tramite un tunnel SSH / inoltro di porta locale, il Gateway può
vedere l'indirizzo remoto come `127.0.0.1`. Per evitare di sovrascrivere un IP valido fornito dal client,
gli indirizzi remoti di loopback vengono ignorati.

<div id="consumers">
  ## Clienti
</div>

<div id="macos-instances-tab">
  ### Scheda Instances in macOS
</div>

L'app macOS visualizza l'output di `system-presence` e applica un piccolo indicatore di stato (Active/Idle/Stale) in base al tempo trascorso dall'ultimo aggiornamento.

<div id="debugging-tips">
  ## Suggerimenti per il debug
</div>

- Per visualizzare l'elenco non elaborato, esegui `system-presence` sul Gateway.
- Se vedi dei duplicati:
  - verifica che i client inviino un `client.instanceId` stabile nell'handshake
  - verifica che i beacon periodici utilizzino lo stesso `instanceId`
  - controlla se la voce derivata dalla connessione è priva di `instanceId` (in questo caso i duplicati sono attesi)