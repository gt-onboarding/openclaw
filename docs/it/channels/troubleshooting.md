---
title: Risoluzione dei problemi
summary: "Scorciatoie per la risoluzione dei problemi specifici del canale (Discord/Telegram/WhatsApp)"
read_when:
  - Un canale è connesso ma i messaggi non vengono recapitati
  - Analisi di una configurazione non corretta del canale (intent, permessi, modalità privacy)
---

<div id="channel-troubleshooting">
  # Risoluzione dei problemi dei canali
</div>

Inizia con:

```bash
openclaw doctor
openclaw channels status --probe
```

`channels status --probe` mostra avvisi quando riesce a rilevare errori di configurazione comuni dei canali e include piccole verifiche in tempo reale (credenziali, alcuni permessi/membership).

<div id="channels">
  ## Canali
</div>

* Discord: [/channels/discord#troubleshooting](/it/channels/discord#troubleshooting)
* Telegram: [/channels/telegram#troubleshooting](/it/channels/telegram#troubleshooting)
* WhatsApp: [/channels/whatsapp#troubleshooting-quick](/it/channels/whatsapp#troubleshooting-quick)

<div id="telegram-quick-fixes">
  ## Soluzioni rapide per Telegram
</div>

* I log mostrano `HttpError: Network request for 'sendMessage' failed` o `sendChatAction` → controlla il DNS IPv6. Se `api.telegram.org` viene risolto prima in IPv6 e l&#39;host non ha connettività IPv6 in uscita, forza IPv4 o abilita IPv6. Consulta [/channels/telegram#troubleshooting](/it/channels/telegram#troubleshooting).
* I log mostrano `setMyCommands failed` → controlla la raggiungibilità HTTPS e DNS in uscita verso `api.telegram.org` (problema comune su VPS o proxy fortemente limitati).