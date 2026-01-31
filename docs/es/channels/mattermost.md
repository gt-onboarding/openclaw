---
title: Mattermost
summary: "Configuración del bot de Mattermost y de OpenClaw"
read_when:
  - Configurar Mattermost
  - Depurar el enrutamiento de Mattermost
---

<div id="mattermost-plugin">
  # Mattermost (plugin)
</div>

Estado: compatible a través de complemento (token de bot + eventos WebSocket). Se admiten canales, grupos y mensajes directos (DMs).
Mattermost es una plataforma de mensajería para equipos autoalojable; consulta el sitio oficial en
[mattermost.com](https://mattermost.com) para obtener información detallada del producto y las descargas.

<div id="plugin-required">
  ## Complemento requerido
</div>

Mattermost se distribuye como un complemento y no viene incluido en la instalación principal.

Instálalo mediante la CLI (registro de npm):

```bash
openclaw plugins install @openclaw/mattermost
```

Repositorio local (cuando se ejecuta desde un repositorio Git):

```bash
openclaw plugins install ./extensions/mattermost
```

Si seleccionas Mattermost durante la configuración/onboarding y se detecta un `git checkout`,
OpenClaw ofrecerá automáticamente la ruta local de instalación.

Detalles: [Complementos](/es/plugin)

<div id="quick-setup">
  ## Configuración rápida
</div>

1. Instala el complemento de Mattermost.
2. Crea una cuenta de bot de Mattermost y copia el **token del bot**.
3. Copia la **URL base** de Mattermost (p. ej., `https://chat.example.com`).
4. Configura OpenClaw e inicia el Gateway.

Configuración mínima:

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing"
    }
  }
}
```

<div id="environment-variables-default-account">
  ## Variables de entorno (cuenta predeterminada)
</div>

Configura lo siguiente en el host donde se ejecuta el Gateway si prefieres variables de entorno:

* `MATTERMOST_BOT_TOKEN=...`
* `MATTERMOST_URL=https://chat.example.com`

Las variables de entorno solo se aplican a la cuenta **predeterminada** (`default`). Las demás cuentas deben usar valores de configuración.

<div id="chat-modes">
  ## Modos de chat
</div>

Mattermost responde a los mensajes directos (DMs) automáticamente. El comportamiento en canales se controla con `chatmode`:

* `oncall` (predeterminado): responde solo cuando lo mencionan con @ en canales.
* `onmessage`: responde a cada mensaje del canal.
* `onchar`: responde cuando un mensaje comienza con un prefijo de activación.

Ejemplo de configuración:

```json5
{
  channels: {
    mattermost: {
      chatmode: "onchar",
      oncharPrefixes: [">", "!"]
    }
  }
}
```

Notas:

* `onchar` sigue respondiendo a menciones explícitas con @.
* `channels.mattermost.requireMention` se sigue respetando para configuraciones heredadas, pero se prefiere `chatmode`.

<div id="access-control-dms">
  ## Control de acceso (DMs)
</div>

* Predeterminado: `channels.mattermost.dmPolicy = "pairing"` (los remitentes desconocidos reciben un código de emparejamiento).
* Aprobar a través de:
  * `openclaw pairing list mattermost`
  * `openclaw pairing approve mattermost <CODE>`
* DMs públicas (el valor open permite aceptar mensajes sin restricciones de cualquier usuario): `channels.mattermost.dmPolicy="open"` más `channels.mattermost.allowFrom=["*"]`.

<div id="channels-groups">
  ## Canales (grupos)
</div>

* Predeterminado: `channels.mattermost.groupPolicy = "allowlist"` (requiere mención).
* Autoriza remitentes en la lista de permitidos con `channels.mattermost.groupAllowFrom` (ID de usuario o `@username`).
* Canales open (`open` permite aceptar mensajes sin restricciones de cualquier usuario): `channels.mattermost.groupPolicy="open"` (requiere mención).

<div id="targets-for-outbound-delivery">
  ## Destinos para el envío saliente
</div>

Utiliza estos formatos de destino con `openclaw message send` o cron/webhooks:

* `channel:<id>` para un canal
* `user:<id>` para un mensaje directo (MD)
* `@username` para un mensaje directo (MD) (resuelto a través de la Mattermost API)

Los ID sin prefijo se tratan como canales.

<div id="multi-account">
  ## Varias cuentas
</div>

Mattermost es compatible con múltiples cuentas en `channels.mattermost.accounts`:

```json5
{
  channels: {
    mattermost: {
      accounts: {
        default: { name: "Primary", botToken: "mm-token", baseUrl: "https://chat.example.com" },
        alerts: { name: "Alerts", botToken: "mm-token-2", baseUrl: "https://alerts.example.com" }
      }
    }
  }
}
```

<div id="troubleshooting">
  ## Solución de problemas
</div>

* No hay respuestas en los canales: asegúrate de que el bot esté en el canal y menciónalo (oncall), usa un prefijo de disparo (onchar) o configura `chatmode: "onmessage"`.
* Errores de autenticación: verifica el token del bot, la URL base y si la cuenta está habilitada.
* Problemas con varias cuentas: las variables de entorno solo se aplican a la cuenta `default`.