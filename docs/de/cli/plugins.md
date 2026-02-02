---
title: Plugins
summary: "CLI-Referenz für `openclaw plugins` (auflisten, installieren, aktivieren/deaktivieren, Diagnose)"
read_when:
  - Du möchtest Plugins installieren oder verwalten, die im Gateway-Prozess laufen
  - Du möchtest Fehler beim Laden von Plugins debuggen
---

<div id="openclaw-plugins">
  # `openclaw plugins`
</div>

Verwaltet Gateway-Plugins/-Erweiterungen (im Prozess geladen).

Verwandte Themen:

* Plugin-System: [Plugins](/de/plugin)
* Plugin-Manifest + Schema: [Plugin-Manifest](/de/plugins/manifest)
* Sicherheitshärtung: [Sicherheit](/de/gateway/security)

<div id="commands">
  ## Befehle
</div>

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

Mit OpenClaw gebündelte Plugins werden zwar mitgeliefert, sind aber zunächst deaktiviert. Verwende `plugins enable`, um sie zu aktivieren.

Alle Plugins müssen eine Datei `openclaw.plugin.json` mit einem eingebetteten JSON-Schema
(`configSchema`, auch wenn es leer ist) enthalten. Fehlende oder ungültige Manifeste oder Schemata verhindern,
dass das Plugin geladen wird, und führen dazu, dass die Konfigurationsvalidierung fehlschlägt.

<div id="install">
  ### Installieren
</div>

```bash
openclaw plugins install <path-or-spec>
```

Sicherheitshinweis: Behandle Plugin-Installationen wie das Ausführen von Code. Verwende nach Möglichkeit fest gepinnte Versionen.

Unterstützte Archive: `.zip`, `.tgz`, `.tar.gz`, `.tar`.

Verwende `--link`, um das Kopieren eines lokalen Verzeichnisses zu vermeiden (wird zu `plugins.load.paths` hinzugefügt):

```bash
openclaw plugins install -l ./my-plugin
```

<div id="update">
  ### Update
</div>

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

Updates gelten nur für Plugins, die über npm installiert wurden (erfasst in `plugins.installs`).
