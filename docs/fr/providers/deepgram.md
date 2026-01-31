---
title: Deepgram
summary: "Transcription avec Deepgram pour les notes vocales entrantes"
read_when:
  - Vous souhaitez utiliser la transcription automatique Deepgram pour les pièces jointes audio
  - Vous avez besoin d’un exemple rapide de configuration de Deepgram
---

<div id="deepgram-audio-transcription">
  # Deepgram (Transcription audio)
</div>

Deepgram est une API de reconnaissance vocale (speech‑to‑text). Dans OpenClaw, elle est utilisée pour la **transcription audio / notes vocales entrantes**
via `tools.media.audio`.

Lorsqu’elle est activée, OpenClaw transfère le fichier audio à Deepgram et injecte la transcription
dans le pipeline de réponse (`{{Transcript}}` + bloc `[Audio]`). Ce **n’est pas du streaming** ;
cela utilise l’endpoint de transcription pour de l’audio pré‑enregistré.

Site web : https://deepgram.com  
Documentation : https://developers.deepgram.com

<div id="quick-start">
  ## Démarrage rapide
</div>

1. Configurez votre clé API :

```
DEEPGRAM_API_KEY=dg_...
```

2. Activez le fournisseur :

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3" }]
      }
    }
  }
}
```


<div id="options">
  ## Options
</div>

* `model` : ID du modèle Deepgram (par défaut : `nova-3`)
* `language` : paramètre de langue (facultatif)
* `tools.media.audio.providerOptions.deepgram.detect_language` : activer la détection de langue (facultatif)
* `tools.media.audio.providerOptions.deepgram.punctuate` : activer la ponctuation (facultatif)
* `tools.media.audio.providerOptions.deepgram.smart_format` : activer la mise en forme intelligente (facultatif)

Exemple avec paramètre de langue :

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "deepgram", model: "nova-3", language: "en" }
        ]
      }
    }
  }
}
```

Exemple avec des options pour Deepgram :

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        providerOptions: {
          deepgram: {
            detect_language: true,
            punctuate: true,
            smart_format: true
          }
        },
        models: [{ provider: "deepgram", model: "nova-3" }]
      }
    }
  }
}
```


<div id="notes">
  ## Notes
</div>

- L’authentification suit l’ordre standard d’authentification des fournisseurs ; `DEEPGRAM_API_KEY` est l’option la plus simple.
- Remplacez les points de terminaison ou les en-têtes par `tools.media.audio.baseUrl` et `tools.media.audio.headers` lors de l’utilisation d’un proxy.
- La sortie suit les mêmes règles audio que pour les autres fournisseurs (limites de taille, délais d’expiration, injection de transcription).