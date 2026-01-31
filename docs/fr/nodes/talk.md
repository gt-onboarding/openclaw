---
title: Talk
summary: "Mode Talk : conversations vocales continues avec ElevenLabs TTS"
read_when:
  - Implémentation du mode Talk sur macOS/iOS/Android
  - Modification de la voix/du TTS/du comportement d’interruption
---

<div id="talk-mode">
  # Mode discussion vocale
</div>

Le mode discussion vocale est une boucle de conversation vocale continue :

1) Écouter la voix de l’utilisateur
2) Envoyer la transcription au modèle (session principale, chat.send)
3) Attendre la réponse
4) La faire lire par ElevenLabs (lecture en streaming)

<div id="behavior-macos">
  ## Comportement (macOS)
</div>

- **Overlay toujours visible** tant que le mode Talk est activé.
- Transitions de phase **Écoute → Réflexion → Parole**.
- Lors d'une **courte pause** (fenêtre de silence), la transcription en cours est envoyée.
- Les réponses sont **écrites dans WebChat** (comme si elles étaient saisies au clavier).
- **Interruption vocale** (activé par défaut) : si l'utilisateur commence à parler pendant que l'assistant parle, nous arrêtons la lecture et enregistrons l'horodatage de l'interruption pour le prochain prompt.

<div id="voice-directives-in-replies">
  ## Directives vocales dans les réponses
</div>

L’assistant peut préfixer sa réponse par **une seule ligne JSON** pour contrôler la voix utilisée :

```json
{"voice":"<voice-id>","once":true}
```

Règles :

* Seule la première ligne non vide est prise en compte.
* Les clés inconnues sont ignorées.
* `once: true` s’applique uniquement à la réponse actuelle.
* Sans `once`, la voix devient la nouvelle valeur par défaut pour le mode Talk.
* La ligne JSON est supprimée avant la lecture TTS.

Clés prises en charge :

* `voice` / `voice_id` / `voiceId`
* `model` / `model_id` / `modelId`
* `speed`, `rate` (WPM), `stability`, `similarity`, `style`, `speakerBoost`
* `seed`, `normalize`, `lang`, `output_format`, `latency_tier`
* `once`


<div id="config-openclawopenclawjson">
  ## Config (`~/.openclaw/openclaw.json`)
</div>

```json5
{
  "talk": {
    "voiceId": "elevenlabs_voice_id",
    "modelId": "eleven_v3",
    "outputFormat": "mp3_44100_128",
    "apiKey": "elevenlabs_api_key",
    "interruptOnSpeech": true
  }
}
```

Valeurs par défaut :

* `interruptOnSpeech` : true
* `voiceId` : utilise `ELEVENLABS_VOICE_ID` / `SAG_VOICE_ID` par défaut (ou la première voix ElevenLabs lorsque la clé API est disponible)
* `modelId` : utilise `eleven_v3` par défaut lorsqu’il n’est pas défini
* `apiKey` : utilise `ELEVENLABS_API_KEY` par défaut (ou le profil shell du Gateway si disponible)
* `outputFormat` : utilise `pcm_44100` par défaut sur macOS/iOS et `pcm_24000` sur Android (définissez `mp3_*` pour forcer le streaming MP3)


<div id="macos-ui">
  ## UI macOS
</div>

- Commutateur dans la barre de menu : **Talk**
- Onglet de configuration : groupe **Talk Mode** (ID de voix + interrupteur d’interruption)
- Superposition :
  - **Listening** : nuage pulsant au niveau du micro
  - **Thinking** : animation de plongée
  - **Speaking** : anneaux rayonnants
  - Cliquer sur le nuage : arrêter la synthèse vocale
  - Cliquer sur X : quitter le mode Talk

<div id="notes">
  ## Notes
</div>

- Nécessite les autorisations pour la reconnaissance vocale et le microphone.
- Utilise `chat.send` avec la clé de session `main`.
- Le TTS utilise l'API de streaming ElevenLabs avec `ELEVENLABS_API_KEY` et une lecture incrémentielle sur macOS/iOS/Android pour réduire la latence.
- `stability` pour `eleven_v3` est validé aux valeurs `0.0`, `0.5` ou `1.0` ; les autres modèles acceptent `0..1`.
- `latency_tier` est validé à `0..4` lorsqu'il est défini.
- Android prend en charge les formats de sortie `pcm_16000`, `pcm_22050`, `pcm_24000` et `pcm_44100` pour le streaming AudioTrack à faible latence.