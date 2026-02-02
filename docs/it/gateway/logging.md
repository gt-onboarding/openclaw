---
title: Logging
summary: "Interfacce di logging, log su file, stili di log WS e formattazione della console"
read_when:
  - Modificare l'output o i formati di logging
  - Effettuare il debug dell'output della CLI o del Gateway
---

<div id="logging">
  # Logging
</div>

Per una panoramica rivolta agli utenti (CLI + Control UI + config), consulta [/logging](/it/logging).

OpenClaw ha due “superfici” di logging:

* **Output della console** (ciò che vedi nel terminale / Debug UI).
* **Log su file** (righe JSON) scritti dal logger del Gateway.

<div id="file-based-logger">
  ## Logger basato su file
</div>

* Il file di log a rotazione predefinito si trova in `/tmp/openclaw/` (un file al giorno): `openclaw-YYYY-MM-DD.log`
  * La data utilizza il fuso orario locale dell&#39;host del Gateway.
* Il percorso e il livello del file di log possono essere configurati tramite `~/.openclaw/openclaw.json`:
  * `logging.file`
  * `logging.level`

Il formato del file prevede un oggetto JSON per riga.

La scheda Logs della Control UI segue in tempo reale questo file tramite il Gateway (`logs.tail`).
La CLI può fare lo stesso:

```bash
openclaw logs --follow
```

**Verbose vs. livelli di log**

* I **log su file** sono controllati esclusivamente da `logging.level`.
* `--verbose` influisce solo sulla **verbosità della console** (e sullo stile dei log WS); **non**
  aumenta il livello di log su file.
* Per acquisire nei log su file i dettagli visibili solo in modalità verbosa, imposta `logging.level` su `debug` o
  `trace`.

<div id="console-capture">
  ## Acquisizione della console
</div>

La CLI cattura `console.log/info/warn/error/debug/trace` e li scrive nei log su file,
continuando comunque a stampare su stdout/stderr.

Puoi regolare in modo indipendente la verbosità della console tramite:

* `logging.consoleLevel` (predefinito `info`)
* `logging.consoleStyle` (`pretty` | `compact` | `json`)

<div id="tool-summary-redaction">
  ## Redazione dei riepiloghi degli strumenti
</div>

I riepiloghi dettagliati degli strumenti (ad esempio `🛠️ Exec: ...`) possono mascherare i token sensibili prima che raggiungano il flusso della console. Questa opzione è **solo per gli strumenti** e non modifica i log su file.

* `logging.redactSensitive`: `off` | `tools` (predefinito: `tools`)
* `logging.redactPatterns`: array di stringhe regex (sovrascrive i valori predefiniti)
  * Usa stringhe regex grezze (auto `gi`), oppure `/pattern/flags` se ti servono flag personalizzati.
  * Le corrispondenze vengono mascherate mantenendo i primi 6 + gli ultimi 4 caratteri (lunghezza &gt;= 18), altrimenti `***`.
  * I valori predefiniti coprono assegnazioni di chiavi comuni, flag CLI, campi JSON, header Bearer, blocchi PEM e prefissi di token più diffusi.

<div id="gateway-websocket-logs">
  ## Log WebSocket del Gateway
</div>

Il Gateway registra i log del protocollo WebSocket in due modalità:

* **Modalità normale (senza `--verbose`)**: vengono registrati solo i risultati RPC rilevanti:
  * errori (`ok=false`)
  * chiamate lente (soglia predefinita: `>= 50ms`)
  * errori di parsing
* **Modalità verbosa (`--verbose`)**: registra tutto il traffico di richiesta/risposta WS.

<div id="ws-log-style">
  ### Stile di log WS
</div>

`openclaw gateway` supporta uno switch di stile per singolo gateway:

* `--ws-log auto` (predefinito): la modalità normale è ottimizzata; la modalità verbosa usa output compatto
* `--ws-log compact`: output compatto (richiesta/risposta abbinate) quando in modalità verbosa
* `--ws-log full`: output completo per frame quando in modalità verbosa
* `--compact`: alias di `--ws-log compact`

Esempi:

```bash
# ottimizzato (solo errori/lenti)
openclaw gateway

# mostra tutto il traffico WS (abbinato)
openclaw gateway --verbose --ws-log compact

# mostra tutto il traffico WS (metadati completi)
openclaw gateway --verbose --ws-log full
```

<div id="console-formatting-subsystem-logging">
  ## Formattazione della console (logging dei sottosistemi)
</div>

Il formatter della console è **TTY-aware** e stampa righe coerenti con prefisso.
I logger dei sottosistemi mantengono l&#39;output raggruppato e facile da scorrere.

Comportamento:

* **Prefissi di sottosistema** su ogni riga (ad es. `[gateway]`, `[canvas]`, `[tailscale]`)
* **Colori per sottosistema** (stabili per ogni sottosistema) più colorazione per livello
* **Usa i colori quando l&#39;output è un TTY o l&#39;ambiente sembra un terminale avanzato** (`TERM`/`COLORTERM`/`TERM_PROGRAM`), rispetta `NO_COLOR`
* **Prefissi di sottosistema abbreviati**: rimuove i prefissi iniziali `gateway/` + `channels/`, mantiene gli ultimi 2 segmenti (ad es. `whatsapp/outbound`)
* **Logger secondari per sottosistema** (prefisso automatico + campo strutturato `{ subsystem }`)
* **`logRaw()`** per output QR/UX (nessun prefisso, nessuna formattazione)
* **Stili della console** (ad es. `pretty | compact | json`)
* **Livello di log della console** separato dal livello di log del file (il file mantiene tutti i dettagli quando `logging.level` è impostato su `debug`/`trace`)
* **Il corpo dei messaggi WhatsApp** viene registrato a livello `debug` (usa `--verbose` per visualizzarli)

Questo mantiene stabili i log su file esistenti rendendo al contempo l&#39;output interattivo facile da scorrere.