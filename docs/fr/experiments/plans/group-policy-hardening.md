---
title: Durcissement de la stratégie de groupe
summary: "Durcissement de la liste d’autorisation Telegram : préfixe + normalisation des espaces"
read_when:
  - Consultation des modifications historiques de la liste d’autorisation Telegram
---

<div id="telegram-allowlist-hardening">
  # Renforcement de la liste d’autorisation Telegram
</div>

**Date**: 2026-01-05\
**Statut**: Terminé\
**PR**: #216

<div id="summary">
  ## Résumé
</div>

Les listes d’autorisation Telegram acceptent désormais les préfixes `telegram:` et `tg:` indépendamment de la casse et ignorent les espaces accidentels. Cela aligne les contrôles de liste d’autorisation sur les messages entrants avec la normalisation des envois sortants.

<div id="what-changed">
  ## Ce qui a changé
</div>

* Les préfixes `telegram:` et `tg:` sont traités de la même manière (insensibles à la casse).
* Les entrées de la liste d’autorisation sont nettoyées (espaces en début et fin supprimés) ; les entrées vides sont ignorées.

<div id="examples">
  ## Exemples
</div>

Tous ces formats sont valides pour le même identifiant :

* `telegram:123456`
* `TG:123456`
* `tg:123456`

<div id="why-it-matters">
  ## Pourquoi c&#39;est important
</div>

Le copier-coller depuis les journaux ou les identifiants de discussion inclut souvent des préfixes et des espaces. La normalisation évite les faux négatifs lorsque vous décidez s&#39;il faut répondre en message privé (DM) ou dans un groupe.

<div id="related-docs">
  ## Documentation associée
</div>

* [Discussions de groupe](/fr/concepts/groups)
* [Fournisseur Telegram](/fr/channels/telegram)