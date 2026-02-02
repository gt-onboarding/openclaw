---
title: Hetzner
summary: "Ejecuta OpenClaw Gateway 24/7 en un VPS económico de Hetzner (Docker) con estado persistente y binarios integrados de serie"
read_when:
  - Quieres ejecutar OpenClaw 24/7 en un VPS en la nube (no en tu portátil)
  - Quieres un Gateway siempre activo y de nivel de producción en tu propio VPS
  - Quieres control total sobre la persistencia, los binarios y el comportamiento de reinicio
  - Estás ejecutando OpenClaw en Docker en Hetzner o en un proveedor similar
---

<div id="openclaw-on-hetzner-docker-production-vps-guide">
  # OpenClaw en Hetzner (Docker, guía de VPS de producción)
</div>

<div id="goal">
  ## Objetivo
</div>

Ejecutar un Gateway OpenClaw persistente en un VPS de Hetzner usando Docker, con estado persistente, binarios integrados y un comportamiento de reinicio seguro.

Si quieres “OpenClaw 24/7 por ~5 $”, esta es la configuración fiable más sencilla.
Los precios de Hetzner cambian; elige el VPS Debian/Ubuntu más pequeño y escálalo si te encuentras con errores de falta de memoria (OOM).

<div id="what-are-we-doing-simple-terms">
  ## ¿Qué vamos a hacer (en términos simples)?
</div>

* Alquilar un pequeño servidor Linux (Hetzner VPS)
* Instalar Docker (entorno de ejecución de aplicaciones aislado)
* Iniciar el Gateway de OpenClaw en Docker
* Conservar `~/.openclaw` + `~/.openclaw/workspace` en el host (sobrevive a reinicios y reconstrucciones)
* Acceder al Control UI desde tu portátil mediante un túnel SSH

Se puede acceder al Gateway mediante:

* Reenvío de puertos SSH desde tu portátil
* Exposición directa de puertos si te encargas tú mismo del firewall y de los tokens

Esta guía asume Ubuntu o Debian en Hetzner.\
Si estás en otro VPS Linux, ajusta los paquetes en consecuencia.
Para el flujo genérico con Docker, consulta [Docker](/es/install/docker).

***

<div id="quick-path-experienced-operators">
  ## Ruta rápida (operadores con experiencia)
</div>

1. Aprovisionar VPS de Hetzner
2. Instalar Docker
3. Clonar el repositorio de OpenClaw
4. Crear directorios persistentes en el host
5. Configurar `.env` y `docker-compose.yml`
6. Incluir los binarios necesarios en la imagen
7. `docker compose up -d`
8. Verificar la persistencia y el acceso al Gateway

***

<div id="what-you-need">
  ## Lo que necesitas
</div>

* VPS de Hetzner con acceso root
* Acceso SSH desde tu portátil
* Conocimientos básicos de SSH y copiar/pegar
* ~20 minutos
* Docker y Docker Compose
* Credenciales de autenticación para modelos
* Credenciales opcionales para proveedores
  * QR de WhatsApp
  * Token de bot de Telegram
  * OAuth de Gmail

***

<div id="1-provision-the-vps">
  ## 1) Provisiona el VPS
</div>

Crea un VPS con Ubuntu o Debian en Hetzner.

Conéctate como root:

```bash
ssh root@TU_IP_VPS
```

Esta guía asume que el VPS mantiene estado.
No lo trates como infraestructura desechable.

***

<div id="2-install-docker-on-the-vps">
  ## 2) Instalar Docker (en tu VPS)
</div>

```bash
apt-get update
apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sh
```

Comprueba:

```bash
docker --version
docker compose version
```

***

<div id="3-clone-the-openclaw-repository">
  ## 3) Clonar el repositorio de OpenClaw
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

Esta guía asume que vas a crear una imagen personalizada para garantizar la persistencia de los binarios.

***

<div id="4-create-persistent-host-directories">
  ## 4) Crear directorios persistentes en el host
</div>

Los contenedores de Docker son efímeros.
Todo el estado persistente debe almacenarse en el host.

```bash
mkdir -p /root/.openclaw
mkdir -p /root/.openclaw/workspace

# Establecer la propiedad al usuario del contenedor (uid 1000):
chown -R 1000:1000 /root/.openclaw
chown -R 1000:1000 /root/.openclaw/workspace
```

***

<div id="5-configure-environment-variables">
  ## 5) Configura las variables de entorno
</div>

Crea el archivo `.env` en la raíz del repositorio.

```bash
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/root/.openclaw
OPENCLAW_WORKSPACE_DIR=/root/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

Genera secretos seguros:

```bash
openssl rand -hex 32
```

**No hagas commit de este archivo.**

***

<div id="6-docker-compose-configuration">
  ## 6) Configuración de Docker Compose
</div>

Crea o actualiza el archivo `docker-compose.yml`.

```yaml
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE}
    build: .
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - HOME=/home/node
      - NODE_ENV=production
      - TERM=xterm-256color
      - OPENCLAW_GATEWAY_BIND=${OPENCLAW_GATEWAY_BIND}
      - OPENCLAW_GATEWAY_PORT=${OPENCLAW_GATEWAY_PORT}
      - OPENCLAW_GATEWAY_TOKEN=${OPENCLAW_GATEWAY_TOKEN}
      - GOG_KEYRING_PASSWORD=${GOG_KEYRING_PASSWORD}
      - XDG_CONFIG_HOME=${XDG_CONFIG_HOME}
      - PATH=/home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      # Recomendado: mantén el Gateway solo en loopback en el VPS; accede mediante túnel SSH.
      # Para exponerlo públicamente, elimina el prefijo `127.0.0.1:` y configura el firewall en consecuencia.
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"

      # Optional: only if you run iOS/Android nodes against this VPS and need Canvas host.
      # If you expose this publicly, read /gateway/security and firewall accordingly.
      # - "18793:18793"
    command:
      [
        "node",
        "dist/index.js",
        "gateway",
        "--bind",
        "${OPENCLAW_GATEWAY_BIND}",
        "--port",
        "${OPENCLAW_GATEWAY_PORT}"
      ]
```

***

<div id="7-bake-required-binaries-into-the-image-critical">
  ## 7) Incluir los binarios necesarios en la imagen (crítico)
</div>

Instalar binarios dentro de un contenedor en ejecución es un error.
Todo lo que se instale en tiempo de ejecución se perderá al reiniciar.

Todos los binarios externos que requieran las habilidades deben instalarse en tiempo de construcción de la imagen.

Los ejemplos siguientes muestran solo tres binarios habituales:

* `gog` para acceso a Gmail
* `goplaces` para Google Places
* `wacli` para WhatsApp

Estos son ejemplos, no una lista completa.
Puedes instalar tantos binarios como necesites usando el mismo patrón.

Si más adelante agregas nuevas habilidades que dependan de binarios adicionales, debes:

1. Actualizar el Dockerfile
2. Reconstruir la imagen
3. Reiniciar los contenedores

**Dockerfile de ejemplo**

```dockerfile
FROM node:22-bookworm

RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# Example binary 1: Gmail CLI
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# Example binary 2: Google Places CLI
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# Example binary 3: WhatsApp CLI
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

# Añada más binarios a continuación usando el mismo patrón

WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN corepack enable
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

***

<div id="8-build-and-launch">
  ## 8) Compilar e iniciar
</div>

```bash
docker compose build
docker compose up -d openclaw-gateway
```

Verificar binarios:

```bash
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli
```

Salida esperada:

```
/usr/local/bin/gog
/usr/local/bin/goplaces
/usr/local/bin/wacli
```

***

<div id="9-verify-gateway">
  ## 9) Verificar el Gateway
</div>

```bash
docker compose logs -f openclaw-gateway
```

Correcto:

```
[gateway] listening on ws://0.0.0.0:18789
```

Desde tu portátil:

```bash
ssh -N -L 18789:127.0.0.1:18789 root@YOUR_VPS_IP
```

En tu navegador, abre:

`http://127.0.0.1:18789/`

Pega tu token de Gateway.

***

<div id="what-persists-where-source-of-truth">
  ## Qué persiste dónde (fuente de la verdad)
</div>

OpenClaw se ejecuta en Docker, pero Docker no es la fuente de la verdad.
Todo estado persistente debe sobrevivir a reinicios, reconstrucciones y reinicios del sistema.

| Componente | Ubicación | Mecanismo de persistencia | Notas |
|---|---|---|---|
| Configuración del Gateway | `/home/node/.openclaw/` | Montaje de volumen del host | Incluye `openclaw.json`, tokens |
| Perfiles de autenticación de modelos | `/home/node/.openclaw/` | Montaje de volumen del host | Tokens OAuth, claves API |
| Configuraciones de habilidades | `/home/node/.openclaw/skills/` | Montaje de volumen del host | Estado a nivel de habilidad |
| Espacio de trabajo del agente | `/home/node/.openclaw/workspace/` | Montaje de volumen del host | Código y artefactos del agente |
| Sesión de WhatsApp | `/home/node/.openclaw/` | Montaje de volumen del host | Conserva el inicio de sesión por QR |
| Llavero de Gmail | `/home/node/.openclaw/` | Volumen del host + contraseña | Requiere `GOG_KEYRING_PASSWORD` |
| Binarios externos | `/usr/local/bin/` | Imagen de Docker | Deben incluirse en tiempo de construcción |
| Entorno de ejecución de Node | Sistema de archivos del contenedor | Imagen de Docker | Se reconstruye en cada construcción de la imagen |
| Paquetes del SO | Sistema de archivos del contenedor | Imagen de Docker | No instalar en tiempo de ejecución |
| Contenedor Docker | Efímero | Reiniciable | Se puede destruir sin riesgo |