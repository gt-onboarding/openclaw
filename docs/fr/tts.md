---
title: Tts
summary: "Synthèse vocale (TTS) pour les réponses sortantes"
read_when:
  - Activation de la synthèse vocale pour les réponses
  - Configuration des fournisseurs ou des limites TTS
  - Utilisation des commandes /tts
---

<div id="text-to-speech-tts">
  # Synthèse vocale (TTS)
</div>

OpenClaw peut convertir les réponses sortantes en audio à l&#39;aide de ElevenLabs, OpenAI ou Edge TTS.
Cela fonctionne partout où OpenClaw peut envoyer de l&#39;audio ; sur Telegram, cela apparaît sous forme de bulle de note vocale.

<div id="supported-services">
  ## Services pris en charge
</div>

* **ElevenLabs** (fournisseur principal ou de secours)
* **OpenAI** (fournisseur principal ou de secours ; également utilisé pour les résumés)
* **Edge TTS** (fournisseur principal ou de secours ; utilise `node-edge-tts`, valeur par défaut si aucune clé API n’est fournie)

<div id="edge-tts-notes">
  ### Remarques sur Edge TTS
</div>

Edge TTS utilise le service en ligne de synthèse vocale neuronale de Microsoft Edge via la bibliothèque `node-edge-tts`.
C’est un service hébergé (non local), qui utilise les points de terminaison (endpoints) de Microsoft et ne
nécessite pas de clé API. `node-edge-tts` expose des options de configuration de la synthèse vocale et des formats de sortie, mais toutes les options ne sont pas prises en charge par le service Edge. citeturn2search0

Étant donné qu’Edge TTS est un service web public sans SLA (contrat de niveau de service) ni quota publiés, traitez‑le
comme du best effort. Si vous avez besoin de limites garanties et de support, utilisez OpenAI ou ElevenLabs.
L’API REST Speech de Microsoft documente une limite de 10 minutes d’audio par requête ; Edge TTS
ne publie pas de limites, donc partez du principe que les limites sont similaires ou inférieures. citeturn0search3

<div id="optional-keys">
  ## Clés facultatives
</div>

Si vous utilisez OpenAI ou ElevenLabs :

* `ELEVENLABS_API_KEY` (ou `XI_API_KEY`)
* `OPENAI_API_KEY`

Edge TTS **ne** nécessite **pas** de clé API. Si aucune clé API n’est trouvée, OpenClaw utilise
Edge TTS par défaut (sauf s’il est désactivé via `messages.tts.edge.enabled=false`).

Si plusieurs fournisseurs sont configurés, le fournisseur sélectionné est utilisé en premier et les autres servent d’options de repli.
Le résumé automatique utilise le `summaryModel` configuré (ou `agents.defaults.model.primary`) ;
ce fournisseur doit donc aussi être authentifié si vous activez les résumés.

<div id="service-links">
  ## Liens vers les services
</div>

* [Guide Text-to-Speech d’OpenAI](https://platform.openai.com/docs/guides/text-to-speech)
* [Référence de l’API Audio d’OpenAI](https://platform.openai.com/docs/api-reference/audio)
* [ElevenLabs Text-to-Speech](https://elevenlabs.io/docs/api-reference/text-to-speech)
* [Authentification ElevenLabs](https://elevenlabs.io/docs/api-reference/authentication)
* [node-edge-tts](https://github.com/SchneeHertz/node-edge-tts)
* [Formats de sortie de Microsoft Speech](https://learn.microsoft.com/azure/ai-services/speech-service/rest-text-to-speech#audio-outputs)

<div id="is-it-enabled-by-default">
  ## Est‑ce activé par défaut ?
</div>

Non. L’Auto‑TTS est **désactivé** par défaut. Pour l’activer, définis `messages.tts.auto` dans la configuration ou active‑le par session avec `/tts always` (alias : `/tts on`).

Edge TTS **est** activé par défaut une fois que le TTS est activé, et est utilisé automatiquement lorsqu’aucune clé api OpenAI ou ElevenLabs n’est configurée.

<div id="config">
  ## Config
</div>

La configuration TTS se trouve sous `messages.tts` dans le fichier `openclaw.json`.
Le schéma complet se trouve dans la section [Configuration du Gateway](/fr/gateway/configuration).

<div id="minimal-config-enable-provider">
  ### Configuration minimale (activation + fournisseur)
</div>

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "elevenlabs"
    }
  }
}
```

<div id="openai-primary-with-elevenlabs-fallback">
  ### OpenAI principal avec bascule sur ElevenLabs
</div>

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "openai",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy"
      },
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0
        }
      }
    }
  }
}
```

<div id="edge-tts-primary-no-api-key">
  ### Edge TTS principal (sans clé d’API)
</div>

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "edge",
      edge: {
        enabled: true,
        voice: "en-US-MichelleNeural",
        lang: "en-US",
        outputFormat: "audio-24khz-48kbitrate-mono-mp3",
        rate: "+10%",
        pitch: "-5%"
      }
    }
  }
}
```

<div id="disable-edge-tts">
  ### Désactiver Edge TTS
</div>

```json5
{
  messages: {
    tts: {
      edge: {
        enabled: false
      }
    }
  }
}
```

<div id="custom-limits-prefs-path">
  ### Limites personnalisées et chemin des préférences
</div>

```json5
{
  messages: {
    tts: {
      auto: "always",
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json"
    }
  }
}
```

<div id="only-reply-with-audio-after-an-inbound-voice-note">
  ### Ne répondre en audio qu&#39;après réception d&#39;une note vocale entrante
</div>

```json5
{
  messages: {
    tts: {
      auto: "inbound"
    }
  }
}
```

<div id="disable-auto-summary-for-long-replies">
  ### Désactiver le résumé automatique des réponses longues
</div>

```json5
{
  messages: {
    tts: {
      auto: "always"
    }
  }
}
```

Puis exécutez :

```
/tts summary off
```

<div id="notes-on-fields">
  ### Notes sur les champs
</div>

* `auto` : mode TTS automatique (`off`, `always`, `inbound`, `tagged`).
  * `inbound` n’envoie de l’audio qu’après une note vocale entrante.
  * `tagged` n’envoie de l’audio que lorsque la réponse inclut des balises `[[tts]]`.
* `enabled` : commutateur hérité (doctor migre ceci vers `auto`).
* `mode` : `"final"` (par défaut) ou `"all"` (inclut aussi les réponses d’outils/blocs).
* `provider` : `"elevenlabs"`, `"openai"` ou `"edge"` (le repli est automatique).
* Si `provider` n’est **pas défini**, OpenClaw privilégie `openai` (si clé), puis `elevenlabs` (si clé),
  sinon `edge`.
* `summaryModel` : modèle peu coûteux optionnel pour l’auto‑résumé ; par défaut `agents.defaults.model.primary`.
  * Accepte `provider/model` ou un alias de modèle configuré.
* `modelOverrides` : autorise le modèle à émettre des directives TTS (activé par défaut).
* `maxTextLength` : limite stricte pour l’entrée TTS (caractères). `/tts audio` échoue si cette limite est dépassée.
* `timeoutMs` : délai d’expiration de la requête (ms).
* `prefsPath` : remplace le chemin JSON des préférences locales (provider/limit/summary).
* Les valeurs `apiKey` se rabattent sur les variables d’environnement (`ELEVENLABS_API_KEY`/`XI_API_KEY`, `OPENAI_API_KEY`).
* `elevenlabs.baseUrl` : remplace l’URL de base de l’API ElevenLabs.
* `elevenlabs.voiceSettings` :
  * `stability`, `similarityBoost`, `style` : `0..1`
  * `useSpeakerBoost` : `true|false`
  * `speed` : `0.5..2.0` (1.0 = normal)
* `elevenlabs.applyTextNormalization` : `auto|on|off`
* `elevenlabs.languageCode` : code ISO 639-1 à 2 lettres (par ex. `en`, `de`)
* `elevenlabs.seed` : entier `0..4294967295` (déterminisme dans la mesure du possible)
* `edge.enabled` : autorise l’utilisation de Edge TTS (par défaut `true` ; pas de clé API).
* `edge.voice` : nom de voix neuronale Edge (par ex. `en-US-MichelleNeural`).
* `edge.lang` : code de langue (par ex. `en-US`).
* `edge.outputFormat` : format de sortie Edge (par ex. `audio-24khz-48kbitrate-mono-mp3`).
  * Consultez les formats de sortie Microsoft Speech pour les valeurs valides ; tous les formats ne sont pas pris en charge par Edge.
* `edge.rate` / `edge.pitch` / `edge.volume` : chaînes de pourcentage (par ex. `+10%`, `-5%`).
* `edge.saveSubtitles` : écrit des sous-titres JSON en parallèle du fichier audio.
* `edge.proxy` : URL de proxy pour les requêtes Edge TTS.
* `edge.timeoutMs` : surcharge du délai d’expiration de la requête (ms).

<div id="model-driven-overrides-default-on">
  ## Remplacements pilotés par le modèle (activés par défaut)
</div>

Par défaut, le modèle **peut** émettre des directives TTS pour une seule réponse.
Quand `messages.tts.auto` est défini sur `tagged`, ces directives sont requises pour déclencher l’audio.

Quand cette option est activée, le modèle peut émettre des directives `[[tts:...]]` pour remplacer la voix
d’une seule réponse, ainsi qu’un bloc optionnel `[[tts:text]]...[[/tts:text]]`
pour fournir des balises expressives (rire, indications de chant, etc.) qui ne doivent apparaître
que dans l’audio.

Exemple de charge utile de réponse :

```
Here you go.

[[tts:provider=elevenlabs voiceId=pMsXgVXv3BLzUgSXRplE model=eleven_v3 speed=1.1]]
[[tts:text]](laughs) Read the song once more.[[/tts:text]]
```

Clés de directive disponibles (lorsqu’elles sont activées) :

* `provider` (`openai` | `elevenlabs` | `edge`)
* `voice` (voix OpenAI) ou `voiceId` (ElevenLabs)
* `model` (modèle TTS OpenAI ou identifiant de modèle ElevenLabs)
* `stability`, `similarityBoost`, `style`, `speed`, `useSpeakerBoost`
* `applyTextNormalization` (`auto|on|off`)
* `languageCode` (ISO 639-1)
* `seed`

Désactiver tous les remplacements de modèle :

```json5
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: false
      }
    }
  }
}
```

Liste d’autorisation facultative (désactiver certains réglages spécifiques tout en gardant les balises activées) :

```json5
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: true,
        allowProvider: false,
        allowSeed: false
      }
    }
  }
}
```

<div id="per-user-preferences">
  ## Préférences par utilisateur
</div>

Les commandes slash écrivent des remplacements locaux dans `prefsPath` (par défaut :
`~/.openclaw/settings/tts.json`, modifiable via `OPENCLAW_TTS_PREFS` ou
`messages.tts.prefsPath`).

Champs enregistrés :

* `enabled`
* `provider`
* `maxLength` (seuil de synthèse ; 1500 caractères par défaut)
* `summarize` (par défaut `true`)

Ces paramètres remplacent `messages.tts.*` pour cet hôte.

<div id="output-formats-fixed">
  ## Formats de sortie (prédéfinis)
</div>

* **Telegram** : note vocale Opus (`opus_48000_64` provenant d’ElevenLabs, `opus` provenant d’OpenAI).
  * 48kHz / 64kbps est un bon compromis pour les notes vocales et nécessaire pour la bulle ronde.
* **Autres canaux** : MP3 (`mp3_44100_128` provenant d’ElevenLabs, `mp3` provenant d’OpenAI).
  * 44,1kHz / 128kbps est l’équilibre par défaut pour la clarté de la voix.
* **Edge TTS** : utilise `edge.outputFormat` (par défaut `audio-24khz-48kbitrate-mono-mp3`).
  * `node-edge-tts` accepte un `outputFormat`, mais tous les formats ne sont pas disponibles
    auprès du service Edge. citeturn2search0
  * Les valeurs de format de sortie suivent les formats de sortie Microsoft Speech (y compris Ogg/WebM Opus). citeturn1search0
  * Telegram `sendVoice` accepte OGG/MP3/M4A ; utilisez OpenAI/ElevenLabs si vous avez besoin
    de notes vocales Opus garanties. citeturn1search1
  * Si le format de sortie Edge configuré échoue, OpenClaw réessaie avec du MP3.

Les formats OpenAI/ElevenLabs sont figés ; Telegram s’attend à de l’Opus pour l’UX des notes vocales.

<div id="auto-tts-behavior">
  ## Comportement de l’Auto-TTS
</div>

Lorsqu’il est activé, OpenClaw :

* n’effectue pas de TTS si la réponse contient déjà un média ou une directive `MEDIA:`.
* ignore les réponses très courtes (&lt; 10 caractères).
* résume les réponses longues, lorsque cette option est activée, en utilisant `agents.defaults.model.primary` (ou `summaryModel`).
* joint l’audio généré à la réponse.

Si la réponse dépasse `maxLength` et que le résumé est désactivé (ou qu’aucune clé d’API n’est configurée pour le modèle de résumé), l’audio est omis et la réponse texte normale est envoyée.

<div id="flow-diagram">
  ## Schéma de flux
</div>

```
Reply -> TTS enabled?
  no  -> send text
  yes -> has media / MEDIA: / short?
          yes -> send text
          no  -> length > limit?
                   no  -> TTS -> attach audio
                   yes -> summary enabled?
                            no  -> send text
                            yes -> summarize (summaryModel or agents.defaults.model.primary)
                                      -> TTS -> attach audio
```

<div id="slash-command-usage">
  ## Utilisation de la commande slash
</div>

Il n&#39;y a qu&#39;une seule commande : `/tts`.
Consultez [Commandes slash](/fr/tools/slash-commands) pour les détails d&#39;activation.

Remarque Discord : `/tts` est une commande intégrée à Discord, donc OpenClaw enregistre
`/voice` comme commande native sur cette plateforme. La saisie de `/tts ...` fonctionne toujours.

```
/tts off
/tts always
/tts inbound
/tts tagged
/tts status
/tts provider openai
/tts limit 2000
/tts summary off
/tts audio Hello from OpenClaw
```

Remarques :

* Les commandes nécessitent un expéditeur autorisé (les règles de liste d’autorisation/propriétaire s’appliquent toujours).
* `commands.text` ou l’enregistrement natif des commandes doit être activé.
* `off|always|inbound|tagged` sont des paramètres par session (`/tts on` est un alias de `/tts always`).
* `limit` et `summary` sont stockés dans les préférences locales, pas dans la configuration principale.
* `/tts audio` génère une réponse audio unique (sans activer le TTS).

<div id="agent-tool">
  ## Outil d&#39;agent
</div>

L&#39;outil `tts` convertit du texte en synthèse vocale et renvoie un chemin `MEDIA:`. Lorsque
le résultat est compatible avec Telegram, l&#39;outil inclut `[[audio_as_voice]]` afin que
Telegram l&#39;envoie sous forme de bulle de message vocal.

<div id="gateway-rpc">
  ## RPC du Gateway
</div>

Méthodes du Gateway :

* `tts.status`
* `tts.enable`
* `tts.disable`
* `tts.convert`
* `tts.setProvider`
* `tts.providers`