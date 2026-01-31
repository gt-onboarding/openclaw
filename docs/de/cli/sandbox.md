---
title: Sandbox-CLI
summary: "Sandbox-Container verwalten und effektive Sandbox-Richtlinien prüfen"
read_when: "Du verwaltest Sandbox-Container oder untersuchst das Verhalten von Sandbox-/Tool-Richtlinien."
status: active
---

<div id="sandbox-cli">
  # Sandbox CLI
</div>

Docker-basierte Sandbox-Container für die isolierte Ausführung von Agents verwalten.

<div id="overview">
  ## Übersicht
</div>

OpenClaw kann Agenten aus Sicherheitsgründen in isolierten Docker-Containern ausführen. Die `sandbox`-Befehle helfen dir, diese Container zu verwalten, insbesondere nach Updates oder Konfigurationsänderungen.

<div id="commands">
  ## Befehle
</div>

<div id="openclaw-sandbox-explain">
  ### `openclaw sandbox explain`
</div>

Zeigt den **effektiven** Sandbox-Modus/-Scope/-Arbeitsbereichszugriff, die Sandbox-Toolrichtlinie und die erhöhten Gates (mit zugehörigen „Fix-it“-Konfigurationsschlüsselpfaden).

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

<div id="openclaw-sandbox-list">
  ### `openclaw sandbox list`
</div>

Listet alle Sandbox-Container mit ihrem Status und ihrer Konfiguration auf.

```bash
openclaw sandbox list
openclaw sandbox list --browser  # Nur Browser-Container auflisten
openclaw sandbox list --json     # JSON output
```

**Die Ausgabe umfasst:**

* Containername und Status (läuft/gestoppt)
* Docker-Image und Übereinstimmung mit der Konfiguration
* Alter (Zeit seit Erstellung)
* Leerlaufzeit (Zeit seit letzter Verwendung)
* Zugeordnete Sitzung/agent

<div id="openclaw-sandbox-recreate">
  ### `openclaw sandbox recreate`
</div>

Entfernt Sandbox-Container, um deren Neuerstellung mit aktualisierten Images/Konfiguration zu erzwingen.

```bash
openclaw sandbox recreate --all                # Alle Container neu erstellen
openclaw sandbox recreate --session main       # Bestimmte Sitzung
openclaw sandbox recreate --agent mybot        # Bestimmter Agent
openclaw sandbox recreate --browser            # Nur Browser-Container
openclaw sandbox recreate --all --force        # Bestätigung überspringen
```

**Optionen:**

* `--all`: Alle Sandbox-Container neu erstellen
* `--session <key>`: Container für bestimmte Sitzung neu erstellen
* `--agent <id>`: Container für einen bestimmten agent neu erstellen
* `--browser`: Nur Browser-Container neu erstellen
* `--force`: Bestätigungsabfrage überspringen

**Wichtig:** Container werden automatisch neu erstellt, wenn der agent das nächste Mal verwendet wird.

<div id="use-cases">
  ## Anwendungsfälle
</div>

<div id="after-updating-docker-images">
  ### Nach dem Aktualisieren der Docker-Images
</div>

```bash
# Neues Image abrufen
docker pull openclaw-sandbox:latest
docker tag openclaw-sandbox:latest openclaw-sandbox:bookworm-slim

# Konfiguration aktualisieren, um das neue Image zu verwenden
# Konfiguration bearbeiten: agents.defaults.sandbox.docker.image (oder agents.list[].sandbox.docker.image)

# Container neu erstellen
openclaw sandbox recreate --all
```

<div id="after-changing-sandbox-configuration">
  ### Nach dem Ändern der sandbox-Konfiguration
</div>

```bash
# Konfiguration bearbeiten: agents.defaults.sandbox.* (oder agents.list[].sandbox.*)

# Neu erstellen, um die neue Konfiguration anzuwenden
openclaw sandbox recreate --all
```

<div id="after-changing-setupcommand">
  ### Nach Änderung von setupCommand
</div>

```bash
openclaw sandbox recreate --all
# oder nur für einen einzelnen Agent:
openclaw sandbox recreate --agent family
```

<div id="for-a-specific-agent-only">
  ### Nur für einen bestimmten agent
</div>

```bash
# Aktualisiere nur die Container eines Agents
openclaw sandbox recreate --agent alfred
```

<div id="why-is-this-needed">
  ## Warum ist das nötig?
</div>

**Problem:** Wenn du sandbox-Docker-Images oder Konfigurationen aktualisierst:

* Bestehende Container laufen mit alten Einstellungen weiter
* Container werden erst nach 24 Stunden Inaktivität bereinigt
* Regelmäßig genutzte Agenten halten alte Container unbegrenzt am Laufen

**Lösung:** Verwende `openclaw sandbox recreate`, um alte Container zu erzwingen zu entfernen. Sie werden bei der nächsten Verwendung automatisch mit den aktuellen Einstellungen neu erstellt.

Tipp: Bevorzuge `openclaw sandbox recreate` gegenüber einem manuellen `docker rm`. Es nutzt das vom Gateway verwendete Containernamensschema und verhindert Inkonsistenzen, wenn scope-/session-Keys sich ändern.

<div id="configuration">
  ## Konfiguration
</div>

Sandbox-Einstellungen liegen in `~/.openclaw/openclaw.json` unter `agents.defaults.sandbox` (agent-spezifische Overrides stehen in `agents.list[].sandbox`):

```jsonc
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "all",                    // off, non-main, all
        "scope": "agent",                 // session, agent, shared
        "docker": {
          "image": "openclaw-sandbox:bookworm-slim",
          "containerPrefix": "openclaw-sbx-"
          // ... more Docker options
        },
        "prune": {
          "idleHours": 24,               // Automatisches Bereinigen nach 24 Stunden Inaktivität
          "maxAgeDays": 7                // Auto-prune after 7 days
        }
      }
    }
  }
}
```

<div id="see-also">
  ## Siehe auch
</div>

* [Sandbox-Dokumentation](/de/gateway/sandboxing)
* [Agent-Konfiguration](/de/concepts/agent-workspace)
* [Doctor-Befehl](/de/gateway/doctor) - Sandbox-Einrichtung überprüfen