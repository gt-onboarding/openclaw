---
title: Clawhub
summary: "ClawHub-Handbuch: öffentliches Fähigkeiten-Registry + CLI-Workflows"
read_when:
  - ClawHub neuen Nutzern vorstellen
  - Fähigkeiten installieren, suchen oder veröffentlichen
  - ClawHub-CLI-Flags und Synchronisationsverhalten erklären
---

<div id="clawhub">
  # ClawHub
</div>

ClawHub ist das **öffentliche Verzeichnis für Fähigkeiten in OpenClaw**. Es ist ein kostenloser Dienst: Alle Fähigkeiten sind öffentlich, offen und für alle zum Teilen und Wiederverwenden sichtbar. Eine Fähigkeit ist einfach ein Ordner mit einer `SKILL.md`-Datei (plus unterstützenden Textdateien). Du kannst Fähigkeiten in der Web-App durchsuchen oder die CLI verwenden, um nach Fähigkeiten zu suchen, sie zu installieren, zu aktualisieren und zu veröffentlichen.

Website: [clawhub.com](https://clawhub.com)

<div id="who-this-is-for-beginner-friendly">
  ## Für wen ist das (einsteigerfreundlich)
</div>

Wenn du deinem OpenClaw-agent neue Fähigkeiten hinzufügen möchtest, ist ClawHub der einfachste Weg, um Fähigkeiten zu finden und zu installieren. Du musst nicht wissen, wie das Backend funktioniert. Du kannst:

* Nach Fähigkeiten mit natürlicher Sprache suchen.
* Eine Fähigkeit in deinen Arbeitsbereich installieren.
* Fähigkeiten später mit einem einzigen Befehl aktualisieren.
* Deine eigenen Fähigkeiten sichern, indem du sie veröffentlichst.

<div id="quick-start-non-technical">
  ## Schnellstart (nicht technisch)
</div>

1. Installiere die CLI (siehe nächsten Abschnitt).
2. Suche nach etwas, das du brauchst:
   * `clawhub search "calendar"`
3. Installiere einen Skill:
   * `clawhub install <skill-slug>`
4. Starte eine neue OpenClaw-Sitzung, damit der neue Skill aktiv wird.

<div id="install-the-cli">
  ## CLI installieren
</div>

Wähle eine Option:

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```

<div id="how-it-fits-into-openclaw">
  ## Wie es in OpenClaw integriert ist
</div>

Standardmäßig installiert die CLI Fähigkeiten in `./skills` unter deinem aktuellen Arbeitsverzeichnis. Wenn ein OpenClaw-Arbeitsbereich konfiguriert ist, greift `clawhub` auf diesen Arbeitsbereich zurück, sofern du `--workdir` (oder `CLAWHUB_WORKDIR`) nicht überschreibst. OpenClaw lädt Arbeitsbereichs-Fähigkeiten aus `<workspace>/skills` und berücksichtigt sie in der **nächsten** Sitzung. Wenn du bereits `~/.openclaw/skills` oder gebündelte Fähigkeiten verwendest, haben die Arbeitsbereichs-Fähigkeiten Vorrang.

Weitere Details dazu, wie Fähigkeiten geladen, geteilt und zugangsbeschränkt werden, findest du unter
[Fähigkeiten](/de/tools/skills).

<div id="what-the-service-provides-features">
  ## Was der Dienst bereitstellt (Funktionen)
</div>

* **Öffentliches Durchsuchen** von Fähigkeiten und deren `SKILL.md`-Inhalten.
* **Suche**, unterstützt durch Embeddings (Vektorsuche), nicht nur Schlagwörter.
* **Versionierung** mit SemVer, Changelogs und Tags (einschließlich `latest`).
* **Downloads** als ZIP-Archiv pro Version.
* **Sterne und Kommentare** für Community-Feedback.
* **Moderations-Hooks** für Freigaben und Audits.
* **CLI-freundliche API** für Automatisierung und Skripting.

<div id="cli-commands-and-parameters">
  ## CLI-Befehle und Parameter
</div>

Globale Optionen (gelten für alle Befehle):

* `--workdir <dir>`: Arbeitsverzeichnis (Standard: aktuelles Verzeichnis; fällt zurück auf den OpenClaw-Arbeitsbereich).
* `--dir <dir>`: Skills-Verzeichnis, relativ zu workdir (Standard: `skills`).
* `--site <url>`: Basis-URL der Website (Browser-Login).
* `--registry <url>`: Basis-URL der Registry-API.
* `--no-input`: Eingabeaufforderungen deaktivieren (nicht interaktiv).
* `-V, --cli-version`: CLI-Version ausgeben.

Auth:

* `clawhub login` (Browser-Flow) oder `clawhub login --token <token>`
* `clawhub logout`
* `clawhub whoami`

Optionen:

* `--token <token>`: Einen API-Token einfügen.
* `--label <label>`: Label, das für Browser-Login-Tokens gespeichert wird (Standard: `CLI token`).
* `--no-browser`: Keinen Browser öffnen (erfordert `--token`).

Suche:

* `clawhub search "query"`
* `--limit <n>`: Maximale Anzahl an Ergebnissen.

Installation:

* `clawhub install <slug>`
* `--version <version>`: Eine bestimmte Version installieren.
* `--force`: Überschreiben, wenn der Ordner bereits existiert.

Update:

* `clawhub update <slug>`
* `clawhub update --all`
* `--version <version>`: Auf eine bestimmte Version aktualisieren (nur einzelner Slug).
* `--force`: Überschreiben, wenn lokale Dateien keiner veröffentlichten Version entsprechen.

Auflisten:

* `clawhub list` (liest `.clawhub/lock.json`)

Veröffentlichen:

* `clawhub publish <path>`
* `--slug <slug>`: Skill-Slug.
* `--name <name>`: Anzeigename.
* `--version <version>`: Semver-Version.
* `--changelog <text>`: Changelog-Text (kann leer sein).
* `--tags <tags>`: Kommagetrennte Tags (Standard: `latest`).

Löschen/Wiederherstellen (nur Owner/Admin):

* `clawhub delete <slug> --yes`
* `clawhub undelete <slug> --yes`

Sync (lokale Fähigkeiten scannen + neue/aktualisierte veröffentlichen):

* `clawhub sync`
* `--root <dir...>`: Zusätzliche Wurzelverzeichnisse für den Scan.
* `--all`: Alles ohne Eingabeaufforderungen hochladen.
* `--dry-run`: Anzeigen, was hochgeladen würde.
* `--bump <type>`: `patch|minor|major` für Updates (Standard: `patch`).
* `--changelog <text>`: Changelog für nicht-interaktive Updates.
* `--tags <tags>`: Kommagetrennte Tags (Standard: `latest`).
* `--concurrency <n>`: Registry-Checks (Standard: 4).

<div id="common-workflows-for-agents">
  ## Typische Workflows für Agenten
</div>

<div id="search-for-skills">
  ### Fähigkeiten suchen
</div>

```bash
clawhub search "postgres backups"
```

<div id="download-new-skills">
  ### Neue Fähigkeiten herunterladen
</div>

```bash
clawhub install my-skill-pack
```

<div id="update-installed-skills">
  ### Installierte Fähigkeiten aktualisieren
</div>

```bash
clawhub update --all
```

<div id="back-up-your-skills-publish-or-sync">
  ### Sichere deine Fähigkeiten (veröffentlichen oder synchronisieren)
</div>

Für einen einzelnen Skill-Ordner:

```bash
clawhub publish ./my-skill --slug my-skill --name "My Skill" --version 1.0.0 --tags latest
```

Um mehrere Fähigkeiten gleichzeitig zu scannen und zu sichern:

```bash
clawhub sync --all
```

<div id="advanced-details-technical">
  ## Erweiterte technische Details
</div>

<div id="versioning-and-tags">
  ### Versionierung und Tags
</div>

* Jede Veröffentlichung erstellt eine neue **semver**-`SkillVersion`.
* Tags (wie `latest`) verweisen auf eine bestimmte Version; durch das Verschieben von Tags kannst du Rollbacks durchführen.
* Changelogs sind versionsbezogen und können beim Synchronisieren oder Veröffentlichen von Updates leer sein.

<div id="local-changes-vs-registry-versions">
  ### Lokale Änderungen vs. Registry-Versionen
</div>

Updates vergleichen die lokalen Skill-Inhalte mit den Registry-Versionen anhand eines Content-Hashes. Wenn lokale Dateien keiner veröffentlichten Version entsprechen, fordert die CLI eine Bestätigung an, bevor sie überschrieben werden (oder erfordert `--force` bei nicht-interaktiven Ausführungen).

<div id="sync-scanning-and-fallback-roots">
  ### Sync-Scan und Fallback-Stammverzeichnisse
</div>

`clawhub sync` scannt zunächst das aktuelle Arbeitsverzeichnis. Wenn keine Fähigkeiten gefunden werden, fällt der Befehl auf bekannte Legacy-Pfade zurück (zum Beispiel `~/openclaw/skills` und `~/.openclaw/skills`). Dies ist so ausgelegt, dass ältere Skill-Installationen ohne zusätzliche Flags gefunden werden.

<div id="storage-and-lockfile">
  ### Speicherung und Lockfile
</div>

* Installierte Fähigkeiten werden in `.clawhub/lock.json` in deinem Arbeitsverzeichnis gespeichert.
* Auth-Tokens werden in der ClawHub-CLI-Konfigurationsdatei gespeichert (überschreibbar über `CLAWHUB_CONFIG_PATH`).

<div id="telemetry-install-counts">
  ### Telemetrie (Installationszahlen)
</div>

Wenn du `clawhub sync` ausführst, während du angemeldet bist, sendet die CLI eine minimale Momentaufnahme, um Installationszahlen zu berechnen. Du kannst dies vollständig deaktivieren:

```bash
export CLAWHUB_DISABLE_TELEMETRY=1
```

<div id="environment-variables">
  ## Umgebungsvariablen
</div>

* `CLAWHUB_SITE`: Überschreibt die Website-URL.
* `CLAWHUB_REGISTRY`: Überschreibt die Registry-API-URL.
* `CLAWHUB_CONFIG_PATH`: Überschreibt den Pfad, in dem die CLI das Token bzw. die Konfiguration speichert.
* `CLAWHUB_WORKDIR`: Überschreibt das Standardarbeitsverzeichnis.
* `CLAWHUB_DISABLE_TELEMETRY=1`: Deaktiviert Telemetrie bei `sync`.