---
title: Gateway
summary: "Runbook für Dienst, Lebenszyklus und Betrieb des Gateway"
read_when:
  - Beim Ausführen oder Debuggen des Gateway-Prozesses
---

<div id="gateway-service-runbook">
  # Runbook für den Gateway-Service
</div>

Zuletzt aktualisiert: 2025-12-09

<div id="what-it-is">
  ## Was es ist
</div>

* Der dauerhaft laufende Prozess, der die einzelne Baileys-/Telegram-Verbindung sowie die Steuer-/Ereignisebene verwaltet.
* Ersetzt den Legacy-Befehl `gateway`. CLI-Einstiegspunkt: `openclaw gateway`.
* Läuft, bis er gestoppt wird; beendet sich bei fatalen Fehlern mit einem Exit-Code ungleich Null, sodass der Supervisor ihn neu startet.

<div id="how-to-run-local">
  ## Lokale Ausführung
</div>

```bash
openclaw gateway --port 18789
# for full debug/trace logs in stdio:
openclaw gateway --port 18789 --verbose
# falls der Port belegt ist, Listener beenden, dann starten:
openclaw gateway --force
# dev loop (auto-reload on TS changes):
pnpm gateway:watch
```

* Config-Hot-Reload überwacht `~/.openclaw/openclaw.json` (oder `OPENCLAW_CONFIG_PATH`).
  * Standardmodus: `gateway.reload.mode="hybrid"` (wendet sichere Änderungen im laufenden Betrieb an, Neustart bei kritischen Änderungen).
  * Hot-Reload verwendet bei Bedarf einen In-Process-Neustart über **SIGUSR1**.
  * Deaktiviere es mit `gateway.reload.mode="off"`.
* Bindet die WebSocket-Control-Plane an `127.0.0.1:<port>` (Standard 18789).
* Derselbe Port dient auch für HTTP (Control UI, Hooks, A2UI). Single-Port-Multiplex.
  * OpenAI Chat Completions (HTTP): [`/v1/chat/completions`](/de/gateway/openai-http-api).
  * OpenResponses (HTTP): [`/v1/responses`](/de/gateway/openresponses-http-api).
  * Tools Invoke (HTTP): [`/tools/invoke`](/de/gateway/tools-invoke-http-api).
* Startet standardmäßig einen Canvas-Dateiserver auf `canvasHost.port` (Standard `18793`), der `http://<gateway-host>:18793/__openclaw__/canvas/` aus `~/.openclaw/workspace/canvas` bereitstellt. Deaktiviere ihn mit `canvasHost.enabled=false` oder `OPENCLAW_SKIP_CANVAS_HOST=1`.
* Protokolliert nach stdout; verwende launchd/systemd, um den Prozess am Laufen zu halten und Logs zu rotieren.
* Verwende `--verbose`, um Debug-Logging (Handshakes, Req/Res, Events) aus der Log-Datei beim Troubleshooting in stdio zu spiegeln.
* `--force` verwendet `lsof`, um Listener auf dem gewählten Port zu finden, sendet SIGTERM, protokolliert, was beendet wurde, und startet dann das Gateway (schlägt früh fehl, wenn `lsof` fehlt).
* Wenn du den Prozess unter einem Supervisor betreibst (launchd/systemd/mac-App-Child-Process-Modus), sendet ein Stop/Restart typischerweise **SIGTERM**; ältere Builds können dies als `pnpm`-`ELIFECYCLE`-Exit-Code **143** (SIGTERM) anzeigen, was ein normaler Shutdown ist, kein Absturz.
* **SIGUSR1** löst einen In-Process-Neustart aus, wenn autorisiert (Anwenden/Aktualisieren von Gateway-Tools/-Config oder aktiviere `commands.restart` für manuelle Neustarts).
* Gateway-Authentifizierung ist standardmäßig erforderlich: Setze `gateway.auth.token` (oder `OPENCLAW_GATEWAY_TOKEN`) oder `gateway.auth.password`. Clients müssen `connect.params.auth.token/password` senden, es sei denn, sie verwenden Tailscale Serve Identity.
* Der Wizard generiert jetzt standardmäßig ein Token, selbst auf Loopback.
* Port-Priorität: `--port` &gt; `OPENCLAW_GATEWAY_PORT` &gt; `gateway.port` &gt; Standard `18789`.

<div id="remote-access">
  ## Remotezugriff
</div>

* Tailscale/VPN bevorzugt; andernfalls SSH-Tunnel:
  ```bash
  ssh -N -L 18789:127.0.0.1:18789 user@host
  ```
* Clients stellen dann über den Tunnel eine Verbindung zu `ws://127.0.0.1:18789` her.
* Wenn ein Token konfiguriert ist, müssen Clients ihn in `connect.params.auth.token` übermitteln, auch wenn die Verbindung über den Tunnel läuft.

<div id="multiple-gateways-same-host">
  ## Mehrere Gateways (gleicher Host)
</div>

In der Regel nicht nötig: Ein Gateway kann mehrere Messaging-Kanäle und Agenten bedienen. Verwende mehrere Gateways nur für Redundanz oder strikte Isolation (z. B. Rescue-Bot).

Mehrere Gateways werden unterstützt, wenn du Zustand und Konfiguration trennst und eindeutige Ports verwendest. Vollständige Anleitung: [Mehrere Gateways](/de/gateway/multiple-gateways).

Dienstnamen sind profilabhängig:

* macOS: `bot.molt.<profile>` (veraltete `com.openclaw.*` können noch existieren)
* Linux: `openclaw-gateway-<profile>.service`
* Windows: `OpenClaw Gateway (<profile>)`

Installationsmetadaten sind in der Dienstkonfiguration eingebettet:

* `OPENCLAW_SERVICE_MARKER=openclaw`
* `OPENCLAW_SERVICE_KIND=gateway`
* `OPENCLAW_SERVICE_VERSION=<version>`

Rescue-Bot-Pattern: Halte ein zweites Gateway isoliert, mit eigenem Profil, Zustandsverzeichnis, Arbeitsbereich und Basisport-Abständen. Vollständige Anleitung: [Rescue-Bot-Anleitung](/de/gateway/multiple-gateways#rescue-bot-guide).

<div id="dev-profile-dev">
  ### Dev-Profil (`--dev`)
</div>

Schnellpfad: Starte eine vollständig isolierte Dev-Instanz (Konfiguration/Status/arbeitsbereich), ohne dein primäres Setup anzutasten.

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
# dann die Dev-Instanz ansprechen:
openclaw --dev status
openclaw --dev health
```

Standardwerte (können über Umgebungsvariablen/Flags/Konfiguration überschrieben werden):

* `OPENCLAW_STATE_DIR=~/.openclaw-dev`
* `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
* `OPENCLAW_GATEWAY_PORT=19001` (Gateway WS + HTTP)
* Port des Browser-Steuerdienstes = `19003` (abgeleitet: `gateway.port+2`, nur Loopback)
* `canvasHost.port=19005` (abgeleitet: `gateway.port+4`)
* Der Standardwert von `agents.defaults.workspace` wird zu `~/.openclaw/workspace-dev`, wenn du `setup`/`onboard` mit `--dev` ausführst.

Abgeleitete Ports (Faustregeln):

* Basisport = `gateway.port` (oder `OPENCLAW_GATEWAY_PORT` / `--port`)
* Port des Browser-Steuerdienstes = Basis + 2 (nur Loopback)
* `canvasHost.port = base + 4` (oder `OPENCLAW_CANVAS_HOST_PORT` / Konfigurationsüberschreibung)
* CDP-Ports der Browser-Profile werden automatisch aus `browser.controlPort + 9 .. + 108` zugewiesen (pro Profil persistent gespeichert).

Checkliste pro Instanz:

* eindeutiger `gateway.port`
* eindeutiger `OPENCLAW_CONFIG_PATH`
* eindeutiger `OPENCLAW_STATE_DIR`
* eindeutiger `agents.defaults.workspace`
* separate WhatsApp-Nummern (bei Verwendung von WA)

Dienstinstallation pro Profil:

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

Beispiel:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

<div id="protocol-operator-view">
  ## Protokoll (Operator-Perspektive)
</div>

* Vollständige Dokumentation: [Gateway protocol](/de/gateway/protocol) und [Bridge protocol (legacy)](/de/gateway/bridge-protocol).
* Obligatorischer erster Frame vom Client: `req {type:"req", id, method:"connect", params:{minProtocol,maxProtocol,client:{id,displayName?,version,platform,deviceFamily?,modelIdentifier?,mode,instanceId?}, caps, auth?, locale?, userAgent? } }`.
* Gateway antwortet mit `res {type:"res", id, ok:true, payload:hello-ok }` (oder mit `ok:false` und einem Fehler und schließt dann).
* Nach dem Handshake:
  * Requests: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
  * Events: `{type:"event", event, payload, seq?, stateVersion?}`
* Strukturierte Presence-Einträge: `{host, ip, version, platform?, deviceFamily?, modelIdentifier?, mode, lastInputSeconds?, ts, reason?, tags?[], instanceId? }` (für WS-Clients stammt `instanceId` aus `connect.client.instanceId`).
* `agent`-Antworten sind zweistufig: zuerst ein `res`-Ack `{runId,status:"accepted"}`, dann eine finale `res` `{runId,status:"ok"|"error",summary}`, nachdem der Run abgeschlossen ist; gestreamte Ausgaben werden als `event:"agent"` gesendet.

<div id="methods-initial-set">
  ## Methods (erste Menge)
</div>

* `health` — vollständiger Health-Snapshot (gleiche Struktur wie `openclaw health --json`).
* `status` — kurze Zusammenfassung.
* `system-presence` — aktuelle Präsenzliste.
* `system-event` — eine Präsenz-/Systemnotiz (strukturiert) erstellen.
* `send` — eine Nachricht über die aktiven Kanal(e) senden.
* `agent` — einen agent-Durchlauf ausführen (streamt Ereignisse über dieselbe Verbindung zurück).
* `node.list` — gekoppelte + aktuell verbundene Knoten auflisten (enthält `caps`, `deviceFamily`, `modelIdentifier`, `paired`, `connected` und angekündigte `commands`).
* `node.describe` — einen Knoten beschreiben (Fähigkeiten + unterstützte `node.invoke`-Befehle; funktioniert für gekoppelte Knoten und aktuell verbundene, nicht gekoppelte Knoten).
* `node.invoke` — einen Befehl auf einem Knoten ausführen (z. B. `canvas.*`, `camera.*`).
* `node.pair.*` — Kopplungs-Lebenszyklus (`request`, `list`, `approve`, `reject`, `verify`).

Siehe auch: [Presence](/de/concepts/presence) für Details dazu, wie Presence erzeugt/zusammengeführt wird und warum eine stabile `client.instanceId` wichtig ist.

<div id="events">
  ## Events
</div>

* `agent` — gestreamte Tool-/Output-Events aus dem agent-Run (mit Sequenz-Tags).
* `presence` — Presence-Updates (Deltas mit stateVersion), die an alle verbundenen Clients gepusht werden.
* `tick` — periodischer Keepalive/No-op zur Bestätigung der Erreichbarkeit.
* `shutdown` — Gateway wird heruntergefahren; die Payload enthält `reason` und optional `restartExpectedMs`. Clients sollten sich erneut verbinden.

<div id="webchat-integration">
  ## WebChat-Integration
</div>

* WebChat ist eine native SwiftUI-UI, die direkt mit dem Gateway-WebSocket für Verlauf, Senden, Abbruch und Events kommuniziert.
* Remote-Zugriff erfolgt über denselben SSH/Tailscale-Tunnel; wenn ein Gateway-Token konfiguriert ist, sendet der Client es während `connect` mit.
* Die macOS-App verbindet sich über einen einzelnen WS (gemeinsam genutzte Verbindung); sie lädt den Presence-Zustand aus dem initialen Snapshot und hört auf `presence`-Events, um die UI zu aktualisieren.

<div id="typing-and-validation">
  ## Typen und Validierung
</div>

* Der Server validiert jeden eingehenden Frame mit AJV gegen das JSON‑Schema, das aus den Protokolldefinitionen generiert wird.
* Clients (TS/Swift) verwenden generierte Typen (TS direkt, Swift über den Generator des Repositorys).
* Protokolldefinitionen sind die maßgebliche Quelle; regeneriere Schema/Modelle mit:
  * `pnpm protocol:gen`
  * `pnpm protocol:gen:swift`

<div id="connection-snapshot">
  ## Verbindungssnapshot
</div>

* `hello-ok` enthält einen `snapshot` mit `presence`, `health`, `stateVersion` und `uptimeMs` plus `policy {maxPayload,maxBufferedBytes,tickIntervalMs}`, sodass Clients sofort ohne zusätzliche Anfragen rendern können.
* `health`/`system-presence` bleiben für manuelle Aktualisierungen verfügbar, sind aber beim Verbindungsaufbau nicht erforderlich.

<div id="error-codes-reserror-shape">
  ## Fehlercodes (res.error-Format)
</div>

* Fehler verwenden `{ code, message, details?, retryable?, retryAfterMs? }`.
* Standard-Codes:
  * `NOT_LINKED` — WhatsApp nicht authentifiziert.
  * `AGENT_TIMEOUT` — agent hat nicht innerhalb der konfigurierten Frist geantwortet.
  * `INVALID_REQUEST` — Schema-/Parametervalidierung fehlgeschlagen.
  * `UNAVAILABLE` — Gateway wird heruntergefahren oder eine Abhängigkeit ist nicht verfügbar.

<div id="keepalive-behavior">
  ## Keepalive-Verhalten
</div>

* `tick`-Events (oder WS-Ping/Pong) werden periodisch gesendet, damit Clients wissen, dass das Gateway noch läuft, auch wenn gerade kein Traffic fließt.
* Send-/Agent-Bestätigungen bleiben eigenständige Antworten; überlade `tick`-Events nicht mit Sends.

<div id="replay-gaps">
  ## Replay / Lücken
</div>

* Events werden nicht erneut wiedergegeben. Clients erkennen `seq`-Lücken und sollten vor dem Fortfahren (`health` + `system-presence`) aktualisieren. WebChat- und macOS-Clients aktualisieren bei einer Lücke jetzt automatisch.

<div id="supervision-macos-example">
  ## Überwachung (macOS-Beispiel)
</div>

* Verwende launchd, um den Dienst am Leben zu halten:
  * Programm: Pfad zu `openclaw`
  * Argumente: `gateway`
  * KeepAlive: true
  * StandardOut/Err: Dateipfade oder `syslog`
* Bei Fehlern startet launchd den Dienst neu; bei einer fatalen Fehlkonfiguration sollte der Prozess weiterhin beendet werden, damit du das bemerkst.
* LaunchAgents sind benutzerspezifisch und erfordern eine angemeldete Benutzer-Sitzung; für headless-Setups verwende einen benutzerdefinierten LaunchDaemon (nicht mitgeliefert).
  * `openclaw gateway install` schreibt `~/Library/LaunchAgents/bot.molt.gateway.plist`
    (oder `bot.molt.<profile>.plist`; veraltete `com.openclaw.*`-Einträge werden bereinigt).
  * `openclaw doctor` überprüft die LaunchAgent-Konfiguration und kann sie auf die aktuellen Standardwerte aktualisieren.

<div id="gateway-service-management-cli">
  ## Gateway-Dienstverwaltung (CLI)
</div>

Zum Installieren, Starten, Stoppen, Neustarten und Abfragen des Status die Gateway-CLI verwenden:

```bash
openclaw gateway status
openclaw gateway install
openclaw gateway stop
openclaw gateway restart
openclaw logs --follow
```

Hinweise:

* `gateway status` prüft standardmäßig das Gateway-RPC anhand des aufgelösten Ports bzw. der aufgelösten Konfiguration des Dienstes (überschreibbar mit `--url`).
* `gateway status --deep` ergänzt systemweite Scans (LaunchDaemons/System Units).
* `gateway status --no-probe` überspringt die RPC-Probe (nützlich, wenn das Netzwerk down ist).
* `gateway status --json` ist stabil für Skripte.
* `gateway status` meldet die **Supervisor-Laufzeit** (launchd/systemd läuft) getrennt von der **RPC-Erreichbarkeit** (WS-Verbindung + Status-RPC).
* `gateway status` gibt Konfigurationspfad + Probe-Ziel aus, um Verwirrung zwischen „localhost vs. LAN-Bindung“ und Profilinkonsistenzen zu vermeiden.
* `gateway status` enthält die letzte Gateway-Fehlerzeile, wenn der Dienst so aussieht, als würde er laufen, der Port aber geschlossen ist.
* `logs` streamt das Gateway-Dateilog via RPC (kein manuelles `tail`/`grep` nötig).
* Wenn andere gateway-ähnliche Dienste erkannt werden, warnt die CLI, außer es sind OpenClaw-Profil-Dienste.
  Wir empfehlen trotzdem **ein Gateway pro Maschine** für die meisten Setups; verwende isolierte Profile/Ports für Redundanz oder einen Rescue-Bot. Siehe [Multiple gateways](/de/gateway/multiple-gateways).
  * Aufräumen: `openclaw gateway uninstall` (aktueller Dienst) und `openclaw doctor` (Legacy-Migrationen).
* `gateway install` ist ein No-op, wenn bereits installiert; verwende `openclaw gateway install --force` zum Neuinstallieren (Profil-/Umgebungs-/Pfadänderungen).

Gebündelte Mac-App:

* OpenClaw.app kann ein Node-basiertes Gateway-Relay bündeln und einen pro-Benutzer-LaunchAgent mit dem Label
  `bot.molt.gateway` installieren (oder `bot.molt.<profile>`; ältere `com.openclaw.*`-Labels werden weiterhin sauber entladen).
* Um sie sauber zu stoppen, verwende `openclaw gateway stop` (oder `launchctl bootout gui/$UID/bot.molt.gateway`).
* Zum Neustarten verwende `openclaw gateway restart` (oder `launchctl kickstart -k gui/$UID/bot.molt.gateway`).
  * `launchctl` funktioniert nur, wenn der LaunchAgent installiert ist; ansonsten zuerst `openclaw gateway install` verwenden.
  * Ersetze das Label durch `bot.molt.<profile>`, wenn ein benanntes Profil läuft.

<div id="supervision-systemd-user-unit">
  ## Überwachung (systemd User-Unit)
</div>

OpenClaw installiert standardmäßig einen **systemd User-Service** auf Linux/WSL2-Systemen. Wir
empfehlen User-Services für Einbenutzer-Systeme (einfachere Umgebung, benutzerspezifische Konfiguration).
Verwende einen **System-Service** für Mehrbenutzer- oder Always-on-Server (kein Lingering
erforderlich, gemeinsame Prozessüberwachung).

`openclaw gateway install` schreibt die User-Unit. `openclaw doctor` prüft die
Unit und kann sie aktualisieren, damit sie den aktuell empfohlenen Standardeinstellungen entspricht.

Erstelle `~/.config/systemd/user/openclaw-gateway[-<profile>].service`:

```
[Unit]
Description=OpenClaw Gateway (profile: <profile>, v<version>)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5
Environment=OPENCLAW_GATEWAY_TOKEN=
WorkingDirectory=/home/youruser

[Install]
WantedBy=default.target
```

Aktiviere Lingering (erforderlich, damit der Benutzerdienst bei Abmeldung oder Inaktivität weiterläuft):

```
sudo loginctl enable-linger youruser
```

Das Onboarding führt dies unter Linux/WSL2 aus (fragt möglicherweise nach `sudo` und schreibt nach `/var/lib/systemd/linger`).
Aktiviere dann den Dienst:

```
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

**Alternative (Systemdienst)** – für Always-on- oder Multi-User-Server kannst du
statt einer User-Unit eine systemd-**System**-Unit installieren (kein Lingering erforderlich).
Erstelle `/etc/systemd/system/openclaw-gateway[-<profile>].service` (kopiere die obige Unit,
ändere `WantedBy=multi-user.target`, setze `User=` + `WorkingDirectory=`) und dann:

```
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

<div id="windows-wsl2">
  ## Windows (WSL2)
</div>

Windows-Installationen sollten **WSL2** verwenden und dem oben beschriebenen Abschnitt zu Linux systemd folgen.

<div id="operational-checks">
  ## Betriebskontrollen
</div>

* Liveness: WS-Verbindung öffnen und `req:connect` senden → `res` mit `payload.type="hello-ok"` (inkl. Snapshot) erwarten.
* Readiness: `health` aufrufen → `ok: true` und einen verknüpften Kanal in `linkChannel` erwarten (falls zutreffend).
* Debug: `tick`- und `presence`-Events abonnieren; sicherstellen, dass `status` das Alter von Verknüpfung/Authentifizierung anzeigt; Presence-Einträge zeigen den Gateway-Host und verbundene Clients.

<div id="safety-guarantees">
  ## Sicherheitsgarantien
</div>

* Gehe standardmäßig von einem Gateway pro Host aus; wenn du mehrere Profile betreibst, isoliere Ports/Zustand und stelle sicher, dass du die richtige Instanz adressierst.
* Kein Fallback auf direkte Baileys-Verbindungen; wenn das Gateway nicht verfügbar ist, schlagen Sendevorgänge sofort fehl.
* Erste Frames, die nicht `connect` sind, oder fehlerhaftes JSON werden abgelehnt und der Socket wird geschlossen.
* Graceful Shutdown: `shutdown`-Event auslösen, bevor der Socket geschlossen wird; Clients müssen Close + Reconnect handhaben.

<div id="cli-helpers">
  ## CLI-Helfer
</div>

* `openclaw gateway health|status` — fordert Health-/Statusinformationen über den Gateway-WS an.
* `openclaw message send --target <num> --message "hi" [--media ...]` — über den Gateway senden (idempotent für WhatsApp).
* `openclaw agent --message "hi" --to <num>` — einen Agent-Turn ausführen (wartet standardmäßig auf das finale Ergebnis).
* `openclaw gateway call <method> --params '{"k":"v"}'` — roher Methodenaufruf für Debugging.
* `openclaw gateway stop|restart` — den überwachten Gateway-Dienst stoppen/neustarten (launchd/systemd).
* Gateway-Helfer-Unterbefehle setzen einen laufenden Gateway unter `--url` voraus; sie starten keinen mehr automatisch.

<div id="migration-guidance">
  ## Hinweise zur Migration
</div>

* Stellen Sie die Verwendung von `openclaw gateway` und des Legacy-TCP-Steuerports ein.
* Aktualisieren Sie Clients so, dass sie das WS-Protokoll mit obligatorischem Connect-Handshake und strukturierter Präsenz verwenden.