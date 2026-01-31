---
title: Knoten
summary: "Knoten: Kopplung, Funktionen, Berechtigungen und CLI-Helfer für canvas/camera/screen/system"
read_when:
  - Kopplung von iOS-/Android-Knoten mit dem Gateway
  - Verwendung von canvas/camera eines Knotens für den Agent-Kontext
  - Hinzufügen neuer Knotenbefehle oder CLI-Helfer
---

<div id="nodes">
  # Knoten
</div>

Ein **Knoten** ist ein Begleitgerät (macOS/iOS/Android/headless), das sich mit `role: "node"` über den **WebSocket** des Gateways (gleicher Port wie für Operatoren) verbindet und eine Befehlsschnittstelle (z. B. `canvas.*`, `camera.*`, `system.*`) über `node.invoke` bereitstellt. Details zum Protokoll: [Gateway protocol](/de/gateway/protocol).

Veralteter Transport: [Bridge protocol](/de/gateway/bridge-protocol) (TCP JSONL; für aktuelle Knoten veraltet/entfernt).

macOS kann auch im **Knoten-Modus** laufen: Die Menüleisten-App verbindet sich mit dem WS-Server des Gateways und stellt ihre lokalen canvas-/camera-Befehle als Knoten bereit (sodass `openclaw nodes …` gegen diesen Mac funktioniert).

Hinweise:

* Knoten sind **Peripheriegeräte**, keine Gateways. Sie führen den Gateway-Dienst nicht aus.
* Telegram-/WhatsApp-/etc.-Nachrichten landen auf dem **Gateway**, nicht auf Knoten.

<div id="pairing-status">
  ## Kopplung + Status
</div>

**WS-Knoten verwenden Gerätekopplung.** Knoten präsentieren beim `connect` eine Geräteidentität; das Gateway
erstellt eine Kopplungsanforderung für `role: node`. Genehmige sie über die Geräte-CLI (oder UI).

Schnelle CLI-Befehle:

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
```

Hinweise:

* `nodes status` markiert einen Knoten als **paired**, wenn seine Geräte-Kopplungsrolle `node` umfasst.
* `node.pair.*` (CLI: `openclaw nodes pending/approve/reject`) ist ein separater, vom Gateway verwalteter
  Kopplungs-Store für Knoten; er steuert **nicht** den WS-`connect`-Handshake.

<div id="remote-node-host-systemrun">
  ## Entfernter Knoten-Host (system.run)
</div>

Verwende einen **Knoten-Host**, wenn dein Gateway auf einem Rechner läuft und du Befehle
auf einem anderen ausführen möchtest. Das Modell spricht weiterhin mit dem **Gateway**; das Gateway
leitet `exec`-Aufrufe an den **Knoten-Host** weiter, wenn `host=node` ausgewählt ist.

<div id="what-runs-where">
  ### Was läuft wo
</div>

* **Gateway-Host**: empfängt Nachrichten, führt das Modell aus, leitet Tool-Aufrufe weiter.
* **Node-Host**: führt `system.run`/`system.which` auf dem Node-Host aus.
* **Freigaben**: werden auf dem Node-Host mithilfe von `~/.openclaw/exec-approvals.json` durchgesetzt.

<div id="start-a-node-host-foreground">
  ### Knoten-Host im Vordergrund starten
</div>

Auf der Knoten-Maschine:

```bash
openclaw node run --host <gateway-host> --port 18789 --display-name "Build Node"
```

<div id="start-a-node-host-service">
  ### Knoten-Host (Dienst) starten
</div>

```bash
openclaw node install --host <gateway-host> --port 18789 --display-name "Build Node"
openclaw node restart
```

<div id="pair-name">
  ### Koppeln + Benennen
</div>

Auf dem Gateway-Host:

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes list
```

Benennungsoptionen:

* `--display-name` bei `openclaw node run` / `openclaw node install` (wird auf dem Knoten in `~/.openclaw/node.json` gespeichert).
* `openclaw nodes rename --node <id|name|ip> --name "Build Node"` (Gateway-seitiger Override).

<div id="allowlist-the-commands">
  ### Befehle zur Allowlist hinzufügen
</div>

Exec-Freigaben gelten **pro Knoten-Host**. Füge die Allowlist-Einträge im Gateway hinzu:

```bash
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/uname"
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/sw_vers"
```

Die Genehmigungen befinden sich auf dem Knoten-Host in `~/.openclaw/exec-approvals.json`.

<div id="point-exec-at-the-node">
  ### Konfiguriere exec für den Knoten
</div>

Standardwerte konfigurieren (Gateway-Konfiguration):

```bash
openclaw config set tools.exec.host node
openclaw config set tools.exec.security allowlist
openclaw config set tools.exec.node "<id-or-name>"
```

Oder pro Sitzung:

```
/exec host=node security=allowlist node=<id-or-name>
```

Sobald diese Einstellung gesetzt ist, wird jeder `exec`-Aufruf mit `host=node` auf dem Knoten-Host ausgeführt (vorbehaltlich der Allowlist und Freigaben des Knotens).

Zugehörige Themen:

* [Node-Host-CLI](/de/cli/node)
* [Exec-Tool](/de/tools/exec)
* [Exec-Freigaben](/de/tools/exec-approvals)

<div id="invoking-commands">
  ## Befehle aufrufen
</div>

Low-level (rohes RPC):

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command canvas.eval --params '{"javaScript":"location.href"}'
```

Es gibt höherstufige Hilfsfunktionen für die gängigen Workflows „dem agent einen MEDIA-Anhang zuweisen“.

<div id="screenshots-canvas-snapshots">
  ## Screenshots (Canvas-Schnappschüsse)
</div>

Wenn der Knoten das Canvas (WebView) anzeigt, gibt `canvas.snapshot` `{ format, base64 }` zurück.

CLI-Helfer (schreibt in eine temporäre Datei und gibt `MEDIA:<path>` aus):

```bash
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format png
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format jpg --max-width 1200 --quality 0.9
```

<div id="canvas-controls">
  ### Canvas-Bedienelemente
</div>

```bash
openclaw nodes canvas present --node <idOrNameOrIp> --target https://example.com
openclaw nodes canvas hide --node <idOrNameOrIp>
openclaw nodes canvas navigate https://example.com --node <idOrNameOrIp>
openclaw nodes canvas eval --node <idOrNameOrIp> --js "document.title"
```

Hinweise:

* `canvas present` akzeptiert URLs oder lokale Dateipfade (`--target`) sowie optionale `--x/--y/--width/--height` für die Positionierung.
* `canvas eval` akzeptiert Inline-JavaScript (`--js`) oder ein Positionsargument.

<div id="a2ui-canvas">
  ### A2UI (Canvas)
</div>

```bash
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --text "Hallo"
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --jsonl ./payload.jsonl
openclaw nodes canvas a2ui reset --node <idOrNameOrIp>
```

Hinweise:

* Es wird nur A2UI v0.8 JSONL unterstützt (v0.9/createSurface wird abgelehnt).

<div id="photos-videos-node-camera">
  ## Fotos + Videos (Knoten-Kamera)
</div>

Fotos (`jpg`):

```bash
openclaw nodes camera list --node <idOrNameOrIp>
openclaw nodes camera snap --node <idOrNameOrIp>            # Standard: beide Ausrichtungen (2 MEDIA-Zeilen)
openclaw nodes camera snap --node <idOrNameOrIp> --facing front
```

Video-Clips (`mp4`):

```bash
openclaw nodes camera clip --node <idOrNameOrIp> --duration 10s
openclaw nodes camera clip --node <idOrNameOrIp> --duration 3000 --no-audio
```

Hinweise:

* Der Node muss im **Vordergrund** ausgeführt werden für `canvas.*` und `camera.*` (Aufrufe im Hintergrund liefern `NODE_BACKGROUND_UNAVAILABLE` zurück).
* Die Clipdauer wird begrenzt (derzeit `<= 60s`), um übergroße Base64-Payloads zu vermeiden.
* Android fordert nach Möglichkeit `CAMERA`-/`RECORD_AUDIO`-Berechtigungen an; verweigerte Berechtigungen schlagen mit `*_PERMISSION_REQUIRED` fehl.

<div id="screen-recordings-nodes">
  ## Bildschirmaufnahmen (Knoten)
</div>

Knoten bieten `screen.record` (MP4) an. Beispiel:

```bash
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10 --no-audio
```

Hinweise:

* `screen.record` erfordert, dass die Knoten-App im Vordergrund ist.
* Android zeigt vor der Aufzeichnung die systemeigene Aufforderung zur Bildschirmaufnahme an.
* Bildschirmaufnahmen sind auf `<= 60s` begrenzt.
* `--no-audio` deaktiviert die Mikrofonaufnahme (unterstützt auf iOS/Android; macOS verwendet die System-Audioaufnahme).
* Verwende `--screen <index>`, um ein Display auszuwählen, wenn mehrere Bildschirme verfügbar sind.

<div id="location-nodes">
  ## Standort (Knoten)
</div>

Knoten stellen `location.get` bereit, wenn der Standortzugriff in den Einstellungen aktiviert ist.

CLI-Helfer:

```bash
openclaw nodes location get --node <idOrNameOrIp>
openclaw nodes location get --node <idOrNameOrIp> --accuracy precise --max-age 15000 --location-timeout 10000
```

Hinweise:

* Die Standortfunktion ist **standardmäßig deaktiviert**.
* „Immer“ erfordert Systemberechtigung; Hintergrundabfragen erfolgen nach Möglichkeit.
* Die Antwort umfasst Breiten-/Längengrad, Genauigkeit (Meter) und Zeitstempel.

<div id="sms-android-nodes">
  ## SMS (Android-Knoten)
</div>

Android-Knoten können `sms.send` bereitstellen, wenn der Benutzer die **SMS**-Berechtigung erteilt hat und das Gerät Telefonie unterstützt.

Low-Level-Aufruf:

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command sms.send --params '{"to":"+15555550123","message":"Hallo von OpenClaw"}'
```

Hinweise:

* Die Berechtigungsabfrage muss auf dem Android-Gerät akzeptiert werden, bevor die Fähigkeit angezeigt wird.
* Reine WLAN-Geräte ohne Telefonie werden `sms.send` nicht anzeigen.

<div id="system-commands-node-host-mac-node">
  ## Systembefehle (Knoten-Host / macOS-Knoten)
</div>

Der macOS-Knoten bietet `system.run`, `system.notify` und `system.execApprovals.get/set` an.
Der Headless-Knoten-Host bietet `system.run`, `system.which` und `system.execApprovals.get/set` an.

Beispiele:

```bash
openclaw nodes run --node <idOrNameOrIp> -- echo "Hello from mac node"
openclaw nodes notify --node <idOrNameOrIp> --title "Ping" --body "Gateway ready"
```

Hinweise:

* `system.run` gibt stdout/stderr/Exitcode im Payload zurück.
* `system.notify` respektiert den Status der Benachrichtigungsberechtigungen in der macOS-App.
* `system.run` unterstützt `--cwd`, `--env KEY=VAL`, `--command-timeout` und `--needs-screen-recording`.
* `system.notify` unterstützt `--priority <passive|active|timeSensitive>` und `--delivery <system|overlay|auto>`.
* macOS-Knoten verwerfen `PATH`-Overrides; headless Knoten-Hosts akzeptieren `PATH` nur, wenn es dem PATH des Knoten-Hosts vorangestellt wird.
* Im macOS-Knoten-Modus wird `system.run` durch Exec-Genehmigungen in der macOS-App abgesichert (Settings → Exec approvals).
  Ask/allowlist/full verhalten sich genauso wie auf dem headless Knoten-Host; abgelehnte Prompts liefern `SYSTEM_RUN_DENIED` zurück.
* Auf dem headless Knoten-Host wird `system.run` durch Exec-Genehmigungen (`~/.openclaw/exec-approvals.json`) abgesichert.

<div id="exec-node-binding">
  ## Exec-Knotenbindung
</div>

Wenn mehrere Knoten verfügbar sind, kannst du `exec` an einen bestimmten Knoten binden.
Dadurch wird der Standardknoten für `exec host=node` festgelegt (und kann pro agent überschrieben werden).

Globaler Standard:

```bash
openclaw config set tools.exec.node "node-id-or-name"
```

Überschreibung pro Agent:

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

Freilassen, um alle Knoten zuzulassen:

```bash
openclaw config unset tools.exec.node
openclaw config unset agents.list[0].tools.exec.node
```

<div id="permissions-map">
  ## Berechtigungs-Map
</div>

Knoten können in `node.list` / `node.describe` eine `permissions`-Map enthalten, die Berechtigungsnamen (z. B. `screenRecording`, `accessibility`) als Schlüssel und boolesche Werte (`true` = gewährt) enthält.

<div id="headless-node-host-cross-platform">
  ## Headless-Knoten-Host (plattformübergreifend)
</div>

OpenClaw kann einen **Headless-Knoten-Host** (ohne UI) ausführen, der sich mit dem Gateway-WS verbindet und `system.run` / `system.which` zur Verfügung stellt. Dies ist nützlich unter Linux oder Windows oder um einen minimalen Knoten neben einem Server auszuführen.

So startest du ihn:

```bash
openclaw node run --host <gateway-host> --port 18789
```

Hinweise:

* Die Kopplung ist weiterhin erforderlich (das Gateway zeigt eine Genehmigungsaufforderung für den Knoten an).
* Der Knoten-Host speichert seine Node-ID, sein Token, seinen Anzeigenamen und die Gateway-Verbindungsinformationen in `~/.openclaw/node.json`.
* Exec-Genehmigungen werden lokal über `~/.openclaw/exec-approvals.json` durchgesetzt
  (siehe [Exec approvals](/de/tools/exec-approvals)).
* Unter macOS bevorzugt der headless Knoten-Host den Companion‑App‑Exec‑Host, wenn dieser erreichbar ist, und fällt auf lokale Ausführung zurück, wenn die App nicht verfügbar ist. Setze `OPENCLAW_NODE_EXEC_HOST=app`, um die App zu erzwingen, oder `OPENCLAW_NODE_EXEC_FALLBACK=0`, um das Fallback zu deaktivieren.
* Füge `--tls` / `--tls-fingerprint` hinzu, wenn das Gateway-WS TLS verwendet.

<div id="mac-node-mode">
  ## Mac-Knotenmodus
</div>

* Die macOS-Menüleisten-App verbindet sich als Knoten mit dem Gateway-WS-Server (sodass `openclaw nodes …` gegen diesen Mac ausgeführt werden kann).
* Im Remote-Modus stellt die App einen SSH-Tunnel zum Gateway-Port her und verbindet sich mit `localhost`.