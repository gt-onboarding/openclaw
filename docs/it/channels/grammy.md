---
title: Grammy
summary: "Integrazione con Telegram Bot API tramite grammY con note di configurazione"
read_when:
  - Quando lavori su integrazioni Telegram o grammY
---

<div id="grammy-integration-telegram-bot-api">
  # Integrazione con grammY (Telegram Bot API)
</div>

<div id="why-grammy">
  # Perché grammY
</div>

- Client Bot API TypeScript-first con helper integrati per long polling + webhook, middleware, gestione degli errori, rate limiter.
- Helper per i media più puliti rispetto a implementare a mano fetch + FormData; supporta tutti i metodi della Bot API.
- Estensibile: supporto proxy tramite fetch personalizzato, middleware di sessione (opzionale), contesto type-safe.

<div id="what-we-shipped">
  # Cosa abbiamo rilasciato
</div>

- **Unico client:** implementazione basata su fetch rimossa; grammY è ora l'unico client Telegram (send + Gateway) con il throttler di grammY abilitato per impostazione predefinita.
- **Gateway:** `monitorTelegramProvider` crea un `Bot` di grammY, collega il gating basato su mention/lista di autorizzati, il download dei media tramite `getFile`/`download` e consegna le risposte con `sendMessage/sendPhoto/sendVideo/sendAudio/sendDocument`. Supporta long polling o webhook tramite `webhookCallback`.
- **Proxy:** `channels.telegram.proxy` opzionale usa `undici.ProxyAgent` tramite il `client.baseFetch` di grammY.
- **Supporto webhook:** `webhook-set.ts` incapsula `setWebhook/deleteWebhook`; `webhook.ts` espone il callback con health check + arresto graduale. Il Gateway abilita la modalità webhook quando `channels.telegram.webhookUrl` è impostato (altrimenti usa il long polling).
- **Sessioni:** le chat dirette confluiscono nella sessione principale dell’agente (`agent:<agentId>:<mainKey>`); i gruppi usano `agent:<agentId>:telegram:group:<chatId>`; le risposte vengono instradate allo stesso canale.
- **Opzioni di configurazione:** `channels.telegram.botToken`, `channels.telegram.dmPolicy`, `channels.telegram.groups` (lista di autorizzati + valori predefiniti per le mention), `channels.telegram.allowFrom`, `channels.telegram.groupAllowFrom`, `channels.telegram.groupPolicy`, `channels.telegram.mediaMaxMb`, `channels.telegram.linkPreview`, `channels.telegram.proxy`, `channels.telegram.webhookSecret`, `channels.telegram.webhookUrl`.
- **Draft streaming:** `channels.telegram.streamMode` opzionale usa `sendMessageDraft` nelle chat private con topic (Bot API 9.3+). Questo è separato dallo streaming a blocchi sui canali.
- **Test:** i mock di grammY coprono il gating per DM/messaggi diretti + mention nei gruppi e l’invio in uscita; sono ancora ben accetti ulteriori fixture per media/webhook.

Questioni aperte

- Plugin opzionali di grammY (throttler) se incontriamo errori Bot API 429.
- Aggiungere test media più strutturati (sticker, messaggi vocali).
- Rendere configurabile la porta di ascolto del webhook (attualmente fissata a 8787 a meno che non sia collegata tramite il Gateway).