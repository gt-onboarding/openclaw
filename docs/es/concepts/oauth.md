---
title: OAuth
summary: "OAuth en OpenClaw: intercambio de tokens, almacenamiento y patrones multicuenta"
read_when:
  - Quieres entender OAuth en OpenClaw de extremo a extremo
  - Te encuentras con problemas de invalidación de tokens o cierre de sesión
  - Quieres flujos de autenticación basados en setup-token o en OAuth
  - Quieres múltiples cuentas o enrutamiento por perfiles
---

<div id="oauth">
  # OAuth
</div>

OpenClaw admite «autenticación por suscripción» mediante OAuth para los proveedores que la ofrecen (en particular **OpenAI Codex (ChatGPT OAuth)**). Para suscripciones de Anthropic, usa el flujo **setup-token**. Esta página explica:

* cómo funciona el **intercambio de tokens** OAuth (PKCE)
* dónde se **almacenan** los tokens (y por qué)
* cómo gestionar **varias cuentas** (perfiles + anulaciones por sesión)

OpenClaw también admite **complementos de proveedor** que incluyen sus propios flujos de OAuth o de clave de API.
Ejecútalos mediante:

```bash
openclaw models auth login --provider <id>
```

<div id="the-token-sink-why-it-exists">
  ## El sumidero de tokens (por qué existe)
</div>

Los proveedores OAuth suelen emitir un **nuevo refresh token** durante los flujos de inicio de sesión/renovación. Algunos proveedores (o clientes OAuth) pueden invalidar los refresh tokens antiguos cuando se emite uno nuevo para el mismo usuario/aplicación.

Síntoma práctico:

* inicias sesión mediante OpenClaw *y* mediante Claude Code / Codex CLI → más tarde, en uno de ellos se cierra la sesión de forma aparentemente aleatoria

Para reducir eso, OpenClaw trata `auth-profiles.json` como un **sumidero de tokens**:

* el runtime lee las credenciales desde **un solo lugar**
* podemos mantener varios perfiles y enrutarlos de forma determinista

<div id="storage-where-tokens-live">
  ## Almacenamiento (dónde residen los tokens)
</div>

Los secretos se almacenan **por agente**:

* Perfiles de autenticación (OAuth + claves de API): `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
* Caché en tiempo de ejecución (gestionada automáticamente; no la edites): `~/.openclaw/agents/<agentId>/agent/auth.json`

Archivo heredado solo para importación (todavía admitido, pero no es el almacén principal):

* `~/.openclaw/credentials/oauth.json` (importado en `auth-profiles.json` en el primer uso)

Todo lo anterior también respeta `$OPENCLAW_STATE_DIR` (para anular el directorio de estado). Referencia completa: [/gateway/configuration](/es/gateway/configuration#auth-storage-oauth--api-keys)

<div id="anthropic-setup-token-subscription-auth">
  ## Anthropic setup-token (autenticación de suscripción)
</div>

Ejecuta `claude setup-token` en cualquier equipo y luego pégalo en OpenClaw:

```bash
openclaw models auth setup-token --provider anthropic
```

Si generaste el token en otra parte, pégalo manualmente:

```bash
openclaw models auth paste-token --provider anthropic
```

Verificar:

```bash
openclaw models status
```

<div id="oauth-exchange-how-login-works">
  ## Intercambio OAuth (cómo funciona el inicio de sesión)
</div>

Los flujos de inicio de sesión interactivos de OpenClaw se implementan en `@mariozechner/pi-ai` y se integran en los asistentes y comandos.

<div id="anthropic-claude-promax-setup-token">
  ### Anthropic (Claude Pro/Max) setup-token
</div>

Flujo:

1. ejecuta `claude setup-token`
2. pega el token en OpenClaw
3. guárdalo como un perfil de autenticación con token (sin renovación)

La ruta del asistente es `openclaw onboard` → opción de autenticación `setup-token` (Anthropic).

<div id="openai-codex-chatgpt-oauth">
  ### OpenAI Codex (ChatGPT OAuth)
</div>

Estructura del flujo (PKCE):

1. generar verificador/desafío PKCE + `state` aleatorio
2. abrir `https://auth.openai.com/oauth/authorize?...`
3. intentar capturar el callback en `http://127.0.0.1:1455/auth/callback`
4. si el callback no se puede enlazar (o estás remoto/headless), pega la URL/código de redirección
5. intercambiar el código en `https://auth.openai.com/oauth/token`
6. extraer `accountId` del token de acceso y almacenar `{ access, refresh, expires, accountId }`

La ruta del asistente es `openclaw onboard` → opción de autenticación `openai-codex`.

<div id="refresh-expiry">
  ## Renovación y vencimiento
</div>

Los perfiles almacenan una marca de tiempo `expires`.

En tiempo de ejecución:

* si `expires` está en el futuro → se usa el token de acceso almacenado
* si ya venció → se renueva (bajo un bloqueo de archivo) y se sobrescriben las credenciales almacenadas

La renovación es automática; por lo general no tienes que gestionar los tokens manualmente.

<div id="multiple-accounts-profiles-routing">
  ## Múltiples cuentas (perfiles) + enrutamiento
</div>

Dos patrones posibles:

<div id="1-preferred-separate-agents">
  ### 1) Recomendado: agentes separados
</div>

Si quieres que lo «personal» y lo «laboral» nunca interactúen entre sí, usa agentes aislados (sesiones, credenciales y espacio de trabajo separados):

```bash
openclaw agents add work
openclaw agents add personal
```

Luego configura la autenticación por cada agente (asistente) y dirige los chats al agente adecuado.

<div id="2-advanced-multiple-profiles-in-one-agent">
  ### 2) Avanzado: múltiples perfiles en un solo agente
</div>

`auth-profiles.json` admite varios IDs de perfil para el mismo proveedor.

Selecciona qué perfil se usa:

* globalmente mediante el orden en la configuración (`auth.order`)
* por sesión mediante `/model ...@<profileId>`

Ejemplo (override de sesión):

* `/model Opus@anthropic:work`

Cómo ver qué IDs de perfil existen:

* `openclaw channels list --json` (muestra `auth[]`)

Documentación relacionada:

* [/concepts/model-failover](/es/concepts/model-failover) (reglas de rotación + periodo de enfriamiento)
* [/tools/slash-commands](/es/tools/slash-commands) (interfaz de comandos)