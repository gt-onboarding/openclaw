---
title: Configuración de habilidades
summary: "Esquema de configuración de habilidades y ejemplos"
read_when:
  - Agregar o modificar la configuración de habilidades
  - Ajustar la lista de permitidos incorporada o el comportamiento de instalación
---

<div id="skills-config">
  # Configuración de habilidades
</div>

Toda la configuración relacionada con las habilidades se encuentra dentro de `skills` en `~/.openclaw/openclaw.json`.

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: [
        "~/Projects/agent-scripts/skills",
        "~/Projects/oss/some-skill-pack/skills"
      ],
      watch: true,
      watchDebounceMs: 250
    },
    install: {
      preferBrew: true,
      nodeManager: "npm" // npm | pnpm | yarn | bun (el runtime del Gateway sigue siendo Node; no se recomienda bun)
    },
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE"
        }
      },
      peekaboo: { enabled: true },
      sag: { enabled: false }
    }
  }
}
```


<div id="fields">
  ## Campos
</div>

- `allowBundled`: lista de permitidos opcional solo para habilidades **incluidas**. Cuando está definido, solo
  las habilidades incluidas en la lista son elegibles (las habilidades gestionadas/en el espacio de trabajo no se ven afectadas).
- `load.extraDirs`: directorios de habilidades adicionales que se escanearán (menor precedencia).
- `load.watch`: supervisa carpetas de habilidades y actualiza la instantánea de habilidades (valor predeterminado: true).
- `load.watchDebounceMs`: tiempo de espera (debounce) para los eventos del observador de habilidades en milisegundos (valor predeterminado: 250).
- `install.preferBrew`: prefiere instaladores de brew cuando estén disponibles (valor predeterminado: true).
- `install.nodeManager`: preferencia de instalador de Node (`npm` | `pnpm` | `yarn` | `bun`, valor predeterminado: npm).
  Esto solo afecta a las **instalaciones de habilidades**; el runtime del Gateway debería seguir siendo Node
  (Bun no se recomienda para WhatsApp/Telegram).
- `entries.<skillKey>`: overrides específicos por habilidad.

Campos por habilidad:

- `enabled`: establece `false` para desactivar una habilidad incluso si viene incluida/instalada.
- `env`: variables de entorno inyectadas para la ejecución del agente (solo si aún no están establecidas).
- `apiKey`: atajo opcional para habilidades que declaran una variable de entorno principal.

<div id="notes">
  ## Notas
</div>

- Las claves debajo de `entries` se asignan al nombre de la habilidad de forma predeterminada. Si una habilidad define
  `metadata.openclaw.skillKey`, utiliza esa clave en su lugar.
- Los cambios en las habilidades se detectan en el siguiente turno del agente cuando el watcher está habilitado.

<div id="sandboxed-skills-env-vars">
  ### Habilidades en sandbox + variables de entorno
</div>

Cuando una sesión está **en sandbox**, los procesos de las habilidades se ejecutan dentro de Docker. El sandbox
**no** hereda el `process.env` del host.

Usa una de las siguientes opciones:

- `agents.defaults.sandbox.docker.env` (o por agente `agents.list[].sandbox.docker.env`)
- incluye el entorno en tu imagen de sandbox personalizada

La configuración global `env` y `skills.entries.<skill>.env/apiKey` se aplica solo a las ejecuciones en el **host**.