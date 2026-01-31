---
title: Remoto
summary: "Flujo de la app de macOS para controlar de forma remota un Gateway OpenClaw mediante SSH"
read_when:
  - Configurar o depurar el control remoto desde macOS
---

<div id="remote-openclaw-macos-remote-host">
  # OpenClaw remoto (macOS ⇄ host remoto)
</div>

Este flujo permite que la aplicación de macOS actúe como un control remoto completo para un Gateway OpenClaw que se ejecuta en otro host (equipo/servidor). Es la funcionalidad de la aplicación **Remote over SSH** (ejecución remota). Todas las funcionalidades —comprobaciones de estado, reenvío de Voice Wake y Web Chat— reutilizan la misma configuración SSH remota desde *Settings → General*.

<div id="modes">
  ## Modos
</div>

* **Local (este Mac)**: Todo se ejecuta en el portátil. No se utiliza SSH.
* **Remoto a través de SSH (predeterminado)**: Los comandos de OpenClaw se ejecutan en el host remoto. La aplicación de macOS abre una conexión SSH con `-o BatchMode`, junto con la identidad/clave que elijas y un reenvío de puerto local.
* **Remoto directo (ws/wss)**: Sin túnel SSH. La aplicación de macOS se conecta directamente a la URL del Gateway (por ejemplo, mediante Tailscale Serve o un proxy inverso HTTPS público).

<div id="remote-transports">
  ## Transportes remotos
</div>

El modo remoto admite dos transportes:

* **Túnel SSH** (predeterminado): Usa `ssh -N -L ...` para reenviar el puerto del Gateway a localhost. El Gateway verá la IP del nodo como `127.0.0.1` porque el túnel usa loopback.
* **Directo (ws/wss)**: Se conecta directamente a la URL del Gateway. El Gateway verá la IP real del cliente.

<div id="prereqs-on-the-remote-host">
  ## Requisitos previos en el host remoto
</div>

1. Instala Node + pnpm y compila/instala la CLI de OpenClaw (`pnpm install && pnpm build && pnpm link --global`).
2. Asegúrate de que `openclaw` esté en el PATH en shells no interactivos (crea un symlink en `/usr/local/bin` o `/opt/homebrew/bin` si es necesario).
3. Habilita SSH con autenticación con clave. Recomendamos usar direcciones IP de **Tailscale** para una conectividad estable fuera de la LAN.

<div id="macos-app-setup">
  ## Configuración de la app en macOS
</div>

1. Abre *Settings → General*.
2. En **OpenClaw runs**, elige **Remote over SSH** y configura:
   * **Transport**: **SSH tunnel** o **Direct (ws/wss)**.
   * **SSH target**: `user@host` (opcional `:port`).
     * Si el Gateway está en la misma LAN y anuncia Bonjour, elígelo de la lista de elementos detectados para rellenar automáticamente este campo.
   * **Gateway URL** (solo Direct): `wss://gateway.example.ts.net` (o `ws://...` para local/LAN).
   * **Identity file** (avanzado): ruta a tu clave.
   * **Project root** (avanzado): ruta del checkout remoto usada para los comandos.
   * **CLI path** (avanzado): ruta opcional a un entrypoint/binario `openclaw` ejecutable (se rellena automáticamente cuando se anuncia).
3. Pulsa **Test remote**. Si se completa correctamente, indica que el comando remoto `openclaw status --json` se ejecuta correctamente. Los fallos suelen indicar problemas con PATH/CLI; el código de salida 127 significa que no se encuentra la CLI en el host remoto.
4. Las comprobaciones de estado (health checks) y Web Chat ahora se ejecutarán automáticamente a través de este túnel SSH.

<div id="web-chat">
  ## Web Chat
</div>

* **Túnel SSH**: Web Chat se conecta al Gateway a través del puerto de control WebSocket reenviado (18789 por defecto).
* **Directo (ws/wss)**: Web Chat se conecta directamente a la URL configurada del Gateway.
* Ya no hay un servidor HTTP independiente para Web Chat.

<div id="permissions">
  ## Permisos
</div>

* El host remoto necesita los mismos permisos TCC que el local (Automatización, Accesibilidad, Grabación de pantalla, Micrófono, Reconocimiento de voz, Notificaciones). Ejecuta el proceso de onboarding en esa máquina para concederlos una sola vez.
* Los nodos anuncian su estado de permisos mediante `node.list` / `node.describe` para que los agentes sepan qué está disponible.

<div id="security-notes">
  ## Notas de seguridad
</div>

* Prioriza enlaces de loopback en el host remoto y conéctate a través de SSH o Tailscale.
* Si vinculas el Gateway a una interfaz que no sea de loopback, requiere autenticación mediante token/contraseña.
* Consulta [Security](/es/gateway/security) y [Tailscale](/es/gateway/tailscale).

<div id="whatsapp-login-flow-remote">
  ## Flujo de inicio de sesión de WhatsApp (remoto)
</div>

* Ejecuta `openclaw channels login --verbose` **en el host remoto**. Escanea el código QR con WhatsApp en tu teléfono.
* Vuelve a ejecutar el inicio de sesión en ese host si la autenticación caduca. La comprobación de estado (health check) mostrará problemas de enlace.

<div id="troubleshooting">
  ## Solución de problemas
</div>

* **exit 127 / not found**: `openclaw` no está en el PATH para shells sin inicio de sesión (non-login). Añádelo a `/etc/paths`, a tu rc de shell o crea un symlink en `/usr/local/bin`/`/opt/homebrew/bin`.
* **Health probe failed**: verifica la conectividad SSH, la variable PATH y que Baileys haya iniciado sesión (`openclaw status --json`).
* **Web Chat atascado**: confirma que el Gateway se esté ejecutando en el host remoto y que el puerto reenviado coincida con el puerto WS del Gateway; la UI requiere una conexión WS activa y en buen estado.
* **La IP del nodo aparece como 127.0.0.1**: esto es lo esperado con el túnel SSH. Cambia **Transport** a **Direct (ws/wss)** si quieres que el Gateway vea la IP real del cliente.
* **Voice Wake**: las frases de activación se reenvían automáticamente en modo remoto; no se necesita un reenviador aparte.

<div id="notification-sounds">
  ## Sonidos de notificación
</div>

Configura sonidos por notificación en scripts con `openclaw` y `node.invoke`, por ejemplo:

```bash
openclaw nodes notify --node <id> --title "Ping" --body "Gateway remoto listo" --sound Glass
```

Ya no hay una opción global de «sonido predeterminado» en la app; quien realiza la llamada elige un sonido (o ninguno) para cada solicitud.
