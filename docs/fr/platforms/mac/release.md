---
title: Version
summary: "Liste de contrôle pour la publication d'une version OpenClaw macOS (flux Sparkle, empaquetage, signature)"
read_when:
  - Préparation ou validation d'une version OpenClaw macOS
  - Mise à jour de l'appcast Sparkle ou des ressources du flux
---

<div id="openclaw-macos-release-sparkle">
  # OpenClaw macOS release (Sparkle)
</div>

Cette application intègre désormais les mises à jour automatiques Sparkle. Les builds de release doivent être signés avec un Developer ID, archivés en zip, puis publiés avec une entrée d'appcast signée.

<div id="prereqs">
  ## Prérequis
</div>

- Certificat Developer ID Application installé (exemple : `Developer ID Application: <Developer Name> (<TEAMID>)`).
- Chemin de la clé privée Sparkle défini dans l'environnement en tant que `SPARKLE_PRIVATE_KEY_FILE` (chemin vers votre clé privée Sparkle ed25519 ; la clé publique est intégrée dans Info.plist). S'il n'est pas défini, vérifiez `~/.profile`.
- Identifiants de notarisation (profil de trousseau d'accès ou clé API) pour `xcrun notarytool` si vous voulez une distribution DMG/zip acceptée par Gatekeeper.
  - Nous utilisons un profil Keychain nommé `openclaw-notary`, créé à partir des variables d'environnement de la clé App Store Connect dans votre profil de shell :
    - `APP_STORE_CONNECT_API_KEY_P8`, `APP_STORE_CONNECT_KEY_ID`, `APP_STORE_CONNECT_ISSUER_ID`
    - `echo "$APP_STORE_CONNECT_API_KEY_P8" | sed 's/\\n/\n/g' > /tmp/openclaw-notary.p8`
    - `xcrun notarytool store-credentials "openclaw-notary" --key /tmp/openclaw-notary.p8 --key-id "$APP_STORE_CONNECT_KEY_ID" --issuer "$APP_STORE_CONNECT_ISSUER_ID"`
- Dépendances `pnpm` installées (`pnpm install --config.node-linker=hoisted`).
- Les outils Sparkle sont récupérés automatiquement via SwiftPM dans `apps/macos/.build/artifacts/sparkle/Sparkle/bin/` (`sign_update`, `generate_appcast`, etc.).

<div id="build-package">
  ## Build &amp; package
</div>

Notes :

* `APP_BUILD` correspond à `CFBundleVersion`/`sparkle:version` ; gardez une valeur numérique et monotone (pas de `-beta`), sinon Sparkle la considère comme identique.
* Par défaut, utilise l’architecture actuelle (`$(uname -m)`). Pour des builds de release universelles, définissez `BUILD_ARCHS="arm64 x86_64"` (ou `BUILD_ARCHS=all`).
* Utilisez `scripts/package-mac-dist.sh` pour les artefacts de release (zip + DMG + notarisation). Utilisez `scripts/package-mac-app.sh` pour l’empaquetage local/développement.

```bash
# Depuis la racine du dépôt ; définir les identifiants de version pour activer le flux Sparkle.
# APP_BUILD doit être numérique et monotone pour la comparaison Sparkle.
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
  ## Entrée d’appcast
</div>

Utilisez le générateur de notes de version pour que Sparkle affiche des notes HTML mises en forme :

```bash
SPARKLE_PRIVATE_KEY_FILE=/path/to/ed25519-private-key scripts/make_appcast.sh dist/OpenClaw-2026.1.27-beta.1.zip https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml
```

Génère des notes de version HTML à partir de `CHANGELOG.md` (via [`scripts/changelog-to-html.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/changelog-to-html.sh)) et les intègre dans l’entrée d’appcast.
Validez le fichier `appcast.xml` mis à jour en même temps que les assets de la version (archive zip + dSYM) lors de la publication.


<div id="publish-verify">
  ## Publication et vérification
</div>

- Téléversez `OpenClaw-2026.1.27-beta.1.zip` (et `OpenClaw-2026.1.27-beta.1.dSYM.zip`) dans la publication GitHub pour le tag `v2026.1.27-beta.1`.
- Vérifiez que l’URL brute de l’appcast correspond bien au flux intégré : `https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml`.
- Vérifications de cohérence :
  - `curl -I https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml` renvoie 200.
  - `curl -I <enclosure url>` renvoie 200 après le téléversement des fichiers de la release.
  - Sur une version publique précédente, exécutez « Check for Updates… » depuis l’onglet « À propos » et vérifiez que Sparkle installe correctement la nouvelle version.

Définition de fin de tâche : l’app signée et l’appcast sont publiés, le flux de mise à jour fonctionne à partir d’une version installée plus ancienne, et les fichiers de la release sont attachés à la publication GitHub.