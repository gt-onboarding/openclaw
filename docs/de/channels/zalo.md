---
title: Zalo
summary: "Status der Unterstützung für Zalo-Bots, Funktionsumfang und Konfiguration"
read_when:
  - Wenn du an Zalo-Funktionen oder Webhooks arbeitest
---

<div id="zalo-bot-api">
  # Zalo (Bot API)
</div>

Status: experimentell. Nur Direktnachrichten; Gruppen werden laut Zalo-Dokumentation bald unterstützt.

<div id="plugin-required">
  ## Plugin erforderlich
</div>

Zalo wird als Plugin bereitgestellt und ist nicht in der Kerninstallation enthalten.

* Über die CLI installieren: `openclaw plugins install @openclaw/zalo`
* Oder **Zalo** während des Onboardings auswählen und die Installationsaufforderung bestätigen
* Details: [Plugins](/de/plugin)

<div id="quick-setup-beginner">
  ## Schnelle Einrichtung (Einsteiger)
</div>

1. Installiere das Zalo-Plugin:
   * Aus einem lokalen Quellcode-Checkout: `openclaw plugins install ./extensions/zalo`
   * Über npm (falls veröffentlicht): `openclaw plugins install @openclaw/zalo`
   * Oder wähle **Zalo** im Onboarding und bestätige die Installationsaufforderung
2. Setze das Token:
   * Umgebungsvariable: `ZALO_BOT_TOKEN=...`
   * Oder Config: `channels.zalo.botToken: "..."`.
3. Starte das Gateway neu (oder schließe das Onboarding ab).
4. DM-Zugriff erfolgt standardmäßig per Kopplung; bestätige den Kopplungscode beim ersten Kontakt.

Minimale Konfiguration:

```json5
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "kopplung"
    }
  }
}
```

<div id="what-it-is">
  ## Was es ist
</div>

Zalo ist eine Messaging-App mit Schwerpunkt Vietnam; die Bot-API ermöglicht es dem Gateway, einen Bot für 1:1-Unterhaltungen zu betreiben.
Sie eignet sich gut für Support oder Benachrichtigungen, bei denen du deterministisches Routing zurück zu Zalo benötigst.

* Ein Zalo-Bot-API-Kanal, der vom Gateway verwaltet wird.
* Deterministisches Routing: Antworten gehen zurück zu Zalo; das Modell wählt niemals Kanäle.
* DMs teilen sich die Hauptsitzung des Agents.
* Gruppen werden noch nicht unterstützt (laut Zalo-Dokumentation „coming soon“).

<div id="setup-fast-path">
  ## Einrichtung (Schnellstart)
</div>

<div id="1-create-a-bot-token-zalo-bot-platform">
  ### 1) Erstelle ein Bot-Token (Zalo Bot Platform)
</div>

1. Rufe **https://bot.zaloplatforms.com** in deinem Browser auf und melde dich an.
2. Erstelle einen neuen Bot und konfiguriere ihn.
3. Kopiere das Bot-Token (Format: `12345689:abc-xyz`).

<div id="2-configure-the-token-env-or-config">
  ### 2) Konfiguriere das Token (env oder config)
</div>

Beispiel:

```json5
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing"
    }
  }
}
```

Env-Option: `ZALO_BOT_TOKEN=...` (funktioniert nur für das Standardkonto).

Unterstützung für mehrere Konten: Verwende `channels.zalo.accounts` mit Tokens pro Konto und optionalem `name`.

3. Starte das Gateway neu. Zalo startet, sobald ein Token verfügbar ist (Umgebung oder Config).
4. DM-Zugriff ist standardmäßig auf Kopplung gesetzt. Bestätige den Code, wenn der Bot zum ersten Mal kontaktiert wird.

<div id="how-it-works-behavior">
  ## Funktionsweise (Verhalten)
</div>

* Eingehende Nachrichten werden in den gemeinsamen Channel-Envelope mit Medienplatzhaltern normalisiert.
* Antworten werden immer an denselben Zalo-Chat zurückgesendet.
* Standardmäßig wird Long-Polling verwendet; Webhook-Modus verfügbar über `channels.zalo.webhookUrl`.

<div id="limits">
  ## Limits
</div>

* Ausgehender Text wird in 2000-Zeichen-Blöcke aufgeteilt (Zalo API-Limit).
* Medien-Downloads/-Uploads sind durch `channels.zalo.mediaMaxMb` begrenzt (Standardwert 5).
* Streaming ist standardmäßig deaktiviert, da das 2000-Zeichen-Limit Streaming kaum sinnvoll macht.

<div id="access-control-dms">
  ## Zugriffskontrolle (DMs)
</div>

<div id="dm-access">
  ### DM-Zugriff
</div>

* Standardwert: `channels.zalo.dmPolicy = "pairing"`. Unbekannte Absender erhalten einen Kopplungscode; Nachrichten werden ignoriert, bis sie genehmigt werden (Codes laufen nach 1 Stunde ab).
* Genehmigung über:
  * `openclaw pairing list zalo`
  * `openclaw pairing approve zalo <CODE>`
* Die Kopplung ist der Standardmechanismus für den Token-Austausch. Details: [Kopplung](/de/start/pairing)
* `channels.zalo.allowFrom` akzeptiert numerische Benutzer-IDs (keine Auflösung nach Benutzernamen verfügbar).

<div id="long-polling-vs-webhook">
  ## Long-Polling vs. Webhook
</div>

* Standard: Long-Polling (keine öffentliche URL erforderlich).
* Webhook-Modus: Setze `channels.zalo.webhookUrl` und `channels.zalo.webhookSecret`.
  * Das Webhook-Secret muss 8–256 Zeichen lang sein.
  * Die Webhook-URL muss HTTPS verwenden.
  * Zalo sendet Events mit dem Header `X-Bot-Api-Secret-Token` zur Verifizierung.
  * Das Gateway-HTTP-Interface verarbeitet Webhook-Anfragen unter `channels.zalo.webhookPath` (standardmäßig der Pfad der Webhook-URL).

**Hinweis:** `getUpdates` (Polling) und Webhook schließen sich laut der Zalo-API-Dokumentation gegenseitig aus.

<div id="supported-message-types">
  ## Unterstützte Nachrichtentypen
</div>

* **Textnachrichten**: Volle Unterstützung mit Chunking in 2000-Zeichen-Blöcke.
* **Bildnachrichten**: Eingehende Bilder werden heruntergeladen und verarbeitet; Bilder werden über `sendPhoto` gesendet.
* **Sticker**: Protokolliert, aber nicht vollständig verarbeitet (keine Agent-Antwort).
* **Nicht unterstützte Nachrichtentypen**: Protokolliert (z. B. Nachrichten von geschützten Nutzern).

<div id="capabilities">
  ## Funktionen
</div>

| Feature | Status |
|---------|--------|
| Direktnachrichten | ✅ Unterstützt |
| Gruppen | ❌ Demnächst verfügbar (laut Zalo-Dokumentation) |
| Medien (Bilder) | ✅ Unterstützt |
| Reaktionen | ❌ Nicht unterstützt |
| Threads | ❌ Nicht unterstützt |
| Umfragen | ❌ Nicht unterstützt |
| Native Befehle | ❌ Nicht unterstützt |
| Streaming | ⚠️ Blockiert (Limit von 2000 Zeichen) |

<div id="delivery-targets-clicron">
  ## Zieladressen (CLI/cron)
</div>

* Verwende eine Chat-ID als Zieladresse.
* Beispiel: `openclaw message send --channel zalo --target 123456789 --message "hi"`.

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

**Bot antwortet nicht:**

* Token auf Gültigkeit prüfen: `openclaw channels status --probe`
* Prüfen, ob der Absender freigeschaltet ist (Kopplung oder allowFrom)
* Gateway-Logs prüfen: `openclaw logs --follow`

**Webhook empfängt keine Ereignisse:**

* Sicherstellen, dass die Webhook-URL HTTPS verwendet
* Prüfen, dass das Secret-Token 8–256 Zeichen lang ist
* Bestätigen, dass der Gateway-HTTP-Endpunkt unter dem konfigurierten Pfad erreichbar ist
* Prüfen, dass getUpdates-Polling nicht läuft (diese Modi schließen sich gegenseitig aus)

<div id="configuration-reference-zalo">
  ## Konfigurationsreferenz (Zalo)
</div>

Vollständige Konfiguration: [Konfiguration](/de/gateway/configuration)

Anbieteroptionen:

* `channels.zalo.enabled`: Kanalstart aktivieren/deaktivieren.
* `channels.zalo.botToken`: Bot-Token von der Zalo Bot Platform.
* `channels.zalo.tokenFile`: Token aus Dateipfad lesen.
* `channels.zalo.dmPolicy`: `pairing | allowlist | open | disabled` (Standard: pairing).
* `channels.zalo.allowFrom`: DM-Allowlist (Benutzer-IDs). `open` erfordert `"*"`. Der Assistent fragt nach numerischen IDs.
* `channels.zalo.mediaMaxMb`: Maximale Größe eingehender/ausgehender Medien (MB, Standard 5).
* `channels.zalo.webhookUrl`: Webhook-Modus aktivieren (HTTPS erforderlich).
* `channels.zalo.webhookSecret`: Webhook-Secret (8–256 Zeichen).
* `channels.zalo.webhookPath`: Webhook-Pfad auf dem Gateway-HTTP-Server.
* `channels.zalo.proxy`: Proxy-URL für API-Anfragen.

Multi-Account-Optionen:

* `channels.zalo.accounts.<id>.botToken`: Token pro Account.
* `channels.zalo.accounts.<id>.tokenFile`: Token-Datei pro Account.
* `channels.zalo.accounts.<id>.name`: Anzeigename.
* `channels.zalo.accounts.<id>.enabled`: Account aktivieren/deaktivieren.
* `channels.zalo.accounts.<id>.dmPolicy`: DM-Richtlinie pro Account.
* `channels.zalo.accounts.<id>.allowFrom`: Allowlist pro Account.
* `channels.zalo.accounts.<id>.webhookUrl`: Webhook-URL pro Account.
* `channels.zalo.accounts.<id>.webhookSecret`: Webhook-Secret pro Account.
* `channels.zalo.accounts.<id>.webhookPath`: Webhook-Pfad pro Account.
* `channels.zalo.accounts.<id>.proxy`: Proxy-URL pro Account.