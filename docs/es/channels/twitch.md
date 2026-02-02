---
title: Twitch
summary: "Configuración y puesta en marcha del bot de chat de Twitch"
read_when:
  - Configuración de la integración de chat de Twitch para OpenClaw
---

<div id="twitch-plugin">
  # Twitch (complemento)
</div>

Compatibilidad con el chat de Twitch mediante una conexión IRC. OpenClaw se conecta como usuario de Twitch (cuenta bot) para recibir y enviar mensajes en canales.

<div id="plugin-required">
  ## Complemento requerido
</div>

Twitch se distribuye como un complemento y no viene incluido en la instalación principal.

Instala mediante la CLI (registro de npm):

```bash
openclaw plugins install @openclaw/twitch
```

Copia local (cuando se ejecuta desde un repositorio Git):

```bash
openclaw plugins install ./extensions/twitch
```

Más detalles: [complementos](/es/plugin)

<div id="quick-setup-beginner">
  ## Configuración rápida (para principiantes)
</div>

1. Crea una cuenta de Twitch dedicada para el bot (o usa una cuenta existente).
2. Genera las credenciales: [Twitch Token Generator](https://twitchtokengenerator.com/)
   * Selecciona **Bot Token**
   * Verifica que los *scopes* `chat:read` y `chat:write` estén seleccionados
   * Copia el **Client ID** y el **Access Token**
3. Obtén tu ID de usuario de Twitch: https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/
4. Configura el token:
   * Env: `OPENCLAW_TWITCH_ACCESS_TOKEN=...` (solo para la cuenta predeterminada)
   * O config: `channels.twitch.accessToken`
   * Si ambos están configurados, la opción de configuración tiene prioridad (la variable de entorno se usa como respaldo solo para la cuenta predeterminada).
5. Inicia el Gateway.

**⚠️ Importante:** Agrega control de acceso (`allowFrom` o `allowedRoles`) para evitar que usuarios no autorizados activen el bot. `requireMention` es `true` de forma predeterminada.

Configuración mínima:

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",              // Cuenta de Twitch del bot
      accessToken: "oauth:abc123...",    // Token de acceso OAuth (o usa la variable de entorno OPENCLAW_TWITCH_ACCESS_TOKEN)
      clientId: "xyz789...",             // ID de cliente del generador de tokens
      channel: "vevisk",                 // Canal de chat de Twitch al que unirse (requerido)
      allowFrom: ["123456789"]           // (recomendado) Solo tu ID de usuario de Twitch - obténlo desde https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/
    }
  }
}
```

<div id="what-it-is">
  ## Qué es
</div>

* Un canal de Twitch propiedad del Gateway.
* Enrutamiento determinista: las respuestas siempre se envían de vuelta a Twitch.
* Cada cuenta se asigna a una clave de sesión aislada `agent:<agentId>:twitch:<accountName>`.
* `username` es la cuenta del bot (quien se autentica), `channel` es la sala de chat a la que debe unirse.

<div id="setup-detailed">
  ## Configuración detallada
</div>

<div id="generate-credentials">
  ### Generar credenciales
</div>

Utiliza [Twitch Token Generator](https://twitchtokengenerator.com/):

* Selecciona **Bot Token**
* Asegúrate de que los permisos `chat:read` y `chat:write` estén seleccionados
* Copia el **Client ID** y el **Access Token**

No necesitas registrar la aplicación manualmente. Los tokens caducan después de varias horas.

<div id="configure-the-bot">
  ### Configurar el bot
</div>

**Variable de entorno (solo para la cuenta predeterminada):**

```bash
OPENCLAW_TWITCH_ACCESS_TOKEN=oauth:abc123...
```

**O bien, configuración:**

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk"
    }
  }
}
```

Si tanto las variables de entorno como la configuración están definidas, prevalece la configuración.

<div id="access-control-recommended">
  ### Control de acceso (recomendado)
</div>

```json5
{
  channels: {
    twitch: {
      allowFrom: ["123456789"],       // (recomendado) Solo tu ID de usuario de Twitch
      allowedRoles: ["moderator"]     // Or restrict to roles
    }
  }
}
```

**Roles disponibles:** `"moderator"`, `"owner"`, `"vip"`, `"subscriber"`, `"all"`.

**¿Por qué usar IDs de usuario?** Los nombres de usuario pueden cambiar, lo que permite suplantaciones. Los IDs de usuario son permanentes.

Encuentra tu ID de usuario de Twitch: https://www.streamweasels.com/tools/convert-twitch-username-%20to-user-id/ (Convierte tu nombre de usuario de Twitch en un ID)

<div id="token-refresh-optional">
  ## Renovación de tokens (opcional)
</div>

Los tokens de [Twitch Token Generator](https://twitchtokengenerator.com/) no se pueden renovar automáticamente; deberás regenerarlos cuando caduquen.

Para la renovación automática de tokens, crea tu propia aplicación de Twitch en [Twitch Developer Console](https://dev.twitch.tv/console) y añádela a tu configuración:

```json5
{
  channels: {
    twitch: {
      clientSecret: "your_client_secret",
      refreshToken: "your_refresh_token"
    }
  }
}
```

El bot renueva automáticamente los tokens antes de que expiren y registra los eventos de renovación.

<div id="multi-account-support">
  ## Compatibilidad con múltiples cuentas
</div>

Usa `channels.twitch.accounts` con tokens independientes por cuenta. Consulta [`gateway/configuration`](/es/gateway/configuration) para ver el patrón común.

Ejemplo (una cuenta de bot en dos canales):

```json5
{
  channels: {
    twitch: {
      accounts: {
        channel1: {
          username: "openclaw",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "vevisk"
        },
        channel2: {
          username: "openclaw",
          accessToken: "oauth:def456...",
          clientId: "uvw012...",
          channel: "secondchannel"
        }
      }
    }
  }
}
```

**Nota:** Cada cuenta requiere su propio token (un token por canal).

<div id="access-control">
  ## Control de acceso
</div>

<div id="role-based-restrictions">
  ### Restricciones por rol
</div>

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowedRoles: ["moderator", "vip"]
        }
      }
    }
  }
}
```

<div id="allowlist-by-user-id-most-secure">
  ### Lista de permitidos por ID de usuario (opción más segura)
</div>

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowFrom: ["123456789", "987654321"]
        }
      }
    }
  }
}
```

<div id="combined-allowlist-roles">
  ### Lista de permitidos y roles combinados
</div>

Los usuarios incluidos en `allowFrom` se saltan las comprobaciones de roles:

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowFrom: ["123456789"],
          allowedRoles: ["moderator"]
        }
      }
    }
  }
}
```

<div id="disable-mention-requirement">
  ### Deshabilitar el requisito de @mención
</div>

De forma predeterminada, `requireMention` es `true`. Para deshabilitarlo y hacer que responda a todos los mensajes:

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          requireMention: false
        }
      }
    }
  }
}
```

<div id="troubleshooting">
  ## Solución de problemas
</div>

Primero, ejecuta los comandos de diagnóstico:

```bash
openclaw doctor
openclaw channels status --probe
```

<div id="bot-doesnt-respond-to-messages">
  ### El bot no responde a los mensajes
</div>

**Verifica el control de acceso:** Configura temporalmente `allowedRoles: ["all"]` para probar.

**Verifica que el bot esté en el canal:** El bot debe unirse al canal especificado en `channel`.

<div id="token-issues">
  ### Problemas con tokens
</div>

**&quot;Failed to connect&quot; o errores de autenticación:**

* Verifica que `accessToken` sea el valor del token de acceso OAuth (normalmente empieza con el prefijo `oauth:`)
* Comprueba que el token tenga los ámbitos `chat:read` y `chat:write`
* Si usas actualización (refresh) de token, verifica que `clientSecret` y `refreshToken` estén configurados

<div id="token-refresh-not-working">
  ### La renovación del token no funciona
</div>

**Revisa los registros (logs) en busca de eventos de renovación:**

```
Using env token source for mybot
Access token refreshed for user 123456 (expires in 14400s)
```

Si ves &quot;token refresh disabled (no refresh token)&quot;:

* Asegúrate de haber configurado `clientSecret`
* Asegúrate de haber configurado `refreshToken`

<div id="config">
  ## Config
</div>

**Configuración de la cuenta:**

* `username` - Nombre de usuario del bot
* `accessToken` - Token de acceso OAuth con `chat:read` y `chat:write`
* `clientId` - Twitch Client ID (del generador de tokens o de tu app)
* `channel` - Canal al que unirse (obligatorio)
* `enabled` - Habilitar esta cuenta (valor por defecto: `true`)
* `clientSecret` - Opcional: para la actualización automática del token
* `refreshToken` - Opcional: para la actualización automática del token
* `expiresIn` - Expiración del token en segundos
* `obtainmentTimestamp` - Marca temporal de obtención del token
* `allowFrom` - Lista de permitidos de IDs de usuario
* `allowedRoles` - Control de acceso basado en roles (`"moderator" | "owner" | "vip" | "subscriber" | "all"`)
* `requireMention` - Requerir @mención (valor por defecto: `true`)

**Opciones del proveedor:**

* `channels.twitch.enabled` - Habilitar/deshabilitar el inicio del canal
* `channels.twitch.username` - Nombre de usuario del bot (configuración simplificada de una sola cuenta)
* `channels.twitch.accessToken` - Token de acceso OAuth (configuración simplificada de una sola cuenta)
* `channels.twitch.clientId` - Twitch Client ID (configuración simplificada de una sola cuenta)
* `channels.twitch.channel` - Canal al que unirse (configuración simplificada de una sola cuenta)
* `channels.twitch.accounts.<accountName>` - Configuración multicuenta (todos los campos de cuenta anteriores)

Ejemplo completo:

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
      clientSecret: "secret123...",
      refreshToken: "refresh456...",
      allowFrom: ["123456789"],
      allowedRoles: ["moderator", "vip"],
      accounts: {
        default: {
          username: "mybot",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "your_channel",
          enabled: true,
          clientSecret: "secret123...",
          refreshToken: "refresh456...",
          expiresIn: 14400,
          obtainmentTimestamp: 1706092800000,
          allowFrom: ["123456789", "987654321"],
          allowedRoles: ["moderator"]
        }
      }
    }
  }
}
```

<div id="tool-actions">
  ## Acciones de la herramienta
</div>

El agente puede llamar a `twitch` con la acción:

* `send` - Enviar un mensaje a un canal

Ejemplo:

```json5
{
  "action": "twitch",
  "params": {
    "message": "Hello Twitch!",
    "to": "#mychannel"
  }
}
```

<div id="safety-ops">
  ## Seguridad y operaciones
</div>

* **Trata los tokens como contraseñas** - Nunca subas tokens a git
* **Usa la actualización automática de tokens** para bots de ejecución prolongada
* **Usa listas de permitidos basadas en ID de usuario** en lugar de nombres de usuario para el control de acceso
* **Supervisa los registros** para eventos de actualización de tokens y estado de la conexión
* **Limita el ámbito de los tokens al mínimo** - Solicita solo `chat:read` y `chat:write`
* **Si te quedas bloqueado**: Reinicia el Gateway después de confirmar que ningún otro proceso tiene la sesión en uso

<div id="limits">
  ## Límites
</div>

* **500 caracteres** por mensaje (se divide automáticamente en bloques en los límites entre palabras)
* Se elimina el Markdown antes de dividir en bloques
* Sin limitación de tasa propia (usa los límites de tasa integrados de Twitch)