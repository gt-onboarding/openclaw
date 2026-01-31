---
title: Hooks
summary: "CLI-Referenz für `openclaw hooks` (Agent-Hooks)"
read_when:
  - Du Agent-Hooks verwalten möchtest
  - Du Hooks installieren oder aktualisieren möchtest
---

<div id="openclaw-hooks">
  # `openclaw hooks`
</div>

Agent-Hooks verwalten (ereignisgesteuerte Automatisierungen für Befehle wie `/new`, `/reset` und beim Start des Gateway).

Verwandte Themen:

* Hooks: [Hooks](/de/hooks)
* Plugin-Hooks: [Plugins](/de/plugin#plugin-hooks)

<div id="list-all-hooks">
  ## Alle Hooks anzeigen
</div>

```bash
openclaw hooks list
```

Listet alle erkannten Hooks aus dem Arbeitsbereich sowie aus verwalteten und gebündelten Verzeichnissen auf.

**Optionen:**

* `--eligible`: Nur geeignete Hooks anzeigen (Anforderungen erfüllt)
* `--json`: Ausgabe als JSON
* `-v, --verbose`: Detaillierte Informationen einschließlich fehlender Anforderungen anzeigen

**Beispielausgabe:**

```
Hooks (4/4 bereit)

Bereit:
  🚀 boot-md ✓ - BOOT.md beim Gateway-Start ausführen
  📝 command-logger ✓ - Alle Befehlsereignisse in einer zentralen Audit-Datei protokollieren
  💾 session-memory ✓ - Sitzungskontext im Speicher sichern, wenn der Befehl /new ausgegeben wird
  😈 soul-evil ✓ - Injizierte SOUL-Inhalte während eines Bereinigungsfensters oder zufällig austauschen
```

**Beispiel (ausführlich):**

```bash
openclaw hooks list --verbose
```

Zeigt fehlende Voraussetzungen für nicht zulässige Hooks an.

**Beispiel (JSON):**

```bash
openclaw hooks list --json
```

Gibt strukturiertes JSON für die programmgesteuerte Verwendung zurück.

<div id="get-hook-information">
  ## Hook-Informationen anzeigen
</div>

```bash
openclaw hooks info <name>
```

Zeigt detaillierte Informationen zu einem bestimmten Hook an.

**Argumente:**

* `<name>`: Hook-Name (z. B. `session-memory`)

**Optionen:**

* `--json`: Ausgabe als JSON

**Beispiel:**

```bash
openclaw hooks info session-memory
```

**Ausgabe:**

```
💾 session-memory ✓ Ready

Speichert den Sitzungskontext im Speicher, wenn der Befehl /new ausgegeben wird

Details:
  Source: openclaw-bundled
  Path: /path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  Handler: /path/to/openclaw/hooks/bundled/session-memory/handler.ts
  Homepage: https://docs.openclaw.ai/hooks#session-memory
  Events: command:new

Requirements:
  Config: ✓ workspace.dir
```

<div id="check-hooks-eligibility">
  ## Hook-Berechtigung prüfen
</div>

```bash
openclaw hooks check
```

Zeigt eine Zusammenfassung des Eignungsstatus der Hooks (wie viele bereit bzw. nicht bereit sind).

**Optionen:**

* `--json`: Ausgabe als JSON

**Beispielausgabe:**

```
Hooks Status

Total hooks: 4
Ready: 4
Not ready: 0
```

<div id="enable-a-hook">
  ## Hook aktivieren
</div>

```bash
openclaw hooks enable <name>
```

Aktiviere einen bestimmten Hook, indem du ihn zu deiner Konfiguration hinzufügst (`~/.openclaw/config.json`).

**Hinweis:** Hooks, die von Plugins verwaltet werden, werden in `openclaw hooks list` als `plugin:<id>` angezeigt und können hier nicht aktiviert/deaktiviert werden. Aktiviere/deaktiviere stattdessen das Plugin.

**Argumente:**

* `<name>`: Hook-Name (z. B. `session-memory`)

**Beispiel:**

```bash
openclaw hooks enable session-memory
```

**Ausgabe:**

```
✓ Hook aktiviert: 💾 session-memory
```

**Was es macht:**

* Prüft, ob der Hook existiert und aktiviert werden kann
* Aktualisiert `hooks.internal.entries.<name>.enabled = true` in deiner Konfiguration
* Speichert die Konfiguration auf die Festplatte

**Nach dem Aktivieren:**

* Starte den Gateway neu, damit die Hooks neu geladen werden (Neustart der Menüleisten-App auf macOS oder Neustart deines Gateway-Prozesses im Development-Modus).

<div id="disable-a-hook">
  ## Hook deaktivieren
</div>

```bash
openclaw hooks disable <name>
```

Deaktiviere einen bestimmten Hook, indem du deine Konfiguration anpasst.

**Argumente:**

* `<name>`: Hook-Name (z. B. `command-logger`)

**Beispiel:**

```bash
openclaw hooks disable command-logger
```

**Ausgabe:**

```
⏸ Disabled hook: 📝 command-logger
```

**Nach dem Deaktivieren:**

* Starte das Gateway neu, damit die Hooks neu geladen werden

<div id="install-hooks">
  ## Hooks installieren
</div>

```bash
openclaw hooks install <path-or-spec>
```

Installiere ein Hook-Pack aus einem lokalen Ordner/Archiv oder per npm.

**Funktionsweise:**

* Kopiert das Hook-Pack nach `~/.openclaw/hooks/<id>`
* Aktiviert die installierten Hooks in `hooks.internal.entries.*`
* Protokolliert die Installation unter `hooks.internal.installs`

**Optionen:**

* `-l, --link`: Verlinkt ein lokales Verzeichnis statt es zu kopieren (fügt es zu `hooks.internal.load.extraDirs` hinzu)

**Unterstützte Archive:** `.zip`, `.tgz`, `.tar.gz`, `.tar`

**Beispiele:**

```bash
# Local directory
openclaw hooks install ./my-hook-pack

# Local archive
openclaw hooks install ./my-hook-pack.zip

# NPM package
openclaw hooks install @openclaw/my-hook-pack

# Lokales Verzeichnis verknüpfen, ohne zu kopieren
openclaw hooks install -l ./my-hook-pack
```

<div id="update-hooks">
  ## Hooks aktualisieren
</div>

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

Installierte Hook-Pakete aktualisieren (nur npm-Installationen).

**Optionen:**

* `--all`: Alle erfassten Hook-Pakete aktualisieren
* `--dry-run`: Anzeigen, welche Änderungen vorgenommen würden, ohne etwas zu schreiben

<div id="bundled-hooks">
  ## Integrierte Hooks
</div>

<div id="session-memory">
  ### Sitzungsspeicher
</div>

Speichert den Sitzungskontext im Speicher, wenn du den Befehl `/new` ausführst.

**Aktivieren:**

```bash
openclaw hooks enable session-memory
```

**Ausgabe:** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md`

**Siehe:** [Dokumentation zu Sitzungsspeicher](/de/hooks#session-memory)

<div id="command-logger">
  ### command-logger
</div>

Protokolliert alle Befehlsereignisse in einer zentralen Audit-Datei.

**Aktivieren:**

```bash
openclaw hooks enable command-logger
```

**Ausgabe:** `~/.openclaw/logs/commands.log`

**Logs anzeigen:**

```bash
# Neueste Befehle
tail -n 20 ~/.openclaw/logs/commands.log

# Formatierte Ausgabe
cat ~/.openclaw/logs/commands.log | jq .

# Nach Aktion filtern
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**Siehe auch:** [command-logger-Dokumentation](/de/hooks#command-logger)

<div id="soul-evil">
  ### soul-evil
</div>

Ersetzt den injizierten `SOUL.md`-Inhalt während eines Bereinigungszeitraums oder mit zufälliger Wahrscheinlichkeit durch `SOUL_EVIL.md`.

**Aktivieren:**

```bash
openclaw hooks enable soul-evil
```

**Siehe auch:** [SOUL Evil Hook](/de/hooks/soul-evil)

<div id="boot-md">
  ### boot-md
</div>

Führt `BOOT.md` aus, wenn das Gateway startet (nachdem die Kanäle gestartet wurden).

**Events**: `gateway:startup`

**Enable**:

```bash
openclaw hooks enable boot-md
```

**Siehe:** [boot-md-Dokumentation](/de/hooks#boot-md)
