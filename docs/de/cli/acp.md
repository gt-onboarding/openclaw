---
title: Acp
summary: "Ausführen der ACP-Bridge für IDE-Integrationen"
read_when:
  - ACP-basierte IDE-Integrationen einrichten
  - Routing von ACP-Sitzungen zum Gateway debuggen
---

<div id="acp">
  # acp
</div>

Startet die ACP (Agent Client Protocol) Bridge, die mit einem OpenClaw Gateway kommuniziert.

Dieser Befehl spricht ACP über stdio für IDEs und leitet Prompts über WebSocket an das Gateway weiter. Er hält eine Zuordnung zwischen ACP-Sitzungen und Gateway-Sitzungsschlüsseln aufrecht.

<div id="usage">
  ## Verwendung
</div>

```bash
openclaw acp

# Remote Gateway
openclaw acp --url wss://gateway-host:18789 --token <token>

# Attach to an existing session key
openclaw acp --session agent:main:main

# Attach by label (must already exist)
openclaw acp --session-label "support inbox"

# Sitzungsschlüssel vor dem ersten Prompt zurücksetzen
openclaw acp --session agent:main:main --reset-session
```

<div id="acp-client-debug">
  ## ACP-Client (Debug)
</div>

Verwende den integrierten ACP-Client, um die Bridge ohne IDE schnell zu überprüfen.
Er startet die ACP-Bridge und ermöglicht dir, Prompts interaktiv einzugeben.

```bash
openclaw acp client

# Die erzeugte Bridge auf ein entferntes Gateway richten
openclaw acp client --server-args --url wss://gateway-host:18789 --token <token>

# Server-Befehl überschreiben (Standard: openclaw)
openclaw acp client --server "node" --server-args openclaw.mjs acp --url ws://127.0.0.1:19001
```

<div id="usage">
  ## Verwendung
</div>

Verwenden Sie ACP, wenn eine IDE (oder ein anderer Client) das Agent Client Protocol spricht und Sie sie zum Steuern einer OpenClaw Gateway-Sitzung verwenden möchten.

1. Stellen Sie sicher, dass das Gateway läuft (lokal oder remote).
2. Konfigurieren Sie das Gateway-Ziel (Konfiguration oder Flags).
3. Weisen Sie Ihre IDE an, `openclaw acp` über stdio auszuführen.

Beispielkonfiguration (dauerhaft gespeichert):

```bash
openclaw config set gateway.remote.url wss://gateway-host:18789
openclaw config set gateway.remote.token <token>
```

Beispiel für eine Direktausführung (ohne Schreiben der Konfiguration):

```bash
openclaw acp --url wss://gateway-host:18789 --token <token>
```

<div id="selecting-agents">
  ## Auswahl von Agenten
</div>

ACP wählt Agenten nicht direkt aus. Es routet über den Gateway-Sitzungsschlüssel.

Verwende agent-spezifische Sitzungsschlüssel, um einen bestimmten Agenten anzusprechen:

```bash
openclaw acp --session agent:main:main
openclaw acp --session agent:design:main
openclaw acp --session agent:qa:bug-123
```

Jede ACP-Sitzung wird einem einzelnen Gateway-Sitzungsschlüssel zugeordnet. Ein Agent kann mehrere Sitzungen haben; standardmäßig verwendet ACP eine isolierte `acp:&lt;uuid&gt;`-Sitzung, sofern du den Schlüssel oder das Label nicht überschreibst.

<div id="zed-editor-setup">
  ## Zed-Editor einrichten
</div>

Füge einen benutzerdefinierten ACP-Agenten in `~/.config/zed/settings.json` hinzu (oder verwende die Zed-Einstellungs-UI):

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": ["acp"],
      "env": {}
    }
  }
}
```

Um ein bestimmtes Gateway oder einen bestimmten agent zu adressieren:

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": [
        "acp",
        "--url", "wss://gateway-host:18789",
        "--token", "<token>",
        "--session", "agent:design:main"
      ],
      "env": {}
    }
  }
}
```

Öffne in Zed das Agent-Panel und wähle im Agent-Panel „OpenClaw ACP“ aus, um einen Thread zu starten.

<div id="session-mapping">
  ## Sitzungszuordnung
</div>

Standardmäßig erhalten ACP-Sitzungen einen eigenen Gateway-Sitzungsschlüssel mit dem Präfix `acp:`.
Um eine bekannte Sitzung wiederzuverwenden, übergib einen Sitzungsschlüssel oder ein Label:

* `--session <key>`: verwendet einen bestimmten Gateway-Sitzungsschlüssel.
* `--session-label <label>`: löst eine bestehende Sitzung anhand des Labels auf.
* `--reset-session`: erzeugt eine neue Sitzungs-ID für diesen Schlüssel (gleicher Schlüssel, neuer Verlauf).

Wenn dein ACP-Client Metadaten unterstützt, kannst du dies pro Sitzung überschreiben:

```json
{
  "_meta": {
    "sessionKey": "agent:main:main",
    "sessionLabel": "Support-Posteingang",
    "resetSession": true
  }
}
```

Weitere Informationen zu Sitzungsschlüsseln findest du unter [/concepts/session](/de/concepts/session).

<div id="options">
  ## Optionen
</div>

* `--url <url>`: Gateway-WebSocket-URL (standardmäßig gateway.remote.url, falls konfiguriert).
* `--token <token>`: Gateway-Auth-Token.
* `--password <password>`: Gateway-Auth-Passwort.
* `--session <key>`: Standard-Sitzungsschlüssel.
* `--session-label <label>`: Standard-Sitzungslabel zur Auflösung.
* `--require-existing`: Schlägt fehl, wenn Sitzungsschlüssel oder Sitzungslabel nicht existieren.
* `--reset-session`: Sitzungsschlüssel vor der ersten Verwendung zurücksetzen.
* `--no-prefix-cwd`: Arbeitsverzeichnis nicht vor Prompts voranstellen.
* `--verbose, -v`: Ausführliche Protokollierung auf stderr.

<div id="acp-client-options">
  ### `acp client` Optionen
</div>

* `--cwd <dir>`: Arbeitsverzeichnis für die ACP-Sitzung.
* `--server <command>`: ACP-Server-Befehl (Standard: `openclaw`).
* `--server-args <args...>`: zusätzliche Argumente, die an den ACP-Server übergeben werden.
* `--server-verbose`: aktiviert ausführliches Logging auf dem ACP-Server.
* `--verbose, -v`: aktiviert ausführliches Client-Logging.