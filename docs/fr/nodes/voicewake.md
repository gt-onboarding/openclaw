---
title: Réveil vocal
summary: "Mots d’activation vocale globaux (gérés par le Gateway) et leur synchronisation entre les nœuds"
read_when:
  - Modifier le comportement ou les valeurs par défaut des mots d’activation vocale
  - Ajouter de nouvelles plateformes de nœuds devant être synchronisées pour les mots d’activation
---

<div id="voice-wake-global-wake-words">
  # Réveil vocal (mots de réveil globaux)
</div>

OpenClaw traite **les mots de réveil comme une unique liste globale** gérée par le **Gateway**.

- Il n’existe **aucun mot de réveil personnalisé par nœud**.
- **N’importe quel nœud ou UI d’app peut modifier** la liste ; les changements sont enregistrés par le Gateway et diffusés à tout le monde.
- Chaque appareil conserve tout de même sa propre option **Réveil vocal activé/désactivé** (l’UX locale et les autorisations diffèrent).

<div id="storage-gateway-host">
  ## Stockage (hôte du Gateway)
</div>

Les mots d’activation sont stockés sur la machine qui héberge le Gateway à l’emplacement :

* `~/.openclaw/settings/voicewake.json`

Structure :

```json
{ "triggers": ["openclaw", "claude", "computer"], "updatedAtMs": 1730000000000 }
```


<div id="protocol">
  ## Protocole
</div>

<div id="methods">
  ### Méthodes
</div>

- `voicewake.get` → `{ triggers: string[] }`
- `voicewake.set` avec les paramètres `{ triggers: string[] }` → `{ triggers: string[] }`

Remarques :

- Les déclencheurs sont normalisés (espaces en bordure supprimés, éléments vides ignorés). Les listes vides reviennent aux valeurs par défaut.
- Des limites sont appliquées pour des raisons de sécurité (plafonds sur le nombre d’éléments et leur longueur).

<div id="events">
  ### Événements
</div>

- `voicewake.changed` payload `{ triggers: string[] }`

Qui le reçoit :

- Tous les clients WebSocket (application macOS, WebChat, etc.)
- Tous les nœuds connectés (iOS/Android), et également lors de la connexion d’un nœud, sous la forme d’un envoi initial de l’« état actuel ».

<div id="client-behavior">
  ## Comportement côté client
</div>

<div id="macos-app">
  ### Application macOS
</div>

- Utilise la liste globale pour contrôler les déclencheurs `VoiceWakeRuntime`.
- La modification des « Mots de déclenchement » dans les paramètres Voice Wake appelle `voicewake.set`, puis se repose sur la diffusion pour maintenir les autres clients synchronisés.

<div id="ios-node">
  ### Nœud iOS
</div>

- Utilise la liste globale pour la détection des déclencheurs de `VoiceWakeManager`.
- La modification des mots de réveil dans les Paramètres appelle `voicewake.set` (via le WS du Gateway) et garantit que la détection locale des mots de réveil reste réactive.

<div id="android-node">
  ### Nœud Android
</div>

- Expose un éditeur de mots de réveil dans Paramètres.
- Appelle `voicewake.set` via le WS du Gateway pour que les modifications soient synchronisées partout.