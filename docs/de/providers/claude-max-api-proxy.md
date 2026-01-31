---
title: Claude Max API-Proxy
summary: "Verwende dein Claude-Max/Pro-Abonnement als OpenAI-kompatiblen API-Endpunkt"
read_when:
  - Du möchtest ein Claude-Max-Abonnement mit OpenAI-kompatiblen Tools verwenden
  - Du möchtest einen lokalen API-Server, der die Claude Code CLI kapselt
  - Du möchtest Geld sparen, indem du ein Abonnement anstelle von API-Schlüsseln verwendest
---

<div id="claude-max-api-proxy">
  # Claude Max API-Proxy
</div>

**claude-max-api-proxy** ist ein Community-Tool, das dein Claude Max/Pro-Abonnement über einen OpenAI-kompatiblen API-Endpunkt verfügbar macht. Damit kannst du dein Abonnement mit jedem Tool verwenden, das das OpenAI API-Format unterstützt.

<div id="why-use-this">
  ## Warum sollte ich das verwenden?
</div>

| Ansatz | Kosten | Am besten geeignet für |
|----------|------|------------------------|
| Anthropic API | Abrechnung pro Token (~15 $/M Eingabe, 75 $/M Ausgabe für Opus) | Produktionsanwendungen, hohes Anfragevolumen |
| Claude Max-Abonnement | 200 $/Monat pauschal | Persönliche Nutzung, Entwicklung, unbegrenzte Verwendung |

Wenn du ein Claude Max-Abonnement hast und es mit OpenAI-kompatiblen Tools nutzen möchtest, kann dir dieser Proxy eine Menge Geld sparen.

<div id="how-it-works">
  ## So funktioniert es
</div>

```
Ihre App → claude-max-api-proxy → Claude Code CLI → Anthropic (via Abonnement)
     (OpenAI-Format)              (konvertiert Format)   (verwendet Ihr Login)
```

Der Proxy:

1. Akzeptiert Anfragen im OpenAI-Format unter `http://localhost:3456/v1/chat/completions`
2. Konvertiert sie in Claude Code CLI-Befehle
3. Gibt Antworten im OpenAI-Format zurück (Streaming wird unterstützt)

<div id="installation">
  ## Installation
</div>

```bash
# Requires Node.js 20+ and Claude Code CLI
npm install -g claude-max-api-proxy

# Überprüfen Sie, ob Claude CLI authentifiziert ist
claude --version
```

<div id="usage">
  ## Verwendung
</div>

<div id="start-the-server">
  ### Server starten
</div>

```bash
claude-max-api
# Server läuft auf http://localhost:3456
```

<div id="test-it">
  ### Testen
</div>

```bash
# Health-Check
curl http://localhost:3456/health

# Modelle auflisten
curl http://localhost:3456/v1/models

# Chat-Completion
curl http://localhost:3456/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-opus-4",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

<div id="with-openclaw">
  ### Mit OpenClaw
</div>

Du kannst OpenClaw so konfigurieren, dass es den Proxy als benutzerdefinierten, OpenAI-kompatiblen Endpunkt verwendet:

```json5
{
  env: {
    OPENAI_API_KEY: "not-needed",
    OPENAI_BASE_URL: "http://localhost:3456/v1"
  },
  agents: {
    defaults: {
      model: { primary: "openai/claude-opus-4" }
    }
  }
}
```

<div id="available-models">
  ## Verfügbare Modelle
</div>

| Modell-ID | Abbildung auf |
|----------|---------|
| `claude-opus-4` | Claude Opus 4 |
| `claude-sonnet-4` | Claude Sonnet 4 |
| `claude-haiku-4` | Claude Haiku 4 |

<div id="auto-start-on-macos">
  ## Automatischer Start unter macOS
</div>

Erstellen Sie einen LaunchAgent, um den Proxy automatisch zu starten:

```bash
cat > ~/Library/LaunchAgents/com.claude-max-api.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>com.claude-max-api</string>
  <key>RunAtLoad</key>
  <true/>
  <key>KeepAlive</key>
  <true/>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/bin/node</string>
    <string>/usr/local/lib/node_modules/claude-max-api-proxy/dist/server/standalone.js</string>
  </array>
  <key>EnvironmentVariables</key>
  <dict>
    <key>PATH</key>
    <string>/usr/local/bin:/opt/homebrew/bin:~/.local/bin:/usr/bin:/bin</string>
  </dict>
</dict>
</plist>
EOF

launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.claude-max-api.plist
```

<div id="links">
  ## Links
</div>

* **npm:** https://www.npmjs.com/package/claude-max-api-proxy
* **GitHub:** https://github.com/atalovesyou/claude-max-api-proxy
* **Issues:** https://github.com/atalovesyou/claude-max-api-proxy/issues

<div id="notes">
  ## Hinweise
</div>

* Dies ist ein **Community-Tool**, das nicht offiziell von Anthropic oder OpenClaw unterstützt wird
* Erfordert ein aktives Claude Max/Pro-Abonnement mit authentifizierter Claude Code CLI
* Der Proxy läuft lokal und überträgt keine Daten an Server von Drittanbietern
* Streaming-Antworten werden vollständig unterstützt

<div id="see-also">
  ## Siehe auch
</div>

* [Anthropic Anbieter](/de/providers/anthropic) - Native OpenClaw-Integration mit Claude-Setup-Token oder API-Schlüsseln
* [OpenAI Anbieter](/de/providers/openai) - Für OpenAI-/Codex-Abonnements