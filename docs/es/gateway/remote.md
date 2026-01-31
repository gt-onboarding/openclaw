---
title: Acceso remoto
summary: "Acceso remoto mediante túneles SSH (Gateway WS) y tailnets"
read_when:
  - Ejecutar o solucionar problemas de despliegues remotos del Gateway
---

<div id="remote-access-ssh-tunnels-and-tailnets">
  # Acceso remoto (SSH, túneles y tailnets)
</div>

Este repositorio admite acceso remoto mediante SSH (“remote over SSH”) manteniendo un único Gateway (el principal) ejecutándose en un host dedicado (escritorio/servidor) y conectando los clientes a él.

* Para **operadores (tú / la aplicación de macOS)**: el tunelizado SSH es el mecanismo de respaldo universal.
* Para **nodos (iOS/Android y dispositivos futuros)**: conéctate al **WebSocket** del Gateway (por LAN/tailnet o mediante un túnel SSH, según sea necesario).

<div id="the-core-idea">
  ## La idea principal
</div>

* El WebSocket del Gateway se vincula a la **loopback** en tu puerto configurado (por defecto, 18789).
* Para uso remoto, reenvía ese puerto de loopback a través de SSH (o usa un tailnet/VPN y reduce la cantidad de tunelización).

<div id="common-vpntailnet-setups-where-the-agent-lives">
  ## Configuraciones comunes de VPN/tailnet (dónde vive el agente)
</div>

Piensa en el **host del Gateway** como “donde vive el agente”. Es el que posee las sesiones, perfiles de autenticación, canales y estado.
Tu portátil/escritorio (y nodos) se conectan a ese host.

<div id="1-always-on-gateway-in-your-tailnet-vps-or-home-server">
  ### 1) Gateway siempre activo en tu tailnet (VPS o servidor en casa)
</div>

Ejecuta el Gateway en un host persistente y accede a él mediante **Tailscale** o SSH.

* **Mejor UX:** mantén `gateway.bind: "loopback"` y usa **Tailscale Serve** para el Control UI.
* **Alternativa:** mantén loopback + túnel SSH desde cualquier máquina que necesite acceso.
* **Ejemplos:** [exe.dev](/es/platforms/exe-dev) (VM sencilla) o [Hetzner](/es/platforms/hetzner) (VPS de producción).

Esto es ideal cuando tu portátil entra en suspensión con frecuencia, pero quieres que el agente esté siempre en ejecución.

<div id="2-home-desktop-runs-the-gateway-laptop-is-remote-control">
  ### 2) El equipo de sobremesa en casa ejecuta el Gateway, el portátil es el control remoto
</div>

El portátil **no** ejecuta el agente. Solo se conecta de forma remota:

* Utiliza el modo **Remote over SSH** de la app de macOS (Settings → General → &quot;OpenClaw runs&quot;).
* La app abre y gestiona el túnel, así que WebChat y las comprobaciones de estado simplemente funcionan.

Runbook: [acceso remoto en macOS](/es/platforms/mac/remote).

<div id="3-laptop-runs-the-gateway-remote-access-from-other-machines">
  ### 3) El portátil ejecuta el Gateway, acceso remoto desde otras máquinas
</div>

Mantén el Gateway local pero expónlo de forma segura:

* Túnel SSH al portátil desde otras máquinas, o
* Usa Tailscale Serve para exponer la Control UI y mantén el Gateway accesible solo por loopback.

Guía: [Tailscale](/es/gateway/tailscale) y [Descripción general de la web](/es/web).

<div id="command-flow-what-runs-where">
  ## Flujo de comandos (qué se ejecuta dónde)
</div>

Un servicio de Gateway gestiona el estado y los canales. Los nodos son periféricos.

Ejemplo de flujo (Telegram → nodo):

* El mensaje de Telegram llega al **Gateway**.
* El Gateway ejecuta el **agente** y decide si debe invocar una herramienta del nodo.
* El Gateway llama al **nodo** a través del WebSocket del Gateway (`node.*` RPC).
* El nodo devuelve el resultado; el Gateway responde de vuelta a Telegram.

Notas:

* **Los nodos no ejecutan el servicio de Gateway.** Solo deberías ejecutar un Gateway por host, a menos que ejecutes perfiles aislados de forma intencional (consulta [Multiple gateways](/es/gateway/multiple-gateways)).
* La app de macOS en “modo nodo” es solo un cliente de nodo que se conecta mediante el WebSocket del Gateway.

<div id="ssh-tunnel-cli-tools">
  ## Túnel SSH (CLI + herramientas)
</div>

Crea un túnel local al WS del Gateway remoto:

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Con el túnel activo:

* `openclaw health` y `openclaw status --deep` ahora se conectan al Gateway remoto a través de `ws://127.0.0.1:18789`.
* `openclaw gateway {status,health,send,agent,call}` también puede apuntar a la URL reenviada mediante `--url` cuando sea necesario.

Nota: reemplaza `18789` por tu `gateway.port` configurado (o `--port`/`OPENCLAW_GATEWAY_PORT`).

<div id="cli-remote-defaults">
  ## Valores predeterminados remotos en la CLI
</div>

Puedes guardar un destino remoto para que los comandos de la CLI lo usen de forma predeterminada:

```json5
{
  gateway: {
    modo: "remote",
    remote: {
      url: "ws://127.0.0.1:18789",
      token: "your-token"
    }
  }
}
```

Cuando el Gateway solo escucha en loopback, mantén la URL en `ws://127.0.0.1:18789` y abre primero el túnel SSH.

<div id="chat-ui-over-ssh">
  ## UI de chat a través de SSH
</div>

WebChat ya no usa un puerto HTTP separado. La UI de chat de SwiftUI se conecta directamente al WebSocket del Gateway.

* Reenvía el puerto `18789` a través de SSH (ver arriba) y luego conecta los clientes a `ws://127.0.0.1:18789`.
* En macOS, usa preferentemente el modo “Remote over SSH” de la app, que gestiona el túnel automáticamente.

<div id="macos-app-remote-over-ssh">
  ## App de macOS “Remote over SSH”
</div>

La app de la barra de menús de macOS puede controlar la misma configuración de extremo a extremo (comprobaciones de estado remotas, WebChat y reenvío de Voice Wake).

Runbook: [Acceso remoto en macOS](/es/platforms/mac/remote).

<div id="security-rules-remotevpn">
  ## Reglas de seguridad (remoto/VPN)
</div>

Versión corta: **mantén el Gateway solo en loopback** a menos que tengas claro que necesitas un bind.

* **Loopback + SSH/Tailscale Serve** es la opción predeterminada más segura (sin exposición pública).
* **Binds que no son de loopback** (`lan`/`tailnet`/`custom`, o `auto` cuando loopback no está disponible) deben usar tokens/contraseñas de autenticación.
* `gateway.remote.token` es **solo** para llamadas remotas de la CLI — **no** habilita autenticación local.
* `gateway.remote.tlsFingerprint` fija el certificado TLS remoto cuando se usa `wss://`.
* **Tailscale Serve** puede autenticar mediante cabeceras de identidad cuando `gateway.auth.allowTailscale: true`.
  Configúralo en `false` si prefieres usar tokens/contraseñas.
* Trata el control vía navegador como acceso de operador: solo mediante tailnet + emparejamiento deliberado de nodos.

Más detalles: [Seguridad](/es/gateway/security).