---
title: CLI
summary: "OpenClaw CLI-Referenz für `openclaw`-Befehle, Unterbefehle und Optionen"
read_when:
  - Beim Hinzufügen oder Ändern von CLI-Befehlen oder -Optionen
  - Beim Dokumentieren neuer Befehlsschnittstellen
---

<div id="cli-reference">
  # CLI-Referenz
</div>

Diese Seite beschreibt das aktuelle CLI-Verhalten. Wenn sich Befehle ändern, aktualisieren Sie dieses Dokument.

<div id="command-pages">
  ## Befehlsseiten
</div>

* [`setup`](/de/cli/setup)
* [`onboard`](/de/cli/onboard)
* [`configure`](/de/cli/configure)
* [`config`](/de/cli/config)
* [`doctor`](/de/cli/doctor)
* [`dashboard`](/de/cli/dashboard)
* [`reset`](/de/cli/reset)
* [`uninstall`](/de/cli/uninstall)
* [`update`](/de/cli/update)
* [`message`](/de/cli/message)
* [`agent`](/de/cli/agent)
* [`agents`](/de/cli/agents)
* [`acp`](/de/cli/acp)
* [`status`](/de/cli/status)
* [`health`](/de/cli/health)
* [`sessions`](/de/cli/sessions)
* [`gateway`](/de/cli/gateway)
* [`logs`](/de/cli/logs)
* [`system`](/de/cli/system)
* [`models`](/de/cli/models)
* [`memory`](/de/cli/memory)
* [`nodes`](/de/cli/nodes)
* [`devices`](/de/cli/devices)
* [`node`](/de/cli/node)
* [`approvals`](/de/cli/approvals)
* [`sandbox`](/de/cli/sandbox)
* [`tui`](/de/cli/tui)
* [`browser`](/de/cli/browser)
* [`cron`](/de/cli/cron)
* [`dns`](/de/cli/dns)
* [`docs`](/de/cli/docs)
* [`hooks`](/de/cli/hooks)
* [`webhooks`](/de/cli/webhooks)
* [`pairing`](/de/cli/pairing)
* [`plugins`](/de/cli/plugins) (Plugin-Befehle)
* [`channels`](/de/cli/channels)
* [`security`](/de/cli/security)
* [`skills`](/de/cli/skills)
* [`voicecall`](/de/cli/voicecall) (Plugin; falls installiert)

<div id="global-flags">
  ## Globale Flags
</div>

* `--dev`: State unter `~/.openclaw-dev` isolieren und Standardports anpassen.
* `--profile <name>`: State unter `~/.openclaw-<name>` isolieren.
* `--no-color`: ANSI-Farbausgabe deaktivieren.
* `--update`: Kurzform für `openclaw update` (nur bei Source-Installationen).
* `-V`, `--version`, `-v`: Version ausgeben und beenden.

<div id="output-styling">
  ## Ausgabestil
</div>

* ANSI-Farben und Fortschrittsanzeigen werden nur in TTY-Sitzungen angezeigt.
* OSC-8-Hyperlinks werden in unterstützten Terminals als anklickbare Links angezeigt; andernfalls weichen wir auf einfache URLs aus.
* `--json` (und `--plain`, wo unterstützt) deaktiviert das Styling für saubere Ausgaben.
* `--no-color` deaktiviert ANSI-Styling; `NO_COLOR=1` wird ebenfalls berücksichtigt.
* Langlaufende Befehle zeigen eine Fortschrittsanzeige (OSC 9;4, sofern unterstützt).

<div id="color-palette">
  ## Farbpalette
</div>

OpenClaw verwendet eine Lobster-Farbpalette für CLI-Ausgaben.

* `accent` (#FF5A2D): Überschriften, Beschriftungen, primäre Hervorhebungen.
* `accentBright` (#FF7A3D): Befehlsnamen, Hervorhebungen.
* `accentDim` (#D14A22): Text für sekundäre Hervorhebungen.
* `info` (#FF8A5B): Informationsanzeigen.
* `success` (#2FBF71): Erfolgszustände.
* `warn` (#FFB020): Warnungen, Fallbacks, Hinweise.
* `error` (#E23D2D): Fehler, Fehlerzustände.
* `muted` (#8B7F77): Dezente Darstellung, Metadaten.

Maßgebliche Quelle der Palette: `src/terminal/palette.ts` (aka „lobster seam“).

<div id="command-tree">
  ## Befehlshierarchie
</div>

```
openclaw [--dev] [--profile <name>] <command>
  setup
  onboard
  configure
  config
    get
    set
    unset
  doctor
  security
    audit
  reset
  uninstall
  update
  channels
    list
    status
    logs
    add
    remove
    login
    logout
  skills
    list
    info
    check
  plugins
    list
    info
    install
    enable
    disable
    doctor
  memory
    status
    index
    search
  message
  agent
  agents
    list
    add
    delete
  acp
  status
  health
  sessions
  gateway
    call
    health
    status
    probe
    discover
    install
    uninstall
    start
    stop
    restart
    run
  logs
  system
    event
    heartbeat last|enable|disable
    presence
  models
    list
    status
    set
    set-image
    aliases list|add|remove
    fallbacks list|add|remove|clear
    image-fallbacks list|add|remove|clear
    scan
    auth add|setup-token|paste-token
    auth order get|set|clear
  sandbox
    list
    recreate
    explain
  cron
    status
    list
    add
    edit
    rm
    enable
    disable
    runs
    run
  nodes
  devices
  node
    run
    status
    install
    uninstall
    start
    stop
    restart
  approvals
    get
    set
    allowlist add|remove
  browser
    status
    start
    stop
    reset-profile
    tabs
    open
    focus
    close
    profiles
    create-profile
    delete-profile
    screenshot
    snapshot
    navigate
    resize
    click
    type
    press
    hover
    drag
    select
    upload
    fill
    dialog
    wait
    evaluate
    console
    pdf
  hooks
    list
    info
    check
    enable
    disable
    install
    update
  webhooks
    gmail setup|run
  pairing
    list
    approve
  docs
  dns
    setup
  tui
```

Hinweis: Plugins können zusätzliche Top-Level-Befehle bereitstellen (zum Beispiel `openclaw voicecall`).

<div id="security">
  ## Sicherheit
</div>

* `openclaw security audit` — prüft Konfiguration und lokalen Zustand auf gängige Sicherheitsfallstricke.
* `openclaw security audit --deep` — Best-Effort-Liveprüfung des Gateways.
* `openclaw security audit --fix` — verschärft sichere Defaults und passt Berechtigungen (chmod) für Zustand/Konfiguration an.

<div id="plugins">
  ## Plugins
</div>

Plugins und deren Konfiguration verwalten:

* `openclaw plugins list` — Plugins auflisten (verwende `--json` für maschinenlesbare Ausgabe).
* `openclaw plugins info <id>` — Details zu einem Plugin anzeigen.
* `openclaw plugins install <path|.tgz|npm-spec>` — ein Plugin installieren (oder einen Plugin-Pfad zu `plugins.load.paths` hinzufügen).
* `openclaw plugins enable <id>` / `disable <id>` — `plugins.entries.<id>.enabled` umschalten.
* `openclaw plugins doctor` — Plugin-Ladefehler ausgeben.

Die meisten Plugin-Änderungen erfordern einen Neustart des Gateways. Siehe [/plugin](/de/plugin).

<div id="memory">
  ## Memory
</div>

Vektorsuche in `MEMORY.md` + `memory/*.md`:

* `openclaw memory status` — Indexstatistiken anzeigen.
* `openclaw memory index` — Memory-Dateien neu indizieren.
* `openclaw memory search "<query>"` — semantische Suche im Memory-Speicher.

<div id="chat-slash-commands">
  ## Chat-Slash-Befehle
</div>

Chat-Nachrichten unterstützen `/...`-Befehle (Text und native). Siehe [/tools/slash-commands](/de/tools/slash-commands).

Highlights:

* `/status` für schnelle Diagnosen.
* `/config` für dauerhafte Konfigurationsänderungen.
* `/debug` für reine Laufzeit-Konfigurations-Overrides (im Speicher, nicht auf der Festplatte; erfordert `commands.debug: true`).

<div id="setup-onboarding">
  ## Einrichtung und Onboarding
</div>

<div id="setup">
  ### `setup`
</div>

Konfiguration und Arbeitsbereich initialisieren.

Optionen:

* `--workspace <dir>`: Pfad zum agent-Arbeitsbereich (Standard `~/.openclaw/workspace`).
* `--wizard`: Onboarding-Assistent ausführen.
* `--non-interactive`: Assistent ohne Rückfragen ausführen.
* `--mode <local|remote>`: Assistenten-Modus.
* `--remote-url <url>`: Remote-Gateway-URL.
* `--remote-token <token>`: Remote-Gateway-Token.

Der Assistent wird automatisch ausgeführt, wenn eines der Assistent-Flags gesetzt ist (`--non-interactive`, `--mode`, `--remote-url`, `--remote-token`).

<div id="onboard">
  ### `onboard`
</div>

Interaktiver Assistent zur Einrichtung von Gateway, Arbeitsbereich und Fähigkeiten.

Optionen:

* `--workspace <dir>`
* `--reset` (Konfiguration + Zugangsdaten + Sitzungen + Arbeitsbereich zurücksetzen, bevor der Assistent gestartet wird)
* `--non-interactive`
* `--mode <local|remote>`
* `--flow <quickstart|advanced|manual>` (manual ist ein Alias für advanced)
* `--auth-choice <setup-token|token|chutes|openai-codex|openai-api-key|openrouter-api-key|ai-gateway-api-key|moonshot-api-key|kimi-code-api-key|synthetic-api-key|venice-api-key|gemini-api-key|zai-api-key|apiKey|minimax-api|minimax-api-lightning|opencode-zen|skip>`
* `--token-provider <id>` (nicht interaktiv; wird mit `--auth-choice token` verwendet)
* `--token <token>` (nicht interaktiv; wird mit `--auth-choice token` verwendet)
* `--token-profile-id <id>` (nicht interaktiv; Standard: `<provider>:manual`)
* `--token-expires-in <duration>` (nicht interaktiv; z. B. `365d`, `12h`)
* `--anthropic-api-key <key>`
* `--openai-api-key <key>`
* `--openrouter-api-key <key>`
* `--ai-gateway-api-key <key>`
* `--moonshot-api-key <key>`
* `--kimi-code-api-key <key>`
* `--gemini-api-key <key>`
* `--zai-api-key <key>`
* `--minimax-api-key <key>`
* `--opencode-zen-api-key <key>`
* `--gateway-port <port>`
* `--gateway-bind <loopback|lan|tailnet|auto|custom>`
* `--gateway-auth <token|password>`
* `--gateway-token <token>`
* `--gateway-password <password>`
* `--remote-url <url>`
* `--remote-token <token>`
* `--tailscale <off|serve|funnel>`
* `--tailscale-reset-on-exit`
* `--install-daemon`
* `--no-install-daemon` (Alias: `--skip-daemon`)
* `--daemon-runtime <node|bun>`
* `--skip-channels`
* `--skip-skills`
* `--skip-health`
* `--skip-ui`
* `--node-manager <npm|pnpm|bun>` (pnpm empfohlen; bun für die Gateway-Laufzeit nicht empfohlen)
* `--json`

<div id="configure">
  ### `configure`
</div>

Interaktiver Konfigurationsassistent (Modelle, Kanäle, Fähigkeiten, Gateway).

<div id="config">
  ### `config`
</div>

Nicht-interaktive Konfigurations-Hilfsbefehle (get/set/unset). Das Ausführen von `openclaw config`
ohne Unterbefehl startet den Assistenten.

Unterbefehle:

* `config get &lt;path&gt;`: gibt einen Konfigurationswert aus (Punkt-/Klammerpfad).
* `config set &lt;path&gt; &lt;value&gt;`: setzt einen Wert (JSON5 oder Rohstring).
* `config unset &lt;path&gt;`: entfernt einen Wert.

<div id="doctor">
  ### `doctor`
</div>

Integritätsprüfungen und schnelle Korrekturen (Konfiguration, Gateway und Legacy-Dienste).

Optionen:

* `--no-workspace-suggestions`: deaktiviert Hinweise zum Arbeitsbereichs-Memory.
* `--yes`: akzeptiert Standardwerte ohne Rückfragen (headless, ohne Benutzerinteraktion).
* `--non-interactive`: überspringt Rückfragen; wendet nur sichere Migrationen an.
* `--deep`: durchsucht Systemdienste nach zusätzlichen Gateway-Installationen.

<div id="channel-helpers">
  ## Channel-Hilfsbefehle
</div>

<div id="channels">
  ### `channels`
</div>

Verwalte Chat-Konten für Kanäle (WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (Plugin)/Signal/iMessage/MS Teams).

Unterbefehle:

* `channels list`: konfigurierte Kanäle und Auth-Profile anzeigen.
* `channels status`: Erreichbarkeit des Gateways und Zustand der Kanäle prüfen (`--probe` führt zusätzliche Prüfungen aus; verwende `openclaw health` oder `openclaw status --deep` für Gateway-Health-Checks).
* Tipp: `channels status` gibt Warnungen mit vorgeschlagenen Korrekturen aus, wenn häufige Fehlkonfigurationen erkannt werden können (und verweist dich dann auf `openclaw doctor`).
* `channels logs`: aktuelle Kanal-Logs aus der Gateway-Protokolldatei anzeigen.
* `channels add`: assistentengestützte Einrichtung, wenn keine Flags übergeben werden; Flags schalten in den nicht-interaktiven Modus.
* `channels remove`: standardmäßig nur deaktivieren; mit `--delete` Konfigurationseinträge ohne Nachfragen entfernen.
* `channels login`: interaktiver Kanal-Login (nur WhatsApp Web).
* `channels logout`: von einer Kanal-Sitzung abmelden (falls unterstützt).

Gemeinsame Optionen:

* `--channel <name>`: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams`
* `--account <id>`: Kanal-Konto-ID (Standard `default`)
* `--name <label>`: Anzeigename für das Konto

Optionen für `channels login`:

* `--channel <channel>` (Standard `whatsapp`; unterstützt `whatsapp`/`web`)
* `--account <id>`
* `--verbose`

Optionen für `channels logout`:

* `--channel <channel>` (Standard `whatsapp`)
* `--account <id>`

Optionen für `channels list`:

* `--no-usage`: Momentaufnahmen zur Modellnutzung und zum Kontingent des Anbieters überspringen (nur OAuth/API-gestützt).
* `--json`: JSON ausgeben (inklusive Nutzung, außer `--no-usage` ist gesetzt).

Optionen für `channels logs`:

* `--channel <name|all>` (Standard `all`)
* `--lines <n>` (Standard `200`)
* `--json`

Weitere Details: [/concepts/oauth](/de/concepts/oauth)

Beispiele:

```bash
openclaw channels add --channel telegram --account alerts --name "Alerts Bot" --token $TELEGRAM_BOT_TOKEN
openclaw channels add --channel discord --account work --name "Work Bot" --token $DISCORD_BOT_TOKEN
openclaw channels remove --channel discord --account work --delete
openclaw channels status --probe
openclaw status --deep
```

<div id="skills">
  ### `skills`
</div>

Verfügbare Fähigkeiten auflisten und anzeigen, einschließlich Bereitschaftsstatus.

Unterbefehle:

* `skills list`: Fähigkeiten auflisten (Standard, wenn kein Unterbefehl angegeben ist).
* `skills info &lt;name&gt;`: Details für eine Fähigkeit anzeigen.
* `skills check`: Übersicht über erfüllte und fehlende Anforderungen.

Optionen:

* `--eligible`: nur einsatzbereite Fähigkeiten anzeigen.
* `--json`: JSON-Ausgabe (ohne Formatierung).
* `-v`, `--verbose`: Details zu fehlenden Anforderungen ausgeben.

Tipp: Nutze `npx clawhub`, um Fähigkeiten zu suchen, zu installieren und zu synchronisieren.

<div id="pairing">
  ### `pairing`
</div>

Kanalübergreifend Kopplungsanfragen für DMs genehmigen.

Unterbefehle:

* `pairing list <channel> [--json]`
* `pairing approve <channel> <code> [--notify]`

<div id="webhooks-gmail">
  ### `webhooks gmail`
</div>

Einrichtung des Gmail Pub/Sub-Hooks + Runner. Siehe [/automation/gmail-pubsub](/de/automation/gmail-pubsub).

Unterbefehle:

* `webhooks gmail setup` (erfordert `--account <email>`; unterstützt `--project`, `--topic`, `--subscription`, `--label`, `--hook-url`, `--hook-token`, `--push-token`, `--bind`, `--port`, `--path`, `--include-body`, `--max-bytes`, `--renew-minutes`, `--tailscale`, `--tailscale-path`, `--tailscale-target`, `--push-endpoint`, `--json`)
* `webhooks gmail run` (Laufzeitüberschreibungen für dieselben Flags)

<div id="dns-setup">
  ### `dns setup`
</div>

DNS-Hilfsprogramm für Wide-Area-Discovery (CoreDNS + Tailscale). Siehe [/gateway/discovery](/de/gateway/discovery).

Optionen:

* `--apply`: CoreDNS-Konfiguration installieren/aktualisieren (erfordert sudo; nur unter macOS).

<div id="messaging-agent">
  ## Messaging + Agent
</div>

<div id="message">
  ### `message`
</div>

Vereinheitlichter Befehl für ausgehende Nachrichten- und Channel-Aktionen.

Siehe: [/cli/message](/de/cli/message)

Unterbefehle:

* `message send|poll|react|reactions|read|edit|delete|pin|unpin|pins|permissions|search|timeout|kick|ban`
* `message thread <create|list|reply>`
* `message emoji <list|upload>`
* `message sticker <send|upload>`
* `message role <info|add|remove>`
* `message channel <info|list>`
* `message member info`
* `message voice status`
* `message event <list|create>`

Beispiele:

* `openclaw message send --target +15555550123 --message "Hi"`
* `openclaw message poll --channel discord --target channel:123 --poll-question "Snack?" --poll-option Pizza --poll-option Sushi`

<div id="agent">
  ### `agent`
</div>

Führe einen einzelnen Turn eines agents über das Gateway aus (oder eingebettet mit `--local`).

Erforderlich:

* `--message <text>`

Optionen:

* `--to <dest>` (für Sitzungsschlüssel und optionale Zustellung)
* `--session-id <id>`
* `--thinking <off|minimal|low|medium|high|xhigh>` (nur GPT-5.2- und Codex-Modelle)
* `--verbose <on|full|off>`
* `--channel <whatsapp|telegram|discord|slack|mattermost|signal|imessage|msteams>`
* `--local`
* `--deliver`
* `--json`
* `--timeout <seconds>`

<div id="agents">
  ### `agents`
</div>

Verwaltet isolierte Agenten (Arbeitsbereiche + Authentifizierung + Routing).

<div id="agents-list">
  #### `agents list`
</div>

Konfigurierte Agenten auflisten.

Optionen:

* `--json`
* `--bindings`

<div id="agents-add-name">
  #### `agents add [name]`
</div>

Fügt einen neuen isolierten Agent hinzu. Führt den geführten Assistenten aus, sofern keine Flags (oder `--non-interactive`) übergeben werden; `--workspace` ist im nichtinteraktiven Modus erforderlich.

Optionen:

* `--workspace <dir>`
* `--model <id>`
* `--agent-dir <dir>`
* `--bind <channel[:accountId]>` (wiederholbar)
* `--non-interactive`
* `--json`

Die Binding-Spezifikationen verwenden `channel[:accountId]`. Wenn `accountId` für WhatsApp weggelassen wird, wird die Standard-Konto-ID verwendet.

<div id="agents-delete-id">
  #### `agents delete <id>`
</div>

Einen Agent löschen und seinen Arbeitsbereich + Zustand bereinigen.

Optionen:

* `--force`
* `--json`

<div id="acp">
  ### `acp`
</div>

Starte die ACP-Bridge, die IDEs mit dem Gateway verbindet.

Siehe [`acp`](/de/cli/acp) für alle verfügbaren Optionen und Beispiele.

<div id="status">
  ### `status`
</div>

Zeigt den Zustand verknüpfter Sitzungen und zuletzt adressierte Empfänger.

Optionen:

* `--json`
* `--all` (vollständige Diagnose; schreibgeschützt, zum Einfügen geeignet)
* `--deep` (Kanäle prüfen)
* `--usage` (Nutzung/Kontingent des Modellanbieters anzeigen)
* `--timeout <ms>`
* `--verbose`
* `--debug` (Alias für `--verbose`)

Hinweise:

* Übersicht enthält, falls verfügbar, den Status des Gateway- und Knoten-Hostdienstes.

<div id="usage-tracking">
  ### Nutzungs-Tracking
</div>

OpenClaw kann die Nutzung bzw. Kontingente von Anbietern anzeigen, wenn OAuth/API-Credentials verfügbar sind.

Anzeigestellen:

* `/status` (fügt, falls verfügbar, eine kurze Zeile zur Anbieternutzung hinzu)
* `openclaw status --usage` (gibt eine vollständige Aufschlüsselung nach Anbietern aus)
* macOS-Menüleiste (Bereich „Usage“ unter „Context“)

Hinweise:

* Die Daten stammen direkt von den Usage-Endpoints der Anbieter (keine Schätzungen).
* Anbieter: Anthropic, GitHub Copilot, OpenAI Codex OAuth sowie Gemini CLI/Antigravity, wenn die zugehörigen Anbieter-Plugins aktiviert sind.
* Wenn keine passenden Zugangsdaten existieren, wird die Nutzung nicht angezeigt.
* Details: siehe [Nutzungs-Tracking](/de/concepts/usage-tracking).

<div id="health">
  ### `health`
</div>

Ruft den Health-Status des laufenden Gateways ab.

Optionen:

* `--json`
* `--timeout <ms>`
* `--verbose`

<div id="sessions">
  ### `sessions`
</div>

Listet gespeicherte Sitzungen auf.

Optionen:

* `--json`
* `--verbose`
* `--store <path>`
* `--active <minutes>`

<div id="reset-uninstall">
  ## Reset / Deinstallation
</div>

<div id="reset">
  ### `reset`
</div>

Setzt lokale Konfiguration und Zustand zurück (CLI bleibt installiert).

Optionen:

* `--scope <config|config+creds+sessions|full>`
* `--yes`
* `--non-interactive`
* `--dry-run`

Hinweise:

* `--non-interactive` erfordert `--scope` und `--yes`.

<div id="uninstall">
  ### `uninstall`
</div>

Deinstalliert den Gateway-Dienst und löscht lokale Daten (CLI bleibt erhalten).

Optionen:

* `--service`
* `--state`
* `--workspace`
* `--app`
* `--all`
* `--yes`
* `--non-interactive`
* `--dry-run`

Hinweise:

* `--non-interactive` setzt `--yes` und explizite Scopes (oder `--all`) voraus.

## Gateway

<div id="gateway">
  ### `gateway`
</div>

Startet das WebSocket-Gateway.

Optionen:

* `--port <port>`
* `--bind <loopback|tailnet|lan|auto|custom>`
* `--token <token>`
* `--auth <token|password>`
* `--password <password>`
* `--tailscale <off|serve|funnel>`
* `--tailscale-reset-on-exit`
* `--allow-unconfigured`
* `--dev`
* `--reset` (setzt Dev-Konfiguration, Anmeldedaten, Sitzungen und Arbeitsbereich zurück)
* `--force` (beendet einen vorhandenen Listener auf dem Port)
* `--verbose`
* `--claude-cli-logs`
* `--ws-log <auto|full|compact>`
* `--compact` (Alias für `--ws-log compact`)
* `--raw-stream`
* `--raw-stream-path <path>`

<div id="gateway-service">
  ### `gateway service`
</div>

Verwalten Sie den Gateway-Dienst (launchd/systemd/schtasks).

Unterbefehle:

* `gateway status` (fragt standardmäßig das Gateway-RPC ab)
* `gateway install` (Dienstinstallation)
* `gateway uninstall`
* `gateway start`
* `gateway stop`
* `gateway restart`

Hinweise:

* `gateway status` fragt standardmäßig das Gateway-RPC über den vom Dienst aufgelösten Port bzw. die aufgelöste Konfiguration ab (kann mit `--url/--token/--password` überschrieben werden).
* `gateway status` unterstützt `--no-probe`, `--deep` und `--json` für Scripting.
* `gateway status` zeigt außerdem veraltete oder zusätzliche Gateway-Dienste an, wenn sie erkannt werden können (`--deep` führt zusätzliche System-Scans aus). OpenClaw-Dienste mit Profilnamen werden als erstklassig behandelt und nicht als „extra“ markiert.
* `gateway status` gibt aus, welchen Konfigurationspfad die CLI verwendet und welche Konfiguration der Dienst voraussichtlich verwendet (Dienst-Umgebung), sowie die aufgelöste Ziel-URL der Abfrage.
* `gateway install|uninstall|start|stop|restart` unterstützen `--json` für Scripting (die Standardausgabe bleibt menschenlesbar).
* `gateway install` nutzt standardmäßig die Node.js-Laufzeitumgebung; bun wird **nicht empfohlen** (Bugs mit WhatsApp/Telegram).
* `gateway install`-Optionen: `--port`, `--runtime`, `--token`, `--force`, `--json`.

<div id="logs">
  ### `logs`
</div>

Gateway-Logdateien per RPC „tailen“.

Hinweise:

* TTY-Sitzungen rendern eine farbig formatierte, strukturierte Ansicht; Nicht-TTY-Sitzungen fallen auf Nur-Text zurück.
* `--json` gibt zeilengetrenntes JSON aus (ein Log-Ereignis pro Zeile).

Beispiele:

```bash
openclaw logs --follow
openclaw logs --limit 200
openclaw logs --plain
openclaw logs --json
openclaw logs --no-color
```

<div id="gateway-subcommand">
  ### `gateway <subcommand>`
</div>

Gateway-CLI-Hilfsbefehle (verwende `--url`, `--token`, `--password`, `--timeout`, `--expect-final` für RPC-Subcommands).

Subcommands:

* `gateway call <method> [--params <json>]`
* `gateway health`
* `gateway status`
* `gateway probe`
* `gateway discover`
* `gateway install|uninstall|start|stop|restart`
* `gateway run`

Häufig verwendete RPCs:

* `config.apply` (Konfiguration prüfen + schreiben + neu starten + aufwecken)
* `config.patch` (teilweises Update zusammenführen + neu starten + aufwecken)
* `update.run` (Update ausführen + neu starten + aufwecken)

Tipp: Wenn du `config.set`/`config.apply`/`config.patch` direkt aufrufst, übergib `baseHash` aus
`config.get`, falls bereits eine Konfiguration existiert.

<div id="models">
  ## Modelle
</div>

Siehe [/concepts/models](/de/concepts/models) für Fallback-Verhalten und Scanstrategie.

Bevorzugte Anthropic-Authentifizierung (setup-token):

```bash
claude setup-token
openclaw models auth setup-token --provider anthropic
openclaw models status
```

<div id="models-root">
  ### `models` (Root-Ebene)
</div>

`openclaw models` ist ein Alias für `models status`.

Optionen auf Root-Ebene:

* `--status-json` (Alias für `models status --json`)
* `--status-plain` (Alias für `models status --plain`)

<div id="models-list">
  ### `models list`
</div>

Optionen:

* `--all`
* `--local`
* `--provider <name>`
* `--json`
* `--plain`

<div id="models-status">
  ### `models status`
</div>

Optionen:

* `--json`
* `--plain`
* `--check` (Exit-Code 1=abgelaufen/fehlend, 2=läuft bald ab)
* `--probe` (Live-Prüfung der konfigurierten Auth-Profile)
* `--probe-provider <name>`
* `--probe-profile <id>` (wiederholbar oder kommasepariert)
* `--probe-timeout <ms>`
* `--probe-concurrency <n>`
* `--probe-max-tokens <n>`

Enthält immer die Auth-Übersicht und den OAuth-Ablaufstatus für Profile im Auth-Store.
`--probe` führt Live-Requests aus (kann Token verbrauchen und Rate Limits auslösen).

<div id="models-set-model">
  ### `models set <model>`
</div>

Setzt `agents.defaults.model.primary`.

<div id="models-set-image-model">
  ### `models set-image <model>`
</div>

Setzt `agents.defaults.imageModel.primary`.

<div id="models-aliases-listaddremove">
  ### `models aliases list|add|remove`
</div>

Optionen:

* `list`: `--json`, `--plain`
* `add <alias> <model>`
* `remove <alias>`

<div id="models-fallbacks-listaddremoveclear">
  ### `models fallbacks list|add|remove|clear`
</div>

Optionen:

* `list`: `--json`, `--plain`
* `add <model>`
* `remove <model>`
* `clear`

<div id="models-image-fallbacks-listaddremoveclear">
  ### `models image-fallbacks list|add|remove|clear`
</div>

Optionen:

* `list`: `--json`, `--plain`
* `add <model>`
* `remove <model>`
* `clear`

<div id="models-scan">
  ### `models scan`
</div>

Optionen:

* `--min-params <b>`
* `--max-age-days <days>`
* `--provider <name>`
* `--max-candidates <n>`
* `--timeout <ms>`
* `--concurrency <n>`
* `--no-probe`
* `--yes`
* `--no-input`
* `--set-default`
* `--set-image`
* `--json`

<div id="models-auth-addsetup-tokenpaste-token">
  ### `models auth add|setup-token|paste-token`
</div>

Optionen:

* `add`: interaktiver Authentifizierungsassistent
* `setup-token`: `--provider <name>` (Standard: `anthropic`), `--yes`
* `paste-token`: `--provider <name>`, `--profile-id <id>`, `--expires-in <duration>`

<div id="models-auth-order-getsetclear">
  ### `models auth order get|set|clear`
</div>

Optionen:

* `get`: `--provider <name>`, `--agent <id>`, `--json`
* `set`: `--provider <name>`, `--agent <id>`, `<profileIds...>`
* `clear`: `--provider <name>`, `--agent <id>`

<div id="system">
  ## System
</div>

<div id="system-event">
  ### `system event`
</div>

Reiht ein Systemereignis in die Warteschlange ein und löst optional einen Herzschlag aus (Gateway-RPC).

Erforderlich:

* `--text <text>`

Optionen:

* `--mode <now|next-heartbeat>`
* `--json`
* `--url`, `--token`, `--timeout`, `--expect-final`

<div id="system-heartbeat-lastenabledisable">
  ### `system heartbeat last|enable|disable`
</div>

Steuerung des Herzschlags (Gateway-RPC).

Optionen:

* `--json`
* `--url`, `--token`, `--timeout`, `--expect-final`

<div id="system-presence">
  ### `system presence`
</div>

Listet System-Presence-Einträge auf (Gateway-RPC).

Optionen:

* `--json`
* `--url`, `--token`, `--timeout`, `--expect-final`

<div id="cron">
  ## Cron
</div>

Geplante Jobs verwalten (Gateway-RPC). Siehe [/automation/cron-jobs](/de/automation/cron-jobs).

Unterbefehle:

* `cron status [--json]`
* `cron list [--all] [--json]` (standardmäßig Tabellenausgabe; `--json` für Rohdaten verwenden)
* `cron add` (Alias: `create`; erfordert `--name` und genau eines von `--at` | `--every` | `--cron` sowie genau eine Nutzlast von `--system-event` | `--message`)
* `cron edit <id>` (Felder per Patch ändern)
* `cron rm <id>` (Aliase: `remove`, `delete`)
* `cron enable <id>`
* `cron disable <id>`
* `cron runs --id <id> [--limit <n>]`
* `cron run <id> [--force]`

Alle `cron`-Befehle unterstützen `--url`, `--token`, `--timeout`, `--expect-final`.

<div id="node-host">
  ## Knoten-Host
</div>

`node` startet einen **headless Knoten-Host** oder verwaltet ihn als Hintergrunddienst. Siehe
[`openclaw node`](/de/cli/node).

Unterbefehle:

* `node run --host <gateway-host> --port 18789`
* `node status`
* `node install [--host <gateway-host>] [--port <port>] [--tls] [--tls-fingerprint <sha256>] [--node-id <id>] [--display-name <name>] [--runtime <node|bun>] [--force]`
* `node uninstall`
* `node stop`
* `node restart`

<div id="nodes">
  ## Knoten
</div>

`nodes` kommuniziert mit dem Gateway und steuert gekoppelte Knoten an. Siehe [/nodes](/de/nodes).

Gängige Optionen:

* `--url`, `--token`, `--timeout`, `--json`

Unterbefehle:

* `nodes status [--connected] [--last-connected <duration>]`
* `nodes describe --node <id|name|ip>`
* `nodes list [--connected] [--last-connected <duration>]`
* `nodes pending`
* `nodes approve <requestId>`
* `nodes reject <requestId>`
* `nodes rename --node <id|name|ip> --name <displayName>`
* `nodes invoke --node <id|name|ip> --command <command> [--params <json>] [--invoke-timeout <ms>] [--idempotency-key <key>]`
* `nodes run --node <id|name|ip> [--cwd <path>] [--env KEY=VAL] [--command-timeout <ms>] [--needs-screen-recording] [--invoke-timeout <ms>] <command...>` (Mac-Node oder Headless-Node-Host)
* `nodes notify --node <id|name|ip> [--title <text>] [--body <text>] [--sound <name>] [--priority <passive|active|timeSensitive>] [--delivery <system|overlay|auto>] [--invoke-timeout <ms>]` (nur Mac)

Kamera:

* `nodes camera list --node <id|name|ip>`
* `nodes camera snap --node <id|name|ip> [--facing front|back|both] [--device-id <id>] [--max-width <px>] [--quality <0-1>] [--delay-ms <ms>] [--invoke-timeout <ms>]`
* `nodes camera clip --node <id|name|ip> [--facing front|back] [--device-id <id>] [--duration <ms|10s|1m>] [--no-audio] [--invoke-timeout <ms>]`

Canvas + Bildschirm:

* `nodes canvas snapshot --node <id|name|ip> [--format png|jpg|jpeg] [--max-width <px>] [--quality <0-1>] [--invoke-timeout <ms>]`
* `nodes canvas present --node <id|name|ip> [--target <urlOrPath>] [--x <px>] [--y <px>] [--width <px>] [--height <px>] [--invoke-timeout <ms>]`
* `nodes canvas hide --node <id|name|ip> [--invoke-timeout <ms>]`
* `nodes canvas navigate <url> --node <id|name|ip> [--invoke-timeout <ms>]`
* `nodes canvas eval [<js>] --node <id|name|ip> [--js <code>] [--invoke-timeout <ms>]`
* `nodes canvas a2ui push --node <id|name|ip> (--jsonl <path> | --text <text>) [--invoke-timeout <ms>]`
* `nodes canvas a2ui reset --node <id|name|ip> [--invoke-timeout <ms>]`
* `nodes screen record --node <id|name|ip> [--screen <index>] [--duration <ms|10s>] [--fps <n>] [--no-audio] [--out <path>] [--invoke-timeout <ms>]`

Standort:

* `nodes location get --node <id|name|ip> [--max-age <ms>] [--accuracy <coarse|balanced|precise>] [--location-timeout <ms>] [--invoke-timeout <ms>]`

<div id="browser">
  ## Browser
</div>

CLI zur Browser-Steuerung (dedizierte Chrome/Brave/Edge/Chromium-Instanz). Siehe [`openclaw browser`](/de/cli/browser) und das [Browser-Tool](/de/tools/browser).

Häufige Optionen:

* `--url`, `--token`, `--timeout`, `--json`
* `--browser-profile <name>`

Verwalten:

* `browser status`
* `browser start`
* `browser stop`
* `browser reset-profile`
* `browser tabs`
* `browser open <url>`
* `browser focus <targetId>`
* `browser close [targetId]`
* `browser profiles`
* `browser create-profile --name <name> [--color <hex>] [--cdp-url <url>]`
* `browser delete-profile --name <name>`

Untersuchen:

* `browser screenshot [targetId] [--full-page] [--ref <ref>] [--element <selector>] [--type png|jpeg]`
* `browser snapshot [--format aria|ai] [--target-id <id>] [--limit <n>] [--interactive] [--compact] [--depth <n>] [--selector <sel>] [--out <path>]`

Aktionen:

* `browser navigate <url> [--target-id <id>]`
* `browser resize <width> <height> [--target-id <id>]`
* `browser click <ref> [--double] [--button <left|right|middle>] [--modifiers <csv>] [--target-id <id>]`
* `browser type <ref> <text> [--submit] [--slowly] [--target-id <id>]`
* `browser press <key> [--target-id <id>]`
* `browser hover <ref> [--target-id <id>]`
* `browser drag <startRef> <endRef> [--target-id <id>]`
* `browser select <ref> <values...> [--target-id <id>]`
* `browser upload <paths...> [--ref <ref>] [--input-ref <ref>] [--element <selector>] [--target-id <id>] [--timeout-ms <ms>]`
* `browser fill [--fields <json>] [--fields-file <path>] [--target-id <id>]`
* `browser dialog --accept|--dismiss [--prompt <text>] [--target-id <id>] [--timeout-ms <ms>]`
* `browser wait [--time <ms>] [--text <value>] [--text-gone <value>] [--target-id <id>]`
* `browser evaluate --fn <code> [--ref <ref>] [--target-id <id>]`
* `browser console [--level <error|warn|info>] [--target-id <id>]`
* `browser pdf [--target-id <id>]`

<div id="docs-search">
  ## Suche in der Dokumentation
</div>

<div id="docs-query">
  ### `docs [query...]`
</div>

Durchsucht den Live-Dokumentationsindex.

## TUI

<div id="tui">
  ### `tui`
</div>

Öffne die Terminal-UI, die mit dem Gateway verbunden ist.

Optionen:

* `--url <url>`
* `--token <token>`
* `--password <password>`
* `--session <key>`
* `--deliver`
* `--thinking <level>`
* `--message <text>`
* `--timeout-ms <ms>` (Standardwert ist `agents.defaults.timeoutSeconds`)
* `--history-limit <n>`