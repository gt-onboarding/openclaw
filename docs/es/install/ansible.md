---
title: Ansible
summary: "Instalación automatizada y endurecida de OpenClaw con Ansible, la VPN Tailscale y aislamiento mediante firewall"
read_when:
  - Quieres un despliegue automatizado del servidor con endurecimiento de la seguridad
  - Necesitas una configuración aislada por firewall con acceso mediante VPN
  - Estás desplegando en servidores remotos Debian/Ubuntu
---

<div id="ansible-installation">
  # Instalación con Ansible
</div>

La forma recomendada de desplegar OpenClaw en servidores de producción es mediante **[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)**, un instalador automatizado con una arquitectura centrada en la seguridad.

<div id="quick-start">
  ## Inicio rápido
</div>

Instalación con un solo comando:

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

> **📦 Guía completa: [github.com/openclaw/openclaw-ansible](https://github.com/openclaw/openclaw-ansible)**
>
> El repositorio openclaw-ansible es la referencia principal para los despliegues con Ansible. Esta página es una visión general rápida.

<div id="what-you-get">
  ## Qué obtienes
</div>

* 🔒 **Seguridad con prioridad al firewall**: aislamiento con UFW + Docker (solo SSH y Tailscale accesibles)
* 🔐 **VPN de Tailscale**: acceso remoto seguro sin exponer servicios públicamente
* 🐳 **Docker**: contenedores sandbox aislados, servicios expuestos solo en localhost
* 🛡️ **Defensa en profundidad**: arquitectura de seguridad de 4 capas
* 🚀 **Configuración con un solo comando**: despliegue completo en minutos
* 🔧 **Integración con systemd**: inicio automático al arrancar, con hardening aplicado

<div id="requirements">
  ## Requisitos
</div>

* **SO**: Debian 11+ o Ubuntu 20.04+
* **Acceso**: Privilegios de root o sudo
* **Red**: Conexión a Internet para la instalación de paquetes
* **Ansible**: 2.14+ (se instala automáticamente mediante el script de inicio rápido)

<div id="what-gets-installed">
  ## Qué se instala
</div>

El playbook de Ansible instala y configura:

1. **Tailscale** (VPN en malla para acceso remoto seguro)
2. **Cortafuegos UFW** (solo puertos de SSH + Tailscale)
3. **Docker CE + Compose V2** (para sandboxes de agentes)
4. **Node.js 22.x + pnpm** (dependencias de tiempo de ejecución)
5. **OpenClaw** (en el host, no en contenedores)
6. **Servicio systemd** (inicio automático con endurecimiento de seguridad)

Nota: El Gateway se ejecuta **directamente en el host** (no en Docker), pero los sandboxes de agentes usan Docker para aislamiento. Consulta [Sandboxing](/es/gateway/sandboxing) para más detalles.

<div id="post-install-setup">
  ## Configuración posterior a la instalación
</div>

Una vez que haya finalizado la instalación, cambia al usuario openclaw:

```bash
sudo -i -u openclaw
```

El script de post-instalación te guiará a través de:

1. **Asistente de incorporación**: Configura la configuración de OpenClaw
2. **Inicio de sesión con el proveedor**: Conecta WhatsApp/Telegram/Discord/Signal
3. **Pruebas del Gateway**: Verifica la instalación
4. **Configuración de Tailscale**: Conéctate a tu red VPN mallada

<div id="quick-commands">
  ### Comandos rápidos
</div>

```bash
# Verificar el estado del servicio
sudo systemctl status openclaw

# Ver logs en vivo
sudo journalctl -u openclaw -f

# Reiniciar el Gateway
sudo systemctl restart openclaw

# Inicio de sesión del proveedor (ejecutar como usuario openclaw)
sudo -i -u openclaw
openclaw channels login
```

<div id="security-architecture">
  ## Arquitectura de seguridad
</div>

<div id="4-layer-defense">
  ### Defensa en 4 capas
</div>

1. **Firewall (UFW)**: Solo SSH (22) + Tailscale (41641/udp) expuestos públicamente
2. **VPN (Tailscale)**: Gateway accesible únicamente a través de la malla VPN
3. **Aislamiento de Docker**: La cadena DOCKER-USER de iptables evita la exposición de puertos externos
4. **Endurecimiento de systemd**: NoNewPrivileges, PrivateTmp, usuario sin privilegios

<div id="verification">
  ### Verificación
</div>

Verifica la superficie de ataque externa:

```bash
nmap -p- YOUR_SERVER_IP
```

Solo debería mostrar **el puerto 22** (SSH) como abierto. Todos los demás servicios (Gateway, Docker) deben permanecer bloqueados.

<div id="docker-availability">
  ### Disponibilidad de Docker
</div>

Docker se instala únicamente para **sandboxes de agentes** (ejecución aislada de herramientas), no para ejecutar el propio Gateway. El Gateway escucha únicamente en localhost y es accesible mediante la VPN Tailscale.

Consulta [Multi-Agent Sandbox &amp; Tools](/es/multi-agent-sandbox-tools) para la configuración del sandbox.

<div id="manual-installation">
  ## Instalación manual
</div>

Si prefieres tener un control más directo que el que proporciona la automatización:

```bash
# 1. Install prerequisites
sudo apt update && sudo apt install -y ansible git

# 2. Clone repository
git clone https://github.com/openclaw/openclaw-ansible.git
cd openclaw-ansible

# 3. Install Ansible collections
ansible-galaxy collection install -r requirements.yml

# 4. Run playbook
./run-playbook.sh

# O ejecutar directamente (luego ejecutar manualmente /tmp/openclaw-setup.sh)
# ansible-playbook playbook.yml --ask-become-pass
```

<div id="updating-openclaw">
  ## Actualizar OpenClaw
</div>

El instalador de Ansible configura OpenClaw para que se pueda actualizar manualmente. Consulta [Actualización](/es/install/updating) para conocer el flujo de actualización estándar.

Para volver a ejecutar el playbook de Ansible (por ejemplo, para aplicar cambios de configuración):

```bash
cd openclaw-ansible
./run-playbook.sh
```

Nota: Esto es idempotente y puedes ejecutarlo con seguridad varias veces.

<div id="troubleshooting">
  ## Solución de problemas
</div>

<div id="firewall-blocks-my-connection">
  ### El firewall bloquea mi conexión
</div>

Si te has quedado sin acceso:

* Asegúrate primero de que puedes acceder a través de la VPN de Tailscale
* El acceso SSH (puerto 22) siempre está permitido
* El Gateway solo es accesible a través de Tailscale por diseño

<div id="service-wont-start">
  ### El servicio no arranca
</div>

```bash
# Check logs
sudo journalctl -u openclaw -n 100

# Verify permissions
sudo ls -la /opt/openclaw

# Probar inicio manual
sudo -i -u openclaw
cd ~/openclaw
pnpm start
```

<div id="docker-sandbox-issues">
  ### Problemas de la sandbox de Docker
</div>

```bash
# Verificar que Docker esté ejecutándose
sudo systemctl status docker

# Verificar la imagen de sandbox
sudo docker images | grep openclaw-sandbox

# Construir la imagen de sandbox si no existe
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

<div id="provider-login-fails">
  ### Error al iniciar sesión del proveedor
</div>

Asegúrate de que estás ejecutando esto como el usuario `openclaw`:

```bash
sudo -i -u openclaw
openclaw channels login
```

<div id="advanced-configuration">
  ## Configuración avanzada
</div>

Para consultar información detallada sobre la arquitectura de seguridad y la resolución de problemas:

* [Arquitectura de seguridad](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
* [Detalles técnicos](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
* [Guía de resolución de problemas](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)

<div id="related">
  ## Relacionado
</div>

* [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — guía completa de despliegue
* [Docker](/es/install/docker) — configuración del Gateway mediante contenedores
* [Sandboxing](/es/gateway/sandboxing) — configuración de la sandbox de agentes
* [Multi-Agent Sandbox &amp; Tools](/es/multi-agent-sandbox-tools) — aislamiento por agente