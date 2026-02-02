---
title: Exe Dev
summary: "Ejecutar OpenClaw Gateway en exe.dev (VM + proxy HTTPS) para acceso remoto"
read_when:
  - Quieres un host Linux barato y siempre encendido para el Gateway
  - Quieres acceso remoto a la Control UI sin administrar tu propio VPS
---

<div id="exedev">
  # exe.dev
</div>

Objetivo: Gateway de OpenClaw ejecutándose en una VM de exe.dev, accesible desde tu portátil vía: `https://<vm-name>.exe.xyz`

Esta página asume la imagen **exeuntu** predeterminada de exe.dev. Si elegiste una distribución diferente, ajusta los paquetes en consecuencia.

<div id="beginner-quick-path">
  ## Ruta rápida para principiantes
</div>

1. [https://exe.new/openclaw](https://exe.new/openclaw)
2. Introduce tu clave/token de autenticación cuando sea necesario
3. Haz clic en &quot;Agent&quot; junto a tu VM y espera...
4. ???
5. Beneficio

<div id="what-you-need">
  ## Lo que necesitas
</div>

* Una cuenta en exe.dev
* Acceso por `ssh exe.dev` a máquinas virtuales de [exe.dev](https://exe.dev) (opcional)

<div id="automated-install-with-shelley">
  ## Instalación automatizada con Shelley
</div>

Shelley, el agente de [exe.dev](https://exe.dev), puede instalar OpenClaw al instante con nuestro prompt. El prompt que se utiliza es el siguiente:

```
Configura OpenClaw (https://docs.openclaw.ai/install) en esta VM. Utiliza las banderas non-interactive y accept-risk para la incorporación de openclaw. Añade la autenticación o token proporcionado según sea necesario. Configura nginx para reenviar desde el puerto predeterminado 18789 a la ubicación raíz en la configuración del sitio habilitado por defecto, asegurándote de habilitar el soporte de Websocket. El emparejamiento se realiza mediante "openclaw devices list" y "openclaw device approve <request id>". Asegúrate de que el panel de control muestre que el estado de OpenClaw es OK. exe.dev maneja el reenvío desde el puerto 8000 al puerto 80/443 y HTTPS por nosotros, por lo que el "reachable" final debe ser <vm-name>.exe.xyz, sin especificación de puerto.
```

<div id="manual-installation">
  ## Instalación manual
</div>

<div id="1-create-the-vm">
  ## 1) Crea la VM
</div>

Desde tu dispositivo:

```bash
ssh exe.dev new 
```

Luego conéctate:

```bash
ssh <vm-name>.exe.xyz
```

Sugerencia: mantén esta VM **stateful** (con estado persistente). OpenClaw almacena su estado en `~/.openclaw/` y `~/.openclaw/workspace/`.

<div id="2-install-prerequisites-on-the-vm">
  ## 2) Instalar requisitos previos (en la VM)
</div>

```bash
sudo apt-get update
sudo apt-get install -y git curl jq ca-certificates openssl
```

<div id="3-install-openclaw">
  ## 3) Instalar OpenClaw
</div>

Ejecuta el script de instalación de OpenClaw:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

<div id="4-setup-nginx-to-proxy-openclaw-to-port-8000">
  ## 4) Configurar nginx para usarlo como proxy inverso de OpenClaw en el puerto 8000
</div>

Edita `/etc/nginx/sites-enabled/default` con

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    listen 8000;
    listen [::]:8000;

    server_name _;

    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_http_version 1.1;

        # WebSocket support
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Standard proxy headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Configuración de timeout para conexiones de larga duración
        proxy_read_timeout 86400s;
        proxy_send_timeout 86400s;
    }
}
```

<div id="5-access-openclaw-and-grant-privileges">
  ## 5) Accede a OpenClaw y concede privilegios
</div>

Accede a `https://<vm-name>.exe.xyz/?token=YOUR-TOKEN-FROM-TERMINAL`. Aprueba
dispositivos con `openclaw devices list` y `openclaw device approve`. Si tienes dudas,
usa Shelley desde tu navegador.

<div id="remote-access">
  ## Acceso remoto
</div>

El acceso remoto se gestiona mediante el sistema de autenticación de [exe.dev](https://exe.dev). De forma predeterminada, el tráfico HTTP del puerto 8000 se reenvía a `https://<vm-name>.exe.xyz` con autenticación por correo electrónico.

<div id="updating">
  ## Actualización
</div>

```bash
npm i -g openclaw@latest
openclaw doctor
openclaw gateway restart
openclaw health
```

Guía: [Actualización](/es/install/updating)
