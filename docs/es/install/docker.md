---
title: Docker
summary: "Configuración opcional con Docker y puesta en marcha inicial de OpenClaw"
read_when:
  - Quieres usar un Gateway en contenedor en lugar de instalaciones locales
  - Estás validando el flujo con Docker
---

<div id="docker-optional">
  # Docker (opcional)
</div>

Docker es **opcional**. Úsalo solo si quieres ejecutar el Gateway en un contenedor o para validar el flujo con Docker.

<div id="is-docker-right-for-me">
  ## ¿Es Docker adecuado para mí?
</div>

* **Sí**: quieres un entorno de Gateway aislado y desechable o necesitas ejecutar OpenClaw en un host sin instalaciones locales.
* **No**: estás usando tu propia máquina y solo quieres el ciclo de desarrollo más rápido. En su lugar, usa el flujo de instalación normal.
* **Nota sobre sandboxing**: el sandboxing de agentes también usa Docker, pero **no** requiere que todo el Gateway se ejecute en Docker. Consulta [Sandboxing](/es/gateway/sandboxing).

Esta guía cubre:

* Gateway en contenedor (OpenClaw completo en Docker)
* Sandbox de Agente por sesión (Gateway en el host + herramientas de agente aisladas con Docker)

Detalles sobre el sandboxing: [Sandboxing](/es/gateway/sandboxing)

<div id="requirements">
  ## Requisitos
</div>

* Docker Desktop (o Docker Engine) + Docker Compose v2
* Espacio en disco suficiente para las imágenes y los logs

<div id="containerized-gateway-docker-compose">
  ## Gateway en contenedor con Docker Compose
</div>

<div id="quick-start-recommended">
  ### Inicio rápido (recomendado)
</div>

Desde la raíz del repositorio:

```bash
./docker-setup.sh
```

Este script:

* construye la imagen del Gateway
* ejecuta el asistente de incorporación
* muestra sugerencias opcionales para la configuración de proveedores
* inicia el Gateway mediante Docker Compose
* genera un token del Gateway y lo escribe en `.env`

Variables de entorno opcionales:

* `OPENCLAW_DOCKER_APT_PACKAGES` — instala paquetes apt adicionales durante la compilación
* `OPENCLAW_EXTRA_MOUNTS` — añade montajes bind adicionales del host
* `OPENCLAW_HOME_VOLUME` — persiste `/home/node` en un volumen con nombre

Cuando finalice:

* Abre `http://127.0.0.1:18789/` en tu navegador.
* Pega el token en el Control UI (Settings → token).

Escribe la configuración y el espacio de trabajo en el host:

* `~/.openclaw/`
* `~/.openclaw/workspace`

¿Lo estás ejecutando en un VPS? Consulta [Hetzner (Docker VPS)](/es/platforms/hetzner).

<div id="manual-flow-compose">
  ### Flujo manual (Compose)
</div>

```bash
docker build -t openclaw:local -f Dockerfile .
docker compose run --rm openclaw-cli onboard
docker compose up -d openclaw-gateway
```

<div id="extra-mounts-optional">
  ### Montajes adicionales (opcional)
</div>

Si quieres montar directorios adicionales del host en los contenedores, define
`OPENCLAW_EXTRA_MOUNTS` antes de ejecutar `docker-setup.sh`. Esta variable acepta una
lista separada por comas de montajes bind de Docker y los aplica tanto a
`openclaw-gateway` como a `openclaw-cli`, generando `docker-compose.extra.yml`.

Ejemplo:

```bash
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

Notas:

* Las rutas deben compartirse con Docker Desktop en macOS/Windows.
* Si editas `OPENCLAW_EXTRA_MOUNTS`, vuelve a ejecutar `docker-setup.sh` para regenerar el
  archivo compose adicional.
* `docker-compose.extra.yml` se genera automáticamente. No lo modifiques manualmente.

<div id="persist-the-entire-container-home-optional">
  ### Conservar todo el home del contenedor (opcional)
</div>

Si quieres que `/home/node` persista a través de recreaciones del contenedor, define un
volumen con nombre mediante `OPENCLAW_HOME_VOLUME`. Esto crea un volumen de Docker y lo monta en
`/home/node`, manteniendo al mismo tiempo los bind mounts estándar de configuración/espacio de trabajo. Usa un
volumen con nombre aquí (no una ruta bind); para los bind mounts, usa
`OPENCLAW_EXTRA_MOUNTS`.

Ejemplo:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

Puedes combinar esto con puntos de montaje adicionales:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

Notas:

* Si modificas `OPENCLAW_HOME_VOLUME`, vuelve a ejecutar `docker-setup.sh` para regenerar
  el archivo adicional de Compose.
* El volumen con nombre persiste hasta que se elimine con `docker volume rm <name>`.

<div id="install-extra-apt-packages-optional">
  ### Instalar paquetes adicionales de apt (opcional)
</div>

Si necesitas paquetes de sistema dentro de la imagen (por ejemplo, herramientas de compilación o bibliotecas multimedia), establece `OPENCLAW_DOCKER_APT_PACKAGES` antes de ejecutar `docker-setup.sh`.
Esto instala los paquetes durante la compilación de la imagen, de modo que se conservan incluso si se elimina el contenedor.

Ejemplo:

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="ffmpeg build-essential"
./docker-setup.sh
```

Notas:

* Esto acepta una lista de nombres de paquetes de apt separados por espacios.
* Si modificas `OPENCLAW_DOCKER_APT_PACKAGES`, vuelve a ejecutar `docker-setup.sh` para reconstruir la imagen.

<div id="faster-rebuilds-recommended">
  ### Reconstrucciones más rápidas (recomendado)
</div>

Para acelerar las reconstrucciones, ordena tu Dockerfile de manera que las capas de dependencias queden en caché.
Esto evita volver a ejecutar `pnpm install` a menos que cambien los archivos de bloqueo (lockfiles):

```dockerfile
FROM node:22-bookworm

# Instalar Bun (requerido para scripts de compilación)
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:${PATH}"

RUN corepack enable

WORKDIR /app

# Cachear dependencias a menos que cambien los metadatos del paquete
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

<div id="channel-setup-optional">
  ### Configuración de canales (opcional)
</div>

Utiliza el contenedor de la CLI para configurar los canales y, si es necesario, reinicia el Gateway.

WhatsApp (QR):

```bash
docker compose run --rm openclaw-cli channels login
```

Telegram (token del bot):

```bash
docker compose run --rm openclaw-cli channels add --channel telegram --token "<token>"
```

Discord (token del bot):

```bash
docker compose run --rm openclaw-cli channels add --channel discord --token "<token>"
```

Documentación: [WhatsApp](/es/channels/whatsapp), [Telegram](/es/channels/telegram), [Discord](/es/channels/discord)

<div id="health-check">
  ### Comprobación de estado
</div>

```bash
docker compose exec openclaw-gateway node dist/index.js health --token "$OPENCLAW_GATEWAY_TOKEN"
```

<div id="e2e-smoke-test-docker">
  ### Prueba básica E2E (Docker)
</div>

```bash
scripts/e2e/onboard-docker.sh
```

<div id="qr-import-smoke-test-docker">
  ### Prueba de humo de importación mediante QR (Docker)
</div>

```bash
pnpm test:docker:qr
```

<div id="notes">
  ### Notas
</div>

* El bind del Gateway usa `lan` como valor predeterminado para uso en contenedores.
* El contenedor del Gateway es la fuente de la verdad para las sesiones (`~/.openclaw/agents/<agentId>/sessions/`).

<div id="agent-sandbox-host-gateway-docker-tools">
  ## Sandbox del agente (Gateway en el host + herramientas Docker)
</div>

Análisis detallado: [Sandboxing](/es/gateway/sandboxing)

<div id="what-it-does">
  ### Qué hace
</div>

Cuando `agents.defaults.sandbox` está habilitado, las **sesiones no principales** ejecutan herramientas dentro de un contenedor Docker. El Gateway permanece en tu host, pero la ejecución de las herramientas está aislada:

* scope: `"agent"` por defecto (un contenedor + espacio de trabajo por agente)
* scope: `"session"` para aislamiento por sesión
* carpeta de espacio de trabajo por scope montada en `/workspace`
* acceso opcional al espacio de trabajo del agente (`agents.defaults.sandbox.workspaceAccess`)
* política de allow/deny para herramientas (deny tiene prioridad)
* el contenido multimedia entrante se copia en el espacio de trabajo del sandbox activo (`media/inbound/*`) para que las herramientas puedan leerlo (con `workspaceAccess: "rw"`, esto termina en el espacio de trabajo del agente)

Advertencia: `scope: "shared"` deshabilita el aislamiento entre sesiones. Todas las sesiones comparten un contenedor y un espacio de trabajo.

<div id="per-agent-sandbox-profiles-multi-agent">
  ### Perfiles de sandbox por agente (multi-agente)
</div>

Si usas enrutamiento multi-agente, cada agente puede sobrescribir la configuración de sandbox y herramientas:
`agents.list[].sandbox` y `agents.list[].tools` (además de `agents.list[].tools.sandbox.tools`). Esto te permite ejecutar
niveles de acceso mixtos en un mismo Gateway:

* Acceso completo (agente personal)
* Herramientas de solo lectura + espacio de trabajo de solo lectura (agente de trabajo/familiar)
* Sin herramientas de sistema de archivos/shell (agente público)

Consulta [Sandbox y herramientas multi-agente](/es/multi-agent-sandbox-tools) para ver ejemplos,
precedencia y resolución de problemas.

<div id="default-behavior">
  ### Comportamiento predeterminado
</div>

* Imagen: `openclaw-sandbox:bookworm-slim`
* Un contenedor por agente
* Acceso al espacio de trabajo del agente: `workspaceAccess: "none"` (predeterminado) usa `~/.openclaw/sandboxes`
  * `"ro"` mantiene el espacio de trabajo del sandbox en `/workspace` y monta el espacio de trabajo del agente en modo solo lectura en `/agent` (desactiva `write`/`edit`/`apply_patch`)
  * `"rw"` monta el espacio de trabajo del agente con lectura y escritura en `/workspace`
* Purga automática: inactividad &gt; 24h O antigüedad &gt; 7d
* Red: `none` de forma predeterminada (actívala explícitamente si necesitas salida a Internet)
* Permitido de forma predeterminada: `exec`, `process`, `read`, `write`, `edit`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* Denegado de forma predeterminada: `browser`, `canvas`, `nodes`, `cron`, `discord`, `gateway`

<div id="enable-sandboxing">
  ### Habilitar sandboxing
</div>

Si planeas instalar paquetes en `setupCommand`, ten en cuenta lo siguiente:

* El valor predeterminado de `docker.network` es `"none"` (sin tráfico saliente).
* `readOnlyRoot: true` bloquea la instalación de paquetes.
* `user` debe ser root para `apt-get` (omite `user` o establece `user: "0:0"`).
  OpenClaw vuelve a crear automáticamente los contenedores cuando `setupCommand` (o la configuración de Docker) cambia
  a menos que el contenedor se haya **usado recientemente** (en los últimos ~5 minutos). Los contenedores activos
  registran una advertencia con el comando exacto `openclaw sandbox recreate ...`.

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent es el predeterminado)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"]
        },
        prune: {
          idleHours: 24, // 0 disables idle pruning
          maxAgeDays: 7  // 0 disables max-age pruning
        }
      }
    }
  },
  tools: {
    sandbox: {
      tools: {
        allow: ["exec", "process", "read", "write", "edit", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"]
      }
    }
  }
}
```

Los parámetros de hardening se encuentran bajo `agents.defaults.sandbox.docker`:
`network`, `user`, `pidsLimit`, `memory`, `memorySwap`, `cpus`, `ulimits`,
`seccompProfile`, `apparmorProfile`, `dns`, `extraHosts`.

Multi-agente: anula `agents.defaults.sandbox.{docker,browser,prune}.*` por agente mediante `agents.list[].sandbox.{docker,browser,prune}.*`
(se ignora cuando `agents.defaults.sandbox.scope` / `agents.list[].sandbox.scope` es `"shared"`).

<div id="build-the-default-sandbox-image">
  ### Construir la imagen de sandbox predeterminada
</div>

```bash
scripts/sandbox-setup.sh
```

Esto construye la imagen `openclaw-sandbox:bookworm-slim` usando `Dockerfile.sandbox`.

<div id="sandbox-common-image-optional">
  ### Imagen común de sandbox (opcional)
</div>

Si quieres una imagen de sandbox con herramientas de compilación habituales (Node, Go, Rust, etc.), compila la imagen común:

```bash
scripts/sandbox-common-setup.sh
```

Esto construye la imagen `openclaw-sandbox-common:bookworm-slim`. Para usarla:

```json5
{
  agents: { defaults: { sandbox: { docker: { image: "openclaw-sandbox-common:bookworm-slim" } } } }
}
```

<div id="sandbox-browser-image">
  ### Imagen de navegador de la sandbox
</div>

Para ejecutar la herramienta de navegador dentro de la sandbox, compila la imagen del navegador:

```bash
scripts/sandbox-browser-setup.sh
```

Esto construye la imagen `openclaw-sandbox-browser:bookworm-slim` usando
`Dockerfile.sandbox-browser`. El contenedor ejecuta Chromium con CDP habilitado y
un visor noVNC opcional (modo con interfaz gráfica mediante Xvfb).

Notas:

* El modo con interfaz gráfica (Xvfb) reduce el bloqueo de bots frente al modo sin interfaz (`headless`).
* El modo `headless` todavía se puede usar configurando `agents.defaults.sandbox.browser.headless=true`.
* No se necesita un entorno de escritorio completo (GNOME); Xvfb proporciona la pantalla.

Usa esta configuración:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: { enabled: true }
      }
    }
  }
}
```

Imagen personalizada del navegador:

```json5
{
  agents: {
    defaults: {
      sandbox: { browser: { image: "my-openclaw-browser" } }
    }
  }
}
```

Cuando está habilitado, el agente recibe:

* una URL de control del navegador de la sandbox (para la herramienta `browser`)
* una URL de noVNC (si está habilitado y headless=false)

Recuerda: si utilizas una lista de permitidos para herramientas, agrega `browser`
(y quítalo de deny) o la herramienta seguirá bloqueada.
Las reglas de limpieza (`agents.defaults.sandbox.prune`) también se aplican a los contenedores del navegador.

<div id="custom-sandbox-image">
  ### Imagen de sandbox personalizada
</div>

Crea tu propia imagen y haz que la configuración apunte a ella:

```bash
docker build -t my-openclaw-sbx -f Dockerfile.sandbox .
```

```json5
{
  agents: {
    defaults: {
      sandbox: { docker: { image: "my-openclaw-sbx" } }
    }
  }
}
```

<div id="tool-policy-allowdeny">
  ### Política de herramientas (permitir/denegar)
</div>

* `deny` tiene prioridad sobre `allow`.
* Si `allow` está vacío: todas las herramientas (excepto las denegadas) están disponibles.
* Si `allow` no está vacío: solo están disponibles las herramientas incluidas en `allow` (excepto las denegadas).

<div id="pruning-strategy">
  ### Estrategia de limpieza
</div>

Dos ajustes:

* `prune.idleHours`: elimina contenedores que no se hayan usado en X horas (0 = desactivar)
* `prune.maxAgeDays`: elimina contenedores con más de X días (0 = desactivar)

Ejemplo:

* Mantener sesiones activas pero limitar su vida útil:
  `idleHours: 24`, `maxAgeDays: 7`
* Nunca realizar limpieza:
  `idleHours: 0`, `maxAgeDays: 0`

<div id="security-notes">
  ### Notas de seguridad
</div>

* Hard wall solo se aplica a las **tools** (exec/read/write/edit/apply&#95;patch).
* Las tools que se ejecutan solo en el host, como browser/camera/canvas, están bloqueadas de forma predeterminada.
* Permitir `browser` en el sandbox **rompe el aislamiento** (el navegador se ejecuta en el host).

<div id="troubleshooting">
  ## Solución de problemas
</div>

* Imagen no encontrada: construye con [`scripts/sandbox-setup.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/sandbox-setup.sh) o configura `agents.defaults.sandbox.docker.image`.
* Contenedor no en ejecución: se creará automáticamente por sesión bajo demanda.
* Errores de permisos en el sandbox: configura `docker.user` en un UID:GID que coincida con el
  propietario del espacio de trabajo montado (o cambia el propietario de la carpeta del espacio de trabajo con chown).
* Herramientas personalizadas no se encuentran: OpenClaw ejecuta comandos con `sh -lc` (login shell), que
  carga `/etc/profile` y puede restablecer el PATH. Configura `docker.env.PATH` para anteponer tus
  rutas de herramientas personalizadas (por ejemplo, `/custom/bin:/usr/local/share/npm-global/bin`), o agrega
  un script en `/etc/profile.d/` en tu Dockerfile.