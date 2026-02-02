---
title: Modelos de dispositivos
summary: "Cómo OpenClaw asigna identificadores de modelo de dispositivo de Apple a nombres descriptivos en la app de macOS."
read_when:
  - Actualizar los mapeos de identificadores de modelo de dispositivo o los archivos NOTICE/license
  - Cambiar cómo la UI de Instances muestra los nombres de los dispositivos
---

<div id="device-model-database-friendly-names">
  # Base de datos de modelos de dispositivos (nombres descriptivos)
</div>

La app complementaria de macOS muestra nombres descriptivos de modelos de dispositivos Apple en la UI de **Instances** al asociar identificadores de modelo de Apple (por ejemplo, `iPad16,6`, `Mac16,6`) con nombres legibles para personas.

El mapeo se distribuye como JSON en:

- `apps/macos/Sources/OpenClaw/Resources/DeviceModels/`

<div id="data-source">
  ## Fuente de datos
</div>

Actualmente incorporamos este mapeo a partir del repositorio con licencia MIT:

- `kyle-seongwoo-jun/apple-device-identifiers`

Para que las compilaciones sean deterministas, los archivos JSON se fijan a *commits* específicos del repositorio original (registrados en `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md`).

<div id="updating-the-database">
  ## Actualizar la base de datos
</div>

1. Elige los commits upstream a los que quieres fijar la versión (uno para iOS, otro para macOS).
2. Actualiza los hashes de esos commits en `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md`.
3. Vuelve a descargar los archivos JSON, fijados a esos commits:

```bash
IOS_COMMIT="<SHA de commit para ios-device-identifiers.json>"
MAC_COMMIT="<SHA de commit para mac-device-identifiers.json>"

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${IOS_COMMIT}/ios-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/ios-device-identifiers.json

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${MAC_COMMIT}/mac-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/mac-device-identifiers.json
```

4. Asegúrate de que `apps/macos/Sources/OpenClaw/Resources/DeviceModels/LICENSE.apple-device-identifiers.txt` siga coincidiendo con la versión upstream (reemplázalo si la licencia upstream cambia).
5. Verifica que la aplicación de macOS se compile sin problemas (sin advertencias de compilación):

```bash
swift build --package-path apps/macos
```
