---
title: Chrome-Erweiterung
summary: "Chrome-Erweiterung: Lass OpenClaw deinen vorhandenen Chrome-Tab steuern"
read_when:
  - Du möchtest, dass der Agent einen vorhandenen Chrome-Tab steuert (Schaltfläche in der Symbolleiste)
  - Du benötigst ein entferntes Gateway + lokale Browser-Automatisierung über Tailscale
  - Du möchtest die Sicherheitsauswirkungen der Übernahme des Browsers verstehen
---

<div id="chrome-extension-browser-relay">
  # Chrome-Erweiterung (Browser-Relay)
</div>

Die OpenClaw-Chrome-Erweiterung ermöglicht es dem Agent, deine **vorhandenen Chrome-Tabs** (dein normales Chrome-Fenster) zu steuern, anstatt ein separates, von openclaw verwaltetes Chrome-Profil zu starten.

Das Verbinden und Trennen erfolgt über eine **einzige Schaltfläche in der Chrome-Symbolleiste**.

<div id="what-it-is-concept">
  ## Was es ist (Konzept)
</div>

Es gibt drei Komponenten:

* **Browser-Steuerungsdienst** (Gateway oder Knoten): die API, die der Agent bzw. das Tool (über das Gateway) aufruft
* **Lokaler Relay-Server** (Loopback-CDP): vermittelt zwischen dem Steuerungsdienst und der Erweiterung (standardmäßig `http://127.0.0.1:18792`)
* **Chrome-MV3-Erweiterung**: hängt sich mit `chrome.debugger` an den aktiven Tab und leitet CDP-Nachrichten an den Relay-Server weiter

OpenClaw steuert den angebundenen Tab dann über die normale `browser`-Tool-Oberfläche (mit dem richtigen Profil).

<div id="install-load-unpacked">
  ## Installieren / Laden (entpackt)
</div>

1. Installiere die Erweiterung in einem festen lokalen Pfad:

```bash
openclaw browser extension install
```

2. Gib den Pfad des Installationsverzeichnisses der Erweiterung aus:

```bash
openclaw browser extension path
```

3. Chrome → `chrome://extensions`

* „Entwicklermodus“ aktivieren
* „Entpackte Erweiterung laden“ → Wähle das oben ausgegebene Verzeichnis aus

4. Erweiterung an die Symbolleiste anheften.

<div id="updates-no-build-step">
  ## Aktualisierungen (kein Build-Schritt erforderlich)
</div>

Die Erweiterung wird als statische Dateien im OpenClaw-Release (npm-Paket) ausgeliefert. Es gibt keinen separaten „Build“-Schritt.

Nach dem Aktualisieren von OpenClaw:

* Führe `openclaw browser extension install` erneut aus, um die installierten Dateien in deinem OpenClaw-State-Verzeichnis zu aktualisieren.
* Chrome → `chrome://extensions` → klicke bei der Erweiterung auf „Neu laden“.

<div id="use-it-no-extra-config">
  ## Verwendung (keine zusätzliche Konfiguration erforderlich)
</div>

OpenClaw wird mit einem integrierten Browserprofil namens `chrome` ausgeliefert, das den Extension-Relay auf dem Standard-Port anspricht.

Verwendung:

* CLI: `openclaw browser --browser-profile chrome tabs`
* Agent-Tool: `browser` mit `profile="chrome"`

Wenn du einen anderen Namen oder einen anderen Relay-Port verwenden möchtest, lege ein eigenes Profil an:

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

<div id="attach-detach-toolbar-button">
  ## Anhängen / Trennen (Symbolleisten-Schaltfläche)
</div>

* Öffne den Tab, den OpenClaw steuern soll.
* Klicke auf das Erweiterungssymbol.
  * Das Badge zeigt `ON`, wenn verbunden.
* Klicke erneut, um die Verbindung zu lösen.

<div id="which-tab-does-it-control">
  ## Welchen Tab steuert sie?
</div>

* Sie steuert **nicht** automatisch „den Tab, den du gerade anschaust“.
* Sie steuert **nur den oder die Tabs, die du explizit zugeordnet hast**, indem du auf die Schaltfläche in der Symbolleiste klickst.
* Zum Wechseln: Wechsle in den anderen Tab und klicke dort auf das Erweiterungssymbol.

<div id="badge-common-errors">
  ## Badge + häufige Fehler
</div>

* `ON`: angebunden; OpenClaw kann diesen Tab steuern.
* `…`: Verbindung zum lokalen Relay wird aufgebaut.
* `!`: Relay nicht erreichbar (am häufigsten: Browser-Relay-Server läuft nicht auf dieser Maschine).

Wenn du `!` siehst:

* Stelle sicher, dass das Gateway lokal läuft (Standardsetup), oder starte einen Knoten-Host auf dieser Maschine, wenn das Gateway woanders läuft.
* Öffne die Optionsseite der Erweiterung; sie zeigt an, ob das Relay erreichbar ist.

<div id="remote-gateway-use-a-node-host">
  ## Remote-Gateway (Knotenhost verwenden)
</div>

<div id="local-gateway-same-machine-as-chrome-usually-no-extra-steps">
  ### Lokaler Gateway (derselbe Rechner wie Chrome) — in der Regel **keine zusätzlichen Schritte**
</div>

Wenn der Gateway auf derselben Maschine wie Chrome läuft, startet er den Browser-Control-Service auf der Loopback-Schnittstelle
und startet den Relay-Server automatisch. Die Erweiterung kommuniziert mit dem lokalen Relay-Server; die CLI/Tool-Aufrufe gehen an den Gateway.

<div id="remote-gateway-gateway-runs-elsewhere-run-a-node-host">
  ### Remote Gateway (Gateway läuft an anderer Stelle) — **einen Knoten-Host starten**
</div>

Wenn dein Gateway auf einem anderen Rechner läuft, starte einen Knoten-Host auf dem Rechner, auf dem Chrome läuft.
Das Gateway leitet Browser-Aktionen an diesen Knoten weiter; Erweiterung und Relay bleiben lokal auf der Browser-Maschine.

Wenn mehrere Knoten verbunden sind, lege einen mit `gateway.nodes.browser.node` fest oder setze `gateway.nodes.browser.mode`.

<div id="sandboxing-tool-containers">
  ## Sandboxing (Tool-Container)
</div>

Wenn deine Agent-Sitzung in einer sandbox läuft (`agents.defaults.sandbox.mode != "off"`), kann das `browser`-Tool eingeschränkt sein:

* Standardmäßig verwenden sandboxed Sitzungen häufig den **Sandbox-Browser** (`target="sandbox"`) und nicht dein Host-Chrome.
* Eine Chrome-Extension-Relay-Übernahme erfordert die Kontrolle über den **Host**-Browser-Control-Server.

Optionen:

* Am einfachsten: Verwende die Erweiterung aus einer **nicht-sandboxed** Sitzung bzw. von einem Agent.
* Oder erlaube Host-Browser-Kontrolle für sandboxed Sitzungen:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: {
          allowHostControl: true
        }
      }
    }
  }
}
```

Stelle anschließend sicher, dass das Tool nicht durch die Tool-Policy untersagt ist und rufe (falls erforderlich) `browser` mit `target="host"` auf.

Debugging: `openclaw sandbox explain`

<div id="remote-access-tips">
  ## Tipps für den Fernzugriff
</div>

* Halte Gateway und Knoten-Host im selben Tailnet; vermeide es, Relay-Ports ins LAN oder ins öffentliche Internet freizugeben.
* Kopple Knoten bewusst; deaktiviere Browser-Proxy-Routing, wenn du keine Fernsteuerung möchtest (`gateway.nodes.browser.mode="off"`).

<div id="how-extension-path-works">
  ## Funktionsweise des „Extension-Pfads“
</div>

`openclaw browser extension path` gibt das **installierte** Verzeichnis auf dem Datenträger aus, das die Erweiterungsdateien enthält.

Die CLI gibt absichtlich **keinen** `node_modules`-Pfad aus. Führe immer zuerst `openclaw browser extension install` aus, um die Erweiterung an einen stabilen Ort im OpenClaw-State-Verzeichnis zu kopieren.

Wenn du dieses Installationsverzeichnis verschiebst oder löschst, markiert Chrome die Erweiterung als beschädigt, bis du sie erneut von einem gültigen Pfad lädst.

<div id="security-implications-read-this">
  ## Sicherheitsimplikationen (unbedingt lesen)
</div>

Das ist mächtig und riskant. Behandle es so, als würdest du dem Modell direkten Zugriff auf deinen Browser geben.

* Die Erweiterung verwendet die Chrome-Debugger-API (`chrome.debugger`). Wenn sie verbunden ist, kann das Modell:
  * in diesem Tab klicken/tippen/navigieren
  * Seiteninhalte lesen
  * auf alles zugreifen, worauf die angemeldete Sitzung dieses Tabs zugreifen kann
* **Das ist nicht isoliert** wie das dedizierte, von openclaw verwaltete Profil.
  * Wenn du dich mit deinem „Daily-Driver“-Profil/Tab verbindest, gewährst du Zugriff auf den Zustand dieses Kontos.

Empfehlungen:

* Verwende bevorzugt ein eigenes Chrome-Profil (getrennt von deinem persönlichen Browsing) für die Nutzung des Extension-Relays.
* Halte den Gateway und alle Knoten-Hosts nur im Tailnet erreichbar; verlasse dich auf Gateway-Authentifizierung + Knoten-Kopplung.
* Vermeide es, Relay-Ports über das LAN (`0.0.0.0`) freizugeben, und vermeide Funnel (öffentlich).

Verwandt:

* Übersicht zum Browser-Tool: [Browser](/de/tools/browser)
* Sicherheitsprüfung: [Security](/de/gateway/security)
* Tailscale-Einrichtung: [Tailscale](/de/gateway/tailscale)