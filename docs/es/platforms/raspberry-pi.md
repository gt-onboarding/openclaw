---
title: Raspberry Pi
summary: "OpenClaw en Raspberry Pi (configuración de autoalojamiento económica)"
read_when:
  - Configurar OpenClaw en una Raspberry Pi
  - Ejecutar OpenClaw en dispositivos ARM
  - Crear una IA personal siempre activa y económica
---

<div id="openclaw-on-raspberry-pi">
  # OpenClaw en Raspberry Pi
</div>

<div id="goal">
  ## Objetivo
</div>

Ejecutar un Gateway OpenClaw persistente y siempre activo en una Raspberry Pi con un coste único aproximado de **~$35-80** (sin cuotas mensuales).

Perfecto para:

* Asistente de IA personal 24/7
* Hub de automatización del hogar
* Bot de Telegram/WhatsApp de bajo consumo y siempre disponible

<div id="hardware-requirements">
  ## Requisitos de hardware
</div>

| Modelo de Raspberry Pi | RAM | ¿Funciona? | Notas |
|----------|-----|--------|-------|
| **Pi 5** | 4GB/8GB | ✅ Mejor | El más rápido, recomendado |
| **Pi 4** | 4GB | ✅ Bueno | Punto óptimo para la mayoría de los usuarios |
| **Pi 4** | 2GB | ✅ Aceptable | Funciona, añade swap |
| **Pi 4** | 1GB | ⚠️ Ajustado | Posible con swap y configuración mínima |
| **Pi 3B+** | 1GB | ⚠️ Lento | Funciona, pero va lento |
| **Pi Zero 2 W** | 512MB | ❌ | No recomendado |

**Especificaciones mínimas:** 1GB RAM, 1 núcleo, 500MB de espacio en disco\
**Recomendado:** 2GB+ RAM, sistema operativo de 64 bits, tarjeta SD de 16GB+ (o SSD USB)

<div id="what-youll-need">
  ## Lo que necesitarás
</div>

* Raspberry Pi 4 o 5 (se recomiendan 2 GB o más)
* Tarjeta microSD (16 GB o más) o SSD USB (mejor rendimiento)
* Fuente de alimentación (se recomienda la oficial de Raspberry Pi)
* Conexión de red (Ethernet o WiFi)
* ~30 minutos

<div id="1-flash-the-os">
  ## 1) Grabar el sistema operativo
</div>

Usa **Raspberry Pi OS Lite (64-bit)**; no se necesita entorno de escritorio para un servidor sin interfaz gráfica (headless).

1. Descarga [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2. Elige el sistema operativo: **Raspberry Pi OS Lite (64-bit)**
3. Haz clic en el icono de engranaje (⚙️) para preconfigurar:
   * Establece el nombre de host (hostname): `gateway-host`
   * Habilita SSH
   * Configura nombre de usuario y contraseña
   * Configura el WiFi (si no usas Ethernet)
4. Graba la imagen en la tarjeta SD o la unidad USB
5. Inserta y enciende la Pi

<div id="2-connect-via-ssh">
  ## 2) Conéctate por SSH
</div>

```bash
ssh user@gateway-host
# o utiliza la dirección IP
ssh user@192.168.x.x
```

<div id="3-system-setup">
  ## 3) Configuración del sistema
</div>

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install essential packages
sudo apt install -y git curl build-essential

# Set timezone (important for cron/reminders)
sudo timedatectl set-timezone America/Chicago  # Cambia a tu zona horaria
```

<div id="4-install-nodejs-22-arm64">
  ## 4) Instalar Node.js 22 (ARM64)
</div>

```bash
# Install Node.js via NodeSource
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# Verify
node --version  # Debería mostrar v22.x.x
npm --version
```

<div id="5-add-swap-important-for-2gb-or-less">
  ## 5) Añadir swap (importante para equipos con 2 GB o menos)
</div>

El swap evita errores por falta de memoria:

```bash
# Create 2GB swap file
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Optimizar para RAM baja (reducir swappiness)
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

<div id="6-install-openclaw">
  ## 6) Instalar OpenClaw
</div>

<div id="option-a-standard-install-recommended">
  ### Opción A: Instalación estándar (recomendada)
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

<div id="option-b-hackable-install-for-tinkering">
  ### Opción B: Instalación hackeable (para cacharrear)
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
npm install
npm run build
npm link
```

La instalación hackeable te proporciona acceso directo a los registros y al código, lo que resulta útil para depurar problemas específicos de ARM.

<div id="7-run-onboarding">
  ## 7) Ejecutar el asistente de incorporación
</div>

```bash
openclaw onboard --install-daemon
```

Sigue el asistente:

1. **Modo de Gateway:** Local
2. **Auth:** Se recomiendan claves API (OAuth puede ser problemático en una Raspberry Pi sin pantalla)
3. **Channels:** Telegram es el más fácil para empezar
4. **Daemon:** Sí (systemd)

<div id="8-verify-installation">
  ## 8) Verificar la instalación
</div>

```bash
# Verificar estado
openclaw status

# Verificar servicio
sudo systemctl status openclaw

# Ver registros
journalctl -u openclaw -f
```

<div id="9-access-the-dashboard">
  ## 9) Accede al panel
</div>

Como la Raspberry Pi es headless, utiliza un túnel SSH:

```bash
# Desde tu laptop/escritorio
ssh -L 18789:localhost:18789 user@gateway-host

# Luego abre en el navegador
open http://localhost:18789
```

O usa Tailscale para tener acceso permanente:

```bash
# On the Pi
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# Actualizar la configuración
openclaw config set gateway.bind tailnet
sudo systemctl restart openclaw
```

***

<div id="performance-optimizations">
  ## Optimizaciones de rendimiento
</div>

<div id="use-a-usb-ssd-huge-improvement">
  ### Utiliza un SSD USB (Gran mejora)
</div>

Las tarjetas SD son lentas y se desgastan con el tiempo. Un SSD USB mejora considerablemente el rendimiento:

```bash
# Verificar si está arrancando desde USB
lsblk
```

Consulta la [guía de arranque USB para Pi](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#usb-mass-storage-boot) para ver cómo configurarlo.

<div id="reduce-memory-usage">
  ### Reducir el consumo de memoria
</div>

```bash
# Deshabilitar la asignación de memoria GPU (sin interfaz gráfica)
echo 'gpu_mem=16' | sudo tee -a /boot/config.txt

# Disable Bluetooth if not needed
sudo systemctl disable bluetooth
```

<div id="monitor-resources">
  ### Monitorizar recursos
</div>

```bash
# Check memory
free -h

# Verificar temperatura del CPU
vcgencmd measure_temp

# Live monitoring
htop
```

***

<div id="arm-specific-notes">
  ## Notas específicas para ARM
</div>

<div id="binary-compatibility">
  ### Compatibilidad binaria
</div>

La mayoría de las funciones de OpenClaw funcionan en ARM64, pero algunos binarios externos pueden necesitar compilaciones para ARM:

| Herramienta | Estado en ARM64 | Notas |
|------------|-----------------|-------|
| Node.js | ✅ | Funciona muy bien |
| WhatsApp (Baileys) | ✅ | JS puro, sin problemas |
| Telegram | ✅ | JS puro, sin problemas |
| gog (Gmail CLI) | ⚠️ | Comprueba si hay una versión ARM |
| Chromium (browser) | ✅ | `sudo apt install chromium-browser` |

Si una skill falla, comprueba si su binario tiene una compilación para ARM. Muchas herramientas en Go/Rust la tienen; algunas no.

<div id="32-bit-vs-64-bit">
  ### 32 bits vs 64 bits
</div>

**Usa siempre un sistema operativo de 64 bits.** Node.js y muchas herramientas modernas lo requieren. Puedes comprobarlo con:

```bash
uname -m
# Debería mostrar: aarch64 (64 bits) y no armv7l (32 bits)
```

***

<div id="recommended-model-setup">
  ## Configuración recomendada de modelos
</div>

Como la Pi solo ejecuta el Gateway (los modelos se ejecutan en la nube), utiliza modelos basados en API:

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-20250514",
        "fallbacks": ["openai/gpt-4o-mini"]
      }
    }
  }
}
```

**No intentes ejecutar LLM locales en una Raspberry Pi** — incluso los modelos pequeños son demasiado lentos. Deja que Claude/GPT se encarguen del trabajo pesado.

***

<div id="auto-start-on-boot">
  ## Inicio automático al arrancar el sistema
</div>

El asistente de configuración inicial deja esto configurado, pero para comprobarlo:

```bash
# Verificar que el servicio esté habilitado
sudo systemctl is-enabled openclaw

# Habilitar si no lo está
sudo systemctl enable openclaw

# Iniciar en el arranque
sudo systemctl start openclaw
```

***

<div id="troubleshooting">
  ## Solución de problemas
</div>

<div id="out-of-memory-oom">
  ### Memoria insuficiente (OOM)
</div>

```bash
# Verificar memoria
free -h

# Agregar más swap (ver Paso 5)
# O reducir servicios en ejecución en la Pi
```

<div id="slow-performance">
  ### Rendimiento lento
</div>

* Utiliza un SSD USB en lugar de una tarjeta SD
* Desactiva servicios que no necesites: `sudo systemctl disable cups bluetooth avahi-daemon`
* Comprueba la limitación de la CPU: `vcgencmd get_throttled` (debería devolver `0x0`)

<div id="service-wont-start">
  ### El servicio no arranca
</div>

```bash
# Verificar registros
journalctl -u openclaw --no-pager -n 100

# Solución común: recompilar
cd ~/openclaw  # si usas instalación hackable
npm run build
sudo systemctl restart openclaw
```

<div id="arm-binary-issues">
  ### Problemas con binarios ARM
</div>

Si una skill falla con &quot;exec format error&quot;:

1. Verifica si el binario tiene una compilación para ARM64
2. Intenta compilarlo a partir del código fuente
3. O usa un contenedor de Docker con soporte para ARM

<div id="wifi-drops">
  ### Cortes de conexión Wi‑Fi
</div>

Para Raspberry Pi sin pantalla (headless) conectadas por Wi‑Fi:

```bash
# Deshabilitar la administración de energía de WiFi
sudo iwconfig wlan0 power off

# Make permanent
echo 'wireless-power off' | sudo tee -a /etc/network/interfaces
```

***

<div id="cost-comparison">
  ## Comparación de costes
</div>

| Configuración | Coste único | Coste mensual | Notas |
|---------------|-------------|---------------|-------|
| **Pi 4 (2GB)** | ~$45 | $0 | + energía (~$5/año) |
| **Pi 4 (4GB)** | ~$55 | $0 | Recomendado |
| **Pi 5 (4GB)** | ~$60 | $0 | Mejor rendimiento |
| **Pi 5 (8GB)** | ~$80 | $0 | Más de lo necesario, pero preparado para el futuro |
| DigitalOcean | $0 | $6/mes | $72/año |
| Hetzner | $0 | 3,79 €/mes | ~$50/año |

**Punto de equilibrio:** Una Pi se amortiza en ~6-12 meses frente a un VPS en la nube.

***

<div id="see-also">
  ## Consulta también
</div>

* [Guía de Linux](/es/platforms/linux) — configuración general de Linux
* [Guía de DigitalOcean](/es/platforms/digitalocean) — alternativa en la nube
* [Guía de Hetzner](/es/platforms/hetzner) — configuración con Docker
* [Tailscale](/es/gateway/tailscale) — acceso remoto
* [Nodos](/es/nodes) — empareja tu portátil o teléfono con el Gateway que se ejecuta en la Pi