---
title: Grammy
summary: "Telegram Bot API-Integration über grammY mit Einrichtungshinweisen"
read_when:
  - Bei Arbeiten an Telegram- oder grammY-Integrationen
---

<div id="grammy-integration-telegram-bot-api">
  # grammY-Integration (Telegram-Bot-API)
</div>

<div id="why-grammy">
  # Warum grammY
</div>

- TS-first-Bot-API-Client mit eingebauten Helpern für Long Polling und Webhooks, Middleware, Fehlerbehandlung und Rate Limiter.
- Sauberere Medien-Helper als selbst geschriebenes fetch + FormData; unterstützt alle Bot-API-Methoden.
- Erweiterbar: Proxy-Unterstützung über benutzerdefiniertes fetch, Sitzungs-Middleware (optional) und typsicherer Kontext.

<div id="what-we-shipped">
  # Was wir ausgeliefert haben
</div>

- **Einzelner Client-Pfad:** fetch-basierte Implementierung entfernt; grammY ist jetzt der einzige Telegram-Client (Senden + Gateway) mit standardmäßig aktiviertem grammY-Throttler.
- **Gateway:** `monitorTelegramProvider` baut einen grammY-`Bot`, verdrahtet Mention-/Allowlist-Gating, Mediendownload über `getFile`/`download` und liefert Antworten mit `sendMessage/sendPhoto/sendVideo/sendAudio/sendDocument` aus. Unterstützt Long-Polling oder Webhook über `webhookCallback`.
- **Proxy:** optionales `channels.telegram.proxy` verwendet `undici.ProxyAgent` über grammYs `client.baseFetch`.
- **Webhook-Unterstützung:** `webhook-set.ts` kapselt `setWebhook/deleteWebhook`; `webhook.ts` hostet den Callback mit Health-Check + sauberem Shutdown. Das Gateway aktiviert den Webhook-Modus, wenn `channels.telegram.webhookUrl` gesetzt ist (ansonsten wird Long-Polling verwendet).
- **Sitzungen:** Direktchats werden in die Hauptsitzung des Agents zusammengeführt (`agent:<agentId>:<mainKey>`); Gruppen verwenden `agent:<agentId>:telegram:group:<chatId>`; Antworten werden zurück in denselben Kanal geroutet.
- **Konfig-Schalter:** `channels.telegram.botToken`, `channels.telegram.dmPolicy`, `channels.telegram.groups` (Allowlist + Mention-Standardwerte), `channels.telegram.allowFrom`, `channels.telegram.groupAllowFrom`, `channels.telegram.groupPolicy`, `channels.telegram.mediaMaxMb`, `channels.telegram.linkPreview`, `channels.telegram.proxy`, `channels.telegram.webhookSecret`, `channels.telegram.webhookUrl`.
- **Draft-Streaming:** optionales `channels.telegram.streamMode` verwendet `sendMessageDraft` in privaten Themen-Chats (Bot API 9.3+). Das ist getrennt vom Channel-Block-Streaming.
- **Tests:** grammY-Mocks decken DM- + Gruppen-Mention-Gating und ausgehendes Senden ab; weitere Medien-/Webhook-Fixtures sind weiterhin willkommen.

Offene Fragen

- Optionale grammY-Plugins (Throttler), falls wir auf Bot-API-429s stoßen.
- Mehr strukturierte Medientests hinzufügen (Sticker, Sprachnachrichten).
- Webhook-Listening-Port konfigurierbar machen (derzeit fest auf 8787, es sei denn, er wird über das Gateway weitergeleitet).