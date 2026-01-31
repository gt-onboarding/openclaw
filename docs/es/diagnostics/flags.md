---
title: Flags
summary: "Flags de diagnóstico para logs de depuración específicos"
read_when:
  - Necesitas logs de depuración específicos sin aumentar los niveles globales de logging
  - Necesitas capturar logs específicos de subsistemas para soporte
---

<div id="diagnostics-flags">
  # Banderas de diagnóstico
</div>

Las banderas de diagnóstico te permiten habilitar registros de depuración específicos sin activar el registro detallado en todas partes. Las banderas son opcionales y no tienen efecto a menos que un subsistema las consulte.

<div id="how-it-works">
  ## Cómo funciona
</div>

* Los flags son cadenas de texto (no distinguen entre mayúsculas y minúsculas).
* Puedes activar flags en la configuración o mediante una variable de entorno de sobrescritura (env override).
* Se admiten comodines:
  * `telegram.*` coincide con `telegram.http`
  * `*` activa todos los flags

<div id="enable-via-config">
  ## Habilitar desde la configuración
</div>

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

Múltiples flags:

```json
{
  "diagnostics": {
    "flags": ["telegram.http", "gateway.*"]
  }
}
```

Reinicia el Gateway después de modificar las flags.

<div id="env-override-one-off">
  ## Sobrescritura de entorno (única vez)
</div>

```bash
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload
```

Desactivar todos los flags:

```bash
OPENCLAW_DIAGNOSTICS=0
```

<div id="where-logs-go">
  ## Dónde se guardan los logs
</div>

Las flags escriben logs en el archivo estándar de diagnósticos. De forma predeterminada:

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

Si configuras `logging.file`, utiliza esa ruta en su lugar. Los registros son JSONL (un objeto JSON por línea). El enmascaramiento de datos sensibles sigue aplicándose según `logging.redactSensitive`.

<div id="extract-logs">
  ## Obtener registros
</div>

Selecciona el archivo de registro más reciente:

```bash
ls -t /tmp/openclaw/openclaw-*.log | head -n 1
```

Filtro para diagnósticos HTTP de Telegram:

```bash
rg "telegram http error" /tmp/openclaw/openclaw-*.log
```

O ejecuta tail mientras reproduces el problema:

```bash
tail -f /tmp/openclaw/openclaw-$(date +%F).log | rg "telegram http error"
```

En gateways remotos, también puedes usar `openclaw logs --follow` (consulta [/cli/logs](/es/cli/logs)).

<div id="notes">
  ## Notas
</div>

* Si `logging.level` se establece en un nivel superior a `warn`, es posible que estos logs se supriman. El valor predeterminado `info` es adecuado.
* Es seguro dejar los flags habilitados; solo afectan el volumen de logs del subsistema específico.
* Usa [/logging](/es/logging) para cambiar destinos de log, niveles y enmascaramiento.