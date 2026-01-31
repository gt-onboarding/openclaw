---
title: Msteams
summary: "Status der Bot-Unterstützung für Microsoft Teams, Funktionsumfang und Konfiguration"
read_when:
  - Arbeit an Funktionen des MS-Teams-Kanals
---

<div id="microsoft-teams-plugin">
  # Microsoft Teams (Plugin)
</div>

> „Lasst, die ihr eintretet, alle Hoffnung fahren.“

Aktualisiert: 2026-01-21

Status: Text- und DM-Anhänge werden unterstützt; das Senden von Dateien in Channel- und Gruppen-Chats erfordert `sharePointSiteId` + Graph-API-Berechtigungen (siehe [Senden von Dateien in Gruppenchats](#sending-files-in-group-chats)). Umfragen werden über Adaptive Cards gesendet.

<div id="plugin-required">
  ## Plugin erforderlich
</div>

Microsoft Teams wird als Plugin ausgeliefert und ist nicht im Core-Installationspaket enthalten.

**Breaking Change (2026.1.15):** MS Teams wurde aus dem Core ausgelagert. Wenn du es nutzt, musst du das Plugin installieren.

Begründung: Hält Core-Installationen schlanker und ermöglicht es, Abhängigkeiten von MS Teams unabhängig zu aktualisieren.

Installation über die CLI (npm-Registry):

```bash
openclaw plugins install @openclaw/msteams
```

Lokaler Checkout (wenn du aus einem Git-Repository arbeitest):

```bash
openclaw plugins install ./extensions/msteams
```

Wenn du während der Konfiguration bzw. beim Onboarding Teams auswählst und ein Git-Checkout erkannt wird,
bietet dir OpenClaw den lokalen Installationspfad automatisch an.

Details: [Plugins](/de/plugin)

<div id="quick-setup-beginner">
  ## Schnelleinrichtung (für Einsteiger)
</div>

1. Installiere das Microsoft-Teams-Plugin.
2. Erstelle einen **Azure Bot** (App-ID + Client-Secret + Tenant-ID).
3. Konfiguriere OpenClaw mit diesen Zugangsdaten.
4. Stelle `/api/messages` (standardmäßig Port 3978) über eine öffentliche URL oder einen Tunnel bereit.
5. Installiere das Teams-App-Paket und starte das Gateway.

Minimalkonfiguration:

```json5
{
  channels: {
    msteams: {
      enabled: true,
      appId: "<APP_ID>",
      appPassword: "<APP_PASSWORD>",
      tenantId: "<TENANT_ID>",
      webhook: { port: 3978, path: "/api/messages" }
    }
  }
}
```

Hinweis: Gruppenchats sind standardmäßig blockiert (`channels.msteams.groupPolicy: "allowlist"`). Um Antworten in Gruppenchats zu erlauben, setze `channels.msteams.groupAllowFrom` (oder verwende `groupPolicy: "open"` – d. h. uneingeschränkte Annahme von Nachrichten von beliebigen Nutzern –, um Antworten von beliebigen Mitgliedern zuzulassen, gesteuert über Erwähnungen).

<div id="goals">
  ## Ziele
</div>

* Mit OpenClaw über Teams-Direktnachrichten, Gruppenchats oder Kanäle kommunizieren.
* Routing deterministisch halten: Antworten gehen immer in den Kanal zurück, über den sie eingegangen sind.
* Standardmäßig sicheres Kanalverhalten (Erwähnungen erforderlich, sofern nicht anders konfiguriert).

<div id="config-writes">
  ## Config-Schreibzugriffe
</div>

Standardmäßig darf Microsoft Teams Konfigurationsupdates schreiben, die durch `/config set|unset` ausgelöst werden (benötigt `commands.config: true`).

Deaktivierst du mit:

```json5
{
  channels: { msteams: { configWrites: false } }
}
```

<div id="access-control-dms-groups">
  ## Zugriffskontrolle (DMs + Gruppen)
</div>

**DM-Zugriff**

* Standard: `channels.msteams.dmPolicy = "pairing"`. Unbekannte Absender werden ignoriert, bis du sie explizit genehmigst.
* `channels.msteams.allowFrom` akzeptiert AAD-Objekt-IDs, UPNs oder Anzeigenamen. Der Wizard löst Namen über Microsoft Graph in IDs auf, wenn die verwendeten Anmeldedaten dies zulassen.

**Gruppenzugriff**

* Standard: `channels.msteams.groupPolicy = "allowlist"` (blockiert, sofern du `groupAllowFrom` nicht hinzufügst). Verwende `channels.defaults.groupPolicy`, um den Standardwert zu überschreiben, wenn er nicht gesetzt ist.
* `channels.msteams.groupAllowFrom` steuert, welche Absender in Gruppen-Chats/-Kanälen triggern können (fällt zurück auf `channels.msteams.allowFrom`).
* Setze `groupPolicy: "open"`, um jedes Mitglied zuzulassen (d. h. standardmäßig weiterhin nur über Erwähnungen, aber mit uneingeschränkter Nachrichtenannahme von allen Mitgliedern).
* Um **keine Kanäle** zuzulassen, setze `channels.msteams.groupPolicy: "disabled"`.

Beispiel:

```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"]
    }
  }
}
```

**Teams + Kanal-Allowlist**

* Begrenze Antworten auf Gruppen/Kanäle, indem du Teams und Kanäle unter `channels.msteams.teams` auflistest.
* Schlüssel können Team-IDs oder -Namen sein; Kanal-Schlüssel können Konversations-IDs oder -Namen sein.
* Wenn `groupPolicy="allowlist"` gesetzt ist und eine Teams-Allowlist konfiguriert ist, werden nur die aufgeführten Teams/Kanäle akzeptiert (mention-gesteuert).
* Der Konfigurationsassistent akzeptiert `Team/Channel`‑Einträge und speichert sie für dich.
* Beim Start löst OpenClaw Team-/Kanal- und Benutzer-Allowlist-Namen in IDs auf (wenn Graph-Berechtigungen dies zulassen)
  und protokolliert die Zuordnung; nicht auflösbare Einträge bleiben wie eingegeben erhalten.

Beispiel:

```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      teams: {
        "My Team": {
          channels: {
            "General": { requireMention: true }
          }
        }
      }
    }
  }
}
```

<div id="how-it-works">
  ## Funktionsweise
</div>

1. Installiere das Microsoft-Teams-Plugin.
2. Erstelle einen **Azure Bot** (App-ID + Secret + Tenant-ID).
3. Baue ein **Teams-App-Paket**, das auf den Bot verweist und die unten aufgeführten RSC-Berechtigungen enthält.
4. Lade die Teams-App in ein Team hoch und installiere sie dort (oder im persönlichen Scope für DMs).
5. Konfiguriere `msteams` in `~/.openclaw/openclaw.json` (oder per Umgebungsvariablen) und starte das Gateway.
6. Das Gateway lauscht standardmäßig auf Bot-Framework-Webhook-Traffic unter `/api/messages`.

<div id="azure-bot-setup-prerequisites">
  ## Azure-Bot-Einrichtung (Voraussetzungen)
</div>

Bevor du OpenClaw konfigurierst, musst du zunächst eine Azure-Bot-Ressource erstellen.

<div id="step-1-create-azure-bot">
  ### Schritt 1: Azure Bot erstellen
</div>

1. Gehe zu [Create Azure Bot](https://portal.azure.com/#create/Microsoft.AzureBot)
2. Fülle die Registerkarte **Basics** aus:

   | Feld | Wert |
   |-------|-------|
   | **Bot handle** | Dein Bot-Name, z. B. `openclaw-msteams` (muss eindeutig sein) |
   | **Subscription** | Wähle dein Azure-Abonnement aus |
   | **Resource group** | Neu erstellen oder vorhandene verwenden |
   | **Pricing tier** | **Free** für Entwicklung und Tests |
   | **Type of App** | **Single Tenant** (empfohlen – siehe Hinweis unten) |
   | **Creation type** | **Create new Microsoft App ID** |

> **Hinweis zur Außerdienststellung:** Die Erstellung neuer Multi-Tenant-Bots ist seit dem 31.07.2025 nicht mehr möglich. Verwende **Single Tenant** für neue Bots.

3. Klicke auf **Review + create** → **Create** (warte ca. 1–2 Minuten)

<div id="step-2-get-credentials">
  ### Schritt 2: Zugangsdaten abrufen
</div>

1. Wechsle zu deiner Azure-Bot-Ressource → **Configuration**
2. Kopiere die **Microsoft App ID** → das ist deine `appId`
3. Klicke auf **Manage Password** → wechsle zur App Registration
4. Unter **Certificates &amp; secrets** → **New client secret** → kopiere den **Value** → das ist dein `appPassword`
5. Gehe zu **Overview** → kopiere **Directory (tenant) ID** → das ist deine `tenantId`

<div id="step-3-configure-messaging-endpoint">
  ### Schritt 3: Messaging-Endpunkt konfigurieren
</div>

1. In Azure Bot → **Configuration**
2. Stelle **Messaging endpoint** auf deine Webhook-URL ein:
   * Produktion: `https://your-domain.com/api/messages`
   * Lokale Entwicklung: Verwende einen Tunnel (siehe [Lokale Entwicklung](#local-development-tunneling) unten)

<div id="step-4-enable-teams-channel">
  ### Schritt 4: Teams-Kanal aktivieren
</div>

1. Im Azure-Bot-Portal → **Channels**
2. Klicke auf **Microsoft Teams** → Configure → Save
3. Akzeptiere die Nutzungsbedingungen

<div id="local-development-tunneling">
  ## Lokale Entwicklung (Tunneling)
</div>

Microsoft Teams kann nicht direkt auf `localhost` zugreifen. Verwende einen Tunnel für die lokale Entwicklung:

**Option A: ngrok**

```bash
ngrok http 3978
# Copy the https URL, e.g., https://abc123.ngrok.io
# Setzen Sie den Messaging-Endpunkt auf: https://abc123.ngrok.io/api/messages
```

**Option B: Tailscale Funnel**

```bash
tailscale funnel 3978
# Verwenden Sie Ihre Tailscale-Funnel-URL als Messaging-Endpunkt
```

<div id="teams-developer-portal-alternative">
  ## Teams Developer Portal (Alternative)
</div>

Anstatt manuell ein Manifest-ZIP zu erstellen, kannst du das [Teams Developer Portal](https://dev.teams.microsoft.com/apps) verwenden:

1. Klicke auf **+ New app**
2. Fülle die Basisinformationen aus (Name, Beschreibung, Entwicklerinformationen)
3. Gehe zu **App features** → **Bot**
4. Wähle **Enter a bot ID manually** und füge deine Azure Bot App ID ein
5. Überprüfe die Scopes: **Personal**, **Team**, **Group Chat**
6. Klicke auf **Distribute** → **Download app package**
7. In Teams: **Apps** → **Manage your apps** → **Upload a custom app** → wähle das ZIP aus

Das ist oft einfacher, als JSON-Manifeste manuell zu bearbeiten.

<div id="testing-the-bot">
  ## Bot testen
</div>

**Option A: Azure Web Chat (zuerst Webhook verifizieren)**

1. Im Azure-Portal → deine Azure Bot-Ressource → **Test in Web Chat**
2. Sende eine Nachricht – du solltest eine Antwort sehen
3. Dies bestätigt, dass dein Webhook-Endpunkt funktioniert, bevor du Teams einrichtest

**Option B: Teams (nach App-Installation)**

1. Installiere die Teams-App (Sideload oder Organisationskatalog)
2. Suche den Bot in Teams und sende ihm eine Direktnachricht
3. Überprüfe die Gateway-Logs auf eingehende Aktivitäten

<div id="setup-minimal-text-only">
  ## Setup (minimal, nur Text)
</div>

1. **Microsoft Teams-Plugin installieren**
   * Über npm: `openclaw plugins install @openclaw/msteams`
   * Aus einem lokalen Checkout: `openclaw plugins install ./extensions/msteams`

2. **Bot-Registrierung**
   * Erstelle einen Azure Bot (siehe oben) und notiere:
     * App ID
     * Client-Secret (App-Passwort)
     * Tenant ID (Single-Tenant)

3. **Teams-App-Manifest**
   * Füge einen `bot`-Eintrag mit `botId = <App ID>` hinzu.
   * Scopes: `personal`, `team`, `groupChat`.
   * `supportsFiles: true` (erforderlich für Datei-Handling im `personal`-Scope).
   * Füge RSC-Berechtigungen hinzu (siehe unten).
   * Erstelle Icons: `outline.png` (32x32) und `color.png` (192x192).
   * Packe alle drei Dateien in ein ZIP-Archiv: `manifest.json`, `outline.png`, `color.png`.

4. **OpenClaw konfigurieren**

   ```json
   {
     "msteams": {
       "enabled": true,
       "appId": "<APP_ID>",
       "appPassword": "<APP_PASSWORD>",
       "tenantId": "<TENANT_ID>",
       "webhook": { "port": 3978, "path": "/api/messages" }
     }
   }
   ```

   Du kannst auch Umgebungsvariablen anstelle von Konfigurationsschlüsseln verwenden:

   * `MSTEAMS_APP_ID`
   * `MSTEAMS_APP_PASSWORD`
   * `MSTEAMS_TENANT_ID`

5. **Bot-Endpunkt**
   * Setze den Azure Bot Messaging Endpoint auf:
     * `https://<host>:3978/api/messages` (oder deinen gewählten Pfad/Port).

6. **Gateway ausführen**
   * Der Teams-Kanal startet automatisch, wenn das Plugin installiert ist und eine `msteams`-Konfiguration mit Zugangsdaten vorhanden ist.

<div id="history-context">
  ## Verlaufskontext
</div>

* `channels.msteams.historyLimit` steuert, wie viele der letzten Kanal-/Gruppennachrichten in den Prompt aufgenommen werden.
* Fällt auf `messages.groupChat.historyLimit` zurück. Setze `0`, um zu deaktivieren (Standard: 50).
* Der DM-Verlauf kann mit `channels.msteams.dmHistoryLimit` begrenzt werden (Benutzernachrichten). Benutzerspezifische Overrides: `channels.msteams.dms["<user_id>"].historyLimit`.

<div id="current-teams-rsc-permissions-manifest">
  ## Aktuelle Teams-RSC-Berechtigungen (Manifest)
</div>

Dies sind die **bestehenden resourceSpecific-Berechtigungen** in unserem Teams-App-Manifest. Sie gelten nur innerhalb des Teams/Chats, in dem die App installiert ist.

**Für Kanäle (Team-Scope):**

* `ChannelMessage.Read.Group` (Application) - alle Kanalnachrichten ohne @mention empfangen
* `ChannelMessage.Send.Group` (Application)
* `Member.Read.Group` (Application)
* `Owner.Read.Group` (Application)
* `ChannelSettings.Read.Group` (Application)
* `TeamMember.Read.Group` (Application)
* `TeamSettings.Read.Group` (Application)

**Für Gruppenchats:**

* `ChatMessage.Read.Chat` (Application) - alle Gruppenchatsnachrichten ohne @mention empfangen

<div id="example-teams-manifest-redacted">
  ## Beispiel eines Teams-Manifests (anonymisiert)
</div>

Minimales, gültiges Beispiel mit den erforderlichen Feldern. Ersetzen Sie IDs und URLs.

```json
{
  "$schema": "https://developer.microsoft.com/en-us/json-schemas/teams/v1.23/MicrosoftTeams.schema.json",
  "manifestVersion": "1.23",
  "version": "1.0.0",
  "id": "00000000-0000-0000-0000-000000000000",
  "name": { "short": "OpenClaw" },
  "developer": {
    "name": "Your Org",
    "websiteUrl": "https://example.com",
    "privacyUrl": "https://example.com/privacy",
    "termsOfUseUrl": "https://example.com/terms"
  },
  "description": { "short": "OpenClaw in Teams", "full": "OpenClaw in Teams" },
  "icons": { "outline": "outline.png", "color": "color.png" },
  "accentColor": "#5B6DEF",
  "bots": [
    {
      "botId": "11111111-1111-1111-1111-111111111111",
      "scopes": ["personal", "team", "groupChat"],
      "isNotificationOnly": false,
      "supportsCalling": false,
      "supportsVideo": false,
      "supportsFiles": true
    }
  ],
  "webApplicationInfo": {
    "id": "11111111-1111-1111-1111-111111111111"
  },
  "authorization": {
    "permissions": {
      "resourceSpecific": [
        { "name": "ChannelMessage.Read.Group", "type": "Application" },
        { "name": "ChannelMessage.Send.Group", "type": "Application" },
        { "name": "Member.Read.Group", "type": "Application" },
        { "name": "Owner.Read.Group", "type": "Application" },
        { "name": "ChannelSettings.Read.Group", "type": "Application" },
        { "name": "TeamMember.Read.Group", "type": "Application" },
        { "name": "TeamSettings.Read.Group", "type": "Application" },
        { "name": "ChatMessage.Read.Chat", "type": "Application" }
      ]
    }
  }
}
```

<div id="manifest-caveats-must-have-fields">
  ### Hinweise zum Manifest (Pflichtfelder)
</div>

* `bots[].botId` **muss** mit der Azure Bot App ID übereinstimmen.
* `webApplicationInfo.id` **muss** mit der Azure Bot App ID übereinstimmen.
* `bots[].scopes` muss die Kontexte enthalten, die du verwenden möchtest (`personal`, `team`, `groupChat`).
* `bots[].supportsFiles: true` ist für die Dateiverarbeitung im persönlichen Scope erforderlich.
* `authorization.permissions.resourceSpecific` muss die Channel-Berechtigungen `read` und `send` enthalten, wenn du Nachrichtenverkehr in Kanälen verarbeiten möchtest.

<div id="updating-an-existing-app">
  ### Aktualisieren einer bestehenden App
</div>

So aktualisierst du eine bereits installierte Teams-App (z. B. um RSC-Berechtigungen hinzuzufügen):

1. Aktualisiere deine `manifest.json` mit den neuen Einstellungen.
2. **Erhöhe die Versionsnummer im Feld `version`** (z. B. `1.0.0` → `1.1.0`).
3. **Erstelle das ZIP-Archiv neu**, das Manifest und Icons enthält (`manifest.json`, `outline.png`, `color.png`).
4. Lade das neue ZIP hoch:
   * **Option A (Teams Admin Center):** Teams Admin Center → Teams apps → Manage apps → deine App suchen → Upload new version
   * **Option B (Sideload):** In Teams → Apps → Manage your apps → Upload a custom app
5. **Für Team-Kanäle:** Installiere die App in jedem Team neu, damit die neuen Berechtigungen greifen.
6. **Beende Teams vollständig und starte es neu** (nicht nur das Fenster schließen), um zwischengespeicherte App-Metadaten zu löschen.

<div id="capabilities-rsc-only-vs-graph">
  ## Funktionen: nur RSC vs. Graph
</div>

<div id="with-teams-rsc-only-app-installed-no-graph-api-permissions">
  ### Mit **nur Teams RSC** (App installiert, keine Graph-API-Berechtigungen)
</div>

Funktioniert:

* Lesen von **Textinhalten** in Kanalnachrichten.
* Senden von **Textinhalten** in Kanalnachrichten.
* Empfangen von **persönlichen (DM)** Dateianhängen.

Funktioniert NICHT:

* **Bild- oder Dateiinhalte** in Kanal-/Gruppennachrichten (Payload enthält nur einen HTML-Stub).
* Herunterladen von Anhängen, die in SharePoint/OneDrive gespeichert sind.
* Lesen des Nachrichtenverlaufs (über das Live-Webhook-Ereignis hinaus).

<div id="with-teams-rsc-microsoft-graph-application-permissions">
  ### Mit **Teams RSC + Microsoft Graph-Anwendungsberechtigungen**
</div>

Ermöglicht:

* Herunterladen gehosteter Inhalte (in Nachrichten eingefügte Bilder).
* Herunterladen von Dateianhängen, die in SharePoint/OneDrive gespeichert sind.
* Lesen des Kanal-/Chat-Nachrichtenverlaufs über Microsoft Graph.

<div id="rsc-vs-graph-api">
  ### RSC vs Graph API
</div>

| Funktionalität | RSC-Berechtigungen | Graph API |
|----------------|--------------------|-----------|
| **Echtzeit-Nachrichten** | Ja (über Webhook) | Nein (nur Polling) |
| **Verlauf von Nachrichten** | Nein | Ja (kann Verlauf abfragen) |
| **Einrichtungsaufwand** | Nur App-Manifest | Erfordert Administratorzustimmung + Token-Flow |
| **Funktioniert offline** | Nein (muss aktiv sein) | Ja (jederzeit abfragbar) |

**Fazit:** RSC ist für das Empfangen von Nachrichten in Echtzeit gedacht; Graph API ist für den Zugriff auf historische Daten. Um verpasste Nachrichten nachzuholen, wenn du offline warst, benötigst du die Graph API mit `ChannelMessage.Read.All` (erfordert Administratorzustimmung).

<div id="graph-enabled-media-history-required-for-channels">
  ## Medien und Verlauf mit Graph-Unterstützung (für Kanäle erforderlich)
</div>

Wenn du Bilder/Dateien in **Kanälen** benötigst oder **Nachrichtenverlauf** abrufen möchtest, musst du Microsoft Graph-Berechtigungen aktivieren und eine Administratorzustimmung erteilen.

1. Füge in der **App-Registrierung** von Entra ID (Azure AD) die folgenden Microsoft Graph-**Anwendungsberechtigungen** hinzu:
   * `ChannelMessage.Read.All` (Kanalanhänge + Verlauf)
   * `Chat.Read.All` oder `ChatMessage.Read.All` (Gruppen-Chats)
2. **Erteile Administratorzustimmung** für den Mandanten.
3. Erhöhe die **Manifestversion** der Teams-App, lade sie erneut hoch und **installiere die App in Teams neu**.
4. **Beende Teams vollständig und starte es neu**, um zwischengespeicherte App-Metadaten zu löschen.

<div id="known-limitations">
  ## Bekannte Einschränkungen
</div>

<div id="webhook-timeouts">
  ### Webhook-Timeouts
</div>

Teams liefert Nachrichten über einen HTTP-Webhook. Wenn die Verarbeitung zu lange dauert (z. B. langsame LLM-Antworten), kannst du Folgendes beobachten:

* Gateway-Timeouts
* Teams sendet die Nachricht erneut (was zu Duplikaten führen kann)
* Verworfene Antworten

OpenClaw geht damit um, indem es schnell antwortet und Antworten proaktiv sendet, aber sehr langsame Antworten können trotzdem Probleme verursachen.

<div id="formatting">
  ### Formatierung
</div>

Teams-Markdown ist stärker eingeschränkt als in Slack oder Discord:

* Grundlegende Formatierung funktioniert: **fett**, *kursiv*, `Code`, Links
* Komplexes Markdown (Tabellen, verschachtelte Listen) wird möglicherweise nicht korrekt gerendert
* Adaptive Cards werden für Umfragen und das Senden beliebiger Karten unterstützt (siehe unten)

<div id="configuration">
  ## Konfiguration
</div>

Wichtige Einstellungen (siehe `/gateway/configuration` für gemeinsame Channel-Muster):

* `channels.msteams.enabled`: Channel aktivieren/deaktivieren.
* `channels.msteams.appId`, `channels.msteams.appPassword`, `channels.msteams.tenantId`: Bot-Anmeldedaten.
* `channels.msteams.webhook.port` (Standard `3978`)
* `channels.msteams.webhook.path` (Standard `/api/messages`)
* `channels.msteams.dmPolicy`: `pairing | allowlist | open | disabled` (Standard: pairing)
* `channels.msteams.allowFrom`: Allowlist für DMs (AAD-Objekt-IDs, UPNs oder Anzeigenamen). Der Wizard löst Namen während der Einrichtung in IDs auf, wenn Graph-Zugriff verfügbar ist.
* `channels.msteams.textChunkLimit`: Größe ausgehender Textblöcke.
* `channels.msteams.chunkMode`: `length` (Standard) oder `newline`, um an Leerzeilen (Absatzgrenzen) zu teilen, bevor nach Länge aufgeteilt wird.
* `channels.msteams.mediaAllowHosts`: Allowlist für eingehende Attachment-Hosts (Standard sind Microsoft-/Teams-Domains).
* `channels.msteams.requireMention`: @Mention in Channels/Gruppen erforderlich (Standard true).
* `channels.msteams.replyStyle`: `thread | top-level` (siehe [Reply Style](#reply-style-threads-vs-posts)).
* `channels.msteams.teams.<teamId>.replyStyle`: teamweite Überschreibung.
* `channels.msteams.teams.<teamId>.requireMention`: teamweite Überschreibung.
* `channels.msteams.teams.<teamId>.tools`: Standard-Teamrichtlinien-Überschreibungen für Tools (`allow`/`deny`/`alsoAllow`), die verwendet werden, wenn eine Channel-Überschreibung fehlt.
* `channels.msteams.teams.<teamId>.toolsBySender`: Standard-Teamrichtlinien-Überschreibungen pro Absender für Tools (Wildcard `"*"` wird unterstützt).
* `channels.msteams.teams.<teamId>.channels.<conversationId>.replyStyle`: Channel-spezifische Überschreibung.
* `channels.msteams.teams.<teamId>.channels.<conversationId>.requireMention`: Channel-spezifische Überschreibung.
* `channels.msteams.teams.<teamId>.channels.<conversationId>.tools`: Channel-spezifische Tool-Richtlinien-Überschreibungen (`allow`/`deny`/`alsoAllow`).
* `channels.msteams.teams.<teamId>.channels.<conversationId>.toolsBySender`: Channel-spezifische Tool-Richtlinien-Überschreibungen pro Absender (Wildcard `"*"` wird unterstützt).
* `channels.msteams.sharePointSiteId`: SharePoint-Site-ID für Datei-Uploads in Gruppenchats/Channels (siehe [Sending files in group chats](#sending-files-in-group-chats)).

<div id="routing-sessions">
  ## Routing &amp; Sitzungen
</div>

* Sitzungsschlüssel folgen dem Standardformat für agents (siehe [/concepts/session](/de/concepts/session)):
  * Direktnachrichten verwenden die Hauptsitzung (`agent:<agentId>:<mainKey>`).
  * Kanal- und Gruppennachrichten verwenden die Konversations-ID:
    * `agent:<agentId>:msteams:channel:<conversationId>`
    * `agent:<agentId>:msteams:group:<conversationId>`

<div id="reply-style-threads-vs-posts">
  ## Antwortstil: Threads vs. Posts
</div>

Teams hat vor Kurzem zwei Kanal-UI-Stile auf Basis desselben zugrunde liegenden Datenmodells eingeführt:

| Stil                        | Beschreibung                                                    | Empfohlener `replyStyle` |
| --------------------------- | --------------------------------------------------------------- | ------------------------ |
| **Posts** (klassisch)       | Nachrichten erscheinen als Karten mit Thread-Antworten darunter | `thread` (Standard)      |
| **Threads** (Slack-ähnlich) | Nachrichten laufen linear durch, eher wie in Slack              | `top-level`              |

**Das Problem:** Die Teams-API macht nicht kenntlich, welchen UI-Stil ein Kanal verwendet. Wenn du den falschen `replyStyle` verwendest:

* `thread` in einem Threads-Stil-Kanal → Antworten erscheinen unnatürlich verschachtelt
* `top-level` in einem Posts-Stil-Kanal → Antworten erscheinen als separate Top-Level-Posts statt im Thread

**Lösung:** Konfiguriere `replyStyle` je Kanal abhängig davon, wie der Kanal eingerichtet ist:

```json
{
  "msteams": {
    "replyStyle": "thread",
    "teams": {
      "19:abc...@thread.tacv2": {
        "channels": {
          "19:xyz...@thread.tacv2": {
            "replyStyle": "top-level"
          }
        }
      }
    }
  }
}
```

<div id="attachments-images">
  ## Anhänge &amp; Bilder
</div>

**Aktuelle Einschränkungen:**

* **DMs:** Bilder und Dateianhänge funktionieren über die Teams-Bot-Datei-APIs.
* **Kanäle/Gruppen:** Anhänge liegen im M365-Speicher (SharePoint/OneDrive). Die Webhook-Payload enthält nur einen HTML-Platzhalter, nicht die eigentlichen Datei-Bytes. **Graph API-Berechtigungen sind erforderlich**, um Kanalanhänge herunterzuladen.

Ohne Graph API-Berechtigungen werden Kanalnachrichten mit Bildern nur als Text empfangen (der Bildinhalt ist für den Bot nicht zugänglich).
Standardmäßig lädt OpenClaw Medien nur von Microsoft-/Teams-Hostnames herunter. Du kannst dies mit `channels.msteams.mediaAllowHosts` überschreiben (verwende `["*"]`, um jeden Host zuzulassen).

<div id="sending-files-in-group-chats">
  ## Senden von Dateien in Gruppenchats
</div>

Bots können Dateien in DMs mit dem integrierten FileConsentCard-Flow senden. Das **Senden von Dateien in Gruppenchats/Kanälen** erfordert jedoch zusätzliche Einrichtung:

| Kontext | Wie Dateien gesendet werden | Erforderliche Einrichtung |
|---------|----------------------------|---------------------------|
| **DMs** | FileConsentCard → Benutzer akzeptiert → Bot lädt hoch | Funktioniert ohne weitere Einrichtung |
| **Gruppenchats/Kanäle** | Upload nach SharePoint → Link freigeben | Erfordert `sharePointSiteId` + Graph-Berechtigungen |
| **Bilder (beliebiger Kontext)** | Inline als Base64-codiert | Funktioniert ohne weitere Einrichtung |

<div id="why-group-chats-need-sharepoint">
  ### Warum Gruppenchats SharePoint benötigen
</div>

Bots haben kein persönliches OneDrive-Laufwerk (der `/me/drive` Graph API-Endpunkt funktioniert nicht für App-Identitäten). Um Dateien in Gruppenchats oder Kanälen zu senden, lädt der Bot sie auf eine **SharePoint-Website** hoch und erstellt einen Freigabelink.

<div id="setup">
  ### Einrichtung
</div>

1. **Graph-API-Berechtigungen hinzufügen** in Entra ID (Azure AD) → App-Registrierung:
   * `Sites.ReadWrite.All` (Application) – Dateien in SharePoint hochladen
   * `Chat.Read.All` (Application) – optional, aktiviert Freigabelinks für einzelne Benutzer

2. **Administratorzustimmung** für den Mandanten erteilen.

3. **SharePoint-Site-ID abrufen:**
   ```bash
   # Über Graph Explorer oder curl mit einem gültigen Token:
   curl -H "Authorization: Bearer $TOKEN" \
     "https://graph.microsoft.com/v1.0/sites/{hostname}:/{site-path}"

   # Beispiel: für eine Site unter „contoso.sharepoint.com/sites/BotFiles“
   curl -H "Authorization: Bearer $TOKEN" \
     "https://graph.microsoft.com/v1.0/sites/contoso.sharepoint.com:/sites/BotFiles"

   # Die Antwort enthält: "id": "contoso.sharepoint.com,guid1,guid2"
   ```

4. **OpenClaw konfigurieren:**
   ```json5
   {
     channels: {
       msteams: {
         // ... weitere Konfiguration ...
         sharePointSiteId: "contoso.sharepoint.com,guid1,guid2"
       }
     }
   }
   ```

<div id="sharing-behavior">
  ### Freigabeverhalten
</div>

| Berechtigung | Freigabeverhalten |
|------------|------------------|
| Nur `Sites.ReadWrite.All` | organisationsweiter Freigabelink (alle in der Organisation können zugreifen) |
| `Sites.ReadWrite.All` + `Chat.Read.All` | benutzerspezifischer Freigabelink (nur Chat-Mitglieder können zugreifen) |

Benutzerspezifische Freigabe ist sicherer, da nur die Chat-Teilnehmenden auf die Datei zugreifen können. Wenn die Berechtigung `Chat.Read.All` fehlt, fällt der Bot auf die organisationsweite Freigabe zurück.

<div id="fallback-behavior">
  ### Fallback-Verhalten
</div>

| Szenario | Ergebnis |
|----------|--------|
| Gruppenchat + Datei + `sharePointSiteId` konfiguriert | Upload nach SharePoint, Freigabelink wird gesendet |
| Gruppenchat + Datei + keine `sharePointSiteId` | Versuch eines OneDrive-Uploads (kann fehlschlagen), nur Text wird gesendet |
| Persönlicher Chat + Datei | FileConsentCard-Flow (funktioniert ohne SharePoint) |
| Beliebiger Kontext + Bild | Base64-codiert inline eingebettet (funktioniert ohne SharePoint) |

<div id="files-stored-location">
  ### Speicherort der Dateien
</div>

Hochgeladene Dateien werden im Ordner `/OpenClawShared/` in der Standarddokumentbibliothek der konfigurierten SharePoint-Website gespeichert.

<div id="polls-adaptive-cards">
  ## Umfragen (Adaptive Cards)
</div>

OpenClaw sendet Umfragen in Teams als Adaptive Cards (es gibt keine native Teams-API für Umfragen).

* CLI: `openclaw message poll --channel msteams --target conversation:<id> ...`
* Stimmen werden vom Gateway in der Datei `~/.openclaw/msteams-polls.json` aufgezeichnet.
* Das Gateway muss online bleiben, um Stimmen zu erfassen.
* Umfragen veröffentlichen Ergebniszusammenfassungen derzeit noch nicht automatisch (prüfe bei Bedarf die Speicherdatei).

<div id="adaptive-cards-arbitrary">
  ## Adaptive Cards (beliebig)
</div>

Sende beliebige Adaptive-Card-JSON-Daten an Teams-Benutzer oder -Unterhaltungen mit dem `message`-Tool oder über die CLI.

Der Parameter `card` akzeptiert ein Adaptive-Card-JSON-Objekt. Wenn `card` angegeben ist, ist der Nachrichtentext optional.

**Agent-Tool:**

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "user:<id>",
  "card": {
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": [{"type": "TextBlock", "text": "Hello!"}]
  }
}
```

**CLI:**

```bash
openclaw message send --channel msteams \
  --target "conversation:19:abc...@thread.tacv2" \
  --card '{"type":"AdaptiveCard","version":"1.5","body":[{"type":"TextBlock","text":"Hello!"}]}'
```

Siehe die [Dokumentation zu Adaptive Cards](https://adaptivecards.io/) für das Kartenschema und Beispiele. Weitere Details zu den Zielformaten findest du weiter unten unter [Ziel-Formate](#target-formats).

<div id="target-formats">
  ## Zielformate
</div>

Microsoft-Teams-Ziele verwenden Präfixe, um zwischen Benutzern und Unterhaltungen zu unterscheiden:

| Zieltyp              | Format                           | Beispiel                                                |
| -------------------- | -------------------------------- | ------------------------------------------------------- |
| Benutzer (nach ID)   | `user:<aad-object-id>`           | `user:40a1a0ed-4ff2-4164-a219-55518990c197`             |
| Benutzer (nach Name) | `user:<display-name>`            | `user:John Smith` (erfordert Graph API)                 |
| Gruppe/Kanal         | `conversation:<conversation-id>` | `conversation:19:abc123...@thread.tacv2`                |
| Gruppe/Kanal (roh)   | `<conversation-id>`              | `19:abc123...@thread.tacv2` (wenn es `@thread` enthält) |

**CLI-Beispiele:**

```bash
# Send to a user by ID
openclaw message send --channel msteams --target "user:40a1a0ed-..." --message "Hello"

# An einen Benutzer per Anzeigename senden (löst Graph API-Lookup aus)
openclaw message send --channel msteams --target "user:John Smith" --message "Hello"

# Send to a group chat or channel
openclaw message send --channel msteams --target "conversation:19:abc...@thread.tacv2" --message "Hello"

# Send an Adaptive Card to a conversation
openclaw message send --channel msteams --target "conversation:19:abc...@thread.tacv2" \
  --card '{"type":"AdaptiveCard","version":"1.5","body":[{"type":"TextBlock","text":"Hello"}]}'
```

**Beispiele für Agent-Tools:**

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "user:John Smith",
  "message": "Hello!"
}
```

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "conversation:19:abc...@thread.tacv2",
  "card": {"type": "AdaptiveCard", "version": "1.5", "body": [{"type": "TextBlock", "text": "Hello"}]}
}
```

Hinweis: Ohne das Präfix `user:` erfolgt die Namensauflösung standardmäßig für Gruppen/Teams. Verwende immer `user:`, wenn du Personen über ihren Anzeigenamen ansprichst.

<div id="proactive-messaging">
  ## Proaktive Nachrichten
</div>

* Proaktive Nachrichten sind erst **möglich**, nachdem ein Benutzer interagiert hat, da wir ab diesem Zeitpunkt Gesprächsreferenzen speichern.
* Siehe `/gateway/configuration` für `dmPolicy`- und Allowlist-Gating.

<div id="team-and-channel-ids-common-gotcha">
  ## Team- und Channel-IDs (häufige Fehlerquelle)
</div>

Der `groupId`-Query-Parameter in Teams-URLs ist **NICHT** die Team-ID, die für die Konfiguration verwendet wird. Extrahiere stattdessen die IDs aus dem URL-Pfad:

**Team-URL:**

```
https://teams.microsoft.com/l/team/19%3ABk4j...%40thread.tacv2/conversations?groupId=...
                                    └────────────────────────────┘
                                    Team-ID (URL-dekodieren)
```

**Kanal-URL:**

```
https://teams.microsoft.com/l/channel/19%3A15bc...%40thread.tacv2/ChannelName?groupId=...
                                      └─────────────────────────┘
                                      Channel ID (URL-decode this)
```

**Für die Konfiguration:**

* Team-ID = Pfadsegment hinter `/team/` (URL-dekodiert, z. B. `19:Bk4j...@thread.tacv2`)
* Channel-ID = Pfadsegment hinter `/channel/` (URL-dekodiert)
* **Ignorieren Sie** den `groupId`-Query-Parameter

<div id="private-channels">
  ## Private Channels
</div>

Bots werden in privaten Kanälen nur eingeschränkt unterstützt:

| Feature | Standardkanäle | Private Kanäle |
|---------|----------------|----------------|
| Bot-Installation | Ja | Eingeschränkt |
| Echtzeitnachrichten (Webhook) | Ja | Funktionieren unter Umständen nicht |
| RSC-Berechtigungen | Ja | Kann sich anders verhalten |
| @-Erwähnungen | Ja | Wenn der Bot zugänglich ist |
| Graph-API-Verlauf | Ja | Ja (mit Berechtigungen) |

**Workarounds, falls private Kanäle nicht funktionieren:**

1. Verwende Standardkanäle für Bot-Interaktionen
2. Verwende DMs – Benutzer können dem Bot immer direkt schreiben
3. Verwende die Graph API für den Zugriff auf den Verlauf (erfordert `ChannelMessage.Read.All`)

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

<div id="common-issues">
  ### Häufige Probleme
</div>

* **Bilder werden in Kanälen nicht angezeigt:** Graph-Berechtigungen oder Adminzustimmung fehlen. Installiere die Teams-App neu und beende Teams vollständig, dann starte es neu.
* **Keine Antworten im Kanal:** Erwähnungen sind standardmäßig erforderlich; setze `channels.msteams.requireMention=false` oder konfiguriere die Einstellung pro Team/Kanal.
* **Versionskonflikt (Teams zeigt weiterhin altes Manifest an):** Entferne die App, füge sie erneut hinzu und beende Teams vollständig, um zu aktualisieren.
* **401 Unauthorized vom Webhook:** Wird erwartet, wenn du manuell ohne Azure JWT testest – bedeutet, dass der Endpunkt erreichbar ist, aber die Authentifizierung fehlgeschlagen ist. Verwende Azure Web Chat für einen korrekten Test.

<div id="manifest-upload-errors">
  ### Manifest-Upload-Fehler
</div>

* **&quot;Icon file cannot be empty&quot;:** Das Manifest verweist auf Icon-Dateien, die 0 Byte groß sind. Erstelle gültige PNG-Icons (32x32 für `outline.png`, 192x192 für `color.png`).
* **&quot;webApplicationInfo.Id already in use&quot;:** Die App ist noch in einem anderen Team/Chat installiert. Suche sie und deinstalliere sie zuerst, oder warte 5–10 Minuten, bis sich die Änderung verbreitet hat.
* **&quot;Something went wrong&quot; beim Upload:** Lade die App stattdessen über https://admin.teams.microsoft.com hoch, öffne die Entwicklertools deines Browsers (F12) → Registerkarte „Network“ und prüfe den Response-Body auf die eigentliche Fehlermeldung.
* **Sideload schlägt fehl:** Versuche „Upload an app to your org&#39;s app catalog“ anstelle von „Upload a custom app“ – das umgeht häufig Sideload-Beschränkungen.

<div id="rsc-permissions-not-working">
  ### RSC-Berechtigungen funktionieren nicht
</div>

1. Überprüfe, ob `webApplicationInfo.id` exakt mit der App-ID deines Bots übereinstimmt
2. Lade die App erneut hoch und installiere sie im Team oder Chat neu
3. Prüfe, ob dein Organisationsadministrator die RSC-Berechtigungen blockiert hat
4. Stelle sicher, dass du den richtigen Scope verwendest: `ChannelMessage.Read.Group` für Teams, `ChatMessage.Read.Chat` für Gruppenchats

<div id="references">
  ## Referenzen
</div>

* [Create Azure Bot](https://learn.microsoft.com/en-us/azure/bot-service/bot-service-quickstart-registration) - Anleitung zur Einrichtung eines Azure Bots
* [Teams Developer Portal](https://dev.teams.microsoft.com/apps) - Teams-Apps erstellen und verwalten
* [Teams app manifest schema](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema) - Schema für das Teams-App-Manifest
* [Receive channel messages with RSC](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/channel-messages-with-rsc) - Kanalnachrichten mit RSC empfangen
* [RSC permissions reference](https://learn.microsoft.com/en-us/microsoftteams/platform/graph-api/rsc/resource-specific-consent) - Referenz zu RSC-Berechtigungen
* [Teams bot file handling](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/bots-filesv4) (Kanal/Gruppen erfordern Graph)
* [Proactive messaging](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/send-proactive-messages) - Proaktive Nachrichten