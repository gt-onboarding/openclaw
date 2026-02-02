---
title: Proxy API Claude Max
summary: "Usa il tuo abbonamento Claude Max/Pro come endpoint API compatibile con OpenAI"
read_when:
  - Vuoi usare l'abbonamento Claude Max con strumenti compatibili con OpenAI
  - Vuoi un server API locale che faccia da wrapper per la CLI Claude Code
  - Vuoi risparmiare sui costi usando l'abbonamento anziché le chiavi API
---

<div id="claude-max-api-proxy">
  # Claude Max API Proxy
</div>

**claude-max-api-proxy** è uno strumento sviluppato dalla community che espone il tuo abbonamento Claude Max/Pro tramite un endpoint API compatibile con OpenAI. Questo ti consente di usare il tuo abbonamento con qualsiasi strumento che supporti il formato dell&#39;API OpenAI.

<div id="why-use-this">
  ## Perché usarlo?
</div>

| Approccio | Costo | Ideale per |
|----------|------|----------|
| Anthropic API | Pay per token (~$15/M input, $75/M output per Opus) | Applicazioni in produzione, alti volumi |
| Abbonamento Claude Max | $200/mese fisso | Uso personale, sviluppo, utilizzo illimitato |

Se hai un abbonamento Claude Max e vuoi usarlo con strumenti compatibili con OpenAI, questo proxy può farti risparmiare notevolmente sui costi.

<div id="how-it-works">
  ## Come funziona
</div>

```
La tua App → claude-max-api-proxy → Claude Code CLI → Anthropic (tramite abbonamento)
     (formato OpenAI)             (converte il formato)  (usa le tue credenziali)
```

Il proxy:

1. Accetta richieste nel formato OpenAI all&#39;indirizzo `http://localhost:3456/v1/chat/completions`
2. Le converte in comandi CLI di Claude Code
3. Restituisce risposte nel formato OpenAI (con supporto per lo streaming)

<div id="installation">
  ## Installazione
</div>

```bash
# Richiede Node.js 20+ e la CLI di Claude Code
npm install -g claude-max-api-proxy

# Verify Claude CLI is authenticated
claude --version
```

<div id="usage">
  ## Utilizzo
</div>

<div id="start-the-server">
  ### Avviare il server
</div>

```bash
claude-max-api
# Il server viene eseguito su http://localhost:3456
```

<div id="test-it">
  ### Provalo
</div>

```bash
# Health check
curl http://localhost:3456/health

# List models
curl http://localhost:3456/v1/models

# Completamento della chat
curl http://localhost:3456/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-opus-4",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

<div id="with-openclaw">
  ### Con OpenClaw
</div>

Puoi configurare OpenClaw per usare il proxy come endpoint personalizzato compatibile con OpenAI:

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
  ## Modelli disponibili
</div>

| Model ID | Corrisponde a |
|----------|---------|
| `claude-opus-4` | Claude Opus 4 |
| `claude-sonnet-4` | Claude Sonnet 4 |
| `claude-haiku-4` | Claude Haiku 4 |

<div id="auto-start-on-macos">
  ## Avvio automatico su macOS
</div>

Crea un LaunchAgent per eseguire automaticamente il proxy:

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
  ## Link
</div>

* **npm:** https://www.npmjs.com/package/claude-max-api-proxy
* **GitHub:** https://github.com/atalovesyou/claude-max-api-proxy
* **Issue:** https://github.com/atalovesyou/claude-max-api-proxy/issues

<div id="notes">
  ## Note
</div>

* Questo è uno strumento della **community**, non ufficialmente supportato da Anthropic o OpenClaw
* Richiede un abbonamento attivo a Claude Max/Pro con Claude Code CLI autenticata
* Il proxy viene eseguito in locale e non invia dati a server di terze parti
* Le risposte in streaming sono pienamente supportate

<div id="see-also">
  ## Vedi anche
</div>

* [Provider Anthropic](/it/providers/anthropic) - Integrazione nativa di OpenClaw con Claude tramite setup token o chiavi API
* [Provider OpenAI](/it/providers/openai) - Per gli abbonamenti OpenAI/Codex