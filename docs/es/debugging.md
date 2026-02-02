---
title: Depuración
summary: "Herramientas de depuración: modo watch, flujos sin procesar del modelo y rastreo de filtraciones de razonamiento"
read_when:
  - Necesitas inspeccionar la salida sin procesar del modelo para detectar filtraciones de razonamiento
  - Quieres ejecutar el Gateway en modo watch mientras iteras
  - Necesitas un flujo de trabajo de depuración repetible
---

<div id="debugging">
  # Depuración
</div>

Esta página describe herramientas de depuración para la salida en streaming, especialmente cuando un proveedor mezcla razonamiento con texto normal.

<div id="runtime-debug-overrides">
  ## Anulaciones de depuración en tiempo de ejecución
</div>

Usa `/debug` en el chat para establecer anulaciones de configuración **solo en tiempo de ejecución** (memoria, no disco).
`/debug` está deshabilitado de forma predeterminada; actívalo con `commands.debug: true`.
Esto es útil cuando necesitas alternar ajustes poco comunes sin editar `openclaw.json`.

Ejemplos:

```
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug unset messages.responsePrefix
/debug reset
```

`/debug reset` restablece todos los valores anulados y vuelve a la configuración almacenada en disco.


<div id="gateway-watch-mode">
  ## Modo de vigilancia del Gateway
</div>

Para iterar rápidamente, ejecuta el Gateway con el monitor de archivos:

```bash
pnpm gateway:watch --force
```

Esto se corresponde con:

```bash
tsx watch src/entry.ts gateway --force
```

Añade cualquier flag de la CLI del Gateway después de `gateway:watch` y se pasarán
en cada reinicio.


<div id="dev-profile-dev-gateway-dev">
  ## Perfil de desarrollo + Gateway de desarrollo (--dev)
</div>

Usa el perfil de desarrollo para aislar el estado y poner en marcha una configuración
segura y desechable para depuración. Hay **dos** flags `--dev`:

* **`--dev` global (perfil):** aísla el estado bajo `~/.openclaw-dev` y
  establece por defecto el puerto del Gateway en `19001` (los puertos derivados se ajustan en consecuencia).
* **`gateway --dev`: le indica al Gateway que cree automáticamente una configuración por defecto +
  espacio de trabajo** si no existe (y omita BOOTSTRAP.md).

Flujo recomendado (perfil de desarrollo + bootstrap de desarrollo):

```bash
pnpm gateway:dev
OPENCLAW_PROFILE=dev openclaw tui
```

Si aún no tienes una instalación global, ejecuta la CLI con `pnpm openclaw ...`.

Qué hace esto:

1. **Aislamiento de perfil** (global `--dev`)
   * `OPENCLAW_PROFILE=dev`
   * `OPENCLAW_STATE_DIR=~/.openclaw-dev`
   * `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
   * `OPENCLAW_GATEWAY_PORT=19001` (el navegador y el canvas se ajustan en consecuencia)

2. **Bootstrap de desarrollo** (`gateway --dev`)
   * Escribe una configuración mínima si falta (`gateway.mode=local`, enlaza a loopback).
   * Establece `agent.workspace` en el espacio de trabajo de desarrollo.
   * Establece `agent.skipBootstrap=true` (sin BOOTSTRAP.md).
   * Inicializa los archivos del espacio de trabajo si faltan:
     `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`.
   * Identidad predeterminada: **C3‑PO** (droide de protocolo).
   * Omite los proveedores de canales en modo de desarrollo (`OPENCLAW_SKIP_CHANNELS=1`).

Flujo de restablecimiento (inicio desde cero):

```bash
pnpm gateway:dev:reset
```

Nota: `--dev` es un flag de perfil **global** y algunos runners se lo comen.
Si necesitas especificarlo explícitamente, usa la forma de variable de entorno:

```bash
OPENCLAW_PROFILE=dev openclaw gateway --dev --reset
```

`--reset` borra la configuración, las credenciales, las sesiones y el espacio de trabajo de desarrollo (usando
`trash` en lugar de `rm`), luego recrea la configuración de desarrollo predeterminada.

Consejo: si ya se está ejecutando un Gateway que no es de desarrollo (launchd/systemd), deténlo primero:

```bash
openclaw gateway stop
```


<div id="raw-stream-logging-openclaw">
  ## Registro de flujo bruto (OpenClaw)
</div>

OpenClaw puede registrar el **flujo bruto del asistente** antes de cualquier filtrado o formateo.
Esta es la mejor manera de ver si el razonamiento llega como deltas de texto sin formato
(o como bloques de razonamiento separados).

Habilítalo desde la CLI:

```bash
pnpm gateway:watch --force --raw-stream
```

Anulación opcional de ruta:

```bash
pnpm gateway:watch --force --raw-stream --raw-stream-path ~/.openclaw/logs/raw-stream.jsonl
```

Variables de entorno equivalentes:

```bash
OPENCLAW_RAW_STREAM=1
OPENCLAW_RAW_STREAM_PATH=~/.openclaw/logs/raw-stream.jsonl
```

Archivo predeterminado:

`~/.openclaw/logs/raw-stream.jsonl`


<div id="raw-chunk-logging-pi-mono">
  ## Registro de chunks sin procesar (pi-mono)
</div>

Para capturar **chunks OpenAI-compat sin procesar** antes de que se conviertan en bloques,
pi-mono expone un logger independiente:

```bash
PI_RAW_STREAM=1
```

Ruta (opcional):

```bash
PI_RAW_STREAM_PATH=~/.pi-mono/logs/raw-openai-completions.jsonl
```

Archivo predeterminado:

`~/.pi-mono/logs/raw-openai-completions.jsonl`

> Nota: este archivo solo lo generan los procesos que usan el proveedor
> `openai-completions` de pi-mono.


<div id="safety-notes">
  ## Notas de seguridad
</div>

- Los registros de streaming en bruto pueden incluir prompts completos, salida de herramientas y datos de usuario.
- Mantén los registros en local y elimínalos cuando termines de depurar.
- Si compartes registros, elimina primero cualquier secreto y datos personales (PII).