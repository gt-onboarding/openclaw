---
title: Openai
summary: "Utiliza OpenAI mediante claves de API o suscripción a Codex en OpenClaw"
read_when:
  - Quieres utilizar modelos de OpenAI en OpenClaw
  - Quieres autenticación con suscripción a Codex en lugar de claves de API
---

<div id="openai">
  # OpenAI
</div>

OpenAI proporciona APIs para desarrolladores de modelos GPT. Codex admite el **inicio de sesión con ChatGPT** para acceso por suscripción o el inicio de sesión mediante **clave de API** para acceso basado en el uso. Codex Cloud requiere iniciar sesión con ChatGPT.

<div id="option-a-openai-api-key-openai-platform">
  ## Opción A: clave de API de OpenAI (OpenAI Platform)
</div>

**Mejor para:** acceso directo a la API y facturación según el uso.
Obtén tu clave de API en el panel de control de OpenAI.

<div id="cli-setup">
  ### Configuración de la CLI
</div>

```bash
openclaw onboard --auth-choice openai-api-key
# o modo no interactivo
openclaw onboard --openai-api-key "$OPENAI_API_KEY"
```

<div id="config-snippet">
  ### Fragmento de configuración
</div>

```json5
{
  env: { OPENAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "openai/gpt-5.2" } } }
}
```

<div id="option-b-openai-code-codex-subscription">
  ## Opción B: suscripción a OpenAI Code (Codex)
</div>

**Mejor para:** usar acceso mediante suscripción a ChatGPT/Codex en lugar de una clave de API.
Codex en la nube requiere iniciar sesión con ChatGPT, mientras que la CLI de Codex admite inicio de sesión con ChatGPT o con clave de API.

<div id="cli-setup">
  ### Configuración de la CLI
</div>

```bash
# Ejecuta OAuth de Codex en el asistente
openclaw onboard --auth-choice openai-codex

# O ejecuta OAuth directamente
openclaw models auth login --provider openai-codex
```

### Ejemplo de configuración

```json5
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.2" } } }
}
```

<div id="notes">
  ## Notas
</div>

* Las referencias de modelos siempre usan `provider/model` (ver [/concepts/models](/es/concepts/models)).
* Los detalles de autenticación y las reglas de reutilización están en [/concepts/oauth](/es/concepts/oauth).