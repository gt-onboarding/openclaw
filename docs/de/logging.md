---
title: Logging
summary: "Logging-Übersicht: Datei-Logs, Konsolenausgabe, CLI-Tailing und die Control UI"
read_when:
  - Du brauchst einen einsteigerfreundlichen Überblick über das Logging
  - Du möchtest Log-Level oder -Formate konfigurieren
  - Du bist bei der Fehlersuche und musst Logs schnell finden
---

<div id="logging">
  # Logging
</div>

OpenClaw schreibt Protokolle an zwei Stellen:

* **Protokolldateien** (JSON-Lines), die vom Gateway geschrieben werden.
* **Konsolenausgabe**, die in Terminals und der Control UI angezeigt wird.

Diese Seite erklärt, wo die Protokolle liegen, wie du sie lesen kannst und wie du Log-Level und -formate konfigurierst.

<div id="where-logs-live">
  ## Wo Logs gespeichert werden
</div>

Standardmäßig schreibt das Gateway eine rotierende Logdatei nach:

`/tmp/openclaw/openclaw-YYYY-MM-DD.log`

Die Datumsangabe verwendet die lokale Zeitzone des Gateway-Hosts.

Du kannst dies in `~/.openclaw/openclaw.json` überschreiben:

```json
{
  "logging": {
    "file": "/path/to/openclaw.log"
  }
}
```

<div id="how-to-read-logs">
  ## So lesen Sie Logs
</div>

<div id="cli-live-tail-recommended">
  ### CLI: Live-Tail (empfohlen)
</div>

Verwende die CLI, um die Gateway-Logdatei über RPC live zu verfolgen:

```bash
openclaw logs --follow
```

Ausgabemodi:

* **TTY-Sitzungen**: ansprechende, farbige, strukturierte Log-Zeilen.
* **Nicht-TTY-Sitzungen**: Klartext.
* `--json`: zeilenweise JSON (ein Log-Ereignis pro Zeile).
* `--plain`: erzwingt Klartext in TTY-Sitzungen.
* `--no-color`: deaktiviert ANSI-Farben.

Im JSON-Modus gibt die CLI Objekte mit `type`-Tag aus:

* `meta`: Stream-Metadaten (Datei, Cursor, Größe)
* `log`: ausgewerteter Log-Eintrag
* `notice`: Hinweise zur Kürzung/Rotation
* `raw`: nicht ausgewertete Log-Zeile

Wenn das Gateway nicht erreichbar ist, gibt die CLI einen kurzen Hinweis aus, diesen Befehl auszuführen:

```bash
openclaw doctor
```

<div id="control-ui-web">
  ### Control UI (web)
</div>

Der Tab **Logs** in der Control UI zeigt dieselbe Datei in Echtzeit mit `logs.tail` an.
Weitere Informationen zum Öffnen finden Sie unter [/web/control-ui](/de/web/control-ui).

<div id="channel-only-logs">
  ### Channel-spezifische Logs
</div>

Um die Channel-Aktivität (WhatsApp/Telegram/usw.) zu filtern, verwende:

```bash
openclaw channels logs --channel whatsapp
```

<div id="log-formats">
  ## Protokollformate
</div>

<div id="file-logs-jsonl">
  ### Datei-Logs (JSONL)
</div>

Jede Zeile in der Logdatei ist ein JSON-Objekt. Die CLI und die Control UI parsen diese
Einträge, um eine strukturierte Ausgabe (Zeitstempel, Level, Subsystem, Meldung) zu erzeugen.

<div id="console-output">
  ### Konsolenausgabe
</div>

Konsolen-Logs sind **TTY-fähig** und für gute Lesbarkeit formatiert:

* Subsystem-Präfixe (z. B. `gateway/channels/whatsapp`)
* Farbige Level-Anzeige (info/warn/error)
* Optional kompaktes oder JSON-Format

Die Konsolenformatierung wird über `logging.consoleStyle` gesteuert.

<div id="configuring-logging">
  ## Logging konfigurieren
</div>

Sämtliche Logging-Konfiguration befindet sich unter `logging` in `~/.openclaw/openclaw.json`.

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
  ### Log-Level
</div>

* `logging.level`: Protokollstufe für **Dateilogs** (JSONL).
* `logging.consoleLevel`: Protokollstufe/Detailgrad der **Konsole**.

`--verbose` wirkt sich nur auf die Konsolenausgabe aus; die Protokollstufen für Dateilogs werden dadurch nicht geändert.

<div id="console-styles">
  ### Konsolenstile
</div>

`logging.consoleStyle`:

* `pretty`: benutzerfreundlich, farbig, mit Zeitstempeln.
* `compact`: kompaktere Ausgabe (am besten für lange Sitzungen).
* `json`: JSON pro Zeile (für Logprozessoren).

<div id="redaction">
  ### Maskierung
</div>

Tool-Zusammenfassungen können sensible Token maskieren, bevor sie in der Konsole ausgegeben werden:

* `logging.redactSensitive`: `off` | `tools` (Standard: `tools`)
* `logging.redactPatterns`: Liste von Regex-Zeichenketten, um den Standardsatz zu überschreiben

Die Maskierung betrifft **nur die Konsolenausgabe** und ändert keine Logdateien.

<div id="diagnostics-opentelemetry">
  ## Diagnostik + OpenTelemetry
</div>

Diagnostikereignisse sind strukturierte, maschinenlesbare Events für Modellläufe **und**
Nachrichtenfluss-Telemetrie (Webhooks, Queueing, Sitzungszustand). Sie **ersetzen nicht**
Logs; sie dienen dazu, Metriken, Traces und andere Exporter zu speisen.

Diagnostikereignisse werden im Prozess erzeugt, aber Exporter werden nur angebunden,
wenn Diagnostik + das Exporter-Plugin aktiviert sind.

<div id="opentelemetry-vs-otlp">
  ### OpenTelemetry vs OTLP
</div>

* **OpenTelemetry (OTel)**: das Datenmodell + SDKs für Traces, Metriken und Logs.
* **OTLP**: das Übertragungsprotokoll („wire protocol“), mit dem OTel-Daten an einen Collector bzw. ein Backend exportiert werden.
* OpenClaw exportiert derzeit über **OTLP/HTTP (protobuf)**.

<div id="signals-exported">
  ### Exportierte Signale
</div>

* **Metriken**: Zähler + Histogramme (Token-Nutzung, Nachrichtenfluss, Queueing).
* **Traces**: Spans für Modellnutzung + Webhook-/Nachrichtenverarbeitung.
* **Logs**: werden über OTLP exportiert, wenn `diagnostics.otel.logs` aktiviert ist. Das Log-Volumen kann hoch sein; behalte `logging.level` und Exporter-Filter im Hinterkopf.

<div id="diagnostic-event-catalog">
  ### Katalog diagnostischer Ereignisse
</div>

Modellnutzung:

* `model.usage`: Tokens, Kosten, Dauer, Kontext, Anbieter/Modell/Kanal, Sitzungs-IDs.

Nachrichtenfluss:

* `webhook.received`: Webhook-Eingang pro Kanal.
* `webhook.processed`: Webhook verarbeitet + Dauer.
* `webhook.error`: Fehler im Webhook-Handler.
* `message.queued`: Nachricht zur Verarbeitung in die Warteschlange eingereiht.
* `message.processed`: Ergebnis + Dauer + optionaler Fehler.

Warteschlange + Sitzung:

* `queue.lane.enqueue`: Einreihung in eine Lane der Befehlswarteschlange + Tiefe.
* `queue.lane.dequeue`: Entnahme aus einer Lane der Befehlswarteschlange + Wartezeit.
* `session.state`: Übergang des Sitzungszustands + Grund.
* `session.stuck`: Warnung zu blockierter Sitzung + Alter.
* `run.attempt`: Metadaten zu Laufversuchen/Wiederholungen.
* `diagnostic.heartbeat`: aggregierte Zähler (Webhooks/Warteschlange/Sitzung).

<div id="enable-diagnostics-no-exporter">
  ### Diagnose aktivieren (ohne Exporter)
</div>

Verwende dies, wenn du Diagnoseereignisse für Plugins oder benutzerdefinierte Sinks verfügbar machen möchtest:

```json
{
  "diagnostics": {
    "enabled": true
  }
}
```

<div id="diagnostics-flags-targeted-logs">
  ### Diagnose-Flags (gezielte Logs)
</div>

Verwende Flags, um zusätzliche, gezielte Debug-Logs zu aktivieren, ohne `logging.level` zu erhöhen.
Flags unterscheiden nicht zwischen Groß- und Kleinschreibung und unterstützen Wildcards (z. B. `telegram.*` oder `*`).

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

Env-Override (einmalig):

```
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload
```

Hinweise:

* Flag-Logs werden in die Standard-Logdatei geschrieben (wie `logging.file`).
* Die Ausgabe wird weiterhin gemäß `logging.redactSensitive` maskiert.
* Vollständige Anleitung: [/diagnostics/flags](/de/diagnostics/flags).

<div id="export-to-opentelemetry">
  ### Export zu OpenTelemetry
</div>

Diagnosedaten können über das `diagnostics-otel`-Plugin (OTLP/HTTP) exportiert werden. Dies
funktioniert mit jedem OpenTelemetry-Collector/-Backend, das OTLP/HTTP akzeptiert.

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

Hinweise:

* Du kannst das Plugin auch mit `openclaw plugins enable diagnostics-otel` aktivieren.
* `protocol` unterstützt derzeit nur `http/protobuf`. `grpc` wird ignoriert.
* Metriken umfassen Tokenverbrauch, Kosten, Kontextgröße, Laufdauer und Nachrichtenfluss-
  Zähler/Histogramme (Webhooks, Warteschlangenverarbeitung, Sitzungszustand, Warteschlangentiefe/-wartezeit).
* Traces/Metriken kannst du mit `traces` / `metrics` ein- und ausschalten (Standard: an). Traces
  enthalten Spans zur Modellausführung plus Spans zur Webhook-/Nachrichtenverarbeitung, wenn aktiviert.
* Setze `headers`, wenn dein Collector eine Authentifizierung erfordert.
* Unterstützte Umgebungsvariablen: `OTEL_EXPORTER_OTLP_ENDPOINT`,
  `OTEL_SERVICE_NAME`, `OTEL_EXPORTER_OTLP_PROTOCOL`.

<div id="exported-metrics-names-types">
  ### Exportierte Metriken (Namen + Typen)
</div>

Modellnutzung:

* `openclaw.tokens` (Zähler, Attribute: `openclaw.token`, `openclaw.channel`,
  `openclaw.provider`, `openclaw.model`)
* `openclaw.cost.usd` (Zähler, Attribute: `openclaw.channel`, `openclaw.provider`,
  `openclaw.model`)
* `openclaw.run.duration_ms` (Histogramm, Attribute: `openclaw.channel`,
  `openclaw.provider`, `openclaw.model`)
* `openclaw.context.tokens` (Histogramm, Attribute: `openclaw.context`,
  `openclaw.channel`, `openclaw.provider`, `openclaw.model`)

Nachrichtenfluss:

* `openclaw.webhook.received` (Zähler, Attribute: `openclaw.channel`,
  `openclaw.webhook`)
* `openclaw.webhook.error` (Zähler, Attribute: `openclaw.channel`,
  `openclaw.webhook`)
* `openclaw.webhook.duration_ms` (Histogramm, Attribute: `openclaw.channel`,
  `openclaw.webhook`)
* `openclaw.message.queued` (Zähler, Attribute: `openclaw.channel`,
  `openclaw.source`)
* `openclaw.message.processed` (Zähler, Attribute: `openclaw.channel`,
  `openclaw.outcome`)
* `openclaw.message.duration_ms` (Histogramm, Attribute: `openclaw.channel`,
  `openclaw.outcome`)

Warteschlangen und Sitzungen:

* `openclaw.queue.lane.enqueue` (Zähler, Attribute: `openclaw.lane`)
* `openclaw.queue.lane.dequeue` (Zähler, Attribute: `openclaw.lane`)
* `openclaw.queue.depth` (Histogramm, Attribute: `openclaw.lane` oder
  `openclaw.channel=heartbeat`)
* `openclaw.queue.wait_ms` (Histogramm, Attribute: `openclaw.lane`)
* `openclaw.session.state` (Zähler, Attribute: `openclaw.state`, `openclaw.reason`)
* `openclaw.session.stuck` (Zähler, Attribute: `openclaw.state`)
* `openclaw.session.stuck_age_ms` (Histogramm, Attribute: `openclaw.state`)
* `openclaw.run.attempt` (Zähler, Attribute: `openclaw.attempt`)

<div id="exported-spans-names-key-attributes">
  ### Exportierte Spans (Namen und Schlüsselattribute)
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
  ### Sampling + Flush
</div>

* Trace-Sampling: `diagnostics.otel.sampleRate` (0,0–1,0, nur Root-Spans).
* Metrik-Exportintervall: `diagnostics.otel.flushIntervalMs` (mind. 1000 ms).

<div id="protocol-notes">
  ### Protokollhinweise
</div>

* OTLP/HTTP-Endpunkte können über `diagnostics.otel.endpoint` oder
  `OTEL_EXPORTER_OTLP_ENDPOINT` konfiguriert werden.
* Wenn der Endpunkt bereits `/v1/traces` oder `/v1/metrics` enthält, wird er unverändert verwendet.
* Wenn der Endpunkt bereits `/v1/logs` enthält, wird er für Logs unverändert verwendet.
* `diagnostics.otel.logs` aktiviert den Export von OTLP-Logs für die Ausgabe des Hauptloggers.

<div id="log-export-behavior">
  ### Verhalten beim Log-Export
</div>

* OTLP-Logs verwenden dieselben strukturierten Einträge, die in `logging.file` geschrieben werden.
* `logging.level` (Datei-Log-Level) wird berücksichtigt. Maskierungen in der Konsole gelten **nicht**
  für OTLP-Logs.
* Installationen mit hohem Log-Aufkommen sollten Sampling/Filterung durch einen OTLP-Collector bevorzugen.

<div id="troubleshooting-tips">
  ## Tipps zur Fehlerbehebung
</div>

* **Gateway nicht erreichbar?** Führe zuerst `openclaw doctor` aus.
* **Logs leer?** Überprüfe, ob der Gateway läuft und in den in `logging.file` angegebenen Dateipfad schreibt.
* **Mehr Details nötig?** Setze `logging.level` auf `debug` oder `trace` und versuche es erneut.