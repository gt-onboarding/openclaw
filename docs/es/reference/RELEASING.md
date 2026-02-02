---
title: LANZAMIENTO
summary: "Lista de verificación paso a paso para lanzamientos en npm y en la app de macOS"
read_when:
  - Al preparar un nuevo lanzamiento de npm
  - Al preparar un nuevo lanzamiento de la app de macOS
  - Al verificar los metadatos antes de publicar
---

<div id="release-checklist-npm-macos">
  # Lista de comprobación de lanzamiento (npm + macOS)
</div>

Usa `pnpm` (Node 22+) desde la raíz del repositorio. Mantén el árbol de trabajo limpio antes de crear la etiqueta/publicar.

<div id="operator-trigger">
  ## Activador del operador
</div>

Cuando el operador diga “release”, ejecuta inmediatamente este preflight (sin preguntas adicionales a menos que estés bloqueado):

* Lee este documento y `docs/platforms/mac/release.md`.
* Carga el entorno desde `~/.profile` y confirma que `SPARKLE_PRIVATE_KEY_FILE` y las variables de App Store Connect estén configuradas (`SPARKLE_PRIVATE_KEY_FILE` debería vivir en `~/.profile`).
* Usa las claves de Sparkle de `~/Library/CloudStorage/Dropbox/Backup/Sparkle` si es necesario.

1. **Versión y metadatos**

* [ ] Aumenta la versión de `package.json` (p. ej., `2026.1.29`).
* [ ] Ejecuta `pnpm plugins:sync` para alinear las versiones de los paquetes de extensiones y los changelogs.
* [ ] Actualiza las cadenas de versión de la CLI: [`src/cli/program.ts`](https://github.com/openclaw/openclaw/blob/main/src/cli/program.ts) y el user agent de Baileys en [`src/provider-web.ts`](https://github.com/openclaw/openclaw/blob/main/src/provider-web.ts).
* [ ] Confirma los metadatos del paquete (name, description, repository, keywords, license) y que el mapa `bin` apunte a [`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs) para `openclaw`.
* [ ] Si cambiaron las dependencias, ejecuta `pnpm install` para que `pnpm-lock.yaml` esté actualizado.

2. **Build y artefactos**

* [ ] Si cambiaron las entradas de A2UI, ejecuta `pnpm canvas:a2ui:bundle` y haz commit de cualquier [`src/canvas-host/a2ui/a2ui.bundle.js`](https://github.com/openclaw/openclaw/blob/main/src/canvas-host/a2ui/a2ui.bundle.js) actualizado.
* [ ] `pnpm run build` (regenera `dist/`).
* [ ] Verifica que la lista `files` del paquete de npm incluya todas las carpetas `dist/*` requeridas (en particular `dist/node-host/**` y `dist/acp/**` para el nodo en modo headless y la CLI de ACP).
* [ ] Confirma que `dist/build-info.json` exista e incluya el hash de `commit` esperado (la CLI lo usa en el banner para instalaciones vía npm).
* [ ] Opcional: `npm pack --pack-destination /tmp` después del build; inspecciona el contenido del tarball y tenlo a mano para el release de GitHub (no lo hagas **commit**).

3. **Changelog y documentación**

* [ ] Actualiza `CHANGELOG.md` con los puntos destacados de cara al usuario (créalo si no existe); mantén las entradas estrictamente en orden descendente por versión.
* [ ] Asegúrate de que los ejemplos/flags del README coincidan con el comportamiento actual de la CLI (en particular nuevos comandos u opciones).

4. **Validación**

* [ ] `pnpm lint`
* [ ] `pnpm test` (o `pnpm test:coverage` si necesitas el informe de cobertura)
* [ ] `pnpm run build` (última verificación de coherencia después de los tests)
* [ ] `pnpm release:check` (verifica el contenido de npm pack)
* [ ] `OPENCLAW_INSTALL_SMOKE_SKIP_NONROOT=1 pnpm test:install:smoke` (prueba de humo de instalación en Docker, ruta rápida; obligatoria antes del release)
  * Si se sabe que el release inmediato anterior de npm está roto, configura `OPENCLAW_INSTALL_SMOKE_PREVIOUS=<last-good-version>` o `OPENCLAW_INSTALL_SMOKE_SKIP_PREVIOUS=1` para el paso de preinstalación.
* [ ] (Opcional) Prueba de humo completa del instalador (añade cobertura para usuario no root + CLI): `pnpm test:install:smoke`
* [ ] (Opcional) E2E del instalador (Docker, ejecuta `curl -fsSL https://openclaw.bot/install.sh | bash`, hace el onboarding y luego ejecuta llamadas reales a herramientas):
  * `pnpm test:install:e2e:openai` (requiere `OPENAI_API_KEY`)
  * `pnpm test:install:e2e:anthropic` (requiere `ANTHROPIC_API_KEY`)
  * `pnpm test:install:e2e` (requiere ambas claves; ejecuta ambos proveedores)
* [ ] (Opcional) Haz una comprobación puntual del Gateway web si tus cambios afectan las rutas de envío/recepción.

5. **App de macOS (Sparkle)**

* [ ] Compila y firma la app para macOS, luego comprímela en un zip para su distribución.
* [ ] Genera el appcast de Sparkle (notas HTML mediante [`scripts/make_appcast.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/make_appcast.sh)) y actualiza `appcast.xml`.
* [ ] Ten listo el zip de la app (y el zip dSYM opcional) para adjuntarlo a la release de GitHub.
* [ ] Sigue [macOS release](/es/platforms/mac/release) para ver los comandos exactos y las variables de entorno requeridas.
  * `APP_BUILD` debe ser numérico y estrictamente creciente (sin `-beta`) para que Sparkle compare las versiones correctamente.
  * Si vas a notarizar, usa el perfil de llavero `openclaw-notary` creado a partir de las variables de entorno de la API de App Store Connect (consulta [macOS release](/es/platforms/mac/release)).

6. **Publicar (npm)**

* [ ] Confirma que el estado de git esté limpio; haz commit y push según sea necesario.
* [ ] Ejecuta `npm login` (verifica el 2FA) si es necesario.
* [ ] Ejecuta `npm publish --access public` (usa `--tag beta` para versiones preliminares).
* [ ] Verifica el registry de npm: `npm view openclaw version`, `npm view openclaw dist-tags` y `npx -y openclaw@X.Y.Z --version` (o `--help`).

<div id="troubleshooting-notes-from-200-beta2-release">
  ### Solución de problemas (notas de la versión 2.0.0-beta2)
</div>

* **`npm pack/publish` se queda colgado o produce un tarball enorme**: el bundle de la app de macOS en `dist/OpenClaw.app` (y los zips de la versión) queda incluido en el paquete. Soluciona esto definiendo explícitamente el contenido a publicar mediante `files` en `package.json` (incluye subdirectorios de `dist`, documentación, habilidades; excluye los bundles de aplicaciones). Confirma con `npm pack --dry-run` que `dist/OpenClaw.app` no aparezca en la lista.
* **Bucle web de autenticación de npm para dist-tags**: usa el modo de autenticación heredado para obtener una solicitud de OTP:
  * `NPM_CONFIG_AUTH_TYPE=legacy npm dist-tag add openclaw@X.Y.Z latest`
* **La verificación de `npx` falla con `ECOMPROMISED: Lock compromised`**: vuelve a intentarlo con una caché nueva:
  * `NPM_CONFIG_CACHE=/tmp/npm-cache-$(date +%s) npx -y openclaw@X.Y.Z --version`
* **La etiqueta necesita volver a apuntarse después de una corrección tardía**: fuerza la actualización y el push de la etiqueta, y luego asegúrate de que los artefactos de la versión en GitHub sigan coincidiendo:
  * `git tag -f vX.Y.Z && git push -f origin vX.Y.Z`

7. **GitHub release + appcast**

* [ ] Etiqueta y haz push: `git tag vX.Y.Z && git push origin vX.Y.Z` (o `git push --tags`).
* [ ] Crea/actualiza el release de GitHub para `vX.Y.Z` con **título `openclaw X.Y.Z`** (no solo la etiqueta); el cuerpo debe incluir la sección de registro de cambios **completa** para esa versión (Novedades destacadas + Cambios + Correcciones), en línea (sin enlaces sueltos) y **no debe repetir el título dentro del cuerpo**.
* [ ] Adjunta artefactos: tarball de `npm pack` (opcional), `OpenClaw-X.Y.Z.zip` y `OpenClaw-X.Y.Z.dSYM.zip` (si se generó).
* [ ] Haz commit del `appcast.xml` actualizado y haz push (Sparkle consume el feed desde `main`).
* [ ] Desde un directorio temporal limpio (sin `package.json`), ejecuta `npx -y openclaw@X.Y.Z send --help` para confirmar que la instalación y los puntos de entrada de la CLI funcionan.
* [ ] Anuncia y comparte las notas de la versión.

<div id="plugin-publish-scope-npm">
  ## Ámbito de publicación de complementos (npm)
</div>

Solo publicamos **complementos npm existentes** bajo el ámbito `@openclaw/*`. Los
complementos integrados que no están en npm permanecen **solo en el árbol de archivos del disco**
(se siguen distribuyendo en `extensions/**`).

Proceso para obtener la lista:

1. Ejecuta `npm search @openclaw --json` y captura los nombres de los paquetes.
2. Compáralos con los nombres de `extensions/*/package.json`.
3. Publica solo la **intersección** (los que ya están en npm).

Lista actual de complementos npm (actualízala según sea necesario):

* @openclaw/bluebubbles
* @openclaw/diagnostics-otel
* @openclaw/discord
* @openclaw/lobster
* @openclaw/matrix
* @openclaw/msteams
* @openclaw/nextcloud-talk
* @openclaw/nostr
* @openclaw/voice-call
* @openclaw/zalo
* @openclaw/zalouser

Las notas de la versión también deben mencionar **nuevos complementos integrados opcionales** que **no están activados por defecto** (por ejemplo: `tlon`).