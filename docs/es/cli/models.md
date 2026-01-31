---
title: Modelos
summary: "Referencia de la CLI para `openclaw models` (status/list/set/scan, alias, alternativas, autenticación)"
read_when:
  - Quieres cambiar los modelos predeterminados o consultar el estado de autenticación de los proveedores
  - Quieres analizar los modelos y proveedores disponibles y depurar los perfiles de autenticación
---

<div id="openclaw-models">
  # `openclaw models`
</div>

Detección, exploración y configuración de modelos (modelo predeterminado, modelos de respaldo, perfiles de autenticación).

Relacionado:

* Proveedores + modelos: [Modelos](/es/providers/models)
* Configuración de autenticación de proveedores: [Primeros pasos](/es/start/getting-started)

<div id="common-commands">
  ## Comandos habituales
</div>

```bash
openclaw models status
openclaw models list
openclaw models set <modelo-o-alias>
openclaw models scan
```

`openclaw models status` muestra los valores predeterminados/de respaldo ya resueltos, además de una vista general de autenticación.
Cuando hay instantáneas de uso de proveedor disponibles, la sección de estado de OAuth/token incluye
encabezados de uso de proveedor.
Añade `--probe` para ejecutar pruebas de autenticación en vivo contra cada perfil de proveedor configurado.
Las pruebas son solicitudes reales (pueden consumir tokens y activar límites de tasa).
Usa `--agent <id>` para inspeccionar el estado de modelo/autenticación de un agente configurado. Si se omite,
el comando usa `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR` si están definidos; de lo contrario, usa el
agente predeterminado configurado.

Notas:

* `models set <model-or-alias>` acepta `provider/model` o un alias.
* Las referencias de modelo se analizan dividiéndolas por la **primera** `/`. Si el ID de modelo incluye `/` (estilo OpenRouter), incluye el prefijo de proveedor (ejemplo: `openrouter/moonshotai/kimi-k2`).
* Si omites el proveedor, OpenClaw trata la entrada como un alias o como un modelo para el **proveedor predeterminado** (solo funciona cuando no hay `/` en el ID de modelo).

<div id="models-status">
  ### `models status`
</div>

Opciones:

* `--json`
* `--plain`
* `--check` (código de salida 1=expirado/ausente, 2=por expirar)
* `--probe` (sondeo en tiempo real de los perfiles de autenticación configurados)
* `--probe-provider <name>` (sondear un proveedor)
* `--probe-profile <id>` (repetir o indicar IDs de perfil separados por comas)
* `--probe-timeout <ms>`
* `--probe-concurrency <n>`
* `--probe-max-tokens <n>`
* `--agent <id>` (ID de agente configurado; anula `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR`)

<div id="aliases-fallbacks">
  ## Alias y alternativas de respaldo
</div>

```bash
openclaw models aliases list
openclaw models fallbacks list
```

<div id="auth-profiles">
  ## Perfiles de autenticación
</div>

```bash
openclaw models auth add
openclaw models auth login --provider <id>
openclaw models auth setup-token
openclaw models auth paste-token
```

`models auth login` ejecuta el flujo de autenticación de un complemento de proveedor (OAuth/API key). Usa
`openclaw plugins list` para ver qué proveedores tienes instalados.

Notas:

* `setup-token` pide que introduzcas un valor de setup-token (genéralo con `claude setup-token` en cualquier máquina).
* `paste-token` acepta un token en forma de cadena de texto generado en otro lugar o desde procesos de automatización.
