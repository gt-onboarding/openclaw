---
title: Webchat
summary: "Alojamiento estático de WebChat en loopback y uso de WS de Gateway para la UI de chat"
read_when:
  - Depuración o configuración del acceso a WebChat
---

<div id="webchat-gateway-websocket-ui">
  # WebChat (Gateway WebSocket UI)
</div>

Estado: la UI de chat basada en SwiftUI para macOS/iOS se comunica directamente con el WebSocket del Gateway.

<div id="what-it-is">
  ## Qué es
</div>

* Una UI de chat nativa para el Gateway (sin navegador incrustado y sin servidor estático local).
* Usa las mismas sesiones y reglas de enrutamiento que otros canales.
* Enrutamiento determinista: las respuestas siempre vuelven a WebChat.

<div id="quick-start">
  ## Inicio rápido
</div>

1. Inicia el Gateway.
2. Abre la UI de WebChat (app de macOS/iOS) o la pestaña de chat del Control UI.
3. Asegúrate de que la autenticación del Gateway esté correctamente configurada (obligatoria por defecto, incluso en loopback).

<div id="how-it-works-behavior">
  ## Cómo funciona (comportamiento)
</div>

* La UI se conecta al WebSocket del Gateway y utiliza `chat.history`, `chat.send` y `chat.inject`.
* `chat.inject` añade una nota del asistente directamente a la transcripción y la difunde a la UI (sin ejecución de agente).
* El historial siempre se obtiene desde el Gateway (sin monitorizar archivos locales).
* Si el Gateway no está disponible, WebChat es de solo lectura.

<div id="remote-use">
  ## Uso remoto
</div>

* El modo remoto encamina el WebSocket del Gateway a través de un túnel SSH/Tailscale.
* No es necesario ejecutar un servidor WebChat independiente.

<div id="configuration-reference-webchat">
  ## Referencia de configuración (WebChat)
</div>

Configuración completa: [Configuración](/es/gateway/configuration)

Opciones del canal:

* No hay un bloque dedicado `webchat.*`. WebChat usa el endpoint del Gateway + los ajustes de autenticación que se indican a continuación.

Opciones globales relacionadas:

* `gateway.port`, `gateway.bind`: host/puerto WebSocket.
* `gateway.auth.mode`, `gateway.auth.token`, `gateway.auth.password`: autenticación WebSocket.
* `gateway.remote.url`, `gateway.remote.token`, `gateway.remote.password`: Gateway remoto de destino.
* `session.*`: almacenamiento de sesión y valores predeterminados de las claves principales.