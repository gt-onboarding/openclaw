---
title: VPS
summary: "Centro de hosting VPS para OpenClaw (Oracle/Fly/Hetzner/GCP/exe.dev)"
read_when:
  - Quieres ejecutar el Gateway en la nube
  - Necesitas una visión general rápida de las guías de VPS/hosting
---

<div id="vps-hosting">
  # Alojamiento en VPS
</div>

Este hub enlaza con las guías de VPS/hosting admitidos y explica, a alto nivel, cómo funcionan los despliegues en la nube.

<div id="pick-a-provider">
  ## Elige un proveedor
</div>

* **Railway** (configuración con un solo clic + en el navegador): [Railway](/es/railway)
* **Northflank** (configuración con un solo clic + en el navegador): [Northflank](/es/northflank)
* **Oracle Cloud (Always Free)**: [Oracle](/es/platforms/oracle) — $0/mes (Always Free, ARM; la capacidad y el registro pueden ser algo problemáticos)
* **Fly.io**: [Fly.io](/es/platforms/fly)
* **Hetzner (Docker)**: [Hetzner](/es/platforms/hetzner)
* **GCP (Compute Engine)**: [GCP](/es/platforms/gcp)
* **exe.dev** (VM + proxy HTTPS): [exe.dev](/es/platforms/exe-dev)
* **AWS (EC2/Lightsail/free tier)**: también funciona bien. Videoguía:
  https://x.com/techfrenAJ/status/2014934471095812547

<div id="how-cloud-setups-work">
  ## Cómo funcionan las implementaciones en la nube
</div>

* El **Gateway se ejecuta en el VPS** y gestiona el estado + el espacio de trabajo.
* Te conectas desde tu portátil/teléfono a través del **Control UI** o **Tailscale/SSH**.
* Trata el VPS como la fuente de verdad y **realiza copias de seguridad** del estado + del espacio de trabajo.
* Configuración segura predeterminada: mantén el Gateway en loopback y accede a él mediante un túnel SSH o Tailscale Serve.
  Si lo vinculas a `lan`/`tailnet`, requiere `gateway.auth.token` o `gateway.auth.password`.

Acceso remoto: [Gateway remoto](/es/gateway/remote)\
Centro de plataformas: [Plataformas](/es/platforms)

<div id="using-nodes-with-a-vps">
  ## Uso de nodos con un VPS
</div>

Puedes mantener el Gateway en la nube y emparejar **nodos** en tus dispositivos locales
(Mac/iOS/Android/sin interfaz gráfica). Los nodos proporcionan capacidades locales de pantalla/cámara/lienzo y `system.run`,
mientras que el Gateway permanece en la nube.

Documentación: [Nodos](/es/nodes), [CLI de nodos](/es/cli/nodes)