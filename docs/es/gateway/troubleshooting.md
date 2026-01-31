---
title: Solución de problemas
summary: "Guía rápida de resolución de problemas para fallos comunes de OpenClaw"
read_when:
  - Al investigar problemas o fallos en tiempo de ejecución
---

<div id="troubleshooting">
  # Solución de problemas 🔧
</div>

Cuando OpenClaw se comporta de forma inesperada, aquí tienes cómo solucionarlo.

Empieza por la sección de [primeros 60 segundos](/es/help/faq#first-60-seconds-if-somethings-broken) de las preguntas frecuentes si solo quieres una receta rápida para un diagnóstico inicial. Esta página profundiza más en fallos de ejecución y diagnósticos.

Atajos específicos por proveedor: [/channels/troubleshooting](/es/channels/troubleshooting)

<div id="status-diagnostics">
  ## Estado y diagnósticos
</div>

Comandos rápidos para diagnóstico inicial (en este orden):

| Command | What it tells you | When to use it |
|---|---|---|
| `openclaw status` | Resumen local: SO + actualización, accesibilidad/modo del Gateway, servicio, agentes/sesiones, estado de configuración de proveedores | Primera comprobación, visión general rápida |
| `openclaw status --all` | Diagnóstico local completo (solo lectura, copiable, relativamente seguro), incl. tramo final del log | Cuando necesitas compartir un informe de depuración |
| `openclaw status --deep` | Ejecuta comprobaciones de salud del Gateway (incl. sondeos a proveedores; requiere un Gateway accesible) | Cuando “configurado” no significa “funcionando” |
| `openclaw gateway probe` | Descubrimiento del Gateway + comprobación de accesibilidad (destinos locales y remotos) | Cuando sospechas que estás sondeando el Gateway equivocado |
| `openclaw channels status --probe` | Le pide al Gateway en ejecución el estado de los canales (y opcionalmente los sondea) | Cuando el Gateway es accesible pero los canales se comportan mal |
| `openclaw gateway status` | Estado del supervisor (launchd/systemd/schtasks), PID/salida del runtime, último error del Gateway | Cuando el servicio “parece levantado” pero nada se ejecuta |
| `openclaw logs --follow` | Registros en vivo (la mejor señal para problemas en tiempo de ejecución) | Cuando necesitas el motivo real del fallo |

**Para compartir la salida:** prioriza `openclaw status --all` (redacta los tokens). Si pegas `openclaw status`, considera establecer primero `OPENCLAW_SHOW_SECRETS=0` (previsualizaciones de tokens).

Consulta también: [Comprobaciones de salud](/es/gateway/health) y [Logging](/es/logging).

<div id="common-issues">
  ## Problemas frecuentes
</div>

<div id="no-api-key-found-for-provider-anthropic">
  ### No se encontró ninguna clave de API para el proveedor &quot;anthropic&quot;
</div>

Esto significa que **el almacén de autenticación del agente está vacío** o que faltan las credenciales de Anthropic.
La autenticación es **por agente**, por lo que un agente nuevo no heredará las claves del agente principal.

Opciones para solucionarlo:

* Vuelve a ejecutar el proceso de onboarding y elige **Anthropic** para ese agente.
* O pega un setup-token en el **host del Gateway**:
  ```bash
  openclaw models auth setup-token --provider anthropic
  ```
* O copia `auth-profiles.json` desde el directorio del agente principal al directorio del nuevo agente.

Verifica:

```bash
openclaw models status
```

<div id="oauth-token-refresh-failed-anthropic-claude-subscription">
  ### Error al actualizar el token OAuth (suscripción a Anthropic Claude)
</div>

Esto significa que el token OAuth de Anthropic almacenado ha caducado y su renovación ha fallado.
Si estás en una suscripción de Claude (sin clave de API), la solución más fiable es
cambiar a un **Claude Code setup-token** y pegarlo en el **host del Gateway**.

**Recomendado (setup-token):**

```bash
# Ejecutar en el host del Gateway (pegar el setup-token)
openclaw models auth setup-token --provider anthropic
openclaw models status
```

Si generaste el token en otro sitio:

```bash
openclaw models auth paste-token --provider anthropic
openclaw models status
```

Más información: [Anthropic](/es/providers/anthropic) y [OAuth](/es/concepts/oauth).

<div id="control-ui-fails-on-http-device-identity-required-connect-failed">
  ### Control UI falla con HTTP (&quot;device identity required&quot; / &quot;connect failed&quot;)
</div>

Si abres el panel a través de HTTP sin cifrar (por ejemplo, `http://<lan-ip>:18789/` o
`http://<tailscale-ip>:18789/`), el navegador se ejecuta en un **contexto no seguro** y
bloquea WebCrypto, por lo que no se puede generar la identidad del dispositivo.

**Solución:**

* Da prioridad a HTTPS mediante [Tailscale Serve](/es/gateway/tailscale).
* O ábrelo localmente en el host del Gateway: `http://127.0.0.1:18789/`.
* Si debes seguir usando HTTP, habilita `gateway.controlUi.allowInsecureAuth: true` y
  usa un token del Gateway (solo token; sin identidad de dispositivo (device identity) ni emparejamiento). Consulta
  [Control UI](/es/web/control-ui#insecure-http).

<div id="ci-secrets-scan-failed">
  ### Falló el escaneo de secretos en CI
</div>

Esto significa que `detect-secrets` encontró nuevos candidatos que aún no están en la línea base.
Consulta [Secret scanning](/es/gateway/security#secret-scanning-detect-secrets).

<div id="service-installed-but-nothing-is-running">
  ### Servicio instalado pero nada en ejecución
</div>

Si el servicio del Gateway está instalado pero el proceso termina inmediatamente, el servicio
puede aparecer como “cargado” aunque en realidad no haya ningún proceso en ejecución.

**Comprueba:**

```bash
openclaw gateway status
openclaw doctor
```

Doctor/servicio mostrará el estado de ejecución (PID/última salida) y pistas en los logs.

**Logs:**

* Opción recomendada: `openclaw logs --follow`
* Logs en archivo (siempre): `/tmp/openclaw/openclaw-YYYY-MM-DD.log` (o tu `logging.file` configurado)
* macOS LaunchAgent (si está instalado): `$OPENCLAW_STATE_DIR/logs/gateway.log` y `gateway.err.log`
* Linux systemd (si está instalado): `journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`
* Windows: `schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST`

**Habilitar más logging:**

* Aumentar el nivel de detalle de los logs en archivo (JSONL persistente):
  ```json
  { "logging": { "level": "debug" } }
  ```
* Aumentar la verbosidad de la consola (solo salida TTY):
  ```json
  { "logging": { "consoleLevel": "debug", "consoleStyle": "pretty" } }
  ```
* Consejo rápido: `--verbose` afecta solo la salida de **consola**. Los logs en archivo siguen controlados por `logging.level`.

Consulta [/logging](/es/logging) para una descripción completa de formatos, configuración y acceso.

<div id="gateway-start-blocked-set-gatewaymodelocal">
  ### &quot;Inicio del Gateway bloqueado: establece gateway.mode=local&quot;
</div>

Esto significa que la configuración existe pero `gateway.mode` no está establecido (o no es `local`), por lo que el
Gateway se niega a iniciarse.

**Corrección (recomendada):**

* Ejecuta el asistente y establece el modo de ejecución del Gateway en **Local**:
  ```bash
  openclaw configure
  ```
* O establécelo directamente:
  ```bash
  openclaw config set gateway.mode local
  ```

**Si en cambio querías ejecutar un Gateway remoto:**

* Configura una URL remota y mantén `gateway.mode=remote`:
  ```bash
  openclaw config set gateway.mode remote
  openclaw config set gateway.remote.url "wss://gateway.example.com"
  ```

**Solo para uso ad‑hoc/desarrollo:** pasa `--allow-unconfigured` para iniciar el Gateway sin
`gateway.mode=local`.

**¿Aún no tienes archivo de configuración?** Ejecuta `openclaw setup` para crear una configuración inicial y luego vuelve a ejecutar
el Gateway.

<div id="service-environment-path-runtime">
  ### Entorno del servicio (PATH + runtime)
</div>

El servicio del Gateway se ejecuta con un **PATH mínimo** para evitar basura del shell/gestor:

* macOS: `/opt/homebrew/bin`, `/usr/local/bin`, `/usr/bin`, `/bin`
* Linux: `/usr/local/bin`, `/usr/bin`, `/bin`

Esto excluye intencionalmente los gestores de versiones (nvm/fnm/volta/asdf) y los
gestores de paquetes (pnpm/npm) porque el servicio no carga la inicialización de tu shell. Las variables de
runtime como `DISPLAY` deben residir en `~/.openclaw/.env` (cargado de forma temprana por el
Gateway).
Las ejecuciones de Exec en `host=gateway` fusionan el `PATH` de tu shell de inicio de sesión en el entorno de ejecución,
por lo que las herramientas que falten normalmente significan que tu shell de inicio de sesión no las está exportando (o configura
`tools.exec.pathPrepend`). Consulta [/tools/exec](/es/tools/exec).

Los canales de WhatsApp y Telegram requieren **Node**; Bun no es compatible. Si tu
servicio se instaló con Bun o con una ruta de Node gestionada por un gestor de versiones, ejecuta `openclaw doctor`
para migrar a una instalación de Node del sistema.

<div id="skill-missing-api-key-in-sandbox">
  ### Skill sin clave de API en el sandbox
</div>

**Síntoma:** La skill funciona en el host pero falla en el sandbox por falta de la clave de API.

**Motivo:** la ejecución en sandbox se realiza dentro de Docker y **no** hereda el `process.env` del host.

**Solución:**

* configura `agents.defaults.sandbox.docker.env` (o, por agente, `agents.list[].sandbox.docker.env`)
* o incluye la clave en tu imagen de sandbox personalizada
* luego ejecuta `openclaw sandbox recreate --agent <id>` (o `--all`)

<div id="service-running-but-port-not-listening">
  ### Servicio en ejecución pero el puerto no está escuchando
</div>

Si el servicio informa que está **running** pero no hay nada escuchando en el puerto del Gateway,
es probable que el Gateway haya rechazado asociarse al puerto (bind).

**Qué significa &quot;running&quot; aquí**

* `Runtime: running` significa que tu supervisor (launchd/systemd/schtasks) cree que el proceso está vivo.
* `RPC probe` significa que el CLI realmente pudo conectarse al WebSocket del Gateway y llamar a `status`.
* Confía siempre en `Probe target:` + `Config (service):` como las líneas de “¿qué intentamos realmente?”.

**Comprueba:**

* `gateway.mode` debe ser `local` tanto para `openclaw gateway` como para el servicio.
* Si configuras `gateway.mode=remote`, el **CLI, por defecto**, usa una URL remota. El servicio puede seguir ejecutándose localmente, pero tu CLI podría estar sondeando el lugar equivocado. Usa `openclaw gateway status` para ver el puerto resuelto del servicio + el destino del sondeo (o pasa `--url`).
* `openclaw gateway status` y `openclaw doctor` muestran el **último error del Gateway** desde los logs cuando el servicio parece estar en ejecución pero el puerto está cerrado.
* Los binds que no son de loopback (`lan`/`tailnet`/`custom`, o `auto` cuando loopback no está disponible) requieren autenticación:
  `gateway.auth.token` (o `OPENCLAW_GATEWAY_TOKEN`).
* `gateway.remote.token` es solo para llamadas de CLI remotas; **no** habilita la autenticación local.
* `gateway.token` se ignora; usa `gateway.auth.token`.

**Si `openclaw gateway status` muestra una discrepancia de configuración**

* `Config (cli): ...` y `Config (service): ...` normalmente deberían coincidir.
* Si no coinciden, casi con total seguridad estás editando una configuración mientras el servicio está usando otra.
* Solución: vuelve a ejecutar `openclaw gateway install --force` desde el mismo `--profile` / `OPENCLAW_STATE_DIR` que quieres que use el servicio.

**Si `openclaw gateway status` informa problemas de configuración del servicio**

* La configuración del supervisor (launchd/systemd/schtasks) no incluye los valores predeterminados actuales.
* Solución: ejecuta `openclaw doctor` para actualizarla (o `openclaw gateway install --force` para una reescritura completa).

**Si `Last gateway error:` menciona “refusing to bind … without auth”**

* Configuraste `gateway.bind` en un modo que no es loopback (`lan`/`tailnet`/`custom`, o `auto` cuando loopback no está disponible) pero no configuraste autenticación.
* Solución: configura `gateway.auth.mode` + `gateway.auth.token` (o exporta `OPENCLAW_GATEWAY_TOKEN`) y reinicia el servicio.

**Si `openclaw gateway status` indica `bind=tailnet` pero no se encontró ninguna interfaz tailnet**

* El Gateway intentó hacer bind a una IP de Tailscale (100.64.0.0/10), pero no se detectó ninguna en el host.
* Solución: activa Tailscale en esa máquina (o cambia `gateway.bind` a `loopback`/`lan`).

**Si `Probe note:` indica que el sondeo usa loopback**

* Eso es lo esperado para `bind=lan`: el Gateway escucha en `0.0.0.0` (todas las interfaces), y loopback aún debería conectarse localmente.
* Para clientes remotos, usa una IP LAN real (no `0.0.0.0`) más el puerto y asegúrate de que la autenticación esté configurada.

<div id="address-already-in-use-port-18789">
  ### Dirección ya en uso (Puerto 18789)
</div>

Esto significa que ya hay algo escuchando en el puerto del Gateway.

**Comprueba:**

```bash
openclaw gateway status
```

Mostrará los listener(s) y las causas probables (Gateway ya en ejecución, túnel SSH).
Si es necesario, detén el servicio o elige otro puerto.

<div id="extra-workspace-folders-detected">
  ### Carpetas de espacio de trabajo adicionales detectadas
</div>

Si has actualizado desde instalaciones anteriores, es posible que aún tengas `~/openclaw` en el disco.
Tener varios directorios de espacio de trabajo puede provocar problemas de autenticación o inconsistencias de estado, porque
solo un espacio de trabajo está activo.

**Solución:** mantén un único espacio de trabajo activo y archiva/elimina el resto. Consulta
[Espacio de trabajo del Agente](/es/concepts/agent-workspace#extra-workspace-folders).

<div id="main-chat-running-in-a-sandbox-workspace">
  ### Chat principal ejecutándose en un espacio de trabajo de sandbox
</div>

Síntomas: `pwd` o las herramientas de archivos muestran `~/.openclaw/sandboxes/...` aunque
esperabas el espacio de trabajo del host.

**Por qué:** `agents.defaults.sandbox.mode: "non-main"` se basa en `session.mainKey` (por defecto `"main"`).
Las sesiones de grupo o canal usan sus propias claves, por lo que se tratan como no principales y
reciben espacios de trabajo de sandbox.

**Opciones de solución:**

* Si quieres espacios de trabajo del host para un agente: establece `agents.list[].sandbox.mode: "off"`.
* Si quieres acceso al espacio de trabajo del host dentro de la sandbox: establece `workspaceAccess: "rw"` para ese agente.

<div id="agent-was-aborted">
  ### &quot;Se abortó el agente&quot;
</div>

El agente fue interrumpido a mitad de la respuesta.

**Causas:**

* El usuario envió `stop`, `abort`, `esc`, `wait` o `exit`
* Se superó el tiempo de espera
* El proceso se bloqueó

**Solución:** Simplemente envía otro mensaje. La sesión continúa.

<div id="agent-failed-before-reply-unknown-model-anthropicclaude-haiku-3-5">
  ### &quot;Agente falló antes de responder: Unknown model: anthropic/claude-haiku-3-5&quot;
</div>

OpenClaw rechaza de forma intencional **modelos antiguos/inseguros** (especialmente
los más vulnerables a inyección de instrucciones). Si ves este error, el nombre del
modelo ya no es compatible.

**Solución:**

* Elige un modelo **más reciente** del proveedor y actualiza tu configuración o el alias de modelo.
* Si no estás seguro de qué modelos están disponibles, ejecuta `openclaw models list` o
  `openclaw models scan` y elige uno compatible.
* Revisa los registros del Gateway para ver el motivo detallado del fallo.

Consulta también: [CLI de modelos](/es/cli/models) y [Proveedores de modelos](/es/concepts/model-providers).

<div id="messages-not-triggering">
  ### Mensajes que no se activan
</div>

**Comprobación 1:** ¿El remitente está en la lista de permitidos?

```bash
openclaw status
```

Busca `AllowFrom: ...` en la salida.

**Comprobación 2:** En los chats de grupo, ¿es obligatorio usar una mención?

```bash
# El mensaje debe coincidir con mentionPatterns o menciones explícitas; los valores predeterminados se encuentran en grupos/gremios de canal.
# Multi-agente: `agents.list[].groupChat.mentionPatterns` anula los patrones globales.
grep -n "agents\\|groupChat\\|mentionPatterns\\|channels\\.whatsapp\\.groups\\|channels\\.telegram\\.groups\\|channels\\.imessage\\.groups\\|channels\\.discord\\.guilds" \
  "${OPENCLAW_CONFIG_PATH:-$HOME/.openclaw/openclaw.json}"
```

**Comprobación 3:** Revisa los logs

```bash
openclaw logs --follow
# o si quieres filtros rápidos:
tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)" | grep "blocked\\|skip\\|unauthorized"
```

<div id="pairing-code-not-arriving">
  ### El código de emparejamiento no llega
</div>

Si `dmPolicy` es `pairing`, los remitentes desconocidos deberían recibir un código y su mensaje se ignora hasta que se apruebe.

**Comprobación 1:** ¿Ya hay una solicitud pendiente?

```bash
openclaw pairing list <channel>
```

Las solicitudes de emparejamiento DM pendientes están limitadas a **3 por canal** de forma predeterminada. Si la lista está llena, las solicitudes nuevas no generarán un código hasta que se apruebe una o caduque.

**Comprobación 2:** ¿Se creó la solicitud pero no se envió ninguna respuesta?

```bash
openclaw logs --follow | grep "pairing request"
```

**Comprobación 3:** Confirma que `dmPolicy` no esté configurado como `open` (permite aceptar mensajes sin restricciones de cualquier usuario)/`allowlist` para ese canal.

<div id="image-mention-not-working">
  ### Imagen + mención no funciona correctamente
</div>

Problema conocido: cuando envías una imagen con SOLO una mención (sin otro texto), WhatsApp a veces no incluye los metadatos de la mención.

**Solución alternativa:** añade algo de texto junto con la mención:

* ❌ `@openclaw` + imagen
* ✅ `@openclaw mira esto` + imagen

<div id="session-not-resuming">
  ### La sesión no se reanuda
</div>

**Comprobación 1:** ¿Existe el archivo de sesión?

```bash
ls -la ~/.openclaw/agents/<agentId>/sessions/
```

**Comprobación 2:** ¿La ventana de reinicio es demasiado corta?

```json
{
  "session": {
    "reset": {
      "mode": "daily",
      "atHour": 4,
      "idleMinutes": 10080  // 7 días
    }
  }
}
```

**Comprobación 3:** ¿Alguien envió `/new`, `/reset` o un disparador de reinicio?

<div id="agent-timing-out">
  ### Agente supera el tiempo de espera
</div>

El tiempo de espera predeterminado es de 30 minutos. Para tareas largas:

```json
{
  "reply": {
    "timeoutSeconds": 3600  // 1 hora
  }
}
```

O bien usa la herramienta `process` para enviar comandos largos al segundo plano.

<div id="whatsapp-disconnected">
  ### Desconexión de WhatsApp
</div>

```bash
# Verificar estado local (credenciales, sesiones, eventos en cola)
openclaw status
# Sondear el Gateway en ejecución + canales (conexión WA + APIs de Telegram + Discord)
openclaw status --deep

# Ver eventos de conexión recientes
openclaw logs --limit 200 | grep "connection\\|disconnect\\|logout"
```

**Solución:** Suele volver a conectarse automáticamente una vez que el Gateway está en ejecución. Si se queda atascado, reinicia el proceso del Gateway (independientemente de cómo lo supervises) o ejecútalo manualmente con salida detallada:

```bash
openclaw gateway --verbose
```

Si has cerrado sesión o estás desvinculado:

```bash
openclaw channels logout
trash "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/credentials" # si logout no puede eliminar todo correctamente
openclaw channels login --verbose       # re-scan QR
```

<div id="media-send-failing">
  ### Fallo al enviar contenido multimedia
</div>

**Comprobación 1:** ¿La ruta del archivo es válida?

```bash
ls -la /path/to/your/image.jpg
```

**Comprobación 2:** ¿Es demasiado grande?

* Imágenes: tamaño máximo 6 MB
* Audio/Vídeo: tamaño máximo 16 MB
* Documentos: tamaño máximo 100 MB

**Comprobación 3:** Revisa los registros de contenido multimedia

```bash
grep "media\\|fetch\\|download" "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)" | tail -20
```

<div id="high-memory-usage">
  ### Uso elevado de memoria
</div>

OpenClaw mantiene el historial de la conversación en memoria.

**Solución:** Reinicia periódicamente o define límites de sesión:

```json
{
  "session": {
    "historyLimit": 100  // Máx. de mensajes a conservar
  }
}
```

<div id="common-issues">
  ## Problemas frecuentes
</div>

<div id="gateway-wont-start-configuration-invalid">
  ### &quot;Gateway no se inicia: configuración inválida&quot;
</div>

OpenClaw ahora no se iniciará cuando la configuración contenga claves desconocidas, valores con formato incorrecto o tipos no válidos.
Esto es intencional por razones de seguridad.

Soluciónalo con Doctor:

```bash
openclaw doctor
openclaw doctor --fix
```

Notas:

* `openclaw doctor` informa de todas las entradas no válidas.
* `openclaw doctor --fix` aplica migraciones/reparaciones y reescribe la configuración.
* Los comandos de diagnóstico como `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw gateway status` y `openclaw gateway probe` se siguen ejecutando incluso si la configuración es inválida.

<div id="all-models-failed-what-should-i-check-first">
  ### “All models failed” — ¿qué es lo primero que debo comprobar?
</div>

* **Credenciales** configuradas para los proveedores que se están usando (perfiles de autenticación + variables de entorno).
* **Enrutamiento de modelos**: confirma que `agents.defaults.model.primary` y las alternativas de respaldo sean modelos a los que puedas acceder.
* **Registros del Gateway** en `/tmp/openclaw/…` para ver el error exacto del proveedor.
* **Estado del modelo**: usa `/model status` (chat) o `openclaw models status` (CLI).

<div id="im-running-on-my-personal-whatsapp-number-why-is-self-chat-weird">
  ### Estoy usando mi número personal de WhatsApp: ¿por qué el chat conmigo mismo funciona de forma extraña?
</div>

Activa el modo de chat contigo mismo y añade tu propio número a la lista de permitidos:

```json5
{
  channels: {
    whatsapp: {
      selfChatMode: true,
      dmPolicy: "allowlist",
      allowFrom: ["+15555550123"]
    }
  }
}
```

Consulta la sección [Configuración de WhatsApp](/es/channels/whatsapp).

<div id="whatsapp-logged-me-out-how-do-i-reauth">
  ### WhatsApp me cerró la sesión. ¿Cómo vuelvo a autenticarme?
</div>

Ejecuta de nuevo el comando de login y escanea el código QR:

```bash
openclaw channels login
```

<div id="build-errors-on-main-whats-the-standard-fix-path">
  ### Errores de compilación en `main` — ¿cuál es la ruta estándar para solucionarlos?
</div>

1. `git pull origin main && pnpm install`
2. `openclaw doctor`
3. Consulta los issues de GitHub o Discord
4. Solución temporal: cambia a un commit anterior

<div id="npm-install-fails-allow-build-scripts-missing-tar-or-yargs-what-now">
  ### La instalación con npm falla (allow-build-scripts / falta tar o yargs). ¿Y ahora qué?
</div>

Si estás ejecutando desde el código fuente, usa el gestor de paquetes del repositorio: **pnpm** (recomendado).
El repositorio declara `packageManager: "pnpm@…"`.

Procedimiento típico de recuperación:

```bash
git status   # asegúrate de estar en la raíz del repositorio
pnpm install
pnpm build
openclaw doctor
openclaw gateway restart
```

Motivo: pnpm es el gestor de paquetes configurado para este repositorio.

<div id="how-do-i-switch-between-git-installs-and-npm-installs">
  ### ¿Cómo cambio entre instalaciones con git e instalaciones con npm?
</div>

Usa el **instalador web** y selecciona el método de instalación mediante un flag. Se actualiza sobre la instalación existente y reescribe el servicio del Gateway para que apunte a la nueva instalación.

Cambiar **a instalación desde git**:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git --no-onboard
```

Cambia **a la instalación global de npm**:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Notas:

* El flujo de git solo realiza un rebase si el repositorio está limpio. Confirma o guarda los cambios en un stash primero.
* Después de cambiar de rama, ejecuta:
  ```bash
  openclaw doctor
  openclaw gateway restart
  ```

<div id="telegram-block-streaming-isnt-splitting-text-between-tool-calls-why">
  ### El streaming por bloques de Telegram no está dividiendo el texto entre llamadas de herramientas. ¿Por qué?
</div>

El block streaming solo envía **bloques de texto completos**. Motivos habituales por los que ves un único mensaje:

* `agents.defaults.blockStreamingDefault` sigue estando `"off"`.
* `channels.telegram.blockStreaming` está configurado en `false`.
* `channels.telegram.streamMode` está en `partial` o `block` **y el draft streaming está activo**
  (chat privado + temas). El draft streaming desactiva el block streaming en ese caso.
* Tus valores de `minChars` / coalesce son demasiado altos, por lo que los fragmentos se fusionan.
* El modelo emite un único bloque de texto grande (sin puntos de vaciado a mitad de la respuesta).

Lista de verificación para corregirlo:

1. Coloca la configuración de block streaming bajo `agents.defaults`, no en la raíz.
2. Establece `channels.telegram.streamMode: "off"` si quieres respuestas reales de bloques en múltiples mensajes.
3. Usa umbrales más pequeños de fragmento/coalesce mientras depuras.

Consulta [Streaming](/es/concepts/streaming).

<div id="discord-doesnt-reply-in-my-server-even-with-requiremention-false-why">
  ### Discord no responde en mi servidor incluso con `requireMention: false`. ¿Por qué?
</div>

`requireMention` solo controla el filtrado por menciones **después** de que el canal pase la lista de permitidos.
De forma predeterminada, `channels.discord.groupPolicy` es **allowlist**, por lo que los servidores deben habilitarse explícitamente.
Si configuras `channels.discord.guilds.<guildId>.channels`, solo se permiten los canales listados; omítelo para permitir todos los canales del servidor.

Lista de comprobación para la solución:

1. Establece `channels.discord.groupPolicy: "open"` **o bien** añade una entrada en la lista de permitidos para el servidor (y opcionalmente una lista de permitidos de canales).
2. Usa **IDs de canal numéricos** en `channels.discord.guilds.<guildId>.channels`.
3. Coloca `requireMention: false` **debajo de** `channels.discord.guilds` (global o por canal).
   La clave de nivel superior `channels.discord.requireMention` no es compatible.
4. Asegúrate de que el bot tenga **Message Content Intent** y permisos en el canal.
5. Ejecuta `openclaw channels status --probe` para obtener pistas de diagnóstico.

Documentación: [Discord](/es/channels/discord), [Solución de problemas de canales](/es/channels/troubleshooting).

<div id="cloud-code-assist-api-error-invalid-tool-schema-400-what-now">
  ### Cloud Code Assist API error: invalid tool schema (400). What now?
</div>

Esto casi siempre se debe a un problema de **compatibilidad del esquema de herramientas**. El endpoint de Cloud Code Assist
acepta un subconjunto estricto de JSON Schema. OpenClaw limpia/normaliza los esquemas
de herramientas en el `main` actual, pero la solución aún no está incluida en la última
versión (a fecha de 13 de enero de 2026).

Lista de comprobación para solucionarlo:

1. **Actualiza OpenClaw**:
   * Si puedes ejecutarlo desde el código fuente, haz `pull` de `main` y reinicia el Gateway.
   * En caso contrario, espera a la próxima versión que incluya el limpiador de esquemas.
2. Evita palabras clave no compatibles como `anyOf/oneOf/allOf`, `patternProperties`,
   `additionalProperties`, `minLength`, `maxLength`, `format`, etc.
3. Si defines herramientas personalizadas, mantén el esquema de nivel superior con `type: "object"` y
   `properties`, y enums simples.

Consulta [Tools](/es/tools) y [TypeBox schemas](/es/concepts/typebox).

<div id="macos-specific-issues">
  ## Problemas específicos en macOS
</div>

<div id="app-crashes-when-granting-permissions-speechmic">
  ### La app se bloquea al conceder permisos (Voz/Micrófono)
</div>

Si la app desaparece o muestra &quot;Abort trap 6&quot; al hacer clic en &quot;Permitir&quot; en un aviso de privacidad:

**Solución 1: Restablecer la caché TCC**

```bash
tccutil reset All bot.molt.mac.debug
```

**Solución 2: Forzar un nuevo identificador de paquete (BUNDLE&#95;ID)**
Si restablecer no funciona, cambia el `BUNDLE_ID` en [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) (por ejemplo, añade un sufijo `.test`) y vuelve a compilar. Esto obliga a macOS a tratarla como si fuera una aplicación nueva.

<div id="gateway-stuck-on-starting">
  ### Gateway atascado en &quot;Starting...&quot;
</div>

La aplicación se conecta a un Gateway local en el puerto `18789`. Si se queda atascado:

**Solución 1: Detener el supervisor (recomendado)**
Si el Gateway está supervisado por launchd, finalizar el proceso (PID) solo hará que se vuelva a iniciar. Detén primero el supervisor:

```bash
openclaw gateway status
openclaw gateway stop
# O: launchctl bootout gui/$UID/bot.molt.gateway (reemplaza con bot.molt.<profile>; el legado com.openclaw.* aún funciona)
```

**Solución 2: Puerto ocupado (localiza el proceso que lo está usando)**

```bash
lsof -nP -iTCP:18789 -sTCP:LISTEN
```

Si es un proceso no supervisado, intenta primero una detención ordenada y luego escala:

```bash
kill -TERM <PID>
sleep 1
kill -9 <PID> # último recurso
```

**Solución 3: Comprueba la instalación de la CLI**
Asegúrate de que la CLI global `openclaw` esté instalada y que su versión coincida con la de la aplicación:

```bash
openclaw --version
npm install -g openclaw@<version>
```

<div id="debug-mode">
  ## Modo de depuración
</div>

Obtén logs detallados:

```bash
# Activa el registro de trazas en la configuración:
#   ${OPENCLAW_CONFIG_PATH:-$HOME/.openclaw/openclaw.json} -> { logging: { level: "trace" } }
#
# Luego ejecuta comandos con --verbose para reflejar la salida de depuración en stdout:
openclaw gateway --verbose
openclaw channels login --verbose
```

<div id="log-locations">
  ## Ubicaciones de registros
</div>

| Registro | Ubicación |
|-----|----------|
| Registros en archivo del Gateway (estructurados) | `/tmp/openclaw/openclaw-YYYY-MM-DD.log` (o `logging.file`) |
| Registros del servicio del Gateway (supervisor) | macOS: `$OPENCLAW_STATE_DIR/logs/gateway.log` + `gateway.err.log` (predeterminado: `~/.openclaw/logs/...`; los perfiles usan `~/.openclaw-<profile>/logs/...`)<br />Linux: `journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`<br />Windows: `schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST` |
| Archivos de sesión | `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/` |
| Caché de contenido multimedia | `$OPENCLAW_STATE_DIR/media/` |
| Credenciales | `$OPENCLAW_STATE_DIR/credentials/` |

<div id="health-check">
  ## Comprobación del estado
</div>

```bash
# Supervisor + probe target + config paths
openclaw gateway status
# Incluye escaneos a nivel de sistema (servicios legacy/adicionales, listeners de puertos)
openclaw gateway status --deep

# Is the gateway reachable?
openclaw health --json
# If it fails, rerun with connection details:
openclaw health --verbose

# Is something listening on the default port?
lsof -nP -iTCP:18789 -sTCP:LISTEN

# Recent activity (RPC log tail)
openclaw logs --follow
# Fallback if RPC is down
tail -20 /tmp/openclaw/openclaw-*.log
```

<div id="reset-everything">
  ## Restablecer todo
</div>

La opción nuclear:

```bash
openclaw gateway stop
# Si instalaste un servicio y deseas una instalación limpia:
# openclaw gateway uninstall

trash "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
openclaw channels login         # re-pair WhatsApp
openclaw gateway restart           # or: openclaw gateway
```

⚠️ Esto elimina todas las sesiones y requiere volver a emparejar WhatsApp.

<div id="getting-help">
  ## Obtener ayuda
</div>

1. Revisa primero los logs: `/tmp/openclaw/` (predeterminado: `openclaw-YYYY-MM-DD.log`, o tu `logging.file` configurado)
2. Busca issues existentes en GitHub
3. Abre un nuevo issue con:
   * Versión de OpenClaw
   * Fragmentos de logs relevantes
   * Pasos para reproducirlo
   * Tu configuración (¡oculta los secretos!)

***

*&quot;¿Has probado a apagarlo y volverlo a encender?&quot;* — Cualquier persona de TI, siempre

🦞🔧

<div id="browser-not-starting-linux">
  ### El navegador no se inicia (Linux)
</div>

Si ves `"Failed to start Chrome CDP on port 18800"`:

**Causa más probable:** Chromium instalado como paquete Snap en Ubuntu.

**Solución rápida:** Instala Google Chrome en su lugar:

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
```

Luego, establece en la configuración:

```json
{
  "browser": {
    "executablePath": "/usr/bin/google-chrome-stable"
  }
}
```

**Guía completa:** Consulta [browser-linux-troubleshooting](/es/tools/browser-linux-troubleshooting)
