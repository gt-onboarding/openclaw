---
title: Release
summary: "OpenClaw macOS Release-Checkliste (Sparkle-Feed, Paketierung, Signierung)"
read_when:
  - Erstellen oder Validieren eines OpenClaw macOS-Releases
  - Aktualisieren des Sparkle-Appcast- oder der Feed-Assets
---

<div id="openclaw-macos-release-sparkle">
  # OpenClaw macOS-Release (Sparkle)
</div>

Diese App unterstützt jetzt automatische Updates über Sparkle. Release-Builds müssen mit einer Developer ID signiert, gezippt und zusammen mit einem signierten Appcast-Eintrag veröffentlicht werden.

<div id="prereqs">
  ## Voraussetzungen
</div>

- Developer ID Application-Zertifikat installiert (Beispiel: `Developer ID Application: <Developer Name> (<TEAMID>)`).
- Pfad zum Sparkle-Privatschlüssel in der Umgebung als `SPARKLE_PRIVATE_KEY_FILE` gesetzt (Pfad zu deinem Sparkle-ed25519-Privatschlüssel; öffentlicher Schlüssel ist in der Info.plist eingebettet). Falls er fehlt, prüfe `~/.profile`.
- Notarisierungs-Zugangsdaten (Schlüsselbund-Profil oder API-Key) für `xcrun notarytool`, wenn du eine Gatekeeper-sichere DMG-/ZIP-Distribution möchtest.
  - Wir verwenden ein Schlüsselbund-Profil namens `openclaw-notary`, erstellt aus den App Store Connect API-Key-Umgebungsvariablen in deinem Shell-Profil:
    - `APP_STORE_CONNECT_API_KEY_P8`, `APP_STORE_CONNECT_KEY_ID`, `APP_STORE_CONNECT_ISSUER_ID`
    - `echo "$APP_STORE_CONNECT_API_KEY_P8" | sed 's/\\n/\n/g' > /tmp/openclaw-notary.p8`
    - `xcrun notarytool store-credentials "openclaw-notary" --key /tmp/openclaw-notary.p8 --key-id "$APP_STORE_CONNECT_KEY_ID" --issuer "$APP_STORE_CONNECT_ISSUER_ID"`
- `pnpm`-Abhängigkeiten installiert (`pnpm install --config.node-linker=hoisted`).
- Sparkle-Tools werden automatisch via SwiftPM unter `apps/macos/.build/artifacts/sparkle/Sparkle/bin/` bezogen (`sign_update`, `generate_appcast`, etc.).

<div id="build-package">
  ## Build &amp; Package
</div>

Hinweise:

* `APP_BUILD` wird `CFBundleVersion`/`sparkle:version` zugeordnet; halte ihn numerisch und monoton (kein `-beta`), sonst betrachtet Sparkle unterschiedliche Werte als gleich.
* Standardmäßig wird die aktuelle Architektur verwendet (`$(uname -m)`). Für Release-/Universal-Builds setze `BUILD_ARCHS="arm64 x86_64"` (oder `BUILD_ARCHS=all`).
* Verwende `scripts/package-mac-dist.sh` für Release-Artefakte (ZIP + DMG + Notarisierung). Verwende `scripts/package-mac-app.sh` für lokale/Dev-Pakete.

```bash
# Vom Repository-Root aus; Release-IDs setzen, damit der Sparkle-Feed aktiviert wird.
# APP_BUILD muss numerisch + monoton steigend für den Sparkle-Vergleich sein.
BUNDLE_ID=bot.molt.mac \
APP_VERSION=2026.1.27-beta.1 \
APP_BUILD="$(git rev-list --count HEAD)" \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-app.sh

# Zip für die Distribution (enthält Resource Forks für Sparkle-Delta-Unterstützung)
ditto -c -k --sequesterRsrc --keepParent dist/OpenClaw.app dist/OpenClaw-2026.1.27-beta.1.zip

# Optional: auch ein gestyltes DMG für Benutzer erstellen (Drag-and-Drop nach /Applications)
scripts/create-dmg.sh dist/OpenClaw.app dist/OpenClaw-2026.1.27-beta.1.dmg

# Empfohlen: Build + Notarisierung/Stapling von Zip + DMG
# Zuerst einmalig ein Keychain-Profil erstellen:
#   xcrun notarytool store-credentials "openclaw-notary" \
#     --apple-id "<apple-id>" --team-id "<team-id>" --password "<app-specific-password>"
NOTARIZE=1 NOTARYTOOL_PROFILE=openclaw-notary \
BUNDLE_ID=bot.molt.mac \
APP_VERSION=2026.1.27-beta.1 \
APP_BUILD="$(git rev-list --count HEAD)" \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-dist.sh

# Optional: dSYM zusammen mit dem Release ausliefern
ditto -c -k --keepParent apps/macos/.build/release/OpenClaw.app.dSYM dist/OpenClaw-2026.1.27-beta.1.dSYM.zip
```


<div id="appcast-entry">
  ## Appcast-Eintrag
</div>

Verwende den Release-Notes-Generator, damit Sparkle formatierte HTML-Notizen rendert:

```bash
SPARKLE_PRIVATE_KEY_FILE=/path/to/ed25519-private-key scripts/make_appcast.sh dist/OpenClaw-2026.1.27-beta.1.zip https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml
```

Erzeugt HTML-Release-Notes aus `CHANGELOG.md` (mit [`scripts/changelog-to-html.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/changelog-to-html.sh)) und fügt sie in den Appcast-Eintrag ein.
Committe die aktualisierte `appcast.xml` zusammen mit den Release-Assets (ZIP + dSYM) beim Veröffentlichen.


<div id="publish-verify">
  ## Veröffentlichen & verifizieren
</div>

- Lade `OpenClaw-2026.1.27-beta.1.zip` (und `OpenClaw-2026.1.27-beta.1.dSYM.zip`) in das GitHub-Release für den Tag `v2026.1.27-beta.1` hoch.
- Stelle sicher, dass die rohe Appcast-URL mit dem finalen Feed übereinstimmt: `https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml`.
- Plausibilitätsprüfungen:
  - `curl -I https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml` liefert 200.
  - `curl -I <enclosure url>` liefert 200, nachdem die Assets hochgeladen wurden.
  - Führe in einem zuvor öffentlichen Build „Check for Updates…” im About-Tab aus und prüfe, dass Sparkle den neuen Build sauber installiert.

Definition of done: signierte App + Appcast sind veröffentlicht, der Update-Flow funktioniert von einer zuvor installierten älteren Version aus, und die Release-Assets sind dem GitHub-Release angehängt.