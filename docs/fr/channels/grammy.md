---
title: Grammy
summary: "Intégration Telegram Bot API via grammY avec notes de configuration"
read_when:
  - Lorsque vous travaillez sur des intégrations Telegram ou grammY
---

<div id="grammy-integration-telegram-bot-api">
  # Intégration de grammY (Telegram Bot API)
</div>

<div id="why-grammy">
  # Pourquoi grammY
</div>

- Client Bot API priorisant TypeScript, avec prise en charge intégrée du long-polling + des webhooks, middleware, gestion des erreurs, limiteur de débit.
- Utilitaires média plus propres qu’un `fetch` + `FormData` faits maison ; prend en charge toutes les méthodes de la Bot API.
- Extensible : prise en charge des proxies via un `fetch` personnalisé, middleware de session (optionnel), contexte à typage sûr.

<div id="what-we-shipped">
  # Ce que nous avons livré
</div>

- **Chemin client unique :** implémentation basée sur `fetch` supprimée ; grammY est désormais le seul client Telegram (envoi + Gateway) avec le throttler grammY activé par défaut.
- **Gateway :** `monitorTelegramProvider` crée un `Bot` grammY, câble la mise sous condition par mention/liste d’autorisation, le téléchargement des médias via `getFile`/`download`, et achemine les réponses avec `sendMessage/sendPhoto/sendVideo/sendAudio/sendDocument`. Prend en charge le long polling ou le webhook via `webhookCallback`.
- **Proxy :** le `channels.telegram.proxy` optionnel utilise `undici.ProxyAgent` via le `client.baseFetch` de grammY.
- **Prise en charge des webhooks :** `webhook-set.ts` encapsule `setWebhook/deleteWebhook` ; `webhook.ts` héberge le callback avec vérification d’état (health) + arrêt en douceur. Gateway active le mode webhook lorsque `channels.telegram.webhookUrl` est défini (sinon il utilise le long polling).
- **Sessions :** les discussions directes sont regroupées dans la session principale de l’agent (`agent:<agentId>:<mainKey>`), les groupes utilisent `agent:<agentId>:telegram:group:<chatId>` ; les réponses sont renvoyées vers le même canal.
- **Réglages de configuration :** `channels.telegram.botToken`, `channels.telegram.dmPolicy`, `channels.telegram.groups` (liste d’autorisation + valeurs par défaut pour les mentions), `channels.telegram.allowFrom`, `channels.telegram.groupAllowFrom`, `channels.telegram.groupPolicy`, `channels.telegram.mediaMaxMb`, `channels.telegram.linkPreview`, `channels.telegram.proxy`, `channels.telegram.webhookSecret`, `channels.telegram.webhookUrl`.
- **Diffusion en mode brouillon :** le `channels.telegram.streamMode` optionnel utilise `sendMessageDraft` dans les discussions de topics privés (Bot API 9.3+). Ceci est séparé de la diffusion en continu par blocs de canal.
- **Tests :** les mocks grammY couvrent la mise sous condition par DM + mention en groupe et l’envoi sortant ; davantage de fixtures médias/webhook sont toujours bienvenues.

Questions en suspens

- Plugins grammY optionnels (throttler) si l’on rencontre des erreurs Bot API 429.
- Ajouter des tests de médias plus structurés (stickers, messages vocaux).
- Rendre le port d’écoute du webhook configurable (actuellement fixé à 8787, sauf s’il est relayé via le Gateway).