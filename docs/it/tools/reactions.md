---
title: Reazioni
summary: "Semantica delle reazioni condivisa tra tutti i canali"
read_when:
  - Quando lavori sulle reazioni in qualsiasi canale
---

<div id="reaction-tooling">
  # Strumenti per le reazioni
</div>

Semantica condivisa delle reazioni tra i canali:

- `emoji` è obbligatorio quando aggiungi una reazione.
- `emoji=""` rimuove le reazioni del bot, se supportato.
- `remove: true` rimuove l'emoji specificata, se supportato (richiede `emoji`).

Note sui canali:

- **Discord/Slack**: `emoji` vuoto rimuove tutte le reazioni del bot sul messaggio; `remove: true` rimuove solo quella emoji.
- **Google Chat**: `emoji` vuoto rimuove le reazioni dell'app sul messaggio; `remove: true` rimuove solo quella emoji.
- **Telegram**: `emoji` vuoto rimuove le reazioni del bot; `remove: true` rimuove comunque le reazioni ma richiede ancora un `emoji` non vuoto per la validazione del tool.
- **WhatsApp**: `emoji` vuoto rimuove la reazione del bot; `remove: true` viene mappato su un'emoji vuota (richiede comunque `emoji`).
- **Signal**: le notifiche di reazione in arrivo emettono eventi di sistema quando `channels.signal.reactionNotifications` è abilitato.