---
title: Resolución de problemas
summary: "Centro de resolución de problemas: síntomas → comprobaciones → soluciones"
read_when:
  - Ves un error y quieres saber cómo solucionarlo
  - El instalador indica «success» pero la CLI no funciona
---

<div id="troubleshooting">
  # Resolución de problemas
</div>

<div id="first-60-seconds">
  ## Primeros 60 segundos
</div>

Ejecuta estos pasos en orden:

```bash
openclaw status
openclaw status --all
openclaw gateway probe
openclaw logs --follow
openclaw doctor
```

Si el Gateway es accesible, realiza pruebas en profundidad:

```bash
openclaw status --deep
```

<div id="common-it-broke-cases">
  ## Casos habituales de «algo se rompió»
</div>

<div id="openclaw-command-not-found">
  ### `openclaw: command not found`
</div>

Casi siempre se debe a un problema con la ruta PATH de Node/npm. Comienza aquí:

* [Instalación (verificación de PATH de Node/npm)](/es/install#nodejs--npm-path-sanity)

<div id="installer-fails-or-you-need-full-logs">
  ### El instalador falla (o necesitas los registros completos)
</div>

Vuelve a ejecutar el instalador en modo detallado para ver la traza completa y la salida de npm:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --verbose
```

En instalaciones beta:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --beta --verbose
```

También puedes configurar `OPENCLAW_VERBOSE=1` en lugar de usar la opción.

<div id="gateway-unauthorized-cant-connect-or-keeps-reconnecting">
  ### Gateway «no autorizado», no se puede conectar o se reconecta constantemente
</div>

* [Resolución de problemas de Gateway](/es/gateway/troubleshooting)
* [Autenticación de Gateway](/es/gateway/authentication)

<div id="control-ui-fails-on-http-device-identity-required">
  ### Control UI falla con HTTP (requiere identidad del dispositivo)
</div>

* [Solución de problemas del Gateway](/es/gateway/troubleshooting)
* [Control UI](/es/web/control-ui#insecure-http)

<div id="docsopenclawai-shows-an-ssl-error-comcastxfinity">
  ### `docs.openclaw.ai` muestra un error de SSL (Comcast/Xfinity)
</div>

Algunas conexiones de Comcast/Xfinity bloquean `docs.openclaw.ai` mediante Xfinity Advanced Security.
Desactiva Advanced Security o añade `docs.openclaw.ai` a la lista de permitidos y vuelve a intentarlo.

* Ayuda de Xfinity Advanced Security: https://www.xfinity.com/support/articles/using-xfinity-xfi-advanced-security
* Comprobaciones rápidas: prueba con un punto de acceso móvil o una VPN para confirmar que se trata de filtrado a nivel de ISP

<div id="service-says-running-but-rpc-probe-fails">
  ### El servicio figura como en ejecución, pero la comprobación RPC falla
</div>

* [Resolución de problemas del Gateway](/es/gateway/troubleshooting)
* [Proceso en segundo plano / servicio](/es/gateway/background-process)

<div id="modelauth-failures-rate-limit-billing-all-models-failed">
  ### Errores de modelos/autenticación (límite de solicitudes, facturación, «fallaron todos los modelos»)
</div>

* [Modelos](/es/cli/models)
* [Conceptos de OAuth / autenticación](/es/concepts/oauth)

<div id="model-says-model-not-allowed">
  ### `/model` dice `model not allowed`
</div>

Esto suele significar que `agents.defaults.models` está configurado como una lista de permitidos. Cuando no está vacía,
solo se pueden seleccionar esas claves de proveedor/modelo.

* Revisa la lista de permitidos: `openclaw config get agents.defaults.models`
* Añade el modelo que quieres (o vacía la lista de permitidos) y vuelve a intentar `/model`
* Usa `/models` para explorar los proveedores/modelos permitidos

<div id="when-filing-an-issue">
  ### Al abrir un issue
</div>

Pega un informe seguro:

```bash
openclaw status --all
```

Si puedes, incluye las últimas líneas relevantes del log de `openclaw logs --follow`.
