---
title: Fähigkeiten
summary: "Fähigkeiten: verwaltet vs. Arbeitsbereich, Gating-Regeln und Config-/Env-Verknüpfung"
read_when:
  - Hinzufügen oder Ändern von Fähigkeiten
  - Ändern von Gating- oder Laderichtlinien für Fähigkeiten
---

<div id="skills-openclaw">
  # Fähigkeiten (OpenClaw)
</div>

OpenClaw verwendet **[AgentSkills](https://agentskills.io)-kompatible** Skill-Ordner, um dem agent beizubringen, wie Tools verwendet werden. Jede Fähigkeit ist ein Verzeichnis, das eine `SKILL.md` mit YAML-Frontmatter und Anweisungen enthält. OpenClaw lädt **mitgelieferte Fähigkeiten** sowie optionale lokale Überschreibungen und filtert sie beim Laden auf Basis von Umgebung, Konfiguration und dem Vorhandensein der Binärprogramme.

<div id="locations-and-precedence">
  ## Speicherorte und Priorität
</div>

Fähigkeiten werden aus **drei** Quellen geladen:

1. **Mitgelieferte Fähigkeiten**: werden mit der Installation ausgeliefert (npm-Paket oder OpenClaw.app)
2. **Verwaltete/lokale Fähigkeiten**: `~/.openclaw/skills`
3. **Arbeitsbereichs-Fähigkeiten**: `<workspace>/skills`

Wenn mehrere Fähigkeiten denselben Namen haben, gilt folgende Priorität:

`<workspace>/skills` (höchste) → `~/.openclaw/skills` → mitgelieferte Fähigkeiten (niedrigste)

Zusätzlich kannst du weitere Fähigkeiten-Ordner (mit niedrigster Priorität) über
`skills.load.extraDirs` in `~/.openclaw/openclaw.json` konfigurieren.

<div id="per-agent-vs-shared-skills">
  ## Agent-spezifische vs. gemeinsam genutzte Fähigkeiten
</div>

In **Multi-Agent**-Setups hat jeder agent seinen eigenen Arbeitsbereich. Das bedeutet:

* **Agent-spezifische Fähigkeiten** liegen in `<workspace>/skills` und gelten nur für diesen agent.
* **Gemeinsam genutzte Fähigkeiten** liegen in `~/.openclaw/skills` (verwaltet/lokal) und sind
  für **alle Agenten** auf demselben Rechner sichtbar.
* **Gemeinsam genutzte Verzeichnisse** können zusätzlich über `skills.load.extraDirs` (niedrigste
  Priorität) eingebunden werden, wenn du ein gemeinsames Fähigkeiten-Paket für mehrere Agenten nutzen möchtest.

Wenn derselbe Fähigkeitsname an mehr als einer Stelle existiert, gilt die übliche Priorität:
Der Arbeitsbereich gewinnt, dann verwaltet/lokal, dann gebündelt.

<div id="plugins-skills">
  ## Plugins + Fähigkeiten
</div>

Plugins können eigene Fähigkeiten mitliefern, indem sie `skills`-Verzeichnisse in
`openclaw.plugin.json` auflisten (Pfade relativ zum Plugin-Root). Plugin-Fähigkeiten werden geladen,
wenn das Plugin aktiviert ist, und unterliegen den normalen Vorrangregeln für Fähigkeiten.
Du kannst sie über `metadata.openclaw.requires.config` im Konfigurationseintrag des Plugins einschränken.
Siehe [Plugins](/de/plugin) für Discovery/Konfiguration und [Tools](/de/tools) für die Tool-Schnittstelle, die diese Fähigkeiten bereitstellen.

<div id="clawhub-install-sync">
  ## ClawHub (Installation + Sync)
</div>

ClawHub ist das öffentliche Fähigkeiten-Verzeichnis für OpenClaw. Besuche
https://clawhub.com. Verwende es, um Fähigkeiten zu entdecken, zu installieren, zu aktualisieren und zu sichern.
Vollständige Anleitung: [ClawHub](/de/tools/clawhub).

Typische Abläufe:

* Eine Fähigkeit in deinen Arbeitsbereich installieren:
  * `clawhub install <skill-slug>`
* Alle installierten Fähigkeiten aktualisieren:
  * `clawhub update --all`
* Synchronisieren (scannen + Updates veröffentlichen):
  * `clawhub sync --all`

Standardmäßig installiert `clawhub` in `./skills` unter deinem aktuellen
Arbeitsverzeichnis (oder greift auf den konfigurierten OpenClaw-Arbeitsbereich zurück). OpenClaw erfasst das bei der nächsten Sitzung als `<workspace>/skills`.

<div id="security-notes">
  ## Sicherheitshinweise
</div>

* Behandle Drittanbieter-Fähigkeiten als **vertrauenswürdigen Code**. Lies sie durch, bevor du sie aktivierst.
* Führe nicht vertrauenswürdige Eingaben und riskante Tools nach Möglichkeit in einer sandbox aus. Siehe [Sandboxing](/de/gateway/sandboxing).
* `skills.entries.*.env` und `skills.entries.*.apiKey` injizieren Secrets in den **Host**-Prozess
  für diesen Agent-Turn (nicht in die sandbox). Halte Secrets aus Prompts und Logs heraus.
* Für ein breiteres Bedrohungsmodell und Checklisten siehe [Security](/de/gateway/security).

<div id="format-agentskills-pi-compatible">
  ## Format (AgentSkills + Pi-kompatibel)
</div>

`SKILL.md` muss mindestens Folgendes enthalten:

```markdown
---
name: nano-banana-pro
description: Bilder mit Gemini 3 Pro Image generieren oder bearbeiten
---
```

Hinweise:

* Wir folgen der AgentSkills-Spezifikation für Layout und Zweck.
* Der vom eingebetteten Agent verwendete Parser unterstützt nur **einzeilige** Frontmatter-Schlüssel.
* `metadata` sollte ein **einzeiliges JSON-Objekt** sein.
* Verwende `{baseDir}` in Anweisungen, um auf den Pfad des Fähigkeiten-Ordners zu verweisen.
* Optionale Frontmatter-Schlüssel:
  * `homepage` — URL, die in der macOS Skills UI als „Website“ angezeigt wird (auch über `metadata.openclaw.homepage` unterstützt).
  * `user-invocable` — `true|false` (Standardwert: `true`). Wenn `true`, wird die Fähigkeit als Slash-Befehl für Benutzer bereitgestellt.
  * `disable-model-invocation` — `true|false` (Standardwert: `false`). Wenn `true`, wird die Fähigkeit aus dem Modell-Prompt ausgeschlossen (weiterhin über Benutzeraufruf verfügbar).
  * `command-dispatch` — `tool` (optional). Wenn auf `tool` gesetzt, umgeht der Slash-Befehl das Modell und leitet direkt an ein Tool weiter.
  * `command-tool` — Toolname, der aufgerufen wird, wenn `command-dispatch: tool` gesetzt ist.
  * `command-arg-mode` — `raw` (Standardwert). Für Tool-Dispatch wird der rohe Argument-String unverändert an das Tool weitergegeben (kein Parsing durch den Core).

    Das Tool wird mit folgenden Parametern aufgerufen:
    `{ command: "<raw args>", commandName: "<slash command>", skillName: "<skill name>" }`.

<div id="gating-load-time-filters">
  ## Gating (Filter beim Laden)
</div>

OpenClaw **filtert Fähigkeiten beim Laden** mithilfe von `metadata` (einzeiliges JSON):

```markdown
---
name: nano-banana-pro
description: Bilder mit Gemini 3 Pro Image generieren oder bearbeiten
metadata: {"openclaw":{"requires":{"bins":["uv"],"env":["GEMINI_API_KEY"],"config":["browser.enabled"]},"primaryEnv":"GEMINI_API_KEY"}}
---
```

Felder unter `metadata.openclaw`:

* `always: true` — Fähigkeit immer einbeziehen (andere Prüfbedingungen überspringen).
* `emoji` — optionales Emoji, das von der macOS Skills UI verwendet wird.
* `homepage` — optionale URL, die als „Website“ in der macOS Skills UI angezeigt wird.
* `os` — optionale Liste von Plattformen (`darwin`, `linux`, `win32`). Wenn gesetzt, ist die Fähigkeit nur auf diesen Betriebssystemen verfügbar.
* `requires.bins` — Liste; jedes Binary muss im `PATH` vorhanden sein.
* `requires.anyBins` — Liste; mindestens eines muss im `PATH` vorhanden sein.
* `requires.env` — Liste; Umgebungsvariable muss existieren **oder** in der Konfiguration bereitgestellt werden.
* `requires.config` — Liste von `openclaw.json`-Pfaden, die als truthy ausgewertet werden müssen.
* `primaryEnv` — Name der Umgebungsvariable, die `skills.entries.<name>.apiKey` zugeordnet ist.
* `install` — optionale Liste von Installer-Spezifikationen, die von der macOS Skills UI verwendet werden (brew/node/go/uv/download).

Hinweis zur Sandbox:

* `requires.bins` wird beim Laden der Fähigkeit auf dem **Host** geprüft.
* Wenn ein agent in einer Sandbox läuft, muss das Binary außerdem **im Container** vorhanden sein.
  Installiere es über `agents.defaults.sandbox.docker.setupCommand` (oder ein benutzerdefiniertes Image).
  `setupCommand` läuft einmal, nachdem der Container erstellt wurde.
  Paketinstallationen erfordern außerdem ausgehenden Netzwerkzugriff, ein beschreibbares Root-Dateisystem und einen Root-Benutzer in der Sandbox.
  Beispiel: Die Fähigkeit `summarize` (`skills/summarize/SKILL.md`) benötigt das `summarize` CLI
  im Sandbox-Container, um dort ausgeführt zu werden.

Installer-Beispiel:

```markdown
---
name: gemini
description: Use Gemini CLI for coding assistance and Google search lookups.
metadata: {"openclaw":{"emoji":"♊️","requires":{"bins":["gemini"]},"install":[{"id":"brew","kind":"brew","formula":"gemini-cli","bins":["gemini"],"label":"Gemini-CLI installieren (brew)"}]}}
---
```

Hinweise:

* Wenn mehrere Installer aufgeführt sind, wählt das Gateway **eine einzige** bevorzugte Option (brew, wenn verfügbar, andernfalls node).
* Wenn alle Installer `download` sind, listet OpenClaw jeden Eintrag auf, damit du die verfügbaren Artefakte sehen kannst.
* Installer-Spezifikationen können `os: ["darwin"|"linux"|"win32"]` enthalten, um Optionen nach Plattform zu filtern.
* Node-Installationen berücksichtigen `skills.install.nodeManager` in `openclaw.json` (Standard: npm; Optionen: npm/pnpm/yarn/bun).
  Dies betrifft nur **Installationen von Fähigkeiten**; die Gateway-Runtime sollte weiterhin Node sein
  (Bun wird für WhatsApp/Telegram nicht empfohlen).
* Go-Installationen: Wenn `go` fehlt und `brew` verfügbar ist, installiert das Gateway Go zuerst über Homebrew und setzt `GOBIN` nach Möglichkeit auf das `bin`-Verzeichnis von Homebrew.
* Download-Installationen: `url` (erforderlich), `archive` (`tar.gz` | `tar.bz2` | `zip`), `extract` (Standard: automatisch, wenn ein Archiv erkannt wird), `stripComponents`, `targetDir` (Standard: `~/.openclaw/tools/<skillKey>`).

Wenn kein `metadata.openclaw` vorhanden ist, ist die Fähigkeit grundsätzlich immer installierbar (sofern sie nicht in der Konfiguration deaktiviert oder durch `skills.allowBundled` für mitgelieferte Fähigkeiten blockiert ist).

<div id="config-overrides-openclawopenclawjson">
  ## Konfigurationsüberschreibungen (`~/.openclaw/openclaw.json`)
</div>

Mitgelieferte/verwaltete Fähigkeiten können ein- bzw. ausgeschaltet und mit Werten aus Umgebungsvariablen versorgt werden:

```json5
{
  skills: {
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE"
        },
        config: {
          endpoint: "https://example.invalid",
          model: "nano-pro"
        }
      },
      peekaboo: { enabled: true },
      sag: { enabled: false }
    }
  }
}
```

Hinweis: Wenn der Skill-Name Bindestriche enthält, setze den Schlüssel in Anführungszeichen (JSON5 erlaubt Anführungszeichen für Schlüssel).

Config-Schlüssel entsprechen standardmäßig dem **Skill-Namen**. Wenn eine Fähigkeit
`metadata.openclaw.skillKey` definiert, verwende diesen Schlüssel unter `skills.entries`.

Regeln:

* `enabled: false` deaktiviert die Fähigkeit, selbst wenn sie gebündelt/installiert ist.
* `env`: wird **nur dann** injiziert, wenn die Variable im Prozess noch nicht gesetzt ist.
* `apiKey`: Komfortkürzel für Fähigkeiten, die `metadata.openclaw.primaryEnv` deklarieren.
* `config`: optionaler Container für benutzerdefinierte Felder pro Fähigkeit; benutzerdefinierte Schlüssel müssen hier liegen.
* `allowBundled`: optionale Allowlist nur für **gebündelte** Fähigkeiten. Wenn gesetzt, sind nur
  gebündelte Fähigkeiten in der Liste zugelassen (verwaltete/Arbeitsbereich-Fähigkeiten bleiben unberührt).

<div id="environment-injection-per-agent-run">
  ## Umgebungsinjektion (pro Agent-Lauf)
</div>

Wenn ein Agent-Lauf startet, führt OpenClaw Folgendes aus:

1. Liest Metadaten der Fähigkeiten ein.
2. Wendet alle `skills.entries.<key>.env`- oder `skills.entries.<key>.apiKey`-Werte auf
   `process.env` an.
3. Baut den System-Prompt mit **verwendbaren** Fähigkeiten.
4. Stellt die ursprüngliche Umgebung wieder her, nachdem der Lauf beendet ist.

Dies ist **auf den Agent-Lauf beschränkt**, nicht auf die globale Shell-Umgebung.

<div id="session-snapshot-performance">
  ## Sitzungs-Snapshot (Leistung)
</div>

OpenClaw erstellt beim Start einer Sitzung einen Snapshot der zulässigen Fähigkeiten und verwendet diese Liste für alle nachfolgenden Turns in derselben Sitzung wieder. Änderungen an Fähigkeiten oder Konfiguration werden in der nächsten neuen Sitzung wirksam.

Fähigkeiten können auch während einer laufenden Sitzung aktualisiert werden, wenn der Skills-Watcher aktiviert ist oder wenn ein neuer zulässiger entfernter Knoten erscheint (siehe unten). Du kannst dir das als **Hot Reload** vorstellen: Die aktualisierte Liste wird beim nächsten Turn des Agents übernommen.

<div id="remote-macos-nodes-linux-gateway">
  ## Remote macOS-Knoten (Linux-Gateway)
</div>

Wenn das Gateway auf Linux läuft, aber ein **macOS-Knoten** verbunden ist, **bei dem `system.run` erlaubt ist** (Sicherheitsrichtlinie für Ausführungsfreigaben nicht auf `deny` gesetzt), kann OpenClaw macOS-spezifische Fähigkeiten als verfügbar behandeln, sofern die erforderlichen Binaries auf diesem Knoten vorhanden sind. Der Agent sollte diese Fähigkeiten über das `nodes`-Tool ausführen (typischerweise `nodes.run`).

Dies setzt voraus, dass der Knoten seine unterstützten Befehle meldet und dass eine Prüfung der Binaries über `system.run` erfolgt. Wenn der macOS-Knoten später offline geht, bleiben die Fähigkeiten sichtbar; Aufrufe können fehlschlagen, bis der Knoten die Verbindung wiederherstellt.

<div id="skills-watcher-auto-refresh">
  ## Fähigkeiten-Watcher (automatische Aktualisierung)
</div>

Standardmäßig überwacht OpenClaw die Skill-Verzeichnisse und aktualisiert den Fähigkeiten-Snapshot, sobald sich `SKILL.md`-Dateien ändern. Konfiguriere dies unter `skills.load`:

```json5
{
  skills: {
    load: {
      watch: true,
      watchDebounceMs: 250
    }
  }
}
```

<div id="token-impact-skills-list">
  ## Token-Auswirkungen (Fähigkeitenliste)
</div>

Wenn Fähigkeiten verfügbar sind, fügt OpenClaw eine kompakte XML-Liste der verfügbaren Fähigkeiten in den System-Prompt ein (über `formatSkillsForPrompt` in `pi-coding-agent`). Die Kosten sind deterministisch:

* **Basis-Overhead (nur wenn ≥1 Fähigkeit):** 195 Zeichen.
* **Pro Fähigkeit:** 97 Zeichen + die Länge der XML-escapten Werte für `<name>`, `<description>` und `<location>`.

Formel (Zeichen):

```
total = 195 + Σ (97 + len(name_escaped) + len(description_escaped) + len(location_escaped))
```

Hinweise:

* XML-Escaping wandelt `& < > " '` in Entitäten (`&amp;`, `&lt;` usw.) um und erhöht damit die Länge.
* Token-Zahlen variieren je nach Tokenizer des Modells. Eine grobe, OpenAI-ähnliche Schätzung ist ~4 Zeichen/Token, also **97 Zeichen ≈ 24 Token** pro Skill plus die tatsächlichen Längen deiner Felder.

<div id="managed-skills-lifecycle">
  ## Verwalteter Lebenszyklus von Fähigkeiten
</div>

OpenClaw wird mit einem Basissatz von Fähigkeiten **als gebündelte Fähigkeiten** im Rahmen der
Installation (npm-Paket oder OpenClaw.app) ausgeliefert. `~/.openclaw/skills` ist für lokale
Überschreibungen vorgesehen (zum Beispiel, um eine Fähigkeit zu pinnen oder zu patchen, ohne die gebündelte
Kopie zu ändern). Fähigkeiten im arbeitsbereich gehören den Benutzer:innen und haben in beiden Fällen Vorrang bei Namenskonflikten.

<div id="config-reference">
  ## Konfigurationsreferenz
</div>

Siehe die [Fähigkeiten-Konfiguration](/de/tools/skills-config) für das vollständige Konfigurationsschema.

<div id="looking-for-more-skills">
  ## Suchen Sie nach weiteren Fähigkeiten?
</div>

Besuchen Sie https://clawhub.com.

***