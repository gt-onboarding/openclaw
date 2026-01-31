---
title: Android
summary: "Aplicación Android (nodo): guía de conexión + Canvas/Chat/Cámara"
read_when:
  - Emparejamiento o reconexión del nodo de Android
  - Depuración del descubrimiento o la autenticación del Gateway en Android
  - Verificación de la consistencia del historial de chat entre clientes
---

<div id="android-app-node">
  # Aplicación Android (nodo)
</div>

<div id="support-snapshot">
  ## Resumen de compatibilidad
</div>

* Rol: aplicación de nodo compañera (Android no ejecuta el Gateway).
* Gateway requerido: sí (ejecútalo en macOS, Linux o Windows mediante WSL2).
* Instalación: [Primeros pasos](/es/start/getting-started) + [Emparejamiento](/es/gateway/pairing).
* Gateway: [Runbook](/es/gateway) + [Configuración](/es/gateway/configuration).
  * Protocolos: [Protocolo del Gateway](/es/gateway/protocol) (nodos + plano de control).

<div id="system-control">
  ## Control del sistema
</div>

El control del sistema (launchd/systemd) reside en el host del Gateway. Consulta [Gateway](/es/gateway).

<div id="connection-runbook">
  ## Runbook de conexión
</div>

App de nodo para Android ⇄ (mDNS/NSD + WebSocket) ⇄ **Gateway**

Android se conecta directamente al WebSocket del Gateway (de forma predeterminada `ws://<host>:18789`) y utiliza el emparejamiento gestionado por el Gateway.

<div id="prerequisites">
  ### Requisitos previos
</div>

* Puedes ejecutar el Gateway en la máquina principal (&quot;master&quot;).
* El dispositivo/emulador Android puede conectarse al WebSocket del gateway:
  * Misma LAN con mDNS/NSD, **o**
  * Misma tailnet de Tailscale usando Wide-Area Bonjour / DNS-SD unicast (ver más abajo), **o**
  * Host/puerto del gateway configurado manualmente (como opción de respaldo)
* Puedes ejecutar la CLI (`openclaw`) en la máquina del gateway (o a través de SSH).

<div id="1-start-the-gateway">
  ### 1) Iniciar el Gateway
</div>

```bash
openclaw gateway --port 18789 --verbose
```

Comprueba en los registros (logs) que aparezca algo como:

* `listening on ws://0.0.0.0:18789`

Para configuraciones solo mediante tailnet (recomendado para Viena ⇄ Londres), vincula el Gateway a la IP de tailnet:

* Establece `gateway.bind: "tailnet"` en `~/.openclaw/openclaw.json` en el host del Gateway.
* Reinicia el Gateway / la aplicación de la barra de menús de macOS.

<div id="2-verify-discovery-optional">
  ### 2) Verificar el descubrimiento (opcional)
</div>

En la máquina del Gateway:

```bash
dns-sd -B _openclaw-gw._tcp local.
```

Más notas de depuración: [Bonjour](/es/gateway/bonjour).

<div id="tailnet-vienna-london-discovery-via-unicast-dns-sd">
  #### Descubrimiento de Tailnet (Viena ⇄ Londres) mediante DNS-SD unicast
</div>

El descubrimiento NSD/mDNS de Android no funciona entre redes distintas. Si tu nodo de Android y el Gateway están en redes diferentes pero conectadas mediante Tailscale, usa Wide-Area Bonjour / DNS-SD unicast en su lugar:

1. Configura una zona DNS-SD (por ejemplo `openclaw.internal.`) en el host del Gateway y publica registros `_openclaw-gw._tcp`.
2. Configura el split DNS de Tailscale para el dominio elegido, apuntando a ese servidor DNS.

Detalles y ejemplo de configuración de CoreDNS: [Bonjour](/es/gateway/bonjour).

<div id="3-connect-from-android">
  ### 3) Conectarse desde Android
</div>

En la app de Android:

* La app mantiene activa su conexión con el Gateway mediante un **servicio en primer plano** (notificación persistente).
* Abre **Settings**.
* En **Discovered Gateways**, selecciona tu Gateway y pulsa **Connect**.
* Si mDNS está bloqueado, usa **Advanced → Manual Gateway** (host + port) y **Connect (Manual)**.

Tras el primer emparejamiento correcto, Android se vuelve a conectar automáticamente al iniciarse:

* Al endpoint manual (si está habilitado), en caso contrario
* Al último Gateway descubierto (en la medida de lo posible).

<div id="4-approve-pairing-cli">
  ### 4) Aprobar el emparejamiento (CLI)
</div>

En la máquina del Gateway:

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

Detalles del emparejamiento: [Emparejamiento del Gateway](/es/gateway/pairing).

<div id="5-verify-the-node-is-connected">
  ### 5) Verifica que el nodo esté conectado
</div>

* Mediante `nodes status`:
  ```bash
  openclaw nodes status
  ```
* Mediante el Gateway:
  ```bash
  openclaw gateway call node.list --params "{}"
  ```

<div id="6-chat-history">
  ### 6) Chat + historial
</div>

El panel de chat del nodo de Android usa la **clave de sesión principal** (`main`) del Gateway, por lo que el historial y las respuestas se comparten con WebChat y otros clientes:

* Historial: `chat.history`
* Enviar: `chat.send`
* Actualizaciones push (mejor esfuerzo): `chat.subscribe` → `event:"chat"`

<div id="7-canvas-camera">
  ### 7) Canvas + cámara
</div>

<div id="gateway-canvas-host-recommended-for-web-content">
  #### Host de Canvas del Gateway (recomendado para contenido web)
</div>

Si quieres que el nodo muestre HTML/CSS/JS real que el agente pueda editar en disco, configura el nodo para que apunte al host de canvas del Gateway.

Nota: los nodos usan el host de canvas independiente en `canvasHost.port` (por defecto `18793`).

1. Crea `~/.openclaw/workspace/canvas/index.html` en el host del Gateway.

2. En el nodo, navega hasta él (LAN):

```bash
openclaw nodes invoke --node "<Android Node>" --command canvas.navigate --params '{"url":"http://<gateway-hostname>.local:18793/__openclaw__/canvas/"}'
```

Tailnet (opcional): si ambos dispositivos están en Tailscale, usa un nombre MagicDNS o la IP de tailnet en lugar de `.local`, por ejemplo `http://<gateway-magicdns>:18793/__openclaw__/canvas/`.

Este servidor inyecta un cliente de recarga en vivo en el HTML y se vuelve a cargar cuando cambian los archivos.
El host de A2UI está en `http://<gateway-host>:18793/__openclaw__/a2ui/`.

Comandos de Canvas (solo primer plano):

* `canvas.eval`, `canvas.snapshot`, `canvas.navigate` (usa `{"url":""}` o `{"url":"/"}` para volver a la estructura predeterminada). `canvas.snapshot` devuelve `{ format, base64 }` (valor predeterminado `format="jpeg"`).
* A2UI: `canvas.a2ui.push`, `canvas.a2ui.reset` (`canvas.a2ui.pushJSONL` alias legado)

Comandos de cámara (solo primer plano; sujetos a permisos):

* `camera.snap` (jpg)
* `camera.clip` (mp4)

Consulta el [nodo de cámara](/es/nodes/camera) para conocer los parámetros y los helpers de la CLI.
