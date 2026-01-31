---
title: Nodo
summary: "Referencia de la CLI para `openclaw node` (host de nodo sin interfaz gráfica)"
read_when:
  - Al ejecutar el host de nodo sin interfaz gráfica
  - Al realizar el emparejamiento de un nodo que no es macOS para system.run
---

<div id="openclaw-node">
  # `openclaw node`
</div>

Ejecuta un **host de nodo sin interfaz gráfica** que se conecta al WebSocket del Gateway y expone
`system.run` y `system.which` en este equipo.

<div id="why-use-a-node-host">
  ## ¿Por qué usar un host de nodo?
</div>

Usa un host de nodo cuando quieras que los agentes **ejecuten comandos en otras máquinas**
de tu red sin instalar allí una app complementaria completa para macOS.

Casos de uso comunes:

* Ejecutar comandos en máquinas Linux/Windows remotas (servidores de compilación, máquinas de laboratorio, NAS).
* Mantener la ejecución **sandboxed** en el Gateway, pero delegar ejecuciones aprobadas a otros hosts.
* Proporcionar un destino de ejecución ligero y sin interfaz (headless) para automatización o nodos de CI.

La ejecución sigue estando protegida por **exec approvals** y listas de permitidos por agente en el
host de nodo, de modo que puedas mantener el acceso a comandos con un ámbito bien delimitado y explícito.

<div id="browser-proxy-zero-config">
  ## Proxy de navegador (sin configuración previa)
</div>

Los hosts de nodo anuncian automáticamente un proxy de navegador si `browser.enabled` no está desactivado en el nodo. Esto permite que el agente use la automatización del navegador en ese nodo sin configuración adicional.

Desactívalo en el nodo si es necesario:

```json5
{
  nodeHost: {
    browserProxy: {
      enabled: false
    }
  }
}
```

<div id="run-foreground">
  ## Ejecución (primer plano)
</div>

```bash
openclaw node run --host <gateway-host> --port 18789
```

Options:

* `--host <host>`: Host WebSocket del Gateway (predeterminado: `127.0.0.1`)
* `--port <port>`: Puerto WebSocket del Gateway (predeterminado: `18789`)
* `--tls`: Usa TLS para la conexión con el Gateway
* `--tls-fingerprint <sha256>`: Huella digital esperada del certificado TLS (sha256)
* `--node-id <id>`: Sobrescribe el ID del nodo (elimina el token de emparejamiento)
* `--display-name <name>`: Sobrescribe el nombre para mostrar del nodo

<div id="service-background">
  ## Servicio (en segundo plano)
</div>

Instala un host de nodo headless como servicio de usuario.

```bash
openclaw node install --host <gateway-host> --port 18789
```

Opciones:

* `--host &lt;host&gt;`: Host WebSocket del Gateway (predeterminado: `127.0.0.1`)
* `--port &lt;port&gt;`: Puerto WebSocket del Gateway (predeterminado: `18789`)
* `--tls`: Usar TLS para la conexión al Gateway
* `--tls-fingerprint &lt;sha256&gt;`: Huella digital esperada del certificado TLS (sha256)
* `--node-id &lt;id&gt;`: Sobrescribir el ID del nodo (borra el token de emparejamiento)
* `--display-name &lt;name&gt;`: Sobrescribir el nombre para mostrar del nodo
* `--runtime &lt;runtime&gt;`: Entorno de ejecución del servicio (`node` o `bun`)
* `--force`: Reinstalar/sobrescribir si ya está instalado

Gestiona el servicio:

```bash
openclaw node status
openclaw node stop
openclaw node restart
openclaw node uninstall
```

Usa `openclaw node run` para ejecutar un host de nodo en primer plano (sin servicio del sistema).

Los comandos de servicio aceptan `--json` para obtener salida legible por máquina.

<div id="pairing">
  ## Emparejamiento
</div>

La primera conexión crea en el Gateway una solicitud pendiente de emparejamiento de nodo.
Apruébala con:

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

El host del nodo almacena su ID de nodo, token, nombre para mostrar e información de conexión con el Gateway en
`~/.openclaw/node.json`.

<div id="exec-approvals">
  ## Aprobaciones de ejecución
</div>

`system.run` está controlado mediante aprobaciones locales de ejecución:

* `~/.openclaw/exec-approvals.json`
* [Aprobaciones de ejecución](/es/tools/exec-approvals)
* `openclaw approvals --node <id|name|ip>` (editar desde el Gateway)