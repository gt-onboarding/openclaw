---
title: Migración
summary: "Transferir (migrar) una instalación de OpenClaw de una máquina a otra"
read_when:
  - Estás migrando OpenClaw a un nuevo equipo o servidor
  - Quieres preservar sesiones, autenticación e inicios de sesión de canales (WhatsApp, etc.)
---

<div id="migrating-openclaw-to-a-new-machine">
  # Migrar OpenClaw a una nueva máquina
</div>

Esta guía explica cómo migrar un Gateway de OpenClaw de una máquina a otra **sin repetir el onboarding**.

Conceptualmente, la migración es sencilla:

* Copia el **directorio de estado** (`$OPENCLAW_STATE_DIR`, predeterminado: `~/.openclaw/`) — esto incluye configuración, autenticación, sesiones y estado de los canales.
* Copia tu **espacio de trabajo** (`~/.openclaw/workspace/` por defecto) — esto incluye tus archivos de agente (memoria, prompts, etc.).

Pero hay puntos problemáticos frecuentes relacionados con **perfiles**, **permisos** y **copias parciales**.

<div id="before-you-start-what-you-are-migrating">
  ## Antes de comenzar (qué vas a migrar)
</div>

<div id="1-identify-your-state-directory">
  ### 1) Identifica tu directorio de estado
</div>

La mayoría de las instalaciones usan el valor predeterminado:

* **State dir:** `~/.openclaw/`

Pero puede ser diferente si usas:

* `--profile <name>` (a menudo pasa a ser `~/.openclaw-<profile>/`)
* `OPENCLAW_STATE_DIR=/some/path`

Si no estás seguro, ejecútalo en la máquina **anterior**:

```bash
openclaw status
```

Busca menciones a `OPENCLAW_STATE_DIR` o al perfil en la salida. Si ejecutas varios Gateways, repite el proceso para cada perfil.

<div id="2-identify-your-workspace">
  ### 2) Identifica tu espacio de trabajo
</div>

Valores predeterminados habituales:

* `~/.openclaw/workspace/` (espacio de trabajo recomendado)
* una carpeta personalizada que creaste

Tu espacio de trabajo es donde se encuentran archivos como `MEMORY.md`, `USER.md` y `memory/*.md`.

<div id="3-understand-what-you-will-preserve">
  ### 3) Comprende qué vas a conservar
</div>

Si copias **tanto** el directorio de estado como el espacio de trabajo, conservarás:

* Configuración del Gateway (`openclaw.json`)
* Perfiles de autenticación / claves API / tokens OAuth
* Historial de sesiones + estado del agente
* Estado de los canales (p. ej., inicio de sesión/sesión de WhatsApp)
* Tus archivos del espacio de trabajo (memoria, notas de habilidades, etc.)

Si copias **solo** el espacio de trabajo (p. ej., mediante Git), **no** conservarás:

* sesiones
* credenciales
* inicios de sesión de canales

Esos datos se encuentran en `$OPENCLAW_STATE_DIR`.

<div id="migration-steps-recommended">
  ## Pasos de migración (recomendados)
</div>

<div id="step-0-make-a-backup-old-machine">
  ### Paso 0 — Realiza una copia de seguridad (máquina antigua)
</div>

En la máquina **antigua**, detén primero el Gateway para que los archivos no cambien durante la copia:

```bash
openclaw gateway stop
```

(Opcional pero recomendado) archivar el directorio de estado y el espacio de trabajo:

```bash
# Ajusta las rutas si utilizas un perfil o ubicaciones personalizadas
cd ~
tar -czf openclaw-state.tgz .openclaw

tar -czf openclaw-workspace.tgz .openclaw/workspace
```

Si tienes varios perfiles o directorios de estado (por ejemplo, `~/.openclaw-main`, `~/.openclaw-work`), crea un archivo de cada uno.

<div id="step-1-install-openclaw-on-the-new-machine">
  ### Paso 1 — Instalar OpenClaw en la nueva máquina
</div>

En la máquina **nueva**, instala la CLI (y Node.js si es necesario):

* Consulta: [Instalación](/es/install)

En esta etapa, no hay problema si el proceso de onboarding crea un `~/.openclaw/` nuevo: lo sobrescribirás en el siguiente paso.

<div id="step-2-copy-the-state-dir-workspace-to-the-new-machine">
  ### Paso 2 — Copia el directorio de estado y el espacio de trabajo a la nueva máquina
</div>

Copia **ambos**:

* `$OPENCLAW_STATE_DIR` (predeterminado `~/.openclaw/`)
* tu espacio de trabajo (predeterminado `~/.openclaw/workspace/`)

Enfoques habituales:

* usar `scp` para copiar los archivos tar y extraerlos
* `rsync -a` por SSH
* unidad de almacenamiento externa

Después de copiar, asegúrate de lo siguiente:

* Que se hayan incluido los directorios ocultos (por ejemplo, `.openclaw/`)
* Que la propiedad de los archivos sea correcta para el usuario que ejecuta el Gateway

<div id="step-3-run-doctor-migrations-service-repair">
  ### Paso 3 — Ejecutar Doctor (migraciones + reparación de servicios)
</div>

En la **nueva** máquina:

```bash
openclaw doctor
```

Doctor es el comando “seguro pero aburrido”. Repara servicios, aplica migraciones de configuración y advierte sobre incongruencias.

Luego:

```bash
openclaw gateway restart
openclaw status
```

<div id="common-footguns-and-how-to-avoid-them">
  ## Trampas habituales (y cómo evitarlas)
</div>

<div id="footgun-profile-state-dir-mismatch">
  ### Footgun: desajuste entre perfil y state-dir
</div>

Si ejecutaste el Gateway antiguo con un perfil (o `OPENCLAW_STATE_DIR`), y el nuevo Gateway usa uno diferente, verás síntomas como:

* cambios de configuración que no se aplican
* canales ausentes / desconectados
* historial de sesiones vacío

Solución: ejecuta el Gateway/servicio usando el **mismo** perfil/directorio de estado que migraste y luego vuelve a ejecutar:

```bash
openclaw doctor
```

<div id="footgun-copying-only-openclawjson">
  ### Error común: copiar solo `openclaw.json`
</div>

`openclaw.json` no basta. Muchos proveedores almacenan el estado en:

* `$OPENCLAW_STATE_DIR/credentials/`
* `$OPENCLAW_STATE_DIR/agents/<agentId>/...`

Migra siempre la carpeta completa `$OPENCLAW_STATE_DIR`.

<div id="footgun-permissions-ownership">
  ### Trampa: permisos / propiedad
</div>

Si copiaste como root o cambiaste de usuario, el Gateway podría no poder leer las credenciales/sesiones.

Solución: asegúrate de que el directorio de estado y el espacio de trabajo pertenezcan al usuario que ejecuta el Gateway.

<div id="footgun-migrating-between-remotelocal-modes">
  ### Footgun: migrar entre modos remoto/local
</div>

* Si tu UI (WebUI/TUI) apunta a un **Gateway remoto**, el host remoto es el propietario del almacén de sesiones y del espacio de trabajo.
* Migrar tu portátil no trasladará el estado del Gateway remoto.

Si estás en modo remoto, migra el **host del Gateway**.

<div id="footgun-secrets-in-backups">
  ### Peligro: secretos en copias de seguridad
</div>

`$OPENCLAW_STATE_DIR` contiene secretos (claves de API, tokens de OAuth, credenciales de WhatsApp). Trata las copias de seguridad como secretos de producción:

* almacénalas cifradas
* evita compartirlas por canales inseguros
* rota las claves si sospechas que se han expuesto

<div id="verification-checklist">
  ## Lista de verificación
</div>

En la nueva máquina, verifica:

* `openclaw status` muestra el Gateway en ejecución
* Tus canales siguen conectados (p. ej., WhatsApp no necesita que la vuelvas a vincular)
* El panel de control se abre y muestra las sesiones existentes
* Tus archivos de espacio de trabajo (memory, configuración) están presentes

<div id="related">
  ## Temas relacionados
</div>

* [Doctor](/es/gateway/doctor)
* [Solución de problemas del Gateway](/es/gateway/troubleshooting)
* [¿Dónde almacena OpenClaw sus datos?](/es/help/faq#where-does-openclaw-store-its-data)