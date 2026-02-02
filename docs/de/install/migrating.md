---
title: Migration
summary: "Eine OpenClaw-Installation von einem Rechner auf einen anderen umziehen (migrieren)"
read_when:
  - Du ziehst mit OpenClaw auf einen neuen Laptop/Server um
  - Du möchtest Sitzungen, Authentifizierungsdaten und Kanal-Logins (WhatsApp usw.) beibehalten
---

<div id="migrating-openclaw-to-a-new-machine">
  # OpenClaw auf eine neue Maschine migrieren
</div>

Diese Anleitung beschreibt, wie du ein OpenClaw Gateway von einer Maschine auf eine andere migrierst — **ohne das Onboarding zu wiederholen**.

Konzeptionell ist die Migration einfach:

* Kopiere das **State-Verzeichnis** (`$OPENCLAW_STATE_DIR`, Standard: `~/.openclaw/`) — dieses umfasst Konfiguration, Authentifizierung, Sitzungen und Kanalzustand.
* Kopiere deinen **Arbeitsbereich** (standardmäßig `~/.openclaw/workspace/`) — dieser umfasst deine agent-Dateien (Memory, Prompts usw.).

Es gibt jedoch typische Fallstricke rund um **Profile**, **Berechtigungen** und **unvollständige Kopien**.

<div id="before-you-start-what-you-are-migrating">
  ## Bevor du beginnst (was du migrierst)
</div>

<div id="1-identify-your-state-directory">
  ### 1) Ermittle dein State-Verzeichnis
</div>

Die meisten Installationen verwenden den Standardpfad:

* **State-Verzeichnis:** `~/.openclaw/`

Es kann aber abweichen, wenn du Folgendes verwendest:

* `--profile <name>` (wird oft zu `~/.openclaw-<profile>/`)
* `OPENCLAW_STATE_DIR=/some/path`

Wenn du dir nicht sicher bist, führe auf der **alten** Maschine Folgendes aus:

```bash
openclaw status
```

Suche in der Ausgabe nach Hinweisen auf `OPENCLAW_STATE_DIR` bzw. das Profil. Wenn du mehrere Gateways betreibst, wiederhole diesen Schritt für jedes Profil.

<div id="2-identify-your-workspace">
  ### 2) Identifiziere deinen Arbeitsbereich
</div>

Gängige Standardpfade:

* `~/.openclaw/workspace/` (empfohlener Arbeitsbereich)
* ein benutzerdefinierter Ordner, den du erstellt hast

Dein Arbeitsbereich ist der Ort, an dem Dateien wie `MEMORY.md`, `USER.md` und `memory/*.md` gespeichert sind.

<div id="3-understand-what-you-will-preserve">
  ### 3) Verstehe, was erhalten bleibt
</div>

Wenn du **sowohl** das State-Verzeichnis als auch den Arbeitsbereich kopierst, behältst du:

* Gateway-Konfiguration (`openclaw.json`)
* Auth-Profile / API-Schlüssel / OAuth-Tokens
* Sitzungsverlauf + Agent-Zustand
* Channel-Zustand (z. B. WhatsApp-Login/Sitzung)
* Deine Arbeitsbereich-Dateien (Memory, Notizen zu Fähigkeiten usw.)

Wenn du **nur** den Arbeitsbereich kopierst (z. B. per Git), dann **bleiben nicht erhalten**:

* Sitzungen
* Zugangsdaten
* Channel-Logins

Diese liegen unter `$OPENCLAW_STATE_DIR`.

<div id="migration-steps-recommended">
  ## Empfohlene Migrationsschritte
</div>

<div id="step-0-make-a-backup-old-machine">
  ### Schritt 0 — Backup erstellen (alte Maschine)
</div>

Auf der **alten** Maschine stoppe das Gateway zuerst, damit sich Dateien während des Kopiervorgangs nicht ändern:

```bash
openclaw gateway stop
```

(Optional, aber empfohlen) State-Verzeichnis und Arbeitsbereich archivieren:

```bash
# Passen Sie die Pfade an, wenn Sie ein Profil oder benutzerdefinierte Speicherorte verwenden
cd ~
tar -czf openclaw-state.tgz .openclaw

tar -czf openclaw-workspace.tgz .openclaw/workspace
```

Wenn du mehrere Profil-/State-Verzeichnisse hast (z. B. `~/.openclaw-main`, `~/.openclaw-work`), archiviere jedes einzelne davon.

<div id="step-1-install-openclaw-on-the-new-machine">
  ### Schritt 1 — OpenClaw auf der neuen Maschine installieren
</div>

Auf der **neuen** Maschine installierst du die CLI (und bei Bedarf Node.js):

* Siehe: [Installation](/de/install)

An diesem Punkt ist es unproblematisch, wenn das Onboarding ein frisches `~/.openclaw/` erzeugt — du überschreibst es im nächsten Schritt.

<div id="step-2-copy-the-state-dir-workspace-to-the-new-machine">
  ### Schritt 2 — State-Verzeichnis + Arbeitsbereich auf die neue Maschine kopieren
</div>

Kopiere **beides**:

* `$OPENCLAW_STATE_DIR` (Standardpfad `~/.openclaw/`)
* deinen Arbeitsbereich (Standardpfad `~/.openclaw/workspace/`)

Gängige Vorgehensweisen:

* Tarballs per `scp` übertragen und entpacken
* `rsync -a` über SSH
* Externe Festplatte

Stelle nach dem Kopieren sicher:

* Versteckte Verzeichnisse wurden mitkopiert (z. B. `.openclaw/`)
* Dateieigentümer sind korrekt für den Benutzer gesetzt, der das Gateway ausführt

<div id="step-3-run-doctor-migrations-service-repair">
  ### Schritt 3 — Doctor ausführen (Migrationen + Dienstreparatur)
</div>

Auf der **neuen** Maschine:

```bash
openclaw doctor
```

Doctor ist der „sichere, unspektakuläre“ Befehl. Er repariert Dienste, führt Konfigurationsmigrationen durch und warnt vor Unstimmigkeiten.

Dann:

```bash
openclaw gateway restart
openclaw status
```

<div id="common-footguns-and-how-to-avoid-them">
  ## Häufige Fallstricke (und wie du sie vermeidest)
</div>

<div id="footgun-profile-state-dir-mismatch">
  ### Fallstrick: Profil-/State-Verzeichnis-Konflikt
</div>

Wenn du das alte Gateway mit einem Profil (oder `OPENCLAW_STATE_DIR`) ausgeführt hast und das neue Gateway ein anderes verwendet, treten Symptome auf wie:

* Konfigurationsänderungen werden nicht übernommen
* Kanäle fehlen oder sind abgemeldet
* Leerer Sitzungsverlauf

Lösung: Starte das Gateway/den Dienst mit **demselben** Profil-/State-Verzeichnis, das du migriert hast, und führe den Befehl dann erneut aus:

```bash
openclaw doctor
```

<div id="footgun-copying-only-openclawjson">
  ### Fallstrick: nur `openclaw.json` kopieren
</div>

`openclaw.json` reicht nicht aus. Viele Anbieter speichern ihren Zustand unter:

* `$OPENCLAW_STATE_DIR/credentials/`
* `$OPENCLAW_STATE_DIR/agents/<agentId>/...`

Migriere immer den gesamten Ordner `$OPENCLAW_STATE_DIR`.

<div id="footgun-permissions-ownership">
  ### Footgun: Berechtigungen / Besitzrechte
</div>

Wenn du als root kopiert oder den Benutzer gewechselt hast, kann das Gateway möglicherweise keine Anmeldedaten/Sitzungen lesen.

Lösung: Stell sicher, dass das State-Verzeichnis + der Arbeitsbereich dem Benutzer gehören, der das Gateway ausführt.

<div id="footgun-migrating-between-remotelocal-modes">
  ### Footgun: Migration zwischen Remote-/lokalen Modi
</div>

* Wenn deine UI (WebUI/TUI) mit einem **remote** Gateway verbunden ist, liegen Sitzungs-Speicher und Arbeitsbereich auf dem Remote-Host.
* Die Migration deines Laptops verschiebt den Status des Remote-Gateways nicht.

Wenn du im Remote-Modus bist, migriere den **Gateway-Host**.

<div id="footgun-secrets-in-backups">
  ### Fallstrick: Secrets in Backups
</div>

`$OPENCLAW_STATE_DIR` enthält Secrets (API-Keys, OAuth-Tokens, WhatsApp-Credentials). Behandle Backups wie Produktions-Secrets:

* immer verschlüsselt speichern
* nicht über unsichere Kanäle teilen
* Schlüssel rotieren, wenn du eine mögliche Kompromittierung vermutest

<div id="verification-checklist">
  ## Checkliste zur Überprüfung
</div>

Überprüfe auf dem neuen Rechner:

* `openclaw status` zeigt, dass das Gateway läuft
* Deine Kanäle sind weiterhin verbunden (z. B. erfordert WhatsApp kein erneutes Pairing)
* Das Dashboard lässt sich öffnen und zeigt bestehende Sitzungen an
* Deine Arbeitsbereichsdateien (Memory, Konfigurationen) sind vorhanden

<div id="related">
  ## Verwandtes
</div>

* [Doctor](/de/gateway/doctor)
* [Fehlerbehebung für das Gateway](/de/gateway/troubleshooting)
* [Wo speichert OpenClaw seine Daten?](/de/help/faq#where-does-openclaw-store-its-data)