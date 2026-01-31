---
title: Webchat
summary: "Comment l’app macOS intègre le WebChat du Gateway et comment le déboguer"
read_when:
  - Débogage de la vue WebChat de l’app macOS ou du port loopback
---

<div id="webchat-macos-app">
  # WebChat (app macOS)
</div>

L’app de barre de menus macOS intègre l’UI WebChat comme vue SwiftUI native. Elle
se connecte au Gateway et utilise par défaut la **session principale** pour l’agent
sélectionné (avec un sélecteur de session pour les autres sessions).

- **Mode local** : se connecte directement au WebSocket local du Gateway.
- **Mode distant** : transfère le port de contrôle du Gateway via SSH et utilise ce
  tunnel comme plan de données.

<div id="launch-debugging">
  ## Lancement et débogage
</div>

- Manuel : menu Lobster → « Open Chat ».
- Ouverture automatique pour les tests :
  ```bash
  dist/OpenClaw.app/Contents/MacOS/OpenClaw --webchat
  ```
- Logs : `./scripts/clawlog.sh` (sous-système `bot.molt`, catégorie `WebChatSwiftUI`).

<div id="how-its-wired">
  ## Architecture de connexion
</div>

- Plan de données : méthodes WS du Gateway `chat.history`, `chat.send`, `chat.abort`,
  `chat.inject` et événements `chat`, `agent`, `presence`, `tick`, `health`.
- Session : par défaut, la session principale est utilisée (`main`, ou `global` lorsque le scope est
  global). L’UI peut passer d’une session à l’autre.
- L’onboarding utilise une session dédiée pour séparer la configuration au premier lancement.

<div id="security-surface">
  ## Surface d’exposition
</div>

- En mode distant, seul le port de contrôle WebSocket du Gateway est redirigé via SSH.

<div id="known-limitations">
  ## Limitations connues
</div>

- L’UI est optimisée pour les sessions de chat (ce n’est pas un sandbox de navigateur à part entière).