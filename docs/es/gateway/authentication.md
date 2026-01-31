---
title: Autenticación
summary: "Autenticación de modelos: OAuth, claves API y setup-token"
read_when:
  - Depurar la autenticación de modelos o la caducidad de OAuth
  - Documentar la autenticación o el almacenamiento de credenciales
---

<div id="authentication">
  # Autenticación
</div>

OpenClaw admite OAuth y claves de API para proveedores de modelos. Para cuentas
de Anthropic, recomendamos usar una **clave de API**. Para el acceso por
suscripción a Claude, utiliza el token de larga duración creado por
`claude setup-token`.

Consulta [/concepts/oauth](/es/concepts/oauth) para ver el flujo completo de OAuth
y la estructura de almacenamiento.

<div id="recommended-anthropic-setup-api-key">
  ## Configuración recomendada para Anthropic (clave de API)
</div>

Si usas Anthropic directamente, utiliza una clave de API.

1. Crea una clave de API en la Consola de Anthropic.
2. Guárdala en el **host del Gateway** (la máquina que ejecuta `openclaw gateway`).

```bash
export ANTHROPIC_API_KEY="..."
openclaw models status
```

3. Si el Gateway se ejecuta con systemd/launchd, es preferible poner la clave en
   `~/.openclaw/.env` para que el demonio pueda leerla:

```bash
cat >> ~/.openclaw/.env <<'EOF'
ANTHROPIC_API_KEY=...
EOF
```

Luego, reinicia el daemon (o reinicia el proceso de tu Gateway) y vuelve a comprobarlo:

```bash
openclaw models status
openclaw doctor
```

Si prefieres no gestionar tú mismo las variables de entorno, el asistente de incorporación puede almacenar
claves de API para uso del daemon: `openclaw onboard`.

Consulta [Ayuda](/es/help) para obtener más detalles sobre la herencia de entorno (`env.shellEnv`,
`~/.openclaw/.env`, systemd/launchd).

<div id="anthropic-setup-token-subscription-auth">
  ## Anthropic: setup-token (autenticación por suscripción)
</div>

Para Anthropic, la vía recomendada es una **clave de API**. Si usas una
suscripción a Claude, también se admite el flujo de setup-token. Ejecútalo en el
**host del Gateway**:

```bash
claude setup-token
```

Luego pégalo en OpenClaw:

```bash
openclaw models auth setup-token --provider anthropic
```

Si el token se creó en otro equipo, pégalo manualmente:

```bash
openclaw models auth paste-token --provider anthropic
```

Si aparece un error de Anthropic como:

```
Esta credencial solo está autorizada para su uso con Claude Code y no se puede utilizar para otras solicitudes de API.
```

…utiliza en su lugar una clave de API de Anthropic.

Introducción manual de token (cualquier proveedor; escribe `auth-profiles.json` + actualiza la configuración):

```bash
openclaw models auth paste-token --provider anthropic
openclaw models auth paste-token --provider openrouter
```

Comprobación apta para automatización (devuelve `1` si está caducado o ausente, `2` si está a punto de caducar):

```bash
openclaw models status --check
```

Los scripts de operaciones opcionales (systemd/Termux) se documentan aquí:
[/automation/auth-monitoring](/es/automation/auth-monitoring)

> `claude setup-token` requiere un TTY interactivo.

<div id="checking-model-auth-status">
  ## Verificar el estado de autenticación del modelo
</div>

```bash
openclaw models status
openclaw doctor
```

<div id="controlling-which-credential-is-used">
  ## Controlar qué credencial se usa
</div>

<div id="per-session-chat-command">
  ### Por sesión (comando de chat)
</div>

Usa `/model <alias-or-id>@<profileId>` para fijar una credencial de proveedor específica para la sesión actual (ID de perfil de ejemplo: `anthropic:default`, `anthropic:work`).

Usa `/model` (o `/model list`) para un selector compacto; usa `/model status` para la vista completa (candidatos + siguiente perfil de autenticación, además de detalles del endpoint del proveedor cuando esté configurado).

<div id="per-agent-cli-override">
  ### Por agente (anulación en la CLI)
</div>

Establece una anulación explícita del orden de perfiles de autenticación para un agente (almacenada en el `auth-profiles.json` de ese agente):

```bash
openclaw models auth order get --provider anthropic
openclaw models auth order set --provider anthropic anthropic:default
openclaw models auth order clear --provider anthropic
```

Utiliza `--agent &lt;id&gt;` para seleccionar un agente específico; omítelo para usar el agente predeterminado configurado.

<div id="troubleshooting">
  ## Solución de problemas
</div>

<div id="no-credentials-found">
  ### «No se encontraron credenciales»
</div>

Si falta el perfil de token de Anthropic, ejecuta `claude setup-token` en el
**host del Gateway** y después vuelve a comprobarlo:

```bash
openclaw models status
```

<div id="token-expiringexpired">
  ### Token a punto de caducar/caducado
</div>

Ejecuta `openclaw models status` para confirmar qué perfil está a punto de caducar. Si el perfil no aparece, vuelve a ejecutar `claude setup-token` y pega de nuevo el token.

<div id="requirements">
  ## Requisitos
</div>

* Suscripción a Claude Max o Pro (para usar `claude setup-token`)
* CLI Claude Code instalada (comando `claude` disponible)