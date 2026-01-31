---
title: Dépannage
summary: "Raccourcis de dépannage par canal (Discord/Telegram/WhatsApp)"
read_when:
  - Un canal est connecté mais les messages ne transitent pas
  - Diagnostic d’une mauvaise configuration du canal (intents, permissions, mode de confidentialité)
---

<div id="channel-troubleshooting">
  # Dépannage des canaux
</div>

Commencez par :

```bash
openclaw doctor
openclaw channels status --probe
```

`channels status --probe` affiche des avertissements lorsqu’il détecte des erreurs de configuration courantes de canaux, et effectue de petits contrôles en temps réel (identifiants, certaines autorisations/appartenances).

<div id="channels">
  ## Canaux
</div>

* Discord : [/channels/discord#troubleshooting](/fr/channels/discord#troubleshooting)
* Telegram : [/channels/telegram#troubleshooting](/fr/channels/telegram#troubleshooting)
* WhatsApp : [/channels/whatsapp#troubleshooting-quick](/fr/channels/whatsapp#troubleshooting-quick)

<div id="telegram-quick-fixes">
  ## Correctifs rapides pour Telegram
</div>

* Les logs affichent `HttpError: Network request for 'sendMessage' failed` ou `sendChatAction` → vérifiez le DNS IPv6. Si `api.telegram.org` se résout en priorité en IPv6 et que l&#39;hôte n&#39;a pas de sortie IPv6, forcez l&#39;IPv4 ou activez l&#39;IPv6. Consultez [/channels/telegram#troubleshooting](/fr/channels/telegram#troubleshooting).
* Les logs affichent `setMyCommands failed` → vérifiez la connectivité sortante HTTPS et DNS vers `api.telegram.org` (problème fréquent sur des VPS verrouillés ou derrière des proxies).