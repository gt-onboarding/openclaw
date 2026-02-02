---
title: Compréhension des médias
summary: "Compréhension des images/audio/vidéo entrantes (optionnelle) avec solutions de repli via le fournisseur et la CLI"
read_when:
  - Concevoir ou remanier la compréhension des médias
  - Ajuster le prétraitement de l'audio/vidéo/image entrantes
---

<div id="media-understanding-inbound-2026-01-17">
  # Compréhension des médias entrants — 2026-01-17
</div>

OpenClaw peut **résumer les médias entrants** (image/audio/vidéo) avant l’exécution du pipeline de réponse. Il détecte automatiquement la disponibilité d’outils locaux ou de clés de fournisseur, et peut être désactivé ou personnalisé. Si la compréhension est désactivée, les modèles reçoivent tout de même les fichiers/URL d’origine comme d’habitude.

<div id="goals">
  ## Objectifs
</div>

* Facultatif : pré‑traiter les médias entrants en texte court pour un routage plus rapide et une meilleure analyse des commandes.
* Préserver systématiquement la remise des médias originaux au modèle.
* Prendre en charge les **API des fournisseurs** et les **solutions de repli via la CLI**.
* Permettre l’utilisation de plusieurs modèles avec bascule ordonnée (erreur/taille/dépassement de délai).

<div id="highlevel-behavior">
  ## Comportement d’ensemble
</div>

1. Collecter les pièces jointes entrantes (`MediaPaths`, `MediaUrls`, `MediaTypes`).
2. Pour chaque fonctionnalité activée (image/audio/vidéo), sélectionner les pièces jointes selon la stratégie (par défaut : **la première**).
3. Choisir la première entrée de modèle éligible (taille + fonctionnalité + authentification).
4. Si un modèle échoue ou si le média est trop volumineux, **revenir à l’entrée suivante**.
5. En cas de réussite :
   * `Body` devient un bloc `[Image]`, `[Audio]` ou `[Video]`.
   * L’audio définit `{{Transcript}}` ; l’analyse des commandes utilise le texte de légende lorsqu’il est présent,
     sinon la transcription.
   * Les légendes sont conservées comme `User text:` à l’intérieur du bloc.

Si la compréhension échoue ou est désactivée, **le flux de réponse continue** avec le corps d’origine + les pièces jointes.

<div id="config-overview">
  ## Aperçu de la configuration
</div>

`tools.media` prend en charge des **modèles partagés** ainsi que des surcharges spécifiques par capacité :

* `tools.media.models` : liste de modèles partagés (utilisez `capabilities` pour filtrer).
* `tools.media.image` / `tools.media.audio` / `tools.media.video` :
  * valeurs par défaut (`prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`)
  * surcharges de fournisseur (`baseUrl`, `headers`, `providerOptions`)
  * options audio Deepgram via `tools.media.audio.providerOptions.deepgram`
  * **liste de `models` par capacité** optionnelle (prioritaire sur les modèles partagés)
  * stratégie `attachments` (`mode`, `maxAttachments`, `prefer`)
  * `scope` (restriction optionnelle par clés channel/chatType/session)
* `tools.media.concurrency` : nombre maximal d&#39;exécutions simultanées par capacité (par défaut **2**).

```json5
{
  tools: {
    media: {
      models: [ /* shared list */ ],
      image: { /* surcharges optionnelles */ },
      audio: { /* surcharges optionnelles */ },
      video: { /* surcharges optionnelles */ }
    }
  }
}
```

<div id="model-entries">
  ### Entrées de modèles
</div>

Chaque entrée dans `models[]` peut être un **fournisseur** ou la **CLI** :

```json5
{
  type: "provider",        // default if omitted
  provider: "openai",
  model: "gpt-5.2",
  prompt: "Describe the image in <= 500 chars.",
  maxChars: 500,
  maxBytes: 10485760,
  timeoutSeconds: 60,
  capabilities: ["image"], // optionnel, utilisé pour les entrées multimodales
  profile: "vision-profile",
  preferredProfile: "vision-fallback"
}
```

```json5
{
  type: "cli",
  command: "gemini",
  args: [
    "-m",
    "gemini-3-flash",
    "--allowed-tools",
    "read_file",
    "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters."
  ],
  maxChars: 500,
  maxBytes: 52428800,
  timeoutSeconds: 120,
  capabilities: ["video", "image"]
}
```

Les modèles CLI peuvent également utiliser :

* `{{MediaDir}}` (répertoire contenant le fichier multimédia)
* `{{OutputDir}}` (répertoire temporaire créé pour cette exécution)
* `{{OutputBase}}` (chemin de base du fichier temporaire, sans extension)

<div id="defaults-and-limits">
  ## Valeurs par défaut et limites
</div>

Valeurs par défaut recommandées :

* `maxChars` : **500** pour l’image/vidéo (court, adapté aux commandes)
* `maxChars` : **non défini** pour l’audio (transcription complète, sauf si vous imposez une limite)
* `maxBytes` :
  * image : **10MB**
  * audio : **20MB**
  * vidéo : **50MB**

Règles :

* Si le média dépasse `maxBytes`, ce modèle est ignoré et **le modèle suivant est utilisé**.
* Si le modèle renvoie plus que `maxChars`, la sortie est tronquée.
* `prompt` a par défaut pour valeur une simple invite « Describe the {media}. », complétée par l’indication `maxChars` (image/vidéo uniquement).
* Si `<capability>.enabled: true` mais qu’aucun modèle n’est configuré, OpenClaw essaie le
  **modèle de réponse actif** lorsque son fournisseur prend en charge cette capacité.

<div id="auto-detect-media-understanding-default">
  ### Détection automatique de la compréhension des médias (par défaut)
</div>

Si `tools.media.<capability>.enabled` n’est **pas** réglé sur `false` et que vous
n’avez pas configuré de modèles, OpenClaw effectue une détection automatique dans cet ordre
et **s’arrête à la première option fonctionnelle** :

1. **CLI locaux** (audio uniquement ; si installés)
   * `sherpa-onnx-offline` (requiert `SHERPA_ONNX_MODEL_DIR` avec encoder/decoder/joiner/tokens)
   * `whisper-cli` (`whisper-cpp` ; utilise `WHISPER_CPP_MODEL` ou le modèle « tiny » fourni)
   * `whisper` (CLI Python ; télécharge automatiquement les modèles)
2. **Gemini CLI** (`gemini`) utilisant `read_many_files`
3. **Clés de fournisseurs**
   * Audio : OpenAI → Groq → Deepgram → Google
   * Image : OpenAI → Anthropic → Google → MiniMax
   * Vidéo : Google

Pour désactiver la détection automatique, définissez :

```json5
{
  tools: {
    media: {
      audio: {
        enabled: false
      }
    }
  }
}
```

Remarque : la détection de l’exécutable est effectuée au mieux sur macOS/Linux/Windows ; assurez-vous que la CLI est présente dans le `PATH` (nous développons le raccourci `~`), ou définissez un modèle CLI explicite avec un chemin complet de la commande.

<div id="capabilities-optional">
  ## Capacités (facultatif)
</div>

Si vous définissez `capabilities`, l’entrée ne s’exécute que pour ces types de média. Pour les listes partagées, OpenClaw peut inférer automatiquement des valeurs par défaut :

* `openai`, `anthropic`, `minimax` : **image**
* `google` (Gemini API) : **image + audio + vidéo**
* `groq` : **audio**
* `deepgram` : **audio**

Pour les entrées CLI, **définissez `capabilities` explicitement** pour éviter des correspondances inattendues.
Si vous omettez `capabilities`, l’entrée est éligible pour la liste dans laquelle elle apparaît.

<div id="provider-support-matrix-openclaw-integrations">
  ## Matrice de prise en charge des fournisseurs (intégrations OpenClaw)
</div>

| Fonctionnalité | Intégration de fournisseur | Remarques |
|----------------|---------------------------|-----------|
| Image | OpenAI / Anthropic / Google / autres via `pi-ai` | Tout modèle compatible image répertorié dans le registre fonctionne. |
| Audio | OpenAI, Groq, Deepgram, Google | Transcription côté fournisseur (Whisper/Deepgram/Gemini). |
| Vidéo | Google (API Gemini) | Compréhension vidéo côté fournisseur. |

<div id="recommended-providers">
  ## Fournisseurs recommandés
</div>

**Image**

* Privilégiez votre modèle actif s’il prend en charge les images.
* Bons choix par défaut : `openai/gpt-5.2`, `anthropic/claude-opus-4-5`, `google/gemini-3-pro-preview`.

**Audio**

* `openai/gpt-4o-mini-transcribe`, `groq/whisper-large-v3-turbo` ou `deepgram/nova-3`.
* Solution de repli en CLI : `whisper-cli` (whisper-cpp) ou `whisper`.
* Configuration Deepgram : [Deepgram (transcription audio)](/fr/providers/deepgram).

**Vidéo**

* `google/gemini-3-flash-preview` (rapide), `google/gemini-3-pro-preview` (plus riche).
* Solution de repli en CLI : CLI `gemini` (prend en charge `read_file` sur la vidéo/l’audio).

<div id="attachment-policy">
  ## Politique de gestion des pièces jointes
</div>

Le paramètre `attachments` par capacité contrôle quelles pièces jointes sont traitées :

* `mode` : `first` (valeur par défaut) ou `all`
* `maxAttachments` : limite le nombre traité (par défaut **1**)
* `prefer` : `first`, `last`, `path`, `url`

Lorsque `mode: "all"`, les sorties sont étiquetées `[Image 1/2]`, `[Audio 2/2]`, etc.

<div id="config-examples">
  ## Exemples de configuration
</div>

<div id="1-shared-models-list-overrides">
  ### 1) Liste partagée de modèles + surcharges
</div>

```json5
{
  tools: {
    media: {
      models: [
        { provider: "openai", model: "gpt-5.2", capabilities: ["image"] },
        { provider: "google", model: "gemini-3-flash-preview", capabilities: ["image", "audio", "video"] },
        {
          type: "cli",
          command: "gemini",
          args: [
            "-m",
            "gemini-3-flash",
            "--allowed-tools",
            "read_file",
            "Lisez le média à {{MediaPath}} et décrivez-le en <= {{MaxChars}} caractères."
          ],
          capabilities: ["image", "video"]
        }
      ],
      audio: {
        attachments: { mode: "all", maxAttachments: 2 }
      },
      video: {
        maxChars: 500
      }
    }
  }
}
```

<div id="2-audio-video-only-image-off">
  ### 2) Audio + vidéo uniquement (sans image)
</div>

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"]
          }
        ]
      },
      video: {
        enabled: true,
        maxChars: 500,
        models: [
          { provider: "google", model: "gemini-3-flash-preview" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters."
            ]
          }
        ]
      }
    }
  }
}
```

<div id="3-optional-image-understanding">
  ### 3) Compréhension d’images optionnelle
</div>

```json5
{
  tools: {
    media: {
      image: {
        enabled: true,
        maxBytes: 10485760,
        maxChars: 500,
        models: [
          { provider: "openai", model: "gpt-5.2" },
          { provider: "anthropic", model: "claude-opus-4-5" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Lisez le média à {{MediaPath}} et décrivez-le en <= {{MaxChars}} caractères."
            ]
          }
        ]
      }
    }
  }
}
```

<div id="4-multimodal-single-entry-explicit-capabilities">
  ### 4) Point d’entrée unique multimodal (capacités explicites)
</div>

```json5
{
  tools: {
    media: {
      image: { models: [{ provider: "google", model: "gemini-3-pro-preview", capabilities: ["image", "video", "audio"] }] },
      audio: { models: [{ provider: "google", model: "gemini-3-pro-preview", capabilities: ["image", "video", "audio"] }] },
      video: { models: [{ provider: "google", model: "gemini-3-pro-preview", capabilities: ["image", "video", "audio"] }] }
    }
  }
}
```

<div id="status-output">
  ## Sortie de statut
</div>

Lorsque l&#39;analyse des médias est en cours d&#39;exécution, `/status` inclut une courte ligne récapitulative :

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

Cela affiche les résultats par capacité, ainsi que le fournisseur/modèle choisi le cas échéant.

<div id="notes">
  ## Notes
</div>

* La compréhension est réalisée **au mieux**. Les erreurs n&#39;empêchent pas l&#39;envoi de réponses.
* Les pièces jointes sont toujours envoyées aux modèles, même lorsque la compréhension est désactivée.
* Utilisez `scope` pour limiter la portée de la compréhension (par ex. uniquement dans les messages privés/DM).

<div id="related-docs">
  ## Documents associés
</div>

* [Configuration](/fr/gateway/configuration)
* [Prise en charge des images et des médias](/fr/nodes/images)