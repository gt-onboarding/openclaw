---
title: Benutzerdefinierte Fähigkeiten erstellen
---

<div id="creating-custom-skills">
  # Eigene Fähigkeiten erstellen 🛠
</div>

OpenClaw ist so konzipiert, dass es sich leicht erweitern lässt. „Fähigkeiten“ sind der Hauptmechanismus, um deinem Assistenten neue Funktionen hinzuzufügen.

<div id="what-is-a-skill">
  ## Was ist ein Skill?
</div>

Ein Skill ist ein Verzeichnis, das eine `SKILL.md`-Datei (die dem LLM Anweisungen und Tool-Definitionen zur Verfügung stellt) und optional zusätzliche Skripte oder Ressourcen enthält.

<div id="step-by-step-your-first-skill">
  ## Schritt-für-Schritt: Dein erster Skill
</div>

<div id="1-create-the-directory">
  ### 1. Lege das Verzeichnis an
</div>

Fähigkeiten befinden sich in deinem Arbeitsbereich, in der Regel unter `~/.openclaw/workspace/skills/`. Erstelle ein neues Verzeichnis für deine Fähigkeit:

```bash
mkdir -p ~/.openclaw/workspace/skills/hello-world
```


<div id="2-define-the-skillmd">
  ### 2. Definiere die `SKILL.md`
</div>

Erstelle in diesem Verzeichnis eine Datei namens `SKILL.md`. Diese Datei verwendet YAML-Frontmatter für Metadaten und Markdown für Anweisungen.

```markdown
---
name: hello_world
description: A simple skill that says hello.
---

# Hello World Skill
When the user asks for a greeting, use the `echo` tool to say "Hello from your custom skill!".
```


<div id="3-add-tools-optional">
  ### 3. Tools hinzufügen (optional)
</div>

Du kannst benutzerdefinierte Tools im Frontmatter definieren oder den Agent anweisen, vorhandene System-Tools zu verwenden (wie `bash` oder `browser`).

<div id="4-refresh-openclaw">
  ### 4. OpenClaw aktualisieren
</div>

Bitte deinen Agenten, den Befehl „refresh skills“ auszuführen, oder starte das Gateway neu. OpenClaw wird das neue Verzeichnis erkennen und die `SKILL.md` indexieren.

<div id="best-practices">
  ## Best Practices
</div>

- **Fasse dich kurz**: Weise das Modell an, *was* es tun soll, nicht wie es eine KI sein soll.
- **Sicherheit zuerst**: Wenn dein Skill `bash` verwendet, stelle sicher, dass die Prompts kein Einschleusen beliebiger Befehle aus nicht vertrauenswürdigen Nutzereingaben erlauben.
- **Lokal testen**: Verwende `openclaw agent --message "use my new skill"` zum Testen.

<div id="shared-skills">
  ## Gemeinsame Fähigkeiten
</div>

Du kannst in [ClawHub](https://clawhub.com) nach Fähigkeiten stöbern und eigene beisteuern.