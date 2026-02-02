---
title: Blocco del Gateway
summary: "Meccanismo singleton del Gateway tramite il binding del listener WebSocket"
read_when:
  - Durante l'esecuzione o il debug del processo del Gateway
  - Durante l'analisi del meccanismo di enforcement della modalità a istanza singola
---

<div id="gateway-lock">
  # Blocco del Gateway
</div>

Ultimo aggiornamento: 11/12/2025

<div id="why">
  ## Perché
</div>

- Garantire che sia in esecuzione una sola istanza del Gateway per ogni porta base sullo stesso host; eventuali Gateway aggiuntivi devono usare profili isolati e porte univoche.
- Resistere a crash/SIGKILL senza lasciare file di lock residui.
- Interrompersi subito con un errore chiaro quando la porta di controllo è già occupata.

<div id="mechanism">
  ## Meccanismo
</div>

- Il Gateway effettua il binding del listener WebSocket (predefinito `ws://127.0.0.1:18789`) immediatamente all'avvio utilizzando un listener TCP esclusivo.
- Se il binding non riesce con `EADDRINUSE`, in fase di avvio viene generato `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:<port>")`.
- Il sistema operativo rilascia automaticamente il listener all'uscita di qualsiasi processo, inclusi crash e SIGKILL — non è necessario alcun file di lock separato o operazione di pulizia.
- Durante l'arresto il Gateway chiude il server WebSocket e il server HTTP sottostante per liberare prontamente la porta.

<div id="error-surface">
  ## Esposizione degli errori
</div>

- Se un altro processo sta già utilizzando la porta, all'avvio viene generato `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:<port>")`.
- Altri errori di bind vengono esposti come `GatewayLockError("failed to bind gateway socket on ws://127.0.0.1:<port>: …")`.

<div id="operational-notes">
  ## Note operative
</div>

- Se la porta è occupata da *un altro* processo, l'errore è lo stesso; libera la porta oppure scegline un'altra con `openclaw gateway --port <port>`.
- L'app macOS mantiene comunque un proprio leggero controllo PID prima di avviare il Gateway; il blocco del runtime è applicato tramite il binding WebSocket.