---
title: Web
summary: "Interfaces web del Gateway: Control UI, modos de bind y seguridad"
read_when:
  - Quieres acceder al Gateway a través de Tailscale
  - Quieres usar la Control UI en el navegador y editar la configuración
---

<div id="web-gateway">
  # Web (Gateway)
</div>

El Gateway sirve una pequeña **Control UI en el navegador** (Vite + Lit) desde el mismo puerto que el WebSocket del Gateway:

* por defecto: `http://<host>:18789/`
* prefijo opcional: configura `gateway.controlUi.basePath` (p. ej. `/openclaw`)

Las funcionalidades están en [Control UI](/es/web/control-ui).
Esta página se centra en los modos de binding, la seguridad y las superficies expuestas en la web.

<div id="webhooks">
  ## Webhooks
</div>

Cuando `hooks.enabled=true`, el Gateway también expone un pequeño endpoint de webhook en el mismo servidor HTTP.
Consulta la [configuración del Gateway](/es/gateway/configuration) → `hooks` para autenticación y payloads.

<div id="config-default-on">
  ## Configuración (activado de forma predeterminada)
</div>

La Control UI está **habilitada de forma predeterminada** cuando los recursos están presentes (`dist/control-ui`).
Puedes controlarla a través de la configuración:

```json5
{
  gateway: {
    controlUi: { enabled: true, basePath: "/openclaw" } // basePath opcional
  }
}
```

<div id="tailscale-access">
  ## Acceso con Tailscale
</div>

<div id="integrated-serve-recommended">
  ### Serve integrado (recomendado)
</div>

Mantén el Gateway en loopback y deja que Tailscale Serve sirva de proxy:

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" }
  }
}
```

A continuación, inicia el Gateway:

```bash
openclaw gateway
```

Abre:

* `https://<magicdns>/` (o la `gateway.controlUi.basePath` que tengas configurada)

<div id="tailnet-bind-token">
  ### Vincular Tailnet + token
</div>

```json5
{
  gateway: {
    bind: "tailnet",
    controlUi: { enabled: true },
    auth: { mode: "token", token: "your-token" }
  }
}
```

Luego inicia el Gateway (se requiere un token para realizar binds en interfaces que no sean loopback):

```bash
openclaw gateway
```

Abre:

* `http://<tailscale-ip>:18789/` (o tu `gateway.controlUi.basePath` configurado)

<div id="public-internet-funnel">
  ### Internet pública (Funnel)
</div>

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password" } // o OPENCLAW_GATEWAY_PASSWORD
  }
}
```

<div id="security-notes">
  ## Notas de seguridad
</div>

* La autenticación del Gateway es obligatoria de forma predeterminada (token/contraseña o cabeceras de identidad de Tailscale).
* Los binds que no sean de loopback aún **requieren** un token/contraseña compartido (`gateway.auth` o variable de entorno).
* El asistente genera un token del Gateway de forma predeterminada (incluso en loopback).
* La UI envía `connect.params.auth.token` o `connect.params.auth.password`.
* Con Serve, las cabeceras de identidad de Tailscale pueden cumplir los requisitos de autenticación cuando
  `gateway.auth.allowTailscale` es `true` (no se requiere token/contraseña). Configura
  `gateway.auth.allowTailscale: false` para exigir credenciales explícitas. Consulta
  [Tailscale](/es/gateway/tailscale) y [Seguridad](/es/gateway/security).
* `gateway.tailscale.mode: "funnel"` requiere `gateway.auth.mode: "password"` (contraseña compartida).

<div id="building-the-ui">
  ## Compilación de la UI
</div>

El Gateway sirve archivos estáticos desde `dist/control-ui`. Compílalos con:

```bash
pnpm ui:build # instala automáticamente las dependencias de la UI en la primera ejecución
```
