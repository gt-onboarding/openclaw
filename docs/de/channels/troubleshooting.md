---
title: Fehlerbehebung
summary: "Kanalspezifische Schnelltipps zur Fehlerbehebung (Discord/Telegram/WhatsApp)"
read_when:
  - Ein Kanal stellt eine Verbindung her, aber Nachrichten werden nicht übertragen
  - Untersuchung von Fehlkonfigurationen eines Kanals (Intents, Berechtigungen, Datenschutzmodus)
---

<div id="channel-troubleshooting">
  # Fehlerbehebung für Kanäle
</div>

Beginne mit:

```bash
openclaw doctor
openclaw channels status --probe
```

`channels status --probe` gibt Warnungen aus, wenn es häufige Channel-Fehlkonfigurationen erkennen kann, und führt kleine Live-Checks durch (Anmeldedaten, bestimmte Berechtigungen/Mitgliedschaften).

<div id="channels">
  ## Kanäle
</div>

* Discord: [/channels/discord#troubleshooting](/de/channels/discord#troubleshooting)
* Telegram: [/channels/telegram#troubleshooting](/de/channels/telegram#troubleshooting)
* WhatsApp: [/channels/whatsapp#troubleshooting-quick](/de/channels/whatsapp#troubleshooting-quick)

<div id="telegram-quick-fixes">
  ## Telegram-Schnellhilfen
</div>

* In den Logs erscheint `HttpError: Network request for 'sendMessage' failed` oder `sendChatAction` → überprüfe die IPv6-DNS-Konfiguration. Falls `api.telegram.org` zuerst auf IPv6 aufgelöst wird und der Host keinen IPv6-Egress hat, erzwinge IPv4 oder aktiviere IPv6. Siehe [/channels/telegram#troubleshooting](/de/channels/telegram#troubleshooting).
* In den Logs erscheint `setMyCommands failed` → überprüfe ausgehendes HTTPS und die DNS-Erreichbarkeit von `api.telegram.org` (häufig auf stark abgeschotteten VPS oder hinter Proxies).