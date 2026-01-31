---
title: Kopplung
summary: "Überblick zur Kopplung: festlegen, wer dir DMs senden darf + welche Knoten sich verbinden dürfen"
read_when:
  - Einrichten der DM-Zugriffssteuerung
  - Koppeln eines neuen iOS/Android-Knotens
  - Überprüfung des OpenClaw-Sicherheitsstatus
---

<div id="pairing">
  # Kopplung
</div>

„Kopplung“ ist in OpenClaw der explizite Schritt zur **Freigabe durch den Owner**.
Die Kopplung wird an zwei Stellen verwendet:

1. **DM-Kopplung** (wer mit dem Bot sprechen darf)
2. **Knoten-Kopplung** (welche Geräte/Knoten dem Gateway-Netzwerk beitreten dürfen)

Sicherheitskontext: [Sicherheit](/de/gateway/security)

<div id="1-dm-pairing-inbound-chat-access">
  ## 1) DM-Kopplung (eingehender Chat-Zugriff)
</div>

Wenn ein Channel mit der DM-Richtlinie `pairing` konfiguriert ist, erhalten unbekannte Absender einen Kurzcode und ihre Nachricht wird **nicht verarbeitet**, bis du sie genehmigst.

Standard-DM-Richtlinien sind dokumentiert unter: [Security](/de/gateway/security)

Kopplungscodes:

* 8 Zeichen, Großbuchstaben, keine mehrdeutigen Zeichen (`0O1I`).
* **Laufen nach 1 Stunde ab**. Der Bot sendet die Kopplungsnachricht nur, wenn eine neue Anfrage erstellt wird (ungefähr einmal pro Stunde pro Absender).
* Ausstehende DM-Kopplungsanfragen sind standardmäßig auf **3 pro Channel** begrenzt; zusätzliche Anfragen werden ignoriert, bis eine abläuft oder genehmigt wird.

<div id="approve-a-sender">
  ### Einen Absender freigeben
</div>

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

Folgende Kanäle werden unterstützt: `Telegram`, `WhatsApp`, `Signal`, `iMessage`, `Discord`, `Slack`.

### Speicherort des Zustands

Gespeichert unter `~/.openclaw/credentials/`:

* Ausstehende Anfragen: `<channel>-pairing.json`
* Ablage für genehmigte Allowlist-Einträge: `<channel>-allowFrom.json`

Behandle diese Dateien als sensibel, da sie den Zugriff auf deinen Assistenten steuern.

<div id="2-node-device-pairing-iosandroidmacosheadless-nodes">
  ## 2) Knoten-Gerätekopplung (iOS/Android/macOS/Headless-Knoten)
</div>

Knoten verbinden sich mit dem Gateway als **Geräte** mit `role: node`. Das Gateway
erstellt eine Kopplungsanfrage für das Gerät, die bestätigt werden muss.

<div id="approve-a-node-device">
  ### Ein Knotengerät zulassen
</div>

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

<div id="where-the-state-lives">
  ### Wo der Zustand gespeichert wird
</div>

Gespeichert unter `~/.openclaw/devices/`:

* `pending.json` (kurzlebig; ausstehende Anfragen verfallen)
* `paired.json` (gekoppelte Geräte + Token)

<div id="notes">
  ### Hinweise
</div>

* Die veraltete `node.pair.*` API (CLI: `openclaw nodes pending/approve`) ist ein
  separater, vom Gateway verwalteter Kopplungsspeicher. WS-Knoten müssen weiterhin mit dem Gerät gekoppelt werden.

<div id="related-docs">
  ## Verwandte Dokumentation
</div>

* Sicherheitsmodell + Prompt-Injection: [Sicherheit](/de/gateway/security)
* Sicher aktualisieren (`doctor` ausführen): [Aktualisierung](/de/install/updating)
* Channel-Konfigurationen:
  * Telegram: [Telegram](/de/channels/telegram)
  * WhatsApp: [WhatsApp](/de/channels/whatsapp)
  * Signal: [Signal](/de/channels/signal)
  * iMessage: [iMessage](/de/channels/imessage)
  * Discord: [Discord](/de/channels/discord)
  * Slack: [Slack](/de/channels/slack)