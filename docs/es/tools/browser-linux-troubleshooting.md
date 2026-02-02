---
title: Solución de problemas del navegador en Linux
summary: "Resuelve problemas de inicio de CDP en Chrome, Brave, Edge y Chromium para el control del navegador de OpenClaw en Linux"
read_when: "El control del navegador falla en Linux, especialmente con Chromium instalado mediante snap"
---

<div id="browser-troubleshooting-linux">
  # Resolución de problemas del navegador (Linux)
</div>

<div id="problem-failed-to-start-chrome-cdp-on-port-18800">
  ## Problema: &quot;Failed to start Chrome CDP on port 18800&quot;
</div>

El servidor de control del navegador de OpenClaw no consigue iniciar Chrome/Brave/Edge/Chromium y devuelve el error:

```
{"error":"Error: Failed to start Chrome CDP on port 18800 for profile \"openclaw\"."}
```


<div id="root-cause">
  ### Causa raíz
</div>

En Ubuntu (y en muchas distribuciones de Linux), la instalación predeterminada de Chromium es un **paquete snap**. El confinamiento de AppArmor de snap interfiere con la forma en que OpenClaw inicia y supervisa el proceso del navegador.

El comando `apt install chromium` instala un paquete stub que simplemente redirige a snap:

```
Note, selecting 'chromium-browser' instead of 'chromium'
chromium-browser is already the newest version (2:1snap1-0ubuntu2).
```

Esto NO es un navegador real — solo es un envoltorio.


<div id="solution-1-install-google-chrome-recommended">
  ### Solución 1: Instalar Google Chrome (recomendado)
</div>

Instala el paquete oficial `.deb` de Google Chrome, que no se ejecuta en un sandbox de snap:

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt --fix-broken install -y  # si hay errores de dependencias
```

A continuación, actualiza la configuración de OpenClaw (`~/.openclaw/openclaw.json`):

```json
{
  "browser": {
    "enabled": true,
    "executablePath": "/usr/bin/google-chrome-stable",
    "headless": true,
    "noSandbox": true
  }
}
```


<div id="solution-2-use-snap-chromium-with-attach-only-mode">
  ### Solución 2: usar Chromium de Snap con modo attach-only
</div>

Si necesitas usar Chromium instalado mediante Snap, configura OpenClaw para que se conecte a un navegador iniciado manualmente:

1. Actualiza la configuración:

```json
{
  "browser": {
    "enabled": true,
    "attachOnly": true,
    "headless": true,
    "noSandbox": true
  }
}
```

2. Inicia Chromium manualmente:

```bash
chromium-browser --headless --no-sandbox --disable-gpu \
  --remote-debugging-port=18800 \
  --user-data-dir=$HOME/.openclaw/browser/openclaw/user-data \
  about:blank &
```

3. Opcionalmente, crea un servicio de usuario de systemd para iniciar Chrome automáticamente:

```ini
# ~/.config/systemd/user/openclaw-browser.service
[Unit]
Description=Navegador de OpenClaw (Chrome CDP)
After=network.target

[Service]
ExecStart=/snap/bin/chromium --headless --no-sandbox --disable-gpu --remote-debugging-port=18800 --user-data-dir=%h/.openclaw/browser/openclaw/user-data about:blank
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

Actívalo con: `systemctl --user enable --now openclaw-browser.service`


<div id="verifying-the-browser-works">
  ### Comprobar que el navegador funciona correctamente
</div>

Comprueba el estado:

```bash
curl -s http://127.0.0.1:18791/ | jq '{running, pid, chosenBrowser}'
```

Prueba la navegación:

```bash
curl -s -X POST http://127.0.0.1:18791/start
curl -s http://127.0.0.1:18791/tabs
```


<div id="config-reference">
  ### Referencia de configuración
</div>

| Option | Description | Default |
|--------|-------------|---------|
| `browser.enabled` | Habilitar control del navegador | `true` |
| `browser.executablePath` | Ruta al binario de un navegador basado en Chromium (Chrome/Brave/Edge/Chromium) | detección automática (prefiere el navegador predeterminado cuando es basado en Chromium) |
| `browser.headless` | Ejecutar sin UI gráfica | `false` |
| `browser.noSandbox` | Agregar el flag `--no-sandbox` (necesario para algunas configuraciones de Linux) | `false` |
| `browser.attachOnly` | No iniciar el navegador, solo conectarse a uno existente | `false` |
| `browser.cdpPort` | Puerto del Chrome DevTools Protocol | `18800` |

<div id="problem-chrome-extension-relay-is-running-but-no-tab-is-connected">
  ### Problema: "Chrome extension relay is running, but no tab is connected"
</div>

Estás usando el perfil `chrome` (extension relay). Este perfil espera que la extensión
de navegador de OpenClaw esté vinculada a una pestaña activa.

Opciones para solucionarlo:

1. **Usar el navegador gestionado:** `openclaw browser start --browser-profile openclaw`
   (o establece `browser.defaultProfile: "openclaw"`).
2. **Usar el extension relay:** instala la extensión, abre una pestaña y haz clic
   en el icono de la extensión de OpenClaw para vincularla.

Notas:

- El perfil `chrome` usa tu **navegador Chromium predeterminado del sistema** cuando es posible.
- Los perfiles locales de `openclaw` asignan automáticamente `cdpPort`/`cdpUrl`; solo configúralos para un CDP remoto.