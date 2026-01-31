---
title: Lanzamiento
summary: "Lista de comprobación de versión de OpenClaw para macOS (feed de Sparkle, empaquetado, firma)"
read_when:
  - Al crear o validar una versión de OpenClaw para macOS
  - Al actualizar el appcast de Sparkle o los recursos del feed
---

<div id="openclaw-macos-release-sparkle">
  # Lanzamiento de OpenClaw para macOS (Sparkle)
</div>

Esta aplicación ahora incorpora actualizaciones automáticas con Sparkle. Las compilaciones de lanzamiento deben estar firmadas con un Developer ID, empaquetadas en un archivo zip y publicadas con una entrada de appcast firmada.

<div id="prereqs">
  ## Requisitos previos
</div>

- Certificado Developer ID Application instalado (por ejemplo: `Developer ID Application: <Developer Name> (<TEAMID>)`).
- Ruta de la clave privada de Sparkle definida en el entorno como `SPARKLE_PRIVATE_KEY_FILE` (ruta a tu clave privada ed25519 de Sparkle; la clave pública va incrustada en Info.plist). Si falta, revisa `~/.profile`.
- Credenciales de notarización (perfil del llavero o clave de API) para `xcrun notarytool` si quieres distribuir DMG/zip seguros para Gatekeeper.
  - Usamos un perfil de llavero llamado `openclaw-notary`, creado a partir de las variables de entorno de la clave de API de App Store Connect en tu perfil de shell:
    - `APP_STORE_CONNECT_API_KEY_P8`, `APP_STORE_CONNECT_KEY_ID`, `APP_STORE_CONNECT_ISSUER_ID`
    - `echo "$APP_STORE_CONNECT_API_KEY_P8" | sed 's/\\n/\n/g' > /tmp/openclaw-notary.p8`
    - `xcrun notarytool store-credentials "openclaw-notary" --key /tmp/openclaw-notary.p8 --key-id "$APP_STORE_CONNECT_KEY_ID" --issuer "$APP_STORE_CONNECT_ISSUER_ID"`
- Dependencias de `pnpm` instaladas (`pnpm install --config.node-linker=hoisted`).
- Las herramientas de Sparkle se descargan automáticamente a través de SwiftPM en `apps/macos/.build/artifacts/sparkle/Sparkle/bin/` (`sign_update`, `generate_appcast`, etc.).

<div id="build-package">
  ## Compilación y empaquetado
</div>

Notas:

* `APP_BUILD` se asigna a `CFBundleVersion`/`sparkle:version`; mantenlo numérico y estrictamente creciente (sin `-beta`), o Sparkle lo tratará como igual.
* De forma predeterminada usa la arquitectura actual (`$(uname -m)`). Para compilaciones de lanzamiento/universales, establece `BUILD_ARCHS="arm64 x86_64"` (o `BUILD_ARCHS=all`).
* Usa `scripts/package-mac-dist.sh` para artefactos de lanzamiento (zip + DMG + notarización). Usa `scripts/package-mac-app.sh` para empaquetado local/dev.

```bash
# Desde la raíz del repositorio; establece los IDs de versión para habilitar el feed de Sparkle.
# APP_BUILD debe ser numérico y monótono para la comparación de Sparkle.
BUNDLE_ID=bot.molt.mac \
APP_VERSION=2026.1.27-beta.1 \
APP_BUILD="$(git rev-list --count HEAD)" \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-app.sh

# Zip for distribution (includes resource forks for Sparkle delta support)
ditto -c -k --sequesterRsrc --keepParent dist/OpenClaw.app dist/OpenClaw-2026.1.27-beta.1.zip

# Optional: also build a styled DMG for humans (drag to /Applications)
scripts/create-dmg.sh dist/OpenClaw.app dist/OpenClaw-2026.1.27-beta.1.dmg

# Recommended: build + notarize/staple zip + DMG
# First, create a keychain profile once:
#   xcrun notarytool store-credentials "openclaw-notary" \
#     --apple-id "<apple-id>" --team-id "<team-id>" --password "<app-specific-password>"
NOTARIZE=1 NOTARYTOOL_PROFILE=openclaw-notary \
BUNDLE_ID=bot.molt.mac \
APP_VERSION=2026.1.27-beta.1 \
APP_BUILD="$(git rev-list --count HEAD)" \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-dist.sh

# Optional: ship dSYM alongside the release
ditto -c -k --keepParent apps/macos/.build/release/OpenClaw.app.dSYM dist/OpenClaw-2026.1.27-beta.1.dSYM.zip
```


<div id="appcast-entry">
  ## Entrada de Appcast
</div>

Utiliza el generador de notas de versión para que Sparkle muestre notas HTML con formato:

```bash
SPARKLE_PRIVATE_KEY_FILE=/path/to/ed25519-private-key scripts/make_appcast.sh dist/OpenClaw-2026.1.27-beta.1.zip https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml
```

Genera notas de la versión en HTML a partir de `CHANGELOG.md` (mediante [`scripts/changelog-to-html.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/changelog-to-html.sh)) e insértalas en la entrada del appcast.
Confirma el `appcast.xml` actualizado junto con los artefactos de la versión (zip + dSYM) al publicar.


<div id="publish-verify">
  ## Publicar y verificar
</div>

- Sube `OpenClaw-2026.1.27-beta.1.zip` (y `OpenClaw-2026.1.27-beta.1.dSYM.zip`) al release de GitHub para la etiqueta `v2026.1.27-beta.1`.
- Asegúrate de que la URL raw del appcast coincida con el feed incorporado: `https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml`.
- Comprobaciones básicas:
  - `curl -I https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml` devuelve 200.
  - `curl -I <enclosure url>` devuelve 200 después de subir los assets.
  - En un build público anterior, ejecuta “Check for Updates…” desde la pestaña About y verifica que Sparkle instale el nuevo build correctamente.

Definición de completado: la app firmada y el appcast están publicados, el flujo de actualización funciona desde una versión instalada anterior y los assets del release están adjuntos al release de GitHub.