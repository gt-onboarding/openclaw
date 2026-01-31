---
title: Commande de localisation
summary: "Commande de localisation pour les nœuds (location.get), modes d’autorisation et comportement en arrière-plan"
read_when:
  - Ajout de la prise en charge de la localisation des nœuds ou de l’UI de gestion des autorisations
  - Conception des flux de localisation en arrière-plan et de notifications push
---

<div id="location-command-nodes">
  # Commande location (nœuds)
</div>

<div id="tldr">
  ## TL;DR
</div>

- `location.get` est une commande de nœud (via `node.invoke`).
- Désactivé par défaut.
- Les paramètres utilisent un sélecteur : Désactivé / Pendant l'utilisation / Toujours.
- Interrupteur distinct : Localisation précise.

<div id="why-a-selector-not-just-a-switch">
  ## Pourquoi un sélecteur (et pas seulement un interrupteur)
</div>

Les autorisations de l’OS sont à plusieurs niveaux. Nous pouvons exposer un sélecteur dans l’application, mais l’OS décide toujours de l’autorisation effective.

- iOS/macOS : l’utilisateur peut choisir **Pendant l’utilisation** ou **Toujours** dans les invites système / Réglages. L’application peut demander une élévation de niveau, mais l’OS peut exiger un passage par Réglages.
- Android : la localisation en arrière-plan est une autorisation distincte ; sur Android 10+ elle nécessite souvent un parcours via Réglages.
- La localisation précise est une autorisation séparée (iOS 14+ « Précise », Android « fine » vs « coarse »).

Le sélecteur dans la UI détermine le mode que nous demandons ; l’autorisation effective est définie dans les réglages de l’OS.

<div id="settings-model">
  ## Modèle de paramètres
</div>

Par nœud/appareil :

- `location.enabledMode`: `off | whileUsing | always`
- `location.preciseEnabled`: bool

Comportement de l’UI :

- Le choix de `whileUsing` demande l’autorisation au premier plan.
- Le choix de `always` garantit d’abord `whileUsing`, puis demande l’autorisation en arrière-plan (ou envoie l’utilisateur vers les Paramètres si nécessaire).
- Si le système d’exploitation refuse le niveau demandé, revenir au niveau le plus élevé accordé et afficher l’état.

<div id="permissions-mapping-nodepermissions">
  ## Correspondance des autorisations (node.permissions)
</div>

Optionnel. Le nœud macOS remonte `location` via la correspondance des autorisations ; iOS/Android peuvent l’omettre.

<div id="command-locationget">
  ## Commande : `location.get`
</div>

Appelée via `node.invoke`.

Paramètres (recommandés) :

```json
{
  "timeoutMs": 10000,
  "maxAgeMs": 15000,
  "desiredAccuracy": "coarse|balanced|precise"
}
```

Corps de la réponse :

```json
{
  "lat": 48.20849,
  "lon": 16.37208,
  "accuracyMeters": 12.5,
  "altitudeMeters": 182.0,
  "speedMps": 0.0,
  "headingDeg": 270.0,
  "timestamp": "2026-01-03T12:34:56.000Z",
  "isPrecise": true,
  "source": "gps|wifi|cell|unknown"
}
```

Erreurs (codes stables) :

* `LOCATION_DISABLED`: sélecteur de localisation désactivé.
* `LOCATION_PERMISSION_REQUIRED`: autorisation manquante pour le mode demandé.
* `LOCATION_BACKGROUND_UNAVAILABLE`: l’app est en arrière-plan mais seul « While Using » est autorisé.
* `LOCATION_TIMEOUT`: aucune position obtenue dans le délai imparti.
* `LOCATION_UNAVAILABLE`: échec du système / aucun fournisseur disponible.


<div id="background-behavior-future">
  ## Comportement en arrière-plan (futur)
</div>

Objectif : permettre au modèle de demander la localisation même lorsque le nœud est en arrière-plan, mais uniquement lorsque :

- L’utilisateur a sélectionné **Toujours**.
- L’OS autorise la localisation en arrière-plan.
- L’application est autorisée à s’exécuter en arrière-plan pour la localisation (mode arrière-plan iOS / service de premier plan Android ou autorisation spéciale).

Flux déclenché par push (futur) :

1) Gateway envoie un push au nœud (push silencieux ou données FCM).
2) Le nœud se réveille brièvement et demande la localisation à l’appareil.
3) Le nœud transmet le payload à Gateway.

Remarques :

- iOS : autorisation « Toujours » + mode localisation en arrière-plan requis. Le push silencieux peut être limité ; attendez-vous à des échecs intermittents.
- Android : la localisation en arrière-plan peut nécessiter un service de premier plan ; sinon, attendez-vous à un refus.

<div id="modeltooling-integration">
  ## Intégration modèles/outils
</div>

- Surface de l’outil : l’outil `nodes` ajoute l’action `location_get` (nœud requis).
- CLI : `openclaw nodes location get --node <id>`.
- Directives pour l’agent : n’appeler que lorsque l’utilisateur a activé la localisation et comprend la portée de cette fonctionnalité.

<div id="ux-copy-suggested">
  ## Texte UX (suggestion)
</div>

- Off : « Le partage de position est désactivé. »
- While Using : « Uniquement lorsque OpenClaw est ouvert. »
- Always : « Autoriser la localisation en arrière-plan. Nécessite une autorisation système. »
- Precise : « Utiliser la localisation GPS précise. Désactivez cette option pour partager une localisation approximative. »