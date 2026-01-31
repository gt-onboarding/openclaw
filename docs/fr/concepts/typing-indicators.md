---
title: Indicateurs de saisie
summary: "Quand OpenClaw affiche des indicateurs de saisie et comment les configurer"
read_when:
  - Modification du comportement ou des paramètres par défaut des indicateurs de saisie
---

<div id="typing-indicators">
  # Indicateurs de saisie
</div>

Les indicateurs de saisie sont envoyés au canal de discussion pendant qu'une exécution est active. Utilisez
`agents.defaults.typingMode` pour contrôler **quand** la saisie commence et `typingIntervalSeconds`
pour contrôler **à quelle fréquence** ils sont actualisés.

<div id="defaults">
  ## Valeurs par défaut
</div>

Lorsque `agents.defaults.typingMode` n’est **pas défini**, OpenClaw conserve le comportement historique :

- **Conversations directes** : l’indicateur de saisie s’active immédiatement dès que la boucle du modèle démarre.
- **Conversations de groupe avec mention** : l’indicateur de saisie s’active immédiatement.
- **Conversations de groupe sans mention** : l’indicateur de saisie ne s’active que lorsque le texte du message commence à être diffusé en continu.
- **Exécutions du signal de vie** : l’indicateur de saisie est désactivé.

<div id="modes">
  ## Modes
</div>

Définissez `agents.defaults.typingMode` sur l'une des valeurs suivantes :

- `never` — aucun indicateur de saisie, jamais.
- `instant` — affiche l'indicateur de saisie **dès que la boucle du modèle démarre**, même si l'exécution
  ne renvoie ensuite que le jeton de réponse silencieuse.
- `thinking` — affiche l'indicateur de saisie au **premier delta de raisonnement** (nécessite
  `reasoningLevel: "stream"` pour l'exécution).
- `message` — affiche l'indicateur de saisie au **premier delta de texte non silencieux** (ignore
  le jeton silencieux `NO_REPLY`).

Ordre de déclenchement « du plus tard au plus tôt » :
`never` → `message` → `thinking` → `instant`

<div id="configuration">
  ## Configuration
</div>

```json5
{
  agent: {
    typingMode: "thinking",
    typingIntervalSeconds: 6
  }
}
```

Vous pouvez surcharger le mode ou la cadence par session :

```json5
{
  session: {
    typingMode: "message",
    typingIntervalSeconds: 4
  }
}
```


<div id="notes">
  ## Notes
</div>

- Le mode `message` n’affichera pas d’indicateur de saisie pour les réponses uniquement silencieuses (par exemple le jeton `NO_REPLY`
  utilisé pour supprimer la sortie).
- `thinking` ne se déclenche que si l’exécution diffuse le raisonnement (`reasoningLevel: "stream"`).
  Si le modèle n’émet pas de deltas de raisonnement, l’indicateur de saisie ne démarre pas.
- Les signaux de vie n’affichent jamais d’indicateur de saisie, quel que soit le mode.
- `typingIntervalSeconds` contrôle la **cadence de rafraîchissement**, pas le moment de déclenchement.
  La valeur par défaut est de 6 secondes.