---
title: Configuración
summary: "Referencia de la CLI para `openclaw config` (obtener/establecer/eliminar valores de configuración)"
read_when:
  - Quieres leer o modificar la configuración de manera no interactiva
---

<div id="openclaw-config">
  # `openclaw config`
</div>

Utilidades de configuración para obtener/establecer/eliminar valores por ruta. Ejecútalo sin un subcomando para abrir
el asistente de configuración (igual que `openclaw configure`).

<div id="examples">
  ## Ejemplos
</div>

```bash
openclaw config get browser.executablePath
openclaw config set browser.executablePath "/usr/bin/google-chrome"
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
openclaw config unset tools.web.search.apiKey
```


<div id="paths">
  ## Rutas
</div>

Las rutas usan notación con puntos o con corchetes:

```bash
openclaw config get agents.defaults.workspace
openclaw config get agents.list[0].id
```

Usa el índice de la lista de agentes para seleccionar un agente específico:

```bash
openclaw config get agents.list
openclaw config set agents.list[1].tools.exec.node "node-id-or-name"
```


<div id="values">
  ## Valores
</div>

Los valores se interpretan como JSON5 siempre que sea posible; en caso contrario se tratan como cadenas de texto.
Usa `--json` para exigir la interpretación como JSON5.

```bash
openclaw config set agents.defaults.heartbeat.every "0m"
openclaw config set gateway.port 19001 --json
openclaw config set channels.whatsapp.groups '["*"]' --json
```

Reinicia el Gateway después de editarlo.
