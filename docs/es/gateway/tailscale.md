---
title: Tailscale
summary: "Integración de Tailscale Serve/Funnel para el panel de control del Gateway"
read_when:
  - Al exponer la Control UI del Gateway fuera de localhost
  - Al automatizar el acceso a la tailnet o al panel público
---

<div id="tailscale-gateway-dashboard">
  # Tailscale (panel de control del Gateway)
</div>

OpenClaw puede configurar automáticamente Tailscale **Serve** (tailnet) o **Funnel** (público) para el
panel de control del Gateway y el puerto WebSocket. Esto mantiene el Gateway vinculado a la interfaz de loopback mientras
Tailscale proporciona HTTPS, enrutamiento y (para Serve) cabeceras de identidad.

<div id="modes">
  ## Modos
</div>

- `serve`: Serve solo dentro de Tailnet mediante `tailscale serve`. El Gateway permanece en `127.0.0.1`.
- `funnel`: HTTPS público mediante `tailscale funnel`. OpenClaw requiere una contraseña compartida.
- `off`: Valor predeterminado (sin automatización de Tailscale).

<div id="auth">
  ## Autenticación
</div>

Configura `gateway.auth.mode` para controlar el handshake:

- `token` (valor predeterminado cuando `OPENCLAW_GATEWAY_TOKEN` está definido)
- `password` (secreto compartido mediante `OPENCLAW_GATEWAY_PASSWORD` o la configuración)

Cuando `tailscale.mode = "serve"` y `gateway.auth.allowTailscale` es `true`,
las solicitudes válidas del proxy Serve pueden autenticarse mediante cabeceras de identidad de Tailscale
(`tailscale-user-login`) sin proporcionar un token/contraseña. OpenClaw verifica
la identidad resolviendo la dirección `x-forwarded-for` a través del demonio local de Tailscale
(`tailscale whois`) y comparándola con la cabecera antes de aceptarla.
OpenClaw solo trata una solicitud como Serve cuando llega desde loopback con
las cabeceras `x-forwarded-for`, `x-forwarded-proto` y `x-forwarded-host`
de Tailscale.
Para exigir credenciales explícitas, establece `gateway.auth.allowTailscale: false` o
fuerza que `gateway.auth.mode` sea `"password"`.

<div id="config-examples">
  ## Ejemplos de configuración
</div>

<div id="tailnet-only-serve">
  ### Solo Tailnet (Serve)
</div>

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" }
  }
}
```

Abre: `https://<magicdns>/` (o la `gateway.controlUi.basePath` que hayas configurado)


<div id="tailnet-only-bind-to-tailnet-ip">
  ### Solo Tailnet (vincular a la IP de Tailnet)
</div>

Usa esto cuando quieras que el Gateway escuche directamente en la IP de Tailnet (sin Serve ni Funnel).

```json5
{
  gateway: {
    bind: "tailnet",
    auth: { mode: "token", token: "your-token" }
  }
}
```

Conéctate desde otro dispositivo de tu Tailnet:

* Control UI: `http://<tailscale-ip>:18789/`
* WebSocket: `ws://<tailscale-ip>:18789`

Nota: la dirección de loopback (`http://127.0.0.1:18789`) **no** funcionará en este modo.


<div id="public-internet-funnel-shared-password">
  ### Internet pública (Funnel + contraseña compartida)
</div>

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password", password: "replace-me" }
  }
}
```

Es preferible usar `OPENCLAW_GATEWAY_PASSWORD` en lugar de guardar la contraseña en disco.


<div id="cli-examples">
  ## Ejemplos de CLI
</div>

```bash
openclaw gateway --tailscale serve
openclaw gateway --tailscale funnel --auth password
```


<div id="notes">
  ## Notas
</div>

- Tailscale Serve/Funnel requiere que la CLI `tailscale` esté instalada y con sesión iniciada.
- `tailscale.mode: "funnel"` se niega a iniciarse a menos que el modo de autenticación sea `password` para evitar la exposición pública.
- Configura `gateway.tailscale.resetOnExit` si quieres que OpenClaw revierta la configuración de
  `tailscale serve` o `tailscale funnel` al apagarse.
- `gateway.bind: "tailnet"` es un enlace directo a Tailnet (sin HTTPS, sin Serve/Funnel).
- `gateway.bind: "auto"` prefiere la interfaz de loopback; usa `tailnet` si solo quieres Tailnet.
- Serve/Funnel solo exponen la **Control UI del Gateway + WS**. Los nodos se conectan mediante
  el mismo endpoint de WS del Gateway, por lo que Serve puede funcionar para el acceso de nodos.

<div id="browser-control-remote-gateway-local-browser">
  ## Control del navegador (Gateway remoto + navegador local)
</div>

Si ejecutas el Gateway en una máquina pero quieres controlar un navegador en otra máquina,
ejecuta un **host de nodo** en la máquina del navegador y mantén ambas en la misma tailnet.
El Gateway hará de proxy para las acciones del navegador hacia el nodo; no se necesita un servidor de control separado ni una URL de Serve.

Evita usar Funnel para el control del navegador; trata el emparejamiento del nodo igual que el acceso de operador.

<div id="tailscale-prerequisites-limits">
  ## Requisitos previos y limitaciones de Tailscale
</div>

- Serve requiere HTTPS habilitado para tu tailnet; la CLI te avisará si no lo está.
- Serve inyecta cabeceras de identidad de Tailscale; Funnel no.
- Funnel requiere Tailscale v1.38.3+, MagicDNS, HTTPS habilitado y un atributo de nodo funnel.
- Funnel solo admite los puertos `443`, `8443` y `10000` sobre TLS.
- Funnel en macOS requiere la variante de app de Tailscale de código abierto.

<div id="learn-more">
  ## Más información
</div>

- Información general de Tailscale Serve: https://tailscale.com/kb/1312/serve
- Comando `tailscale serve`: https://tailscale.com/kb/1242/tailscale-serve
- Información general de Tailscale Funnel: https://tailscale.com/kb/1223/tailscale-funnel
- Comando `tailscale funnel`: https://tailscale.com/kb/1311/tailscale-funnel