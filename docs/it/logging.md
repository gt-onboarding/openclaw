---
title: Logging
summary: "Panoramica del logging: log su file, output in console, tail via CLI e Control UI"
read_when:
  - Hai bisogno di una panoramica di base sul logging
  - Vuoi configurare i livelli o i formati di log
  - Stai eseguendo attività di troubleshooting e devi individuare rapidamente i log
---

<div id="logging">
  # Log
</div>

OpenClaw scrive i log in due posizioni:

* **File di log** (righe JSON) scritti dal Gateway.
* **Output della console** mostrato nei terminali e nella Control UI.

Questa pagina spiega dove si trovano i log, come leggerli e come
configurare livelli e formati di log.

<div id="where-logs-live">
  ## Dove vengono salvati i log
</div>

Per impostazione predefinita, il Gateway scrive un file di log a rotazione in:

`/tmp/openclaw/openclaw-YYYY-MM-DD.log`

La data utilizza il fuso orario locale dell&#39;host del Gateway.

Puoi modificare questo percorso in `~/.openclaw/openclaw.json`:

```json
{
  "logging": {
    "file": "/path/to/openclaw.log"
  }
}
```

<div id="how-to-read-logs">
  ## Come leggere i log
</div>

<div id="cli-live-tail-recommended">
  ### CLI: tail in tempo reale (consigliato)
</div>

Usa la CLI per eseguire il tail del file di log del Gateway tramite RPC:

```bash
openclaw logs --follow
```

Modalità di output:

* **Sessioni TTY**: righe di log formattate, a colori e strutturate.
* **Sessioni non TTY**: testo semplice.
* `--json`: JSON delimitato per riga (un evento di log per riga).
* `--plain`: forza il testo semplice nelle sessioni TTY.
* `--no-color`: disabilita i colori ANSI.

In modalità JSON, la CLI emette oggetti etichettati con il campo `type`:

* `meta`: metadati del flusso (file, cursore, dimensione)
* `log`: voce di log analizzata
* `notice`: suggerimenti relativi a troncamento/rotazione
* `raw`: riga di log non elaborata

Se il Gateway non è raggiungibile, la CLI stampa un breve suggerimento che invita a eseguire:

```bash
openclaw doctor
```

<div id="control-ui-web">
  ### Control UI (web)
</div>

La scheda **Logs** della Control UI mostra in tempo reale lo stesso file usando `logs.tail`.
Vedi [/web/control-ui](/it/web/control-ui) per sapere come aprirla.

<div id="channel-only-logs">
  ### Log solo dei canali
</div>

Per filtrare l&#39;attività dei canali (WhatsApp/Telegram/etc), utilizza:

```bash
openclaw channels logs --channel whatsapp
```

<div id="log-formats">
  ## Formati dei log
</div>

<div id="file-logs-jsonl">
  ### Log su file (JSONL)
</div>

Ogni riga del file di log è un oggetto JSON. La CLI e la Control UI elaborano queste
voci per produrre un output strutturato (ora, livello, sottosistema, messaggio).

<div id="console-output">
  ### Output della console
</div>

I log della console sono **TTY-aware** e formattati per facilitarne la lettura:

* Prefissi dei sottosistemi (ad es. `gateway/channels/whatsapp`)
* Colorazione in base al livello (info/warn/error)
* Modalità compatta o JSON opzionale

La formattazione della console è controllata da `logging.consoleStyle`.

<div id="configuring-logging">
  ## Configurazione del logging
</div>

Tutta la configurazione del logging risiede nella sezione `logging` in `~/.openclaw/openclaw.json`.

```json
{
  "logging": {
    "level": "info",
    "file": "/tmp/openclaw/openclaw-YYYY-MM-DD.log",
    "consoleLevel": "info",
    "consoleStyle": "pretty",
    "redactSensitive": "tools",
    "redactPatterns": [
      "sk-.*"
    ]
  }
}
```

<div id="log-levels">
  ### Livelli di logging
</div>

* `logging.level`: livello dei **log su file** (JSONL).
* `logging.consoleLevel`: livello di verbosità della **console**.

`--verbose` influisce solo sull&#39;output della console; non modifica i livelli di log su file.

<div id="console-styles">
  ### Stili della console
</div>

`logging.consoleStyle`:

* `pretty`: leggibile, colorato, con timestamp.
* `compact`: output più compatto (ideale per sessioni lunghe).
* `json`: un JSON per riga (per i processori di log).

<div id="redaction">
  ### Offuscamento
</div>

I riepiloghi degli strumenti possono offuscare i token sensibili prima che raggiungano la console:

* `logging.redactSensitive`: `off` | `tools` (predefinito: `tools`)
* `logging.redactPatterns`: elenco di stringhe regex per sovrascrivere il set predefinito

L’offuscamento influisce **solo sull’output della console** e non modifica i log su file.

<div id="diagnostics-opentelemetry">
  ## Diagnostica + OpenTelemetry
</div>

La diagnostica consiste in eventi strutturati, in formato leggibile dalle macchine, relativi alle esecuzioni dei modelli **e**
alla telemetria del flusso dei messaggi (webhook, messa in coda, stato della sessione). Non
sostituisce i log; serve ad alimentare metriche, tracce e altri exporter.

Gli eventi di diagnostica vengono emessi all&#39;interno del processo, ma gli exporter si collegano solo quando
la diagnostica e il plugin dell&#39;exporter sono abilitati.

<div id="opentelemetry-vs-otlp">
  ### OpenTelemetry vs OTLP
</div>

* **OpenTelemetry (OTel)**: il modello di dati + gli SDK per tracce, metriche e log.
* **OTLP**: il protocollo di comunicazione utilizzato per esportare i dati OTel verso un collector/backend.
* OpenClaw oggi esporta tramite **OTLP/HTTP (protobuf)**.

<div id="signals-exported">
  ### Segnali esportati
</div>

* **Metriche**: contatori + istogrammi (utilizzo dei token, flusso dei messaggi, messa in coda).
* **Tracce**: span per l&#39;utilizzo del modello + l&#39;elaborazione di webhook/messaggi.
* **Log**: esportati tramite OTLP quando `diagnostics.otel.logs` è abilitato. Il volume
  dei log può essere elevato; tieni in considerazione `logging.level` e i filtri dell&#39;exporter.

<div id="diagnostic-event-catalog">
  ### Catalogo degli eventi diagnostici
</div>

Utilizzo del modello:

* `model.usage`: token, costo, durata, contesto, provider/modello/canale, ID sessione.

Flusso dei messaggi:

* `webhook.received`: webhook in ingresso per canale.
* `webhook.processed`: webhook gestito + durata.
* `webhook.error`: errori del gestore webhook.
* `message.queued`: messaggio messo in coda per l&#39;elaborazione.
* `message.processed`: esito + durata + errore opzionale.

Coda + sessione:

* `queue.lane.enqueue`: enqueue nella lane della coda di comandi + profondità.
* `queue.lane.dequeue`: dequeue nella lane della coda di comandi + tempo di attesa.
* `session.state`: transizione di stato della sessione + motivo.
* `session.stuck`: avviso di sessione bloccata + anzianità.
* `run.attempt`: metadati sul tentativo/ritentativo di esecuzione.
* `diagnostic.heartbeat`: contatori aggregati (webhook/coda/sessione).

<div id="enable-diagnostics-no-exporter">
  ### Abilita diagnostica (senza exporter)
</div>

Utilizza questa opzione se vuoi che gli eventi di diagnostica siano disponibili per i plugin o per sink personalizzati:

```json
{
  "diagnostics": {
    "enabled": true
  }
}
```

<div id="diagnostics-flags-targeted-logs">
  ### Flag di diagnostica (log mirati)
</div>

Usa i flag per attivare log di debug aggiuntivi e mirati senza aumentare il valore di `logging.level`.
I flag non fanno distinzione tra maiuscole e minuscole e supportano i caratteri jolly (ad es. `telegram.*` o `*`).

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

Override dell&#39;ambiente (una sola volta):

```
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload
```

Note:

* I log dei flag vengono scritti nel file di log standard (lo stesso di `logging.file`).
* L&#39;output viene comunque censurato in base a `logging.redactSensitive`.
* Guida completa: [/diagnostics/flags](/it/diagnostics/flags).

<div id="export-to-opentelemetry">
  ### Esportazione in OpenTelemetry
</div>

I dati di diagnostica possono essere esportati tramite il plugin `diagnostics-otel` (OTLP/HTTP). Funziona con qualsiasi collector/backend OpenTelemetry che supporti OTLP/HTTP.

```json
{
  "plugins": {
    "allow": ["diagnostics-otel"],
    "entries": {
      "diagnostics-otel": {
        "enabled": true
      }
    }
  },
  "diagnostics": {
    "enabled": true,
    "otel": {
      "enabled": true,
      "endpoint": "http://otel-collector:4318",
      "protocol": "http/protobuf",
      "serviceName": "openclaw-gateway",
      "traces": true,
      "metrics": true,
      "logs": true,
      "sampleRate": 0.2,
      "flushIntervalMs": 60000
    }
  }
}
```

Note:

* Puoi anche abilitare il plugin con `openclaw plugins enable diagnostics-otel`.
* `protocol` al momento supporta solo `http/protobuf`. `grpc` viene ignorato.
* Le metriche includono utilizzo di token, costo, dimensione del contesto, durata
  dell&#39;esecuzione e contatori/istogrammi del flusso dei messaggi (webhook, accodamento, stato della sessione, profondità/attesa della coda).
* Tracce/metriche possono essere abilitate o disabilitate con `traces` / `metrics` (predefinito: on). Le tracce includono gli span di utilizzo del modello più gli span di elaborazione di webhook/messaggi quando sono abilitate.
* Imposta `headers` quando il collector richiede autenticazione.
* Variabili d&#39;ambiente supportate: `OTEL_EXPORTER_OTLP_ENDPOINT`,
  `OTEL_SERVICE_NAME`, `OTEL_EXPORTER_OTLP_PROTOCOL`.

<div id="exported-metrics-names-types">
  ### Metriche esportate (nomi + tipi)
</div>

Utilizzo dei modelli:

* `openclaw.tokens` (contatore, attributi: `openclaw.token`, `openclaw.channel`,
  `openclaw.provider`, `openclaw.model`)
* `openclaw.cost.usd` (contatore, attributi: `openclaw.channel`, `openclaw.provider`,
  `openclaw.model`)
* `openclaw.run.duration_ms` (istogramma, attributi: `openclaw.channel`,
  `openclaw.provider`, `openclaw.model`)
* `openclaw.context.tokens` (istogramma, attributi: `openclaw.context`,
  `openclaw.channel`, `openclaw.provider`, `openclaw.model`)

Flusso dei messaggi:

* `openclaw.webhook.received` (contatore, attributi: `openclaw.channel`,
  `openclaw.webhook`)
* `openclaw.webhook.error` (contatore, attributi: `openclaw.channel`,
  `openclaw.webhook`)
* `openclaw.webhook.duration_ms` (istogramma, attributi: `openclaw.channel`,
  `openclaw.webhook`)
* `openclaw.message.queued` (contatore, attributi: `openclaw.channel`,
  `openclaw.source`)
* `openclaw.message.processed` (contatore, attributi: `openclaw.channel`,
  `openclaw.outcome`)
* `openclaw.message.duration_ms` (istogramma, attributi: `openclaw.channel`,
  `openclaw.outcome`)

Code + sessioni:

* `openclaw.queue.lane.enqueue` (contatore, attributi: `openclaw.lane`)
* `openclaw.queue.lane.dequeue` (contatore, attributi: `openclaw.lane`)
* `openclaw.queue.depth` (istogramma, attributi: `openclaw.lane` oppure
  `openclaw.channel=heartbeat`)
* `openclaw.queue.wait_ms` (istogramma, attributi: `openclaw.lane`)
* `openclaw.session.state` (contatore, attributi: `openclaw.state`, `openclaw.reason`)
* `openclaw.session.stuck` (contatore, attributi: `openclaw.state`)
* `openclaw.session.stuck_age_ms` (istogramma, attributi: `openclaw.state`)
* `openclaw.run.attempt` (contatore, attributi: `openclaw.attempt`)

<div id="exported-spans-names-key-attributes">
  ### Spans esportati (nomi + attributi chiave)
</div>

* `openclaw.model.usage`
  * `openclaw.channel`, `openclaw.provider`, `openclaw.model`
  * `openclaw.sessionKey`, `openclaw.sessionId`
  * `openclaw.tokens.*` (input/output/cache&#95;read/cache&#95;write/total)
* `openclaw.webhook.processed`
  * `openclaw.channel`, `openclaw.webhook`, `openclaw.chatId`
* `openclaw.webhook.error`
  * `openclaw.channel`, `openclaw.webhook`, `openclaw.chatId`,
    `openclaw.error`
* `openclaw.message.processed`
  * `openclaw.channel`, `openclaw.outcome`, `openclaw.chatId`,
    `openclaw.messageId`, `openclaw.sessionKey`, `openclaw.sessionId`,
    `openclaw.reason`
* `openclaw.session.stuck`
  * `openclaw.state`, `openclaw.ageMs`, `openclaw.queueDepth`,
    `openclaw.sessionKey`, `openclaw.sessionId`

<div id="sampling-flushing">
  ### Campionamento + flush
</div>

* Campionamento delle tracce: `diagnostics.otel.sampleRate` (0,0–1,0, solo span radice).
* Intervallo di esportazione delle metriche: `diagnostics.otel.flushIntervalMs` (min 1000 ms).

<div id="protocol-notes">
  ### Note sul protocollo
</div>

* Gli endpoint OTLP/HTTP possono essere impostati tramite `diagnostics.otel.endpoint` o
  `OTEL_EXPORTER_OTLP_ENDPOINT`.
* Se l&#39;endpoint contiene già `/v1/traces` o `/v1/metrics`, viene usato così com&#39;è.
* Se l&#39;endpoint contiene già `/v1/logs`, viene usato così com&#39;è per i log.
* `diagnostics.otel.logs` abilita l&#39;esportazione dei log OTLP per l&#39;output del logger principale.

<div id="log-export-behavior">
  ### Comportamento di esportazione dei log
</div>

* I log OTLP utilizzano gli stessi record strutturati scritti in `logging.file`.
* I log OTLP rispettano `logging.level` (livello di log del file). Il mascheramento in console **non** si applica
  ai log OTLP.
* Le installazioni con volumi elevati dovrebbero preferire il campionamento/filtraggio tramite un collector OTLP.

<div id="troubleshooting-tips">
  ## Suggerimenti per la risoluzione dei problemi
</div>

* **Gateway non raggiungibile?** Esegui prima `openclaw doctor`.
* **Log vuoti?** Verifica che il Gateway sia in esecuzione e stia scrivendo nel percorso del file indicato in `logging.file`.
* **Hai bisogno di maggiori dettagli?** Imposta `logging.level` su `debug` o `trace` e riprova.