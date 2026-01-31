---
title: Windows
summary: "Compatibilidad con Windows (WSL2) y estado de la aplicación complementaria"
read_when:
  - Cuando instalas OpenClaw en Windows
  - Cuando buscas el estado de la aplicación complementaria para Windows
---

<div id="windows-wsl2">
  # Windows (WSL2)
</div>

Se recomienda usar OpenClaw en Windows **a través de WSL2** (Ubuntu recomendado). El
CLI y el Gateway se ejecutan dentro de Linux, lo que mantiene el entorno de ejecución consistente y hace
que las herramientas sean mucho más compatibles (Node/Bun/pnpm, binarios de Linux, habilidades). Las instalaciones
nativas en Windows no están probadas y son más problemáticas.

Se planean aplicaciones nativas de Windows como aplicaciones compañeras.

<div id="install-wsl2">
  ## Instalar (WSL2)
</div>

* [Primeros pasos](/es/start/getting-started) (usar dentro de WSL)
* [Instalación y actualizaciones](/es/install/updating)
* Guía oficial de WSL2 de Microsoft: https://learn.microsoft.com/windows/wsl/install

<div id="gateway">
  ## Gateway
</div>

* [Runbook del Gateway](/es/gateway)
* [Configuración](/es/gateway/configuration)

<div id="gateway-service-install-cli">
  ## Instalación del servicio Gateway (CLI)
</div>

En WSL2:

```
openclaw onboard --install-daemon
```

O bien:

```
openclaw gateway install
```

O bien:

```
openclaw configure
```

Selecciona **Gateway service** cuando se te pida.

Reparar/migrar:

```
openclaw doctor
```

<div id="advanced-expose-wsl-services-over-lan-portproxy">
  ## Avanzado: exponer servicios de WSL a través de la LAN (portproxy)
</div>

WSL tiene su propia red virtual. Si otra máquina necesita acceder a un servicio
que se ejecuta **dentro de WSL** (SSH, un servidor TTS local o el Gateway), debes
reenviar un puerto de Windows a la IP actual de WSL. La IP de WSL cambia cada vez que se reinicia,
por lo que es posible que tengas que actualizar la regla de reenvío.

Ejemplo (PowerShell **como Administrador**):

```powershell
$Distro = "Ubuntu-24.04"
$ListenPort = 2222
$TargetPort = 22

$WslIp = (wsl -d $Distro -- hostname -I).Trim().Split(" ")[0]
if (-not $WslIp) { throw "No se encontró la IP de WSL." }

netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=$ListenPort `
  connectaddress=$WslIp connectport=$TargetPort
```

Permite el tráfico de ese puerto a través del Firewall de Windows (solo una vez):

```powershell
New-NetFirewallRule -DisplayName "WSL SSH $ListenPort" -Direction Inbound `
  -Protocol TCP -LocalPort $ListenPort -Action Allow
```

Restablece el portproxy después de que WSL se reinicie:

```powershell
netsh interface portproxy delete v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 | Out-Null
netsh interface portproxy add v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 `
  connectaddress=$WslIp connectport=$TargetPort | Out-Null
```

Notas:

* El SSH desde otra máquina debe dirigirse a la **IP del host de Windows** (por ejemplo: `ssh user@windows-host -p 2222`).
* Los nodos remotos deben apuntar a una URL del Gateway **accesible** (no `127.0.0.1`); usa
  `openclaw status --all` para confirmarlo.
* Usa `listenaddress=0.0.0.0` para acceso desde la LAN; `127.0.0.1` lo mantiene solo en local.
* Si quieres que esto sea automático, registra una tarea programada para ejecutar el paso de actualización al iniciar sesión.

<div id="step-by-step-wsl2-install">
  ## Instalación de WSL2 paso a paso
</div>

<div id="1-install-wsl2-ubuntu">
  ### 1) Instalar WSL2 + Ubuntu
</div>

Abre PowerShell como administrador:

```powershell
wsl --install
# O elige una distro explícitamente:
wsl --list --online
wsl --install -d Ubuntu-24.04
```

Reinicia el sistema si Windows te lo pide.

<div id="2-enable-systemd-required-for-gateway-install">
  ### 2) Habilitar systemd (obligatorio para instalar el Gateway)
</div>

En tu terminal de WSL:

```bash
sudo tee /etc/wsl.conf >/dev/null <<'EOF'
[boot]
systemd=true
EOF
```

Luego, desde PowerShell:

```powershell
wsl --shutdown
```

Vuelve a abrir Ubuntu y comprueba lo siguiente:

```bash
systemctl --user status
```

<div id="3-install-openclaw-inside-wsl">
  ### 3) Instalar OpenClaw (dentro de WSL)
</div>

Sigue la guía de primeros pasos para Linux dentro de WSL:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # instala automáticamente las dependencias de la UI en la primera ejecución
pnpm build
openclaw onboard
```

Guía completa: [Primeros pasos](/es/start/getting-started)

<div id="windows-companion-app">
  ## Aplicación complementaria para Windows
</div>

Todavía no tenemos una aplicación complementaria para Windows. Las contribuciones son bienvenidas si quieres contribuir para que esto sea posible.