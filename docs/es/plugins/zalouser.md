---
title: Zalouser
summary: "Complemento Zalo Personal (no oficial): inicio de sesión mediante QR + mensajería vía zca-cli (instalación del complemento + configuración del canal + CLI + herramienta)"
read_when:
  - Quieres soporte no oficial de Zalo Personal en OpenClaw
  - Estás configurando o desarrollando el complemento zalouser
---

<div id="zalo-personal-plugin">
  # Zalo Personal (complemento)
</div>

Compatibilidad de Zalo Personal con OpenClaw a través de un complemento, usando `zca-cli` para automatizar una cuenta de usuario normal de Zalo.

> **Advertencia:** La automatización no oficial puede provocar la suspensión o el bloqueo de la cuenta. Úsalo bajo tu propia responsabilidad.

<div id="naming">
  ## Nomenclatura
</div>

El ID del canal es `zalouser` para dejar claro que esto automatiza una **cuenta de usuario personal de Zalo** (no oficial). Mantenemos `zalo` reservado para una posible integración oficial futura con la API de Zalo.

<div id="where-it-runs">
  ## Dónde se ejecuta
</div>

Este complemento se ejecuta **dentro del proceso del Gateway**.

Si utilizas un Gateway remoto, instálalo/configúralo en la **máquina donde se ejecuta el Gateway** y luego reinicia el Gateway.

<div id="install">
  ## Instalación
</div>

<div id="option-a-install-from-npm">
  ### Opción A: instalar desde npm
</div>

```bash
openclaw plugins install @openclaw/zalouser
```

Luego, reinicia el Gateway.


<div id="option-b-install-from-a-local-folder-dev">
  ### Opción B: instalar desde una carpeta local (dev)
</div>

```bash
openclaw plugins install ./extensions/zalouser
cd ./extensions/zalouser && pnpm install
```

Luego, reinicia el Gateway.


<div id="prerequisite-zca-cli">
  ## Requisito previo: zca-cli
</div>

La máquina que ejecuta el Gateway debe tener `zca` en el `PATH`:

```bash
zca --version
```


<div id="config">
  ## Configuración
</div>

La configuración del canal se define en `channels.zalouser` (no en `plugins.entries.*`):

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing"
    }
  }
}
```


<div id="cli">
  ## CLI
</div>

```bash
openclaw channels login --channel zalouser
openclaw channels logout --channel zalouser
openclaw channels status --probe
openclaw message send --channel zalouser --target <threadId> --message "Hello from OpenClaw"
openclaw directory peers list --channel zalouser --query "name"
```


<div id="agent-tool">
  ## Herramienta de agente
</div>

Nombre de la herramienta: `zalouser`

Acciones: `send`, `image`, `link`, `friends`, `groups`, `me`, `status`