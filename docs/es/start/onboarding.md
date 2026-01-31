---
title: Onboarding
summary: "Flujo de onboarding en la primera ejecución para OpenClaw (app de macOS)"
read_when:
  - Diseñar el asistente de onboarding de macOS
  - Implementar la configuración de autenticación o identidad
---

<div id="onboarding-macos-app">
  # Onboarding (app de macOS)
</div>

Este documento describe el flujo de onboarding **actual** en el primer inicio. El objetivo es una experiencia fluida del “día 0”: elegir dónde se ejecuta el Gateway, configurar la autenticación, ejecutar el asistente y dejar que el agente se auto‑configure.

<div id="page-order-current">
  ## Orden de las páginas (actual)
</div>

1. Bienvenida + aviso de seguridad
2. **Selección de Gateway** (Local / Remoto / Configurar más tarde)
3. **Autenticación (Anthropic OAuth)** — solo en local
4. **Asistente de configuración** (controlado por el Gateway)
5. **Permisos** (solicitudes TCC)
6. **CLI** (opcional)
7. **Chat de incorporación** (sesión dedicada)
8. Listo

<div id="1-local-vs-remote">
  ## 1) Local vs Remote
</div>

¿Dónde se ejecuta el **Gateway**?

* **Local (este Mac):** el onboarding puede ejecutar flujos de OAuth y escribir credenciales localmente.
* **Remoto (vía SSH/Tailnet):** el onboarding **no** ejecuta OAuth localmente; las credenciales deben existir en el host del Gateway.
* **Configurar más tarde:** omites la configuración y dejas la app sin configurar.

Consejo sobre autenticación del Gateway:

* El asistente ahora genera un **token** incluso para loopback, por lo que los clientes WS locales deben autenticarse.
* Si desactivas la autenticación, cualquier proceso local puede conectarse; úsalo solo en máquinas totalmente confiables.
* Usa un **token** para acceso entre varias máquinas o vinculaciones que no sean de loopback.

<div id="2-local-only-auth-anthropic-oauth">
  ## 2) Autenticación solo en local (Anthropic OAuth)
</div>

La app de macOS admite Anthropic OAuth (Claude Pro/Max). El flujo es el siguiente:

* Abre el navegador para OAuth (PKCE)
* Pide al usuario que pegue el valor `code#state`
* Escribe las credenciales en `~/.openclaw/credentials/oauth.json`

Por ahora, otros proveedores (OpenAI, API personalizadas) se configuran mediante variables de entorno
o archivos de configuración.

<div id="3-setup-wizard-gatewaydriven">
  ## 3) Asistente de configuración (controlado por el Gateway)
</div>

La aplicación puede ejecutar el mismo asistente de configuración que la CLI. Esto mantiene el proceso de onboarding alineado con el comportamiento del Gateway y evita duplicar lógica en SwiftUI.

<div id="4-permissions">
  ## 4) Permisos
</div>

El proceso de onboarding solicita los permisos TCC necesarios para:

* Notificaciones
* Accesibilidad
* Grabación de pantalla
* Micrófono y reconocimiento de voz
* Automatización (AppleScript)

<div id="5-cli-optional">
  ## 5) CLI (opcional)
</div>

La aplicación puede instalar la CLI global `openclaw` mediante npm/pnpm para que los flujos de trabajo en terminal y las tareas de launchd funcionen desde el primer momento.

<div id="6-onboarding-chat-dedicated-session">
  ## 6) Chat de incorporación (sesión dedicada)
</div>

Después de la configuración, la aplicación abre una sesión de chat de incorporación dedicada para que el agente pueda
presentarse y guiar los siguientes pasos. Esto mantiene la guía del primer uso separada
de tu conversación normal.

<div id="agent-bootstrap-ritual">
  ## Ritual de arranque del Agente
</div>

En la primera ejecución del agente, OpenClaw inicializa un espacio de trabajo (por defecto `~/.openclaw/workspace`):

* Genera `AGENTS.md`, `BOOTSTRAP.md`, `IDENTITY.md`, `USER.md`
* Ejecuta un breve ritual de preguntas y respuestas (una pregunta a la vez)
* Escribe la identidad y las preferencias en `IDENTITY.md`, `USER.md`, `SOUL.md`
* Elimina `BOOTSTRAP.md` al finalizar, de modo que solo se ejecute una vez

<div id="optional-gmail-hooks-manual">
  ## Opcional: hooks de Gmail (manual)
</div>

La configuración de Pub/Sub de Gmail es actualmente un paso manual. Utiliza:

```bash
openclaw webhooks gmail setup --account you@gmail.com
```

Consulta [/automation/gmail-pubsub](/es/automation/gmail-pubsub) para obtener más información.

<div id="remote-mode-notes">
  ## Notas sobre el modo remoto
</div>

Cuando el Gateway se ejecuta en otra máquina, las credenciales y los archivos del espacio de trabajo se almacenan
**en ese host**. Si necesitas OAuth en modo remoto, crea:

* `~/.openclaw/credentials/oauth.json`
* `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

en el host del Gateway.