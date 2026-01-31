---
title: Strikte Konfiguration
summary: "Strikte Konfigurationsvalidierung + Migrationen ausschließlich über doctor"
read_when:
  - Beim Entwerfen oder Implementieren von Konfigurationsvalidierung
  - Bei der Arbeit an Konfigurationsmigrationen oder doctor-Workflows
  - Beim Umgang mit Plugin-Konfigurationsschemata oder der Steuerung des Plugin-Ladevorgangs
---

<div id="strict-config-validation-doctor-only-migrations">
  # Strikte Konfigurationsvalidierung (Migrationen ausschließlich über `doctor`)
</div>

<div id="goals">
  ## Ziele
</div>

- **Überall unbekannte Config-Keys ablehnen** (Root + verschachtelt).
- **Plugin-Config ohne Schema ablehnen**; dieses Plugin nicht laden.
- **Legacy-Automigration beim Laden entfernen**; Migrationen laufen nur über `doctor`.
- **`doctor` beim Start automatisch (Dry-Run) ausführen**; bei ungültiger Konfiguration nicht-diagnostische Befehle blockieren.

<div id="non-goals">
  ## Nichtziele
</div>

- Abwärtskompatibilität beim Laden (Legacy-Keys werden nicht automatisch migriert).
- Stilles Verwerfen nicht erkannter Keys.

<div id="strict-validation-rules">
  ## Strikte Validierungsregeln
</div>

- Die Konfiguration muss auf allen Ebenen exakt dem Schema entsprechen.
- Unbekannte Schlüssel sind Validierungsfehler (kein Durchreichen auf Root- oder verschachtelten Ebenen).
- `plugins.entries.<id>.config` muss anhand des Schemas des Plugins validiert werden.
  - Wenn ein Plugin kein Schema besitzt, **Laden des Plugins verweigern** und einen klaren Fehler ausgeben.
- Unbekannte Schlüssel unter `channels.<id>` sind Fehler, außer ein Plugin-Manifest deklariert die Channel-ID.
- Plugin-Manifeste (`openclaw.plugin.json`) sind für alle Plugins erforderlich.

<div id="plugin-schema-enforcement">
  ## Durchsetzung des Plugin-Schemas
</div>

- Jedes Plugin stellt ein striktes JSON-Schema für seine Konfiguration bereit (inline im Manifest).
- Plugin-Ladeablauf:
  1) Plugin-Manifest + Schema ermitteln (`openclaw.plugin.json`).
  2) Konfiguration gegen das Schema validieren.
  3) Wenn Schema fehlt oder Konfiguration ungültig ist: Plugin-Laden blockieren, Fehler protokollieren.
- Die Fehlermeldung enthält:
  - Plugin-ID
  - Grund (fehlendes Schema / ungültige Konfiguration)
  - Pfad/Pfade, die die Validierung nicht bestanden haben
- Deaktivierte Plugins behalten ihre Konfiguration, aber Doctor + Logs melden eine Warnung.

<div id="doctor-flow">
  ## Doctor-Flow
</div>

- Doctor wird **jedes Mal** ausgeführt, wenn die Konfiguration geladen wird (standardmäßig als Dry-Run).
- Wenn die Konfiguration ungültig ist:
  - Gibt eine Zusammenfassung + konkrete Fehlermeldungen aus.
  - Weist an: `openclaw doctor --fix`.
- `openclaw doctor --fix`:
  - Führt Migrationen durch.
  - Entfernt unbekannte Schlüssel.
  - Schreibt die aktualisierte Konfiguration.

<div id="command-gating-when-config-is-invalid">
  ## Befehlsbeschränkung (bei ungültiger Konfiguration)
</div>

Erlaubt (nur zu Diagnosezwecken):

- `openclaw doctor`
- `openclaw logs`
- `openclaw health`
- `openclaw help`
- `openclaw status`
- `openclaw gateway status`

Alle anderen Befehle müssen mit einem Hard-Fail abbrechen und ausgeben: „Config invalid. Run `openclaw doctor --fix`.“

<div id="error-ux-format">
  ## Fehler-UX-Format
</div>

- Einzelne Zusammenfassungsüberschrift.
- Gruppierte Abschnitte:
  - Unbekannte Schlüssel (vollständige Pfade)
  - Legacy-Schlüssel / erforderliche Migrationen
  - Plugin-Ladefehler (Plugin-ID + Grund + Pfad)

<div id="implementation-touchpoints">
  ## Implementierungsstellen
</div>

- `src/config/zod-schema.ts`: Root-Passthrough entfernen; überall strikte Objekte.
- `src/config/zod-schema.providers.ts`: strikte Channel-Schemata sicherstellen.
- `src/config/validation.ts`: bei unbekannten Schlüsseln fehlschlagen; keine Legacy-Migrationen anwenden.
- `src/config/io.ts`: Legacy-Auto-Migrationen entfernen; `doctor` immer im Dry-Run ausführen.
- `src/config/legacy*.ts`: Verwendung ausschließlich in `doctor` verschieben.
- `src/plugins/*`: Schema-Registry + Gating hinzufügen.
- CLI-Befehls-Gating in `src/cli`.

<div id="tests">
  ## Tests
</div>

- Zurückweisung unbekannter Schlüssel (Root + verschachtelt).
- Fehlendes Plugin-Schema → Laden des Plugins mit klarer Fehlermeldung blockiert.
- Ungültige Konfiguration → Start des Gateway blockiert, Diagnosebefehle ausgenommen.
- Automatischer Trockenlauf von `doctor`; `doctor --fix` schreibt die korrigierte Konfiguration.