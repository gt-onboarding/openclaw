---
title: Doctor
summary: "Referencia de la CLI de `openclaw doctor` (comprobaciones de estado + reparaciones guiadas)"
read_when:
  - Tienes problemas de conectividad/autenticación y quieres soluciones guiadas
  - Has actualizado y quieres hacer una comprobación rápida
---

<div id="openclaw-doctor">
  # `openclaw doctor`
</div>

Comprobaciones de estado y soluciones rápidas para el Gateway y los canales.

Relacionado:

* Resolución de problemas: [Resolución de problemas](/es/gateway/troubleshooting)
* Auditoría de seguridad: [Seguridad](/es/gateway/security)

<div id="examples">
  ## Ejemplos
</div>

```bash
openclaw doctor
openclaw doctor --repair
openclaw doctor --deep
```

Notas:

* Los prompts interactivos (como correcciones de llavero/OAuth) solo se ejecutan cuando stdin es un TTY y `--non-interactive` **no** está establecido. Las ejecuciones sin interacción (cron, Telegram, sin terminal) omitirán estos prompts.
* `--fix` (alias de `--repair`) crea una copia de seguridad en `~/.openclaw/openclaw.json.bak` y elimina las claves de configuración desconocidas, mostrando cada eliminación.

<div id="macos-launchctl-env-overrides">
  ## macOS: `launchctl` anulaciones de variables de entorno
</div>

Si ejecutaste anteriormente `launchctl setenv OPENCLAW_GATEWAY_TOKEN ...` (o `...PASSWORD`), ese valor tendrá prioridad sobre tu archivo de configuración y puede generar errores persistentes de «no autorizado».

```bash
launchctl getenv OPENCLAW_GATEWAY_TOKEN
launchctl getenv OPENCLAW_GATEWAY_PASSWORD

launchctl unsetenv OPENCLAW_GATEWAY_TOKEN
launchctl unsetenv OPENCLAW_GATEWAY_PASSWORD
```
