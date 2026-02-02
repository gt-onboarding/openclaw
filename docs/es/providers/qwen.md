---
title: Qwen
summary: "Usar Qwen OAuth (plan gratuito) en OpenClaw"
read_when:
  - Quieres usar Qwen con OpenClaw
  - Quieres acceso mediante OAuth al plan gratuito de Qwen Coder
---

<div id="qwen">
  # Qwen
</div>

Qwen ofrece un flujo OAuth gratuito para los modelos Qwen Coder y Qwen Vision
(hasta 2.000 solicitudes/día, sujeto a los límites de frecuencia de Qwen).

<div id="enable-the-plugin">
  ## Habilitar el complemento
</div>

```bash
openclaw plugins enable qwen-portal-auth
```

Reinicia el Gateway después de habilitarlo.

<div id="authenticate">
  ## Autenticación
</div>

```bash
openclaw models auth login --provider qwen-portal --set-default
```

Esto ejecuta el flujo OAuth de código de dispositivo de Qwen y crea una entrada de proveedor en tu
`models.json` (además de un alias `qwen` para cambiar rápidamente).

<div id="model-ids">
  ## Identificadores de modelos
</div>

* `qwen-portal/coder-model`
* `qwen-portal/vision-model`

Cambia de modelo con:

```bash
openclaw models set qwen-portal/coder-model
```

<div id="reuse-qwen-code-cli-login">
  ## Reutilizar el inicio de sesión de Qwen Code CLI
</div>

Si ya has iniciado sesión con Qwen Code CLI, OpenClaw sincronizará las credenciales
desde `~/.qwen/oauth_creds.json` cuando cargue el almacén de autenticación. Todavía necesitas
una entrada `models.providers.qwen-portal` (usa el comando de inicio de sesión anterior para crearla).

<div id="notes">
  ## Notas
</div>

* Los tokens se renuevan automáticamente; vuelve a ejecutar el comando de inicio de sesión si la renovación falla o se revoca el acceso.
* URL base predeterminada: `https://portal.qwen.ai/v1` (sobrescríbela con
  `models.providers.qwen-portal.baseUrl` si Qwen proporciona un endpoint diferente).
* Consulta [Proveedores de modelos](/es/concepts/model-providers) para ver las reglas generales de proveedores.