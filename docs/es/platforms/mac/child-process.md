---
title: Proceso hijo
summary: "Ciclo de vida del Gateway en macOS (launchd)"
read_when:
  - Integrar la aplicación de macOS con el ciclo de vida del Gateway
---

<div id="gateway-lifecycle-on-macos">
  # Ciclo de vida del Gateway en macOS
</div>

La app de macOS **administra el Gateway mediante launchd** de forma predeterminada y no lanza
el Gateway como proceso hijo. Primero intenta conectarse a un Gateway ya en ejecución
en el puerto configurado; si no hay ninguno accesible, habilita el servicio launchd
mediante la CLI `openclaw` externa (sin runtime incrustado). Esto te proporciona
un arranque automático fiable al iniciar sesión y reinicio en caso de fallos.

El modo de proceso hijo (Gateway lanzado directamente por la app) **no se utiliza** actualmente.
Si necesitas un acoplamiento más estrecho con la UI, ejecuta el Gateway manualmente en una terminal.

<div id="default-behavior-launchd">
  ## Comportamiento predeterminado (launchd)
</div>

* La app instala un LaunchAgent por usuario etiquetado como `bot.molt.gateway`
  (o `bot.molt.<profile>` cuando se usa `--profile`/`OPENCLAW_PROFILE`; se admite el esquema heredado `com.openclaw.*`).
* Cuando el modo Local está habilitado, la app garantiza que el LaunchAgent esté cargado y
  inicia el Gateway si es necesario.
* Los registros se escriben en la ruta de log del Gateway en launchd (visible en Debug Settings).

Comandos comunes:

```bash
launchctl kickstart -k gui/$UID/bot.molt.gateway
launchctl bootout gui/$UID/bot.molt.gateway
```

Sustituye la etiqueta por `bot.molt.&lt;profile&gt;` al ejecutar un perfil con nombre.


<div id="unsigned-dev-builds">
  ## Compilaciones de desarrollo sin firmar
</div>

`scripts/restart-mac.sh --no-sign` es para compilaciones locales rápidas cuando no tienes
claves de firma de código. Para evitar que launchd apunte a un binario de relay sin firmar, hace lo siguiente:

* Escribe `~/.openclaw/disable-launchagent`.

Las ejecuciones firmadas de `scripts/restart-mac.sh` eliminan esta anulación si el marcador
está presente. Para restablecerlo manualmente:

```bash
rm ~/.openclaw/disable-launchagent
```


<div id="attach-only-mode">
  ## Modo solo adjuntar
</div>

Para forzar que la aplicación de macOS **no instale ni gestione nunca launchd**, ejecútala con
`--attach-only` (o `--no-launchd`). Esto establece `~/.openclaw/disable-launchagent`,
de modo que la aplicación solo se conecta a un Gateway que ya esté en ejecución. Puedes activar o desactivar el mismo
comportamiento en Debug Settings.

<div id="remote-mode">
  ## Modo remoto
</div>

El modo remoto nunca inicia un Gateway local. La aplicación usa un túnel SSH al
host remoto y se conecta a través de ese túnel.

<div id="why-we-prefer-launchd">
  ## Por qué preferimos launchd
</div>

- Inicio automático al iniciar sesión.
- Mecanismos integrados de reinicio/KeepAlive.
- Registros y supervisión predecibles.

Si alguna vez se volviera a necesitar un modo de proceso hijo real, debería documentarse como un modo separado, explícito y solo para desarrollo.