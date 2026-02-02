---
title: Nextcloud Talk
summary: "Unterstützungsstatus, Funktionsumfang und Konfiguration von Nextcloud Talk"
read_when:
  - Bei der Arbeit an Funktionen des Nextcloud-Talk-Kanals
---

<div id="nextcloud-talk-plugin">
  # Nextcloud Talk (Plugin)
</div>

Status: wird über ein Plugin (Webhook-Bot) unterstützt. Direktnachrichten, Räume, Reaktionen und Markdown-Nachrichten werden unterstützt.

<div id="plugin-required">
  ## Plugin erforderlich
</div>

Nextcloud Talk wird als Plugin ausgeliefert und ist nicht in der Kerninstallation enthalten.

Installation per CLI (npm-Registry):

```bash
openclaw plugins install @openclaw/nextcloud-talk
```

Lokaler Checkout (bei Ausführung aus einem Git-Repository):

```bash
openclaw plugins install ./extensions/nextcloud-talk
```

Wenn du während der Konfiguration/Ersteinrichtung Nextcloud Talk auswählst und ein Git-Checkout erkannt wird,
wird dir von OpenClaw automatisch der lokale Installationspfad vorgeschlagen.

Details: [Plugins](/de/plugin)

<div id="quick-setup-beginner">
  ## Schnellstart (Einsteiger)
</div>

1. Installiere das Nextcloud-Talk-Plugin.
2. Erstelle auf deinem Nextcloud-Server einen Bot:
   ```bash
   ./occ talk:bot:install "OpenClaw" "<shared-secret>" "<webhook-url>" --feature reaction
   ```
3. Aktiviere den Bot in den Einstellungen des Zielraums.
4. Konfiguriere OpenClaw:
   * Config: `channels.nextcloud-talk.baseUrl` + `channels.nextcloud-talk.botSecret`
   * Oder env: `NEXTCLOUD_TALK_BOT_SECRET` (nur für das Standardkonto)
5. Starte das Gateway neu (oder schließe das Onboarding ab).

Minimale Konfiguration:

```json5
{
  channels: {
    "nextcloud-talk": {
      enabled: true,
      baseUrl: "https://cloud.example.com",
      botSecret: "shared-secret",
      dmPolicy: "pairing"
    }
  }
}
```

<div id="notes">
  ## Hinweise
</div>

* Bots können keine DMs initiieren. Der Nutzer muss den Bot zuerst anschreiben.
* Die Webhook-URL muss vom Gateway erreichbar sein; setze `webhookPublicUrl`, wenn es hinter einem Proxy liegt.
* Medien-Uploads werden von der Bot-API nicht unterstützt; Medien werden als URLs gesendet.
* Die Webhook-Payload unterscheidet nicht zwischen DMs und Räumen; setze `apiUser` + `apiPassword`, um Raumtyp-Abfragen zu aktivieren (ansonsten werden DMs als Räume behandelt).

<div id="access-control-dms">
  ## Zugriffskontrolle (DMs)
</div>

* Standard: `channels.nextcloud-talk.dmPolicy = "pairing"`. Unbekannte Absender erhalten einen Kopplungscode.
* Freigabe über:
  * `openclaw pairing list nextcloud-talk`
  * `openclaw pairing approve nextcloud-talk <CODE>`
* Öffentliche DMs (Policy-Token `open`, das uneingeschränkten Nachrichteneingang von allen Benutzern erlaubt): `channels.nextcloud-talk.dmPolicy="open"` plus `channels.nextcloud-talk.allowFrom=["*"]`.

<div id="rooms-groups">
  ## Räume (Gruppen)
</div>

* Standard: `channels.nextcloud-talk.groupPolicy = "allowlist"` (nur über Erwähnungen zugänglich).
* Räume über `channels.nextcloud-talk.rooms` auf die Allowlist setzen:

```json5
{
  channels: {
    "nextcloud-talk": {
      rooms: {
        "room-token": { requireMention: true }
      }
    }
  }
}
```

* Um keine Räume zuzulassen, lass die Allowlist leer oder setze `channels.nextcloud-talk.groupPolicy="disabled"`.

<div id="capabilities">
  ## Funktionen
</div>

| Funktion | Status |
|---------|--------|
| Direktnachrichten | Unterstützt |
| Räume | Unterstützt |
| Threads | Nicht unterstützt |
| Medien | Nur per URL |
| Reaktionen | Unterstützt |
| Native Befehle | Nicht unterstützt |

<div id="configuration-reference-nextcloud-talk">
  ## Konfigurationsreferenz (Nextcloud Talk)
</div>

Vollständige Konfiguration: [Konfiguration](/de/gateway/configuration)

Anbieter-Optionen:

* `channels.nextcloud-talk.enabled`: Kanalstart aktivieren/deaktivieren.
* `channels.nextcloud-talk.baseUrl`: Nextcloud-Instanz-URL.
* `channels.nextcloud-talk.botSecret`: Gemeinsames Bot-Secret.
* `channels.nextcloud-talk.botSecretFile`: Secret-Dateipfad.
* `channels.nextcloud-talk.apiUser`: API-Benutzer für Raumabfragen (DM-Erkennung).
* `channels.nextcloud-talk.apiPassword`: API-/App-Passwort für Raumabfragen.
* `channels.nextcloud-talk.apiPasswordFile`: API-Passwort-Dateipfad.
* `channels.nextcloud-talk.webhookPort`: Webhook-Listener-Port (Standard: 8788).
* `channels.nextcloud-talk.webhookHost`: Webhook-Host (Standard: 0.0.0.0).
* `channels.nextcloud-talk.webhookPath`: Webhook-Pfad (Standard: /nextcloud-talk-webhook).
* `channels.nextcloud-talk.webhookPublicUrl`: Extern erreichbare Webhook-URL.
* `channels.nextcloud-talk.dmPolicy`: `pairing | allowlist | open | disabled`.
* `channels.nextcloud-talk.allowFrom`: DM-Allowlist (Benutzer-IDs). `open` erfordert `"*"`.
* `channels.nextcloud-talk.groupPolicy`: `allowlist | open | disabled`.
* `channels.nextcloud-talk.groupAllowFrom`: Gruppen-Allowlist (Benutzer-IDs).
* `channels.nextcloud-talk.rooms`: Raumbezogene Einstellungen und Allowlist.
* `channels.nextcloud-talk.historyLimit`: Gruppenverlaufs-Limit (0 deaktiviert).
* `channels.nextcloud-talk.dmHistoryLimit`: DM-Verlaufs-Limit (0 deaktiviert).
* `channels.nextcloud-talk.dms`: DM-spezifische Overrides (`historyLimit`).
* `channels.nextcloud-talk.textChunkLimit`: Größe ausgehender Textblöcke (Zeichen).
* `channels.nextcloud-talk.chunkMode`: `length` (Standard) oder `newline`, um an Leerzeilen (Absatzgrenzen) vor dem Längen-Chunken zu teilen.
* `channels.nextcloud-talk.blockStreaming`: Block-Streaming für diesen Kanal deaktivieren.
* `channels.nextcloud-talk.blockStreamingCoalesce`: Feineinstellungen für die Block-Streaming-Zusammenführung.
* `channels.nextcloud-talk.mediaMaxMb`: Eingehendes Medien-Limit (MB).