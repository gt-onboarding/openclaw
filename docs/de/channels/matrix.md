---
title: Matrix
summary: "Status der Matrix-Unterstützung, Features und Konfiguration"
read_when:
  - Wenn du an Matrix-Kanal-Features arbeitest
---

<div id="matrix-plugin">
  # Matrix (Plugin)
</div>

Matrix ist ein offenes, dezentralisiertes Messaging-Protokoll. OpenClaw verbindet sich als Matrix-**Benutzer**
auf einem beliebigen Homeserver, daher benötigst du ein Matrix-Konto für den Bot. Sobald er eingeloggt ist, kannst du
dem Bot direkt eine Direktnachricht (DM) senden oder ihn in Räume (Matrix-„Gruppen“) einladen. Beeper ist ebenfalls eine geeignete Client-Option,
erfordert aber, dass E2EE aktiviert ist.

Status: unterstützt über ein Plugin (@vector-im/matrix-bot-sdk). Direktnachrichten, Räume, Threads, Medien, Reaktionen,
Umfragen (Senden + Start der Umfrage als Text), Standortfreigabe und E2EE (mit Krypto-Unterstützung).

<div id="plugin-required">
  ## Plugin erforderlich
</div>

Matrix wird als Plugin bereitgestellt und ist nicht in der Kerninstallation enthalten.

Installation über die CLI (npm-Registry):

```bash
openclaw plugins install @openclaw/matrix
```

Lokaler Checkout (bei Ausführung aus einem Git-Repository):

```bash
openclaw plugins install ./extensions/matrix
```

Wenn du während der Konfiguration/Ersteinrichtung Matrix auswählst und ein Git-Checkout erkannt wird,
wird dir von OpenClaw automatisch der lokale Installationspfad vorgeschlagen.

Details: [Plugins](/de/plugin)

<div id="setup">
  ## Einrichtung
</div>

1. Installiere das Matrix-Plugin:
   * Über npm: `openclaw plugins install @openclaw/matrix`
   * Aus einem lokalen Checkout: `openclaw plugins install ./extensions/matrix`
2. Erstelle ein Matrix-Konto auf einem Homeserver:
   * Sieh dir Hosting-Optionen unter [https://matrix.org/ecosystem/hosting/](https://matrix.org/ecosystem/hosting/) an
   * Oder hoste ihn selbst.
3. Hole ein Access-Token für das Bot-Konto:

   * Verwende die Matrix-Login-API mit `curl` auf deinem Homeserver:

   ```bash
   curl --request POST \
     --url https://matrix.example.org/_matrix/client/v3/login \
     --header 'Content-Type: application/json' \
     --data '{
     "type": "m.login.password",
     "identifier": {
       "type": "m.id.user",
       "user": "your-user-name"
     },
     "password": "your-password"
   }'
   ```

   * Ersetze `matrix.example.org` durch die URL deines Homeservers.
   * Oder setze `channels.matrix.userId` + `channels.matrix.password`: OpenClaw ruft denselben
     Login-Endpunkt auf, speichert das Access-Token in `~/.openclaw/credentials/matrix/credentials.json`
     und verwendet es beim nächsten Start erneut.
4. Konfiguriere Zugangsdaten:
   * Umgebungsvariablen: `MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN` (oder `MATRIX_USER_ID` + `MATRIX_PASSWORD`)
   * Oder Konfiguration: `channels.matrix.*`
   * Wenn beides gesetzt ist, hat die Konfiguration Vorrang.
   * Mit Access-Token: Die User-ID wird automatisch über `/whoami` abgerufen.
   * Wenn gesetzt, sollte `channels.matrix.userId` die vollständige Matrix-ID sein (Beispiel: `@bot:example.org`).
5. Starte das Gateway neu (oder schließe das Onboarding ab).
6. Starte eine Direktnachricht (DM) mit dem Bot oder lade ihn von einem beliebigen Matrix-Client in einen Raum ein
   (Element, Beeper usw.; siehe https://matrix.org/ecosystem/clients/). Beeper erfordert E2EE,
   daher setze `channels.matrix.encryption: true` und verifiziere das Gerät.

Minimale Konfiguration (Access-Token, User-ID wird automatisch abgerufen):

```json5
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      dm: { policy: "pairing" }
    }
  }
}
```

E2EE-Konfiguration (mit aktivierter Ende-zu-Ende-Verschlüsselung):

```json5
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      encryption: true,
      dm: { policy: "pairing" }
    }
  }
}
```

<div id="encryption-e2ee">
  ## Verschlüsselung (E2EE)
</div>

Ende-zu-Ende-Verschlüsselung wird über das Rust-Crypto-SDK **unterstützt**.

Aktiviere sie mit `channels.matrix.encryption: true`:

* Wenn das Crypto-Modul geladen wird, werden verschlüsselte Räume automatisch entschlüsselt.
* Ausgehende Medien werden beim Senden in verschlüsselte Räume verschlüsselt.
* Beim ersten Verbindungsaufbau fordert OpenClaw eine Geräteverifizierung von deinen anderen Sitzungen an.
* Verifiziere das Gerät in einem anderen Matrix-Client (Element usw.), um die Schlüsselweitergabe zu aktivieren.
* Wenn das Crypto-Modul nicht geladen werden kann, wird E2EE deaktiviert und verschlüsselte Räume werden nicht entschlüsselt;
  OpenClaw protokolliert eine Warnung.
* Wenn Fehler zu einem fehlenden Crypto-Modul angezeigt werden (zum Beispiel `@matrix-org/matrix-sdk-crypto-nodejs-*`),
  erlaube Build-Skripte für `@matrix-org/matrix-sdk-crypto-nodejs` und führe
  `pnpm rebuild @matrix-org/matrix-sdk-crypto-nodejs` aus oder lade die Binärdatei mit
  `node node_modules/@matrix-org/matrix-sdk-crypto-nodejs/download-lib.js` herunter.

Der Crypto-Status wird pro Konto + Access Token in
`~/.openclaw/matrix/accounts/<account>/<homeserver>__<user>/<token-hash>/crypto/`
(SQLite-Datenbank) gespeichert. Der Sync-Status liegt daneben in `bot-storage.json`.
Wenn sich das Access Token (Gerät) ändert, wird ein neuer Store angelegt und der Bot
muss für verschlüsselte Räume erneut verifiziert werden.

**Geräteverifizierung:**
Wenn E2EE aktiviert ist, fordert der Bot beim Start eine Verifizierung von deinen anderen Sitzungen an.
Öffne Element (oder einen anderen Client) und bestätige die Verifizierungsanfrage, um Vertrauen herzustellen.
Nach der Verifizierung kann der Bot Nachrichten in verschlüsselten Räumen entschlüsseln.

<div id="routing-model">
  ## Routing-Modell
</div>

* Antworten werden immer an Matrix zurückgesendet.
* DMs nutzen gemeinsam die Hauptsitzung des agents; Räume werden eigenen Gruppensitzungen zugeordnet.

<div id="access-control-dms">
  ## Zugriffskontrolle (DMs)
</div>

* Standardwert: `channels.matrix.dm.policy = "pairing"`. Unbekannte Absender erhalten einen Pairing-Code (Kopplungscode).
* Freigabe über:
  * `openclaw pairing list matrix`
  * `openclaw pairing approve matrix <CODE>`
* Öffentliche DMs: `channels.matrix.dm.policy="open"` (Modus `open`, der uneingeschränkte Nachrichtenannahme von beliebigen Nutzern erlaubt) plus `channels.matrix.dm.allowFrom=["*"]`.
* `channels.matrix.dm.allowFrom` akzeptiert Benutzer-IDs oder Anzeigenamen. Der Assistent löst Anzeigenamen in Benutzer-IDs auf, wenn die Verzeichnissuche verfügbar ist.

<div id="rooms-groups">
  ## Räume (Gruppen)
</div>

* Standard: `channels.matrix.groupPolicy = "allowlist"` (nur bei Erwähnung). Verwende `channels.defaults.groupPolicy`, um den Standardwert zu überschreiben, wenn er nicht gesetzt ist.
* Räume mit `channels.matrix.groups` zur Allowlist hinzufügen (Room-IDs, Aliasse oder Namen):

```json5
{
  channels: {
    matrix: {
      groupPolicy: "allowlist",
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true }
      },
      groupAllowFrom: ["@owner:example.org"]
    }
  }
}
```

* `requireMention: false` aktiviert automatische Antworten in diesem Raum.
* `groups."*"` kann Standardwerte für die Erwähnungs-Pflicht (Mention-Gating) über Räume hinweg setzen.
* `groupAllowFrom` beschränkt, welche Absender den Bot in Räumen auslösen können (optional).
* Raumspezifische `users`-Allowlists können Absender innerhalb eines bestimmten Raums weiter einschränken.
* Der Konfigurationsassistent fragt nach Raum-Allowlists (Raum-IDs, Aliasse oder Namen) und löst Namen auf, wenn möglich.
* Beim Start löst OpenClaw Raum-/Benutzernamen in Allowlists zu IDs auf und protokolliert die Zuordnung; nicht auflösbare Einträge bleiben wie eingegeben.
* Einladungen werden standardmäßig automatisch angenommen; steuere das mit `channels.matrix.autoJoin` und `channels.matrix.autoJoinAllowlist`.
* Um **keine Räume** zu erlauben, setze `channels.matrix.groupPolicy: "disabled"` (oder lass die Allowlist leer).
* Veralteter Schlüssel: `channels.matrix.rooms` (gleiche Struktur wie `groups`).

<div id="threads">
  ## Threads
</div>

* Thread-Antworten werden unterstützt.
* `channels.matrix.threadReplies` steuert, ob Antworten in Threads bleiben:
  * `off`, `inbound` (Standard), `always`
* `channels.matrix.replyToMode` steuert die Reply-to-Metadaten, wenn nicht in einem Thread geantwortet wird:
  * `off` (Standard), `first`, `all`

<div id="capabilities">
  ## Funktionen
</div>

| Funktion | Status |
|---------|--------|
| Direktnachrichten | ✅ Unterstützt |
| Räume | ✅ Unterstützt |
| Threads | ✅ Unterstützt |
| Medien | ✅ Unterstützt |
| E2EE | ✅ Unterstützt (Krypto-Modul erforderlich) |
| Reaktionen | ✅ Unterstützt (send/read über Tools) |
| Umfragen | ✅ Senden unterstützt; eingehende Umfragestarts werden in Text konvertiert (Antworten und Beendigungen werden ignoriert) |
| Standort | ✅ Unterstützt (Geo-URI; Höhe wird ignoriert) |
| Native Befehle | ✅ Unterstützt |

<div id="configuration-reference-matrix">
  ## Konfigurationsreferenz (Matrix)
</div>

Vollständige Konfiguration: [Konfiguration](/de/gateway/configuration)

Anbieteroptionen:

* `channels.matrix.enabled`: Kanalstart aktivieren/deaktivieren.
* `channels.matrix.homeserver`: Homeserver-URL.
* `channels.matrix.userId`: Matrix-Benutzer-ID (optional mit Access-Token).
* `channels.matrix.accessToken`: Access-Token.
* `channels.matrix.password`: Passwort für Login (Token wird gespeichert).
* `channels.matrix.deviceName`: Anzeigename des Geräts.
* `channels.matrix.encryption`: E2EE aktivieren (Standard: false).
* `channels.matrix.initialSyncLimit`: Limit für die initiale Synchronisation.
* `channels.matrix.threadReplies`: `off | inbound | always` (Standard: inbound).
* `channels.matrix.textChunkLimit`: Größe ausgehender Textblöcke (Zeichen).
* `channels.matrix.chunkMode`: `length` (Standard) oder `newline`, um an Leerzeilen (Absatzgrenzen) aufzuteilen, bevor nach Länge segmentiert wird.
* `channels.matrix.dm.policy`: `pairing | allowlist | open | disabled` (Standard: pairing).
* `channels.matrix.dm.allowFrom`: DM-Allowlist (Benutzer-IDs oder Anzeigenamen). `open` erfordert `"*"`. Der Einrichtungsassistent löst Namen nach Möglichkeit in IDs auf.
* `channels.matrix.groupPolicy`: `allowlist | open | disabled` (Standard: allowlist).
* `channels.matrix.groupAllowFrom`: Allowlist für Absender von Gruppennachrichten.
* `channels.matrix.allowlistOnly`: Allowlist-Regeln für DMs und Räume erzwingen.
* `channels.matrix.groups`: Gruppen-Allowlist plus Mapping für raumspezifische Einstellungen.
* `channels.matrix.rooms`: Legacy-Gruppen-Allowlist/-Konfiguration.
* `channels.matrix.replyToMode`: Reply-to-Modus für Threads/Tags.
* `channels.matrix.mediaMaxMb`: Limit für eingehende/ausgehende Medien (MB).
* `channels.matrix.autoJoin`: Behandlung von Einladungen (`always | allowlist | off`, Standard: always).
* `channels.matrix.autoJoinAllowlist`: Erlaubte Raum-IDs/Aliasse für Auto-Join.
* `channels.matrix.actions`: Tool-Gating pro Aktion (reactions/messages/pins/memberInfo/channelInfo).