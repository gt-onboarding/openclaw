---
title: VM de macOS
summary: "Ejecuta OpenClaw en una VM de macOS con sandbox (local o alojada) cuando necesites aislamiento o integración con iMessage"
read_when:
  - Quieres aislar OpenClaw de tu entorno principal de macOS
  - Quieres integración con iMessage (BlueBubbles) en un entorno sandbox
  - Quieres un entorno de macOS restablecible que puedas clonar
  - Quieres comparar opciones de VM de macOS locales frente a alojadas
---

<div id="openclaw-on-macos-vms-sandboxing">
  # OpenClaw en máquinas virtuales de macOS (sandboxing)
</div>

<div id="recommended-default-most-users">
  ## Predeterminado recomendado (para la mayoría de los usuarios)
</div>

* **VPS Linux pequeño** para un Gateway siempre encendido y de bajo coste. Consulta [VPS hosting](/es/vps).
* **Hardware dedicado** (Mac mini o máquina Linux) si quieres control total y una **IP residencial** para automatización del navegador. Muchos sitios bloquean direcciones IP de centros de datos, así que la navegación local suele funcionar mejor.
* **Híbrido:** mantén el Gateway en un VPS barato y conecta tu Mac como **nodo** cuando necesites automatización del navegador/UI. Consulta [Nodes](/es/nodes) y [Gateway remote](/es/gateway/remote).

Usa una VM de macOS cuando necesites específicamente capacidades exclusivas de macOS (iMessage/BlueBubbles) o quieras un aislamiento estricto de tu Mac de uso diario.

<div id="macos-vm-options">
  ## Opciones de máquinas virtuales de macOS
</div>

<div id="local-vm-on-your-apple-silicon-mac-lume">
  ### VM local en tu Mac con Apple Silicon (Lume)
</div>

Ejecuta OpenClaw en una VM de macOS aislada (sandbox) en tu Mac con Apple Silicon usando [Lume](https://cua.ai/docs/lume).

Esto te ofrece:

* Entorno macOS completo en aislamiento (tu host se mantiene limpio)
* Compatibilidad con iMessage mediante BlueBubbles (imposible en Linux/Windows)
* Restablecimiento instantáneo clonando VMs
* Sin hardware adicional ni costes en la nube

<div id="hosted-mac-providers-cloud">
  ### Proveedores de Mac alojados (en la nube)
</div>

Si necesitas macOS en la nube, los proveedores de Mac alojados también funcionan:

* [MacStadium](https://www.macstadium.com/) (Mac alojados)
* Otros proveedores de Mac alojados también sirven; sigue su documentación de VM + SSH

Una vez que tengas acceso SSH a una VM de macOS, continúa en el paso 6 más abajo.

***

<div id="quick-path-lume-experienced-users">
  ## Ruta rápida (Lume, usuarios con experiencia)
</div>

1. Instala Lume
2. `lume create openclaw --os macos --ipsw latest`
3. Completa el Asistente de configuración y habilita Inicio de sesión remoto (Remote Login, SSH)
4. `lume run openclaw --no-display`
5. Conéctate por SSH, instala OpenClaw y configura los canales
6. Listo

***

<div id="what-you-need-lume">
  ## Lo que necesitas (Lume)
</div>

* Mac con chip Apple Silicon (M1/M2/M3/M4)
* macOS Sequoia o posterior en el equipo host
* ~60 GB de espacio libre en disco por VM
* ~20 minutos

***

<div id="1-install-lume">
  ## 1) Instalar Lume
</div>

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/trycua/cua/main/libs/lume/scripts/install.sh)"
```

Si `~/.local/bin` no está en tu variable PATH:

```bash
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.zshrc && source ~/.zshrc
```

Comprueba:

```bash
lume --version
```

Documentación: [Instalación de Lume](https://cua.ai/docs/lume/guide/getting-started/installation)

***

<div id="2-create-the-macos-vm">
  ## 2) Crear la VM de macOS
</div>

```bash
lume create openclaw --os macos --ipsw latest
```

Esto descarga macOS y crea la máquina virtual (VM). Una ventana de VNC se abre automáticamente.

Nota: La descarga puede tardar un rato dependiendo de tu conexión.

***

<div id="3-complete-setup-assistant">
  ## 3) Completa el asistente de configuración
</div>

En la ventana de VNC:

1. Selecciona el idioma y la región
2. Omite el ID de Apple (o inicia sesión si quieres usar iMessage más adelante)
3. Crea una cuenta de usuario (recuerda el nombre de usuario y la contraseña)
4. Omite todas las funciones opcionales

Después de completar la configuración, activa SSH:

1. Abre System Settings → General → Sharing (Configuración del Sistema → General → Compartir)
2. Activa &quot;Remote Login&quot; (Inicio de sesión remoto)

***

<div id="4-get-the-vms-ip-address">
  ## 4) Obtener la dirección IP de la VM
</div>

```bash
lume get openclaw
```

Busca la dirección IP (normalmente `192.168.64.x`).

***

<div id="5-ssh-into-the-vm">
  ## 5) Accede a la VM por SSH
</div>

```bash
ssh youruser@192.168.64.X
```

Sustituye `youruser` por la cuenta que creaste y la dirección IP por la de tu máquina virtual.

***

<div id="6-install-openclaw">
  ## 6) Instalar OpenClaw
</div>

Dentro de la VM:

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

Sigue las indicaciones del asistente para configurar tu proveedor de modelos (Anthropic, OpenAI, etc.).

***

<div id="7-configure-channels">
  ## 7) Configura los canales
</div>

Edita el archivo de configuración:

```bash
nano ~/.openclaw/openclaw.json
```

Añade tus canales:

```json
{
  "channels": {
    "whatsapp": {
      "dmPolicy": "allowlist",
      "allowFrom": ["+15551234567"]
    },
    "telegram": {
      "botToken": "YOUR_BOT_TOKEN"
    }
  }
}
```

Luego inicia sesión en WhatsApp escaneando el código QR:

```bash
openclaw channels login
```

***

<div id="8-run-the-vm-headlessly">
  ## 8) Ejecutar la VM en modo sin interfaz gráfica (headless)
</div>

Detén la VM y reiníciala sin interfaz gráfica:

```bash
lume stop openclaw
lume run openclaw --no-display
```

La VM se ejecuta en segundo plano. El daemon de OpenClaw mantiene el Gateway en ejecución.

Para comprobar el estado:

```bash
ssh youruser@192.168.64.X "openclaw status"
```

***

<div id="bonus-imessage-integration">
  ## Extra: integración con iMessage
</div>

Esta es la característica estrella de usarlo en macOS. Usa [BlueBubbles](https://bluebubbles.app) para añadir iMessage a OpenClaw.

Dentro de la VM:

1. Descarga BlueBubbles desde bluebubbles.app
2. Inicia sesión con tu Apple ID
3. Habilita la API web y establece una contraseña
4. Dirige los webhooks de BlueBubbles a tu Gateway (ejemplo: `https://your-gateway-host:3000/bluebubbles-webhook?password=<password>`)

Añade lo siguiente a tu configuración de OpenClaw:

```json
{
  "channels": {
    "bluebubbles": {
      "serverUrl": "http://localhost:1234",
      "password": "your-api-password",
      "webhookPath": "/bluebubbles-webhook"
    }
  }
}
```

Reinicia el Gateway. Ahora tu agente puede enviar y recibir iMessages.

Configuración completa: [Canal de BlueBubbles](/es/channels/bluebubbles)

***

<div id="save-a-golden-image">
  ## Guarda una imagen base
</div>

Antes de seguir personalizando, crea una instantánea de tu estado limpio:

```bash
lume stop openclaw
lume clone openclaw openclaw-golden
```

Restablecer en cualquier momento:

```bash
lume stop openclaw && lume delete openclaw
lume clone openclaw-golden openclaw
lume run openclaw --no-display
```

***

<div id="running-247">
  ## Ejecución 24/7
</div>

Mantén la VM en ejecución:

* Mantén tu Mac conectado a la corriente
* Desactiva el reposo en Ajustes del Sistema → Economizador de energía
* Usa `caffeinate` si es necesario

Para un funcionamiento realmente ininterrumpido, considera un Mac mini dedicado o un pequeño VPS. Consulta [Alojamiento en VPS](/es/vps).

***

<div id="troubleshooting">
  ## Solución de problemas
</div>

| Problema | Solución |
|---------|----------|
| No puedes conectarte por SSH a la VM | Comprueba que &quot;Remote Login&quot; esté habilitado en la Configuración del Sistema de la VM |
| La IP de la VM no aparece | Espera a que la VM termine de arrancar y vuelve a ejecutar `lume get openclaw` |
| El comando `lume` no se encuentra | Añade `~/.local/bin` a tu PATH |
| El código QR de WhatsApp no se puede escanear | Asegúrate de haber iniciado sesión en la VM (y no en el host) al ejecutar `openclaw channels login` |

***

<div id="related-docs">
  ## Documentos relacionados
</div>

* [Alojamiento en VPS](/es/vps)
* [Nodos](/es/nodes)
* [Gateway remoto](/es/gateway/remote)
* [Canal BlueBubbles](/es/channels/bluebubbles)
* [Inicio rápido de Lume](https://cua.ai/docs/lume/guide/getting-started/quickstart)
* [Referencia de la CLI de Lume](https://cua.ai/docs/lume/reference/cli-reference)
* [Configuración desatendida de máquinas virtuales](https://cua.ai/docs/lume/guide/fundamentals/unattended-setup) (avanzado)
* [Sandboxing con Docker](/es/install/docker) (método de aislamiento alternativo)