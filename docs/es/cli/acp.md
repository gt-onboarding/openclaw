---
title: Acp
summary: "Ejecutar el puente ACP para integraciones con IDE"
read_when:
  - Configurar integraciones con IDE basadas en ACP
  - Depurar el enrutamiento de sesiones ACP hacia el Gateway
---

<div id="acp">
  # acp
</div>

Ejecuta el puente ACP (Agent Client Protocol) que se comunica con el Gateway de OpenClaw.

Este comando habla ACP sobre `stdio` para IDE y reenvía los prompts al Gateway
a través de WebSocket. Mantiene las sesiones ACP asociadas a claves de sesión del Gateway.

<div id="usage">
  ## Uso
</div>

```bash
openclaw acp

# Remote Gateway
openclaw acp --url wss://gateway-host:18789 --token <token>

# Attach to an existing session key
openclaw acp --session agent:main:main

# Attach by label (must already exist)
openclaw acp --session-label "support inbox"

# Restablecer la clave de sesión antes del primer prompt
openclaw acp --session agent:main:main --reset-session
```

<div id="acp-client-debug">
  ## Cliente ACP (depuración)
</div>

Usa el cliente ACP integrado para hacer una comprobación rápida del puente ACP sin un IDE.
Inicia el puente ACP y te permite escribir prompts de forma interactiva.

```bash
openclaw acp client

# Point the spawned bridge at a remote Gateway
openclaw acp client --server-args --url wss://gateway-host:18789 --token <token>

# Sobrescribe el comando del servidor (predeterminado: openclaw)
openclaw acp client --server "node" --server-args openclaw.mjs acp --url ws://127.0.0.1:19001
```

<div id="how-to-use-this">
  ## Cómo usar esto
</div>

Usa ACP cuando un IDE (u otro cliente) sea compatible con Agent Client Protocol y quieras
que controle una sesión del Gateway de OpenClaw.

1. Asegúrate de que el Gateway se esté ejecutando (local o remoto).
2. Configura el destino del Gateway (configuración o flags).
3. Configura tu IDE para ejecutar `openclaw acp` a través de stdio.

Ejemplo de configuración (persistente):

```bash
openclaw config set gateway.remote.url wss://gateway-host:18789
openclaw config set gateway.remote.token <token>
```

Ejemplo de ejecución directa (sin guardar la configuración):

```bash
openclaw acp --url wss://gateway-host:18789 --token <token>
```

<div id="selecting-agents">
  ## Selección de agentes
</div>

ACP no selecciona agentes directamente. Realiza el enrutamiento por la clave de sesión del Gateway.

Utiliza claves de sesión con ámbito de agente para dirigirte a un agente específico:

```bash
openclaw acp --session agent:main:main
openclaw acp --session agent:design:main
openclaw acp --session agent:qa:bug-123
```

Cada sesión de ACP corresponde a una única clave de sesión de Gateway. Un agente puede tener muchas
sesiones; ACP utiliza de forma predeterminada una sesión aislada `acp:&lt;uuid&gt;` a menos que cambies
la clave o la etiqueta.

<div id="zed-editor-setup">
  ## Configuración del editor Zed
</div>

Añade un agente ACP personalizado en `~/.config/zed/settings.json` (o utiliza la UI de Configuración de Zed):

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

Para dirigirse a un Gateway o agente específico:

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

En Zed, abre el panel Agente y selecciona «OpenClaw ACP» para iniciar una conversación.

<div id="session-mapping">
  ## Asignación de sesiones
</div>

De forma predeterminada, las sesiones de ACP obtienen una clave de sesión aislada del Gateway con un prefijo `acp:`.
Para reutilizar una sesión conocida, proporciona una clave o etiqueta de sesión:

* `--session <key>`: utiliza una clave de sesión específica del Gateway.
* `--session-label <label>`: resuelve una sesión existente a partir de la etiqueta.
* `--reset-session`: genera un ID de sesión nuevo para esa clave (misma clave, nueva transcripción).

Si tu cliente ACP admite metadatos, puedes sobrescribirlos por sesión:

```json
{
  "_meta": {
    "sessionKey": "agent:main:main",
    "sessionLabel": "bandeja de entrada de soporte",
    "resetSession": true
  }
}
```

Obtén más información sobre las claves de sesión en [/concepts/session](/es/concepts/session).

<div id="options">
  ## Opciones
</div>

* `--url <url>`: URL WebSocket del Gateway (por defecto usa gateway.remote.url si está configurada).
* `--token <token>`: token de autenticación del Gateway.
* `--password <password>`: contraseña de autenticación del Gateway.
* `--session <key>`: clave de sesión predeterminada.
* `--session-label <label>`: etiqueta de sesión predeterminada que se va a resolver.
* `--require-existing`: produce un error si la clave/etiqueta de sesión no existe.
* `--reset-session`: restablece la clave de sesión antes del primer uso.
* `--no-prefix-cwd`: no antepone el directorio de trabajo a los prompts.
* `--verbose, -v`: registro detallado en stderr.

<div id="acp-client-options">
  ### Opciones de `acp client`
</div>

* `--cwd <dir>`: directorio de trabajo de la sesión de ACP.
* `--server <command>`: comando del servidor ACP (valor predeterminado: `openclaw`).
* `--server-args <args...>`: argumentos adicionales pasados al servidor ACP.
* `--server-verbose`: habilita el registro detallado en el servidor ACP.
* `--verbose, -v`: habilita el registro detallado en el cliente.