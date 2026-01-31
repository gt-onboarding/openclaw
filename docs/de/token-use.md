---
title: Token-Nutzung
summary: "Wie OpenClaw Prompt-Kontext aufbaut und Token-Nutzung sowie Kosten erfasst und ausgibt"
read_when:
  - Erklären von Token-Nutzung, Kosten oder Kontextfenstern
  - Debuggen von Kontextwachstum oder -verdichtung
---

<div id="token-use-costs">
  # Tokennutzung &amp; Kosten
</div>

OpenClaw erfasst **Tokens**, nicht Zeichen. Tokens sind modellspezifisch, aber die meisten
OpenAI-ähnlichen Modelle kommen im Durchschnitt auf etwa 4 Zeichen pro Token für englischen Text.

<div id="how-the-system-prompt-is-built">
  ## Wie der System-Prompt aufgebaut ist
</div>

OpenClaw stellt bei jeder Ausführung seinen eigenen System-Prompt zusammen. Er enthält:

* Tool-Liste + kurze Beschreibungen
* Fähigkeitenliste (nur Metadaten; Anweisungen werden bei Bedarf mit `read` geladen)
* Anweisungen zur Selbstaktualisierung
* Arbeitsbereich + Bootstrap-Dateien (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`, falls neu). Große Dateien werden durch `agents.defaults.bootstrapMaxChars` gekürzt (Standard: 20000).
* Zeit (UTC + Zeitzone des Benutzers)
* Antwort-Tags + Herzschlagverhalten
* Laufzeit-Metadaten (Host/OS/Modell/Thinking)

Die vollständige Aufschlüsselung findest du unter [System Prompt](/de/concepts/system-prompt).

<div id="what-counts-in-the-context-window">
  ## Was im Kontextfenster zählt
</div>

Alles, was das Modell erhält, zählt gegen das Kontextlimit:

* System-Prompt (alle oben aufgeführten Abschnitte)
* Gesprächsverlauf (Benutzer- und Assistenten-Nachrichten)
* Tool-Aufrufe und Tool-Ergebnisse
* Anhänge/Transkripte (Bilder, Audio, Dateien)
* Zusammenfassungen aus der Kompaktierung und Pruning-Artefakte
* Anbieter-Wrapper oder Safety-Header (nicht sichtbar, werden aber trotzdem gezählt)

Für eine praktische Aufschlüsselung (pro injizierter Datei, Tools, Fähigkeiten und System-Prompt-Größe) verwende `/context list` oder `/context detail`. Siehe [Kontext](/de/concepts/context).

<div id="how-to-see-current-token-usage">
  ## So überprüfst du die aktuelle Token-Nutzung
</div>

Verwende diese Befehle im Chat:

* `/status` → **Emoji‑reiche Statuskarte** mit dem Sitzungsmodell, der Kontextnutzung,
  den letzten Antwort‑Input/Output‑Tokens und **geschätzten Kosten** (nur API‑Key).
* `/usage off|tokens|full` → fügt jeder Antwort eine **Fußzeile mit Nutzungsdaten** hinzu.
  * Bleibt pro Sitzung erhalten (gespeichert als `responseUsage`).
  * OAuth-Authentifizierung **blendet Kosten aus** (nur Tokens).
* `/usage cost` → zeigt eine lokale Kostenzusammenfassung auf Basis der OpenClaw-Sitzungslogs.

Andere Oberflächen:

* **TUI/Web TUI:** `/status` + `/usage` werden unterstützt.
* **CLI:** `openclaw status --usage` und `openclaw channels list` zeigen
  Anbieter‑Kontingentzeiträume (keine Kosten pro Antwort).

<div id="cost-estimation-when-shown">
  ## Kostenschätzung (wenn angezeigt)
</div>

Die Kosten werden anhand deiner Modellpreis-Konfiguration geschätzt:

```
models.providers.<provider>.models[].cost
```

Dies sind **USD pro 1 Mio. Token** für `input`, `output`, `cacheRead` und
`cacheWrite`. Wenn keine Preise angegeben sind, zeigt OpenClaw nur die Tokenanzahl an. Für OAuth-Token werden keine Dollar-Kosten angezeigt.

<div id="cache-ttl-and-pruning-impact">
  ## Cache-TTL und Auswirkungen des Prunings
</div>

Provider-Prompt-Caching greift nur innerhalb des Cache-TTL-Fensters. OpenClaw
kann optional ein **Cache-TTL-Pruning** durchführen: Dabei wird die Sitzung
bereinigt, sobald die Cache-TTL abgelaufen ist, und anschließend das
Cache-Fenster zurückgesetzt, sodass nachfolgende Anfragen den frisch
zwischengespeicherten Kontext wiederverwenden können, anstatt die komplette
Historie erneut zu cachen. Das hält die Cache-Schreibkosten niedriger, wenn eine
Sitzung nach Ablauf der TTL inaktiv wird.

Konfiguriere dies in der [Gateway configuration](/de/gateway/configuration) und
Details zum Verhalten findest du unter [Session pruning](/de/concepts/session-pruning).

Der Heartbeat kann den Cache über Leerlaufphasen hinweg **warm** halten. Wenn
deine Modell-Cache-TTL `1h` beträgt, kann es helfen, das Heartbeat-Intervall
knapp darunter zu setzen (z. B. `55m`), um ein erneutes Caching des gesamten
Prompts zu vermeiden und so die Cache-Schreibkosten zu reduzieren.

Bei der Anthropic-API-Preisgestaltung sind Cache-Lesezugriffe deutlich
günstiger als Input-Tokens, während Cache-Schreibvorgänge mit einem höheren
Multiplikator abgerechnet werden. Die aktuellen Preise und TTL-Multiplikatoren
findest du in Anthropics Dokumentation zur Prompt-Caching-Preisgestaltung:
https://docs.anthropic.com/docs/build-with-claude/prompt-caching

<div id="example-keep-1h-cache-warm-with-heartbeat">
  ### Beispiel: 1-Stunden-Cache mit Herzschlag warmhalten
</div>

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-5"
    models:
      "anthropic/claude-opus-4-5":
        params:
          cacheControlTtl: "1h"
    heartbeat:
      every: "55m"
```

<div id="tips-for-reducing-token-pressure">
  ## Tipps zur Reduzierung der Token-Belastung
</div>

* Verwende `/compact`, um lange Sitzungen zusammenzufassen.
* Kürze große Tool-Ausgaben in deinen Workflows.
* Halte Fähigkeitsbeschreibungen kurz (die Fähigkeitsliste wird in den Prompt eingebettet).
* Bevorzuge kleinere Modelle für ausführliche, explorative Arbeit.

Siehe [Fähigkeiten](/de/tools/skills) für die genaue Formel zum Overhead der Fähigkeitsliste.