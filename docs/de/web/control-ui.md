---
title: Control UI
summary: "Browserbasierte Control UI des Gateways (Chat, Knoten, Konfiguration)"
read_when:
  - Du möchtest das Gateway über einen Browser bedienen
  - Du möchtest Tailnet-Zugriff ohne SSH-Tunnel
---

<div id="control-ui-browser">
  # Control UI (browser)
</div>

Die Control UI ist eine kleine **Vite + Lit** Single-Page-App, die vom Gateway bereitgestellt wird:

* Standard: `http://<host>:18789/`
* optionales Präfix: konfiguriere `gateway.controlUi.basePath` (z. B. `/openclaw`)

Sie kommuniziert **direkt über WebSocket mit dem Gateway** auf demselben Port.

<div id="quick-open-local">
  ## Schnell öffnen (lokal)
</div>

Wenn das Gateway auf demselben Rechner läuft, öffne:

* http://127.0.0.1:18789/ (oder http://localhost:18789/)

Wenn die Seite nicht geladen wird, starte zuerst das Gateway: `openclaw gateway`.

Die Authentifizierung wird während des WebSocket-Handshakes über folgende Parameter übergeben:

* `connect.params.auth.token`
* `connect.params.auth.password`
  Im Einstellungsbereich des Dashboards kannst du ein Token speichern; Passwörter werden nicht dauerhaft gespeichert.
  Der Onboarding-Assistent erzeugt standardmäßig ein Gateway-Token, füge dieses daher bei der ersten Verbindung hier ein.

<div id="what-it-can-do-today">
  ## Was es (heute) kann
</div>

* Chat mit dem Modell über Gateway-WS (`chat.history`, `chat.send`, `chat.abort`, `chat.inject`)
* Tool-Aufrufe streamen + Live-Tool-Ausgabekarten im Chat (Agent-Ereignisse)
* Channels: WhatsApp/Telegram/Discord/Slack + Plugin-Channels (Mattermost usw.), Status + QR-Login + Channel-spezifische Konfiguration (`channels.status`, `web.login.*`, `config.patch`)
* Instanzen: Präsenzliste + Aktualisierung (`system-presence`)
* Sitzungen: Liste + Thinking/Verbose-Overrides pro Sitzung (`sessions.list`, `sessions.patch`)
* Cron-Jobs: auflisten/hinzufügen/ausführen/aktivieren/deaktivieren + Ausführungsverlauf (`cron.*`)
* Fähigkeiten: Status, aktivieren/deaktivieren, installieren, API-Key-Updates (`skills.*`)
* Knoten: Liste + Fähigkeiten (`node.list`)
* Exec-Genehmigungen: Gateway- oder Knoten-Allowlists bearbeiten + Policy für `exec host=gateway/node` abfragen (`exec.approvals.*`)
* Konfiguration: `~/.openclaw/openclaw.json` anzeigen/bearbeiten (`config.get`, `config.set`)
* Konfiguration: anwenden + mit Validierung neu starten (`config.apply`) und die zuletzt aktive Sitzung aufwecken
* Konfigurationsschreibvorgänge enthalten einen Base-Hash-Schutzmechanismus, um zu verhindern, dass gleichzeitige Änderungen überschrieben werden
* Konfigurationsschema + Formular-Rendering (`config.schema`, einschließlich Plugin- und Channel-Schemata); Roh-JSON-Editor bleibt verfügbar
* Debug: Status-/Health-/Model-Snapshots + Ereignis-Log + manuelle RPC-Aufrufe (`status`, `health`, `models.list`)
* Logs: fortlaufende Anzeige der Gateway-Datei-Logs mit Filter/Export (`logs.tail`)
* Update: Paket-/Git-Update ausführen + Neustart (`update.run`) mit Neustart-Report

<div id="chat-behavior">
  ## Chat-Verhalten
</div>

* `chat.send` ist **nicht blockierend**: es bestätigt sofort mit `{ runId, status: "started" }` und die Antwort wird über `chat`-Events gestreamt.
* Erneutes Senden mit demselben `idempotencyKey` gibt `{ status: "in_flight" }` während der Ausführung und `{ status: "ok" }` nach Abschluss zurück.
* `chat.inject` hängt eine Assistentennotiz an das Sitzungsprotokoll an und sendet ein `chat`-Event für reine UI-Updates (kein Agent-Run, keine Kanalzustellung).
* Stopp:
  * Klicke auf **Stop** (ruft `chat.abort` auf)
  * Tippe `/stop` (oder `stop|esc|abort|wait|exit|interrupt`), um Out-of-Band abzubrechen
  * `chat.abort` unterstützt `{ sessionKey }` (kein `runId`), um alle aktiven Runs für diese Sitzung abzubrechen

<div id="tailnet-access-recommended">
  ## Tailnet-Zugriff (empfohlen)
</div>

<div id="integrated-tailscale-serve-preferred">
  ### Integriertes Tailscale Serve (bevorzugt)
</div>

Belasse das Gateway auf Loopback und lasse Tailscale Serve es per HTTPS als Proxy bereitstellen:

```bash
openclaw gateway --tailscale serve
```

Öffne:

* `https://<magicdns>/` (oder deinen konfigurierten `gateway.controlUi.basePath`)

Standardmäßig können Serve-Requests anhand von Tailscale-Identity-Headern
(`tailscale-user-login`) authentifiziert werden, wenn `gateway.auth.allowTailscale` auf `true` gesetzt ist. OpenClaw
prüft die Identität, indem es die `x-forwarded-for`-Adresse mit
`tailscale whois` auflöst und mit dem Header abgleicht, und akzeptiert diese nur, wenn die
Anfrage die Loopback-Schnittstelle mit den `x-forwarded-*`-Headern von Tailscale erreicht. Setze
`gateway.auth.allowTailscale: false` (oder erzwinge `gateway.auth.mode: "password"`),
wenn du auch für Serve-Traffic zwingend ein Token/Passwort verlangen möchtest.

<div id="bind-to-tailnet-token">
  ### Mit Tailnet und Token verknüpfen
</div>

```bash
openclaw gateway --bind tailnet --token "$(openssl rand -hex 32)"
```

Öffne anschließend:

* `http://<tailscale-ip>:18789/` (oder deinen konfigurierten `gateway.controlUi.basePath`)

Füge das Token in den UI-Einstellungen ein (als `connect.params.auth.token` gesendet).

<div id="insecure-http">
  ## Unsicheres HTTP
</div>

Wenn du das Dashboard über reines HTTP (`http://<lan-ip>` oder `http://<tailscale-ip>`) öffnest,
läuft der Browser in einem **unsicheren Kontext** und blockiert WebCrypto. Standardmäßig
**blockiert** OpenClaw Control UI‑Verbindungen ohne Geräteidentität.

**Empfohlene Maßnahme:** Verwende HTTPS (Tailscale Serve) oder öffne die UI lokal:

* `https://<magicdns>/` (Serve)
* `http://127.0.0.1:18789/` (auf dem Gateway-Host)

**Beispiel für ein Downgrade (nur Token über HTTP):**

```json5
{
  gateway: {
    controlUi: { allowInsecureAuth: true },
    bind: "tailnet",
    auth: { mode: "token", token: "replace-me" }
  }
}
```

Dies deaktiviert die Geräteidentität und Kopplung für die Control UI (auch bei HTTPS). Verwende dies nur, wenn du dem Netzwerk vertraust.

Siehe [Tailscale](/de/gateway/tailscale) für Hinweise zur Einrichtung von HTTPS.

<div id="building-the-ui">
  ## Die UI bauen
</div>

Das Gateway stellt statische Dateien aus `dist/control-ui` bereit. Erstelle sie mit:

```bash
pnpm ui:build # installiert UI-Abhängigkeiten beim ersten Ausführen automatisch
```

Optionale absolute Basis-URL (wenn du feste Asset-URLs verwenden möchtest):

```bash
OPENCLAW_CONTROL_UI_BASE_PATH=/openclaw/ pnpm ui:build
```

Für die lokale Entwicklung (separaten Dev-Server):

```bash
pnpm ui:dev # installiert UI-Abhängigkeiten beim ersten Durchlauf automatisch
```

Konfiguriere die UI dann so, dass sie auf die WS-URL deines Gateways zeigt (z. B. `ws://127.0.0.1:18789`).

<div id="debuggingtesting-dev-server-remote-gateway">
  ## Debugging/Tests: Dev-Server + Remote-Gateway
</div>

Die Control UI besteht aus statischen Dateien; das WebSocket-Ziel ist konfigurierbar und kann
sich vom HTTP-Origin unterscheiden. Das ist praktisch, wenn du den Vite-Dev-Server
lokal laufen lassen möchtest, während das Gateway anderswo läuft.

1. Starte den UI-Dev-Server: `pnpm ui:dev`
2. Öffne eine URL wie:

```text
http://localhost:5173/?gatewayUrl=ws://<gateway-host>:18789
```

Optionale einmalige Authentifizierung (bei Bedarf):

```text
http://localhost:5173/?gatewayUrl=wss://<gateway-host>:18789&token=<gateway-token>
```

Hinweise:

* `gatewayUrl` wird nach dem Laden in localStorage gespeichert und aus der URL entfernt.
* `token` wird in localStorage gespeichert; `password` wird nur im Arbeitsspeicher gehalten.
* Verwende `wss://`, wenn der Gateway hinter TLS läuft (Tailscale Serve, HTTPS-Proxy usw.).

Einzelheiten zur Einrichtung des Remotezugriffs: [Remote access](/de/gateway/remote).
