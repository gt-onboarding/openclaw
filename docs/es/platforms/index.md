---
title: Plataformas
summary: "Resumen de compatibilidad de plataformas (Gateway + aplicaciones complementarias)"
read_when:
  - Buscar compatibilidad de sistemas operativos o rutas de instalación
  - Decidir dónde ejecutar el Gateway
---

<div id="platforms">
  # Plataformas
</div>

El núcleo de OpenClaw está escrito en TypeScript. **Node es el entorno de ejecución recomendado**.
No se recomienda usar Bun para el Gateway (errores con WhatsApp/Telegram).

Existen aplicaciones complementarias para macOS (aplicación de la barra de menús) y nodos móviles (iOS/Android). Las aplicaciones complementarias para Windows y
Linux están previstas, pero el Gateway ya es totalmente compatible hoy.
También se prevén aplicaciones complementarias nativas para Windows; se recomienda ejecutar el Gateway a través de WSL2.

<div id="choose-your-os">
  ## Selecciona tu sistema operativo
</div>

* macOS: [macOS](/es/platforms/macos)
* iOS: [iOS](/es/platforms/ios)
* Android: [Android](/es/platforms/android)
* Windows: [Windows](/es/platforms/windows)
* Linux: [Linux](/es/platforms/linux)

<div id="vps-hosting">
  ## VPS y hosting
</div>

* Hub de VPS: [Alojamiento VPS](/es/vps)
* Fly.io: [Fly.io](/es/platforms/fly)
* Hetzner (Docker): [Hetzner](/es/platforms/hetzner)
* GCP (Compute Engine): [GCP](/es/platforms/gcp)
* exe.dev (VM + proxy HTTPS): [exe.dev](/es/platforms/exe-dev)

<div id="common-links">
  ## Enlaces habituales
</div>

* Guía de instalación: [Primeros pasos](/es/start/getting-started)
* Runbook de Gateway: [Gateway](/es/gateway)
* Configuración de Gateway: [Configuración](/es/gateway/configuration)
* Estado del servicio: `openclaw gateway status`

<div id="gateway-service-install-cli">
  ## Instalación del servicio Gateway (CLI)
</div>

Usa una de estas opciones (todas admitidas):

* Asistente (recomendado): `openclaw onboard --install-daemon`
* Directo: `openclaw gateway install`
* Flujo de configuración: `openclaw configure` → selecciona **Gateway service**
* Reparación/migración: `openclaw doctor` (ofrece instalar o reparar el servicio)

El destino del servicio depende del sistema operativo:

* macOS: LaunchAgent (`bot.molt.gateway` o `bot.molt.<profile>`; heredado `com.openclaw.*`)
* Linux/WSL2: servicio de usuario systemd (`openclaw-gateway[-<profile>].service`)