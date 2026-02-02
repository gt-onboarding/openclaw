---
title: Doctor
summary: "Comando Doctor: comprobaciones de estado, migraciones de configuración y pasos de reparación"
read_when:
  - Agregar o modificar migraciones de Doctor
  - Introducir cambios de configuración con rupturas incompatibles
---

<div id="doctor">
  # Doctor
</div>

`openclaw doctor` es la herramienta de reparación y migración para OpenClaw. Corrige configuración y estado obsoletos, verifica el estado de salud del sistema y proporciona pasos concretos para la reparación.

<div id="quick-start">
  ## Guía rápida
</div>

```bash
openclaw doctor
```

<div id="headless-automation">
  ### Modo sin interfaz / automatización
</div>

```bash
openclaw doctor --yes
```

Acepta los valores predeterminados sin pedir confirmación (incluidos los pasos de reinicio/servicio/reparación de sandbox cuando corresponda).

```bash
openclaw doctor --repair
```

Aplicar las reparaciones recomendadas sin pedir confirmación (reparaciones + reinicios cuando sea seguro).

```bash
openclaw doctor --repair --force
```

Aplica también correcciones agresivas (sobrescribe las configuraciones personalizadas de supervisor).

```bash
openclaw doctor --non-interactive
```

Se ejecuta sin solicitudes de confirmación y solo aplica migraciones seguras (normalización de configuración + movimientos de estado en disco). Omite acciones de reinicio/servicio/sandbox que requieren confirmación humana.
Las migraciones de estado heredado se ejecutan automáticamente cuando se detectan.

```bash
openclaw doctor --deep
```

Escanea los servicios del sistema en busca de instalaciones adicionales del Gateway (launchd/systemd/schtasks).

Si quieres revisar los cambios antes de guardarlos, abre primero el archivo de configuración:

```bash
cat ~/.openclaw/openclaw.json
```

<div id="what-it-does-summary">
  ## Qué hace (resumen)
</div>

* Actualización previa opcional para instalaciones desde git (solo en modo interactivo).
* Comprobación de vigencia del protocolo de UI (reconstruye el Control UI cuando el esquema de protocolo es más reciente).
* Comprobación de estado y aviso para reiniciar.
* Resumen del estado de las habilidades (aptas/faltantes/bloqueadas).
* Normalización de la configuración para valores heredados.
* Advertencias de anulación del proveedor OpenCode Zen (`models.providers.opencode`).
* Migración de estado heredado en disco (sesiones/directorio de agente/autenticación de WhatsApp).
* Comprobaciones de integridad del estado y permisos (sesiones, transcripciones, directorio de estado).
* Comprobaciones de permisos del archivo de configuración (`chmod 600`) cuando se ejecuta localmente.
* Estado de la autenticación de modelos: comprueba el vencimiento de OAuth, puede renovar tokens próximos a expirar e informa estados de enfriamiento/desactivación de perfiles de autenticación.
* Detección de directorios de espacio de trabajo adicionales (`~/openclaw`).
* Reparación de la imagen de sandbox cuando el sandboxing está habilitado.
* Migración de servicios heredados y detección de Gateways adicionales.
* Comprobaciones de tiempo de ejecución del Gateway (servicio instalado pero no en ejecución; etiqueta `launchd` en caché).
* Advertencias sobre el estado de los canales (sondeadas desde el Gateway en ejecución).
* Auditoría de la configuración del supervisor (`launchd`/`systemd`/`schtasks`) con reparación opcional.
* Comprobaciones de mejores prácticas de tiempo de ejecución del Gateway (Node vs Bun, rutas de gestores de versiones).
* Diagnóstico de colisión de puerto del Gateway (por defecto `18789`).
* Advertencias de seguridad para políticas de mensajes directos con ajuste `open`.
* Advertencias de autenticación del Gateway cuando no se ha establecido `gateway.auth.token` (modo local; ofrece generación de token).
* Comprobación de `systemd linger` en Linux.
* Comprobaciones de instalación desde código fuente (incongruencia de espacio de trabajo de pnpm, recursos de UI faltantes, binario `tsx` faltante).
* Escribe la configuración actualizada y los metadatos del asistente.

<div id="detailed-behavior-and-rationale">
  ## Funcionamiento detallado y fundamentos
</div>

<div id="0-optional-update-git-installs">
  ### 0) Actualización opcional (instalaciones desde git)
</div>

Si se trata de un checkout de git y doctor se está ejecutando en modo interactivo, ofrece
actualizar (fetch/rebase/build) antes de ejecutar doctor.

<div id="1-config-normalization">
  ### 1) Normalización de la configuración
</div>

Si la configuración contiene formatos de valores heredados (por ejemplo, `messages.ackReaction`
sin una personalización específica por canal), doctor los normaliza al esquema
actual.

<div id="2-legacy-config-key-migrations">
  ### 2) Migraciones de claves de configuración heredadas
</div>

Cuando la configuración contiene claves heredadas, otros comandos se niegan a ejecutarse y te piden
que ejecutes `openclaw doctor`.

Doctor hará lo siguiente:

* Explicar qué claves heredadas se encontraron.
* Mostrar la migración que aplicó.
* Reescribir `~/.openclaw/openclaw.json` con el esquema actualizado.

El Gateway también ejecuta automáticamente las migraciones de doctor al iniciar cuando detecta un
formato de configuración heredado, de modo que las configuraciones desactualizadas se reparan sin intervención manual.

Migraciones actuales:

* `routing.allowFrom` → `channels.whatsapp.allowFrom`
* `routing.groupChat.requireMention` → `channels.whatsapp/telegram/imessage.groups."*".requireMention`
* `routing.groupChat.historyLimit` → `messages.groupChat.historyLimit`
* `routing.groupChat.mentionPatterns` → `messages.groupChat.mentionPatterns`
* `routing.queue` → `messages.queue`
* `routing.bindings` → `bindings` de nivel superior
* `routing.agents`/`routing.defaultAgentId` → `agents.list` + `agents.list[].default`
* `routing.agentToAgent` → `tools.agentToAgent`
* `routing.transcribeAudio` → `tools.media.audio.models`
* `bindings[].match.accountID` → `bindings[].match.accountId`
* `identity` → `agents.list[].identity`
* `agent.*` → `agents.defaults` + `tools.*` (tools/elevated/exec/sandbox/subagents)
* `agent.model`/`allowedModels`/`modelAliases`/`modelFallbacks`/`imageModelFallbacks`
  → `agents.defaults.models` + `agents.defaults.model.primary/fallbacks` + `agents.defaults.imageModel.primary/fallbacks`

<div id="2b-opencode-zen-provider-overrides">
  ### 2b) Sobrescrituras del proveedor OpenCode Zen
</div>

Si has añadido `models.providers.opencode` (o `opencode-zen`) manualmente, eso
sobrescribe el catálogo integrado de OpenCode Zen de `@mariozechner/pi-ai`. Eso puede
forzar que todos los modelos usen una única API o deje los costes en cero. Doctor te advertirá para que puedas
eliminar la sobrescritura y restaurar el enrutamiento de API por modelo y los costes.

<div id="3-legacy-state-migrations-disk-layout">
  ### 3) Migraciones de estado heredado (distribución en disco)
</div>

Doctor puede migrar distribuciones antiguas en disco a la estructura actual:

* Almacenamiento de sesiones + transcripciones:
  * de `~/.openclaw/sessions/` a `~/.openclaw/agents/<agentId>/sessions/`
* Directorio del agente:
  * de `~/.openclaw/agent/` a `~/.openclaw/agents/<agentId>/agent/`
* Estado de autenticación de WhatsApp (Baileys):
  * de la ruta heredada `~/.openclaw/credentials/*.json` (excepto `oauth.json`)
  * a `~/.openclaw/credentials/whatsapp/<accountId>/...` (ID de cuenta predeterminado: `default`)

Estas migraciones son de mejor esfuerzo e idempotentes; doctor emitirá advertencias si deja carpetas heredadas como copias de seguridad. El Gateway/CLI también migra automáticamente
las sesiones heredadas y el directorio del agente al iniciarse, de modo que el historial, la autenticación y los modelos queden ubicados en la
ruta específica de cada agente sin necesidad de ejecutar manualmente doctor. La autenticación de WhatsApp se migra intencionalmente solo
a través de `openclaw doctor`.

<div id="4-state-integrity-checks-session-persistence-routing-and-safety">
  ### 4) State integrity checks (session persistence, routing, and safety)
</div>

El directorio de estado es el tronco cerebral operativo. Si desaparece, pierdes
sesiones, credenciales, registros y configuración (a menos que tengas copias de seguridad en otro lugar).

Comprobaciones de doctor:

* **State dir missing**: advierte sobre una pérdida catastrófica de estado, te pide que vuelvas a crear
  el directorio y te recuerda que no puede recuperar los datos que falten.
* **State dir permissions**: verifica que se pueda escribir; ofrece reparar los permisos
  (y emite una sugerencia de `chown` cuando se detecta una discrepancia de propietario/grupo).
* **Session dirs missing**: `sessions/` y el directorio de almacenamiento de sesiones son
  necesarios para persistir el historial y evitar errores `ENOENT`.
* **Transcript mismatch**: advierte cuando las entradas recientes de la sesión tienen archivos
  de transcripción ausentes.
* **Main session “1-line JSONL”**: marca cuando la transcripción principal solo tiene una
  línea (el historial no se está acumulando).
* **Multiple state dirs**: advierte cuando existen varias carpetas `~/.openclaw` en distintos
  directorios de inicio o cuando `OPENCLAW_STATE_DIR` apunta a otro lugar (el historial puede
  dividirse entre instalaciones).
* **Remote mode reminder**: si `gateway.mode=remote`, doctor te recuerda que debes ejecutarlo
  en el host remoto (el estado vive allí).
* **Config file permissions**: advierte si `~/.openclaw/openclaw.json` es legible por grupo/otros
  y ofrece restringir los permisos a `600`.

<div id="5-model-auth-health-oauth-expiry">
  ### 5) Estado de autenticación de modelos (caducidad de OAuth)
</div>

Doctor inspecciona los perfiles OAuth en el almacén de autenticación, advierte cuando los tokens están a punto de caducar o ya han caducado y puede renovarlos cuando es seguro hacerlo. Si el perfil de Anthropic Claude Code está obsoleto, sugiere ejecutar `claude setup-token` (o pegar un setup-token). Los avisos de renovación solo aparecen cuando se ejecuta de forma interactiva (TTY); `--non-interactive` omite los intentos de renovación.

Doctor también informa sobre perfiles de autenticación que son temporalmente inutilizables debido a:

* breves periodos de enfriamiento (límites de frecuencia/timeouts/errores de autenticación)
* desactivaciones más prolongadas (fallos de facturación/crédito)

<div id="6-hooks-model-validation">
  ### 6) Validación del modelo de hooks
</div>

Si `hooks.gmail.model` está configurado, doctor valida la referencia del modelo contra el
catálogo y la lista de permitidos, y advierte cuando no se puede resolver o no está permitido.

<div id="7-sandbox-image-repair">
  ### 7) Reparación de la imagen de sandbox
</div>

Cuando el sandboxing está habilitado, `doctor` comprueba las imágenes de Docker y ofrece crear o usar nombres heredados si la imagen actual no existe.

<div id="8-gateway-service-migrations-and-cleanup-hints">
  ### 8) Migraciones del servicio Gateway y sugerencias de limpieza
</div>

Doctor detecta servicios Gateway heredados (launchd/systemd/schtasks) y
ofrece eliminarlos e instalar el servicio OpenClaw utilizando el puerto
actual del Gateway. También puede escanear servicios adicionales similares a Gateway e imprimir sugerencias de limpieza.
Los servicios Gateway de OpenClaw con nombre de perfil se consideran de primera clase y no se marcan como «extra».

<div id="9-security-warnings">
  ### 9) Advertencias de seguridad
</div>

Doctor emite advertencias cuando un proveedor está configurado como open (acepta mensajes directos de cualquiera) sin una lista de permitidos, o cuando una política está configurada de forma peligrosa.

<div id="10-systemd-linger-linux">
  ### 10) systemd linger (Linux)
</div>

Cuando se ejecuta como un servicio de usuario de systemd, doctor verifica que `lingering` esté habilitado para que el Gateway siga en ejecución después de cerrar la sesión.

<div id="11-skills-status">
  ### 11) Estado de las habilidades
</div>

Doctor muestra un resumen rápido de las habilidades válidas/faltantes/bloqueadas para el espacio de trabajo actual.

<div id="12-gateway-auth-checks-local-token">
  ### 12) Comprobaciones de autenticación del Gateway (token local)
</div>

Doctor advierte cuando falta `gateway.auth` en un Gateway local y ofrece
generar un token. Usa `openclaw doctor --generate-gateway-token` para forzar
la creación de un token en automatización.

<div id="13-gateway-health-check-restart">
  ### 13) Comprobación del estado del Gateway + reinicio
</div>

Doctor ejecuta una comprobación del estado y ofrece reiniciar el Gateway cuando
detecta que no está en buen estado.

<div id="14-channel-status-warnings">
  ### 14) Advertencias de estado de canales
</div>

Si el Gateway está funcionando correctamente, `doctor` ejecuta un sondeo del estado de los canales y comunica
advertencias con correcciones sugeridas.

<div id="15-supervisor-config-audit-repair">
  ### 15) Auditoría y reparación de la configuración del supervisor
</div>

Doctor comprueba la configuración del supervisor instalada (launchd/systemd/schtasks) para detectar valores predeterminados ausentes u obsoletos (por ejemplo, dependencias de `network-online` en systemd y retraso de reinicio). Cuando encuentra una discrepancia, recomienda una actualización y puede reescribir el archivo de servicio/tarea con los valores predeterminados actuales.

Notas:

* `openclaw doctor` solicita confirmación antes de reescribir la configuración del supervisor.
* `openclaw doctor --yes` acepta las solicitudes de reparación predeterminadas.
* `openclaw doctor --repair` aplica las correcciones recomendadas sin solicitar confirmación.
* `openclaw doctor --repair --force` sobrescribe configuraciones personalizadas del supervisor.
* Siempre puedes forzar una reescritura completa mediante `openclaw gateway install --force`.

<div id="16-gateway-runtime-port-diagnostics">
  ### 16) Diagnóstico del runtime y puertos del Gateway
</div>

Doctor inspecciona el runtime del servicio (PID, último estado de salida) y advierte cuando el
servicio está instalado pero no se está ejecutando. También verifica colisiones de puertos
en el puerto del Gateway (por defecto `18789`) y muestra causas probables (Gateway ya
en ejecución, túnel SSH).

<div id="17-gateway-runtime-best-practices">
  ### 17) Mejores prácticas de ejecución del Gateway
</div>

Doctor muestra advertencias cuando el servicio Gateway se ejecuta en Bun o en una ruta de Node gestionada por un gestor de versiones
(`nvm`, `fnm`, `volta`, `asdf`, etc.). Los canales de WhatsApp y Telegram requieren Node,
y las rutas gestionadas por un gestor de versiones pueden dejar de funcionar después de las actualizaciones porque el servicio no
carga los scripts de inicialización de tu shell. Doctor ofrece migrar a una instalación de Node del sistema cuando
está disponible (Homebrew/apt/choco).

<div id="18-config-write-wizard-metadata">
  ### 18) Escritura de configuración + metadatos del asistente
</div>

Doctor guarda de forma permanente cualquier cambio de configuración y añade metadatos del asistente para registrar la ejecución de Doctor.

<div id="19-workspace-tips-backup-memory-system">
  ### 19) Consejos sobre el espacio de trabajo (copias de seguridad + sistema de memoria)
</div>

Doctor sugiere un sistema de memoria del espacio de trabajo cuando no existe y muestra un consejo de copia de seguridad
si el espacio de trabajo aún no está bajo git.

Consulta [/concepts/agent-workspace](/es/concepts/agent-workspace) para obtener una guía completa sobre
la estructura del espacio de trabajo y las copias de seguridad con git (se recomienda usar repositorios privados en GitHub o GitLab).