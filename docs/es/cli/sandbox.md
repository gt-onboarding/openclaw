---
title: CLI de sandbox
summary: "Administra contenedores de sandbox e inspecciona la política efectiva de sandbox"
read_when: "Estás administrando contenedores de sandbox o depurando el comportamiento de sandbox/tool-policy."
status: active
---

<div id="sandbox-cli">
  # CLI de sandbox
</div>

Gestiona contenedores de sandbox basados en Docker para ejecutar agentes de forma aislada.

<div id="overview">
  ## Descripción general
</div>

OpenClaw puede ejecutar agentes dentro de contenedores Docker aislados por motivos de seguridad. Los comandos de `sandbox` te ayudan a administrar estos contenedores, especialmente después de actualizaciones o cambios de configuración.

<div id="commands">
  ## Comandos
</div>

<div id="openclaw-sandbox-explain">
  ### `openclaw sandbox explain`
</div>

Inspecciona el modo de sandbox efectivo, el `scope` efectivo, el acceso efectivo al espacio de trabajo, la política de herramientas de sandbox y los gates elevados (incluidas las rutas de las claves de configuración correspondientes para corregirlos).

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

<div id="openclaw-sandbox-list">
  ### `openclaw sandbox list`
</div>

Enumera todos los contenedores sandbox con su estado y configuración.

```bash
openclaw sandbox list
openclaw sandbox list --browser  # Listar solo los contenedores de navegador
openclaw sandbox list --json     # Salida en JSON
```

**La salida incluye:**

* Nombre del contenedor y estado (en ejecución/detenido)
* Imagen de Docker y si coincide con la configuración
* Antigüedad (tiempo desde la creación)
* Tiempo de inactividad (tiempo desde el último uso)
* Sesión/agente asociado

<div id="openclaw-sandbox-recreate">
  ### `openclaw sandbox recreate`
</div>

Elimina los contenedores de sandbox para forzar su recreación con imágenes y configuración actualizadas.

```bash
openclaw sandbox recreate --all                # Recreate all containers
openclaw sandbox recreate --session main       # Specific session
openclaw sandbox recreate --agent mybot        # Specific agent
openclaw sandbox recreate --browser            # Solo contenedores del navegador
openclaw sandbox recreate --all --force        # Skip confirmation
```

**Opciones:**

* `--all`: Recrea todos los contenedores sandbox
* `--session <key>`: Recrea el contenedor para una sesión específica
* `--agent <id>`: Recrea los contenedores para un agente específico
* `--browser`: Solo recrea los contenedores del navegador
* `--force`: Omite el mensaje de confirmación

**Importante:** Los contenedores se recrean automáticamente la próxima vez que se use el agente.

<div id="use-cases">
  ## Casos de uso
</div>

<div id="after-updating-docker-images">
  ### Después de actualizar las imágenes de Docker
</div>

```bash
# Pull new image
docker pull openclaw-sandbox:latest
docker tag openclaw-sandbox:latest openclaw-sandbox:bookworm-slim

# Update config to use new image
# Editar configuración: agents.defaults.sandbox.docker.image (o agents.list[].sandbox.docker.image)

# Recreate containers
openclaw sandbox recreate --all
```

<div id="after-changing-sandbox-configuration">
  ### Tras cambiar la configuración de la sandbox
</div>

```bash
# Editar configuración: agents.defaults.sandbox.* (o agents.list[].sandbox.*)

# Recrear para aplicar la nueva configuración
openclaw sandbox recreate --all
```

<div id="after-changing-setupcommand">
  ### Después de modificar setupCommand
</div>

```bash
openclaw sandbox recreate --all
# o solo un agente:
openclaw sandbox recreate --agent family
```

<div id="for-a-specific-agent-only">
  ### Solo para un agente específico
</div>

```bash
# Actualizar únicamente los contenedores de un agente
openclaw sandbox recreate --agent alfred
```

<div id="why-is-this-needed">
  ## ¿Por qué es necesario?
</div>

**Problema:** Cuando actualizas las imágenes Docker del sandbox o su configuración:

* Los contenedores existentes siguen ejecutándose con la configuración anterior
* Los contenedores solo se eliminan después de 24 h de inactividad
* Los agentes que se usan con regularidad mantienen los contenedores antiguos en ejecución indefinidamente

**Solución:** Usa `openclaw sandbox recreate` para forzar la eliminación de contenedores antiguos. Se volverán a crear automáticamente con la configuración actual cuando se necesiten de nuevo.

Consejo: prefiere `openclaw sandbox recreate` frente al uso manual de `docker rm`. Utiliza el esquema de nombres de contenedores del Gateway y evita desajustes cuando cambian las claves scope/session.

<div id="configuration">
  ## Configuración
</div>

La configuración de la sandbox se encuentra en `~/.openclaw/openclaw.json` bajo `agents.defaults.sandbox` (las anulaciones específicas de cada agente van en `agents.list[].sandbox`):

```jsonc
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "all",                    // off, non-main, all
        "scope": "agent",                 // session, agent, shared
        "docker": {
          "image": "openclaw-sandbox:bookworm-slim",
          "containerPrefix": "openclaw-sbx-"
          // ... more Docker options
        },
        "prune": {
          "idleHours": 24,               // Limpieza automática tras 24 h de inactividad
          "maxAgeDays": 7                // Auto-prune after 7 days
        }
      }
    }
  }
}
```

<div id="see-also">
  ## Consulta también
</div>

* [Documentación de la sandbox](/es/gateway/sandboxing)
* [Configuración del Agente](/es/concepts/agent-workspace)
* [Comando doctor](/es/gateway/doctor) - Comprueba la configuración de la sandbox