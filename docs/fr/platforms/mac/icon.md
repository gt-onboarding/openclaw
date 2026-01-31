---
title: Icône
summary: "États et animations de l’icône de la barre de menus pour OpenClaw sur macOS"
read_when:
  - Modifier le comportement de l’icône de la barre de menus
---

<div id="menu-bar-icon-states">
  # États de l’icône dans la barre de menus
</div>

Author: steipete · Mis à jour : 2025-12-06 · Portée : app macOS (`apps/macos`)

- **Inactif :** animation normale de l’icône (clignement, petit frétillement occasionnel).
- **En pause :** l’élément de statut utilise `appearsDisabled` ; aucun mouvement.
- **Déclencheur vocal (grandes oreilles) :** le détecteur d’activation vocale appelle `AppState.triggerVoiceEars(ttl: nil)` quand le mot d’activation est entendu, en maintenant `earBoostActive=true` pendant la capture de l’énoncé. Les oreilles grossissent (1,9×), obtiennent des trous circulaires pour améliorer la lisibilité, puis reviennent à la normale via `stopVoiceEars()` après 1 s de silence. Déclenché uniquement depuis le pipeline vocal intégré à l’app.
- **En activité (agent en cours d’exécution) :** `AppState.isWorking=true` déclenche un micro-mouvement de « queue/pattes qui s’affairent » : oscillation des pattes plus rapide et léger décalage pendant que le travail est en cours. Actuellement activé autour des exécutions de l’agent WebChat ; ajoutez le même basculement autour d’autres tâches longues lorsque vous les câblerez.

Points de raccordement

- Réveil vocal : appelez `AppState.triggerVoiceEars(ttl: nil)` au déclenchement et `stopVoiceEars()` après 1 s de silence pour correspondre à la fenêtre de capture.
- Activité de l’agent : définissez `AppStateStore.shared.setWorking(true/false)` autour des plages de travail (déjà fait dans l’appel à l’agent WebChat). Gardez ces plages courtes et réinitialisez dans des blocs `defer` pour éviter des animations bloquées.

Formes et tailles

- Icône de base dessinée dans `CritterIconRenderer.makeIcon(blink:legWiggle:earWiggle:earScale:earHoles:)`.
- L’échelle des oreilles est par défaut `1.0` ; le boost vocal définit `earScale=1.9` et active `earHoles=true` sans changer le cadre global (image modèle 18×18 pt rendue dans un backing store Retina de 36×36 px).
- Le mouvement « scurry » utilise une oscillation des pattes jusqu’à ~1.0 avec un léger mouvement horizontal ; il s’ajoute à tout frétillement inactif existant.

Notes comportementales

- Aucun basculement externe via la CLI/broker pour les oreilles/état en activité ; gardez cela interne aux signaux propres de l’app pour éviter des oscillations accidentelles.
- Gardez les TTL courts (&lt;10 s) afin que l’icône revienne rapidement à l’état de base si un job se bloque.