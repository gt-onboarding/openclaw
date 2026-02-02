---
title: Prosa
summary: "OpenProse: .prose-Workflows, Slash-Befehle und Zustandsverwaltung in OpenClaw"
read_when:
  - Du .prose-Workflows ausführen oder schreiben möchtest
  - Du das OpenProse-Plugin aktivieren möchtest
  - Du die Zustandsverwaltung verstehen musst
---

<div id="openprose">
  # OpenProse
</div>

OpenProse ist ein portables, Markdown-first-Workflow-Format zur Orchestrierung von KI-Sitzungen. In OpenClaw wird es als Plugin bereitgestellt, das ein OpenProse-Skill-Pack sowie den `/prose`-Slash-Befehl installiert. Programme werden in `.prose`‑Dateien definiert und können mehrere Sub-Agents mit expliziter Ablaufsteuerung erzeugen.

Offizielle Website: https://www.prose.md

<div id="what-it-can-do">
  ## Was es kann
</div>

* Multi-Agent-Recherche und -Synthese mit expliziter Parallelisierung.
* Wiederholbare, freigabesichere Workflows (Code-Review, Incident-Triage, Content-Pipelines).
* Wiederverwendbare `.prose`-Programme, die du über alle unterstützten Agent-Runtimes hinweg ausführen kannst.

<div id="install-enable">
  ## Installieren + aktivieren
</div>

Mitgelieferte Plugins sind standardmäßig deaktiviert. Aktiviere OpenProse:

```bash
openclaw plugins enable open-prose
```

Starte das Gateway nach dem Aktivieren des Plugins neu.

Dev/lokaler Checkout: `openclaw plugins install ./extensions/open-prose`

Verwandte Dokumentation: [Plugins](/de/plugin), [Plugin-Manifest](/de/plugins/manifest), [Fähigkeiten](/de/tools/skills).

<div id="slash-command">
  ## Slash-Befehl
</div>

OpenProse registriert `/prose` als von Benutzer:innen aufrufbares Skill-Kommando. Dieser Befehl leitet an die Anweisungen der OpenProse-VM weiter und verwendet dabei im Hintergrund OpenClaw-Tools.

Häufig verwendete Befehle:

```
/prose help
/prose run <file.prose>
/prose run <handle/slug>
/prose run <https://example.com/file.prose>
/prose compile <file.prose>
/prose examples
/prose update
```

<div id="example-a-simple-prose-file">
  ## Beispiel: eine einfache `.prose`-Datei
</div>

```prose
# Research + synthesis with two agents running in parallel.

input topic: "What should we research?"

agent researcher:
  model: sonnet
  prompt: "You research thoroughly and cite sources."

agent writer:
  model: opus
  prompt: "You write a concise summary."

parallel:
  findings = session: researcher
    prompt: "Research {topic}."
  draft = session: writer
    prompt: "Summarize {topic}."

session "Merge the findings + draft into a final answer."
context: { findings, draft }
```

<div id="file-locations">
  ## Dateispeicherorte
</div>

OpenProse speichert den Zustand unter `.prose/` in deinem Arbeitsbereich:

```
.prose/
├── .env
├── runs/
│   └── {YYYYMMDD}-{HHMMSS}-{random}/
│       ├── program.prose
│       ├── state.md
│       ├── bindings/
│       └── agents/
└── agents/
```

Benutzerspezifische persistente Agenten befinden sich unter:

```
~/.prose/agents/
```

<div id="state-modes">
  ## Zustandsmodi
</div>

OpenProse unterstützt mehrere State-Backends:

* **filesystem** (Standard): `.prose/runs/...`
* **in-context**: flüchtig, für kleine Programme
* **sqlite** (experimentell): erfordert das `sqlite3`-Binary
* **postgres** (experimentell): erfordert `psql` und einen Connection-String

Hinweise:

* sqlite/postgres müssen explizit aktiviert werden und sind experimentell.
* Postgres-Zugangsdaten können in Subagent-Logs landen; verwende eine dedizierte Datenbank mit minimalen Rechten.

<div id="remote-programs">
  ## Remoteprogramme
</div>

`/prose run <handle/slug>` wird zu `https://p.prose.md/<handle>/<slug>` aufgelöst.
Direkte URLs werden unverändert abgerufen. Dabei wird das Tool `web_fetch` verwendet (bzw. `exec` für POST-Anfragen).

<div id="openclaw-runtime-mapping">
  ## OpenClaw-Laufzeitzuordnung
</div>

OpenProse-Programme werden auf OpenClaw-Primitive gemappt:

| OpenProse-Konzept | OpenClaw-Tool |
| --- | --- |
| Sitzung starten / Task-Tool | `sessions_spawn` |
| Datei lesen/schreiben | `read` / `write` |
| Webabruf | `web_fetch` |

Wenn Ihre Tool-Allowlist diese Tools blockiert, schlagen OpenProse-Programme fehl. Siehe [Skills-Konfiguration](/de/tools/skills-config).

<div id="security-approvals">
  ## Sicherheit + Freigaben
</div>

Behandle `.prose`-Dateien wie Code. Überprüfe sie, bevor du sie ausführst. Verwende OpenClaw-Tool-Allowlists und Approval-Gates, um Seiteneffekte zu kontrollieren.

Für deterministische, freigabegesteuerte Workflows siehe [Lobster](/de/tools/lobster).