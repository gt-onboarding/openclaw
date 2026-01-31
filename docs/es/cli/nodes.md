---
title: Nodos
summary: "Referencia de la CLI para `openclaw nodes` (listar/estado/aprobar/invocar, cámara/lienzo/pantalla)"
read_when:
  - Estás gestionando nodos emparejados (cámaras, pantalla, lienzo)
  - Necesitas aprobar solicitudes o invocar comandos de nodo
---

<div id="openclaw-nodes">
  # `openclaw nodes`
</div>

Administra nodos (dispositivos) emparejados e invoca las capacidades del nodo.

Relacionado:

* Descripción general de nodos: [Nodos](/es/nodes)
* Cámara: [Nodos de cámara](/es/nodes/camera)
* Imágenes: [Nodos de imágenes](/es/nodes/images)

Opciones comunes:

* `--url`, `--token`, `--timeout`, `--json`

<div id="common-commands">
  ## Comandos comunes
</div>

```bash
openclaw nodes list
openclaw nodes list --connected
openclaw nodes list --last-connected 24h
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes status
openclaw nodes status --connected
openclaw nodes status --last-connected 24h
```

`nodes list` imprime tablas de nodos pendientes y emparejados. Las filas emparejadas incluyen el tiempo desde la conexión más reciente (Última conexión).
Usa `--connected` para mostrar solo los nodos conectados actualmente. Usa `--last-connected <duration>` para
filtrar nodos que se conectaron dentro de un intervalo (por ejemplo, `24h`, `7d`).

<div id="invoke-run">
  ## Invocar / ejecutar
</div>

```bash
openclaw nodes invoke --node <id|name|ip> --command <command> --params <json>
openclaw nodes run --node <id|name|ip> <command...>
openclaw nodes run --raw "git status"
openclaw nodes run --agent main --node <id|name|ip> --raw "git status"
```

Flags de invocación:

* `--params <json>`: cadena con un objeto JSON (valor predeterminado `{}`).
* `--invoke-timeout <ms>`: tiempo de espera para la invocación del nodo (valor predeterminado `15000`).
* `--idempotency-key <key>`: clave de idempotencia opcional.

<div id="exec-style-defaults">
  ### Valores predeterminados de estilo exec
</div>

`nodes run` refleja el comportamiento exec del modelo (valores predeterminados + aprobaciones):

* Lee `tools.exec.*` (más las anulaciones `agents.list[].tools.exec.*`).
* Usa aprobaciones de exec (`exec.approval.request`) antes de invocar `system.run`.
* `--node` se puede omitir cuando `tools.exec.node` está configurado.
* Requiere un nodo que anuncie `system.run` (app complementaria de macOS o nodo host sin interfaz gráfica).

Flags:

* `--cwd <path>`: directorio de trabajo.
* `--env <key=val>`: anulación de variable de entorno (se puede repetir).
* `--command-timeout <ms>`: tiempo de espera del comando.
* `--invoke-timeout <ms>`: tiempo de espera de invocación del nodo (predeterminado `30000`).
* `--needs-screen-recording`: requiere permiso de grabación de pantalla.
* `--raw <command>`: ejecuta una cadena de shell (`/bin/sh -lc` o `cmd.exe /c`).
* `--agent <id>`: aprobaciones y listas de permitidos con ámbito de agente (predeterminado: agente configurado).
* `--ask <off|on-miss|always>`, `--security <deny|allowlist|full>`: anulaciones.