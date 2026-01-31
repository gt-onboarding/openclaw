---
title: Complementos
summary: "Referencia de la CLI para `openclaw plugins` (listar, instalar, habilitar/deshabilitar, doctor)"
read_when:
  - Quieres instalar o gestionar complementos del Gateway que se ejecutan en el mismo proceso
  - Quieres depurar errores de carga de complementos
---

<div id="openclaw-plugins">
  # `openclaw plugins`
</div>

Gestionar complementos/extensiones del Gateway (cargados en el mismo proceso).

Relacionados:

* Sistema de complementos: [Plugins](/es/plugin)
* Manifiesto del complemento y esquema: [Plugin manifest](/es/plugins/manifest)
* Refuerzo de la seguridad: [Security](/es/gateway/security)

<div id="commands">
  ## Comandos
</div>

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

Los complementos incluidos vienen con OpenClaw, pero se inician deshabilitados. Usa `plugins enable` para
activarlos.

Todos los complementos deben incluir un archivo `openclaw.plugin.json` con un esquema JSON incrustado
(`configSchema`, incluso si está vacío). La ausencia de manifiestos o esquemas, o que sean no válidos, impide
que el complemento se cargue y hace que falle la validación de la configuración.

<div id="install">
  ### Instalar
</div>

```bash
openclaw plugins install <path-or-spec>
```

Nota de seguridad: trata la instalación de complementos como si estuvieras ejecutando código. Da preferencia a versiones fijadas.

Formatos de archivo admitidos: `.zip`, `.tgz`, `.tar.gz`, `.tar`.

Usa `--link` para evitar copiar un directorio local (lo añade a `plugins.load.paths`):

```bash
openclaw plugins install -l ./my-plugin
```

<div id="update">
  ### Actualizar
</div>

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

Las actualizaciones solo se aplican a los complementos instalados desde npm (registrados en `plugins.installs`).
