---
title: Gcp
summary: "Ejecuta OpenClaw Gateway 24/7 en una VM de GCP Compute Engine (Docker) con estado persistente"
read_when:
  - Quieres que OpenClaw se ejecute 24/7 en GCP
  - Quieres un Gateway de nivel de producción, siempre activo, en tu propia VM
  - Quieres control completo sobre la persistencia, los binarios y el comportamiento de reinicio
---

<div id="openclaw-on-gcp-compute-engine-docker-production-vps-guide">
  # OpenClaw en GCP Compute Engine (Docker, guía de VPS en producción)
</div>

<div id="goal">
  ## Objetivo
</div>

Ejecutar un Gateway de OpenClaw persistente en una VM de GCP Compute Engine usando Docker, con estado duradero, binarios integrados en la imagen y un comportamiento de reinicio seguro.

Si quieres &quot;OpenClaw 24/7 por ~5-12 USD/mes&quot;, esta es una configuración confiable en Google Cloud.
El precio varía según el tipo de máquina y la región; elige la VM más pequeña que se adapte a tu carga de trabajo y escala si empiezas a tener errores de falta de memoria (OOM).

<div id="what-are-we-doing-simple-terms">
  ## ¿Qué vamos a hacer (en términos simples)?
</div>

* Crear un proyecto en GCP y habilitar la facturación
* Crear una VM de Compute Engine
* Instalar Docker (entorno de ejecución de aplicaciones aislado)
* Iniciar el Gateway de OpenClaw en Docker
* Persistir `~/.openclaw` + `~/.openclaw/workspace` en el host (sobrevive a reinicios/reconstrucciones)
* Acceder al Control UI desde tu portátil mediante un túnel SSH

Puedes acceder al Gateway a través de:

* Reenvío de puertos SSH desde tu portátil
* Exposición directa de puertos si gestionas tú mismo el firewall y los tokens

Esta guía usa Debian en GCP Compute Engine.
Ubuntu también funciona; ajusta los paquetes en consecuencia.
Para el flujo genérico con Docker, consulta [Docker](/es/install/docker).

***

<div id="quick-path-experienced-operators">
  ## Ruta rápida (operadores experimentados)
</div>

1. Crear un proyecto de GCP y habilitar la API de Compute Engine
2. Crear una VM de Compute Engine (e2-small, Debian 12, 20GB)
3. Conectarse por SSH a la VM
4. Instalar Docker
5. Clonar el repositorio de OpenClaw
6. Crear directorios persistentes en el host
7. Configurar `.env` y `docker-compose.yml`
8. Generar los binarios necesarios, compilar e iniciar

***

<div id="what-you-need">
  ## Lo que necesitas
</div>

* Cuenta de GCP (nivel gratuito con opción a e2-micro)
* CLI de gcloud instalada (o usar Cloud Console)
* Acceso SSH desde tu portátil
* Familiaridad básica con SSH y copiar/pegar
* ~20-30 minutos
* Docker y Docker Compose
* Credenciales de autenticación de modelos
* Credenciales de proveedor opcionales
  * QR de WhatsApp
  * Token de bot de Telegram
  * OAuth de Gmail

***

<div id="1-install-gcloud-cli-or-use-console">
  ## 1) Instalar la CLI de gcloud (o usar la Consola)
</div>

**Opción A: CLI de gcloud** (recomendado para automatización)

Instálala desde https://cloud.google.com/sdk/docs/install

Inicialízala y autentícate:

```bash
gcloud init
gcloud auth login
```

**Opción B: Cloud Console**

Todos los pasos pueden realizarse desde la UI web en https://console.cloud.google.com

***

<div id="2-create-a-gcp-project">
  ## 2) Crea un proyecto de GCP
</div>

**CLI:**

```bash
gcloud projects create my-openclaw-project --name="OpenClaw Gateway"
gcloud config set project my-openclaw-project
```

Habilita la facturación en https://console.cloud.google.com/billing (obligatorio para Compute Engine).

Habilita la API de Compute Engine:

```bash
gcloud services enable compute.googleapis.com
```

**Consola:**

1. Ve a IAM &amp; Admin &gt; Create Project
2. Asígnale un nombre y créalo
3. Habilita la facturación del proyecto
4. Ve a APIs &amp; Services &gt; Enable APIs &gt; busca «Compute Engine API» &gt; Enable

***

<div id="3-create-the-vm">
  ## 3) Crear la VM
</div>

**Tipos de máquina:**

| Tipo     | Especificaciones               | Costo                       | Notas                                 |
| -------- | ------------------------------ | --------------------------- | ------------------------------------- |
| e2-small | 2 vCPU, 2 GB RAM               | ~$12/mes                    | Recomendado                           |
| e2-micro | 2 vCPU (compartidas), 1 GB RAM | Apto para el nivel gratuito | Puede quedarse sin memoria bajo carga |

**CLI:**

```bash
gcloud compute instances create openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small \
  --boot-disk-size=20GB \
  --image-family=debian-12 \
  --image-project=debian-cloud
```

**Consola:**

1. Ve a Compute Engine &gt; Instancias de VM &gt; Crear instancia
2. Nombre: `openclaw-gateway`
3. Región: `us-central1`, Zona: `us-central1-a`
4. Tipo de máquina: `e2-small`
5. Disco de arranque: Debian 12, 20 GB
6. Crear

***

<div id="4-ssh-into-the-vm">
  ## 4) Conéctate a la VM por SSH
</div>

**CLI:**

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

**Consola:**

Haz clic en el botón &quot;SSH&quot; junto a tu VM en el panel de Compute Engine.

Nota: la propagación de la clave SSH puede tardar entre 1 y 2 minutos después de la creación de la VM. Si la conexión es rechazada, espera y vuelve a intentarlo.

***

<div id="5-install-docker-on-the-vm">
  ## 5) Instalar Docker (en la máquina virtual)
</div>

```bash
sudo apt-get update
sudo apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
```

Cierra sesión y vuelve a iniciarla para que el cambio de grupo se aplique:

```bash
exit
```

Luego inicia sesión de nuevo por SSH:

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

Comprueba:

```bash
docker --version
docker compose version
```

***

<div id="6-clone-the-openclaw-repository">
  ## 6) Clona el repositorio de OpenClaw
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

Esta guía asume que construirás una imagen personalizada para garantizar la persistencia de los binarios.

***

<div id="7-create-persistent-host-directories">
  ## 7) Crear directorios persistentes en el host
</div>

Los contenedores de Docker son efímeros.
Todo el estado persistente debe almacenarse en el host.

```bash
mkdir -p ~/.openclaw
mkdir -p ~/.openclaw/workspace
```

***

<div id="8-configure-environment-variables">
  ## 8) Configurar variables de entorno
</div>

Crea el archivo `.env` en la raíz del repositorio.

```bash
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/home/$USER/.openclaw
OPENCLAW_WORKSPACE_DIR=/home/$USER/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

Genera secretos seguros:

```bash
openssl rand -hex 32
```

**No hagas commit de este archivo.**

***

<div id="9-docker-compose-configuration">
  ## 9) Configuración de Docker Compose
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
      # Recomendado: mantén el Gateway solo en loopback en la VM; accede mediante túnel SSH.
      # Para exponerlo públicamente, elimina el prefijo `127.0.0.1:` y configura el firewall en consecuencia.
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"

      # Optional: only if you run iOS/Android nodes against this VM and need Canvas host.
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

<div id="10-bake-required-binaries-into-the-image-critical">
  ## 10) Incluir los binarios necesarios en la imagen (crítico)
</div>

Instalar binarios dentro de un contenedor en ejecución es una mala práctica.
Cualquier cosa instalada en tiempo de ejecución se perderá al reiniciar.

Todos los binarios externos que requieran las habilidades deben instalarse al construir la imagen.

Los siguientes ejemplos muestran solo tres binarios habituales:

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

# Binario de ejemplo 1: Gmail CLI
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# Binario de ejemplo 2: Google Places CLI
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# Binario de ejemplo 3: WhatsApp CLI
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

# Añade más binarios a continuación usando el mismo patrón

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

<div id="11-build-and-launch">
  ## 11) Compilar y lanzar
</div>

```bash
docker compose build
docker compose up -d openclaw-gateway
```

Verificar los binarios:

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

<div id="12-verify-gateway">
  ## 12) Verificar el Gateway
</div>

```bash
docker compose logs -f openclaw-gateway
```

Éxito:

```
[gateway] listening on ws://0.0.0.0:18789
```

***

<div id="13-access-from-your-laptop">
  ## 13) Acceso desde tu portátil
</div>

Crea un túnel SSH para redirigir el puerto del Gateway:

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a -- -L 18789:127.0.0.1:18789
```

Ábrelo en tu navegador:

`http://127.0.0.1:18789/`

Pega tu token de Gateway.

***

<div id="what-persists-where-source-of-truth">
  ## Qué persiste dónde (fuente de verdad)
</div>

OpenClaw se ejecuta en Docker, pero Docker no es la fuente de verdad.
Todo estado de larga duración debe sobrevivir a reinicios, reconstrucciones y reinicios del sistema.

| Componente | Ubicación | Mecanismo de persistencia | Notas |
|---|---|---|---|
| Configuración del Gateway | `/home/node/.openclaw/` | Volumen del host montado | Incluye `openclaw.json`, tokens |
| Perfiles de autenticación de modelos | `/home/node/.openclaw/` | Volumen del host montado | Tokens OAuth, claves de API |
| Configuraciones de habilidades | `/home/node/.openclaw/skills/` | Volumen del host montado | Estado a nivel de habilidad |
| Espacio de trabajo del agente | `/home/node/.openclaw/workspace/` | Volumen del host montado | Código y artefactos del agente |
| Sesión de WhatsApp | `/home/node/.openclaw/` | Volumen del host montado | Conserva el inicio de sesión por QR |
| Llavero de Gmail | `/home/node/.openclaw/` | Volumen del host + contraseña | Requiere `GOG_KEYRING_PASSWORD` |
| Binarios externos | `/usr/local/bin/` | Imagen de Docker | Deben incluirse en tiempo de creación de la imagen |
| Entorno de ejecución de Node.js | Sistema de archivos del contenedor | Imagen de Docker | Se reconstruye en cada build de imagen |
| Paquetes del SO | Sistema de archivos del contenedor | Imagen de Docker | No instalar en tiempo de ejecución |
| Contenedor Docker | Efímero | Reiniciable | Se puede destruir sin riesgo |

***

<div id="updates">
  ## Actualizaciones
</div>

Para actualizar OpenClaw en la VM:

```bash
cd ~/openclaw
git pull
docker compose build
docker compose up -d
```

***

<div id="troubleshooting">
  ## Solución de problemas
</div>

**Conexión SSH rechazada**

La propagación de la clave SSH puede tardar 1 o 2 minutos después de crear la VM. Espera y vuelve a intentarlo.

**Problemas con OS Login**

Revisa tu perfil de OS Login:

```bash
gcloud compute os-login describe-profile
```

Asegúrate de que tu cuenta tenga los permisos necesarios de IAM (Compute OS Login o Compute OS Admin Login).

**Falta de memoria (OOM)**

Si estás usando e2-micro y se producen errores OOM, actualiza a e2-small o e2-medium:

```bash
# Detén la VM primero
gcloud compute instances stop openclaw-gateway --zone=us-central1-a

# Cambia el tipo de máquina
gcloud compute instances set-machine-type openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small

# Inicia la VM
gcloud compute instances start openclaw-gateway --zone=us-central1-a
```

***

<div id="service-accounts-security-best-practice">
  ## Cuentas de servicio (mejores prácticas de seguridad)
</div>

Para uso personal, tu cuenta de usuario predeterminada funciona bien.

Para automatización o pipelines de CI/CD, crea una cuenta de servicio dedicada con permisos mínimos:

1. Crea una cuenta de servicio:
   ```bash
   gcloud iam service-accounts create openclaw-deploy \
     --display-name="OpenClaw Deployment"
   ```

2. Otorga el rol Compute Instance Admin (o un rol personalizado más restringido):
   ```bash
   gcloud projects add-iam-policy-binding my-openclaw-project \
     --member="serviceAccount:openclaw-deploy@my-openclaw-project.iam.gserviceaccount.com" \
     --role="roles/compute.instanceAdmin.v1"
   ```

Evita usar el rol Owner para automatización. Aplica el principio de privilegios mínimos.

Consulta https://cloud.google.com/iam/docs/understanding-roles para obtener detalles sobre los roles de IAM.

***

<div id="next-steps">
  ## Próximos pasos
</div>

* Configura los canales de mensajería: [Canales](/es/channels)
* Empareja dispositivos locales como nodos: [Nodos](/es/nodes)
* Configura el Gateway: [Configuración del Gateway](/es/gateway/configuration)