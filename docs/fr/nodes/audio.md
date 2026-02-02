---
title: Audio
summary: "Comment les notes audio ou vocales entrantes sont téléchargées, transcrites et intégrées aux réponses"
read_when:
  - Modifier la transcription audio ou la gestion des médias
---

<div id="audio-voice-notes-2026-01-17">
  # Audio / notes vocales — 2026-01-17
</div>

<div id="what-works">
  ## Ce qui fonctionne
</div>

* **Compréhension des médias (audio)** : Si la compréhension audio est activée (ou détectée automatiquement), OpenClaw :
  1. Repère la première pièce jointe audio (chemin local ou URL) et la télécharge si nécessaire.
  2. Applique `maxBytes` avant l’envoi à chaque entrée de modèle.
  3. Exécute la première entrée de modèle éligible dans l’ordre (fournisseur ou CLI).
  4. En cas d’échec ou si l’entrée est ignorée (taille/délai), essaie l’entrée suivante.
  5. En cas de succès, remplace `Body` par un bloc `[Audio]` et renseigne `{{Transcript}}`.
* **Analyse des commandes** : Quand la transcription réussit, `CommandBody`/`RawBody` sont définis sur la transcription pour que les commandes slash continuent de fonctionner.
* **Journalisation détaillée** : Avec `--verbose`, nous consignons dans les journaux le moment où la transcription s’exécute et celui où elle remplace le corps du message.

<div id="auto-detection-default">
  ## Détection automatique (par défaut)
</div>

Si vous **ne configurez pas de modèles** et que `tools.media.audio.enabled` n’est **pas** défini à `false`,
OpenClaw effectue une détection automatique dans cet ordre et s’arrête à la première option fonctionnelle :

1. **CLI locaux** (s’ils sont installés)
   * `sherpa-onnx-offline` (nécessite `SHERPA_ONNX_MODEL_DIR` avec encodeur/décodeur/joiner/tokens)
   * `whisper-cli` (fourni par `whisper-cpp` ; utilise `WHISPER_CPP_MODEL` ou le modèle tiny embarqué)
   * `whisper` (CLI Python ; télécharge les modèles automatiquement)
2. **Gemini CLI** (`gemini`) utilisant `read_many_files`
3. **Clés de fournisseurs** (OpenAI → Groq → Deepgram → Google)

Pour désactiver la détection automatique, définissez `tools.media.audio.enabled: false`.
Pour la personnaliser, définissez `tools.media.audio.models`.
Remarque : la détection des binaires est effectuée au mieux (« best-effort ») sur macOS/Linux/Windows ; assurez-vous que la CLI est dans le `PATH` (nous développons le `~`), ou définissez un modèle CLI explicite avec un chemin de commande complet.

<div id="config-examples">
  ## Exemples de configuration
</div>

<div id="provider-cli-fallback-openai-whisper-cli">
  ### Fournisseur + repli via la CLI (OpenAI + Whisper CLI)
</div>

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
            timeoutSeconds: 45
          }
        ]
      }
    }
  }
}
```

<div id="provider-only-with-scope-gating">
  ### Fournisseur uniquement avec contrôle par portée
</div>

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        scope: {
          default: "allow",
          rules: [
            { action: "deny", match: { chatType: "group" } }
          ]
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" }
        ]
      }
    }
  }
}
```

<div id="provider-only-deepgram">
  ### Uniquement avec le fournisseur Deepgram
</div>

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

<div id="notes-limits">
  ## Notes et limites
</div>

* L’authentification des fournisseurs suit l’ordre standard d’authentification des modèles (profils d’authentification, variables d’environnement, `models.providers.*.apiKey`).
* Deepgram utilise `DEEPGRAM_API_KEY` lorsque `provider: "deepgram"` est spécifié.
* Détails de configuration Deepgram : [Deepgram (audio transcription)](/fr/providers/deepgram).
* Les fournisseurs audio peuvent remplacer `baseUrl`, `headers` et `providerOptions` via `tools.media.audio`.
* La limite de taille par défaut est de 20 Mo (`tools.media.audio.maxBytes`). L’audio trop volumineux est ignoré pour ce modèle et l’entrée suivante est tentée.
* La valeur par défaut de `maxChars` pour l’audio est **non définie** (transcription complète). Configurez `tools.media.audio.maxChars` ou un `maxChars` par entrée pour tronquer la sortie.
* Le modèle par défaut automatique pour OpenAI est `gpt-4o-mini-transcribe` ; configurez `model: "gpt-4o-transcribe"` pour une précision supérieure.
* Utilisez `tools.media.audio.attachments` pour traiter plusieurs notes vocales (`mode: "all"` + `maxAttachments`).
* La transcription est disponible dans les modèles sous `{{Transcript}}`.
* Le stdout de la CLI est plafonné (5 Mo) ; gardez la sortie de la CLI concise.

<div id="gotchas">
  ## Points d’attention
</div>

* Les règles de portée suivent le principe « première correspondance, première servie ». `chatType` est normalisé en `direct`, `group` ou `room`.
* Assurez-vous que votre CLI se termine avec le code de sortie 0 et affiche du texte brut ; le JSON doit être post-traité via `jq -r .text`.
* Gardez des délais d&#39;expiration raisonnables (`timeoutSeconds`, valeur par défaut 60s) pour éviter de bloquer la file d&#39;attente des réponses.