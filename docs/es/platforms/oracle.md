---
title: Oracle
summary: "OpenClaw en Oracle Cloud (Always Free ARM)"
read_when:
  - Configurar OpenClaw en Oracle Cloud
  - Buscar alojamiento VPS económico para OpenClaw
  - Tener OpenClaw 24/7 en un servidor pequeño
---

<div id="openclaw-on-oracle-cloud-oci">
  # OpenClaw en Oracle Cloud (OCI)
</div>

<div id="goal">
  ## Objetivo
</div>

Ejecutar un Gateway persistente de OpenClaw en el nivel ARM **Always Free** de Oracle Cloud.

El nivel gratuito de Oracle puede encajar muy bien con OpenClaw (especialmente si ya tienes una cuenta de OCI), pero tiene sus contrapartidas:

* Arquitectura ARM (la mayoría de las cosas funcionan, pero algunos binarios pueden estar disponibles solo para x86)
* La disponibilidad de capacidad y el proceso de registro pueden resultar algo problemáticos

<div id="cost-comparison-2026">
  ## Comparación de costos (2026)
</div>

| Proveedor | Plan | Especificaciones | Precio/mes | Notas |
|----------|------|-------|----------|-------|
| Oracle Cloud | Always Free ARM | hasta 4 OCPU, 24GB RAM | $0 | ARM, capacidad limitada |
| Hetzner | CX22 | 2 vCPU, 4GB RAM | ~ $4 | Opción de pago más económica |
| DigitalOcean | Basic | 1 vCPU, 1GB RAM | $6 | UI sencilla, buena documentación |
| Vultr | Cloud Compute | 1 vCPU, 1GB RAM | $6 | Muchas ubicaciones |
| Linode | Nanode | 1 vCPU, 1GB RAM | $5 | Ahora parte de Akamai |

***

<div id="prerequisites">
  ## Requisitos previos
</div>

* Cuenta de Oracle Cloud ([registro](https://www.oracle.com/cloud/free/)) — consulta la [guía de registro de la comunidad](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd) si encuentras problemas
* Cuenta de Tailscale (gratuita en [tailscale.com](https://tailscale.com))
* ~30 minutos

<div id="1-create-an-oci-instance">
  ## 1) Crear una instancia de OCI
</div>

1. Inicia sesión en [Oracle Cloud Console](https://cloud.oracle.com/)
2. Ve a **Compute → Instances → Create Instance**
3. Configura:
   * **Name:** `openclaw`
   * **Image:** Ubuntu 24.04 (aarch64)
   * **Shape:** `VM.Standard.A1.Flex` (Ampere ARM)
   * **OCPUs:** 2 (o hasta 4)
   * **Memory:** 12 GB (o hasta 24 GB)
   * **Boot volume:** 50 GB (hasta 200 GB sin costo adicional)
   * **SSH key:** Agrega tu clave pública
4. Haz clic en **Create**
5. Toma nota de la dirección IP pública

**Consejo:** Si la creación de la instancia falla con &quot;Out of capacity&quot;, prueba con un dominio de disponibilidad diferente o inténtalo de nuevo más tarde. La capacidad de la capa gratuita es limitada.

<div id="2-connect-and-update">
  ## 2) Conéctate y actualiza
</div>

```bash
# Conectar mediante IP pública
ssh ubuntu@YOUR_PUBLIC_IP

# Actualizar el sistema
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential
```

**Nota:** `build-essential` es necesario para compilar algunas dependencias en ARM.

<div id="3-configure-user-and-hostname">
  ## 3) Configurar el usuario y el nombre de host
</div>

```bash
# Establecer nombre de host
sudo hostnamectl set-hostname openclaw

# Establecer contraseña para el usuario ubuntu
sudo passwd ubuntu

# Habilitar lingering (mantiene los servicios de usuario en ejecución después de cerrar sesión)
sudo loginctl enable-linger ubuntu
```

<div id="4-install-tailscale">
  ## 4) Instalar Tailscale
</div>

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --ssh --hostname=openclaw
```

Esto habilita SSH de Tailscale, para que puedas conectarte mediante `ssh openclaw` desde cualquier dispositivo de tu tailnet, sin necesidad de una IP pública.

Verifica:

```bash
tailscale status
```

**A partir de ahora, conéctate a través de Tailscale:** `ssh ubuntu@openclaw` (o usa la IP de Tailscale).

<div id="5-install-openclaw">
  ## 5) Instalar OpenClaw
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
source ~/.bashrc
```

Cuando se te pregunte &quot;How do you want to hatch your bot?&quot;, selecciona **&quot;Do this later&quot;**.

> Nota: Si encuentras problemas al compilar de forma nativa para ARM, empieza instalando paquetes del sistema (por ejemplo, `sudo apt install -y build-essential`) antes de recurrir a Homebrew.

<div id="6-configure-gateway-loopback-token-auth-and-enable-tailscale-serve">
  ## 6) Configurar Gateway (loopback + autenticación con token) y habilitar Tailscale Serve
</div>

Usa la autenticación con token como valor predeterminado. Es predecible y evita tener que usar flags de Control UI de “autenticación insegura”.

```bash
# Keep the Gateway private on the VM
openclaw config set gateway.bind loopback

# Require auth for the Gateway + Control UI
openclaw config set gateway.auth.mode token
openclaw doctor --generate-gateway-token

# Expón mediante Tailscale Serve (HTTPS + acceso a tailnet)
openclaw config set gateway.tailscale.mode serve
openclaw config set gateway.trustedProxies '["127.0.0.1"]'

systemctl --user restart openclaw-gateway
```

<div id="7-verify">
  ## 7) Verificar
</div>

```bash
# Verificar versión
openclaw --version

# Verificar estado del daemon
systemctl --user status openclaw-gateway

# Verificar Tailscale Serve
tailscale serve status

# Probar respuesta local
curl http://localhost:18789
```

<div id="8-lock-down-vcn-security">
  ## 8) Refuerza la seguridad de la VCN
</div>

Ahora que todo está funcionando, refuerza la VCN para bloquear todo el tráfico excepto Tailscale. La Virtual Cloud Network de OCI actúa como un firewall en el borde de la red: el tráfico se bloquea antes de que llegue a tu instancia.

1. Ve a **Networking → Virtual Cloud Networks** en la consola de OCI
2. Haz clic en tu VCN → **Security Lists** → Default Security List
3. **Elimina** todas las reglas de entrada (ingress) excepto:
   * `0.0.0.0/0 UDP 41641` (Tailscale)
4. Mantén las reglas de salida (egress) predeterminadas (permitir todo el tráfico saliente)

Esto bloquea SSH en el puerto 22, HTTP, HTTPS y todo lo demás en el borde de la red. A partir de ahora, solo podrás conectarte mediante Tailscale.

***

<div id="access-the-control-ui">
  ## Accede a Control UI
</div>

Desde cualquier dispositivo de tu red de Tailscale:

```
https://openclaw.<tailnet-name>.ts.net/
```

Reemplaza `<tailnet-name>` por el nombre de tu tailnet (visible en `tailscale status`).

No necesitas un túnel SSH. Tailscale proporciona:

* Cifrado HTTPS (certificados automáticos)
* Autenticación mediante la identidad de Tailscale
* Acceso desde cualquier dispositivo de tu tailnet (portátil, teléfono, etc.)

***

<div id="security-vcn-tailscale-recommended-baseline">
  ## Seguridad: VCN + Tailscale (línea base recomendada)
</div>

Con la VCN bloqueada (solo el UDP 41641 abierto) y el Gateway vinculado a la interfaz de loopback, obtienes una sólida defensa en profundidad: el tráfico público se bloquea en el borde de la red y el acceso de administración se realiza a través de tu tailnet.

Esta configuración a menudo elimina la *necesidad* de reglas adicionales de firewall basadas en el host únicamente para detener ataques de fuerza bruta de SSH a escala de todo Internet, pero aun así debes mantener el sistema operativo actualizado, ejecutar `openclaw security audit` y verificar que no estés escuchando accidentalmente en interfaces públicas.

<div id="whats-already-protected">
  ### Lo que ya está protegido
</div>

| Paso tradicional | ¿Es necesario? | Por qué |
|------------------|----------------|--------|
| Firewall UFW | No | La VCN bloquea antes de que el tráfico llegue a la instancia |
| fail2ban | No | No hay ataques de fuerza bruta si el puerto 22 está bloqueado en la VCN |
| Fortalecimiento de sshd | No | Tailscale SSH no usa sshd |
| Deshabilitar el inicio de sesión de root | No | Tailscale usa la identidad de Tailscale, no los usuarios del sistema |
| Autenticación SSH solo con clave | No | Tailscale autentica a través de tu tailnet |
| Fortalecimiento de IPv6 | Normalmente no | Depende de la configuración de tu VCN/subred; verifica qué está realmente asignado o expuesto |

<div id="still-recommended">
  ### Sigue siendo recomendable
</div>

* **Permisos de las credenciales:** `chmod 700 ~/.openclaw`
* **Auditoría de seguridad:** `openclaw security audit`
* **Actualizaciones del sistema:** ejecuta periódicamente `sudo apt update && sudo apt upgrade`
* **Supervisar Tailscale:** supervisa los dispositivos en la [consola de administración de Tailscale](https://login.tailscale.com/admin)

<div id="verify-security-posture">
  ### Verificar el estado de seguridad
</div>

```bash
# Confirmar que no hay puertos públicos en escucha
sudo ss -tlnp | grep -v '127.0.0.1\|::1'

# Verificar que Tailscale SSH esté activo
tailscale status | grep -q 'offers: ssh' && echo "Tailscale SSH active"

# Opcional: deshabilitar sshd por completo
sudo systemctl disable --now ssh
```

***

<div id="fallback-ssh-tunnel">
  ## Alternativa: túnel SSH
</div>

Si Tailscale Serve no funciona, utiliza un túnel SSH:

```bash
# Desde tu máquina local (a través de Tailscale)
ssh -L 18789:127.0.0.1:18789 ubuntu@openclaw
```

Luego, abre `http://localhost:18789`.

***

<div id="troubleshooting">
  ## Solución de problemas
</div>

<div id="instance-creation-fails-out-of-capacity">
  ### Error al crear la instancia (&quot;Out of capacity&quot;)
</div>

Las instancias ARM del nivel gratuito son muy populares. Intenta lo siguiente:

* Usar un dominio de disponibilidad distinto
* Reintentar en horas de baja demanda (temprano por la mañana)
* Usar el filtro &quot;Always Free&quot; al seleccionar el shape

<div id="tailscale-wont-connect">
  ### Tailscale no se puede conectar
</div>

```bash
# Verificar el estado
sudo tailscale status

# Volver a autenticar
sudo tailscale up --ssh --hostname=openclaw --reset
```

<div id="gateway-wont-start">
  ### El Gateway no arranca
</div>

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl --user -u openclaw-gateway -n 50
```

<div id="cant-reach-control-ui">
  ### No se puede acceder a Control UI
</div>

```bash
# Verificar que Tailscale Serve esté ejecutándose
tailscale serve status

# Verificar que el Gateway esté escuchando
curl http://localhost:18789

# Reiniciar si es necesario
systemctl --user restart openclaw-gateway
```

<div id="arm-binary-issues">
  ### Problemas con binarios ARM
</div>

Es posible que algunas herramientas no tengan compilaciones para ARM. Comprueba lo siguiente:

```bash
uname -m  # Debería mostrar aarch64
```

La mayoría de los paquetes de npm funcionan bien. Para los binarios, busca versiones `linux-arm64` o `aarch64`.

***

<div id="persistence">
  ## Persistencia
</div>

Todo el estado se almacena en:

* `~/.openclaw/` — configuración, credenciales, datos de sesión
* `~/.openclaw/workspace/` — espacio de trabajo (SOUL.md, memoria, artefactos)

Realiza copias de seguridad periódicamente:

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

***

<div id="see-also">
  ## Ver también
</div>

* [Gateway remote access](/es/gateway/remote) — otros patrones de acceso remoto
* [Tailscale integration](/es/gateway/tailscale) — documentación completa de Tailscale
* [Gateway configuration](/es/gateway/configuration) — todas las opciones de configuración
* [DigitalOcean guide](/es/platforms/digitalocean) — si quieres una opción de pago con un registro más sencillo
* [Hetzner guide](/es/platforms/hetzner) — alternativa basada en Docker