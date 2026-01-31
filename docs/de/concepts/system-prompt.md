---
title: System-Prompt
summary: "Was der OpenClaw-System-Prompt enthält und wie er zusammengesetzt wird"
read_when:
  - System-Prompt-Text, Tool-Liste oder Zeit-/Herzschlag-Abschnitte bearbeiten
  - Arbeitsbereichs-Bootstrap oder Verhalten bei der Fähigkeiten-Einbindung ändern
---

<div id="system-prompt">
  # System-Prompt
</div>

OpenClaw erstellt für jeden Agent-Run einen benutzerdefinierten System-Prompt. Der Prompt ist **OpenClaw-eigen** und verwendet nicht den Standard-Prompt von p-coding-agent.

Der Prompt wird von OpenClaw zusammengesetzt und in jeden Agent-Run eingespeist.

<div id="structure">
  ## Struktur
</div>

Der Prompt ist bewusst kompakt gehalten und verwendet feste Abschnitte:

* **Tooling**: aktuelle Werkzeugliste + kurze Beschreibungen.
* **Fähigkeiten** (falls verfügbar): erklärt dem Modell, wie Anweisungen zu Fähigkeiten bei Bedarf geladen werden.
* **OpenClaw-Selbstupdate**: wie `config.apply` und `update.run` ausgeführt werden.
* **Arbeitsbereich**: Arbeitsverzeichnis (`agents.defaults.workspace`).
* **Dokumentation**: lokaler Pfad zur OpenClaw-Dokumentation (Repo oder npm-Paket) und wann sie gelesen werden soll.
* **Arbeitsbereichsdateien (injiziert)**: zeigt an, dass Bootstrap-Dateien unten eingebunden sind.
* **Sandbox** (falls aktiviert): zeigt an, dass in einer sandbox-Laufzeit ausgeführt wird, mit Sandbox-Pfaden und ob privilegierte Ausführung (elevated exec) verfügbar ist.
* **Aktuelles Datum &amp; Uhrzeit**: benutzerlokale Zeit, Zeitzone und Zeitformat.
* **Reply-Tags**: optionale Reply-Tag-Syntax für unterstützte Anbieter.
* **Heartbeats**: Herzschlag-Prompt und Bestätigungsverhalten.
* **Laufzeit**: Host, OS, Knoten, Modell, Repo-Root (falls erkannt), Denkniveau (eine Zeile).
* **Reasoning**: aktuelle Sichtbarkeitsebene + Hinweis auf den /reasoning-Umschalter.

<div id="prompt-modes">
  ## Prompt-Modi
</div>

OpenClaw kann kleinere System-Prompts für Sub-Agenten rendern. Zur Laufzeit wird
für jeden Durchlauf ein `promptMode` gesetzt (keine benutzerkonfigurierbare Einstellung):

* `full` (Standard): enthält alle oben aufgeführten Abschnitte.
* `minimal`: wird für Sub-Agenten verwendet; lässt **Fähigkeiten**, **Memory Recall**, **OpenClaw
  Self-Update**, **Model Aliases**, **User Identity**, **Reply Tags**,
  **Messaging**, **Silent Replies** und **Herzschläge** weg. Tooling, Arbeitsbereich,
  sandbox, Current Date &amp; Time (falls bekannt), Runtime und injizierter Kontext bleiben
  verfügbar.
* `none`: gibt nur die grundlegende Identitätszeile zurück.

Bei `promptMode=minimal` werden zusätzlich injizierte Prompts als **Subagent
Context** statt als **Group Chat Context** gekennzeichnet.

<div id="workspace-bootstrap-injection">
  ## Arbeitsbereich-Bootstrap-Injektion
</div>

Bootstrap-Dateien werden gekürzt und unter **Projektkontext** angehängt, damit das Modell Identitäts- und Profilkontext sieht, ohne explizite `read`-Aufrufe zu benötigen:

* `AGENTS.md`
* `SOUL.md`
* `TOOLS.md`
* `IDENTITY.md`
* `USER.md`
* `HEARTBEAT.md`
* `BOOTSTRAP.md` (nur in brandneuen Arbeitsbereichen)

Große Dateien werden abgeschnitten und mit einem Marker versehen. Die maximale Größe pro Datei wird durch
`agents.defaults.bootstrapMaxChars` gesteuert (Standard: 20000). Fehlende Dateien führen zu einem
kurzen Marker für fehlende Dateien.

Interne Hooks können diesen Schritt über `agent:bootstrap` abfangen, um die injizierten Bootstrap-Dateien zu verändern oder zu ersetzen (zum Beispiel `SOUL.md` durch eine alternative Persona austauschen).

Um zu untersuchen, wie viel jede injizierte Datei beiträgt (Rohinhalt vs. injizierte Variante, Kürzung sowie Tool-Schema-Overhead), verwende `/context list` oder `/context detail`. Siehe [Kontext](/de/concepts/context).

<div id="time-handling">
  ## Zeitverarbeitung
</div>

Der Systemprompt enthält einen eigenen Abschnitt **Current Date &amp; Time**, wenn
die Zeitzone des Benutzers bekannt ist. Um den Prompt-Cache stabil zu halten, enthält er jetzt nur noch
die **time zone** (keine dynamische Uhrzeit oder Zeitformat).

Verwende `session_status`, wenn der Agent die aktuelle Zeit benötigt; die Statuskarte
enthält eine Zeile mit einem Zeitstempel.

Konfiguriere dies mit:

* `agents.defaults.userTimezone`
* `agents.defaults.timeFormat` (`auto` | `12` | `24`)

Siehe [Date &amp; Time](/de/date-time) für vollständige Details zum Verhalten.

<div id="skills">
  ## Fähigkeiten
</div>

Wenn verwendbare Fähigkeiten vorhanden sind, fügt OpenClaw eine kompakte
**Liste verfügbarer Fähigkeiten** (`formatSkillsForPrompt`) ein, die den
**Dateipfad** für jede Fähigkeit enthält. Der Prompt weist das Modell an, `read`
zu verwenden, um die SKILL.md am angegebenen Speicherort (arbeitsbereich,
verwaltet oder gebündelt) zu laden. Wenn keine Fähigkeiten verwendbar sind,
wird der Abschnitt „Fähigkeiten“ weggelassen.

```
<available_skills>
  <skill>
    <name>...</name>
    <description>...</description>
    <location>...</location>
  </skill>
</available_skills>
```

Dadurch bleibt der Basis-Prompt klein, ermöglicht aber weiterhin den gezielten Einsatz von Skills.

<div id="documentation">
  ## Dokumentation
</div>

Wenn verfügbar, enthält der System‑Prompt einen Abschnitt **Dokumentation**, der auf das
lokale OpenClaw‑Dokumentationsverzeichnis verweist (entweder `docs/` im Repo‑arbeitsbereich oder die mitgelieferte
npm‑Paketdokumentation) und außerdem den öffentlichen Mirror, das Quell‑Repository, den Community‑Discord‑Server und
ClawHub (https://clawhub.com) zum Auffinden von Fähigkeiten aufführt. Der Prompt weist das Modell an, zuerst die lokale
Dokumentation zu OpenClaw‑Verhalten, Befehlen, Konfiguration oder Architektur zu konsultieren und, wenn möglich,
`openclaw status` selbst auszuführen (und den Benutzer nur zu fragen, wenn kein Zugriff besteht).