---
title: Debugging
summary: "Strumenti di debug: modalità watch, stream grezzi del modello e tracciamento delle fughe di ragionamento"
read_when:
  - Devi ispezionare l'output grezzo del modello per individuare fughe di ragionamento
  - Vuoi eseguire il Gateway in modalità watch mentre iteri
  - Ti serve un workflow di debug ripetibile
---

<div id="debugging">
  # Debug
</div>

Questa pagina illustra gli strumenti di debug per l'output in streaming, in particolare quando un provider mescola il ragionamento al testo normale.

<div id="runtime-debug-overrides">
  ## Override di debug a runtime
</div>

Usa `/debug` in chat per impostare override **solo a runtime** della configurazione (in memoria, non su disco).
`/debug` è disabilitato per impostazione predefinita; abilitalo con `commands.debug: true`.
Questo è utile quando devi attivare o disattivare impostazioni poco comuni senza modificare `openclaw.json`.

Esempi:

```
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug unset messages.responsePrefix
/debug reset
```

`/debug reset` cancella tutti gli override e ripristina la configurazione presente su disco.


<div id="gateway-watch-mode">
  ## Modalità watch del Gateway
</div>

Per iterare velocemente, esegui il Gateway con il watcher dei file attivo:

```bash
pnpm gateway:watch --force
```

Questo corrisponde a:

```bash
tsx watch src/entry.ts gateway --force
```

Aggiungi eventuali flag CLI del Gateway dopo `gateway:watch` e verranno inoltrati
a ogni riavvio.


<div id="dev-profile-dev-gateway-dev">
  ## Profilo dev + dev gateway (--dev)
</div>

Usa il profilo di sviluppo per isolare lo stato e avviare una configurazione sicura e usa e getta per
il debug. Ci sono **due** flag `--dev`:

* **`--dev` globale (profilo):** isola lo stato in `~/.openclaw-dev` e
  imposta come predefinita la porta del Gateway a `19001` (le porte derivate si spostano di conseguenza).
* **`gateway --dev`: indica al Gateway di creare automaticamente una configurazione predefinita +
  spazio di lavoro** se mancano (e di ignorare BOOTSTRAP.md).

Procedura consigliata (profilo dev + dev bootstrap):

```bash
pnpm gateway:dev
OPENCLAW_PROFILE=dev openclaw tui
```

Se non hai ancora un’installazione globale, esegui la CLI tramite `pnpm openclaw ...`.

Cosa fa:

1. **Isolamento del profilo** (global `--dev`)
   * `OPENCLAW_PROFILE=dev`
   * `OPENCLAW_STATE_DIR=~/.openclaw-dev`
   * `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
   * `OPENCLAW_GATEWAY_PORT=19001` (con conseguente spostamento di browser/canvas)

2. **Bootstrap di sviluppo** (`gateway --dev`)
   * Scrive una configurazione minima se mancante (`gateway.mode=local`, bind su loopback).
   * Imposta `agent.workspace` sullo spazio di lavoro di sviluppo.
   * Imposta `agent.skipBootstrap=true` (nessun BOOTSTRAP.md).
   * Inizializza i file nello spazio di lavoro se mancanti:
     `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`.
   * Identità predefinita: **C3‑PO** (droide protocollare).
   * Salta i provider dei canali in modalità sviluppo (`OPENCLAW_SKIP_CHANNELS=1`).

Flusso di reset (partenza da zero):

```bash
pnpm gateway:dev:reset
```

Nota: `--dev` è un flag di profilo **globale** e alcuni runner lo intercettano e non lo propagano.
Se devi specificarlo esplicitamente, usa la forma tramite variabile d&#39;ambiente:

```bash
OPENCLAW_PROFILE=dev openclaw gateway --dev --reset
```

`--reset` elimina configurazione, credenziali, sessioni e lo spazio di lavoro di sviluppo (usando
`trash`, non `rm`), quindi ricrea la configurazione di sviluppo predefinita.

Suggerimento: se un Gateway non di sviluppo è già in esecuzione (launchd/systemd), arrestalo prima:

```bash
openclaw gateway stop
```


<div id="raw-stream-logging-openclaw">
  ## Registrazione del flusso grezzo (OpenClaw)
</div>

OpenClaw può registrare il **flusso grezzo dell&#39;assistente** prima di qualsiasi filtraggio/formattazione.
Questo è il modo migliore per verificare se il ragionamento arriva come delta di testo in chiaro
(o come blocchi di “pensiero” separati).

Abilitala tramite CLI:

```bash
pnpm gateway:watch --force --raw-stream
```

Percorso alternativo opzionale:

```bash
pnpm gateway:watch --force --raw-stream --raw-stream-path ~/.openclaw/logs/raw-stream.jsonl
```

Variabili di ambiente equivalenti:

```bash
OPENCLAW_RAW_STREAM=1
OPENCLAW_RAW_STREAM_PATH=~/.openclaw/logs/raw-stream.jsonl
```

File predefinito:

`~/.openclaw/logs/raw-stream.jsonl`


<div id="raw-chunk-logging-pi-mono">
  ## Registrazione dei chunk grezzi (pi-mono)
</div>

Per catturare i **chunk grezzi compatibili con OpenAI** prima che vengano elaborati in blocchi,
pi-mono espone un logger separato:

```bash
PI_RAW_STREAM=1
```

Percorso facoltativo:

```bash
PI_RAW_STREAM_PATH=~/.pi-mono/logs/raw-openai-completions.jsonl
```

File predefinito:

`~/.pi-mono/logs/raw-openai-completions.jsonl`

> Nota: questo file viene prodotto solo dai processi che utilizzano il provider
> `openai-completions` di pi-mono.


<div id="safety-notes">
  ## Note sulla sicurezza
</div>

- I log grezzi dello streaming possono includere prompt completi, output degli strumenti e dati degli utenti.
- Mantieni i log solo in locale ed eliminali dopo il debugging.
- Se condividi i log, rimuovi prima segreti e dati personali identificabili (PII).