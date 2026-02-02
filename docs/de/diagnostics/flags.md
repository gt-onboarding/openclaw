---
title: Flags
summary: "Diagnose-Flags für gezielte Debug-Logs"
read_when:
  - Du benötigst gezielte Debug-Logs, ohne die globalen Logging-Levels zu erhöhen
  - Du musst subsystem-spezifische Logs für Supportzwecke erfassen
---

<div id="diagnostics-flags">
  # Diagnose-Flags
</div>

Diagnose-Flags ermöglichen dir, gezielte Debug-Logs zu aktivieren, ohne überall die ausführliche Protokollierung einschalten zu müssen. Flags sind opt-in und haben keine Wirkung, solange kein Subsystem sie auswertet.

<div id="how-it-works">
  ## Funktionsweise
</div>

* Flags sind Zeichenketten (Groß- und Kleinschreibung wird ignoriert).
* Sie können Flags in der Konfiguration oder über ein Env-Override aktivieren.
* Wildcards werden unterstützt:
  * `telegram.*` entspricht `telegram.http`
  * `*` aktiviert alle Flags

<div id="enable-via-config">
  ## Per Konfiguration aktivieren
</div>

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

Mehrere Flags:

```json
{
  "diagnostics": {
    "flags": ["telegram.http", "gateway.*"]
  }
}
```

Starte das Gateway nach dem Ändern der Flags neu.

<div id="env-override-one-off">
  ## Einmaliges Env-Override
</div>

```bash
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload
```

Deaktiviere alle Flags:

```bash
OPENCLAW_DIAGNOSTICS=0
```

<div id="where-logs-go">
  ## Wohin Logs geschrieben werden
</div>

Flags schreiben Logs in die Standard-Diagnose-Logdatei. Standardmäßig gilt:

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

Wenn du `logging.file` konfigurierst, verwende stattdessen diesen Pfad. Logs liegen im JSONL-Format vor (ein JSON-Objekt pro Zeile). Die Schwärzung sensibler Daten greift weiterhin und richtet sich nach `logging.redactSensitive`.

<div id="extract-logs">
  ## Logs extrahieren
</div>

Wähle die neueste Logdatei aus:

```bash
ls -t /tmp/openclaw/openclaw-*.log | head -n 1
```

Filter für Telegram-HTTP-Diagnosemeldungen:

```bash
rg "telegram http error" /tmp/openclaw/openclaw-*.log
```

Oder beim Reproduzieren mit `tail` verfolgen:

```bash
tail -f /tmp/openclaw/openclaw-$(date +%F).log | rg "telegram http error"
```

Für Remote-Gateways kannst du auch `openclaw logs --follow` verwenden (siehe [/cli/logs](/de/cli/logs)).

<div id="notes">
  ## Hinweise
</div>

* Wenn `logging.level` höher als `warn` konfiguriert ist, können diese Logs unterdrückt werden. Der Standardwert `info` ist ausreichend.
* Flags können gefahrlos aktiviert bleiben; sie beeinflussen nur das Logaufkommen für das jeweilige Subsystem.
* Verwende [/logging](/de/logging), um Log-Ziele, -Level und Schwärzung zu ändern.