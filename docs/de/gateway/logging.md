---
title: Logging
summary: "Log-Ausgaben, Datei-Logs, WS-Logstile und Konsolenformatierung"
read_when:
  - Ändern der Logging-Ausgabe oder -Formate
  - Debuggen der CLI- oder Gateway-Ausgabe
---

<div id="logging">
  # Logging
</div>

Für einen benutzerorientierten Überblick (CLI + Control UI + Konfiguration) siehe [/logging](/de/logging).

OpenClaw stellt zwei Arten von Logs bereit:

* **Konsolenausgabe** (was du im Terminal bzw. in der Debug-UI siehst).
* **Datei-Logs** (JSON-Zeilen), die vom Gateway-Logger geschrieben werden.

<div id="file-based-logger">
  ## Dateibasierter Logger
</div>

* Die standardmäßige Logdatei mit Rotation liegt unter `/tmp/openclaw/` (eine Datei pro Tag): `openclaw-YYYY-MM-DD.log`
  * Das Datum verwendet die lokale Zeitzone des Gateway-Hosts.
* Der Pfad der Logdatei und der Loglevel können über `~/.openclaw/openclaw.json` konfiguriert werden:
  * `logging.file`
  * `logging.level`

Das Dateiformat ist: eine JSON-Objektzeile pro Eintrag.

Der Tab „Logs“ in der Control UI folgt dieser Datei über das Gateway (`logs.tail`).
Die CLI kann dasselbe tun:

```bash
openclaw logs --follow
```

**Verbose vs. Log-Level**

* **Datei-Logs** werden ausschließlich über `logging.level` gesteuert.
* `--verbose` beeinflusst nur die **Ausführlichkeit der Konsolenausgabe** (und den WS-Logstil); es **erhöht nicht**
  den Log-Level der Datei-Logs.
* Um Details, die nur im Verbose-Modus erscheinen, auch in Datei-Logs zu erfassen, setze `logging.level` auf `debug` oder
  `trace`.

<div id="console-capture">
  ## Erfassung der Konsolenausgabe
</div>

Die CLI fängt `console.log/info/warn/error/debug/trace` ab und schreibt sie in Protokolldateien,
während weiterhin auf stdout/stderr ausgegeben wird.

Sie können die Ausführlichkeit der Konsole unabhängig einstellen über:

* `logging.consoleLevel` (Standardwert `info`)
* `logging.consoleStyle` (`pretty` | `compact` | `json`)

<div id="tool-summary-redaction">
  ## Schwärzung von Tool-Zusammenfassungen
</div>

Ausführliche Tool-Zusammenfassungen (z. B. `🛠️ Exec: ...`) können sensible Token maskieren, bevor sie im
Konsolenstream erscheinen. Dies gilt **nur für Tools** und ändert keine Dateilogs.

* `logging.redactSensitive`: `off` | `tools` (Standard: `tools`)
* `logging.redactPatterns`: Array von Regex-Strings (überschreibt Standardwerte)
  * Verwende rohe Regex-Strings (automatisch `gi`), oder `/pattern/flags`, wenn du benutzerdefinierte Flags brauchst.
  * Treffer werden maskiert, indem die ersten 6 + letzten 4 Zeichen beibehalten werden (Länge &gt;= 18), andernfalls `***`.
  * Standardwerte decken gängige Schlüsselzuweisungen, CLI-Flags, JSON-Felder, Bearer-Header, PEM-Blöcke und verbreitete Token-Präfixe ab.

<div id="gateway-websocket-logs">
  ## Gateway WebSocket logs
</div>

Das Gateway gibt WebSocket-Protokoll-Logs in zwei Modi aus:

* **Normaler Modus (ohne `--verbose`)**: Es werden nur „interessante“ RPC-Ergebnisse ausgegeben:
  * Fehler (`ok=false`)
  * langsame Aufrufe (Standard-Schwellenwert: `>= 50ms`)
  * Parse-Fehler
* **Ausführlicher Modus (`--verbose`)**: gibt den gesamten WS-Anfrage-/Antwortverkehr aus.

<div id="ws-log-style">
  ### WS-Log-Stil
</div>

`openclaw gateway` unterstützt eine gateway-spezifische Einstellung des Stils:

* `--ws-log auto` (Standardwert): Normalmodus ist optimiert; ausführlicher Modus verwendet kompakte Ausgabe
* `--ws-log compact`: kompakte Ausgabe (gepaarte Request/Response) im ausführlichen Modus
* `--ws-log full`: vollständige Ausgabe pro Frame im ausführlichen Modus
* `--compact`: Alias für `--ws-log compact`

Beispiele:

```bash
# optimized (only errors/slow)
openclaw gateway

# show all WS traffic (paired)
openclaw gateway --verbose --ws-log compact

# zeige gesamten WS-Traffic (vollständige Metadaten)
openclaw gateway --verbose --ws-log full
```

<div id="console-formatting-subsystem-logging">
  ## Konsolenformatierung (Subsystem-Logging)
</div>

Der Konsolen-Formatter ist **TTY-aware** und gibt konsistente, mit Präfix versehene Zeilen aus.
Subsystem-Logger halten die Ausgabe gruppiert und gut erfassbar.

Verhalten:

* **Subsystem-Präfixe** in jeder Zeile (z. B. `[gateway]`, `[canvas]`, `[tailscale]`)
* **Subsystem-Farben** (stabil pro Subsystem) plus Level-Färbung
* **Farbausgabe, wenn die Ausgabe ein TTY ist oder die Umgebung wie ein „riches“ Terminal aussieht** (`TERM`/`COLORTERM`/`TERM_PROGRAM`), respektiert `NO_COLOR`
* **Verkürzte Subsystem-Präfixe**: entfernt führendes `gateway/` + `channels/`, behält die letzten 2 Segmente (z. B. `whatsapp/outbound`)
* **Sub-Logger pro Subsystem** (automatisches Präfix + strukturiertes Feld `{ subsystem }`)
* **`logRaw()`** für QR-/UX-Ausgabe (kein Präfix, keine Formatierung)
* **Konsolenstile** (z. B. `pretty | compact | json`)
* **Konsolen-Log-Level** getrennt vom Datei-Log-Level (Datei-Logs behalten die vollen Details, wenn `logging.level` auf `debug`/`trace` gesetzt ist)
* **WhatsApp-Nachrichteninhalte** werden auf `debug` geloggt (verwende `--verbose`, um sie zu sehen)

Dadurch bleiben bestehende Datei-Logs stabil, während interaktive Ausgaben gut scannbar bleiben.