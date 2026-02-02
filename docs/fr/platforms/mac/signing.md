---
title: Signature
summary: "Étapes de signature pour les builds de débogage macOS générés par les scripts de packaging"
read_when:
  - Compilation ou signature de builds de débogage macOS
---

<div id="mac-signing-debug-builds">
  # signature mac (builds de débogage)
</div>

Cette application est généralement construite via [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh), qui désormais :

* définit un identifiant de bundle de débogage stable : `ai.openclaw.mac.debug`
* écrit le fichier Info.plist avec cet identifiant de bundle (que vous pouvez surcharger via `BUNDLE_ID=...`)
* appelle [`scripts/codesign-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/codesign-mac-app.sh) pour signer le binaire principal et le bundle de l’app, afin que macOS traite chaque rebuild comme le même bundle signé et conserve les autorisations TCC (notifications, accessibilité, enregistrement d’écran, micro, synthèse vocale). Pour des autorisations stables, utilisez une véritable identité de signature ; la signature ad hoc est en opt-in et fragile (voir [autorisations macOS](/fr/platforms/mac/permissions)).
* utilise `CODESIGN_TIMESTAMP=auto` par défaut ; cela active les horodatages de confiance pour les signatures Developer ID. Définissez `CODESIGN_TIMESTAMP=off` pour ignorer l’horodatage (builds de débogage hors ligne).
* injecte des métadonnées de build dans Info.plist : `OpenClawBuildTimestamp` (UTC) et `OpenClawGitCommit` (hash court), afin que le panneau À propos puisse afficher les informations de build, le commit git et le canal debug/release.
* **Le packaging nécessite Node 22+** : le script exécute les builds TypeScript et le build du Control UI.
* lit `SIGN_IDENTITY` depuis l’environnement. Ajoutez `export SIGN_IDENTITY="Apple Development: Your Name (TEAMID)"` (ou votre certificat Developer ID Application) à votre rc de shell pour toujours signer avec votre certificat. La signature ad hoc nécessite un opt-in explicite via `ALLOW_ADHOC_SIGNING=1` ou `SIGN_IDENTITY="-"` (non recommandé pour les tests d’autorisations).
* exécute un audit de Team ID après la signature et échoue si un binaire Mach-O à l’intérieur du bundle de l’app est signé par un Team ID différent. Définissez `SKIP_TEAM_ID_CHECK=1` pour contourner ce contrôle.

<div id="usage">
  ## Utilisation
</div>

```bash
# depuis la racine du dépôt
scripts/package-mac-app.sh               # sélectionne automatiquement l'identité ; erreur si aucune trouvée
SIGN_IDENTITY="Developer ID Application: Your Name" scripts/package-mac-app.sh   # certificat réel
ALLOW_ADHOC_SIGNING=1 scripts/package-mac-app.sh    # ad-hoc (les permissions ne seront pas conservées)
SIGN_IDENTITY="-" scripts/package-mac-app.sh        # ad-hoc explicite (même limitation)
DISABLE_LIBRARY_VALIDATION=1 scripts/package-mac-app.sh   # contournement dev uniquement pour non-correspondance Sparkle Team ID
```

<div id="ad-hoc-signing-note">
  ### Remarque sur la signature ad hoc
</div>

Lorsque vous signez avec `SIGN_IDENTITY="-"` (ad hoc), le script désactive automatiquement le **Hardened Runtime** (`--options runtime`). Cela est nécessaire pour éviter les plantages lorsque l’app tente de charger des frameworks intégrés (comme Sparkle) qui ne partagent pas le même Team ID. Les signatures ad hoc rompent également la persistance des autorisations TCC ; consultez la section [Autorisations macOS](/fr/platforms/mac/permissions) pour les étapes de récupération.

<div id="build-metadata-for-about">
  ## Métadonnées de build pour À propos
</div>

`package-mac-app.sh` ajoute au bundle :

* `OpenClawBuildTimestamp` : horodatage ISO 8601 en UTC au moment de l’empaquetage
* `OpenClawGitCommit` : hash Git abrégé (ou `unknown` si non disponible)

L’onglet « À propos » lit ces clés pour afficher la version, la date de build, le commit Git et indiquer s’il s’agit d’une build de débogage (via `#if DEBUG`). Exécutez le packager pour actualiser ces valeurs après des modifications du code.

<div id="why">
  ## Pourquoi
</div>

Les autorisations TCC sont liées à l’identifiant de bundle *et* à la signature du code. Les builds de débogage non signées, avec des UUID changeants, amenaient macOS à « oublier » les autorisations après chaque recompilation. Signer les binaires (ad‑hoc par défaut) et conserver un identifiant de bundle/chemin fixe (`dist/OpenClaw.app`) préserve les autorisations entre les builds, en reproduisant l’approche de VibeTunnel.