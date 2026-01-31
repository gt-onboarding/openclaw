---
title: Reaktionen
summary: "Kanalübergreifende Reaktionssemantik"
read_when:
  - Beim Arbeiten mit Reaktionen in einem beliebigen Kanal
---

<div id="reaction-tooling">
  # Reaktions-Tools
</div>

Kanalübergreifende Reaktionssemantik:

- `emoji` ist beim Hinzufügen einer Reaktion erforderlich.
- `emoji=""` entfernt die Reaktion(en) des Bots, sofern unterstützt.
- `remove: true` entfernt das angegebene Emoji, sofern unterstützt (erfordert `emoji`).

Hinweise zu Kanälen:

- **Discord/Slack**: Ein leeres `emoji` entfernt alle Reaktionen des Bots auf die Nachricht; `remove: true` entfernt nur dieses eine Emoji.
- **Google Chat**: Ein leeres `emoji` entfernt die Reaktionen der App auf die Nachricht; `remove: true` entfernt nur dieses eine Emoji.
- **Telegram**: Ein leeres `emoji` entfernt die Reaktionen des Bots; `remove: true` entfernt ebenfalls Reaktionen, erfordert aber weiterhin ein nicht-leeres `emoji` für die Tool-Validierung.
- **WhatsApp**: Ein leeres `emoji` entfernt die Bot-Reaktion; `remove: true` wird als leeres Emoji interpretiert (erfordert weiterhin `emoji`).
- **Signal**: Eingehende Reaktionsbenachrichtigungen erzeugen Systemereignisse, wenn `channels.signal.reactionNotifications` aktiviert ist.