---
title: Kontext
summary: "Kontext: was das Modell sieht, wie er aufgebaut ist und wie du ihn inspizierst"
read_when:
  - Wenn du verstehen möchtest, was „Kontext“ in OpenClaw bedeutet
  - Wenn du debuggen möchtest, warum das Modell etwas „weiß“ (oder es vergessen hat)
  - Wenn du den Kontext-Overhead reduzieren möchtest (/context, /status, /compact)
---

<div id="context">
  # Kontext
</div>

„Kontext“ ist **alles, was OpenClaw für einen Lauf an das Modell sendet**. Er ist durch das **Kontextfenster** (Tokenlimit) des Modells begrenzt.

Mentales Modell für Einsteiger:

* **System-Prompt** (von OpenClaw erstellt): Regeln, Tools, Fähigkeitenliste, Zeit-/Laufzeitinformationen und injizierte Arbeitsbereichsdateien.
* **Konversationsverlauf**: deine Nachrichten + die Nachrichten des Assistenten für diese Sitzung.
* **Tool-Aufrufe/-Ergebnisse + Anhänge**: Befehlsausgaben, Datei-Lesevorgänge, Bilder/Audio etc.

Kontext ist *nicht dasselbe* wie „Speicher“: Speicher kann auf dem Datenträger abgelegt und später erneut geladen werden; Kontext ist das, was sich im aktuellen Fenster des Modells befindet.

<div id="quick-start-inspect-context">
  ## Schnellstart (Kontext inspizieren)
</div>

* `/status` → Schnellansicht „Wie voll ist mein Fenster?“ + Sitzungseinstellungen.
* `/context list` → was eingespeist wird + grobe Größen (pro Datei und insgesamt).
* `/context detail` → detailliertere Aufschlüsselung: Schema-Größen pro Datei und Tool, Eintragsgrößen pro Skill und Größe des Systemprompts.
* `/usage tokens` → fügt normalen Antworten eine Nutzungs-Fußzeile mit Verbrauch pro Antwort hinzu.
* `/compact` → fasst ältere Historie zu einem kompakten Eintrag zusammen, um Platz im Fenster freizugeben.

Siehe auch: [Slash-Commands](/de/tools/slash-commands), [Token-Nutzung &amp; Kosten](/de/token-use), [Verdichtung](/de/concepts/compaction).

<div id="example-output">
  ## Beispielausgabe
</div>

Die Werte variieren je nach Modell, Anbieter, Tool-Richtlinie und dem Inhalt deines Arbeitsbereichs.

<div id="context-list">
  ### `/context list`
</div>

```
🧠 Context breakdown
Workspace: <workspaceDir>
Bootstrap max/file: 20,000 chars
Sandbox: mode=non-main sandboxed=false
System prompt (run): 38,412 chars (~9,603 tok) (Project Context 23,901 chars (~5,976 tok))

Injected workspace files:
- AGENTS.md: OK | raw 1,742 chars (~436 tok) | injected 1,742 chars (~436 tok)
- SOUL.md: OK | raw 912 chars (~228 tok) | injected 912 chars (~228 tok)
- TOOLS.md: TRUNCATED | raw 54,210 chars (~13,553 tok) | injected 20,962 chars (~5,241 tok)
- IDENTITY.md: OK | raw 211 chars (~53 tok) | injected 211 chars (~53 tok)
- USER.md: OK | raw 388 chars (~97 tok) | injected 388 chars (~97 tok)
- HEARTBEAT.md: MISSING | raw 0 | injected 0
- BOOTSTRAP.md: OK | raw 0 chars (~0 tok) | injected 0 chars (~0 tok)

Skills list (system prompt text): 2,184 chars (~546 tok) (12 skills)
Tools: read, edit, write, exec, process, browser, message, sessions_send, …
Tool list (system prompt text): 1,032 chars (~258 tok)
Tool schemas (JSON): 31,988 chars (~7,997 tok) (counts toward context; not shown as text)
Tools: (same as above)

Session tokens (cached): 14,250 total / ctx=32,000
```

<div id="context-detail">
  ### `/context detail`
</div>

```
🧠 Context breakdown (detailed)
…
Top skills (prompt entry size):
- frontend-design: 412 chars (~103 tok)
- oracle: 401 chars (~101 tok)
… (+10 more skills)

Top tools (schema size):
- browser: 9,812 chars (~2,453 tok)
- exec: 6,240 chars (~1,560 tok)
… (+N more tools)
```

<div id="what-counts-toward-the-context-window">
  ## Was auf das Kontextfenster angerechnet wird
</div>

Alles, was das Modell erhält, zählt, einschließlich:

* Systemprompt (alle Abschnitte).
* Gesprächsverlauf.
* Toolaufrufe und -ergebnisse.
* Anhänge/Transkripte (Bilder/Audio/Dateien).
* Kompaktionszusammenfassungen und Pruning-Artefakte.
* Anbieter-„Wrapper“ oder versteckte Header (nicht sichtbar, werden trotzdem gezählt).

<div id="how-openclaw-builds-the-system-prompt">
  ## Wie OpenClaw den System-Prompt aufbaut
</div>

Der System-Prompt wird **vollständig von OpenClaw verwaltet** und bei jedem Durchlauf neu aufgebaut. Er enthält:

* Tool-Liste + kurze Beschreibungen.
* Fähigkeitenliste (nur Metadaten; siehe unten).
* Speicherort des Arbeitsbereichs.
* Zeit (UTC + umgerechnete Benutzerzeit, falls konfiguriert).
* Laufzeit-Metadaten (Host/OS/Modell/Denkmodus).
* Eingebundene Bootstrap-Dateien des Arbeitsbereichs im Abschnitt **Project Context**.

Vollständige Erläuterung: [System-Prompt](/de/concepts/system-prompt).

<div id="injected-workspace-files-project-context">
  ## Injizierte Arbeitsbereichsdateien (Projektkontext)
</div>

Standardmäßig injiziert OpenClaw eine feste Menge von Arbeitsbereichsdateien (falls vorhanden):

* `AGENTS.md`
* `SOUL.md`
* `TOOLS.md`
* `IDENTITY.md`
* `USER.md`
* `HEARTBEAT.md`
* `BOOTSTRAP.md` (nur beim ersten Durchlauf)

Große Dateien werden pro Datei mithilfe von `agents.defaults.bootstrapMaxChars` gekürzt (Standard: `20000` Zeichen). `/context` zeigt **Roh- vs. injizierte** Größen und ob eine Kürzung stattgefunden hat.

<div id="skills-whats-injected-vs-loaded-on-demand">
  ## Fähigkeiten: was injiziert wird vs. was bei Bedarf geladen wird
</div>

Der System-Prompt enthält eine kompakte **Fähigkeitenliste** (Name + Beschreibung + Speicherort). Diese Liste verursacht echten Overhead.

Fähigkeitsanweisungen werden *nicht* standardmäßig einbezogen. Das Modell soll die `SKILL.md` der Fähigkeit **nur bei Bedarf** `read`en.

<div id="tools-there-are-two-costs">
  ## Tools: Es gibt zwei Kostenarten
</div>

Tools beeinflussen den Kontext auf zwei Arten:

1. **Tool-Listen-Text** im System-Prompt (das, was du als „Tooling“ siehst).
2. **Tool-Schemas** (JSON). Diese werden an das Modell gesendet, damit es Tools aufrufen kann. Sie zählen zum Kontext, auch wenn du sie nicht als Klartext siehst.

`/context detail` listet die größten Tool-Schemas auf, damit du sehen kannst, was den Kontext dominiert.

<div id="commands-directives-and-inline-shortcuts">
  ## Befehle, Direktiven und „Inline-Shortcuts“
</div>

Slash-Befehle werden vom Gateway verarbeitet. Es gibt verschiedene Verhaltensweisen:

* **Eigenständige Befehle**: eine Nachricht, die nur aus `/...` besteht, wird als Befehl ausgeführt.
* **Direktiven**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/model`, `/queue` werden entfernt, bevor das Modell die Nachricht sieht.
  * Nachrichten, die nur aus Direktiven bestehen, speichern Sitzungseinstellungen.
  * Inline-Direktiven in einer normalen Nachricht wirken als Hinweise, die nur für diese Nachricht gelten.
* **Inline-Shortcuts** (nur Absender in der Allowlist): Bestimmte `/...`-Token innerhalb einer normalen Nachricht können sofort ausgeführt werden (Beispiel: „hey /status“) und werden entfernt, bevor das Modell den verbleibenden Text sieht.

Details: [Slash-Befehle](/de/tools/slash-commands).

<div id="sessions-compaction-and-pruning-what-persists">
  ## Sitzungen, Komprimierung und Bereinigung (was fortbesteht)
</div>

Was über Nachrichten hinweg fortbesteht, hängt vom Mechanismus ab:

* **Normale Historie** bleibt im Sitzungsprotokoll bestehen, bis sie gemäß Richtlinien komprimiert/bereinigt wird.
* **Komprimierung** speichert eine Zusammenfassung im Protokoll und lässt die jüngsten Nachrichten unverändert.
* **Bereinigung** entfernt alte Tool-Ergebnisse aus dem *In-Memory*-Prompt für einen Run, schreibt das Protokoll jedoch nicht um.

Dokumentation: [Sitzung](/de/concepts/session), [Komprimierung](/de/concepts/compaction), [Sitzungsbereinigung](/de/concepts/session-pruning).

<div id="what-context-actually-reports">
  ## Was `/context` tatsächlich meldet
</div>

`/context` bevorzugt, wenn verfügbar, den neuesten **run-built**-Bericht zum System-Prompt:

* `System prompt (run)` = erfasst aus dem letzten eingebetteten (mit Tools ausführbaren) Run und im Sitzungsspeicher persistiert.
* `System prompt (estimate)` = wird dynamisch berechnet, wenn kein Run-Bericht existiert (oder wenn über ein CLI-Backend ausgeführt wird, das diesen Bericht nicht erzeugt).

In beiden Fällen meldet es die Größen und wichtigsten Beitragenden; es gibt **nicht** den vollständigen System-Prompt oder Tool-Schemata aus.