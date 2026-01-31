---
title: Modèles d’appareils
summary: "Comment OpenClaw convertit les identifiants de modèles d’appareils Apple en noms lisibles pour l’utilisateur dans l’app macOS."
read_when:
  - Mise à jour des mappages d’identifiants de modèles d’appareils ou des fichiers NOTICE/licence
  - Modification de l’affichage des noms d’appareils dans l’UI Instances
---

<div id="device-model-database-friendly-names">
  # Base de données des modèles d’appareils (noms lisibles)
</div>

L’app compagnon macOS affiche des noms lisibles de modèles d’appareils Apple dans l’UI **Instances** en associant les identifiants de modèles Apple (par exemple `iPad16,6`, `Mac16,6`) à des noms lisibles par l’humain.

Cette table de correspondance est intégrée au projet sous forme de JSON dans :

- `apps/macos/Sources/OpenClaw/Resources/DeviceModels/`

<div id="data-source">
  ## Source de données
</div>

Nous utilisons actuellement le mapping fourni par le dépôt sous licence MIT :

- `kyle-seongwoo-jun/apple-device-identifiers`

Pour que les builds restent déterministes, les fichiers JSON sont figés sur des commits amont précis (répertoriés dans `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md`).

<div id="updating-the-database">
  ## Mise à jour de la base de données
</div>

1. Choisissez les commits amont sur lesquels vous voulez vous baser (un pour iOS, un pour macOS).
2. Mettez à jour les hachages de commits dans `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md`.
3. Téléchargez de nouveau les fichiers JSON, figés sur ces commits :

```bash
IOS_COMMIT="<commit sha for ios-device-identifiers.json>"
MAC_COMMIT="<SHA du commit pour mac-device-identifiers.json>"

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${IOS_COMMIT}/ios-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/ios-device-identifiers.json

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${MAC_COMMIT}/mac-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/mac-device-identifiers.json
```

4. Assurez-vous que `apps/macos/Sources/OpenClaw/Resources/DeviceModels/LICENSE.apple-device-identifiers.txt` reste identique à la version en amont (remplacez-le si la licence en amont change).
5. Vérifiez que l’app macOS se compile sans avertissements :

```bash
swift build --package-path apps/macos
```
