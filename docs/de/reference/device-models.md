---
title: Gerätemodelle
summary: "Wie OpenClaw Apple-Gerätemodell-Identifikatoren auf lesbare Namen in der macOS-App abbildet."
read_when:
  - Aktualisieren von Zuordnungen für Gerätemodell-Identifikatoren oder NOTICE-/Lizenzdateien
  - Ändern, wie die Instances-UI Gerätenamen anzeigt
---

<div id="device-model-database-friendly-names">
  # Datenbank für Gerätemodelle (sprechende Namen)
</div>

Die macOS-Companion-App zeigt in der **Instances**-UI sprechende Namen für Apple-Gerätemodelle an, indem Apple-Modellbezeichner (z. B. `iPad16,6`, `Mac16,6`) auf menschenlesbare Namen abgebildet werden.

Diese Zuordnung ist als JSON im folgenden Verzeichnis hinterlegt:

- `apps/macos/Sources/OpenClaw/Resources/DeviceModels/`

<div id="data-source">
  ## Datenquelle
</div>

Aktuell beziehen wir das Mapping aus dem MIT-lizenzierten Repository:

- `kyle-seongwoo-jun/apple-device-identifiers`

Um Builds deterministisch zu halten, sind die JSON-Dateien auf bestimmte Upstream-Commits fixiert (dokumentiert in `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md`).

<div id="updating-the-database">
  ## Aktualisieren der Datenbank
</div>

1. Wähle die Upstream-Commits aus, auf die du dich festlegen möchtest (je einen für iOS und macOS).
2. Aktualisiere die Commit-Hashes in `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md`.
3. Lade die JSON-Dateien erneut herunter, jeweils auf diese Commits fixiert:

```bash
IOS_COMMIT="<Commit-SHA für ios-device-identifiers.json>"
MAC_COMMIT="<Commit-SHA für mac-device-identifiers.json>"

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${IOS_COMMIT}/ios-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/ios-device-identifiers.json

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${MAC_COMMIT}/mac-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/mac-device-identifiers.json
```

4. Stelle sicher, dass `apps/macos/Sources/OpenClaw/Resources/DeviceModels/LICENSE.apple-device-identifiers.txt` weiterhin mit Upstream übereinstimmt (ersetze die Datei, falls sich die Upstream-Lizenz ändert).
5. Überprüfe, dass sich die macOS-App sauber bauen lässt (keine Warnungen):

```bash
swift build --package-path apps/macos
```
