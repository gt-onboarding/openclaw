---
title: Modelos
summary: "CLI de modelos: listar, establecer, alias, fallbacks, escanear, estado"
read_when:
  - Agregar o modificar la CLI de modelos (models list/set/scan/aliases/fallbacks)
  - Cambiar el comportamiento de fallbacks de modelos o la experiencia de selección de modelos (UX)
  - Actualizar las sondas de escaneo de modelos (tools/images)
---

<div id="models-cli">
  # CLI de modelos
</div>

Consulta [/concepts/model-failover](/es/concepts/model-failover) para la rotación de perfiles de autenticación, los períodos de enfriamiento (cooldowns) y cómo interactúan con los mecanismos de reserva.
Descripción general rápida de los proveedores + ejemplos: [/concepts/model-providers](/es/concepts/model-providers).

<div id="how-model-selection-works">
  ## Cómo funciona la selección de modelos
</div>

OpenClaw selecciona modelos en este orden:

1. Modelo **primario** (`agents.defaults.model.primary` o `agents.defaults.model`).
2. **Modelos de respaldo** en `agents.defaults.model.fallbacks` (en ese orden).
3. La **conmutación por error de autenticación del proveedor** ocurre dentro de un proveedor antes de pasar al siguiente modelo.

Relacionado:

* `agents.defaults.models` es la lista de permitidos (catálogo) de modelos que OpenClaw puede usar (más alias).
* `agents.defaults.imageModel` se usa **solo cuando** el modelo primario no puede aceptar imágenes.
* Los valores predeterminados por agente pueden sobrescribir `agents.defaults.model` mediante `agents.list[].model` más vinculaciones (consulta [/concepts/multi-agent](/es/concepts/multi-agent)).

<div id="quick-model-picks-anecdotal">
  ## Recomendaciones rápidas de modelos (anecdóticas)
</div>

* **GLM**: un poco mejor para programar/hacer llamadas a herramientas.
* **MiniMax**: mejor para redacción y tono general.

<div id="setup-wizard-recommended">
  ## Asistente de configuración (recomendado)
</div>

Si no quieres editar la configuración a mano, ejecuta el asistente de configuración inicial:

```bash
openclaw onboard
```

Puede configurar modelos y autenticación para proveedores comunes, incluida la **suscripción a OpenAI Code (Codex)**
(OAuth) y **Anthropic** (se recomienda clave de API; `claude
setup-token` también se admite).

<div id="config-keys-overview">
  ## Claves de configuración (descripción general)
</div>

* `agents.defaults.model.primary` y `agents.defaults.model.fallbacks`
* `agents.defaults.imageModel.primary` y `agents.defaults.imageModel.fallbacks`
* `agents.defaults.models` (lista de permitidos + alias + parámetros de proveedor)
* `models.providers` (proveedores personalizados escritos en `models.json`)

Las referencias de modelos se normalizan a minúsculas. Los alias de proveedor como `z.ai/*` se normalizan como
`zai/*`.

Los ejemplos de configuración de proveedores (incluido OpenCode Zen) se encuentran en
[/gateway/configuration](/es/gateway/configuration#opencode-zen-multi-model-proxy).

<div id="model-is-not-allowed-and-why-replies-stop">
  ## “El modelo no está permitido” (y por qué dejan de enviarse respuestas)
</div>

Si `agents.defaults.models` está configurado, se convierte en la **lista de permitidos** para `/model` y para las anulaciones de configuración de sesión. Cuando un usuario selecciona un modelo que no está en esa lista de permitidos, OpenClaw devuelve:

```
Model "provider/model" is not allowed. Use /model to list available models.
```

Esto ocurre **antes** de que se genere una respuesta normal, por lo que el mensaje puede parecer
que “no respondió”. La solución es:

* Añadir el modelo a `agents.defaults.models`, o
* Vaciar la lista de permitidos (eliminar `agents.defaults.models`), o
* Elegir un modelo desde `/model list`.

Ejemplo de configuración de lista de permitidos:

```json5
{
  agent: {
    model: { primary: "anthropic/claude-sonnet-4-5" },
    models: {
      "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
      "anthropic/claude-opus-4-5": { alias: "Opus" }
    }
  }
}
```

<div id="switching-models-in-chat-model">
  ## Cambiar de modelo en el chat (`/model`)
</div>

Puedes cambiar el modelo de la sesión actual sin reiniciar:

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model status
```

Notas:

* `/model` (y `/model list`) es un selector compacto y numerado (familia de modelos + proveedores disponibles).
* `/model <#>` selecciona desde ese selector.
* `/model status` es la vista detallada (candidatos de autenticación y, cuando está configurado, endpoint del proveedor `baseUrl` + modo `api`).
* Las referencias de modelo se analizan dividiéndolas por la **primera** `/`. Usa `provider/model` al escribir `/model <ref>`.
* Si el propio ID del modelo contiene `/` (estilo OpenRouter), debes incluir el prefijo del proveedor (ejemplo: `/model openrouter/moonshotai/kimi-k2`).
* Si omites el proveedor, OpenClaw trata la entrada como un alias o un modelo para el **proveedor predeterminado** (solo funciona cuando no hay `/` en el ID del modelo).

Comportamiento y configuración completos del comando: [Slash commands](/es/tools/slash-commands).

<div id="cli-commands">
  ## Comandos de la CLI
</div>

```bash
openclaw models list
openclaw models status
openclaw models set <proveedor/modelo>
openclaw models set-image <proveedor/modelo>

openclaw models aliases list
openclaw models aliases add <alias> <proveedor/modelo>
openclaw models aliases remove <alias>

openclaw models fallbacks list
openclaw models fallbacks add <proveedor/modelo>
openclaw models fallbacks remove <proveedor/modelo>
openclaw models fallbacks clear

openclaw models image-fallbacks list
openclaw models image-fallbacks add <proveedor/modelo>
openclaw models image-fallbacks remove <proveedor/modelo>
openclaw models image-fallbacks clear
```

`openclaw models` (sin subcomando) es equivalente a `models status`.

<div id="models-list">
  ### `models list`
</div>

Muestra por defecto los modelos configurados. Opciones útiles:

* `--all`: catálogo completo
* `--local`: solo proveedores locales
* `--provider <name>`: filtrar por proveedor
* `--plain`: un modelo por línea
* `--json`: salida legible por máquina

<div id="models-status">
  ### `models status`
</div>

Muestra el modelo primario resuelto, los fallbacks, el modelo de imagen y una vista general de autenticación
de los proveedores configurados. También muestra el estado de expiración de OAuth para los perfiles encontrados
en el almacén de autenticación (avisa dentro de las 24 h de forma predeterminada). `--plain` imprime solo el
modelo primario resuelto.
El estado de OAuth siempre se muestra (y se incluye en la salida de `--json`). Si un proveedor configurado no tiene credenciales, `models status` imprime una sección de **Missing auth**.
El JSON incluye `auth.oauth` (ventana de aviso + perfiles) y `auth.providers`
(autenticación efectiva por proveedor).
Usa `--check` para automatización (sale con código `1` cuando falta/está expirada, `2` cuando está por expirar).

La autenticación preferida de Anthropic es el token de configuración setup-token de Claude Code CLI (ejecútalo en cualquier parte; pégalo en el host del Gateway si es necesario):

```bash
claude setup-token
openclaw models status
```

<div id="scanning-openrouter-free-models">
  ## Escaneo (modelos gratuitos de OpenRouter)
</div>

`openclaw models scan` inspecciona el **catálogo de modelos gratuitos** de OpenRouter y puede,
de forma opcional, sondear modelos para comprobar compatibilidad con herramientas e imágenes.

Flags principales:

* `--no-probe`: omitir sondeos en vivo (solo metadatos)
* `--min-params <b>`: tamaño mínimo de parámetros (en miles de millones)
* `--max-age-days <days>`: omitir modelos más antiguos
* `--provider <name>`: filtro por prefijo de proveedor
* `--max-candidates <n>`: tamaño de la lista de alternativas de reserva
* `--set-default`: establece `agents.defaults.model.primary` en la primera selección
* `--set-image`: establece `agents.defaults.imageModel.primary` en la primera selección de imagen

El sondeo requiere una clave de API de OpenRouter (de los perfiles de autenticación o
`OPENROUTER_API_KEY`). Sin una clave, usa `--no-probe` para listar solo los candidatos.

Los resultados del escaneo se ordenan por:

1. Compatibilidad con imágenes
2. Latencia de herramientas
3. Tamaño de contexto
4. Número de parámetros

Entrada

* Lista `/models` de OpenRouter (filtro `:free`)
* Requiere una clave de API de OpenRouter desde los perfiles de autenticación o `OPENROUTER_API_KEY` (consulta [/environment](/es/environment))
* Filtros opcionales: `--max-age-days`, `--min-params`, `--provider`, `--max-candidates`
* Controles de sondeo: `--timeout`, `--concurrency`

Cuando se ejecuta en un TTY, puedes seleccionar alternativas de reserva de forma interactiva. En modo no interactivo,
pasa `--yes` para aceptar los valores predeterminados.

<div id="models-registry-modelsjson">
  ## Registro de modelos (`models.json`)
</div>

Los proveedores personalizados en `models.providers` se guardan en `models.json` dentro del
directorio del agente (por defecto `~/.openclaw/agents/<agentId>/models.json`). Este archivo
se combina de forma predeterminada, a menos que `models.mode` esté establecido en `replace`.