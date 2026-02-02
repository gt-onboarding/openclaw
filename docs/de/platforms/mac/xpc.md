---
title: Xpc
summary: "macOS-IPC-Architektur für die OpenClaw-App, den Gateway-Knoten-Transport und PeekabooBridge"
read_when:
  - Bearbeiten von IPC-Verträgen oder Menüleisten-App-IPC
---

<div id="openclaw-macos-ipc-architecture">
  # OpenClaw macOS IPC-Architektur
</div>

**Aktuelles Modell:** Ein lokaler Unix-Socket verbindet den **node host service** mit der **macOS-App** für Ausführungsfreigaben und `system.run`. Eine `openclaw-mac` Debug-CLI besteht für Discovery- und Verbindungsprüfungen; Agent-Aktionen laufen weiterhin über den Gateway-WebSocket und `node.invoke`. Für die UI-Automatisierung wird PeekabooBridge verwendet.

<div id="goals">
  ## Ziele
</div>

* Eine einzige GUI-App-Instanz, die sämtliche TCC-relevanten Aufgaben übernimmt (Benachrichtigungen, Bildschirmaufnahme, Mikrofon, Sprache, AppleScript).
* Eine schlanke Schnittstelle für Automatisierung: Gateway- und Knoten-Befehle sowie PeekabooBridge für UI-Automatisierung.
* Vorhersehbare Berechtigungen: stets dieselbe signierte Bundle-ID, von launchd gestartet, sodass TCC-Freigaben erhalten bleiben.

<div id="how-it-works">
  ## So funktioniert es
</div>

<div id="gateway-node-transport">
  ### Gateway- und Knoten-Transport
</div>

* Die App führt das Gateway (im lokalen Modus) aus und verbindet sich als Knoten damit.
* Agent-Aktionen werden über `node.invoke` ausgeführt (z. B. `system.run`, `system.notify`, `canvas.*`).

<div id="node-service-app-ipc">
  ### Knoten-Dienst + App-IPC
</div>

* Ein headless Knoten-Host-Service verbindet sich mit dem Gateway-WebSocket.
* `system.run`-Anfragen werden über einen lokalen Unix-Socket an die macOS-App weitergeleitet.
* Die App führt das Kommando im UI-Kontext aus, fordert bei Bedarf eine Bestätigung an und gibt die Ausgabe zurück.

Diagramm (SCI):

```
Agent -> Gateway -> Node Service (WS)
                      |  IPC (UDS + token + HMAC + TTL)
                      v
                  Mac App (UI + TCC + system.run)
```

<div id="peekaboobridge-ui-automation">
  ### PeekabooBridge (UI-Automatisierung)
</div>

* Die UI-Automatisierung verwendet einen separaten Unix-Socket namens `bridge.sock` und das PeekabooBridge-JSON-Protokoll.
* Reihenfolge der Host-Priorität (clientseitig): Peekaboo.app → Claude.app → OpenClaw.app → lokale Ausführung.
* Sicherheit: Bridge-Hosts erfordern eine zugelassene TeamID; die DEBUG-only-Escape-Hatch für denselben UID-Benutzer wird durch `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` (Peekaboo-Konvention) abgesichert.
* Siehe: [Verwendung von PeekabooBridge](/de/platforms/mac/peekaboo) für Details.

<div id="operational-flows">
  ## Betriebsabläufe
</div>

* Neustart/Neu-Build: `SIGN_IDENTITY="Apple Development: <Developer Name> (<TEAMID>)" scripts/restart-mac.sh`
  * Beendet vorhandene Instanzen
  * Swift-Build + Packaging
  * Schreibt/initialisiert/startet den LaunchAgent
* Einzelinstanz: Die App beendet sich frühzeitig, wenn bereits eine andere Instanz mit derselben Bundle-ID läuft.

<div id="hardening-notes">
  ## Härtungshinweise
</div>

* Nach Möglichkeit eine TeamID-Übereinstimmung für alle privilegierten Oberflächen verlangen.
* PeekabooBridge: `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` (nur für DEBUG) kann Aufrufer mit derselbe UID für die lokale Entwicklung zulassen.
* Sämtliche Kommunikation bleibt ausschließlich lokal; es werden keine Netzwerksockets geöffnet.
* TCC-Aufforderungen stammen nur aus dem GUI-App-Bundle; halte die signierte Bundle-ID über Rebuilds hinweg stabil.
* IPC-Härtung: Socket-Modus `0600`, Token, Peer-UID-Prüfungen, HMAC-Challenge/Response, kurze TTL.