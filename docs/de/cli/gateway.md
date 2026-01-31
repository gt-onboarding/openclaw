---
title: Gateway
summary: "OpenClaw Gateway CLI (`openclaw gateway`) — Gateways ausführen, abfragen und auffinden"
read_when:
  - Ausführen des Gateway über die CLI (Dev- oder Server-Umgebung)
  - Debugging von Gateway-Authentifizierung, Bindemodi und Konnektivität
  - Auffinden von Gateways über Bonjour (LAN + tailnet)
---

<div id="gateway-cli">
  # Gateway-CLI
</div>

Das Gateway ist der WebSocket-Server von OpenClaw (Kanäle, Knoten, Sitzungen, Hooks).

Die auf dieser Seite beschriebenen Unterbefehle werden mit `openclaw gateway …` aufgerufen.

Verwandte Dokumentation:

* [/gateway/bonjour](/de/gateway/bonjour)
* [/gateway/discovery](/de/gateway/discovery)
* [/gateway/configuration](/de/gateway/configuration)

<div id="run-the-gateway">
  ## Gateway starten
</div>

Lokalen Gateway-Prozess starten:

```bash
openclaw gateway
```

Vordergrund-Alias:

```bash
openclaw gateway run
```

Hinweise:

* Standardmäßig startet der Gateway nicht, sofern nicht `gateway.mode=local` in `~/.openclaw/openclaw.json` gesetzt ist. Verwende `--allow-unconfigured` für Ad-hoc-/Dev-Ausführungen.
* Das Binden an Schnittstellen jenseits von Loopback ohne Authentifizierung ist blockiert (Sicherheitsmechanismus).
* `SIGUSR1` löst einen Neustart im laufenden Prozess aus, wenn autorisiert (aktiviere `commands.restart` oder verwende das Gateway-Tool/Config-Apply/Update).
* `SIGINT`/`SIGTERM`-Handler beenden den Gateway-Prozess, stellen aber keinen benutzerdefinierten Terminalzustand wieder her. Wenn du die CLI in einem TUI oder mit Raw-Mode-Eingabe einbettest, stelle das Terminal vor dem Beenden wieder her.

<div id="options">
  ### Optionen
</div>

* `--port <port>`: WebSocket-Port (Standardwert stammt aus Konfiguration/Umgebung; üblicherweise `18789`).
* `--bind <loopback|lan|tailnet|auto|custom>`: Bindemodus des Listeners.
* `--auth <token|password>`: Authentifizierungsmodus überschreiben.
* `--token <token>`: Token überschreiben (setzt außerdem `OPENCLAW_GATEWAY_TOKEN` für den Prozess).
* `--password <password>`: Passwort überschreiben (setzt außerdem `OPENCLAW_GATEWAY_PASSWORD` für den Prozess).
* `--tailscale <off|serve|funnel>`: Gateway über Tailscale bereitstellen.
* `--tailscale-reset-on-exit`: Tailscale-Serve/Funnel-Konfiguration beim Herunterfahren zurücksetzen.
* `--allow-unconfigured`: Gateway-Start ohne `gateway.mode=local` in der Konfiguration erlauben.
* `--dev`: Dev-Konfiguration + Arbeitsbereich erstellen, falls nicht vorhanden (überspringt BOOTSTRAP.md).
* `--reset`: Dev-Konfiguration + Zugangsdaten + Sitzungen + Arbeitsbereich zurücksetzen (erfordert `--dev`).
* `--force`: Vorhandenen Listener auf dem ausgewählten Port vor dem Start beenden.
* `--verbose`: Ausführliche Logs.
* `--claude-cli-logs`: Nur claude-cli-Logs in der Konsole anzeigen (und dessen stdout/stderr aktivieren).
* `--ws-log <auto|full|compact>`: WebSocket-Log-Stil (Standard `auto`).
* `--compact`: Alias für `--ws-log compact`.
* `--raw-stream`: Rohe Modell-Stream-Events im jsonl-Format protokollieren.
* `--raw-stream-path <path>`: Pfad für rohe Stream-jsonl-Dateien.

<div id="query-a-running-gateway">
  ## Ein laufendes Gateway abfragen
</div>

Alle Abfragebefehle verwenden WebSocket-RPC.

Ausgabemodi:

* Standard: menschenlesbar (farbig in TTY).
* `--json`: maschinenlesbares JSON (kein Styling/Spinner).
* `--no-color` (oder `NO_COLOR=1`): ANSI-Farben deaktivieren, menschenlesbares Layout beibehalten.

Gemeinsame Optionen (sofern unterstützt):

* `--url <url>`: Gateway-WebSocket-URL.
* `--token <token>`: Gateway-Token.
* `--password <password>`: Gateway-Passwort.
* `--timeout <ms>`: Timeout/Budget (unterscheidet sich je nach Befehl).
* `--expect-final`: auf eine endgültige Antwort warten (bei Agent-Aufrufen).

<div id="gateway-health">
  ### `gateway health`
</div>

```bash
openclaw gateway health --url ws://127.0.0.1:18789
```

<div id="gateway-status">
  ### `gateway status`
</div>

`gateway status` zeigt den Gateway-Dienst (launchd/systemd/schtasks) sowie optional eine RPC-Probe an.

```bash
openclaw gateway status
openclaw gateway status --json
```

Optionen:

* `--url <url>`: die Probe-URL überschreiben.
* `--token <token>`: Token-Authentifizierung für die Probe.
* `--password <password>`: Passwort-Authentifizierung für die Probe.
* `--timeout <ms>`: Timeout für die Probe (Standard: `10000`).
* `--no-probe`: RPC-Probe überspringen (nur Dienstansicht).
* `--deep`: auch Systemdienste scannen.

<div id="gateway-probe">
  ### `gateway probe`
</div>

`gateway probe` ist der „alles debuggen“-Befehl. Er prüft immer:

* Ihr konfiguriertes Remote-Gateway (falls gesetzt) und
* localhost (Loopback) **auch wenn ein Remote-Gateway konfiguriert ist**.

Wenn mehrere Gateways erreichbar sind, werden alle ausgegeben. Mehrere Gateways werden unterstützt, wenn Sie isolierte Profile/Ports verwenden (z. B. einen Rescue‑Bot), aber die meisten Installationen betreiben weiterhin nur ein einzelnes Gateway.

```bash
openclaw gateway probe
openclaw gateway probe --json
```

<div id="remote-over-ssh-mac-app-parity">
  #### Remote über SSH (entspricht der Mac‑App)
</div>

Der Modus „Remote über SSH“ der macOS‑App verwendet ein lokales Port-Forwarding, sodass das Remote-Gateway (das ggf. nur an die Loopback-Schnittstelle gebunden ist) unter `ws://127.0.0.1:<port>` erreichbar wird.

CLI-Äquivalent:

```bash
openclaw gateway probe --ssh user@gateway-host
```

Optionen:

* `--ssh <target>`: `user@host` oder `user@host:port` (Port ist standardmäßig `22`).
* `--ssh-identity <path>`: Identity-Datei.
* `--ssh-auto`: wählt den zuerst gefundenen Gateway-Host als SSH-Ziel (nur LAN/WAB).

Konfiguration (optional, dient als Standardwerte):

* `gateway.remote.sshTarget`
* `gateway.remote.sshIdentity`

<div id="gateway-call-method">
  ### `gateway call <method>`
</div>

Low-Level-RPC-Hilfsbefehl.

```bash
openclaw gateway call status
openclaw gateway call logs.tail --params '{"sinceMs": 60000}'
```

<div id="manage-the-gateway-service">
  ## Gateway-Dienst verwalten
</div>

```bash
openclaw gateway install
openclaw gateway start
openclaw gateway stop
openclaw gateway restart
openclaw gateway uninstall
```

Hinweise:

* `gateway install` unterstützt `--port`, `--runtime`, `--token`, `--force`, `--json`.
* Lifecycle-Befehle akzeptieren `--json` für den Einsatz in Skripten.

<div id="discover-gateways-bonjour">
  ## Gateways entdecken (Bonjour)
</div>

`gateway discover` scannt nach Gateway-Beacons (`_openclaw-gw._tcp`).

* Multicast DNS-SD: `local.`
* Unicast DNS-SD (Wide-Area Bonjour): Wähle eine Domain (Beispiel: `openclaw.internal.`) und richte Split-DNS + einen DNS-Server ein; siehe [/gateway/bonjour](/de/gateway/bonjour)

Nur Gateways mit aktivierter Bonjour-Erkennung (Standard) senden das Beacon aus.

Wide-Area-Discovery-Einträge enthalten (TXT):

* `role` (Gateway-Rollenhinweis)
* `transport` (Transporthinweis, z. B. `gateway`)
* `gatewayPort` (WebSocket-Port, normalerweise `18789`)
* `sshPort` (SSH-Port; Standard ist `22`, falls nicht vorhanden)
* `tailnetDns` (MagicDNS-Hostname, falls verfügbar)
* `gatewayTls` / `gatewayTlsSha256` (TLS aktiviert + Zertifikatsfingerabdruck)
* `cliPath` (optionaler Hinweis für Remote-Installationen)

<div id="gateway-discover">
  ### `gateway discover`
</div>

```bash
openclaw gateway discover
```

Optionen:

* `--timeout <ms>`: Timeout für jeden Befehl (browse/resolve); Standardwert `2000`.
* `--json`: maschinenlesbare Ausgabe (deaktiviert außerdem Styling/Spinner).

Beispiele:

```bash
openclaw gateway discover --timeout 4000
openclaw gateway discover --json | jq '.beacons[].wsUrl'
```
