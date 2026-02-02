---
title: Canales
summary: "Referencia de la CLI para `openclaw channels` (cuentas, estado, inicio/cierre de sesión, registros)"
read_when:
  - Quieres agregar o eliminar cuentas de canales (WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (complemento)/Signal/iMessage)
  - Quieres comprobar el estado de los canales o seguir en tiempo real los registros de los canales
---

<div id="openclaw-channels">
  # `openclaw channels`
</div>

Gestiona las cuentas de canales de chat y su estado en tiempo de ejecución en el Gateway.

Documentación relacionada:

* Guías de canales: [Canales](/es/channels/index)
* Configuración del Gateway: [Configuración](/es/gateway/configuration)

<div id="common-commands">
  ## Comandos habituales
</div>

```bash
openclaw channels list
openclaw channels status
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels logs --channel all
```

<div id="add-remove-accounts">
  ## Agregar / eliminar cuentas
</div>

```bash
openclaw channels add --channel telegram --token <bot-token>
openclaw channels remove --channel telegram --delete
```

Consejo: `openclaw channels add --help` muestra las opciones específicas de cada canal (token, token de la aplicación, rutas de signal-cli, etc.).

<div id="login-logout-interactive">
  ## Iniciar / cerrar sesión (interactivo)
</div>

```bash
openclaw channels login --channel whatsapp
openclaw channels logout --channel whatsapp
```

<div id="troubleshooting">
  ## Solución de problemas
</div>

* Ejecuta `openclaw status --deep` para un sondeo amplio.
* Usa `openclaw doctor` para correcciones guiadas.
* `openclaw channels list` imprime `Claude: HTTP 403 ... user:profile` → la instantánea de uso requiere el ámbito `user:profile`. Usa `--no-usage`, o proporciona una clave de sesión de claude.ai (`CLAUDE_WEB_SESSION_KEY` / `CLAUDE_WEB_COOKIE`), o vuelve a autenticarte mediante Claude Code CLI.

<div id="capabilities-probe">
  ## Sondeo de capacidades
</div>

Obtiene pistas sobre las capacidades del proveedor (intents/ámbitos cuando estén disponibles), además de compatibilidad con funciones estáticas:

```bash
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
```

Notas:

* `--channel` es opcional; omítelo para listar todos los canales (incluidas las extensiones).
* `--target` acepta `channel:<id>` o un ID numérico de canal sin procesar y solo aplica a Discord.
* Las sondas son específicas del proveedor: intents de Discord + permisos de canal opcionales; bot de Slack + ámbitos de usuario; flags de bot de Telegram + webhook; versión del daemon de Signal; token de app de MS Teams + roles/ámbitos de Graph (anotado cuando se conoce). Los canales sin sondas muestran `Probe: unavailable`.

<div id="resolve-names-to-ids">
  ## Resolver nombres en IDs
</div>

Resuelve nombres de canales y usuarios en IDs usando el directorio del proveedor:

```bash
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels resolve --channel discord "My Server/#support" "@someone"
openclaw channels resolve --channel matrix "Project Room"
```

Notas:

* Usa `--kind user|group|auto` para forzar el tipo de entidad.
* La resolución prioriza las coincidencias activas cuando varias entradas comparten el mismo nombre.
