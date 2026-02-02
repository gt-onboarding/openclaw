---
title: Agent-Arbeitsbereich
summary: "Agent-Arbeitsbereich: Speicherort, Struktur und Backup-Strategie"
read_when:
  - Du musst den Agent-Arbeitsbereich oder dessen Dateistruktur erklären
  - Du möchtest einen Agent-Arbeitsbereich sichern oder migrieren
---

<div id="agent-workspace">
  # Agent workspace
</div>

Der Arbeitsbereich ist das Zuhause des Agents. Er ist das einzige Arbeitsverzeichnis, das für
Datei-Tools und für den Kontext des Arbeitsbereichs verwendet wird. Halte ihn privat und behandle ihn wie ein Gedächtnis.

Dies ist etwas anderes als `~/.openclaw/`, das Konfiguration, Zugangsdaten und
Sitzungen speichert.

**Wichtig:** Der Arbeitsbereich ist das **Standard-`cwd`**, keine strikte sandbox. Tools
lösen relative Pfade relativ zum Arbeitsbereich auf, aber absolute Pfade können weiterhin
an andere Orte auf dem Host gelangen, sofern sandboxing nicht aktiviert ist. Wenn du Isolation brauchst, verwende
[`agents.defaults.sandbox`](/de/gateway/sandboxing) (und/oder sandbox-Konfiguration pro Agent).
Wenn sandboxing aktiviert ist und `workspaceAccess` nicht `"rw"` ist, arbeiten Tools
in einem sandbox-Arbeitsbereich unter `~/.openclaw/sandboxes` statt in deinem Host-Arbeitsbereich.

<div id="default-location">
  ## Standardpfad
</div>

* Standard: `~/.openclaw/workspace`
* Wenn `OPENCLAW_PROFILE` gesetzt ist und nicht auf `"default"` steht, wird der Standardpfad
  `~/.openclaw/workspace-<profile>`.
* Kann in `~/.openclaw/openclaw.json` überschrieben werden:

```json5
{
  agent: {
    workspace: "~/.openclaw/workspace"
  }
}
```

`openclaw onboard`, `openclaw configure` oder `openclaw setup` erstellen den
Arbeitsbereich und legen die Bootstrap-Dateien an, falls sie fehlen.

Wenn du die Arbeitsbereichsdateien bereits selbst verwaltest, kannst du das Anlegen der Bootstrap-Dateien deaktivieren:

```json5
{ agent: { skipBootstrap: true } }
```

<div id="extra-workspace-folders">
  ## Zusätzliche Arbeitsbereichsordner
</div>

Ältere Installationen haben möglicherweise `~/openclaw` erstellt. Mehrere
Arbeitsbereichsverzeichnisse können zu verwirrenden Abweichungen bei Authentifizierung oder Zustand führen, da immer nur ein Arbeitsbereich aktiv ist.

**Empfehlung:** Verwende nur einen aktiven Arbeitsbereich. Wenn du die
zusätzlichen Ordner nicht mehr benötigst, archiviere sie oder verschiebe sie in
den Papierkorb (zum Beispiel `trash ~/openclaw`). Wenn du absichtlich mehrere
Arbeitsbereiche behältst, stelle sicher, dass
`agents.defaults.workspace` auf den aktiven Arbeitsbereich zeigt.

`openclaw doctor` warnt, wenn zusätzliche Arbeitsbereichsverzeichnisse erkannt werden.

<div id="workspace-file-map-what-each-file-means">
  ## Dateiübersicht im Arbeitsbereich (was jede Datei bedeutet)
</div>

Dies sind die Standarddateien, die OpenClaw im arbeitsbereich erwartet:

* `AGENTS.md`
  * Betriebshinweise für den agenten und wie er Speicher verwenden soll.
  * Wird zu Beginn jeder Sitzung geladen.
  * Guter Ort für Regeln, Prioritäten und Details zum „Verhalten“.

* `SOUL.md`
  * Persona, Tonfall und Grenzen.
  * Wird in jeder Sitzung geladen.

* `USER.md`
  * Wer der Benutzer ist und wie er angesprochen werden soll.
  * Wird in jeder Sitzung geladen.

* `IDENTITY.md`
  * Name, „Vibe“ und Emoji des agenten.
  * Wird während des Bootstrap-Rituals erstellt/aktualisiert.

* `TOOLS.md`
  * Notizen zu deinen lokalen Tools und Konventionen.
  * Steuert nicht die Toolverfügbarkeit; dient nur als Orientierung.

* `HEARTBEAT.md`
  * Optionale kurze Checkliste für Herzschlag-Läufe.
  * Halte sie kurz, um Token-Verbrauch zu vermeiden.

* `BOOT.md`
  * Optionale Start-Checkliste, die bei einem Gateway-Neustart ausgeführt wird, wenn interne Hooks aktiviert sind.
  * Halte sie kurz; verwende das Nachrichtentool für ausgehende Sendungen.

* `BOOTSTRAP.md`
  * Einmaliges Erststart-Ritual.
  * Wird nur für einen brandneuen arbeitsbereich erstellt.
  * Lösche sie, nachdem das Ritual abgeschlossen ist.

* `memory/YYYY-MM-DD.md`
  * Tägliches Speicherprotokoll (eine Datei pro Tag).
  * Es wird empfohlen, beim Sitzungsstart die Dateien von heute + gestern mit read einzulesen.

* `MEMORY.md` (optional)
  * Kuratierter Langzeitspeicher.
  * Nur in der Haupt-, privaten Sitzung laden (nicht in geteilten/Gruppenkontexten).

Siehe [Memory](/de/concepts/memory) für den Workflow und das automatische Leeren des Speichers.

* `skills/` (optional)
  * Arbeitsbereichsspezifische Fähigkeiten.
  * Überschreibt verwaltete/gebündelte Fähigkeiten, wenn Namen kollidieren.

* `canvas/` (optional)
  * Canvas-UI-Dateien für knoten-Displays (zum Beispiel `canvas/index.html`).

Wenn eine Bootstrap-Datei fehlt, fügt OpenClaw eine „missing file“-Markierung in
die Sitzung ein und macht weiter. Große Bootstrap-Dateien werden beim Einfügen gekürzt;
passe das Limit mit `agents.defaults.bootstrapMaxChars` an (Standard: 20000).
`openclaw setup` kann fehlende Standarddateien neu erstellen, ohne vorhandene
Dateien zu überschreiben.

<div id="what-is-not-in-the-workspace">
  ## Was NICHT im Arbeitsbereich ist
</div>

Diese befinden sich unter `~/.openclaw/` und sollten NICHT in das Arbeitsbereichs-Repo eingecheckt werden:

* `~/.openclaw/openclaw.json` (Konfiguration)
* `~/.openclaw/credentials/` (OAuth-Tokens, API-Schlüssel)
* `~/.openclaw/agents/<agentId>/sessions/` (Sitzungsprotokolle + Metadaten)
* `~/.openclaw/skills/` (verwaltete Fähigkeiten)

Wenn du Sitzungen oder Konfiguration migrieren musst, kopiere sie separat und halte sie außerhalb der Versionsverwaltung.

<div id="git-backup-recommended-private">
  ## Git-Backup (empfohlen, privat)
</div>

Behandle den Arbeitsbereich wie einen privaten Speicher. Lege ihn in ein **privates** Git-Repository, sodass er gesichert und wiederherstellbar ist.

Führe diese Schritte auf der Maschine aus, auf der das Gateway läuft (dort befindet sich der Arbeitsbereich).

<div id="1-initialize-the-repo">
  ### 1) Repo initialisieren
</div>

Wenn Git installiert ist, werden neue Arbeitsbereiche automatisch initialisiert. Wenn dieser
Arbeitsbereich noch kein Repo ist, führe Folgendes aus:

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md SOUL.md TOOLS.md IDENTITY.md USER.md HEARTBEAT.md memory/
git commit -m "Agent-Arbeitsbereich hinzufügen"
```

<div id="2-add-a-private-remote-beginner-friendly-options">
  ### 2) Privates Remote-Repository hinzufügen (einsteigerfreundliche Optionen)
</div>

Option A: GitHub-Web-UI

1. Erstelle ein neues **privates** Repository auf GitHub.
2. Initialisiere es nicht mit einer README-Datei (vermeidet Merge-Konflikte).
3. Kopiere die HTTPS-Remote-URL.
4. Füge das Remote hinzu und führe einen Push durch:

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

Option B: GitHub CLI (`gh`)

```bash
gh auth login
gh repo create openclaw-workspace --private --source . --remote origin --push
```

Option C: GitLab Web-UI

1. Erstelle ein neues **privates** Repository auf GitLab.
2. Initialisiere es nicht mit einer README-Datei (vermeidet Merge-Konflikte).
3. Kopiere die HTTPS-Remote-URL.
4. Füge das Remote hinzu und pushe:

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

<div id="3-ongoing-updates">
  ### 3) Fortlaufende Aktualisierungen
</div>

```bash
git status
git add .
git commit -m "Aktualisiere Speicher"
git push
```

<div id="do-not-commit-secrets">
  ## Keine Secrets committen
</div>

Vermeide es, Secrets im arbeitsbereich zu speichern – selbst in einem privaten Repository:

* API-Keys, OAuth-Tokens, Passwörter oder private Zugangsdaten.
* Alles unter `~/.openclaw/`.
* Unbearbeitete Dumps von Chats oder sensible Anhänge.

Wenn du unbedingt vertrauliche Referenzen speichern musst, verwende Platzhalter und bewahre das eigentliche Secret an einem anderen Ort auf (Passwortmanager, Umgebungsvariablen oder `~/.openclaw/`).

Vorgeschlagene `.gitignore`-Vorlage:

```gitignore
.DS_Store
.env
**/*.key
**/*.pem
**/secrets*
```

<div id="moving-the-workspace-to-a-new-machine">
  ## Verschieben des Arbeitsbereichs auf einen neuen Rechner
</div>

1. Klone das Repository in den gewünschten Pfad (Standard `~/.openclaw/workspace`).
2. Setze `agents.defaults.workspace` in `~/.openclaw/openclaw.json` auf diesen Pfad.
3. Führe `openclaw setup --workspace <path>` aus, um ggf. fehlende Dateien anzulegen.
4. Falls du Sitzungen benötigst, kopiere `~/.openclaw/agents/<agentId>/sessions/` von deinem
   alten Rechner separat.

<div id="advanced-notes">
  ## Erweiterte Hinweise
</div>

* Multi-Agent-Routing kann für jeden Agenten unterschiedliche Arbeitsbereiche verwenden. Siehe
  [Channel routing](/de/concepts/channel-routing) für die Konfiguration des Routings.
* Wenn `agents.defaults.sandbox` aktiviert ist, können alle Sitzungen außer der Hauptsitzung eigene sandbox-Arbeitsbereiche unter `agents.defaults.sandbox.workspaceRoot` verwenden.