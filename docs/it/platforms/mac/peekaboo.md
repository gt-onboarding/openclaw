---
title: Peekaboo
summary: "Integrazione di PeekabooBridge per l'automazione della UI in macOS"
read_when:
  - Eseguire PeekabooBridge all'interno di OpenClaw.app
  - Integrare Peekaboo tramite Swift Package Manager
  - Modificare il protocollo o i percorsi di PeekabooBridge
---

<div id="peekaboo-bridge-macos-ui-automation">
  # Peekaboo Bridge (automazione UI su macOS)
</div>

OpenClaw può ospitare **PeekabooBridge** come broker locale per l’automazione dell’UI,
che tiene conto dei permessi. Questo consente alla CLI `peekaboo` di automatizzare
l’UI riutilizzando le autorizzazioni TCC dell’app macOS.

<div id="what-this-is-and-isnt">
  ## Di cosa si tratta (e cosa non è)
</div>

- **Host**: OpenClaw.app può fungere da host per PeekabooBridge.
- **Client**: usa la CLI `peekaboo` (nessuna UI separata `openclaw ui ...`).
- **UI**: gli overlay visivi rimangono in Peekaboo.app; OpenClaw è un host/broker leggero.

<div id="enable-the-bridge">
  ## Abilita il bridge
</div>

Nell'app per macOS:

- Impostazioni → **Abilita Peekaboo Bridge**

Quando è abilitato, OpenClaw avvia un server socket UNIX locale. Se è disabilitato, l'host
viene arrestato e `peekaboo` passerà ad altri host disponibili.

<div id="client-discovery-order">
  ## Ordine di ricerca del client
</div>

I client Peekaboo in genere provano gli host in questo ordine:

1. Peekaboo.app (UX completa)
2. Claude.app (se installata)
3. OpenClaw.app (broker leggero)

Usa `peekaboo bridge status --verbose` per vedere quale host è attivo e quale
percorso del socket è in uso. Puoi sovrascriverlo con:

```bash
export PEEKABOO_BRIDGE_SOCKET=/path/to/bridge.sock
```


<div id="security-permissions">
  ## Sicurezza e autorizzazioni
</div>

- Il bridge convalida le **firme del codice del chiamante**; viene applicata una lista di autorizzati di TeamID
  (TeamID dell'host Peekaboo + TeamID dell'app OpenClaw).
- Le richieste scadono dopo circa 10 secondi.
- Se mancano le autorizzazioni richieste, il bridge restituisce un messaggio di errore chiaro
  invece di aprire Impostazioni di Sistema.

<div id="snapshot-behavior-automation">
  ## Comportamento degli snapshot (automazione)
</div>

Gli snapshot vengono memorizzati in memoria e scadono automaticamente dopo un breve intervallo di tempo.
Se hai bisogno di una conservazione più lunga, riacquisisci lo snapshot dal client.

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

- Se `peekaboo` segnala “bridge client is not authorized”, assicurati che il client
  sia correttamente firmato oppure esegui l'host con `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1`
  solo in modalità **debug**.
- Se non viene trovato alcun host, apri una delle app host (Peekaboo.app o OpenClaw.app)
  e verifica che le autorizzazioni siano state concesse.