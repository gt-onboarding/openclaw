---
title: Proxy de API de Claude Max
summary: "Usa la suscripción Claude Max/Pro como un endpoint de API compatible con OpenAI"
read_when:
  - Quieres usar la suscripción Claude Max con herramientas compatibles con OpenAI
  - Quieres un servidor de API local que envuelva la CLI de Claude Code
  - Quieres ahorrar dinero usando la suscripción en lugar de claves de API
---

<div id="claude-max-api-proxy">
  # Proxy de la API de Claude Max
</div>

**claude-max-api-proxy** es una herramienta comunitaria que expone tu suscripción de Claude Max/Pro como un endpoint de API compatible con OpenAI. Esto te permite usar tu suscripción con cualquier herramienta que admita el formato de la API de OpenAI.

<div id="why-use-this">
  ## ¿Por qué usar esto?
</div>

| Enfoque | Costo | Mejor para |
|----------|------|----------|
| Anthropic API | Pago por token (~15 USD/M de entrada, 75 USD/M de salida para Opus) | Aplicaciones en producción, grandes volúmenes |
| Suscripción a Claude Max | 200 USD/mes fijo | Uso personal, desarrollo, uso ilimitado |

Si tienes una suscripción a Claude Max y quieres usarla con herramientas compatibles con OpenAI, este proxy puede ahorrarte una cantidad considerable de dinero.

<div id="how-it-works">
  ## Cómo funciona
</div>

```
Tu App → claude-max-api-proxy → Claude Code CLI → Anthropic (vía suscripción)
     (formato OpenAI)             (convierte el formato)    (usa tu inicio de sesión)
```

El proxy:

1. Acepta solicitudes en formato OpenAI en `http://localhost:3456/v1/chat/completions`
2. Las convierte en comandos de la CLI de Claude Code
3. Devuelve respuestas en formato OpenAI (admite streaming)

<div id="installation">
  ## Instalación
</div>

```bash
# Requiere Node.js 20+ y la CLI de Claude Code
npm install -g claude-max-api-proxy

# Verify Claude CLI is authenticated
claude --version
```

<div id="usage">
  ## Uso
</div>

<div id="start-the-server">
  ### Iniciar el servidor
</div>

```bash
claude-max-api
# El servidor se ejecuta en http://localhost:3456
```

<div id="test-it">
  ### Probarlo
</div>

```bash
# Comprobación de estado
curl http://localhost:3456/health

# Listar modelos
curl http://localhost:3456/v1/models

# Completado de chat
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

Puedes configurar OpenClaw para usar el proxy como un endpoint personalizado compatible con OpenAI:

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
  ## Modelos disponibles
</div>

| ID del modelo | Corresponde a |
|----------|---------|
| `claude-opus-4` | Claude Opus 4 |
| `claude-sonnet-4` | Claude Sonnet 4 |
| `claude-haiku-4` | Claude Haiku 4 |

<div id="auto-start-on-macos">
  ## Inicio automático en macOS
</div>

Configura un LaunchAgent para que el proxy se ejecute automáticamente:

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
  ## Enlaces
</div>

* **npm:** https://www.npmjs.com/package/claude-max-api-proxy
* **GitHub:** https://github.com/atalovesyou/claude-max-api-proxy
* **Incidencias:** https://github.com/atalovesyou/claude-max-api-proxy/issues

<div id="notes">
  ## Notas
</div>

* Esta es una **herramienta comunitaria**, no está oficialmente respaldada por Anthropic ni por OpenClaw
* Requiere una suscripción activa a Claude Max/Pro con Claude Code CLI autenticado
* El proxy se ejecuta localmente y no envía datos a ningún servidor de terceros
* Las respuestas en streaming están totalmente admitidas

<div id="see-also">
  ## Véase también
</div>

* [Proveedor Anthropic](/es/providers/anthropic) - Integración nativa de OpenClaw con token de configuración de Claude o claves de api
* [Proveedor OpenAI](/es/providers/openai) - Para suscripciones a OpenAI/Codex