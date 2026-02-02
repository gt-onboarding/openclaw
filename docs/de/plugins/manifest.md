---
title: Manifest
summary: "Plugin-Manifest + JSON-Schema-Anforderungen (strenge Konfigurationsvalidierung)"
read_when:
  - Du entwickelst ein OpenClaw-Plugin
  - Du musst ein Plugin-Konfigurationsschema bereitstellen oder Plugin-Validierungsfehler debuggen
---

<div id="plugin-manifest-openclawpluginjson">
  # Plugin-Manifest (openclaw.plugin.json)
</div>

Jedes Plugin **muss** im **Plugin-Root-Verzeichnis** eine Datei `openclaw.plugin.json` mitliefern.
OpenClaw verwendet dieses Manifest, um die Konfiguration zu validieren,
**ohne Plugin-Code auszuführen**. Fehlende oder ungültige Manifeste werden als
Plugin-Fehler behandelt und blockieren die Validierung der Konfiguration.

Die vollständige Anleitung zum Plugin-System findest du hier: [Plugins](/de/plugin).

<div id="required-fields">
  ## Pflichtfelder
</div>

```json
{
  "id": "voice-call",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {}
  }
}
```

Erforderliche Schlüssel:

* `id` (string): kanonische Plugin-ID.
* `configSchema` (object): JSON-Schema für die Plugin-Konfiguration (inline).

Optionale Schlüssel:

* `kind` (string): Plugin-Typ (Beispiel: `"memory"`).
* `channels` (array): Kanal-IDs, die von diesem Plugin registriert werden (Beispiel: `["matrix"]`).
* `providers` (array): Anbieter-IDs, die von diesem Plugin registriert werden.
* `skills` (array): Verzeichnisse von Fähigkeiten, die geladen werden sollen (relativ zum Plugin-Root-Verzeichnis).
* `name` (string): Anzeigename für das Plugin.
* `description` (string): kurze Plugin-Zusammenfassung.
* `uiHints` (object): Beschriftungen/Platzhalter/sensible Flags für Konfigurationsfelder zur Darstellung in der UI.
* `version` (string): Plugin-Version (informativ).

<div id="json-schema-requirements">
  ## Anforderungen an JSON-Schemas
</div>

* **Jedes Plugin muss ein JSON-Schema bereitstellen**, auch wenn es keine Konfiguration akzeptiert.
* Ein leeres Schema ist zulässig (zum Beispiel `{ "type": "object", "additionalProperties": false }`).
* Die Schemas werden beim Lesen/Schreiben der Konfiguration validiert, nicht zur Laufzeit.

<div id="validation-behavior">
  ## Validierungsverhalten
</div>

* Unbekannte `channels.*`-Schlüssel sind **Fehler**, es sei denn, die Channel-ID
  wird von einem Plugin-Manifest deklariert.
* `plugins.entries.<id>`, `plugins.allow`, `plugins.deny` und `plugins.slots.*`
  müssen auf **auffindbare** Plugin-IDs verweisen. Unbekannte IDs sind
  **Fehler**.
* Wenn ein Plugin installiert ist, aber ein beschädigtes oder fehlendes
  Manifest oder Schema hat, schlägt die Validierung fehl und Doctor meldet den
  Plugin-Fehler.
* Wenn eine Plugin-Konfiguration vorhanden ist, das Plugin jedoch
  **deaktiviert** ist, bleibt die Konfiguration erhalten und eine **Warnung**
  wird in Doctor und in den Logs ausgegeben.

<div id="notes">
  ## Hinweise
</div>

* Das Manifest ist **für alle Plugins erforderlich**, einschließlich lokaler Ladevorgänge aus dem Dateisystem.
* Zur Laufzeit wird das Plugin-Modul weiterhin separat geladen; das Manifest dient nur zur
  Auffindbarkeit und Validierung.
* Wenn dein Plugin von nativen Modulen abhängt, dokumentiere die Build-Schritte und alle
  Allowlist-Anforderungen des Paketmanagers (zum Beispiel pnpm `allow-build-scripts`
  * `pnpm rebuild <package>`).