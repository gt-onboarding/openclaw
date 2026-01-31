---
title: Webchat
summary: "Cómo la app de macOS integra el Webchat del Gateway y cómo depurarlo"
read_when:
  - Depuración de la vista Webchat de macOS o del puerto de loopback
---

<div id="webchat-macos-app">
  # WebChat (app de macOS)
</div>

La app de la barra de menús de macOS incrusta la UI de WebChat como una vista nativa de SwiftUI. Se
conecta al Gateway y usa por defecto la **sesión principal** para el agente
seleccionado (con un selector de sesión para otras sesiones).

- **Modo local**: se conecta directamente al WebSocket local del Gateway.
- **Modo remoto**: reenvía el puerto de control del Gateway a través de SSH y usa ese
  túnel como canal de datos.

<div id="launch-debugging">
  ## Inicio y depuración
</div>

- Manual: menú Lobster → “Open Chat”.
- Apertura automática para pruebas:
  ```bash
  dist/OpenClaw.app/Contents/MacOS/OpenClaw --webchat
  ```
- Registros: `./scripts/clawlog.sh` (subsistema `bot.molt`, categoría `WebChatSwiftUI`).

<div id="how-its-wired">
  ## Cómo está conectado
</div>

- Plano de datos: métodos WS del Gateway `chat.history`, `chat.send`, `chat.abort`,
  `chat.inject` y eventos `chat`, `agent`, `presence`, `tick`, `health`.
- Sesión: de forma predeterminada usa la sesión principal (`main`, o `global` cuando scope es
  global). La UI puede cambiar entre sesiones.
- El proceso de onboarding utiliza una sesión dedicada para mantener separada la configuración de la primera ejecución.

<div id="security-surface">
  ## Superficie de ataque
</div>

- El modo remoto solo reenvía el puerto de control WebSocket del Gateway por SSH.

<div id="known-limitations">
  ## Limitaciones conocidas
</div>

- La UI está optimizada para sesiones de chat (no para actuar como un sandbox de navegador completo).