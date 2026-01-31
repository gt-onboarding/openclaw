---
title: Firma
summary: "Pasos de firma para compilaciones de depuración de macOS generadas por los scripts de empaquetado"
read_when:
  - Al compilar o firmar compilaciones de depuración de macOS
---

<div id="mac-signing-debug-builds">
  # firma en mac (compilaciones de depuración)
</div>

Esta app suele compilarse desde [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh), que ahora:

* establece un identificador de bundle de depuración estable: `ai.openclaw.mac.debug`
* escribe el Info.plist con ese id de bundle (puedes sobrescribirlo con `BUNDLE_ID=...`)
* llama a [`scripts/codesign-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/codesign-mac-app.sh) para firmar el binario principal y el bundle de la app, de modo que macOS trate cada recompilación como el mismo bundle firmado y conserve los permisos TCC (notificaciones, accesibilidad, grabación de pantalla, micrófono, voz). Para permisos estables, usa una identidad de firma real; la firma ad-hoc es opcional y frágil (consulta [permisos de macOS](/es/platforms/mac/permissions)).
* usa `CODESIGN_TIMESTAMP=auto` de forma predeterminada; habilita marcas de tiempo de confianza para firmas de Developer ID. Establece `CODESIGN_TIMESTAMP=off` para omitir las marcas de tiempo (compilaciones de depuración sin conexión).
* inyecta metadatos de compilación en Info.plist: `OpenClawBuildTimestamp` (UTC) y `OpenClawGitCommit` (hash corto), para que el panel Acerca de pueda mostrar la compilación, la información de git y el canal de depuración/publicación.
* **El empaquetado requiere Node 22+**: el script ejecuta las compilaciones de TS y de la Control UI.
* lee `SIGN_IDENTITY` del entorno. Añade `export SIGN_IDENTITY="Apple Development: Your Name (TEAMID)"` (o tu certificado Developer ID Application) a tu shell rc para firmar siempre con tu certificado. La firma ad-hoc requiere una aceptación explícita mediante `ALLOW_ADHOC_SIGNING=1` o `SIGN_IDENTITY="-"` (no recomendado para pruebas de permisos).
* ejecuta una auditoría del Team ID después de firmar y falla si cualquier Mach-O dentro del bundle de la app está firmado por un Team ID diferente. Establece `SKIP_TEAM_ID_CHECK=1` para omitir esta comprobación.

<div id="usage">
  ## Uso
</div>

```bash
# from repo root
scripts/package-mac-app.sh               # auto-selects identity; errors if none found
SIGN_IDENTITY="Developer ID Application: Your Name" scripts/package-mac-app.sh   # real cert
ALLOW_ADHOC_SIGNING=1 scripts/package-mac-app.sh    # ad-hoc (permissions will not stick)
SIGN_IDENTITY="-" scripts/package-mac-app.sh        # explicit ad-hoc (same caveat)
DISABLE_LIBRARY_VALIDATION=1 scripts/package-mac-app.sh   # solución temporal solo para desarrollo por discrepancia de Sparkle Team ID
```

<div id="ad-hoc-signing-note">
  ### Nota sobre la firma ad-hoc
</div>

Al firmar con `SIGN_IDENTITY="-"` (ad-hoc), el script deshabilita automáticamente el **Hardened Runtime** (`--options runtime`). Esto es necesario para evitar bloqueos cuando la aplicación intenta cargar frameworks integrados (como Sparkle) que no comparten el mismo Team ID. Las firmas ad-hoc también invalidan la persistencia de permisos de TCC; consulta [permisos de macOS](/es/platforms/mac/permissions) para conocer los pasos de recuperación.

<div id="build-metadata-for-about">
  ## Metadatos de compilación para Acerca de
</div>

`package-mac-app.sh` marca el paquete con:

* `OpenClawBuildTimestamp`: marca de tiempo ISO8601 en UTC en el momento del empaquetado
* `OpenClawGitCommit`: hash corto de git (o `unknown` si no está disponible)

La pestaña Acerca de usa estas claves para mostrar la versión, la fecha de compilación, el commit de git y si es una compilación de depuración (mediante `#if DEBUG`). Vuelve a ejecutar el script de empaquetado para actualizar estos valores después de cambios en el código.

<div id="why">
  ## Por qué
</div>

Los permisos de TCC están vinculados tanto al identificador del bundle *como* a la firma de código. Las builds de depuración sin firmar, con UUID cambiantes, hacían que macOS olvidara las autorizaciones después de cada recompilación. Firmar los binarios (ad-hoc por defecto) y mantener un identificador/ruta de bundle fijo (`dist/OpenClaw.app`) preserva las autorizaciones entre compilaciones, coincidiendo con el enfoque de VibeTunnel.