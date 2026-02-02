---
title: Stato di salute
summary: "Come l'app macOS segnala gli stati di salute di Gateway/Baileys"
read_when:
  - Debug degli indicatori di stato di salute dell'app macOS
---

<div id="health-checks-on-macos">
  # Verifiche di integrità su macOS
</div>

Come verificare, tramite l&#39;app nella barra dei menu, se il canale collegato è operativo.

<div id="menu-bar">
  ## Barra dei menu
</div>

* L&#39;indicatore di stato ora riflette lo stato di salute di Baileys:
  * Verde: collegato + socket aperto di recente.
  * Arancione: in connessione/nuovo tentativo.
  * Rosso: disconnesso oppure controllo non riuscito.
* La seconda riga mostra &quot;linked · auth 12m&quot; oppure il motivo dell&#39;errore.
* La voce di menu &quot;Run Health Check&quot; avvia un controllo su richiesta.

<div id="settings">
  ## Impostazioni
</div>

* La scheda General include una scheda Health che mostra: anzianità dell’autenticazione collegata, percorso/conteggio del session-store, ora dell’ultimo controllo, ultimo errore/codice di stato e pulsanti per Esegui Health Check / Mostra log.
* Usa uno snapshot in cache così la UI si carica istantaneamente ed esegue un fallback in modo corretto quando è offline.
* La **scheda Channels** mostra lo stato dei canali + i controlli per WhatsApp/Telegram (QR di login, logout, probe/verifica, ultima disconnessione/errore).

<div id="how-the-probe-works">
  ## Come funziona il probe
</div>

* L&#39;app esegue `openclaw health --json` tramite `ShellExecutor` ogni ~60s e su richiesta. Il probe carica le credenziali e riporta lo stato senza inviare messaggi.
* Il probe mette in cache separatamente l&#39;ultima istantanea valida e l&#39;ultimo errore per evitare lampeggiamenti; mostra il timestamp di ciascuno.

<div id="when-in-doubt">
  ## In caso di dubbio
</div>

* Puoi comunque utilizzare la procedura tramite CLI descritta in [Gateway health](/it/gateway/health) (`openclaw status`, `openclaw status --deep`, `openclaw health --json`) ed eseguire un tail su `/tmp/openclaw/openclaw-*.log` per individuare `web-heartbeat` / `web-reconnect`.