---
title: "README del Gateway remoto"
summary: "Configuración de un túnel SSH para que OpenClaw.app se conecte a un Gateway remoto"
read_when: "Conexión de la app de macOS a un Gateway remoto mediante SSH"
---

<div id="running-openclawapp-with-a-remote-gateway">
  # Ejecutar OpenClaw.app con un Gateway remoto
</div>

OpenClaw.app usa túneles SSH para conectarse a un Gateway remoto. Esta guía explica cómo configurarlo.

<div id="overview">
  ## Descripción general
</div>

```
┌─────────────────────────────────────────────────────────────┐
│                        Client Machine                          │
│                                                              │
│  OpenClaw.app ──► ws://127.0.0.1:18789 (local port)           │
│                     │                                        │
│                     ▼                                        │
│  SSH Tunnel ────────────────────────────────────────────────│
│                     │                                        │
└─────────────────────┼──────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                         Remote Machine                        │
│                                                              │
│  Gateway WebSocket ──► ws://127.0.0.1:18789 ──►              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```


<div id="quick-setup">
  ## Configuración rápida
</div>

<div id="step-1-add-ssh-config">
  ### Paso 1: Agregar la configuración de SSH
</div>

Edita `~/.ssh/config` y agrega:

```ssh
Host remote-gateway
    HostName <REMOTE_IP>          # p. ej., 172.27.187.184
    User <REMOTE_USER>            # e.g., jefferson
    LocalForward 18789 127.0.0.1:18789
    IdentityFile ~/.ssh/id_rsa
```

Sustituye `<REMOTE_IP>` y `<REMOTE_USER>` por tus propios valores.


<div id="step-2-copy-ssh-key">
  ### Paso 2: Copiar clave SSH
</div>

Copia tu clave pública en la máquina remota (solo tendrás que introducir la contraseña una vez):

```bash
ssh-copy-id -i ~/.ssh/id_rsa <REMOTE_USER>@<REMOTE_IP>
```


<div id="step-3-set-gateway-token">
  ### Paso 3: Configurar el token de Gateway
</div>

```bash
launchctl setenv OPENCLAW_GATEWAY_TOKEN "<your-token>"
```


<div id="step-4-start-ssh-tunnel">
  ### Paso 4: Inicia el túnel SSH
</div>

```bash
ssh -N remote-gateway &
```


<div id="step-5-restart-openclawapp">
  ### Paso 5: Reiniciar OpenClaw.app
</div>

```bash
# Cierra OpenClaw.app (⌘Q) y luego vuelve a abrirla:
open /path/to/OpenClaw.app
```

La app ahora se conectará al Gateway remoto a través del túnel SSH.

***


<div id="auto-start-tunnel-on-login">
  ## Iniciar automáticamente el túnel al iniciar sesión
</div>

Para que el túnel SSH se inicie automáticamente al iniciar sesión, crea un Launch Agent.

<div id="create-the-plist-file">
  ### Crea el archivo PLIST
</div>

Guarda el siguiente contenido como `~/Library/LaunchAgents/bot.molt.ssh-tunnel.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>bot.molt.ssh-tunnel</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/ssh</string>
        <string>-N</string>
        <string>remote-gateway</string>
    </array>
    <key>KeepAlive</key>
    <true/>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```


<div id="load-the-launch-agent">
  ### Cargar el agente de lanzamiento
</div>

```bash
launchctl bootstrap gui/$UID ~/Library/LaunchAgents/bot.molt.ssh-tunnel.plist
```

El túnel ahora:

* Se iniciará automáticamente cuando inicies sesión
* Se reiniciará si se cierra inesperadamente
* Seguirá ejecutándose en segundo plano

Nota sobre versiones anteriores: elimina cualquier LaunchAgent `com.openclaw.ssh-tunnel` que quede, si está presente.

***


<div id="troubleshooting">
  ## Solución de problemas
</div>

**Verifica si el túnel está en ejecución:**

```bash
ps aux | grep "ssh -N remote-gateway" | grep -v grep
lsof -i :18789
```

**Reinicia el túnel:**

```bash
launchctl kickstart -k gui/$UID/bot.molt.ssh-tunnel
```

**Detener el túnel:**

```bash
launchctl bootout gui/$UID/bot.molt.ssh-tunnel
```

***


<div id="how-it-works">
  ## Cómo funciona
</div>

| Componente | Qué hace |
|-----------|--------------|
| `LocalForward 18789 127.0.0.1:18789` | Reenvía el puerto local 18789 al puerto remoto 18789 |
| `ssh -N` | SSH sin ejecutar comandos remotos (solo reenvío de puertos) |
| `KeepAlive` | Reinicia automáticamente el túnel si falla |
| `RunAtLoad` | Inicia el túnel cuando se carga el agente |

OpenClaw.app se conecta a `ws://127.0.0.1:18789` en tu máquina cliente. El túnel SSH reenvía esa conexión al puerto 18789 en la máquina remota donde se está ejecutando el Gateway.