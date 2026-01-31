---
title: Varios Gateways
summary: "Ejecutar varios Gateways de OpenClaw en un solo host (aislamiento, puertos y perfiles)"
read_when:
  - Ejecutas más de un Gateway en la misma máquina
  - Necesitas configuración, estado y puertos aislados para cada Gateway
---

<div id="multiple-gateways-same-host">
  # Múltiples Gateways (mismo host)
</div>

La mayoría de las instalaciones deberían usar un solo Gateway, porque un único Gateway puede gestionar múltiples conexiones de mensajería y agentes. Si necesitas un aislamiento o una redundancia mayores (por ejemplo, un bot de rescate), ejecuta Gateways separados con perfiles/puertos aislados.

<div id="isolation-checklist-required">
  ## Lista de comprobación de aislamiento (obligatoria)
</div>

- `OPENCLAW_CONFIG_PATH` — archivo de configuración por instancia
- `OPENCLAW_STATE_DIR` — sesiones, credenciales y cachés por instancia
- `agents.defaults.workspace` — raíz del espacio de trabajo por instancia
- `gateway.port` (o `--port`) — único por instancia
- Los puertos derivados (navegador/canvas) no deben solaparse

Si se comparten, aparecerán condiciones de carrera en la configuración y conflictos de puertos.

<div id="recommended-profiles-profile">
  ## Recomendado: perfiles (`--profile`)
</div>

Los perfiles asignan automáticamente el ámbito de `OPENCLAW_STATE_DIR` y `OPENCLAW_CONFIG_PATH`, y añaden un sufijo a los nombres de servicio.

```bash
# main
openclaw --profile main setup
openclaw --profile main gateway --port 18789

# rescue
openclaw --profile rescue setup
openclaw --profile rescue gateway --port 19001
```

Servicios por perfil:

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```


<div id="rescue-bot-guide">
  ## Guía de Rescue-bot
</div>

Ejecuta un segundo Gateway en el mismo host con su propio:

- perfil/config
- directorio de estado
- espacio de trabajo
- puerto base (más puertos derivados)

Esto mantiene al bot de rescate aislado del bot principal, de modo que pueda depurar o aplicar cambios de configuración si el bot primario no está disponible.

Separación de puertos: deja al menos 20 puertos entre puertos base para que los puertos derivados de navegador/canvas/CDP nunca entren en conflicto.

<div id="how-to-install-rescue-bot">
  ### Cómo instalar (bot de rescate)
</div>

```bash
# Main bot (existing or fresh, without --profile param)
# Runs on port 18789 + Chrome CDC/Canvas/... Ports 
openclaw onboard
openclaw gateway install

# Rescue bot (isolated profile + ports)
openclaw --profile rescue onboard
# Notas: 
# - el nombre del espacio de trabajo tendrá el sufijo -rescue por defecto
# - El puerto debe ser al menos 18789 + 20 puertos, 
#   es mejor elegir un puerto base completamente diferente, como 19789,
# - el resto del proceso de incorporación es el mismo que el normal

# To install the service (if not happened automatically during onboarding)
openclaw --profile rescue gateway install
```


<div id="port-mapping-derived">
  ## Mapeo de puertos (derivado)
</div>

Puerto base = `gateway.port` (o `OPENCLAW_GATEWAY_PORT` / `--port`).

- puerto del servicio de control del navegador = base + 2 (solo loopback/local)
- `canvasHost.port = base + 4`
- Los puertos CDP del perfil del navegador se asignan automáticamente desde `browser.controlPort + 9 .. + 108`

Si modificas alguno de estos en la configuración o en el entorno, debes asegurarte de que sigan siendo únicos por instancia.

<div id="browsercdp-notes-common-footgun">
  ## Notas sobre el navegador/CDP (fuente habitual de problemas)
</div>

- **No** configures `browser.cdpUrl` con los mismos valores en varias instancias.
- Cada instancia necesita su propio puerto de control del navegador y su propio rango de CDP (derivado de su puerto de Gateway).
- Si necesitas puertos de CDP explícitos, establece `browser.profiles.<name>.cdpPort` por instancia.
- Chrome remoto: usa `browser.profiles.<name>.cdpUrl` (por perfil, por instancia).

<div id="manual-env-example">
  ## Ejemplo de configuración manual del entorno
</div>

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/main.json \
OPENCLAW_STATE_DIR=~/.openclaw-main \
openclaw gateway --port 18789

OPENCLAW_CONFIG_PATH=~/.openclaw/rescue.json \
OPENCLAW_STATE_DIR=~/.openclaw-rescue \
openclaw gateway --port 19001
```


<div id="quick-checks">
  ## Verificaciones rápidas
</div>

```bash
openclaw --profile main status
openclaw --profile rescue status
openclaw --profile rescue browser status
```
