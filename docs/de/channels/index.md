---
title: Kanäle
summary: "Messaging-Plattformen, mit denen sich OpenClaw verbinden kann"
read_when:
  - Du möchtest einen Chatkanal für OpenClaw auswählen
  - Du brauchst einen schnellen Überblick über die unterstützten Messaging-Plattformen
---

<div id="chat-channels">
  # Chat-Kanäle
</div>

OpenClaw kann in jeder Chat-App mit dir kommunizieren, die du bereits verwendest. Jeder Kanal ist über das Gateway angebunden.
Text wird überall unterstützt; Medien und Reaktionen variieren je nach Kanal.

<div id="supported-channels">
  ## Unterstützte Kanäle
</div>

* [WhatsApp](/de/channels/whatsapp) — Am beliebtesten; verwendet Baileys und erfordert QR-Kopplung.
* [Telegram](/de/channels/telegram) — Bot-API über grammY; unterstützt Gruppen.
* [Discord](/de/channels/discord) — Discord Bot API + Gateway; unterstützt Server, Kanäle und DMs.
* [Slack](/de/channels/slack) — Bolt SDK; Workspace-Apps.
* [Google Chat](/de/channels/googlechat) — Google-Chat-API-App über HTTP-Webhook.
* [Mattermost](/de/channels/mattermost) — Bot-API + WebSocket; Kanäle, Gruppen, DMs (Plugin, separat installiert).
* [Signal](/de/channels/signal) — signal-cli; datenschutzorientiert.
* [BlueBubbles](/de/channels/bluebubbles) — **Empfohlen für iMessage**; verwendet die BlueBubbles-macOS-Server-REST-API mit vollständiger Feature-Unterstützung (Bearbeiten, Zurückziehen von Nachrichten, Effekte, Reaktionen, Gruppenverwaltung — Bearbeiten funktioniert derzeit auf macOS 26 Tahoe nicht).
* [iMessage](/de/channels/imessage) — Nur macOS; native Integration über imsg (veraltet, für neue Setups BlueBubbles in Betracht ziehen).
* [Microsoft Teams](/de/channels/msteams) — Bot Framework; Enterprise-Support (Plugin, separat installiert).
* [LINE](/de/channels/line) — LINE-Messaging-API-Bot (Plugin, separat installiert).
* [Nextcloud Talk](/de/channels/nextcloud-talk) — Selbstgehosteter Chat über Nextcloud Talk (Plugin, separat installiert).
* [Matrix](/de/channels/matrix) — Matrix-Protokoll (Plugin, separat installiert).
* [Nostr](/de/channels/nostr) — Dezentralisierte DMs über NIP-04 (Plugin, separat installiert).
* [Tlon](/de/channels/tlon) — Urbit-basierter Messenger (Plugin, separat installiert).
* [Twitch](/de/channels/twitch) — Twitch-Chat über IRC-Verbindung (Plugin, separat installiert).
* [Zalo](/de/channels/zalo) — Zalo Bot API; populärer Messenger in Vietnam (Plugin, separat installiert).
* [Zalo Personal](/de/channels/zalouser) — Persönliches Zalo-Konto über QR-Login (Plugin, separat installiert).
* [WebChat](/de/web/webchat) — Gateway-WebChat-UI über WebSocket.

<div id="notes">
  ## Hinweise
</div>

* Channels können gleichzeitig laufen; konfiguriere mehrere und OpenClaw routet pro Chat.
* Die schnellste Einrichtung ist in der Regel **Telegram** (einfacher Bot-Token). WhatsApp erfordert QR-Kopplung und speichert mehr Zustand auf der Festplatte.
* Das Gruppenverhalten variiert je nach Channel; siehe [Groups](/de/concepts/groups).
* DM-Kopplung und Allowlists werden aus Sicherheitsgründen durchgesetzt; siehe [Security](/de/gateway/security).
* Telegram-Interna: [grammY-Hinweise](/de/channels/grammy).
* Problembehandlung: [Channel-Troubleshooting](/de/channels/troubleshooting).
* Modellanbieter sind separat dokumentiert; siehe [Model Providers](/de/providers/models).