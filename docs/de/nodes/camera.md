---
title: Kamera
summary: "Kameraaufnahme (iOS-Knoten + macOS-App) für die Nutzung durch Agents: Fotos (jpg) und kurze Videoclips (mp4)"
read_when:
  - Hinzufügen oder Ändern von Kameraaufnahmen auf iOS-Knoten oder unter macOS
  - Erweitern von für Agents zugänglichen MEDIA-Temp-Datei-Workflows
---

<div id="camera-capture-agent">
  # Kameraaufnahme (Agent)
</div>

OpenClaw unterstützt **Kameraaufnahmen** für Agent-Workflows:

- **iOS node** (über das Gateway gekoppelt): Nimm ein **Foto** (`jpg`) oder einen **kurzen Videoclip** (`mp4`, mit optionalem Audio) über `node.invoke` auf.
- **Android node** (über das Gateway gekoppelt): Nimm ein **Foto** (`jpg`) oder einen **kurzen Videoclip** (`mp4`, mit optionalem Audio) über `node.invoke` auf.
- **macOS-App** (node über Gateway): Nimm ein **Foto** (`jpg`) oder einen **kurzen Videoclip** (`mp4`, mit optionalem Audio) über `node.invoke` auf.

Sämtliche Kamerazugriffe sind durch **benutzerkontrollierte Einstellungen** abgesichert.

<div id="ios-node">
  ## iOS-Knoten
</div>

<div id="user-setting-default-on">
  ### Benutzereinstellung (standardmäßig aktiviert)
</div>

- iOS-Einstellungen-Tab → **Camera** → **Allow Camera** (`camera.enabled`)
  - Standardwert: **an** (fehlender Schlüssel wird als aktiviert behandelt).
  - Wenn deaktiviert: `camera.*`-Befehle geben `CAMERA_DISABLED` zurück.

<div id="commands-via-gateway-nodeinvoke">
  ### Befehle (über Gateway `node.invoke`)
</div>

- `camera.list`
  - Antwort-Payload:
    - `devices`: Array mit `{ id, name, position, deviceType }`

- `camera.snap`
  - Parameter:
    - `facing`: `front|back` (Standard: `front`)
    - `maxWidth`: number (optional; Standardwert `1600` auf dem iOS-Knoten)
    - `quality`: `0..1` (optional; Standardwert `0.9`)
    - `format`: derzeit `jpg`
    - `delayMs`: number (optional; Standardwert `0`)
    - `deviceId`: string (optional; aus `camera.list`)
  - Antwort-Payload:
    - `format: "jpg"`
    - `base64: "<...>"`
    - `width`, `height`
  - Payload-Begrenzung: Fotos werden erneut komprimiert, damit die Base64-Payload unter 5 MB bleibt.

- `camera.clip`
  - Parameter:
    - `facing`: `front|back` (Standard: `front`)
    - `durationMs`: number (Standardwert `3000`, begrenzt auf maximal `60000`)
    - `includeAudio`: boolean (Standardwert `true`)
    - `format`: derzeit `mp4`
    - `deviceId`: string (optional; aus `camera.list`)
  - Antwort-Payload:
    - `format: "mp4"`
    - `base64: "<...>"`
    - `durationMs`
    - `hasAudio`

<div id="foreground-requirement">
  ### Anforderung für den Vordergrund
</div>

Wie bei `canvas.*` lässt der iOS-Knoten `camera.*`-Befehle nur im **Vordergrund** zu. Aufrufe im Hintergrund geben `NODE_BACKGROUND_UNAVAILABLE` zurück.

<div id="cli-helper-temp-files-media">
  ### CLI-Hilfsprogramm (temporäre Dateien + MEDIA)
</div>

Am einfachsten erhältst du Anhänge über das CLI-Hilfsprogramm, das dekodierte Medien in eine temporäre Datei schreibt und `MEDIA:<path>` ausgibt.

Beispiele:

```bash
openclaw nodes camera snap --node <id>               # Standard: vorne + hinten (2 MEDIA-Zeilen)
openclaw nodes camera snap --node <id> --facing front
openclaw nodes camera clip --node <id> --duration 3000
openclaw nodes camera clip --node <id> --no-audio
```

Hinweise:

* `nodes camera snap` verwendet standardmäßig **beide** Kameras (Front- und Rückkamera), damit der agent beide Ansichten erhält.
* Die Ausgabedateien sind temporär (im temporären Verzeichnis des Betriebssystems), es sei denn, du baust deinen eigenen Wrapper.


<div id="android-node">
  ## Android-Knoten
</div>

### Benutzereinstellung (standardmäßig ein)

- Android-Einstellungen → **Kamera** → **Kamera zulassen** (`camera.enabled`)
  - Standard: **ein** (fehlender Schlüssel wird als „aktiviert“ behandelt).
  - Wenn aus: `camera.*`-Befehle geben `CAMERA_DISABLED` zurück.

<div id="permissions">
  ### Berechtigungen
</div>

- Android erfordert Berechtigungen zur Laufzeit:
  - `CAMERA` sowohl für `camera.snap` als auch `camera.clip`.
  - `RECORD_AUDIO` für `camera.clip`, wenn `includeAudio=true`.

Wenn Berechtigungen fehlen, fordert die App sie wenn möglich an; wenn sie verweigert werden, schlagen `camera.*`‑Anfragen mit einem `*_PERMISSION_REQUIRED`‑Fehler fehl.

### Vordergrundanforderung

Wie bei `canvas.*` erlaubt der Android-Knoten `camera.*`-Befehle nur im **Vordergrund**. Aufrufe im Hintergrund liefern `NODE_BACKGROUND_UNAVAILABLE` zurück.

<div id="payload-guard">
  ### Payload-Schutz
</div>

Fotos werden erneut komprimiert, damit die Base64-Payload unter 5 MB bleibt.

<div id="macos-app">
  ## macOS-App
</div>

<div id="user-setting-default-off">
  ### Benutzereinstellung (standardmäßig deaktiviert)
</div>

Die macOS-Companion-App stellt ein Kontrollkästchen bereit:

- **Settings → General → Allow Camera** (`openclaw.cameraEnabled`)
  - Standard: **aus**
  - Wenn deaktiviert: Kameraanforderungen geben „Camera disabled by user“ zurück.

<div id="cli-helper-node-invoke">
  ### CLI-Hilfsfunktion (node invoke)
</div>

Verwende die `openclaw` CLI, um Kamerabefehle auf dem macOS-Knoten auszuführen.

Beispiele:

```bash
openclaw nodes camera list --node <id>            # list camera ids
openclaw nodes camera snap --node <id>            # prints MEDIA:<path>
openclaw nodes camera snap --node <id> --max-width 1280
openclaw nodes camera snap --node <id> --delay-ms 2000
openclaw nodes camera snap --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --duration 10s          # prints MEDIA:<path>
openclaw nodes camera clip --node <id> --duration-ms 3000      # gibt MEDIA:<path> aus (veraltetes Flag)
openclaw nodes camera clip --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --no-audio
```

Hinweise:

* `openclaw nodes camera snap` verwendet standardmäßig `maxWidth=1600`, sofern nicht anders gesetzt.
* Unter macOS wartet `camera.snap` um `delayMs` (standardmäßig 2000 ms) nach dem Aufwärmen bzw. nachdem sich die Belichtung stabilisiert hat, bevor das Bild aufgenommen wird.
* Foto-Payloads werden neu komprimiert, damit Base64 unter 5 MB bleibt.


<div id="safety-practical-limits">
  ## Sicherheit + praktische Grenzen
</div>

- Kamera- und Mikrofonzugriff lösen die üblichen OS-Berechtigungsabfragen aus (und erfordern Usage-Strings in der Info.plist).
- Videoclips sind begrenzt (derzeit `<= 60s`), um übergroße Knoten-Payloads zu vermeiden (Base64-Overhead + Nachrichtenlimits).

<div id="macos-screen-video-os-level">
  ## macOS-Bildschirmaufnahme (OS-Ebene)
</div>

Für *Bildschirm*-Aufnahmen (nicht Kamera) verwenden Sie die macOS-Companion-App:

```bash
openclaw nodes screen record --node <id> --duration 10s --fps 15   # gibt MEDIA:<path> aus
```

Hinweise:

* Erfordert die macOS-Berechtigung **„Bildschirmaufnahme“** (TCC).
