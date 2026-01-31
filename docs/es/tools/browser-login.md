---
title: Inicio de sesión en el navegador
summary: "Inicios de sesión manuales para automatización del navegador y publicaciones en X/Twitter"
read_when:
  - Necesitas iniciar sesión en sitios web para la automatización del navegador
  - Quieres publicar actualizaciones en X/Twitter
---

<div id="browser-login-xtwitter-posting">
  # Inicio de sesión desde el navegador + publicación en X/Twitter
</div>

<div id="manual-login-recommended">
  ## Inicio de sesión manual (recomendado)
</div>

Cuando un sitio requiera que inicies sesión, **inicia sesión manualmente** en el perfil del navegador **host** (el navegador de openclaw).

**No** le des tus credenciales al modelo. Los inicios de sesión automatizados suelen activar defensas anti‑bot y pueden bloquear la cuenta.

Volver a la documentación principal del navegador: [Browser](/es/tools/browser).

<div id="which-chrome-profile-is-used">
  ## ¿Qué perfil de Chrome se utiliza?
</div>

OpenClaw controla un **perfil de Chrome dedicado** (llamado `openclaw`, UI con tono anaranjado). Este es independiente de tu perfil de navegador diario.

Dos formas sencillas de acceder a él:

1. **Pídele al agente que abra el navegador** y luego inicia sesión tú mismo.
2. **Ábrelo mediante la CLI**:

```bash
openclaw browser start
openclaw browser open https://x.com
```

Si tienes varios perfiles, proporciona `--browser-profile <name>` (el valor predeterminado es `openclaw`).

<div id="xtwitter-recommended-flow">
  ## X/Twitter: flujo recomendado
</div>

* **Leer/buscar/hilos:** usa la skill de la CLI **bird** (sin navegador, estable).
  * Repositorio: https://github.com/steipete/bird
* **Publicar actualizaciones:** usa el navegador **host** (inicio de sesión manual).

<div id="sandboxing-host-browser-access">
  ## Sandbox + acceso al navegador del host
</div>

Las sesiones de navegador en sandbox tienen **más probabilidades** de activar la detección de bots. Para X/Twitter (y otros sitios estrictos), da preferencia al navegador del **host**.

Si el agente está en sandbox, la herramienta de navegador usa por defecto la sandbox. Para permitir el control desde el host:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        browser: {
          allowHostControl: true
        }
      }
    }
  }
}
```

A continuación, apunta al navegador del host:

```bash
openclaw browser open https://x.com --browser-profile openclaw --target host
```

O bien desactiva la sandbox para el agente que publica actualizaciones.
