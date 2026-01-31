---
title: Configuración de desarrollo
summary: "Guía de configuración para desarrolladores que trabajan en la app de OpenClaw para macOS"
read_when:
  - Configurar el entorno de desarrollo en macOS
---

<div id="macos-developer-setup">
  # Configuración para desarrolladores en macOS
</div>

Esta guía describe los pasos necesarios para compilar y ejecutar la aplicación de OpenClaw para macOS desde el código fuente.

<div id="prerequisites">
  ## Requisitos previos
</div>

Antes de compilar la aplicación, asegúrate de tener instalados lo siguiente:

1.  **Xcode 26.2+**: Requerido para el desarrollo en Swift.
2.  **Node.js 22+ & pnpm**: Requeridos para el Gateway, la CLI y los scripts de empaquetado.

<div id="1-install-dependencies">
  ## 1. Instalar dependencias
</div>

Instala las dependencias globales del proyecto:

```bash
pnpm install
```


<div id="2-build-and-package-the-app">
  ## 2. Compila y empaqueta la aplicación
</div>

Para compilar la aplicación de macOS y empaquetarla en `dist/OpenClaw.app`, ejecuta:

```bash
./scripts/package-mac-app.sh
```

Si no tienes un certificado de Apple Developer ID, el script usará automáticamente **firma ad-hoc** (`-`).

Para modos de ejecución de desarrollo, opciones de firma y resolución de problemas con el Team ID, consulta el README de la app de macOS:
https://github.com/openclaw/openclaw/blob/main/apps/macos/README.md

> **Nota**: Las aplicaciones firmadas ad-hoc pueden generar avisos de seguridad. Si la aplicación se bloquea inmediatamente con &quot;Abort trap 6&quot;, consulta la sección de [Solución de problemas](#troubleshooting).


<div id="3-install-the-cli">
  ## 3. Instala la CLI
</div>

La app de macOS requiere una instalación global de la CLI `openclaw` para gestionar tareas en segundo plano.

**Para instalarla (recomendado):**

1. Abre la app de OpenClaw.
2. Ve a la pestaña de ajustes **General**.
3. Haz clic en **&quot;Install CLI&quot;**.

Como alternativa, instálala manualmente:

```bash
npm install -g openclaw@<version>
```


<div id="troubleshooting">
  ## Solución de problemas
</div>

<div id="build-fails-toolchain-or-sdk-mismatch">
  ### Error de compilación: incompatibilidad entre toolchain y SDK
</div>

La compilación de la app de macOS requiere el SDK más reciente de macOS y la toolchain de Swift 6.2.

**Dependencias del sistema (obligatorias):**

* **Versión más reciente de macOS disponible en Actualización de Software** (requerida por los SDK de Xcode 26.2)
* **Xcode 26.2** (toolchain de Swift 6.2)

**Comprobaciones:**

```bash
xcodebuild -version
xcrun swift --version
```

Si las versiones no coinciden, actualiza macOS/Xcode y vuelve a compilar.


<div id="app-crashes-on-permission-grant">
  ### La app se bloquea al conceder permisos
</div>

Si la app se bloquea cuando intentas conceder acceso a **Speech Recognition** o al **Microphone**, puede deberse a una caché TCC dañada o a una incompatibilidad de firma.

**Solución:**

1. Restablece los permisos de TCC:
   ```bash
   tccutil reset All bot.molt.mac.debug
   ```
2. Si eso falla, cambia temporalmente el `BUNDLE_ID` en [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) para forzar un "estado limpio" en macOS.

<div id="gateway-starting-indefinitely">
  ### Gateway &quot;Iniciando...&quot; indefinidamente
</div>

Si el estado del Gateway permanece en &quot;Iniciando...&quot;, comprueba si hay algún proceso zombi que esté ocupando el puerto:

```bash
openclaw gateway status
openclaw gateway stop

# Si no usas un LaunchAgent (modo dev / ejecuciones manuales), busca el listener:
lsof -nP -iTCP:18789 -sTCP:LISTEN
```

Si una ejecución manual está usando el puerto, detén ese proceso con Ctrl+C. Como último recurso, mata el proceso con el PID que encontraste arriba.
