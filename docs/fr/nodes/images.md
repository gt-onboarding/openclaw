---
title: Images
summary: "Règles de gestion des images et des médias pour send, Gateway et les réponses d’agent"
read_when:
  - Modification du pipeline média ou des pièces jointes
---

<div id="image-media-support-2025-12-05">
  # Prise en charge des images et des médias — 2025-12-05
</div>

Le canal WhatsApp fonctionne via **Baileys Web**. Ce document décrit les règles actuelles de gestion des médias pour l'action Envoyer, Gateway et les réponses d’agent.

<div id="goals">
  ## Objectifs
</div>

- Envoyer des contenus média avec éventuellement une légende via `openclaw message send --media`.
- Permettre aux réponses automatiques depuis la boîte de réception web d'inclure des contenus média en plus du texte.
- Maintenir des limites par type raisonnables et prévisibles.

<div id="cli-surface">
  ## Interface CLI
</div>

- `openclaw message send --media <path-or-url> [--message <caption>]`
  - `--media` optionnel ; la légende peut être vide pour des envois contenant uniquement des médias.
  - `--dry-run` affiche la charge utile résolue ; `--json` émet `{ channel, to, messageId, mediaUrl, caption }`.

<div id="whatsapp-web-channel-behavior">
  ## Comportement du canal WhatsApp Web
</div>

- Entrée : chemin de fichier local **ou** URL HTTP(S).
- Traitement : chargement dans un `Buffer`, détection du type de média, puis construction de la charge utile appropriée :
  - **Images :** redimensionnement et recompression en JPEG (côté max. 2048 px) en visant `agents.defaults.mediaMaxMb` (5 MB par défaut), avec un plafond à 6 MB.
  - **Audio/Voix/Vidéo :** passage direct jusqu’à 16 MB ; l’audio est envoyé comme note vocale (`ptt: true`).
  - **Documents :** tout le reste, jusqu’à 100 MB, avec le nom de fichier préservé lorsque disponible.
- Lecture façon GIF WhatsApp : envoyer un MP4 avec `gifPlayback: true` (CLI : `--gif-playback`) pour que les clients mobiles le lisent en boucle en ligne.
- La détection MIME privilégie les magic bytes, puis les en-têtes, puis l’extension de fichier.
- La légende provient de `--message` ou `reply.text` ; une légende vide est autorisée.
- Journalisation : en mode non verbeux, affiche `↩️`/`✅` ; en mode verbeux, inclut la taille et le chemin ou l’URL source.

<div id="auto-reply-pipeline">
  ## Pipeline de réponse automatique
</div>

- `getReplyFromConfig` renvoie `{ text?, mediaUrl?, mediaUrls? }`.
- Lorsque des médias sont présents, l’expéditeur web résout les chemins locaux ou les URL en utilisant le même pipeline que `openclaw message send`.
- Plusieurs médias sont envoyés séquentiellement s’ils sont fournis.

<div id="inbound-media-to-commands-pi">
  ## Médias entrants vers les commandes (Pi)
</div>

- Lorsque les messages web entrants incluent des médias, OpenClaw les télécharge dans un fichier temporaire et expose des variables de templating :
  - `{{MediaUrl}}` pseudo-URL pour le média entrant.
  - `{{MediaPath}}` chemin temporaire local écrit avant l’exécution de la commande.
- Lorsqu’un sandbox Docker par session est activé, les médias entrants sont copiés dans l’espace de travail du sandbox et `MediaPath`/`MediaUrl` sont réécrits en un chemin relatif de type `media/inbound/<filename>`.
- L’interprétation des médias (si configurée via `tools.media.*` ou le `tools.media.models` partagé) s’exécute avant le templating et peut insérer des blocs `[Image]`, `[Audio]` et `[Video]` dans `Body`.
  - L’audio définit `{{Transcript}}` et utilise la transcription pour l’analyse de commande, de sorte que les commandes slash continuent de fonctionner.
  - Les descriptions vidéo et image conservent tout texte de légende pour l’analyse de commande.
- Par défaut, seule la première pièce jointe image/audio/vidéo correspondante est traitée ; définissez `tools.media.<cap>.attachments` pour traiter plusieurs pièces jointes.

<div id="limits-errors">
  ## Limites et erreurs
</div>

**Limites d’envoi sortant (envoi via WhatsApp Web)**

- Images : limite d’environ 6 Mo après recompression.
- Audio/voix/vidéo : limite de 16 Mo ; documents : limite de 100 Mo.
- Médias trop volumineux ou illisibles → erreur explicite dans les journaux et la réponse est ignorée.

**Limites d’analyse des médias (transcription/description)**

- Image par défaut : 10 Mo (`tools.media.image.maxBytes`).
- Audio par défaut : 20 Mo (`tools.media.audio.maxBytes`).
- Vidéo par défaut : 50 Mo (`tools.media.video.maxBytes`).
- Les médias trop volumineux ne sont pas analysés, mais les réponses sont tout de même envoyées avec le contenu original.

<div id="notes-for-tests">
  ## Notes pour les tests
</div>

- Couvrir les flux d’envoi et de réponse pour les cas image/audio/document.
- Valider la recompression des images (borne de taille) et l’indicateur de note vocale pour l’audio.
- S’assurer que les réponses multimédias sont émises sous forme d’envois séquentiels.