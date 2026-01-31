---
title: Bluebubbles
summary: "iMessage über den BlueBubbles-macOS-Server (REST-Senden/-Empfangen, Tippstatus, Reaktionen, kopplung, erweiterte Aktionen)."
read_when:
  - Einrichten des BlueBubbles-Kanals
  - Fehlerbehebung bei der Webhook-kopplung
  - Konfigurieren von iMessage auf macOS
---

<div id="bluebubbles-macos-rest">
  # BlueBubbles (macOS REST)
</div>

Status: Mitgeliefertes Plugin, das über HTTP mit dem BlueBubbles-macOS-Server kommuniziert. **Für iMessage-Integration empfohlen**, da es im Vergleich zum Legacy-Channel `imsg` eine umfangreichere API und eine einfachere Einrichtung bietet.

<div id="overview">
  ## Übersicht
</div>

* Läuft auf macOS über die BlueBubbles-Helper-App ([bluebubbles.app](https://bluebubbles.app)).
* Empfohlen/getestet: macOS Sequoia (15). macOS Tahoe (26) funktioniert; Bearbeiten ist derzeit unter Tahoe defekt, und Gruppen-Icon-Updates können als erfolgreich gemeldet werden, aber nicht synchronisiert werden.
* OpenClaw kommuniziert darüber per REST-API (`GET /api/v1/ping`, `POST /message/text`, `POST /chat/:id/*`).
* Eingehende Nachrichten kommen über Webhooks; ausgehende Antworten, Schreibindikatoren, Lesebestätigungen und Tapbacks sind REST-Aufrufe.
* Anhänge und Sticker werden als eingehende Medien übernommen (und dem agent nach Möglichkeit zur Verfügung gestellt).
* Kopplung/Allowlist funktioniert genauso wie bei anderen Kanälen (`/start/pairing` usw.) mit `channels.bluebubbles.allowFrom` plus Kopplungscodes.
* Reaktionen werden wie bei Slack/Telegram als Systemereignisse bereitgestellt, sodass Agenten sie vor einer Antwort „erwähnen“ können.
* Erweiterte Funktionen: Bearbeiten, Zurückrufen (unsend), Antwort-Threads, Nachrichteneffekte, Gruppenverwaltung.

<div id="quick-start">
  ## Schnellstart
</div>

1. Installiere den BlueBubbles-Server auf deinem Mac und folge den Anweisungen unter [bluebubbles.app/install](https://bluebubbles.app/install).
2. Aktiviere in der BlueBubbles-Konfiguration die Web-API und lege ein Passwort fest.
3. Führe `openclaw onboard` aus und wähle BlueBubbles, oder konfiguriere manuell:
   ```json5
   {
     channels: {
       bluebubbles: {
         enabled: true,
         serverUrl: "http://192.168.1.100:1234",
         password: "example-password",
         webhookPath: "/bluebubbles-webhook"
       }
     }
   }
   ```
4. Lass die BlueBubbles-Webhooks auf dein Gateway zeigen (Beispiel: `https://your-gateway-host:3000/bluebubbles-webhook?password=<password>`).
5. Starte das Gateway; es registriert den Webhook-Handler und startet die Kopplung.

<div id="onboarding">
  ## Onboarding
</div>

BlueBubbles ist im interaktiven Einrichtungsassistenten verfügbar:

```
openclaw onboard
```

Der Assistent fragt nach:

* **Server-URL** (erforderlich): BlueBubbles-Serveradresse (z. B. `http://192.168.1.100:1234`)
* **Passwort** (erforderlich): API-Passwort aus den BlueBubbles-Servereinstellungen
* **Webhook-Pfad** (optional): Standardwert ist `/bluebubbles-webhook`
* **DM-Richtlinie**: pairing, allowlist, open (erlaubt uneingeschränkte Nachrichtenannahme von beliebigen Nutzern) oder disabled
* **Allowlist**: Telefonnummern, E-Mail-Adressen oder Chat-Ziele

Du kannst BlueBubbles auch per CLI hinzufügen:

```
openclaw channels add bluebubbles --http-url http://192.168.1.100:1234 --password <password>
```

<div id="access-control-dms-groups">
  ## Zugriffskontrolle (DMs + Gruppen)
</div>

DMs:

* Standardwert: `channels.bluebubbles.dmPolicy = "pairing"`.
* Unbekannte Absender erhalten einen Kopplungscode; Nachrichten werden ignoriert, bis sie genehmigt werden (Codes laufen nach 1 Stunde ab).
* Genehmigen über:
  * `openclaw pairing list bluebubbles`
  * `openclaw pairing approve bluebubbles <CODE>`
* Kopplung ist der Standardmechanismus für den Token-Austausch. Details: [Pairing](/de/start/pairing)

Gruppen:

* `channels.bluebubbles.groupPolicy = open | allowlist | disabled` (Standardwert: `allowlist`).
* `channels.bluebubbles.groupAllowFrom` steuert, wer in Gruppen triggern kann, wenn `allowlist` gesetzt ist.

<div id="mention-gating-groups">
  ### Mention-Gating (Gruppen)
</div>

BlueBubbles unterstützt Mention-Gating für Gruppenchats und entspricht dabei dem Verhalten von iMessage/WhatsApp:

* Verwendet `agents.list[].groupChat.mentionPatterns` (oder `messages.groupChat.mentionPatterns`), um Erwähnungen zu erkennen.
* Wenn `requireMention` für eine Gruppe aktiviert ist, antwortet der agent nur, wenn er erwähnt wird.
* Steuerbefehle von autorisierten Absendern umgehen das Mention-Gating.

Konfiguration pro Gruppe:

```json5
{
  channels: {
    bluebubbles: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true },  // default for all groups
        "iMessage;-;chat123": { requireMention: false }  // Überschreibung für bestimmte Gruppe
      }
    }
  }
}
```

<div id="command-gating">
  ### Befehlsfreigabe
</div>

* Kontrollbefehle (z. B. `/config`, `/model`) erfordern eine Autorisierung.
* Verwendet `allowFrom` und `groupAllowFrom`, um die Befehlsautorisierung zu steuern.
* Autorisierte Absender können Kontrollbefehle auch in Gruppen ausführen, ohne erwähnt zu werden.

<div id="typing-read-receipts">
  ## Schreib- und Lesebestätigungen
</div>

* **Schreibindikatoren**: Werden automatisch vor und während der Antwortgenerierung gesendet.
* **Lesebestätigungen**: Gesteuert durch `channels.bluebubbles.sendReadReceipts` (Standard: `true`).
* **Schreibindikatoren**: OpenClaw sendet Ereignisse zum Start des Schreibens; BlueBubbles entfernt die Schreibanzeige automatisch beim Senden oder bei einem Timeout (manuelles Beenden per DELETE ist unzuverlässig).

```json5
{
  channels: {
    bluebubbles: {
      sendReadReceipts: false  // Lesebestätigungen deaktivieren
    }
  }
}
```

<div id="advanced-actions">
  ## Erweiterte Aktionen
</div>

BlueBubbles unterstützt erweiterte Aktionen für Nachrichten, sofern diese in der Konfiguration aktiviert sind:

```json5
{
  channels: {
    bluebubbles: {
      actions: {
        reactions: true,       // tapbacks (default: true)
        edit: true,            // gesendete Nachrichten bearbeiten (macOS 13+, defekt unter macOS 26 Tahoe)
        unsend: true,          // unsend messages (macOS 13+)
        reply: true,           // reply threading by message GUID
        sendWithEffect: true,  // message effects (slam, loud, etc.)
        renameGroup: true,     // rename group chats
        setGroupIcon: true,    // set group chat icon/photo (flaky on macOS 26 Tahoe)
        addParticipant: true,  // add participants to groups
        removeParticipant: true, // remove participants from groups
        leaveGroup: true,      // leave group chats
        sendAttachment: true   // send attachments/media
      }
    }
  }
}
```

Verfügbare Aktionen:

* **react**: Tapback-Reaktionen hinzufügen/entfernen (`messageId`, `emoji`, `remove`)
* **edit**: Gesendete Nachricht bearbeiten (`messageId`, `text`)
* **unsend**: Nachricht zurückrufen (`messageId`)
* **reply**: Auf eine bestimmte Nachricht antworten (`messageId`, `text`, `to`)
* **sendWithEffect**: Mit iMessage-Effekt senden (`text`, `to`, `effectId`)
* **renameGroup**: Gruppenchat umbenennen (`chatGuid`, `displayName`)
* **setGroupIcon**: Symbol/Foto eines Gruppenchats festlegen (`chatGuid`, `media`) — unzuverlässig unter macOS 26 Tahoe (API kann Erfolg zurückgeben, aber das Symbol wird nicht synchronisiert).
* **addParticipant**: Jemanden zu einer Gruppe hinzufügen (`chatGuid`, `address`)
* **removeParticipant**: Jemanden aus einer Gruppe entfernen (`chatGuid`, `address`)
* **leaveGroup**: Gruppenchat verlassen (`chatGuid`)
* **sendAttachment**: Medien/Dateien senden (`to`, `buffer`, `filename`, `asVoice`)
  * Sprachnachrichten: Setze `asVoice: true` mit **MP3**- oder **CAF**-Audio, um als iMessage-Sprachnachricht zu senden. BlueBubbles konvertiert beim Senden von Sprachnachrichten MP3 in CAF.

<div id="message-ids-short-vs-full">
  ### Nachrichten-IDs (kurz vs. vollständig)
</div>

OpenClaw kann *kurze* Nachrichten-IDs (z. B. `1`, `2`) ausgeben, um Token zu sparen.

* `MessageSid` / `ReplyToId` können kurze IDs sein.
* `MessageSidFull` / `ReplyToIdFull` enthalten die vollständigen Anbieter-IDs.
* Kurze IDs werden nur im Speicher gehalten; sie können bei einem Neustart oder wenn der Cache geleert wird verfallen.
* Aktionen akzeptieren kurze oder vollständige `messageId`-Werte, aber kurze IDs führen zu einem Fehler, wenn sie nicht mehr verfügbar sind.

Verwende vollständige IDs für dauerhafte Automatisierungen und Speicherung:

* Templates: `{{MessageSidFull}}`, `{{ReplyToIdFull}}`
* Kontext: `MessageSidFull` / `ReplyToIdFull` in eingehenden Payloads

Siehe [Konfiguration](/de/gateway/configuration) für Template-Variablen.

<div id="block-streaming">
  ## Block-Streaming
</div>

Lege fest, ob Antworten als einzelne Nachricht gesendet oder in Blöcken gestreamt werden:

```json5
{
  channels: {
    bluebubbles: {
      blockStreaming: true  // Block-Streaming aktivieren (Standardverhalten)
    }
  }
}
```

<div id="media-limits">
  ## Medien + Limits
</div>

* Eingehende Anhänge werden heruntergeladen und im Medien-Cache gespeichert.
* Medienobergrenze über `channels.bluebubbles.mediaMaxMb` (Standard: 8 MB).
* Ausgehender Text wird in Blöcke der Größe `channels.bluebubbles.textChunkLimit` aufgeteilt (Standard: 4000 Zeichen).

<div id="configuration-reference">
  ## Konfigurationsreferenz
</div>

Vollständige Konfiguration: [Konfiguration](/de/gateway/configuration)

Anbieteroptionen:

* `channels.bluebubbles.enabled`: Kanal aktivieren/deaktivieren.
* `channels.bluebubbles.serverUrl`: BlueBubbles REST-API-Basis-URL.
* `channels.bluebubbles.password`: API-Passwort.
* `channels.bluebubbles.webhookPath`: Webhook-Endpunktpfad (Standard: `/bluebubbles-webhook`).
* `channels.bluebubbles.dmPolicy`: `pairing | allowlist | open | disabled` (Standard: `pairing`).
* `channels.bluebubbles.allowFrom`: DM-Allowlist (Handles, E-Mails, E.164-Nummern, `chat_id:*`, `chat_guid:*`).
* `channels.bluebubbles.groupPolicy`: `open | allowlist | disabled` (Standard: `allowlist`).
* `channels.bluebubbles.groupAllowFrom`: Allowlist für Gruppensender.
* `channels.bluebubbles.groups`: Konfiguration pro Gruppe (`requireMention` usw.).
* `channels.bluebubbles.sendReadReceipts`: Lesebestätigungen senden (Standard: `true`).
* `channels.bluebubbles.blockStreaming`: Block-Streaming aktivieren (Standard: `true`).
* `channels.bluebubbles.textChunkLimit`: Ausgehende Chunk-Größe in Zeichen (Standard: 4000).
* `channels.bluebubbles.chunkMode`: `length` (Standard) teilt nur auf, wenn `textChunkLimit` überschritten wird; `newline` teilt zunächst an Leerzeilen (Absatzgrenzen), bevor anschließend nach Länge gechunkt wird.
* `channels.bluebubbles.mediaMaxMb`: Maximale Größe eingehender Medien in MB (Standard: 8).
* `channels.bluebubbles.historyLimit`: Maximale Anzahl von Gruppennachrichten für Kontext (0 deaktiviert).
* `channels.bluebubbles.dmHistoryLimit`: Limit für DM-Verlauf.
* `channels.bluebubbles.actions`: Bestimmte Aktionen aktivieren/deaktivieren.
* `channels.bluebubbles.accounts`: Multi-Account-Konfiguration.

Zugehörige globale Optionen:

* `agents.list[].groupChat.mentionPatterns` (oder `messages.groupChat.mentionPatterns`).
* `messages.responsePrefix`.

<div id="addressing-delivery-targets">
  ## Adressierung / Zustellziele
</div>

Verwende bevorzugt `chat_guid` für stabiles Routing:

* `chat_guid:iMessage;-;+15555550123` (bevorzugt für Gruppen)
* `chat_id:123`
* `chat_identifier:...`
* Direkte Handles: `+15555550123`, `user@example.com`
  * Wenn für ein direktes Handle kein bestehender DM-Chat vorhanden ist, erstellt OpenClaw einen solchen über `POST /api/v1/chat/new`. Dafür muss die BlueBubbles Private API aktiviert sein.

<div id="security">
  ## Sicherheit
</div>

* Webhook-Anfragen werden authentifiziert, indem die `guid`-/`password`-Query-Parameter bzw. -Header mit `channels.bluebubbles.password` verglichen werden. Anfragen von `localhost` werden ebenfalls akzeptiert.
* Halte das API-Passwort und den Webhook-Endpunkt geheim (behandle sie wie Zugangsdaten).
* Das Vertrauen in localhost bedeutet, dass ein Reverse-Proxy auf demselben Host das Passwort unbeabsichtigt umgehen kann. Wenn du das Gateway per Proxy bereitstellst, verlange Authentifizierung am Proxy und konfiguriere `gateway.trustedProxies`. Siehe [Gateway-Sicherheit](/de/gateway/security#reverse-proxy-configuration).
* Aktiviere HTTPS und Firewall-Regeln auf dem BlueBubbles-Server, wenn du ihn außerhalb deines LAN öffentlich zugänglich machst.

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

* Wenn Tippen-/read-Ereignisse nicht mehr funktionieren, überprüfe die BlueBubbles-Webhook-Logs und stelle sicher, dass der Gateway-Pfad mit `channels.bluebubbles.webhookPath` übereinstimmt.
* Pairing-Codes laufen nach einer Stunde ab; verwende `openclaw pairing list bluebubbles` und `openclaw pairing approve bluebubbles <code>`.
* Reaktionen erfordern die private BlueBubbles-API (`POST /api/v1/message/react`); stelle sicher, dass die Serverversion sie bereitstellt.
* Bearbeiten/Zurückrufen erfordern macOS 13+ und eine kompatible BlueBubbles-Serverversion. Unter macOS 26 (Tahoe) ist Bearbeiten derzeit aufgrund von Änderungen an der privaten API fehlerhaft.
* Gruppen-Icon-Aktualisierungen können unter macOS 26 (Tahoe) unzuverlässig sein: Die API kann Erfolg zurückgeben, aber das neue Icon wird nicht synchronisiert.
* OpenClaw blendet bekannte fehlerhafte Aktionen automatisch aus, basierend auf der macOS-Version des BlueBubbles-Servers. Wenn Bearbeiten unter macOS 26 (Tahoe) weiterhin angezeigt wird, deaktiviere es manuell mit `channels.bluebubbles.actions.edit=false`.
* Für Status-/Health-Infos: `openclaw status --all` oder `openclaw status --deep`.

Für allgemeine Referenzen zu Channel-Workflows siehe [Channels](/de/channels) und den Leitfaden [Plugins](/de/plugins).