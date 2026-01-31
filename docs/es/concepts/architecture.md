---
title: Arquitectura
summary: "Arquitectura del Gateway WebSocket, componentes y flujos de clientes"
read_when:
  - Al trabajar en el protocolo del Gateway, en clientes o en transportes
---

<div id="gateway-architecture">
  # Arquitectura de Gateway
</div>

Última actualización: 2026-01-22

<div id="overview">
  ## Descripción general
</div>

* Un único **Gateway** persistente posee todos los canales de mensajería (WhatsApp mediante
  Baileys, Telegram mediante grammY, Slack, Discord, Signal, iMessage, WebChat).
* Los clientes del plano de control (app de macOS, CLI, UI web, automatizaciones) se conectan al
  Gateway mediante **WebSocket** en el host de enlace configurado (por defecto
  `127.0.0.1:18789`).
* Los **nodos** (macOS/iOS/Android/headless) también se conectan mediante **WebSocket**, pero
  declaran `role: node` con capacidades/comandos explícitos.
* Un Gateway por host; es el único lugar donde se abre una sesión de WhatsApp.
* Un **host de canvas** (por defecto `18793`) sirve HTML editable por los agentes y A2UI.

<div id="components-and-flows">
  ## Componentes y flujos
</div>

<div id="gateway-daemon">
  ### Gateway (daemon)
</div>

* Mantiene las conexiones con proveedores.
* Expone una API WS tipada (peticiones, respuestas, eventos enviados por el servidor).
* Valida los frames entrantes contra JSON Schema.
* Emite eventos como `agent`, `chat`, `presence`, `health`, `heartbeat`, `cron`.

<div id="clients-mac-app-cli-web-admin">
  ### Clientes (aplicación de macOS / CLI / administración web)
</div>

* Una conexión WS por cliente.
* Enviar solicitudes (`health`, `status`, `send`, `agent`, `system-presence`).
* Suscribirse a eventos (`tick`, `agent`, `presence`, `shutdown`).

<div id="nodes-macos-ios-android-headless">
  ### Nodos (macOS / iOS / Android / headless)
</div>

* Se conectan al **mismo servidor WS** con `role: node`.
* Proporcionan una identidad de dispositivo en `connect`; el emparejamiento es **por dispositivo** (rol `node`) y
  la aprobación se guarda en el almacén de emparejamiento de dispositivos.
* Exponen comandos como `canvas.*`, `camera.*`, `screen.record`, `location.get`.

Detalles del protocolo:

* [Protocolo de Gateway](/es/gateway/protocol)

<div id="webchat">
  ### WebChat
</div>

* UI estática que usa la API WS del Gateway para el historial de chat y las operaciones de envío.
* En entornos remotos, se conecta a través del mismo túnel SSH/Tailscale que otros clientes.

<div id="connection-lifecycle-single-client">
  ## Ciclo de vida de la conexión (un solo cliente)
</div>

```
Client                    Gateway
  |                          |
  |---- req:connect -------->|
  |<------ res (ok) ---------|   (or res error + close)
  |   (payload=hello-ok carries snapshot: presence + health)
  |                          |
  |<------ event:presence ---|
  |<------ event:tick -------|
  |                          |
  |------- req:agent ------->|
  |<------ res:agent --------|   (ack: {runId,status:"accepted"})
  |<------ event:agent ------|   (streaming)
  |<------ res:agent --------|   (final: {runId,status,summary})
  |                          |
```

<div id="wire-protocol-summary">
  ## Protocolo de comunicación (resumen)
</div>

* Transporte: WebSocket, tramas de texto con payloads JSON.
* La primera trama **debe** ser `connect`.
* Después del handshake:
  * Peticiones: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
  * Eventos: `{type:"event", event, payload, seq?, stateVersion?}`
* Si `OPENCLAW_GATEWAY_TOKEN` (o `--token`) está configurado, `connect.params.auth.token`
  debe coincidir o el socket se cerrará.
* Se requieren claves de idempotencia para métodos con efectos secundarios (`send`, `agent`) para
  reintentar de forma segura; el servidor mantiene una caché de desduplicación de corta vida.
* Los nodos deben incluir `role: "node"` más capacidades/comandos/permisos en `connect`.

<div id="pairing-local-trust">
  ## Emparejamiento + confianza local
</div>

* Todos los clientes WS (operadores + nodos) incluyen una **identidad de dispositivo** en `connect`.
* Los nuevos ID de dispositivo requieren aprobación de emparejamiento; el Gateway emite un **token de dispositivo**
  para conexiones posteriores.
* Las conexiones **locales** (loopback o la propia dirección tailnet del host del Gateway) se pueden
  aprobar automáticamente para mantener una experiencia de usuario (UX) fluida en el mismo host.
* Las conexiones **no locales** deben firmar el nonce `connect.challenge` y requieren
  aprobación explícita.
* La autenticación del Gateway (`gateway.auth.*`) sigue aplicándose a **todas** las conexiones, locales o
  remotas.

Detalles: [Protocolo del Gateway](/es/gateway/protocol), [Emparejamiento](/es/start/pairing),
[Seguridad](/es/gateway/security).

<div id="protocol-typing-and-codegen">
  ## Tipado de protocolos y generación de código
</div>

* Los esquemas de TypeBox definen el protocolo.
* A partir de esos esquemas se genera el JSON Schema.
* Los modelos Swift se generan a partir del JSON Schema.

<div id="remote-access">
  ## Acceso remoto
</div>

* Recomendado: Tailscale o VPN.
* Alternativa: túnel SSH
  ```bash
  ssh -N -L 18789:127.0.0.1:18789 user@host
  ```
* El mismo handshake y el mismo token de autenticación se aplican a través del túnel.
* TLS y, opcionalmente, pinning de certificados se pueden habilitar para WS en configuraciones remotas.

<div id="operations-snapshot">
  ## Instantánea de operaciones
</div>

* Inicio: `openclaw gateway` (en primer plano, escribe registros en stdout).
* Comprobación de estado: `health` vía WS (también incluido en `hello-ok`).
* Supervisión: launchd/systemd para reinicio automático.

<div id="invariants">
  ## Invariantes
</div>

* Exactamente un Gateway controla una única sesión de Baileys por host.
* El handshake es obligatorio; cualquier primer frame que no sea JSON o `connect` provoca un cierre forzado.
* Los eventos no se reproducen; los clientes deben actualizarse cuando detecten huecos.