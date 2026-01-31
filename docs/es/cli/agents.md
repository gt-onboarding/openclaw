---
title: Agentes
summary: "Referencia de la CLI de `openclaw agents` (listar/añadir/eliminar/establecer identidad)"
read_when:
  - Quieres múltiples agentes aislados (espacios de trabajo + enrutamiento + autenticación)
---

<div id="openclaw-agents">
  # `openclaw agents`
</div>

Administra agentes aislados (espacios de trabajo + autenticación + enrutamiento).

Relacionado:

* Enrutamiento multiagente: [Enrutamiento multiagente](/es/concepts/multi-agent)
* Espacio de trabajo del agente: [Espacio de trabajo del agente](/es/concepts/agent-workspace)

<div id="examples">
  ## Ejemplos
</div>

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

<div id="identity-files">
  ## Archivos de identidad
</div>

Cada espacio de trabajo de un agente puede incluir un `IDENTITY.md` en la raíz del espacio de trabajo:

* Ruta de ejemplo: `~/.openclaw/workspace/IDENTITY.md`
* `set-identity --from-identity` lee desde la raíz del espacio de trabajo (o desde un `--identity-file` explícito)

Las rutas de los avatares se resuelven de forma relativa a la raíz del espacio de trabajo.

<div id="set-identity">
  ## Establecer identidad
</div>

`set-identity` escribe campos en `agents.list[].identity`:

* `name`
* `theme`
* `emoji`
* `avatar` (ruta relativa al espacio de trabajo, URL http(s) o URI de datos)

Cargar desde `IDENTITY.md`:

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

Sobrescribe los campos explícitamente:

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

Ejemplo de configuración:

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "OpenClaw",
          theme: "space lobster",
          emoji: "🦞",
          avatar: "avatars/openclaw.png"
        }
      }
    ]
  }
}
```
