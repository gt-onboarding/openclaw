---
title: Hygiène des transcriptions
summary: "Référence : règles de nettoyage et de réparation des transcriptions spécifiques au fournisseur"
read_when:
  - Vous déboguez des rejets de requêtes de fournisseurs liés à la structure du transcript
  - Vous modifiez la logique de nettoyage des transcripts ou de réparation des appels d’outils
  - Vous enquêtez sur des incohérences d’identifiants d’appels d’outils entre fournisseurs
---

<div id="transcript-hygiene-provider-fixups">
  # Hygiène des transcriptions (correctifs fournisseur)
</div>

Ce document décrit les **correctifs spécifiques au fournisseur** appliqués aux transcriptions avant un run
(construction du contexte du modèle). Il s’agit d’ajustements **en mémoire** utilisés pour satisfaire les exigences
strictes du fournisseur concerné. Ils ne réécrivent **pas** le journal JSONL stocké sur disque.

Le périmètre couvre :

* Nettoyage des identifiants d’appels d’outils
* Réparation de l’appairage des résultats d’outils
* Validation / ordonnancement des tours de parole
* Nettoyage des signatures de réflexion
* Nettoyage des charges utiles d’images

Si vous avez besoin de détails sur le stockage des transcriptions, voir :

* [/reference/session-management-compaction](/fr/reference/session-management-compaction)

***

<div id="where-this-runs">
  ## Où cela s’exécute
</div>

Toute l’hygiène des transcriptions est centralisée dans le runner embarqué :

* Sélection de la politique : `src/agents/transcript-policy.ts`
* Application du nettoyage/réparation : `sanitizeSessionHistory` dans `src/agents/pi-embedded-runner/google.ts`

Cette politique utilise `provider`, `modelApi` et `modelId` pour déterminer ce qu’il faut appliquer.

***

<div id="global-rule-image-sanitization">
  ## Règle globale : nettoyage des images
</div>

Les payloads d’images sont toujours nettoyés pour éviter un rejet côté fournisseur en raison des limites de taille (réduction et recompression des images base64 surdimensionnées).

Implémentation :

* `sanitizeSessionMessagesImages` dans `src/agents/pi-embedded-helpers/images.ts`
* `sanitizeContentBlocksImages` dans `src/agents/tool-images.ts`

***

<div id="provider-matrix-current-behavior">
  ## Matrice des fournisseurs (comportement actuel)
</div>

**OpenAI / OpenAI Codex**

* Assainissement d’images uniquement.
* Lors d’un changement de modèle vers OpenAI Responses/Codex, supprimer les signatures de raisonnement orphelines (éléments de raisonnement isolés sans bloc de contenu suivant).
* Aucun assainissement des identifiants d’appels d’outils.
* Aucune réparation de l’association des résultats d’outils.
* Aucune validation ni réordonnancement des tours.
* Aucun résultat d’outil synthétique.
* Aucun retrait des signatures de raisonnement.

**Google (Generative AI / Gemini CLI / Antigravity)**

* Assainissement des identifiants d’appels d’outils : alphanumérique strict.
* Réparation de l’association des résultats d’outils et résultats d’outils synthétiques.
* Validation des tours (alternance de tours à la Gemini).
* Correction de l’ordre des tours Google (préfixer avec un minuscule message d’amorçage utilisateur si l’historique commence par l’assistant).
* Antigravity Claude : normaliser les signatures de raisonnement ; supprimer les blocs de raisonnement non signés.

**Anthropic / Minimax (compatible Anthropic)**

* Réparation de l’association des résultats d’outils et résultats d’outils synthétiques.
* Validation des tours (fusionner les tours utilisateur consécutifs pour respecter l’alternance stricte).

**Mistral (y compris détection basée sur `model-id`)**

* Assainissement des identifiants d’appels d’outils : strict9 (alphanumérique de longueur 9).

**OpenRouter Gemini**

* Nettoyage des signatures de raisonnement : retirer les valeurs de `thought_signature` non base64 (conserver le base64).

**Tout le reste**

* Assainissement d’images uniquement.

***

<div id="historical-behavior-pre-2026122">
  ## Comportement historique (avant 2026.1.22)
</div>

Avant la version 2026.1.22, OpenClaw appliquait plusieurs couches d&#39;hygiène des transcriptions :

* Une **extension transcript-sanitize** s&#39;exécutait à chaque construction de contexte et pouvait :
  * Réparer l&#39;appairage entre utilisation d&#39;outils et résultats.
  * Assainir les IDs d&#39;appel d&#39;outils (`tool call ids`), avec un mode non strict qui préservait `_`/`-`.
* Le runner effectuait également un nettoyage spécifique au fournisseur, ce qui dupliquait le travail.
* Des mutations supplémentaires se produisaient en dehors de la politique du fournisseur, notamment :
  * La suppression des balises `<final>` du texte de l&#39;assistant avant la persistance.
  * La suppression des tours d&#39;erreur vides de l&#39;assistant.
  * La troncature du contenu de l&#39;assistant après des appels d&#39;outils.

Cette complexité a provoqué des régressions inter-fournisseurs (notamment l&#39;appairage `call_id|fc_id` pour `openai-responses`). Le nettoyage de 2026.1.22 a supprimé l&#39;extension, centralisé la logique dans le runner et fait qu&#39;OpenAI est désormais **no-touch** en dehors du nettoyage des images.