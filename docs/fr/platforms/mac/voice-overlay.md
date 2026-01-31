---
title: Superposition vocale
summary: "Cycle de vie de la superposition vocale lorsque le mot d’activation et le push-to-talk se chevauchent"
read_when:
  - Ajuster le comportement de la superposition vocale
---

<div id="voice-overlay-lifecycle-macos">
  # Cycle de vie de l’overlay vocal (macOS)
</div>

Public cible : contributeurs à l’app macOS. Objectif : garder l’overlay vocal prévisible lorsque l’activation par mot-clé et l’appui‑pour‑parler se chevauchent.

<div id="current-intent">
  ### Intention actuelle
</div>

- Si l’overlay est déjà visible grâce au mot de réveil et que l’utilisateur appuie sur le raccourci clavier, la session associée au raccourci *adopte* le texte existant au lieu de le réinitialiser. L’overlay reste affichée tant que le raccourci est maintenu. Au relâchement : envoyer s’il y a du texte après troncature, sinon fermer.
- Le mot de réveil seul continue d’envoyer automatiquement après un silence ; le mode push-to-talk envoie immédiatement au relâchement.

<div id="implemented-dec-9-2025">
  ### Implémenté (9 décembre 2025)
</div>

- Les sessions d’overlay portent désormais un jeton par capture (mot de réveil ou appuyer‑pour‑parler). Les mises à jour partial/final/send/dismiss/level sont ignorées lorsque le jeton ne correspond pas, ce qui évite les rappels obsolètes.
- Appuyer‑pour‑parler adopte tout texte d’overlay visible comme préfixe (ainsi, appuyer sur le raccourci clavier pendant que l’overlay de réveil est affiché conserve le texte et y ajoute la nouvelle parole). Il attend jusqu’à 1,5 s une transcription finale avant de revenir au texte actuel.
- La journalisation chime/overlay est émise au niveau `info` dans les catégories `voicewake.overlay`, `voicewake.ptt` et `voicewake.chime` (démarrage de session, partial, final, send, dismiss, raison du signal sonore).

<div id="next-steps">
  ### Étapes suivantes
</div>

1. **VoiceSessionCoordinator (actor)**
   - Possède exactement une `VoiceSession` à la fois.
   - API (basée sur des jetons) : `beginWakeCapture`, `beginPushToTalk`, `updatePartial`, `endCapture`, `cancel`, `applyCooldown`.
   - Ignore les callbacks portant des jetons périmés (empêche d’anciens moteurs de reconnaissance de rouvrir l’overlay).
2. **VoiceSession (model)**
   - Champs : `token`, `source` (wakeWord|pushToTalk), texte validé/volatile, indicateurs de carillon, minuteries (envoi automatique, inactivité), `overlayMode` (display|editing|sending), échéance de cooldown.
3. **Liaison avec l’overlay**
   - `VoiceSessionPublisher` (`ObservableObject`) reflète la session active dans SwiftUI.
   - `VoiceWakeOverlayView` est rendu uniquement via le publisher ; il ne modifie jamais directement les singletons globaux.
   - Les actions utilisateur de l’overlay (`sendNow`, `dismiss`, `edit`) rappellent le coordinateur avec le jeton de session.
4. **Chemin d’envoi unifié**
   - Sur `endCapture` : si le texte tronqué est vide → fermer ; sinon `performSend(session:)` (joue une seule fois le carillon d’envoi, transfère, ferme).
   - Push-to-talk : aucun délai ; wake-word : délai optionnel pour l’envoi automatique.
   - Applique un court cooldown au runtime du wake-word après la fin du push-to-talk afin que le wake-word ne se réenclenche pas immédiatement.
5. **Journalisation**
   - Le coordinateur émet des journaux `.info` dans le sous-système `bot.molt`, catégories `voicewake.overlay` et `voicewake.chime`.
   - Événements clés : `session_started`, `adopted_by_push_to_talk`, `partial`, `finalized`, `send`, `dismiss`, `cancel`, `cooldown`.

<div id="debugging-checklist">
  ### Liste de vérifications pour le débogage
</div>

- Affichez les journaux en continu pendant que vous reproduisez une superposition persistante :

  ```bash
  sudo log stream --predicate 'subsystem == "bot.molt" AND category CONTAINS "voicewake"' --level info --style compact
  ```
- Vérifiez qu'il n'y a qu'un seul jeton de session actif ; les callbacks obsolètes doivent être ignorés par le coordinateur.
- Assurez-vous que le relâchement du bouton push-to-talk appelle toujours `endCapture` avec le jeton actif ; si le texte est vide, attendez-vous à un `dismiss` sans signal sonore ni envoi.

<div id="migration-steps-suggested">
  ### Étapes de migration (recommandées)
</div>

1. Ajouter `VoiceSessionCoordinator`, `VoiceSession` et `VoiceSessionPublisher`.
2. Refactoriser `VoiceWakeRuntime` pour créer, mettre à jour et terminer des sessions plutôt que de manipuler directement `VoiceWakeOverlayController`.
3. Refactoriser `VoicePushToTalk` pour réutiliser les sessions existantes et appeler `endCapture` au relâchement ; appliquer un cooldown côté runtime.
4. Connecter `VoiceWakeOverlayController` au publisher ; supprimer les appels directs depuis le runtime/PTT.
5. Ajouter des tests d'intégration pour l’adoption de sessions, le cooldown et la fermeture en cas de texte vide.