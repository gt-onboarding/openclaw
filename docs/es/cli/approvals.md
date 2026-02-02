---
title: Aprobaciones
summary: "Referencia de la CLI para `openclaw approvals` (aprobaciones de ejecución para hosts del Gateway o nodos)"
read_when:
  - Quieres editar aprobaciones de ejecución desde la CLI
  - Necesitas gestionar listas de permitidos en hosts del Gateway o nodos
---

<div id="openclaw-approvals">
  # `openclaw approvals`
</div>

Administra las aprobaciones de ejecución (exec) para el **host local**, el **host del Gateway** o el **host de un nodo**.
De forma predeterminada, los comandos se aplican al archivo local de aprobaciones en disco. Usa `--gateway` para apuntar al Gateway o `--node` para apuntar a un nodo específico.

Relacionado:

* Aprobaciones de ejecución (exec): [Exec approvals](/es/tools/exec-approvals)
* Nodos: [Nodes](/es/nodes)

<div id="common-commands">
  ## Comandos habituales
</div>

```bash
openclaw approvals get
openclaw approvals get --node <id|name|ip>
openclaw approvals get --gateway
```

<div id="replace-approvals-from-a-file">
  ## Reemplazar aprobaciones a partir de un archivo
</div>

```bash
openclaw approvals set --file ./exec-approvals.json
openclaw approvals set --node <id|name|ip> --file ./exec-approvals.json
openclaw approvals set --gateway --file ./exec-approvals.json
```

<div id="allowlist-helpers">
  ## Funciones auxiliares de la lista de permitidos
</div>

```bash
openclaw approvals allowlist add "~/Projects/**/bin/rg"
openclaw approvals allowlist add --agent main --node <id|name|ip> "/usr/bin/uptime"
openclaw approvals allowlist add --agent "*" "/usr/bin/uname"

openclaw approvals allowlist remove "~/Projects/**/bin/rg"
```

<div id="notes">
  ## Notas
</div>

* `--node` usa el mismo mecanismo de resolución que `openclaw nodes` (id, nombre, ip o prefijo de id).
* `--agent` tiene como valor predeterminado `"*"`, lo que aplica a todos los agentes.
* El host del nodo debe anunciar `system.execApprovals.get/set` (app de macOS o host de nodo sin interfaz).
* Los archivos de aprobaciones se almacenan por host en `~/.openclaw/exec-approvals.json`.