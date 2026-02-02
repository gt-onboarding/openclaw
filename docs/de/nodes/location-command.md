---
title: Standortbefehl
summary: "Standortbefehl für Knoten (location.get), Berechtigungsmodi und Hintergrundverhalten"
read_when:
  - Wenn du Standortunterstützung für Knoten oder für die Berechtigungs-UI hinzufügst
  - Beim Entwerfen von Hintergrund-Standort- und Push-Flows
---

<div id="location-command-nodes">
  # Befehl „location“ (Knoten)
</div>

<div id="tldr">
  ## TL;DR
</div>

- `location.get` ist ein Node-Befehl (über `node.invoke`).
- Ist standardmäßig deaktiviert.
- Die Einstellungen verwenden eine Auswahl: Aus / Nur während der Nutzung / Immer.
- Separater Schalter: Genaue Position.

<div id="why-a-selector-not-just-a-switch">
  ## Warum eine Auswahl (nicht nur ein Schalter)
</div>

OS-Berechtigungen sind mehrstufig. Wir können in der App eine Auswahl anbieten, aber das OS entscheidet weiterhin über die tatsächlich erteilte Berechtigung.

- iOS/macOS: Benutzer können in Systemdialogen/Einstellungen **Beim Verwenden** oder **Immer** wählen. Die App kann eine höhere Stufe der Berechtigung anfordern, aber das OS kann dafür den Weg über die Einstellungen erzwingen.
- Android: Hintergrundstandort ist eine separate Berechtigung; unter Android 10+ erfordert sie häufig einen Flow über die Einstellungen.
- Genaue Position ist eine separate Berechtigung (iOS 14+ „Genaue Position“, Android „fine“ vs. „coarse“).

Die Auswahl in der UI steuert den von uns angeforderten Modus; die tatsächlich erteilte Berechtigung wird in den OS-Einstellungen festgelegt.

<div id="settings-model">
  ## Einstellungsmodell
</div>

Pro Node-Gerät:

- `location.enabledMode`: `off | whileUsing | always`
- `location.preciseEnabled`: bool

UI-Verhalten:

- Die Auswahl von `whileUsing` fordert die Berechtigung für den Vordergrund an.
- Die Auswahl von `always` aktiviert zunächst `whileUsing` und fordert dann die Berechtigung für den Hintergrund an (oder leitet den Benutzer bei Bedarf zu den Einstellungen weiter).
- Wenn das Betriebssystem die angeforderte Stufe verweigert, wird auf die höchste gewährte Stufe zurückgefallen und der Status angezeigt.

<div id="permissions-mapping-nodepermissions">
  ## Berechtigungszuordnung (node.permissions)
</div>

Optional. Der macOS-Knoten meldet `location` über die Berechtigungszuordnung; unter iOS/Android kann dieser Eintrag fehlen.

<div id="command-locationget">
  ## Befehl: `location.get`
</div>

Aufgerufen über `node.invoke`.

Empfohlene Parameter:

```json
{
  "timeoutMs": 10000,
  "maxAgeMs": 15000,
  "desiredAccuracy": "coarse|balanced|precise"
}
```

Antwortdaten:

```json
{
  "lat": 48.20849,
  "lon": 16.37208,
  "accuracyMeters": 12.5,
  "altitudeMeters": 182.0,
  "speedMps": 0.0,
  "headingDeg": 270.0,
  "timestamp": "2026-01-03T12:34:56.000Z",
  "isPrecise": true,
  "source": "gps|wifi|cell|unknown"
}
```

Fehler (stabile Fehlercodes):

* `LOCATION_DISABLED`: Standortschalter ist deaktiviert.
* `LOCATION_PERMISSION_REQUIRED`: Berechtigung für den angeforderten Modus fehlt.
* `LOCATION_BACKGROUND_UNAVAILABLE`: App läuft im Hintergrund, aber nur „Während der Nutzung“ ist erlaubt.
* `LOCATION_TIMEOUT`: keine Positionsbestimmung innerhalb des Zeitlimits.
* `LOCATION_UNAVAILABLE`: Systemfehler / keine anbieter verfügbar.


<div id="background-behavior-future">
  ## Hintergrundverhalten (zukünftig)
</div>

Ziel: Das Modell kann den Standort anfordern, selbst wenn der Knoten im Hintergrund ausgeführt wird, aber nur wenn:

- Der Nutzer **Immer** ausgewählt hat.
- Das Betriebssystem Standortzugriff im Hintergrund gewährt.
- Die App im Hintergrund für Standortzugriff ausgeführt werden darf (iOS-Hintergrundmodus / Android-Vordergrunddienst oder spezielle Freigabe).

Push-ausgelöster Ablauf (zukünftig):

1) Gateway sendet einen Push an den Knoten (stiller Push oder FCM-Daten).
2) Knoten wird kurz aufgeweckt und fordert den Standort vom Gerät an.
3) Knoten leitet die Nutzlast an das Gateway weiter.

Hinweise:

- iOS: Berechtigung „Immer“ + Hintergrundmodus für Standort erforderlich. Stiller Push kann gedrosselt werden; es ist mit sporadischen Ausfällen zu rechnen.
- Android: Standort im Hintergrund kann einen Vordergrunddienst erfordern; andernfalls ist mit einer Ablehnung zu rechnen.

<div id="modeltooling-integration">
  ## Modell-/Tooling-Integration
</div>

- Tool-Oberfläche: Das `nodes`-Tool fügt die Aktion `location_get` hinzu (Knoten erforderlich).
- CLI: `openclaw nodes location get --node <id>`.
- Agent-Richtlinien: Nur aufrufen, wenn der Benutzer die Standortfreigabe aktiviert hat und den Scope versteht.

<div id="ux-copy-suggested">
  ## UX-Text (Vorschlag)
</div>

- Aus: „Standortfreigabe ist deaktiviert.“
- Während der Nutzung: „Nur wenn OpenClaw geöffnet ist.“
- Immer: „Standortzugriff im Hintergrund erlauben. Erfordert Systemberechtigung.“
- Präzise: „Präzise GPS-Position verwenden. Deaktivieren, um nur einen ungefähren Standort zu teilen.“