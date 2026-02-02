---
title: Canali
summary: "Piattaforme di messaggistica a cui OpenClaw può collegarsi"
read_when:
  - Vuoi scegliere un canale chat per OpenClaw
  - Hai bisogno di una rapida panoramica delle piattaforme di messaggistica supportate
---

<div id="chat-channels">
  # Canali di chat
</div>

OpenClaw può comunicare con te su qualsiasi app di chat che già usi. Ogni canale si connette tramite il Gateway.
I messaggi di testo sono supportati ovunque; il supporto per contenuti multimediali e reazioni varia a seconda del canale.

<div id="supported-channels">
  ## Canali supportati
</div>

* [WhatsApp](/it/channels/whatsapp) — Il più diffuso; usa Baileys e richiede l&#39;abbinamento tramite QR.
* [Telegram](/it/channels/telegram) — Bot API tramite grammY; supporta i gruppi.
* [Discord](/it/channels/discord) — Discord Bot API + Gateway; supporta server, canali e DM (messaggi diretti).
* [Slack](/it/channels/slack) — Bolt SDK; app per spazi di lavoro.
* [Google Chat](/it/channels/googlechat) — App Google Chat API tramite webhook HTTP.
* [Mattermost](/it/channels/mattermost) — Bot API + WebSocket; canali, gruppi, DM (plugin, installato separatamente).
* [Signal](/it/channels/signal) — signal-cli; incentrato sulla privacy.
* [BlueBubbles](/it/channels/bluebubbles) — **Consigliato per iMessage**; usa la REST API del server BlueBubbles per macOS con supporto completo delle funzionalità (modifica, annullamento dell&#39;invio, effetti, reazioni, gestione dei gruppi — modifica attualmente non funzionante su macOS 26 Tahoe).
* [iMessage](/it/channels/imessage) — Solo macOS; integrazione nativa tramite imsg (legacy, valuta BlueBubbles per nuove configurazioni).
* [Microsoft Teams](/it/channels/msteams) — Bot Framework; supporto enterprise (plugin, installato separatamente).
* [LINE](/it/channels/line) — Bot LINE Messaging API (plugin, installato separatamente).
* [Nextcloud Talk](/it/channels/nextcloud-talk) — Chat auto-ospitata tramite Nextcloud Talk (plugin, installato separatamente).
* [Matrix](/it/channels/matrix) — Protocollo Matrix (plugin, installato separatamente).
* [Nostr](/it/channels/nostr) — DM decentralizzati tramite NIP-04 (plugin, installato separatamente).
* [Tlon](/it/channels/tlon) — Messenger basato su Urbit (plugin, installato separatamente).
* [Twitch](/it/channels/twitch) — Chat Twitch tramite connessione IRC (plugin, installato separatamente).
* [Zalo](/it/channels/zalo) — Zalo Bot API; app di messaggistica molto diffusa in Vietnam (plugin, installato separatamente).
* [Zalo Personal](/it/channels/zalouser) — Account personale Zalo tramite login con QR (plugin, installato separatamente).
* [WebChat](/it/web/webchat) — UI WebChat del Gateway su WebSocket.

<div id="notes">
  ## Note
</div>

* I canali possono essere eseguiti contemporaneamente; configura più canali e OpenClaw instraderà i messaggi in base alla chat.
* La configurazione più rapida di solito è **Telegram** (semplice bot token). WhatsApp richiede una procedura di abbinamento tramite QR e conserva più stato su disco.
* Il comportamento nei gruppi varia in base al canale; vedi [Gruppi](/it/concepts/groups).
* L&#39;abbinamento in DM e la lista di autorizzati vengono applicati per motivi di sicurezza; vedi [Sicurezza](/it/gateway/security).
* Dettagli interni di Telegram: [note su grammY](/it/channels/grammy).
* Risoluzione dei problemi: [Risoluzione dei problemi dei canali](/it/channels/troubleshooting).
* I provider di modelli sono documentati separatamente; vedi [provider di modelli](/it/providers/models).