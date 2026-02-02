---
title: Panel de control
summary: "Acceso y autenticación del panel del Gateway (Control UI)"
read_when:
  - Cambiar la autenticación o los modos de exposición del panel
---

<div id="dashboard-control-ui">
  # Panel (Control UI)
</div>

El panel del Gateway es la Control UI en el navegador, servida en `/` de forma predeterminada
(se puede cambiar con `gateway.controlUi.basePath`).

Acceso rápido (Gateway local):

* http://127.0.0.1:18789/ (o http://localhost:18789/)

Referencias clave:

* [Control UI](/es/web/control-ui) para detalles de uso y capacidades de la UI.
* [Tailscale](/es/gateway/tailscale) para automatización con Serve/Funnel.
* [Superficies web](/es/web) para modos de enlace y notas de seguridad.

La autenticación se aplica en el handshake de WebSocket mediante `connect.params.auth`
(token o contraseña). Consulta `gateway.auth` en la [configuración del Gateway](/es/gateway/configuration).

Nota de seguridad: la Control UI es una **superficie de administración** (chat, configuración, aprobaciones de ejecución).
No la expongas públicamente. La UI almacena el token en `localStorage` después de la primera carga.
Prioriza localhost, Tailscale Serve o un túnel SSH.

<div id="fast-path-recommended">
  ## Ruta rápida (recomendada)
</div>

* Después del onboarding, la CLI ahora abre automáticamente el dashboard con tu token y muestra el mismo enlace con el token incrustado.
* Vuelve a abrirlo en cualquier momento: `openclaw dashboard` (copia el enlace, abre el navegador si es posible, muestra una sugerencia sobre SSH si está en modo sin interfaz gráfica/headless).
* El token permanece local (solo como parámetro de consulta/query param); la UI lo elimina después de la primera carga y lo almacena en localStorage.

<div id="token-basics-local-vs-remote">
  ## Conceptos básicos de los tokens (local vs remoto)
</div>

* **Localhost**: abre `http://127.0.0.1:18789/`. Si ves “unauthorized”, ejecuta `openclaw dashboard` y usa el enlace con token (`?token=...`).
* **Origen del token**: `gateway.auth.token` (o `OPENCLAW_GATEWAY_TOKEN`); la UI lo almacena tras la primera carga.
* **Si no es localhost**: usa Tailscale Serve (sin necesidad de token si `gateway.auth.allowTailscale: true`), realiza un bind del tailnet con un token o usa un túnel SSH. Consulta [Superficies web](/es/web).

<div id="if-you-see-unauthorized-1008">
  ## Si ves “unauthorized” / 1008
</div>

* Ejecuta `openclaw dashboard` para obtener un nuevo enlace con token.
* Asegúrate de que el Gateway esté accesible (local: `openclaw status`; remoto: túnel SSH `ssh -N -L 18789:127.0.0.1:18789 user@host` y luego abre `http://127.0.0.1:18789/?token=...`).
* En la configuración del dashboard, pega el mismo token que configuraste en `gateway.auth.token` (o `OPENCLAW_GATEWAY_TOKEN`).