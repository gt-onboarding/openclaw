---
title: Digitalocean
summary: "OpenClaw en DigitalOcean (opción simple de VPS de pago)"
read_when:
  - Configurar OpenClaw en DigitalOcean
  - Buscar alojamiento VPS económico para OpenClaw
---

<div id="openclaw-on-digitalocean">
  # OpenClaw en DigitalOcean
</div>

<div id="goal">
  ## Objetivo
</div>

Ejecutar un Gateway persistente de OpenClaw en DigitalOcean por **6 USD al mes** (o 4 USD al mes con precios de reserva).

Si quieres una opción de 0 USD/mes y no te importa usar ARM y una configuración específica del proveedor, consulta la [guía de Oracle Cloud](/es/platforms/oracle).

<div id="cost-comparison-2026">
  ## Comparación de costos (2026)
</div>

| Proveedor | Plan | Especificaciones | Precio/mes | Notas |
|----------|------|------------------|-----------|-------|
| Oracle Cloud | Always Free ARM | hasta 4 OCPU, 24GB RAM | $0 | ARM, capacidad limitada / peculiaridades al registrarse |
| Hetzner | CX22 | 2 vCPU, 4GB RAM | €3.79 (~$4) | Opción de pago más económica |
| DigitalOcean | Basic | 1 vCPU, 1GB RAM | $6 | UI fácil de usar, buena documentación |
| Vultr | Cloud Compute | 1 vCPU, 1GB RAM | $6 | Muchas ubicaciones |
| Linode | Nanode | 1 vCPU, 1GB RAM | $5 | Ahora parte de Akamai |

**Cómo elegir un proveedor:**

* DigitalOcean: UX más simple + configuración predecible (esta guía)
* Hetzner: buena relación precio/rendimiento (consulta la [guía de Hetzner](/es/platforms/hetzner))
* Oracle Cloud: puede salir por $0/mes, pero es más problemático y solo ARM (consulta la [guía de Oracle](/es/platforms/oracle))

***

<div id="prerequisites">
  ## Requisitos previos
</div>

* Cuenta en DigitalOcean ([regístrate y obtén 200 USD de crédito gratuito](https://m.do.co/c/signup))
* Par de claves SSH (o estar dispuesto a usar autenticación por contraseña)
* ~20 minutos

<div id="1-create-a-droplet">
  ## 1) Crear un Droplet
</div>

1. Inicia sesión en [DigitalOcean](https://cloud.digitalocean.com/)
2. Haz clic en **Create → Droplets**
3. Selecciona:
   * **Region:** La más cercana a ti (o a tus usuarios)
   * **Image:** Ubuntu 24.04 LTS
   * **Size:** Basic → Regular → **$6/mo** (1 vCPU, 1GB RAM, 25GB SSD)
   * **Authentication:** Clave SSH (recomendada) o contraseña
4. Haz clic en **Create Droplet**
5. Anota la dirección IP

<div id="2-connect-via-ssh">
  ## 2) Conéctate mediante SSH
</div>

```bash
ssh root@YOUR_DROPLET_IP
```

<div id="3-install-openclaw">
  ## 3) Instalar OpenClaw
</div>

```bash
# Update system
apt update && apt upgrade -y

# Instalar Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs

# Install OpenClaw
curl -fsSL https://openclaw.bot/install.sh | bash

# Verify
openclaw --version
```

<div id="4-run-onboarding">
  ## 4) Ejecutar el asistente de incorporación
</div>

```bash
openclaw onboard --install-daemon
```

El asistente te guiará a través de:

* Autenticación de modelos (claves API o OAuth)
* Configuración de canales (Telegram, WhatsApp, Discord, etc.)
* Token del Gateway (generado automáticamente)
* Instalación del servicio (systemd)

<div id="5-verify-the-gateway">
  ## 5) Verifica el Gateway
</div>

```bash
# Verificar el estado
openclaw status

# Verificar el servicio
systemctl --user status openclaw-gateway.service

# Ver los registros
journalctl --user -u openclaw-gateway.service -f
```

<div id="6-access-the-dashboard">
  ## 6) Accede al panel de control
</div>

El Gateway escucha en la interfaz de loopback de forma predeterminada. Para acceder al Control UI:

**Opción A: túnel SSH (recomendado)**

```bash
# Desde tu máquina local
ssh -L 18789:localhost:18789 root@YOUR_DROPLET_IP

# Luego abre: http://localhost:18789
```

**Opción B: Tailscale Serve (HTTPS, solo en loopback)**

```bash
# On the droplet
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up

# Configure Gateway to use Tailscale Serve
openclaw config set gateway.tailscale.mode serve
openclaw gateway restart
```

Abre: `https://<magicdns>/`

Notas:

* Serve mantiene el Gateway accesible solo por loopback y autentica mediante cabeceras de identidad de Tailscale.
* Para requerir token/contraseña en su lugar, establece `gateway.auth.allowTailscale: false` o usa `gateway.auth.mode: "password"`.

**Opción C: vinculación a Tailnet (sin Serve)**

```bash
openclaw config set gateway.bind tailnet
openclaw gateway restart
```

Abre: `http://<tailscale-ip>:18789` (requiere token).

<div id="7-connect-your-channels">
  ## 7) Conecta tus canales
</div>

<div id="telegram">
  ### Telegram
</div>

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

<div id="whatsapp">
  ### WhatsApp
</div>

```bash
openclaw channels login whatsapp
# Escanea el código QR
```

Consulta [Channels](/es/channels) para otros proveedores.

***

<div id="optimizations-for-1gb-ram">
  ## Optimizaciones para 1 GB de RAM
</div>

El droplet de $6 tiene solo 1 GB de RAM. Para mantener todo funcionando sin problemas:

<div id="add-swap-recommended">
  ### Añade swap (recomendado)
</div>

```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

<div id="use-a-lighter-model">
  ### Usa un modelo más ligero
</div>

Si te encuentras con errores OOM, considera:

* Usar modelos basados en API (Claude, GPT) en lugar de modelos locales
* Establecer `agents.defaults.model.primary` en un modelo más pequeño

<div id="monitor-memory">
  ### Supervisar la memoria
</div>

```bash
free -h
htop
```

***

<div id="persistence">
  ## Persistencia
</div>

Todo el estado se almacena en:

* `~/.openclaw/` — configuración, credenciales, datos de sesión
* `~/.openclaw/workspace/` — espacio de trabajo (SOUL.md, memoria, etc.)

Estos directorios persisten entre reinicios. Realiza copias de seguridad periódicamente:

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

***

<div id="oracle-cloud-free-alternative">
  ## Alternativa gratuita en Oracle Cloud
</div>

Oracle Cloud ofrece instancias ARM **Always Free** que son significativamente más potentes que cualquier opción de pago aquí — por 0 $/mes.

| Qué recibes | Especificaciones |
|-------------|------------------|
| **4 OCPUs** | ARM Ampere A1 |
| **24GB RAM** | Más que suficiente |
| **200GB storage** | Volumen de bloque |
| **Forever free** | Sin cargos en la tarjeta de crédito |

**Advertencias:**

* El registro puede ser problemático (vuelve a intentarlo si falla)
* Arquitectura ARM: la mayoría de las cosas funcionan, pero algunos binarios necesitan compilaciones específicas para ARM

Para la guía completa de configuración, consulta [Oracle Cloud](/es/platforms/oracle). Para consejos de registro y solución de problemas durante el proceso de alta, consulta esta [guía de la comunidad](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd).

***

<div id="troubleshooting">
  ## Solución de problemas
</div>

<div id="gateway-wont-start">
  ### Gateway no se inicia
</div>

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl -u openclaw --no-pager -n 50
```

<div id="port-already-in-use">
  ### Puerto en uso
</div>

```bash
lsof -i :18789
kill <PID>
```

<div id="out-of-memory">
  ### Falta de memoria
</div>

```bash
# Verificar memoria
free -h

# Agregar más swap
# O actualizar al droplet de $12/mes (2GB RAM)
```

***

<div id="see-also">
  ## Ver también
</div>

* [Guía de Hetzner](/es/platforms/hetzner) — más barato, más potente
* [Instalación con Docker](/es/install/docker) — despliegue en contenedores
* [Tailscale](/es/gateway/tailscale) — acceso remoto seguro
* [Configuración](/es/gateway/configuration) — referencia completa de la configuración