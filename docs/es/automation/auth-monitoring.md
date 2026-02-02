---
title: Supervisión de autenticación
summary: "Supervisa la caducidad de OAuth para proveedores de modelos"
read_when:
  - Configurar la supervisión o las alertas de caducidad de OAuth
  - Automatizar las comprobaciones de renovación de OAuth de Claude Code / Codex
---

<div id="auth-monitoring">
  # Monitorización de autenticación
</div>

OpenClaw expone el estado de expiración de OAuth mediante `openclaw models status`. Úsalo para
automatización y alertas; los scripts son complementos opcionales para flujos de trabajo en el teléfono.

<div id="preferred-cli-check-portable">
  ## Opción recomendada: comprobación mediante CLI (portátil)
</div>

```bash
openclaw models status --check
```

Códigos de salida:

* `0`: OK
* `1`: credenciales caducadas o faltantes
* `2`: caducan pronto (en menos de 24 h)

Funciona con cron/systemd y no requiere scripts adicionales.


<div id="optional-scripts-ops-phone-workflows">
  ## Scripts opcionales (operaciones / flujos en teléfono)
</div>

Estos se encuentran en `scripts/` y son **opcionales**. Suponen acceso SSH al
host del Gateway y están ajustados para systemd + Termux.

- `scripts/claude-auth-status.sh` ahora usa `openclaw models status --json` como
  fuente de referencia (recurriendo a lecturas directas de archivos si la CLI no está disponible),
  así que mantén `openclaw` en el `PATH` para los temporizadores.
- `scripts/auth-monitor.sh`: objetivo de temporizador cron/systemd; envía alertas (ntfy o teléfono).
- `scripts/systemd/openclaw-auth-monitor.{service,timer}`: temporizador de usuario systemd.
- `scripts/claude-auth-status.sh`: comprobador de autenticación de Claude Code + OpenClaw (completo/json/simple).
- `scripts/mobile-reauth.sh`: flujo guiado de reautenticación vía SSH.
- `scripts/termux-quick-auth.sh`: widget de un toque para estado + abrir URL de autenticación.
- `scripts/termux-auth-widget.sh`: flujo completo guiado mediante widget.
- `scripts/termux-sync-widget.sh`: sincroniza credenciales de Claude Code → OpenClaw.

Si no necesitas automatización en el teléfono o temporizadores systemd, omite estos scripts.