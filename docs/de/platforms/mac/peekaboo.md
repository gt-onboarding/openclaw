---
title: Peekaboo
summary: "PeekabooBridge-Integration für UI-Automatisierung unter macOS"
read_when:
  - Hosting von PeekabooBridge in OpenClaw.app
  - Integration von Peekaboo über den Swift Package Manager
  - Ändern des PeekabooBridge-Protokolls bzw. der Pfade
---

<div id="peekaboo-bridge-macos-ui-automation">
  # Peekaboo Bridge (macOS-UI-Automatisierung)
</div>

OpenClaw kann **PeekabooBridge** als lokalen, berechtigungsbewussten UI‑Automatisierungsbroker bereitstellen. Dadurch kann die `peekaboo` CLI UI‑Automatisierung steuern und dabei die TCC-Berechtigungen der macOS-App mitbenutzen.

<div id="what-this-is-and-isnt">
  ## Was das ist (und was nicht)
</div>

- **Host**: OpenClaw.app kann als PeekabooBridge-Host dienen.
- **Client**: Verwenden Sie die `peekaboo` CLI (keine separate `openclaw ui ...`-Oberfläche).
- **UI**: Visuelle Overlays bleiben in Peekaboo.app; OpenClaw fungiert nur als schlanker Broker-Host.

<div id="enable-the-bridge">
  ## Bridge aktivieren
</div>

In der macOS-App:

- Settings → **Enable Peekaboo Bridge**

Wenn diese Option aktiviert ist, startet OpenClaw einen lokalen UNIX-Socket-Server. Wenn sie deaktiviert ist, wird der Host beendet und `peekaboo` greift auf andere verfügbare Hosts zurück.

<div id="client-discovery-order">
  ## Reihenfolge der Client-Erkennung
</div>

Peekaboo-Clients probieren Hosts typischerweise in dieser Reihenfolge aus:

1. Peekaboo.app (vollständige UX)
2. Claude.app (falls installiert)
3. OpenClaw.app (schlanker Broker)

Verwende `peekaboo bridge status --verbose`, um zu sehen, welcher Host aktiv ist
und welcher Socket-Pfad verwendet wird. Du kannst dies mit folgendem Befehl überschreiben:

```bash
export PEEKABOO_BRIDGE_SOCKET=/path/to/bridge.sock
```


<div id="security-permissions">
  ## Sicherheit & Berechtigungen
</div>

- Die Bridge validiert **Code-Signaturen des Aufrufers**; sie erzwingt eine Allowlist von Team-IDs
  (Peekaboo-Host-Team-ID + OpenClaw-App-Team-ID).
- Anfragen laufen nach etwa 10 Sekunden in einen Timeout.
- Wenn erforderliche Berechtigungen fehlen, gibt die Bridge eine eindeutige Fehlermeldung
  zurück, anstatt die Systemeinstellungen zu öffnen.

<div id="snapshot-behavior-automation">
  ## Snapshot-Verhalten (Automatisierung)
</div>

Snapshots werden im Speicher abgelegt und verfallen nach einem kurzen Zeitraum automatisch.
Wenn du eine längere Aufbewahrungsdauer brauchst, erfasse sie erneut vom Client aus.

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

- Wenn `peekaboo` „bridge client is not authorized“ meldet, stelle sicher, dass der Client
  korrekt signiert ist oder starte den Host mit `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1`
  ausschließlich im **Debug**-Modus.
- Wenn keine Hosts gefunden werden, öffne eine der Host-Apps (Peekaboo.app oder OpenClaw.app)
  und prüfe, ob die erforderlichen Berechtigungen erteilt wurden.