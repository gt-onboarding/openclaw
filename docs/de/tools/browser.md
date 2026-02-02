---
title: Browser
summary: "Integrierter Dienst zur Browser-Steuerung + Aktionsbefehle"
read_when:
  - Hinzufügen von agentgesteuerter Browser-Automatisierung
  - Debuggen, warum openclaw in deinen eigenen Chrome-Browser eingreift
  - Implementieren von Browser-Einstellungen und Lebenszyklus in der macOS-App
---

<div id="browser-openclaw-managed">
  # Browser (von openclaw verwaltet)
</div>

OpenClaw kann ein **dediziertes Chrome/Brave/Edge/Chromium-Profil** betreiben, das vom Agent gesteuert wird.
Es ist von deinem persönlichen Browser isoliert und wird über einen kleinen lokalen
Steuerdienst innerhalb des Gateway verwaltet (nur Loopback).

Für Einsteiger:

* Stell es dir als einen **separaten Browser nur für den Agent** vor.
* Das `openclaw`-Profil berührt **nicht** dein persönliches Browserprofil.
* Der Agent kann **Tabs öffnen, Seiten lesen, klicken und tippen** – in einer sicheren Umgebung.
* Das Standardprofil `chrome` verwendet den **systemweiten Standard-Chromium-Browser** über das
  Erweiterungs-Relay; wechsle zu `openclaw` für den isolierten, verwalteten Browser.

<div id="what-you-get">
  ## Was du bekommst
</div>

* Ein separates Browser-Profil namens **openclaw** (standardmäßig mit oranger Akzentfarbe).
* Deterministische Steuerung der Tabs (auflisten/öffnen/fokussieren/schließen).
* Agent-Aktionen (klicken/tippen/ziehen/auswählen), Snapshots, Screenshots, PDFs.
* Optionale Multi-Profil-Unterstützung (`openclaw`, `work`, `remote`, ...).

Dieser Browser ist **nicht** dein Alltagsbrowser. Er ist eine sichere, isolierte Umgebung für Agent-Automatisierung und -Verifikation.

<div id="quick-start">
  ## Schnellstart
</div>

```bash
openclaw browser --browser-profile openclaw status
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

Wenn die Meldung „Browser disabled“ erscheint, aktiviere ihn in der Konfiguration (siehe unten) und starte das Gateway neu.

<div id="profiles-openclaw-vs-chrome">
  ## Profile: `openclaw` vs `chrome`
</div>

* `openclaw`: verwalteter, isolierter Browser (keine Erweiterung erforderlich).
* `chrome`: Weiterleitung über die Erweiterung an deinen **Systembrowser** (erfordert, dass die OpenClaw-Erweiterung in einem Tab aktiv ist).

Setze `browser.defaultProfile: "openclaw"`, wenn du standardmäßig den verwalteten Modus verwenden möchtest.

<div id="configuration">
  ## Konfiguration
</div>

Browser-Einstellungen werden in `~/.openclaw/openclaw.json` gespeichert.

```json5
{
  browser: {
    enabled: true,                    // Standard: true
    // cdpUrl: "http://127.0.0.1:18792", // veraltete Einzelprofil-Überschreibung
    remoteCdpTimeoutMs: 1500,         // Remote-CDP-HTTP-Timeout (ms)
    remoteCdpHandshakeTimeoutMs: 3000, // Remote-CDP-WebSocket-Handshake-Timeout (ms)
    defaultProfile: "chrome",
    color: "#FF4500",
    headless: false,
    noSandbox: false,
    attachOnly: false,
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" }
    }
  }
}
```

Hinweise:

* Der Browser-Steuerdienst bindet an Loopback auf einem Port, der von `gateway.port` abgeleitet wird
  (Standard: `18791`, also Gateway-Port + 2). Das Relay verwendet den nächsten Port (`18792`).
* Wenn du den Gateway-Port änderst (`gateway.port` oder `OPENCLAW_GATEWAY_PORT`),
  verschieben sich die abgeleiteten Browser-Ports, sodass sie in derselben „Familie“ bleiben.
* `cdpUrl` verwendet standardmäßig den Relay-Port, wenn nicht gesetzt.
* `remoteCdpTimeoutMs` gilt für Remote-CDP-Erreichbarkeitsprüfungen (nicht-Loopback).
* `remoteCdpHandshakeTimeoutMs` gilt für Remote-CDP-WebSocket-Erreichbarkeitsprüfungen.
* `attachOnly: true` bedeutet „niemals einen lokalen Browser starten; nur verbinden, wenn er bereits läuft.“
* `color` + profilbezogenes `color` färben die Browser-UI ein, damit du sehen kannst, welches Profil aktiv ist.
* Das Standardprofil ist `chrome` (Erweiterungs-Relay). Verwende `defaultProfile: "openclaw"` für den verwalteten Browser.
* Reihenfolge der automatischen Erkennung: systemweiter Standardbrowser, falls Chromium-basiert; andernfalls Chrome → Brave → Edge → Chromium → Chrome Canary.
* Lokale `openclaw`-Profile weisen `cdpPort`/`cdpUrl` automatisch zu — setze diese nur für Remote-CDP.

<div id="use-brave-or-another-chromium-based-browser">
  ## Brave (oder einen anderen Chromium-basierten Browser) verwenden
</div>

Wenn dein **System-Standardbrowser** Chromium-basiert ist (Chrome/Brave/Edge usw.),
verwendet OpenClaw ihn automatisch. Setze `browser.executablePath`, um die
automatische Erkennung zu überschreiben:

CLI-Beispiel:

```bash
openclaw config set browser.executablePath "/usr/bin/google-chrome"
```

```json5
// macOS
{
  browser: {
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser"
  }
}

// Windows
{
  browser: {
    executablePath: "C:\\Program Files\\BraveSoftware\\Brave-Browser\\Application\\brave.exe"
  }
}

// Linux
{
  browser: {
    executablePath: "/usr/bin/brave-browser"
  }
}
```

<div id="local-vs-remote-control">
  ## Lokale vs. Remote-Steuerung
</div>

* **Lokale Steuerung (Standard):** das Gateway startet den Loopback-Control-Service und kann einen lokalen Browser starten.
* **Remote-Steuerung (Knoten-Host):** führe einen Knoten-Host auf der Maschine aus, auf der sich der Browser befindet; das Gateway leitet Browser-Aktionen an ihn weiter.
* **Remote-CDP:** setze `browser.profiles.<name>.cdpUrl` (oder `browser.cdpUrl`), um
  eine Verbindung zu einem entfernten Chromium-basierten Browser herzustellen. In diesem Fall startet OpenClaw keinen lokalen Browser.

Remote-CDP-URLs können Authentifizierung enthalten:

* Query-Token (z. B. `https://provider.example?token=<token>`)
* HTTP-Basic-Auth (z. B. `https://user:pass@provider.example`)

OpenClaw übernimmt die Authentifizierung bei Aufrufen von `/json/*`-Endpoints und bei der Verbindung
zum CDP-WebSocket. Verwende vorzugsweise Umgebungsvariablen oder Secrets-Manager für Token, anstatt sie in Konfigurationsdateien zu speichern.

<div id="node-browser-proxy-zero-config-default">
  ## Node-Browser-Proxy (Zero-Config-Standard)
</div>

Wenn Sie einen **Knoten-Host** auf der Maschine mit Ihrem Browser betreiben, kann OpenClaw
Browser-Toolaufrufe automatisch ohne zusätzliche Browserkonfiguration zu diesem Knoten weiterleiten.
Dies ist der Standardpfad für entfernte Gateways.

Hinweise:

* Der Knoten-Host stellt seinen lokalen Browser-Steuerungsserver über einen **Proxy-Befehl** bereit.
* Profile stammen aus der eigenen `browser.profiles`-Konfiguration des Knotens (entspricht der lokalen Konfiguration).
* Deaktivieren Sie dies, wenn Sie es nicht verwenden möchten:
  * Auf dem Knoten: `nodeHost.browserProxy.enabled=false`
  * Auf dem Gateway: `gateway.nodes.browser.mode="off"`

<div id="browserless-hosted-remote-cdp">
  ## Browserless (gehostetes Remote-CDP)
</div>

[Browserless](https://browserless.io) ist ein gehosteter Chromium-Dienst, der
CDP-Endpunkte über HTTPS bereitstellt. Du kannst ein OpenClaw-Browserprofil so konfigurieren,
dass es auf einen Browserless-Regionsendpunkt zeigt, und dich mit deinem API-Schlüssel authentifizieren.

Beispiel:

```json5
{
  browser: {
    enabled: true,
    defaultProfile: "browserless",
    remoteCdpTimeoutMs: 2000,
    remoteCdpHandshakeTimeoutMs: 4000,
    profiles: {
      browserless: {
        cdpUrl: "https://production-sfo.browserless.io?token=<BROWSERLESS_API_KEY>",
        color: "#00AA00"
      }
    }
  }
}
```

Hinweise:

* Ersetze `<BROWSERLESS_API_KEY>` durch deinen gültigen Browserless-Token.
* Wähle den Endpunkt für die Region, der zu deinem Browserless-Konto passt (siehe deren Dokumentation).

<div id="security">
  ## Sicherheit
</div>

Zentrale Punkte:

* Browser-Steuerung ist nur über Loopback verfügbar; der Zugriff erfolgt über die Authentifizierung des Gateways oder die Knoten-Kopplung.
* Halte das Gateway und alle Knoten-Hosts in einem privaten Netzwerk (Tailscale); vermeide öffentliche Offenlegung.
* Behandle Remote-CDP-URLs/-Token als Secrets; bevorzuge Umgebungsvariablen oder einen Secrets-Manager.

Tipps zu Remote-CDP:

* Bevorzuge HTTPS-Endpunkte und kurzlebige Token, wo immer möglich.
* Vermeide es, langlebige Token direkt in Konfigurationsdateien zu hinterlegen.

<div id="profiles-multi-browser">
  ## Profile (Multi-Browser)
</div>

OpenClaw unterstützt mehrere benannte Profile (Routing-Konfigurationen). Profile können folgende Typen haben:

* **openclaw-managed**: eine dedizierte, Chromium-basierte Browser-Instanz mit eigenem Benutzerdatenverzeichnis + CDP-Port
* **remote**: eine explizite CDP-URL (Chromium-basierter Browser, der an anderer Stelle läuft)
* **extension relay**: deine vorhandenen Chrome-Tabs über das lokale Relay + Chrome-Erweiterung

Standardwerte:

* Das Profil `openclaw` wird automatisch erstellt, falls es nicht existiert.
* Das Profil `chrome` ist für das Chrome-Erweiterungs-Relay vorkonfiguriert (zeigt standardmäßig auf `http://127.0.0.1:18792`).
* Lokale CDP-Ports werden standardmäßig aus dem Bereich **18800–18899** zugewiesen.
* Beim Löschen eines Profils wird sein lokales Datenverzeichnis in den Papierkorb verschoben.

Alle Steuerendpunkte akzeptieren `?profile=<name>`; die CLI verwendet `--browser-profile`.

<div id="chrome-extension-relay-use-your-existing-chrome">
  ## Chrome-Erweiterungs-Relay (verwende dein bestehendes Chrome)
</div>

OpenClaw kann auch **deine bestehenden Chrome-Tabs** steuern (keine separate „openclaw“-Chrome-Instanz), über ein lokales CDP-Relay plus eine Chrome-Erweiterung.

Vollständige Anleitung: [Chrome extension](/de/tools/chrome-extension)

Ablauf:

* Das Gateway läuft lokal (gleicher Rechner), oder ein Node-Host läuft auf dem Browser-Rechner.
* Ein lokaler **Relay-Server** lauscht auf einer Loopback-`cdpUrl` (Standard: `http://127.0.0.1:18792`).
* Du klickst in einem Tab auf das Symbol der **OpenClaw Browser Relay**-Erweiterung, um diesen Tab anzubinden (es erfolgt keine automatische Anbindung).
* Der agent steuert diesen Tab über das normale `browser`-Tool, indem du das passende Profil auswählst.

Wenn das Gateway anderswo läuft, starte einen Node-Host auf dem Browser-Rechner, damit das Gateway Browser-Aktionen als Proxy ausführen kann.

<div id="sandboxed-sessions">
  ### Sandboxed-Sitzungen
</div>

Wenn die Agent-Sitzung in einer Sandbox ausgeführt wird, kann das `browser`-Tool standardmäßig `target="sandbox"` (Sandbox-Browser) verwenden.
Die Übernahme über das Chrome-Extension-Relay erfordert die Steuerung des Host-Browsers, daher entweder:

* die Sitzung ohne Sandbox ausführen oder
* `agents.defaults.sandbox.browser.allowHostControl: true` setzen und `target="host"` beim Aufruf des Tools verwenden.

<div id="setup">
  ### Einrichtung
</div>

1. Laden Sie die Erweiterung (dev/unpacked):

```bash
openclaw browser extension install
```

* Chrome → `chrome://extensions` → „Developer Mode“ aktivieren
* „Entpackte Erweiterung laden“ → wähle das Verzeichnis aus, das von `openclaw browser extension path` ausgegeben wurde
* Pinne die Erweiterung an und klicke dann auf dem Tab, den du steuern möchtest, darauf (Badge zeigt `ON`).

2. Nutzung:

* CLI: `openclaw browser --browser-profile chrome tabs`
* Agent-Tool: `browser` mit `profile="chrome"`

Optional: Wenn du einen anderen Namen oder Relay-Port verwenden möchtest, erstelle dein eigenes Profil:

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

Hinweise:

* Dieser Modus stützt sich für die meisten Vorgänge (Screenshots/Snapshots/Aktionen) auf Playwright-on-CDP.
* Trenne die Verbindung, indem du erneut auf das Symbol der Erweiterung klickst.

<div id="isolation-guarantees">
  ## Isolationsgarantien
</div>

* **Eigenes Benutzerdatenverzeichnis**: greift niemals auf dein persönliches Browserprofil zu.
* **Eigene Ports**: meidet `9222`, um Konflikte mit Dev-Workflows zu vermeiden.
* **Deterministische Tab-Steuerung**: adressiert Tabs über `targetId`, nicht über den „zuletzt aktiven Tab“.

<div id="browser-selection">
  ## Browser-Auswahl
</div>

Beim lokalen Starten wählt OpenClaw automatisch den ersten verfügbaren Browser:

1. Chrome
2. Brave
3. Edge
4. Chromium
5. Chrome Canary

Du kannst dies mit `browser.executablePath` überschreiben.

Plattformen:

* macOS: prüft `/Applications` und `~/Applications`.
* Linux: sucht nach `google-chrome`, `brave`, `microsoft-edge`, `chromium` usw.
* Windows: prüft gängige Standard-Installationspfade.

<div id="control-api-optional">
  ## Control API (optional)
</div>

Ausschließlich für lokale Integrationen stellt der Gateway eine kleine Loopback-HTTP-API bereit:

* Status/Start/Stopp: `GET /`, `POST /start`, `POST /stop`
* Tabs: `GET /tabs`, `POST /tabs/open`, `POST /tabs/focus`, `DELETE /tabs/:targetId`
* Snapshot/Screenshot: `GET /snapshot`, `POST /screenshot`
* Aktionen: `POST /navigate`, `POST /act`
* Hooks: `POST /hooks/file-chooser`, `POST /hooks/dialog`
* Downloads: `POST /download`, `POST /wait/download`
* Debugging: `GET /console`, `POST /pdf`
* Debugging: `GET /errors`, `GET /requests`, `POST /trace/start`, `POST /trace/stop`, `POST /highlight`
* Netzwerk: `POST /response/body`
* Zustand: `GET /cookies`, `POST /cookies/set`, `POST /cookies/clear`
* Zustand: `GET /storage/:kind`, `POST /storage/:kind/set`, `POST /storage/:kind/clear`
* Einstellungen: `POST /set/offline`, `POST /set/headers`, `POST /set/credentials`, `POST /set/geolocation`, `POST /set/media`, `POST /set/timezone`, `POST /set/locale`, `POST /set/device`

Alle Endpunkte akzeptieren `?profile=<name>`.

<div id="playwright-requirement">
  ### Playwright-Voraussetzung
</div>

Einige Funktionen (navigate/act/AI snapshot/role snapshot, Element-Screenshots, PDF) erfordern
Playwright. Wenn Playwright nicht installiert ist, liefern diese Endpunkte einen klaren 501-Fehler.
ARIA-Snapshots und einfache Screenshots funktionieren weiterhin für von openclaw verwaltetes Chrome.
Beim Relay-Treiber für die Chrome-Extension erfordern ARIA-Snapshots und Screenshots Playwright.

Wenn du `Playwright is not available in this gateway build` siehst, installiere das vollständige
Playwright-Paket (nicht `playwright-core`) und starte das Gateway neu oder installiere
OpenClaw mit Browserunterstützung neu.

<div id="how-it-works-internal">
  ## Funktionsweise (intern)
</div>

Ablauf auf hoher Ebene:

* Ein kleiner **Control-Server** akzeptiert HTTP-Anfragen.
* Er verbindet sich über **CDP** mit Chromium-basierten Browsern (Chrome/Brave/Edge/Chromium).
* Für erweiterte Aktionen (Klicken/Tippen/Snapshot/PDF) verwendet er **Playwright** zusätzlich
  zu CDP.
* Wenn Playwright fehlt, stehen nur Aktionen ohne Playwright zur Verfügung.

Dieses Design sorgt dafür, dass der Agent über eine stabile, deterministische Schnittstelle läuft, während du
lokale und Remote-Browser sowie Profile austauschen kannst.

<div id="cli-quick-reference">
  ## CLI-Schnellreferenz
</div>

Alle Befehle akzeptieren `--browser-profile <name>`, um ein bestimmtes Profil anzusprechen.
Alle Befehle unterstützen außerdem `--json` für maschinenlesbare Ausgabe (stabile Payloads).

Grundlagen:

* `openclaw browser status`
* `openclaw browser start`
* `openclaw browser stop`
* `openclaw browser tabs`
* `openclaw browser tab`
* `openclaw browser tab new`
* `openclaw browser tab select 2`
* `openclaw browser tab close 2`
* `openclaw browser open https://example.com`
* `openclaw browser focus abcd1234`
* `openclaw browser close abcd1234`

Inspektion:

* `openclaw browser screenshot`
* `openclaw browser screenshot --full-page`
* `openclaw browser screenshot --ref 12`
* `openclaw browser screenshot --ref e12`
* `openclaw browser snapshot`
* `openclaw browser snapshot --format aria --limit 200`
* `openclaw browser snapshot --interactive --compact --depth 6`
* `openclaw browser snapshot --efficient`
* `openclaw browser snapshot --labels`
* `openclaw browser snapshot --selector "#main" --interactive`
* `openclaw browser snapshot --frame "iframe#main" --interactive`
* `openclaw browser console --level error`
* `openclaw browser errors --clear`
* `openclaw browser requests --filter api --clear`
* `openclaw browser pdf`
* `openclaw browser responsebody "**/api" --max-chars 5000`

Aktionen:

* `openclaw browser navigate https://example.com`
* `openclaw browser resize 1280 720`
* `openclaw browser click 12 --double`
* `openclaw browser click e12 --double`
* `openclaw browser type 23 "hello" --submit`
* `openclaw browser press Enter`
* `openclaw browser hover 44`
* `openclaw browser scrollintoview e12`
* `openclaw browser drag 10 11`
* `openclaw browser select 9 OptionA OptionB`
* `openclaw browser download e12 /tmp/report.pdf`
* `openclaw browser waitfordownload /tmp/report.pdf`
* `openclaw browser upload /tmp/file.pdf`
* `openclaw browser fill --fields '[{"ref":"1","type":"text","value":"Ada"}]'`
* `openclaw browser dialog --accept`
* `openclaw browser wait --text "Done"`
* `openclaw browser wait "#main" --url "**/dash" --load networkidle --fn "window.ready===true"`
* `openclaw browser evaluate --fn '(el) => el.textContent' --ref 7`
* `openclaw browser highlight e12`
* `openclaw browser trace start`
* `openclaw browser trace stop`

Zustand:

* `openclaw browser cookies`
* `openclaw browser cookies set session abc123 --url "https://example.com"`
* `openclaw browser cookies clear`
* `openclaw browser storage local get`
* `openclaw browser storage local set theme dark`
* `openclaw browser storage session clear`
* `openclaw browser set offline on`
* `openclaw browser set headers --json '{"X-Debug":"1"}'`
* `openclaw browser set credentials user pass`
* `openclaw browser set credentials --clear`
* `openclaw browser set geo 37.7749 -122.4194 --origin "https://example.com"`
* `openclaw browser set geo --clear`
* `openclaw browser set media dark`
* `openclaw browser set timezone America/New_York`
* `openclaw browser set locale en-US`
* `openclaw browser set device "iPhone 14"`

Hinweise:

* `upload` und `dialog` sind **arming**-Aufrufe; führe sie vor dem Klick/Tastendruck aus,
  der den Auswahldialog bzw. das Dialogfenster auslöst.
* `upload` kann Datei-Eingabefelder auch direkt über `--input-ref` oder `--element` setzen.
* `snapshot`:
  * `--format ai` (Standard, wenn Playwright installiert ist): gibt einen AI-Snapshot mit numerischen Refs zurück (`aria-ref="<n>"`).
  * `--format aria`: gibt den Accessibility-Baum zurück (keine Refs; nur zur Inspektion).
  * `--efficient` (oder `--mode efficient`): kompaktes Rollen-Snapshot-Preset (interaktiv + kompakt + Tiefe + geringere maxChars).
  * Standard in der Konfiguration (nur Tool/CLI): setze `browser.snapshotDefaults.mode: "efficient"`, um effiziente Snapshots zu verwenden, wenn der Aufrufer keinen Modus übergibt (siehe [Gateway configuration](/de/gateway/configuration#browser-openclaw-managed-browser)).
  * Rollen-Snapshot-Optionen (`--interactive`, `--compact`, `--depth`, `--selector`) erzwingen einen rollenbasierten Snapshot mit Refs wie `ref=e12`.
  * `--frame "<iframe selector>"` begrenzt Rollen-Snapshots auf ein iframe (passend zu Rollen-Refs wie `e12`).
  * `--interactive` gibt eine flache, leicht auswählbare Liste interaktiver Elemente aus (am besten zum Ausführen von Aktionen).
  * `--labels` fügt einen reinen Viewport-Screenshot mit überlagerten Ref-Labels hinzu (gibt `MEDIA:<path>` aus).
* `click`/`type`/etc erfordern eine `ref` aus `snapshot` (entweder numerisch `12` oder Rollen-Ref `e12`).
  CSS-Selektoren werden für Aktionen absichtlich nicht unterstützt.

<div id="snapshots-and-refs">
  ## Snapshots und Refs
</div>

OpenClaw unterstützt zwei „Snapshot“-Stile:

* **AI-Snapshot (numerische Refs)**: `openclaw browser snapshot` (Standard; `--format ai`)
  * Ausgabe: ein Text-Snapshot, der numerische Refs enthält.
  * Aktionen: `openclaw browser click 12`, `openclaw browser type 23 "hello"`.
  * Intern wird die Ref über Playwrights `aria-ref` aufgelöst.

* **Role-Snapshot (Rollen-Refs wie `e12`)**: `openclaw browser snapshot --interactive` (oder `--compact`, `--depth`, `--selector`, `--frame`)
  * Ausgabe: eine rollenbasierte Liste bzw. Baumstruktur mit `[ref=e12]` (und optional `[nth=1]`).
  * Aktionen: `openclaw browser click e12`, `openclaw browser highlight e12`.
  * Intern wird die Ref über `getByRole(...)` aufgelöst (plus `nth()` für Duplikate).
  * Füge `--labels` hinzu, um einen Viewport-Screenshot mit überlagerten `e12`-Labels zu erzeugen.

Ref-Verhalten:

* Refs sind **nicht stabil über Seitenwechsel/Navigationsvorgänge hinweg**; wenn etwas fehlschlägt, führe `snapshot` erneut aus und verwende eine frische Ref.
* Wenn der Role-Snapshot mit `--frame` aufgenommen wurde, sind Rollen-Refs auf dieses `iframe` beschränkt, bis der nächste Role-Snapshot erstellt wird.

<div id="wait-power-ups">
  ## Erweiterte Wartemechanismen
</div>

Du kannst auf mehr warten als nur auf Zeit oder Text:

* Auf eine URL warten (Glob-Muster werden von Playwright unterstützt):
  * `openclaw browser wait --url "**/dash"`
* Auf einen Ladezustand warten:
  * `openclaw browser wait --load networkidle`
* Auf eine JS-Bedingung warten:
  * `openclaw browser wait --fn "window.ready===true"`
* Warten, bis ein Selektor sichtbar wird:
  * `openclaw browser wait "#main"`

Diese können kombiniert werden:

```bash
openclaw browser wait "#main" \
  --url "**/dash" \
  --load networkidle \
  --fn "window.ready===true" \
  --timeout-ms 15000
```

<div id="debug-workflows">
  ## Debug-Workflows
</div>

Wenn eine Aktion fehlschlägt (z. B. „not visible“, „strict mode violation“, „covered“):

1. `openclaw browser snapshot --interactive`
2. Verwende `click <ref>` / `type <ref>` (im interaktiven Modus nach Möglichkeit Role-Refs verwenden)
3. Wenn es immer noch fehlschlägt: `openclaw browser highlight <ref>`, um zu sehen, welches Element Playwright ansteuert
4. Wenn sich die Seite seltsam verhält:
   * `openclaw browser errors --clear`
   * `openclaw browser requests --filter api --clear`
5. Für detailliertes Debugging: Trace aufzeichnen:
   * `openclaw browser trace start`
   * Problem reproduzieren
   * `openclaw browser trace stop` (gibt `TRACE:<path>` aus)

<div id="json-output">
  ## JSON-Ausgabe
</div>

`--json` ist für Skripting und strukturierte Tools gedacht.

Beispiele:

```bash
openclaw browser status --json
openclaw browser snapshot --interactive --json
openclaw browser requests --filter api --json
openclaw browser cookies --json
```

Rollensnapshots im JSON enthalten `refs` sowie einen kleinen `stats`-Block (lines/chars/refs/interactive), damit Tools die Payload-Größe und -dichte einschätzen können.

<div id="state-and-environment-knobs">
  ## Stellschrauben für Zustand und Umgebung
</div>

Diese sind nützlich für Workflows, mit denen du die Website so konfigurieren kannst, dass sie sich wie X verhält:

* Cookies: `cookies`, `cookies set`, `cookies clear`
* Speicher: `storage local|session get|set|clear`
* Offline: `set offline on|off`
* Header: `set headers --json '{"X-Debug":"1"}'` (oder `--clear`)
* HTTP Basic Auth: `set credentials user pass` (oder `--clear`)
* Geolocation: `set geo <lat> <lon> --origin "https://example.com"` (oder `--clear`)
* Medien: `set media dark|light|no-preference|none`
* Zeitzone / Gebietsschema: `set timezone ...`, `set locale ...`
* Gerät / Viewport:
  * `set device "iPhone 14"` (Playwright-Gerätevorgaben)
  * `set viewport 1280 720`

<div id="security-privacy">
  ## Sicherheit &amp; Datenschutz
</div>

* Das openclaw-Browser-Profil kann angemeldete Sitzungen enthalten; behandle es als sensibel.
* `browser act kind=evaluate` / `openclaw browser evaluate` und `wait --fn`
  führen beliebigen JavaScript-Code im Seitenkontext aus. Prompt-Injection kann dies steuern.
  Deaktiviere es mit `browser.evaluateEnabled=false`, wenn du es nicht benötigst.
* Für Logins und Anti-Bot-Hinweise (X/Twitter usw.) siehe [Browser login + X/Twitter posting](/de/tools/browser-login).
* Halte den Gateway-/Knoten-Host privat (nur Loopback oder ausschließlich im Tailnet).
* Entfernte CDP-Endpunkte sind sehr mächtig; leite sie nur über gesicherte Tunnel weiter und schütze sie.

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

Bei Linux-spezifischen Problemen (insbesondere mit Chromium im Snap-Paket) siehe
[Browser-Fehlerbehebung](/de/tools/browser-linux-troubleshooting).

<div id="agent-tools-how-control-works">
  ## Agent-Tools + Funktionsweise der Steuerung
</div>

Der agent erhält **ein Tool** für Browser-Automatisierung:

* `browser` — status/start/stop/tabs/open/focus/close/snapshot/screenshot/navigate/act

So wird zugeordnet:

* `browser snapshot` liefert einen stabilen UI-Baum (AI oder ARIA).
* `browser act` nutzt die Snapshot-`ref`-IDs zum Klicken/Tippen/Ziehen/Auswählen.
* `browser screenshot` erfasst Pixel (ganze Seite oder einzelnes Element).
* `browser` akzeptiert:
  * `profile`, um ein benanntes Browser-Profil auszuwählen (openclaw, chrome oder remote CDP).
  * `target` (`sandbox` | `host` | `node`), um auszuwählen, wo der Browser läuft.
  * In Sandbox-Sitzungen erfordert `target: "host"` `agents.defaults.sandbox.browser.allowHostControl=true`.
  * Wenn `target` weggelassen wird: Sandbox-Sitzungen nutzen standardmäßig `sandbox`, Nicht-Sandbox-Sitzungen standardmäßig `host`.
  * Wenn ein browser-fähiger Knoten verbunden ist, kann das Tool automatisch dorthin routen, sofern du `target="host"` oder `target="node"` nicht fest vorgibst.

Dadurch bleibt der agent deterministisch und vermeidet fragile Selektoren.