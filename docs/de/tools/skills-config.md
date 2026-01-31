---
title: Fähigkeiten-Konfiguration
summary: "Schema und Beispiele für die Fähigkeitenkonfiguration"
read_when:
  - Hinzufügen oder Ändern der Fähigkeitenkonfiguration
  - Anpassen der gebündelten Allowlist oder des Installationsverhaltens
---

<div id="skills-config">
  # Fähigkeiten-Konfiguration
</div>

Die gesamte Fähigkeiten-Konfiguration liegt unter `skills` in `~/.openclaw/openclaw.json`.

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: [
        "~/Projects/agent-scripts/skills",
        "~/Projects/oss/some-skill-pack/skills"
      ],
      watch: true,
      watchDebounceMs: 250
    },
    install: {
      preferBrew: true,
      nodeManager: "npm" // npm | pnpm | yarn | bun (Gateway-Laufzeit verwendet weiterhin Node; bun wird nicht empfohlen)
    },
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE"
        }
      },
      peekaboo: { enabled: true },
      sag: { enabled: false }
    }
  }
}
```


<div id="fields">
  ## Felder
</div>

- `allowBundled`: optionale Allowlist nur für **gebündelte** Fähigkeiten. Wenn gesetzt, sind nur
  gebündelte Fähigkeiten in der Liste zulässig (verwaltete/Arbeitsbereich-Fähigkeiten bleiben unberührt).
- `load.extraDirs`: zusätzliche Verzeichnisse mit Fähigkeiten, die gescannt werden sollen (niedrigste Priorität).
- `load.watch`: Fähigkeitsordner beobachten und den Fähigkeiten-Snapshot aktualisieren (Standard: true).
- `load.watchDebounceMs`: Debounce für Ereignisse des Fähigkeiten-Watchers in Millisekunden (Standard: 250).
- `install.preferBrew`: bevorzugt brew-Installer, wenn verfügbar (Standard: true).
- `install.nodeManager`: bevorzugter Node-Installer (`npm` | `pnpm` | `yarn` | `bun`, Standard: npm).
  Dies betrifft nur **Fähigkeiten-Installationen**; die Gateway-Laufzeit sollte weiterhin Node sein
  (Bun wird für WhatsApp/Telegram nicht empfohlen).
- `entries.<skillKey>`: Überschreibungen pro Fähigkeit.

Felder pro Fähigkeit:

- `enabled`: setze `false`, um eine Fähigkeit zu deaktivieren, selbst wenn sie gebündelt/installiert ist.
- `env`: Umgebungsvariablen, die für den Agent-Lauf injiziert werden (nur, wenn sie noch nicht gesetzt sind).
- `apiKey`: optionale Komfortangabe für Fähigkeiten, die eine primäre Umgebungsvariable deklarieren.

<div id="notes">
  ## Hinweise
</div>

- Schlüssel unter `entries` werden standardmäßig dem Skill-Namen zugeordnet. Wenn ein Skill
  `metadata.openclaw.skillKey` definiert, wird stattdessen dieser Schlüssel verwendet.
- Änderungen an Fähigkeiten werden beim nächsten Turn des agents übernommen, sofern der Watcher aktiviert ist.

<div id="sandboxed-skills-env-vars">
  ### Sandbox-Fähigkeiten + Umgebungsvariablen
</div>

Wenn eine Sitzung in einer **sandbox** ausgeführt wird, laufen Skill-Prozesse in Docker. Die sandbox
übernimmt **nicht** die `process.env` des Hosts.

Verwende eine der folgenden Optionen:

- `agents.defaults.sandbox.docker.env` (oder pro Agent `agents.list[].sandbox.docker.env`)
- baue die Umgebungsvariablen in dein benutzerdefiniertes sandbox-Image ein

Globale `env`- und `skills.entries.<skill>.env/apiKey`-Werte gelten nur für Ausführungen auf dem **Host**.