---
title: Rilascio
summary: "Checklist per il rilascio di OpenClaw su macOS (feed Sparkle, packaging, firma)"
read_when:
  - Durante la creazione o la convalida di una release di OpenClaw per macOS
  - Durante l'aggiornamento dell'appcast di Sparkle o degli asset del feed
---

<div id="openclaw-macos-release-sparkle">
  # OpenClaw macOS release (Sparkle)
</div>

Questa app ora supporta aggiornamenti automatici tramite Sparkle. Le build di rilascio devono essere firmate con Developer ID, compresse in un file zip e pubblicate con una voce di appcast firmata.

<div id="prereqs">
  ## Prerequisiti
</div>

- Certificato Developer ID Application installato (esempio: `Developer ID Application: <Developer Name> (<TEAMID>)`).
- Percorso della chiave privata Sparkle impostato nell'ambiente come `SPARKLE_PRIVATE_KEY_FILE` (percorso della tua chiave privata Sparkle ed25519; la chiave pubblica è incorporata in Info.plist). Se manca, controlla `~/.profile`.
- Credenziali per la notarizzazione (profilo del portachiavi o chiave API) per `xcrun notarytool` se vuoi distribuire DMG/zip sicuri per Gatekeeper.
  - Usiamo un profilo Keychain chiamato `openclaw-notary`, creato dalle variabili d'ambiente della chiave API di App Store Connect nel tuo profilo shell:
    - `APP_STORE_CONNECT_API_KEY_P8`, `APP_STORE_CONNECT_KEY_ID`, `APP_STORE_CONNECT_ISSUER_ID`
    - `echo "$APP_STORE_CONNECT_API_KEY_P8" | sed 's/\\n/\n/g' > /tmp/openclaw-notary.p8`
    - `xcrun notarytool store-credentials "openclaw-notary" --key /tmp/openclaw-notary.p8 --key-id "$APP_STORE_CONNECT_KEY_ID" --issuer "$APP_STORE_CONNECT_ISSUER_ID"`
- Dipendenze `pnpm` installate (`pnpm install --config.node-linker=hoisted`).
- Gli strumenti Sparkle vengono recuperati automaticamente tramite SwiftPM in `apps/macos/.build/artifacts/sparkle/Sparkle/bin/` (`sign_update`, `generate_appcast`, ecc.).

<div id="build-package">
  ## Build &amp; package
</div>

Note:

* `APP_BUILD` corrisponde a `CFBundleVersion`/`sparkle:version`; mantienilo numerico e strettamente crescente (niente `-beta`), altrimenti Sparkle li considera uguali.
* Per impostazione predefinita usa l&#39;architettura corrente (`$(uname -m)`). Per build di release/universal, imposta `BUILD_ARCHS="arm64 x86_64"` (oppure `BUILD_ARCHS=all`).
* Usa `scripts/package-mac-dist.sh` per gli artefatti di release (zip + DMG + notarizzazione). Usa `scripts/package-mac-app.sh` per il packaging locale/dev.

```bash
# Dalla radice del repository; imposta gli ID di rilascio per abilitare il feed Sparkle.
# APP_BUILD deve essere numerico e monotono per il confronto Sparkle.
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
  ## Voce dell’Appcast
</div>

Usa il generatore di note di rilascio in modo che Sparkle visualizzi note HTML formattate:

```bash
SPARKLE_PRIVATE_KEY_FILE=/path/to/ed25519-private-key scripts/make_appcast.sh dist/OpenClaw-2026.1.27-beta.1.zip https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml
```

Genera le note di rilascio in HTML da `CHANGELOG.md` (tramite [`scripts/changelog-to-html.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/changelog-to-html.sh)) e le incorpora nella voce dell&#39;appcast.
Effettua il commit del file `appcast.xml` aggiornato insieme alle risorse del rilascio (archivio zip + dSYM) al momento della pubblicazione.


<div id="publish-verify">
  ## Pubblica e verifica
</div>

- Carica `OpenClaw-2026.1.27-beta.1.zip` (e `OpenClaw-2026.1.27-beta.1.dSYM.zip`) nella release GitHub per il tag `v2026.1.27-beta.1`.
- Verifica che l’URL raw dell’appcast corrisponda al feed incorporato: `https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml`.
- Verifiche di base:
  - `curl -I https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml` restituisce 200.
  - `curl -I <enclosure url>` restituisce 200 dopo il caricamento degli asset.
  - Su una build pubblica precedente, esegui “Check for Updates…” dalla scheda About e verifica che Sparkle installi la nuova build senza problemi.

Definition of done: l’app firmata e l’appcast sono pubblicati, il flusso di aggiornamento funziona a partire da una versione installata precedente e gli asset della release sono allegati alla release GitHub.