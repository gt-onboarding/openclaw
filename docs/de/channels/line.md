---
title: LINE
summary: "Einrichtung, Konfiguration und Nutzung des LINE Messaging API Plugin"
read_when:
  - Du möchtest OpenClaw mit LINE verbinden
  - Du musst LINE-Webhook und Zugangsdaten einrichten
  - Du möchtest LINE-spezifische Nachrichtenoptionen verwenden
---

<div id="line-plugin">
  # LINE (plugin)
</div>

LINE verbindet sich über die LINE Messaging API mit OpenClaw. Das Plugin läuft als
Webhook-Empfänger auf dem Gateway und verwendet dein Channel-Access-Token und dein
Channel-Secret zur Authentifizierung.

Status: per Plugin unterstützt. Direktnachrichten, Gruppenchats, Medien, Standorte,
Flex Messages, Template Messages und Quick Replies werden unterstützt. Reactions und
Threads werden nicht unterstützt.

<div id="plugin-required">
  ## Plugin erforderlich
</div>

Installiere das LINE-Plugin:

```bash
openclaw plugins install @openclaw/line
```

Lokaler Checkout (bei Ausführung aus einem Git-Repository):

```bash
openclaw plugins install ./extensions/line
```


<div id="setup">
  ## Einrichtung
</div>

1. Erstelle ein LINE Developers-Konto und öffne die Console:
   https://developers.line.biz/console/
2. Erstelle (oder wähle) einen Anbieter und füge einen **Messaging API**-Kanal hinzu.
3. Kopiere das **Channel access token** und das **Channel secret** aus den Kanaleinstellungen.
4. Aktiviere **Use webhook** in den Messaging-API-Einstellungen.
5. Setze die Webhook-URL auf den Gateway-Endpunkt (HTTPS erforderlich):

```
https://gateway-host/line/webhook
```

Das Gateway antwortet auf die Webhook-Verifizierung (GET) von LINE und verarbeitet eingehende Events (POST).
Wenn du einen benutzerdefinierten Pfad brauchst, setze `channels.line.webhookPath` oder
`channels.line.accounts.<id>.webhookPath` und aktualisiere die URL entsprechend.


<div id="configure">
  ## Konfigurieren
</div>

Minimale Konfiguration:

```json5
{
  channels: {
    line: {
      enabled: true,
      channelAccessToken: "LINE_CHANNEL_ACCESS_TOKEN",
      channelSecret: "LINE_CHANNEL_SECRET",
      dmPolicy: "pairing"
    }
  }
}
```

Umgebungsvariablen (nur Standardkonto):

* `LINE_CHANNEL_ACCESS_TOKEN`
* `LINE_CHANNEL_SECRET`

Token-/Secret-Dateien:

```json5
{
  channels: {
    line: {
      tokenFile: "/path/to/line-token.txt",
      secretFile: "/path/to/line-secret.txt"
    }
  }
}
```

Mehrere Accounts:

```json5
{
  channels: {
    line: {
      accounts: {
        marketing: {
          channelAccessToken: "...",
          channelSecret: "...",
          webhookPath: "/line/marketing"
        }
      }
    }
  }
}
```


<div id="access-control">
  ## Zugriffskontrolle
</div>

Direktnachrichten erfordern standardmäßig eine Kopplung. Unbekannte Absender erhalten einen Kopplungscode und ihre Nachrichten werden ignoriert, bis sie genehmigt werden.

```bash
openclaw pairing list line
openclaw pairing approve line <CODE>
```

Allowlists und Richtlinien:

* `channels.line.dmPolicy`: `pairing | allowlist | open | disabled`
* `channels.line.allowFrom`: in der Allowlist eingetragene LINE-Benutzer-IDs für DMs
* `channels.line.groupPolicy`: `allowlist | open | disabled`
* `channels.line.groupAllowFrom`: in der Allowlist eingetragene LINE-Benutzer-IDs für Gruppen
* gruppenspezifische Überschreibungen: `channels.line.groups.<groupId>.allowFrom`

LINE-IDs unterscheiden Groß- und Kleinschreibung. Gültige IDs sehen wie folgt aus:

* Benutzer: `U` + 32 hexadezimale Zeichen
* Gruppe: `C` + 32 hexadezimale Zeichen
* Raum: `R` + 32 hexadezimale Zeichen


<div id="message-behavior">
  ## Nachrichtenverhalten
</div>

- Text wird in Blöcke mit 5000 Zeichen aufgeteilt.
- Markdown-Formatierung wird entfernt; Codeblöcke und Tabellen werden, wenn möglich, in Flex Cards konvertiert.
- Streaming-Antworten werden gepuffert; LINE erhält vollständige Abschnitte mit einer Ladeanimation, während der agent arbeitet.
- Mediendownloads sind durch `channels.line.mediaMaxMb` begrenzt (Standard 10).

<div id="channel-data-rich-messages">
  ## Kanal-Daten (Rich Messages)
</div>

Verwende `channelData.line`, um Schnellantworten, Standorte, Flex Cards oder Vorlagennachrichten zu senden.

```json5
{
  text: "Here you go",
  channelData: {
    line: {
      quickReplies: ["Status", "Help"],
      location: {
        title: "Office",
        address: "123 Main St",
        latitude: 35.681236,
        longitude: 139.767125
      },
      flexMessage: {
        altText: "Status card",
        contents: { /* Flex payload */ }
      },
      templateMessage: {
        type: "confirm",
        text: "Proceed?",
        confirmLabel: "Yes",
        confirmData: "yes",
        cancelLabel: "No",
        cancelData: "no"
      }
    }
  }
}
```

Das LINE-Plugin stellt außerdem einen `/card`-Befehl für Flex-Message-Vorlagen bereit:

```
/card info "Welcome" "Thanks for joining!"
```


<div id="troubleshooting">
  ## Fehlerbehebung
</div>

- **Webhook-Überprüfung schlägt fehl:** Stell sicher, dass die Webhook-URL per HTTPS erreichbar ist und
  `channelSecret` mit der Einstellung in der LINE-Konsole übereinstimmt.
- **Keine eingehenden Events:** Überprüfe, ob der Webhook-Pfad mit `channels.line.webhookPath`
  übereinstimmt und dass das Gateway von LINE aus erreichbar ist.
- **Fehler beim Herunterladen von Medien:** Erhöhe `channels.line.mediaMaxMb`, wenn Mediendateien
  das Standardlimit überschreiten.