---
title: Fly.io
description: Implementar OpenClaw en Fly.io
---

<div id="flyio-deployment">
  # Despliegue en Fly.io
</div>

**Objetivo:** Gateway de OpenClaw en ejecución en una máquina de [Fly.io](https://fly.io) con almacenamiento persistente, HTTPS automático y acceso a Discord y otros canales.

<div id="what-you-need">
  ## Lo que necesitas
</div>

- [CLI de flyctl](https://fly.io/docs/hands-on/install-flyctl/) instalada
- Cuenta de Fly.io (el nivel gratuito sirve)
- Autenticación de modelos: clave de API de Anthropic (u otras claves de proveedor)
- Credenciales de canal: token de bot de Discord, token de Telegram, etc.

<div id="beginner-quick-path">
  ## Ruta rápida para principiantes
</div>

1. Clona el repositorio → personaliza `fly.toml`
2. Crea la aplicación y el volumen → configura los secretos
3. Despliega con `fly deploy`
4. Conéctate por SSH para crear la configuración o usa Control UI

<div id="1-create-the-fly-app">
  ## 1) Crear la aplicación en Fly.io
</div>

```bash
# Clonar el repositorio
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# Crear una nueva aplicación de Fly (elige tu propio nombre)
fly apps create my-openclaw

# Crear un volumen persistente (1GB suele ser suficiente)
fly volumes create openclaw_data --size 1 --region iad
```

**Sugerencia:** Elige una región cercana a ti. Opciones habituales: `lhr` (Londres), `iad` (Virginia), `sjc` (San José).


<div id="2-configure-flytoml">
  ## 2) Configurar fly.toml
</div>

Edita `fly.toml` para que se adapte al nombre de tu aplicación y a tus requisitos.

**Nota de seguridad:** La configuración predeterminada expone una URL pública. Para una implementación reforzada sin IP pública, consulta [Despliegue privado](#private-deployment-hardened) o usa `fly.private.toml`.

```toml
app = "my-openclaw"  # Nombre de tu aplicación
primary_region = "iad"

[build]
  dockerfile = "Dockerfile"

[env]
  NODE_ENV = "production"
  OPENCLAW_PREFER_PNPM = "1"
  OPENCLAW_STATE_DIR = "/data"
  NODE_OPTIONS = "--max-old-space-size=1536"

[processes]
  app = "node dist/index.js gateway --allow-unconfigured --port 3000 --bind lan"

[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = false
  auto_start_machines = true
  min_machines_running = 1
  processes = ["app"]

[[vm]]
  size = "shared-cpu-2x"
  memory = "2048mb"

[mounts]
  source = "openclaw_data"
  destination = "/data"
```

**Ajustes clave:**

| Setting                        | Why                                                                                                   |
| ------------------------------ | ----------------------------------------------------------------------------------------------------- |
| `--bind lan`                   | Se vincula a `0.0.0.0` para que el proxy de Fly pueda llegar al Gateway                               |
| `--allow-unconfigured`         | Se inicia sin un archivo de configuración (crearás uno después)                                       |
| `internal_port = 3000`         | Debe coincidir con `--port 3000` (o `OPENCLAW_GATEWAY_PORT`) para las comprobaciones de estado de Fly |
| `memory = "2048mb"`            | 512MB es demasiado poco; se recomiendan 2GB                                                           |
| `OPENCLAW_STATE_DIR = "/data"` | Conserva el estado en el volumen                                                                      |


<div id="3-set-secrets">
  ## 3) Configura los secretos
</div>

```bash
# Requerido: Token del Gateway (para enlace no-loopback)
fly secrets set OPENCLAW_GATEWAY_TOKEN=$(openssl rand -hex 32)

# Model provider API keys
fly secrets set ANTHROPIC_API_KEY=sk-ant-...

# Optional: Other providers
fly secrets set OPENAI_API_KEY=sk-...
fly secrets set GOOGLE_API_KEY=...

# Channel tokens
fly secrets set DISCORD_BOT_TOKEN=MTQ...
```

**Notas:**

* Los binds non-loopback (`--bind lan`) requieren `OPENCLAW_GATEWAY_TOKEN` por motivos de seguridad.
* Trata estos tokens como contraseñas.
* **Da preferencia a las variables de entorno frente al archivo de configuración** para todas las claves de API y tokens. Esto mantiene los secretos fuera de `openclaw.json`, donde podrían quedar expuestos o registrarse en los logs accidentalmente.


<div id="4-deploy">
  ## 4) Desplegar
</div>

```bash
fly deploy
```

El primer despliegue construye la imagen de Docker (unos 2-3 minutos). Los despliegues posteriores son más rápidos.

Después del despliegue, comprueba:

```bash
fly status
fly logs
```

Deberías ver lo siguiente:

```
[gateway] listening on ws://0.0.0.0:3000 (PID xxx)
[discord] logged in to discord as xxx
```


<div id="5-create-config-file">
  ## 5) Crear archivo de configuración
</div>

Conéctate por SSH a la máquina para crear una configuración adecuada:

```bash
fly ssh console
```

Crea el directorio y el archivo de configuración:

```bash
mkdir -p /data
cat > /data/openclaw.json << 'EOF'
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-5",
        "fallbacks": ["anthropic/claude-sonnet-4-5", "openai/gpt-4o"]
      },
      "maxConcurrent": 4
    },
    "list": [
      {
        "id": "main",
        "default": true
      }
    ]
  },
  "auth": {
    "profiles": {
      "anthropic:default": { "mode": "token", "provider": "anthropic" },
      "openai:default": { "mode": "token", "provider": "openai" }
    }
  },
  "bindings": [
    {
      "agentId": "main",
      "match": { "channel": "discord" }
    }
  ],
  "channels": {
    "discord": {
      "enabled": true,
      "groupPolicy": "allowlist",
      "guilds": {
        "YOUR_GUILD_ID": {
          "channels": { "general": { "allow": true } },
          "requireMention": false
        }
      }
    }
  },
  "gateway": {
    "mode": "local",
    "bind": "auto"
  },
  "meta": {
    "lastTouchedVersion": "2026.1.29"
  }
}
EOF
```

**Nota:** Con `OPENCLAW_STATE_DIR=/data`, la ruta de configuración es `/data/openclaw.json`.

**Nota:** El token de Discord puede provenir de:

* Variable de entorno: `DISCORD_BOT_TOKEN` (recomendado para secretos)
* Archivo de configuración: `channels.discord.token`

Si usas la variable de entorno, no es necesario agregar el token a la configuración. El Gateway lee `DISCORD_BOT_TOKEN` automáticamente.

Reinicia para aplicar los cambios:

```bash
exit
fly machine restart <machine-id>
```


<div id="6-access-the-gateway">
  ## 6) Acceder al Gateway
</div>

<div id="control-ui">
  ### Control UI
</div>

Abre en el navegador:

```bash
fly open
```

O visita `https://my-openclaw.fly.dev/`

Pega tu token de Gateway (el de `OPENCLAW_GATEWAY_TOKEN`) para autenticarte.


<div id="logs">
  ### Registros
</div>

```bash
fly logs              # Logs en tiempo real
fly logs --no-tail    # Recent logs
```


<div id="ssh-console">
  ### Consola SSH
</div>

```bash
fly ssh console
```


<div id="troubleshooting">
  ## Solución de problemas
</div>

<div id="app-is-not-listening-on-expected-address">
  ### "La aplicación no está escuchando en la dirección esperada"
</div>

El Gateway se está vinculando a `127.0.0.1` en lugar de a `0.0.0.0`.

**Solución:** Agrega `--bind lan` al comando del proceso en `fly.toml`.

<div id="health-checks-failing-connection-refused">
  ### Comprobaciones de estado fallidas / conexión rechazada
</div>

Fly no puede conectarse al Gateway en el puerto configurado.

**Solución:** Asegúrate de que `internal_port` coincida con el puerto del Gateway (establece `--port 3000` o `OPENCLAW_GATEWAY_PORT=3000`).

<div id="oom-memory-issues">
  ### Problemas de OOM / memoria
</div>

El contenedor se reinicia continuamente o se termina. Síntomas: `SIGABRT`, `v8::internal::Runtime_AllocateInYoungGeneration` o reinicios silenciosos.

**Solución:** Aumenta la memoria en `fly.toml`:

```toml
[[vm]]
  memory = "2048mb"
```

O bien actualiza una máquina existente:

```bash
fly machine update <machine-id> --vm-memory 2048 -y
```

**Nota:** 512MB es demasiado poco. 1GB puede funcionar, pero podría quedarse sin memoria (OOM) bajo carga o con registros muy detallados. **Se recomiendan 2GB.**


<div id="gateway-lock-issues">
  ### Problemas con el bloqueo de Gateway
</div>

Gateway no se inicia y muestra errores de «already running».

Esto ocurre cuando el contenedor se reinicia pero el archivo de bloqueo de PID permanece en el volumen.

**Solución:** Elimina el archivo de bloqueo:

```bash
fly ssh console --command "rm -f /data/gateway.*.lock"
fly machine restart <machine-id>
```

El archivo de bloqueo se encuentra en `/data/gateway.*.lock` (no en un subdirectorio).


<div id="config-not-being-read">
  ### La configuración no se carga
</div>

Si usas `--allow-unconfigured`, el Gateway crea una configuración mínima. Tu configuración personalizada en `/data/openclaw.json` debería cargarse al reiniciar.

Verifica que la configuración exista:

```bash
fly ssh console --command "cat /data/openclaw.json"
```


<div id="writing-config-via-ssh">
  ### Escribir la configuración mediante SSH
</div>

El comando `fly ssh console -C` no admite la redirección del shell. Para escribir un archivo de configuración:

```bash
# Use echo + tee (pipe from local to remote)
echo '{"your":"config"}' | fly ssh console -C "tee /data/openclaw.json"

# Or use sftp
fly sftp shell
> put /local/path/config.json /data/openclaw.json
```

**Nota:** `fly sftp` puede fallar si el archivo ya existe. Elimina primero el archivo:

```bash
fly ssh console --command "rm /data/openclaw.json"
```


<div id="state-not-persisting">
  ### El estado no persiste
</div>

Si pierdes credenciales o sesiones después de un reinicio, el directorio de estado se está guardando en el sistema de archivos del contenedor.

**Solución:** Asegúrate de que `OPENCLAW_STATE_DIR=/data` esté configurado en `fly.toml` y vuelve a desplegar.

<div id="updates">
  ## Actualizaciones
</div>

```bash
# Pull latest changes
git pull

# Redeploy
fly deploy

# Check health
fly status
fly logs
```


<div id="updating-machine-command">
  ### Actualización del comando de la máquina
</div>

Si necesitas cambiar el comando de arranque sin hacer un despliegue completo:

```bash
# Obtener ID de máquina
fly machines list

# Actualizar comando
fly machine update <machine-id> --command "node dist/index.js gateway --port 3000 --bind lan" -y

# O con aumento de memoria
fly machine update <machine-id> --vm-memory 2048 --command "node dist/index.js gateway --port 3000 --bind lan" -y
```

**Nota:** Después de `fly deploy`, el comando de la máquina puede restablecerse a lo definido en `fly.toml`. Si realizaste cambios manuales, vuelve a aplicarlos después del despliegue.


<div id="private-deployment-hardened">
  ## Despliegue privado (reforzado)
</div>

De forma predeterminada, Fly asigna direcciones IP públicas, lo que hace que tu Gateway sea accesible en `https://your-app.fly.dev`. Esto es conveniente, pero significa que tu despliegue es detectable por escáneres de Internet (Shodan, Censys, etc.).

Para un despliegue reforzado sin **exposición pública**, usa la plantilla privada.

<div id="when-to-use-private-deployment">
  ### Cuándo usar un despliegue privado
</div>

- Solo realizas llamadas o mensajes **salientes** (sin webhooks entrantes)
- Usas túneles de **ngrok o Tailscale** para cualquier callback de webhook
- Accedes al Gateway mediante **SSH, un proxy o WireGuard** en lugar de un navegador
- Quieres que el despliegue esté **oculto a los escáneres de Internet**

<div id="setup">
  ### Configuración
</div>

Utiliza `fly.private.toml` en lugar de la configuración estándar:

```bash
# Desplegar con configuración privada
fly deploy -c fly.private.toml
```

O convierte un despliegue ya existente:

```bash
# List current IPs
fly ips list -a my-openclaw

# Release public IPs
fly ips release <public-ipv4> -a my-openclaw
fly ips release <public-ipv6> -a my-openclaw

# Cambiar a configuración privada para que despliegues futuros no reasignen IPs públicas
# (eliminar [http_service] o desplegar con la plantilla privada)
fly deploy -c fly.private.toml

# Allocate private-only IPv6
fly ips allocate-v6 --private -a my-openclaw
```

Después de esto, `fly ips list` debería mostrar solo una IP de tipo `private`:

```
VERSION  IP                   TYPE             REGION
v6       fdaa:x:x:x:x::x      private          global
```


<div id="accessing-a-private-deployment">
  ### Acceder a un despliegue privado
</div>

Como no hay ninguna URL pública, utiliza uno de estos métodos:

**Opción 1: proxy local (la opción más sencilla)**

```bash
# Reenviar el puerto local 3000 a la aplicación
fly proxy 3000:3000 -a my-openclaw

# Luego abrir http://localhost:3000 en el navegador
```

**Opción 2: VPN de WireGuard**

```bash
# Crear configuración de WireGuard (una sola vez)
fly wireguard create

# Importar al cliente de WireGuard, luego acceder mediante IPv6 interna
# Ejemplo: http://[fdaa:x:x:x:x::x]:3000
```

**Opción 3: solo mediante SSH**

```bash
fly ssh console -a my-openclaw
```


<div id="webhooks-with-private-deployment">
  ### Webhooks con despliegue privado
</div>

Si necesitas callbacks de webhooks (Twilio, Telnyx, etc.) sin exposición pública:

1. **túnel ngrok** - Ejecuta ngrok dentro del contenedor o como sidecar
2. **Tailscale Funnel** - Expón rutas específicas a través de Tailscale
3. **Solo saliente** - Algunos proveedores (Twilio) funcionan bien para llamadas salientes sin webhooks

Ejemplo de configuración de llamada de voz con ngrok:

```json
{
  "plugins": {
    "entries": {
      "voice-call": {
        "enabled": true,
        "config": {
          "provider": "twilio",
          "tunnel": { "provider": "ngrok" }
        }
      }
    }
  }
}
```

El túnel de ngrok se ejecuta dentro del contenedor y proporciona una URL pública de webhook sin exponer directamente la aplicación de Fly.


<div id="security-benefits">
  ### Beneficios de seguridad
</div>

| Aspecto | Público | Privado |
|--------|--------|---------|
| Escáneres de internet | Detectable | Oculto |
| Ataques directos | Posibles | Bloqueados |
| Acceso a Control UI | Navegador | Proxy/VPN |
| Entrega de webhooks | Directa | A través de un túnel |

<div id="notes">
  ## Notas
</div>

- Fly.io usa **arquitectura x86** (no ARM)
- El Dockerfile es compatible con ambas arquitecturas
- Para el onboarding de WhatsApp/Telegram, usa `fly ssh console`
- Los datos persistentes se almacenan en el volumen en `/data`
- Signal requiere Java + signal-cli; usa una imagen personalizada y mantén la memoria en al menos 2 GB.

<div id="cost">
  ## Coste
</div>

Con la configuración recomendada (`shared-cpu-2x`, 2GB RAM):

- ~10-15 USD/mes según el uso
- El plan gratuito incluye un cupo de uso

Consulta los [precios de Fly.io](https://fly.io/docs/about/pricing/) para más detalles.