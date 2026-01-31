---
title: Anthropic
summary: "Usa Anthropic Claude mediante claves de API o setup-token en OpenClaw"
read_when:
  - Quieres usar modelos de Anthropic en OpenClaw
  - Quieres usar setup-token en lugar de claves de API
---

<div id="anthropic-claude">
  # Anthropic (Claude)
</div>

Anthropic desarrolla la familia de modelos **Claude** y proporciona acceso mediante una API.
En OpenClaw puedes autenticarte con una clave de API o con un **setup-token**.

<div id="option-a-anthropic-api-key">
  ## Opción A: Clave de API de Anthropic
</div>

**Ideal para:** acceso estándar a la API y facturación basada en el uso.
Crea tu clave de API en Anthropic Console.

<div id="cli-setup">
  ### Configuración de la CLI
</div>

```bash
openclaw onboard
# elige: clave de API de Anthropic

# or non-interactive
openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
```

<div id="config-snippet">
  ### Fragmento de configuración
</div>

```json5
{
  env: { ANTHROPIC_API_KEY: "sk-ant-..." },
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="prompt-caching-anthropic-api">
  ## Almacenamiento en caché de prompts (Anthropic API)
</div>

OpenClaw **no** modifica el TTL de caché predeterminado de Anthropic a menos que lo configures.
Esto es **solo mediante API**; la autenticación por suscripción no tiene en cuenta los ajustes de TTL.

Para establecer el TTL por modelo, usa `cacheControlTtl` en los parámetros (`params`) del modelo:

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": {
          params: { cacheControlTtl: "5m" } // o "1h"
        }
      }
    }
  }
}
```

OpenClaw incluye la *flag* beta `extended-cache-ttl-2025-04-11` para las solicitudes a la API de Anthropic; mantenla si anulas los encabezados del proveedor (consulta [/gateway/configuration](/es/gateway/configuration)).

<div id="option-b-claude-setup-token">
  ## Opción B: Claude setup-token
</div>

**Ideal para:** usar tu suscripción a Claude.

<div id="where-to-get-a-setup-token">
  ### Dónde conseguir un setup-token
</div>

Los setup-tokens se crean con la **CLI de Claude Code**, no en la consola de Anthropic. Puedes ejecutar esto en **cualquier máquina**:

```bash
claude setup-token
```

Pega el token en OpenClaw (asistente de configuración: **Anthropic token (pegar setup-token)**), o ejecútalo en el host del Gateway:

```bash
openclaw models auth setup-token --provider anthropic
```

Si generaste el token en otra máquina, pégalo aquí:

```bash
openclaw models auth paste-token --provider anthropic
```

<div id="cli-setup">
  ### Configuración de la CLI
</div>

```bash
# Pega un setup-token durante la incorporación
openclaw onboard --auth-choice setup-token
```

<div id="config-snippet">
  ### Fragmento de configuración
</div>

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="notes">
  ## Notas
</div>

* Genera el setup-token con `claude setup-token` y pégalo, o ejecuta `openclaw models auth setup-token` en el host del Gateway.
* Si ves “OAuth token refresh failed …” en una suscripción de Claude, vuelve a autenticar con un setup-token. Consulta [/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription](/es/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription).
* Los detalles de autenticación y las reglas de reutilización se encuentran en [/concepts/oauth](/es/concepts/oauth).

<div id="troubleshooting">
  ## Solución de problemas
</div>

**Errores 401 / token que de repente deja de ser válido**

* La autenticación de la suscripción de Claude puede caducar o ser revocada. Vuelve a ejecutar `claude setup-token`
  y pégalo en el **gateway host**.
* Si el inicio de sesión de la CLI de Claude está en otra máquina, usa
  `openclaw models auth paste-token --provider anthropic` en el gateway host.

**No se encontró ninguna clave de API para el proveedor &quot;anthropic&quot;**

* La autenticación es **por agente**. Los nuevos agentes no heredan las claves del agente principal.
* Vuelve a ejecutar el proceso de onboarding para ese agente, o pega un setup-token / clave de API en el
  gateway host, y luego verifica con `openclaw models status`.

**No se encontraron credenciales para el perfil `anthropic:default`**

* Ejecuta `openclaw models status` para ver qué perfil de autenticación está activo.
* Vuelve a ejecutar el onboarding, o pega un setup-token / clave de API para ese perfil.

**No hay ningún perfil de autenticación disponible (todos en cooldown/no disponibles)**

* Comprueba `openclaw models status --json` para ver `auth.unusableProfiles`.
* Añade otro perfil de Anthropic o espera al período de cooldown.

Más información: [/gateway/troubleshooting](/es/gateway/troubleshooting) y [/help/faq](/es/help/faq).