---
title: Browser
summary: "CLI-Referenz für `openclaw browser` (Profile, Tabs, Aktionen, Chrome-Extension-Relay)"
read_when:
  - Sie verwenden `openclaw browser` und benötigen Beispiele für typische Aufgaben
  - Sie möchten einen Browser steuern, der auf einem anderen Rechner über einen Knoten-Host ausgeführt wird
  - Sie möchten das Chrome-Extension-Relay verwenden (An-/Abkoppeln über die Schaltfläche in der Symbolleiste)
---

<div id="openclaw-browser">
  # `openclaw browser`
</div>

Verwalte den Browser-Steuerungsserver von OpenClaw und führe Browser-Aktionen aus (Tabs, Snapshots, Screenshots, Navigation, Klicks, Tippen).

Verwandte Themen:

* Browser-Tool + API: [Browser-Tool](/de/tools/browser)
* Weiterleitung per Chrome-Erweiterung: [Chrome-Erweiterung](/de/tools/chrome-extension)

<div id="common-flags">
  ## Allgemeine Flags
</div>

* `--url <gatewayWsUrl>`: Gateway-WebSocket-URL (Standardwert laut Konfiguration).
* `--token <token>`: Gateway-Token (falls erforderlich).
* `--timeout <ms>`: Anfrage-Timeout (ms).
* `--browser-profile <name>`: Browser-Profil auswählen (Standardprofil laut Konfiguration).
* `--json`: maschinenlesbare Ausgabe (falls unterstützt).

<div id="quick-start-local">
  ## Schnellstart (lokal)
</div>

```bash
openclaw browser --browser-profile chrome tabs
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

<div id="profiles">
  ## Profile
</div>

Profile sind benannte Browser-Routing-Konfigurationen. In der Praxis:

* `openclaw`: startet bzw. verbindet sich mit einer dedizierten, von OpenClaw verwalteten Chrome-Instanz (mit isoliertem Benutzerdatenverzeichnis).
* `chrome`: steuert deine vorhandenen Chrome-Tabs über das Chrome-Erweiterungs-Relay.

```bash
openclaw browser profiles
openclaw browser create-profile --name work --color "#FF5A36"
openclaw browser delete-profile --name work
```

Ein bestimmtes Profil verwenden:

```bash
openclaw browser --browser-profile work tabs
```

<div id="tabs">
  ## Tabs
</div>

```bash
openclaw browser tabs
openclaw browser open https://docs.openclaw.ai
openclaw browser focus <targetId>
openclaw browser close <targetId>
```

<div id="snapshot-screenshot-actions">
  ## Snapshot / Screenshot / Aktionen
</div>

Schnappschuss:

```bash
openclaw browser snapshot
```

Screenshot:

```bash
openclaw browser screenshot
```

Navigieren/Klicken/Eingeben (ref-basierte UI-Automatisierung):

```bash
openclaw browser navigate https://example.com
openclaw browser click <ref>
openclaw browser type <ref> "hello"
```

<div id="chrome-extension-relay-attach-via-toolbar-button">
  ## Chrome-Erweiterungs-Relay (Verknüpfung über Symbolleistenschaltfläche)
</div>

In diesem Modus kann der agent einen bestehenden Chrome-Tab steuern, den du manuell verknüpfst (es erfolgt keine automatische Verknüpfung).

Installiere die entpackte Erweiterung in einem festen Pfad:

```bash
openclaw browser extension install
openclaw browser extension path
```

Dann in Chrome → `chrome://extensions` → „Entwicklermodus“ aktivieren → „Entpackte Erweiterung laden“ → den ausgegebenen Ordner auswählen.

Vollständige Anleitung: [Chrome-Erweiterung](/de/tools/chrome-extension)

<div id="remote-browser-control-node-host-proxy">
  ## Remote-Browser-Steuerung (Knoten-Host-Proxy)
</div>

Wenn das Gateway auf einem anderen Rechner als der Browser läuft, führe auf dem Rechner mit Chrome/Brave/Edge/Chromium einen **Knoten-Host** aus. Das Gateway leitet Browser-Aktionen an diesen Knoten weiter (kein separater Browser-Steuerungsserver erforderlich).

Verwende `gateway.nodes.browser.mode`, um das automatische Routing zu steuern, und `gateway.nodes.browser.node`, um einen bestimmten Knoten festzulegen, wenn mehrere verbunden sind.

Sicherheit + Remote-Einrichtung: [Browser-Tool](/de/tools/browser), [Remotezugriff](/de/gateway/remote), [Tailscale](/de/gateway/tailscale), [Sicherheit](/de/gateway/security)