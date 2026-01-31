---
title: CLI
summary: "Referencia de la CLI de OpenClaw para los comandos, subcomandos y opciones de `openclaw`"
read_when:
  - Al agregar o modificar comandos u opciones de la CLI
  - Al documentar nuevas superficies de comandos de la CLI
---

<div id="cli-reference">
  # Referencia de la CLI
</div>

Esta página describe el comportamiento actual de la CLI. Si cambian los comandos, actualiza este documento.

<div id="command-pages">
  ## Páginas de comandos
</div>

* [`setup`](/es/cli/setup)
* [`onboard`](/es/cli/onboard)
* [`configure`](/es/cli/configure)
* [`config`](/es/cli/config)
* [`doctor`](/es/cli/doctor)
* [`dashboard`](/es/cli/dashboard)
* [`reset`](/es/cli/reset)
* [`uninstall`](/es/cli/uninstall)
* [`update`](/es/cli/update)
* [`message`](/es/cli/message)
* [`agent`](/es/cli/agent)
* [`agents`](/es/cli/agents)
* [`acp`](/es/cli/acp)
* [`status`](/es/cli/status)
* [`health`](/es/cli/health)
* [`sessions`](/es/cli/sessions)
* [`gateway`](/es/cli/gateway)
* [`logs`](/es/cli/logs)
* [`system`](/es/cli/system)
* [`models`](/es/cli/models)
* [`memory`](/es/cli/memory)
* [`nodes`](/es/cli/nodes)
* [`devices`](/es/cli/devices)
* [`node`](/es/cli/node)
* [`approvals`](/es/cli/approvals)
* [`sandbox`](/es/cli/sandbox)
* [`tui`](/es/cli/tui)
* [`browser`](/es/cli/browser)
* [`cron`](/es/cli/cron)
* [`dns`](/es/cli/dns)
* [`docs`](/es/cli/docs)
* [`hooks`](/es/cli/hooks)
* [`webhooks`](/es/cli/webhooks)
* [`pairing`](/es/cli/pairing)
* [`plugins`](/es/cli/plugins) (comandos del complemento)
* [`channels`](/es/cli/channels)
* [`security`](/es/cli/security)
* [`skills`](/es/cli/skills)
* [`voicecall`](/es/cli/voicecall) (complemento; si está instalado)

<div id="global-flags">
  ## Opciones globales
</div>

* `--dev`: aísla el estado en `~/.openclaw-dev` y ajusta los puertos predeterminados.
* `--profile &lt;name&gt;`: aísla el estado en `~/.openclaw-&lt;name&gt;`.
* `--no-color`: desactiva los colores ANSI.
* `--update`: abreviatura de `openclaw update` (solo instalaciones desde código fuente).
* `-V`, `--version`, `-v`: imprime la versión y termina.

<div id="output-styling">
  ## Estilo de salida
</div>

* Los colores ANSI y los indicadores de progreso solo se muestran en sesiones TTY.
* Los hipervínculos OSC-8 se muestran como enlaces en los que se puede hacer clic en terminales compatibles; de lo contrario, se usan URL de texto plano.
* `--json` (y `--plain` donde sea compatible) desactiva el estilo para obtener una salida limpia.
* `--no-color` desactiva el estilo ANSI; también se respeta `NO_COLOR=1`.
* Los comandos de larga duración muestran un indicador de progreso (OSC 9;4 cuando se admita).

<div id="color-palette">
  ## Paleta de colores
</div>

OpenClaw usa una paleta inspirada en la langosta para la salida de la CLI.

* `accent` (#FF5A2D): encabezados, etiquetas, resaltados principales.
* `accentBright` (#FF7A3D): nombres de comandos, énfasis.
* `accentDim` (#D14A22): texto de resaltado secundario.
* `info` (#FF8A5B): valores informativos.
* `success` (#2FBF71): estados de éxito.
* `warn` (#FFB020): advertencias, comportamientos de reserva, atención.
* `error` (#E23D2D): errores, fallos.
* `muted` (#8B7F77): desénfasis, metadatos.

Fuente de referencia de la paleta: `src/terminal/palette.ts` (también llamado “lobster seam”).

<div id="command-tree">
  ## Árbol de comandos
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

Nota: los complementos pueden añadir comandos de nivel superior adicionales (por ejemplo, `openclaw voicecall`).

<div id="security">
  ## Seguridad
</div>

* `openclaw security audit` — audita la configuración y el estado local para detectar errores de seguridad habituales.
* `openclaw security audit --deep` — sondeo en vivo del Gateway con el mejor esfuerzo posible.
* `openclaw security audit --fix` — refuerza los valores seguros predeterminados y ajusta los permisos (chmod) del estado y la configuración.

<div id="plugins">
  ## Complementos
</div>

Administra las extensiones y su configuración:

* `openclaw plugins list` — descubre complementos (usa `--json` para salida legible por máquinas).
* `openclaw plugins info <id>` — muestra detalles de un complemento.
* `openclaw plugins install <path|.tgz|npm-spec>` — instala un complemento (o añade una ruta de complemento a `plugins.load.paths`).
* `openclaw plugins enable <id>` / `disable <id>` — habilita o deshabilita `plugins.entries.<id>.enabled`.
* `openclaw plugins doctor` — informa de errores de carga de complementos.

La mayoría de los cambios en los complementos requieren reiniciar el Gateway. Consulta [/plugin](/es/plugin).

<div id="memory">
  ## Memoria
</div>

Búsqueda vectorial en `MEMORY.md` + `memory/*.md`:

* `openclaw memory status` — muestra las estadísticas del índice.
* `openclaw memory index` — reindexa los archivos de memoria.
* `openclaw memory search "<query>"` — búsqueda semántica en la memoria.

<div id="chat-slash-commands">
  ## Comandos slash en el chat
</div>

Los mensajes de chat permiten comandos `/...` (de texto y nativos). Consulta [/tools/slash-commands](/es/tools/slash-commands).

Resumen:

* `/status` para diagnósticos rápidos.
* `/config` para cambios de configuración persistentes.
* `/debug` para anulaciones de configuración solo en tiempo de ejecución (en memoria, no en disco; requiere `commands.debug: true`).

<div id="setup-onboarding">
  ## Configuración + primeros pasos
</div>

<div id="setup">
  ### `setup`
</div>

Inicializa la configuración y el espacio de trabajo.

Opciones:

* `--workspace <dir>`: ruta del espacio de trabajo del agente (predeterminado `~/.openclaw/workspace`).
* `--wizard`: ejecuta el asistente de incorporación.
* `--non-interactive`: ejecuta el asistente sin interacción (sin solicitar entradas).
* `--mode <local|remote>`: modo del asistente.
* `--remote-url <url>`: URL del Gateway remoto.
* `--remote-token <token>`: token del Gateway remoto.

El asistente se ejecuta automáticamente cuando se especifica cualquiera de las banderas del asistente (`--non-interactive`, `--mode`, `--remote-url`, `--remote-token`).

<div id="onboard">
  ### `onboard`
</div>

Asistente interactivo para configurar el Gateway, el espacio de trabajo y las habilidades.

Opciones:

* `--workspace <dir>`
* `--reset` (restablece la configuración + credenciales + sesiones + espacio de trabajo antes del asistente)
* `--non-interactive`
* `--mode <local|remote>`
* `--flow <quickstart|advanced|manual>` (manual es un alias de advanced)
* `--auth-choice <setup-token|token|chutes|openai-codex|openai-api-key|openrouter-api-key|ai-gateway-api-key|moonshot-api-key|kimi-code-api-key|synthetic-api-key|venice-api-key|gemini-api-key|zai-api-key|apiKey|minimax-api|minimax-api-lightning|opencode-zen|skip>`
* `--token-provider <id>` (no interactivo; se usa con `--auth-choice token`)
* `--token <token>` (no interactivo; se usa con `--auth-choice token`)
* `--token-profile-id <id>` (no interactivo; valor predeterminado: `<provider>:manual`)
* `--token-expires-in <duration>` (no interactivo; p. ej., `365d`, `12h`)
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
* `--no-install-daemon` (alias: `--skip-daemon`)
* `--daemon-runtime <node|bun>`
* `--skip-channels`
* `--skip-skills`
* `--skip-health`
* `--skip-ui`
* `--node-manager <npm|pnpm|bun>` (se recomienda pnpm; bun no se recomienda para el runtime de Gateway)
* `--json`

<div id="configure">
  ### `configure`
</div>

Asistente de configuración interactivo (modelos, canales, habilidades, Gateway).

<div id="config">
  ### `config`
</div>

Utilidades de configuración no interactivas (`get`/`set`/`unset`). Ejecutar `openclaw config` sin
subcomando inicia el asistente interactivo.

Subcomandos:

* `config get <path>`: muestra un valor de configuración (ruta con puntos/corchetes).
* `config set <path> <value>`: establece un valor (JSON5 o cadena sin procesar).
* `config unset <path>`: elimina un valor.

<div id="doctor">
  ### `doctor`
</div>

Comprobaciones de estado + correcciones rápidas (configuración + Gateway + servicios heredados).

Opciones:

* `--no-workspace-suggestions`: desactivar las sugerencias de memoria del espacio de trabajo.
* `--yes`: aceptar los valores predeterminados sin preguntar (sin interfaz).
* `--non-interactive`: omitir preguntas; aplicar solo migraciones seguras.
* `--deep`: escanear los servicios del sistema en busca de instalaciones adicionales de Gateway.

<div id="channel-helpers">
  ## Utilidades de canal
</div>

<div id="channels">
  ### `channels`
</div>

Administra cuentas de canales de chat (WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (complemento)/Signal/iMessage/MS Teams).

Subcomandos:

* `channels list`: muestra los canales configurados y los perfiles de autenticación.
* `channels status`: comprueba la conectividad del Gateway y el estado de los canales (`--probe` ejecuta comprobaciones adicionales; usa `openclaw health` u `openclaw status --deep` para sondas de estado del Gateway).
* Consejo: `channels status` imprime advertencias con correcciones sugeridas cuando detecta configuraciones erróneas comunes (y luego te remite a `openclaw doctor`).
* `channels logs`: muestra registros recientes de canales desde el archivo de registro del Gateway.
* `channels add`: asistente interactivo cuando no se pasan flags (opciones); las flags cambian al modo no interactivo.
* `channels remove`: deshabilita de forma predeterminada; pasa `--delete` para eliminar entradas de configuración sin confirmaciones.
* `channels login`: inicio de sesión interactivo del canal (solo WhatsApp Web).
* `channels logout`: cierra la sesión de un canal (si es compatible).

Opciones comunes:

* `--channel <name>`: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams`
* `--account <id>`: ID de la cuenta del canal (valor predeterminado `default`)
* `--name <label>`: nombre para mostrar de la cuenta

Opciones de `channels login`:

* `--channel <channel>` (predeterminado `whatsapp`; admite `whatsapp`/`web`)
* `--account <id>`
* `--verbose`

Opciones de `channels logout`:

* `--channel <channel>` (predeterminado `whatsapp`)
* `--account <id>`

Opciones de `channels list`:

* `--no-usage`: omite instantáneas del uso y la cuota del proveedor de modelos (solo OAuth/API).
* `--json`: salida en JSON (incluye uso a menos que se establezca `--no-usage`).

Opciones de `channels logs`:

* `--channel <name|all>` (predeterminado `all`)
* `--lines <n>` (predeterminado `200`)
* `--json`

Más detalles: [/concepts/oauth](/es/concepts/oauth)

Ejemplos:

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

Lista e inspecciona las habilidades disponibles, junto con información de estado de disponibilidad.

Subcomandos:

* `skills list`: lista las habilidades (predeterminado cuando no hay subcomando).
* `skills info <name>`: muestra detalles de una habilidad.
* `skills check`: resumen de requisitos satisfechos vs faltantes.

Opciones:

* `--eligible`: muestra solo las habilidades listas.
* `--json`: genera salida en formato JSON (sin formato adicional).
* `-v`, `--verbose`: incluye detalles de los requisitos faltantes.

Consejo: usa `npx clawhub` para buscar, instalar y sincronizar habilidades.

<div id="pairing">
  ### `pairing`
</div>

Aprueba solicitudes de emparejamiento de mensajes directos (DM) en todos los canales.

Subcommands:

* `pairing list <channel> [--json]`
* `pairing approve <channel> <code> [--notify]`

<div id="webhooks-gmail">
  ### `webhooks gmail`
</div>

Configuración de webhook de Gmail Pub/Sub y runner. Consulta [/automation/gmail-pubsub](/es/automation/gmail-pubsub).

Subcomandos:

* `webhooks gmail setup` (requiere `--account <email>`; admite `--project`, `--topic`, `--subscription`, `--label`, `--hook-url`, `--hook-token`, `--push-token`, `--bind`, `--port`, `--path`, `--include-body`, `--max-bytes`, `--renew-minutes`, `--tailscale`, `--tailscale-path`, `--tailscale-target`, `--push-endpoint`, `--json`)
* `webhooks gmail run` (valores de sustitución en tiempo de ejecución para las mismas opciones)

<div id="dns-setup">
  ### `dns setup`
</div>

Asistente de DNS para descubrimiento de área amplia (CoreDNS + Tailscale). Consulta [/gateway/discovery](/es/gateway/discovery).

Opciones:

* `--apply`: instala/actualiza la configuración de CoreDNS (requiere sudo; solo macOS).

<div id="messaging-agent">
  ## Mensajería y agente
</div>

<div id="message">
  ### `message`
</div>

Mensajería de salida unificada + acciones de canal.

Ver: [/cli/message](/es/cli/message)

Subcomandos:

* `message send|poll|react|reactions|read|edit|delete|pin|unpin|pins|permissions|search|timeout|kick|ban`
* `message thread <create|list|reply>`
* `message emoji <list|upload>`
* `message sticker <send|upload>`
* `message role <info|add|remove>`
* `message channel <info|list>`
* `message member info`
* `message voice status`
* `message event <list|create>`

Ejemplos:

* `openclaw message send --target +15555550123 --message "Hola"`
* `openclaw message poll --channel discord --target channel:123 --poll-question "¿Snack?" --poll-option Pizza --poll-option Sushi`

<div id="agent">
  ### `agent`
</div>

Ejecuta un turno de un agente a través del Gateway (o `--local` integrado).

Obligatorio:

* `--message <text>`

Opciones:

* `--to <dest>` (para la clave de sesión y la entrega opcional)
* `--session-id <id>`
* `--thinking <off|minimal|low|medium|high|xhigh>` (solo modelos GPT-5.2 + Codex)
* `--verbose <on|full|off>`
* `--channel <whatsapp|telegram|discord|slack|mattermost|signal|imessage|msteams>`
* `--local`
* `--deliver`
* `--json`
* `--timeout <seconds>`

<div id="agents">
  ### `agents`
</div>

Gestiona agentes aislados (espacios de trabajo + autenticación + enrutamiento).

<div id="agents-list">
  #### `agents list`
</div>

Enumera los agentes configurados.

Opciones:

* `--json`
* `--bindings`

<div id="agents-add-name">
  #### `agents add [name]`
</div>

Añade un nuevo agente aislado. Ejecuta el asistente guiado a menos que se pasen flags (o `--non-interactive`); `--workspace` es obligatorio en modo no interactivo.

Opciones:

* `--workspace &lt;dir&gt;`
* `--model &lt;id&gt;`
* `--agent-dir &lt;dir&gt;`
* `--bind &lt;channel[:accountId]&gt;` (repetible)
* `--non-interactive`
* `--json`

Las especificaciones de vinculación usan `channel[:accountId]`. Cuando se omite `accountId` para WhatsApp, se usa el ID de cuenta predeterminado.

<div id="agents-delete-id">
  #### `agents delete <id>`
</div>

Elimina un agente y suprime su espacio de trabajo y su estado.

Opciones:

* `--force`
* `--json`

<div id="acp">
  ### `acp`
</div>

Ejecuta el puente ACP que conecta los IDE al Gateway.

Consulta [`acp`](/es/cli/acp) para ver la lista completa de opciones y ejemplos.

<div id="status">
  ### `status`
</div>

Mostrar el estado de la sesión asociada y los destinatarios recientes.

Opciones:

* `--json`
* `--all` (diagnóstico completo; solo lectura, listo para pegar)
* `--deep` (inspeccionar canales)
* `--usage` (mostrar uso/cuota del proveedor de modelos)
* `--timeout <ms>`
* `--verbose`
* `--debug` (alias de `--verbose`)

Notas:

* La vista general incluye el estado del servicio del Gateway y del servicio host del nodo cuando están disponibles.

<div id="usage-tracking">
  ### Seguimiento de uso
</div>

OpenClaw puede mostrar el uso y la cuota del proveedor cuando hay credenciales OAuth/API disponibles.

Puntos de visualización:

* `/status` (añade una breve línea de uso del proveedor cuando está disponible)
* `openclaw status --usage` (imprime el desglose completo por proveedor)
* Barra de menús de macOS (sección Usage en Context)

Notas:

* Los datos provienen directamente de los endpoints de uso del proveedor (sin estimaciones).
* Proveedores: Anthropic, GitHub Copilot, OpenAI Codex OAuth, además de Gemini CLI/Antigravity cuando esos complementos de proveedor están habilitados.
* Si no existen credenciales coincidentes, el uso no se muestra.
* Detalles: consulta [Seguimiento de uso](/es/concepts/usage-tracking).

<div id="health">
  ### `health`
</div>

Obtiene el estado de salud del Gateway en ejecución.

Opciones:

* `--json`
* `--timeout <ms>`
* `--verbose`

<div id="sessions">
  ### `sessions`
</div>

Muestra las sesiones de conversación guardadas.

Opciones:

* `--json`
* `--verbose`
* `--store <path>`
* `--active <minutes>`

<div id="reset-uninstall">
  ## Restablecer / Desinstalar
</div>

<div id="reset">
  ### `reset`
</div>

Restablece la configuración y el estado locales (sin desinstalar la CLI).

Opciones:

* `--scope <config|config+creds+sessions|full>`
* `--yes`
* `--non-interactive`
* `--dry-run`

Notas:

* `--non-interactive` requiere `--scope` y `--yes`.

<div id="uninstall">
  ### `uninstall`
</div>

Desinstala el servicio del Gateway y los datos locales (la CLI se mantiene).

Opciones:

* `--service`
* `--state`
* `--workspace`
* `--app`
* `--all`
* `--yes`
* `--non-interactive`
* `--dry-run`

Notas:

* `--non-interactive` requiere `--yes` y ámbitos explícitos (o `--all`).

## Gateway

<div id="gateway">
  ### `gateway`
</div>

Ejecuta el Gateway WebSocket.

Opciones:

* `--port <port>`
* `--bind <loopback|tailnet|lan|auto|custom>`
* `--token <token>`
* `--auth <token|password>`
* `--password <password>`
* `--tailscale <off|serve|funnel>`
* `--tailscale-reset-on-exit`
* `--allow-unconfigured`
* `--dev`
* `--reset` (restablece la configuración de desarrollo, credenciales, sesiones y espacio de trabajo)
* `--force` (finaliza el proceso en escucha existente en el puerto)
* `--verbose`
* `--claude-cli-logs`
* `--ws-log <auto|full|compact>`
* `--compact` (alias de `--ws-log compact`)
* `--raw-stream`
* `--raw-stream-path <ruta>`

<div id="gateway-service">
  ### `gateway service`
</div>

Administra el servicio Gateway (launchd/systemd/schtasks).

Subcomandos:

* `gateway status` (consulta el RPC del Gateway de forma predeterminada)
* `gateway install` (instalación del servicio)
* `gateway uninstall`
* `gateway start`
* `gateway stop`
* `gateway restart`

Notas:

* `gateway status` consulta el RPC del Gateway de forma predeterminada usando el puerto/configuración resueltos del servicio (se puede sobrescribir con `--url/--token/--password`).
* `gateway status` admite `--no-probe`, `--deep` y `--json` para uso en scripts.
* `gateway status` también muestra servicios Gateway heredados o adicionales cuando puede detectarlos (`--deep` añade escaneos a nivel de sistema). Los servicios OpenClaw con nombre de perfil se tratan como de primera clase y no se marcan como &quot;extra&quot;.
* `gateway status` imprime qué ruta de configuración usa la CLI frente a qué configuración probablemente use el servicio (entorno del servicio), además de la URL de destino de sondeo resultante.
* `gateway install|uninstall|start|stop|restart` admiten `--json` para uso en scripts (la salida predeterminada sigue siendo legible para humanos).
* `gateway install` usa por defecto el runtime de Node; bun **no se recomienda** (errores con WhatsApp/Telegram).
* Opciones de `gateway install`: `--port`, `--runtime`, `--token`, `--force`, `--json`.

<div id="logs">
  ### `logs`
</div>

Transmite en tiempo real los logs de archivos del Gateway mediante RPC.

Notas:

* Las sesiones TTY muestran una vista estructurada y con color; en entornos no TTY se usa texto sin formato.
* `--json` emite JSON delimitado por líneas (un evento de log por línea).

Ejemplos:

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

Utilidades de la CLI del Gateway (usa `--url`, `--token`, `--password`, `--timeout`, `--expect-final` para subcomandos RPC).

Subcomandos:

* `gateway call <method> [--params <json>]`
* `gateway health`
* `gateway status`
* `gateway probe`
* `gateway discover`
* `gateway install|uninstall|start|stop|restart`
* `gateway run`

Llamadas RPC comunes:

* `config.apply` (validar + escribir configuración + reiniciar + activar)
* `config.patch` (fusionar una actualización parcial + reiniciar + activar)
* `update.run` (ejecutar actualización + reiniciar + activar)

Consejo: cuando invoques directamente `config.set`/`config.apply`/`config.patch`, pasa `baseHash` obtenido de
`config.get` si ya existe una configuración.

<div id="models">
  ## Modelos
</div>

Consulta [/concepts/models](/es/concepts/models) para ver el comportamiento de conmutación por error y la estrategia de exploración.

Autenticación preferida para Anthropic (setup-token):

```bash
claude setup-token
openclaw models auth setup-token --provider anthropic
openclaw models status
```

<div id="models-root">
  ### `models` (raíz)
</div>

`openclaw models` es un alias para `models status`.

Opciones a nivel raíz:

* `--status-json` (alias para `models status --json`)
* `--status-plain` (alias para `models status --plain`)

<div id="models-list">
  ### `models list`
</div>

Opciones:

* `--all`
* `--local`
* `--provider <name>`
* `--json`
* `--plain`

<div id="models-status">
  ### `models status`
</div>

Opciones:

* `--json`
* `--plain`
* `--check` (exit 1=caducado/ausente, 2=próximo a caducar)
* `--probe` (sondeo en tiempo real de los perfiles de autenticación configurados)
* `--probe-provider <name>`
* `--probe-profile <id>` (repetir o separar por comas)
* `--probe-timeout <ms>`
* `--probe-concurrency <n>`
* `--probe-max-tokens <n>`

Siempre incluye el resumen de autenticación y el estado de caducidad de OAuth para los perfiles en el almacén de autenticación.
`--probe` ejecuta solicitudes en tiempo real (puede consumir tokens y activar límites de frecuencia).

<div id="models-set-model">
  ### `models set <model>`
</div>

Configura `agents.defaults.model.primary`.

<div id="models-set-image-model">
  ### `models set-image <model>`
</div>

Configura `agents.defaults.imageModel.primary`.

<div id="models-aliases-listaddremove">
  ### `models aliases list|add|remove`
</div>

Opciones:

* `list`: `--json`, `--plain`
* `add <alias> <model>`
* `remove <alias>`

<div id="models-fallbacks-listaddremoveclear">
  ### `models fallbacks list|add|remove|clear`
</div>

Opciones:

* `list`: `--json`, `--plain`
* `add <modelo>`
* `remove <modelo>`
* `clear`

<div id="models-image-fallbacks-listaddremoveclear">
  ### `models image-fallbacks list|add|remove|clear`
</div>

Opciones:

* `list`: `--json`, `--plain`
* `add <model>`
* `remove <model>`
* `clear`

<div id="models-scan">
  ### `models scan`
</div>

Opciones:

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

Opciones:

* `add`: asistente interactivo de autenticación
* `setup-token`: `--provider <nombre>` (predeterminado: `anthropic`), `--yes`
* `paste-token`: `--provider <nombre>`, `--profile-id <id>`, `--expires-in <duración>`

<div id="models-auth-order-getsetclear">
  ### `models auth order get|set|clear`
</div>

Opciones:

* `get`: `--provider <name>`, `--agent <id>`, `--json`
* `set`: `--provider <name>`, `--agent <id>`, `<profileIds...>`
* `clear`: `--provider <name>`, `--agent <id>`

<div id="system">
  ## Sistema
</div>

<div id="system-event">
  ### `system event`
</div>

Pone en cola un evento del sistema y, opcionalmente, activa un latido (Gateway RPC).

Requerido:

* `--text <text>`

Opciones:

* `--mode <now|next-heartbeat>`
* `--json`
* `--url`, `--token`, `--timeout`, `--expect-final`

<div id="system-heartbeat-lastenabledisable">
  ### `system heartbeat last|enable|disable`
</div>

Controles de latido (RPC del Gateway).

Opciones:

* `--json`
* `--url`, `--token`, `--timeout`, `--expect-final`

<div id="system-presence">
  ### `system presence`
</div>

Muestra las entradas de presencia del sistema (RPC del Gateway).

Opciones:

* `--json`
* `--url`, `--token`, `--timeout`, `--expect-final`

<div id="cron">
  ## Cron
</div>

Administra trabajos programados (Gateway RPC). Consulta [/automation/cron-jobs](/es/automation/cron-jobs).

Subcomandos:

* `cron status [--json]`
* `cron list [--all] [--json]` (salida tabular por defecto; usa `--json` para salida en bruto)
* `cron add` (alias: `create`; requiere `--name` y exactamente uno de `--at` | `--every` | `--cron`, y exactamente un payload de `--system-event` | `--message`)
* `cron edit <id>` (modifica campos)
* `cron rm <id>` (alias: `remove`, `delete`)
* `cron enable <id>`
* `cron disable <id>`
* `cron runs --id <id> [--limit <n>]`
* `cron run <id> [--force]`

Todos los comandos `cron` aceptan `--url`, `--token`, `--timeout`, `--expect-final`.

<div id="node-host">
  ## Host de nodo
</div>

`node` ejecuta un **host de nodo sin interfaz (headless)** o lo gestiona como un servicio en segundo plano. Consulta
[`openclaw node`](/es/cli/node).

Subcomandos:

* `node run --host <gateway-host> --port 18789`
* `node status`
* `node install [--host <gateway-host>] [--port <port>] [--tls] [--tls-fingerprint <sha256>] [--node-id <id>] [--display-name <name>] [--runtime <node|bun>] [--force]`
* `node uninstall`
* `node stop`
* `node restart`

<div id="nodes">
  ## Nodos
</div>

`nodes` se comunica con el Gateway y opera sobre nodos emparejados. Consulta [/nodes](/es/nodes).

Opciones comunes:

* `--url`, `--token`, `--timeout`, `--json`

Subcomandos:

* `nodes status [--connected] [--last-connected <duration>]`
* `nodes describe --node <id|name|ip>`
* `nodes list [--connected] [--last-connected <duration>]`
* `nodes pending`
* `nodes approve <requestId>`
* `nodes reject <requestId>`
* `nodes rename --node <id|name|ip> --name <displayName>`
* `nodes invoke --node <id|name|ip> --command <command> [--params <json>] [--invoke-timeout <ms>] [--idempotency-key <key>]`
* `nodes run --node <id|name|ip> [--cwd <path>] [--env KEY=VAL] [--command-timeout <ms>] [--needs-screen-recording] [--invoke-timeout <ms>] <command...>` (nodo macOS o host de nodo sin interfaz)
* `nodes notify --node <id|name|ip> [--title <text>] [--body <text>] [--sound <name>] [--priority <passive|active|timeSensitive>] [--delivery <system|overlay|auto>] [--invoke-timeout <ms>]` (solo macOS)

Cámara:

* `nodes camera list --node <id|name|ip>`
* `nodes camera snap --node <id|name|ip> [--facing front|back|both] [--device-id <id>] [--max-width <px>] [--quality <0-1>] [--delay-ms <ms>] [--invoke-timeout <ms>]`
* `nodes camera clip --node <id|name|ip> [--facing front|back] [--device-id <id>] [--duration <ms|10s|1m>] [--no-audio] [--invoke-timeout <ms>]`

Canvas + pantalla:

* `nodes canvas snapshot --node <id|name|ip> [--format png|jpg|jpeg] [--max-width <px>] [--quality <0-1>] [--invoke-timeout <ms>]`
* `nodes canvas present --node <id|name|ip> [--target <urlOrPath>] [--x <px>] [--y <px>] [--width <px>] [--height <px>] [--invoke-timeout <ms>]`
* `nodes canvas hide --node <id|name|ip> [--invoke-timeout <ms>]`
* `nodes canvas navigate <url> --node <id|name|ip> [--invoke-timeout <ms>]`
* `nodes canvas eval [<js>] --node <id|name|ip> [--js <code>] [--invoke-timeout <ms>]`
* `nodes canvas a2ui push --node <id|name|ip> (--jsonl <path> | --text <text>) [--invoke-timeout <ms>]`
* `nodes canvas a2ui reset --node <id|name|ip> [--invoke-timeout <ms>]`
* `nodes screen record --node <id|name|ip> [--screen <index>] [--duration <ms|10s>] [--fps <n>] [--no-audio] [--out <path>] [--invoke-timeout <ms>]`

Ubicación:

* `nodes location get --node <id|name|ip> [--max-age <ms>] [--accuracy <coarse|balanced|precise>] [--location-timeout <ms>] [--invoke-timeout <ms>]`

<div id="browser">
  ## Navegador
</div>

CLI de control del navegador (instancias dedicadas de Chrome/Brave/Edge/Chromium). Consulta [`openclaw browser`](/es/cli/browser) y la [herramienta Browser](/es/tools/browser).

Opciones comunes:

* `--url`, `--token`, `--timeout`, `--json`
* `--browser-profile <name>`

Gestionar:

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

Inspeccionar:

* `browser screenshot [targetId] [--full-page] [--ref <ref>] [--element <selector>] [--type png|jpeg]`
* `browser snapshot [--format aria|ai] [--target-id <id>] [--limit <n>] [--interactive] [--compact] [--depth <n>] [--selector <sel>] [--out <path>]`

Acciones:

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
  ## Búsqueda en los docs
</div>

<div id="docs-query">
  ### `docs [query...]`
</div>

Busca en el índice de la documentación en línea.

## TUI

<div id="tui">
  ### `tui`
</div>

Abre la UI de terminal conectada al Gateway.

Opciones:

* `--url <url>`
* `--token <token>`
* `--password <password>`
* `--session <key>`
* `--deliver`
* `--thinking <level>`
* `--message <text>`
* `--timeout-ms <ms>` (valor predeterminado: `agents.defaults.timeoutSeconds`)
* `--history-limit <n>`