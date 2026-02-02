---
title: Voicewake
summary: "Modes de réveil vocal et de push-to-talk, ainsi que les détails de routage dans l'app macOS"
read_when:
  - Lorsque vous travaillez sur les flux de réveil vocal ou de PTT
---

<div id="voice-wake-push-to-talk">
  # Activation vocale et appui‑pour‑parler
</div>

<div id="modes">
  ## Modes
</div>

- **Mode mot-clé de réveil** (par défaut) : le moteur de reconnaissance vocale toujours actif attend les mots déclencheurs (`swabbleTriggerWords`). En cas de correspondance, il démarre la capture, affiche l’overlay avec le texte partiel et l’envoie automatiquement après un silence.
- **Push-to-talk (maintien de la touche Option droite)** : maintenez la touche Option droite pour capturer immédiatement — aucun déclencheur n’est nécessaire. L’overlay apparaît pendant que la touche est enfoncée ; son relâchement finalise la capture et envoie le texte après un court délai afin que vous puissiez l’ajuster.

<div id="runtime-behavior-wake-word">
  ## Comportement à l’exécution (mot d’activation)
</div>

- Le moteur de reconnaissance vocale réside dans `VoiceWakeRuntime`.
- Le déclencheur ne se produit que lorsqu’il y a une **pause significative** entre le mot d’activation et le mot suivant (~0,55 s d’intervalle). L’overlay ou le carillon peuvent démarrer pendant cette pause, avant même que la commande ne commence.
- Fenêtres de silence : 2,0 s lorsque la parole est continue, 5,0 s si seul le déclencheur a été entendu.
- Arrêt forcé : 120 s pour éviter les sessions incontrôlées.
- Délai anti-rebond entre les sessions : 350 ms.
- L’overlay est piloté via `VoiceWakeOverlayController` avec une coloration persistante/volatile.
- Après l’envoi, le moteur de reconnaissance vocale redémarre proprement pour écouter le prochain déclencheur.

<div id="lifecycle-invariants">
  ## Invariants de cycle de vie
</div>

- Si Voice Wake est activé et que les autorisations nécessaires ont été accordées, le détecteur de mot d’activation doit être à l’écoute (sauf pendant une capture explicite en push-to-talk).
- La visibilité de l’overlay (y compris sa fermeture manuelle via le bouton X) ne doit jamais empêcher le détecteur de reprendre l’écoute.

<div id="sticky-overlay-failure-mode-previous">
  ## Échec de l’overlay persistant (comportement précédent)
</div>

Auparavant, si l’overlay restait affiché et que vous le fermiez manuellement, Voice Wake pouvait sembler « mort » car la tentative de redémarrage du runtime pouvait être bloquée par la visibilité de l’overlay et aucun redémarrage ultérieur n’était planifié.

Durcissement :

- Le redémarrage du runtime Wake n’est plus bloqué par la visibilité de l’overlay.
- La fin de la fermeture de l’overlay déclenche un `VoiceWakeRuntime.refresh(...)` via `VoiceSessionCoordinator`, de sorte qu’une fermeture manuelle via le X relance toujours l’écoute.

<div id="push-to-talk-specifics">
  ## Spécificités du push-to-talk
</div>

- La détection de raccourci clavier utilise un moniteur `.flagsChanged` global pour la touche **Option droite** (`keyCode 61` + `.option`). Nous nous contentons d’observer les événements (aucune interception).
- Le pipeline de capture réside dans `VoicePushToTalk` : il démarre Speech immédiatement, diffuse les résultats partiels vers l’overlay et appelle `VoiceWakeForwarder` au relâchement.
- Lorsque le push-to-talk démarre, nous mettons le runtime du mot d’activation en pause pour éviter des prises audio concurrentes ; il redémarre automatiquement après le relâchement.
- Permissions : nécessite Microphone + Speech ; l’observation des événements requiert l’approbation Accessibilité / Surveillance de l’entrée.
- Claviers externes : certains peuvent ne pas exposer la touche Option droite comme prévu — prévoyez un raccourci de secours si des utilisateurs signalent des ratés.

<div id="user-facing-settings">
  ## Paramètres visibles par l’utilisateur
</div>

- Interrupteur **Voice Wake** : active le réveil par mot déclencheur.
- **Maintenir Cmd+Fn pour parler** : active le mode appuyer‑pour‑parler. Désactivé sur macOS &lt; 26.
- Sélecteurs de langue et de micro, vumètre en direct, tableau des mots déclencheurs, outil de test (local uniquement ; ne transmet rien).
- Le sélecteur de micro conserve la dernière sélection si un appareil est déconnecté, affiche une indication de déconnexion et revient temporairement à la valeur par défaut du système jusqu’au retour de l’appareil.
- **Sons** : tintement lors de la détection du déclencheur et à l’envoi ; par défaut, le son système macOS « Glass ». Vous pouvez choisir n’importe quel fichier chargeable par `NSSound` (par ex. MP3/WAV/AIFF) pour chaque événement ou sélectionner **Aucun son**.

<div id="forwarding-behavior">
  ## Comportement de transfert
</div>

- Lorsque Voice Wake est activé, les transcriptions sont acheminées vers le Gateway ou l’agent actif (en utilisant le même mode local vs distant que le reste de l’app macOS).
- Les réponses sont envoyées au **dernier fournisseur principal utilisé** (WhatsApp/Telegram/Discord/WebChat). En cas d’échec de l’envoi, l’erreur est consignée dans les logs et l’exécution reste visible dans WebChat et dans les logs de session.

<div id="forwarding-payload">
  ## Acheminement de la charge utile
</div>

- `VoiceWakeForwarder.prefixedTranscript(_:)` ajoute en préfixe l’indice machine avant d’envoyer. Partagé entre les chemins de traitement mot‑clé de réveil et appuyer‑pour‑parler.

<div id="quick-verification">
  ## Vérification rapide
</div>

- Active le mode appuyer‑pour‑parler, maintiens Cmd+Fn, parle, relâche : la superposition doit afficher des transcriptions partielles puis envoyer le message.
- Pendant que tu maintiens les touches enfoncées, les oreilles dans la barre de menu doivent rester agrandies (utilise `triggerVoiceEars(ttl:nil)`); elles se rétractent après le relâchement.