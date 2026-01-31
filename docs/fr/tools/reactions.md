---
title: Réactions
summary: "Sémantique des réactions commune à tous les canaux"
read_when:
  - Lorsque vous travaillez sur les réactions dans n'importe quel canal
---

<div id="reaction-tooling">
  # Outils de gestion des réactions
</div>

Sémantique commune des réactions entre les canaux :

- `emoji` est requis lors de l'ajout d'une réaction.
- `emoji=""` supprime la/les réaction(s) du bot lorsque cette opération est prise en charge.
- `remove: true` supprime l'emoji spécifié lorsque cette opération est prise en charge (nécessite `emoji`).

Notes par canal :

- **Discord/Slack** : un `emoji` vide supprime toutes les réactions du bot sur le message ; `remove: true` supprime uniquement cet emoji.
- **Google Chat** : un `emoji` vide supprime les réactions de l'application sur le message ; `remove: true` supprime uniquement cet emoji.
- **Telegram** : un `emoji` vide supprime les réactions du bot ; `remove: true` supprime aussi les réactions mais nécessite toujours un `emoji` non vide pour la validation de l'outil.
- **WhatsApp** : un `emoji` vide supprime la réaction du bot ; `remove: true` correspond à un emoji vide (nécessite toujours `emoji`).
- **Signal** : les notifications de réactions entrantes émettent des événements système lorsque `channels.signal.reactionNotifications` est activé.