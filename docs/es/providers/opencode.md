---
title: Opencode
summary: "Usa OpenCode Zen (modelos seleccionados) con OpenClaw"
read_when:
  - Quieres OpenCode Zen para acceder a modelos
  - Quieres una lista seleccionada de modelos optimizados para programación
---

<div id="opencode-zen">
  # OpenCode Zen
</div>

OpenCode Zen es una **lista seleccionada de modelos** recomendada por el equipo de OpenCode para agentes de programación.
Es una ruta de acceso a modelos, alojada y opcional, que utiliza una clave de API y el proveedor `opencode`.
Zen está actualmente en fase beta.

<div id="cli-setup">
  ## Configuración de la CLI
</div>

```bash
openclaw onboard --auth-choice opencode-zen
# o de forma no interactiva
openclaw onboard --opencode-zen-api-key "$OPENCODE_API_KEY"
```


<div id="config-snippet">
  ## Ejemplo de configuración
</div>

```json5
{
  env: { OPENCODE_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-5" } } }
}
```


<div id="notes">
  ## Notas
</div>

- `OPENCODE_ZEN_API_KEY` también se admite.
- Inicias sesión en Zen, añades los datos de facturación y copias tu clave de API.
- OpenCode Zen factura por solicitud; consulta el panel de control de OpenCode para obtener más detalles.