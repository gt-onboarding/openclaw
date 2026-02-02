---
title: Signal
summary: "Signal-Unterstützung über signal-cli (JSON-RPC + SSE), Einrichtung und Modell für Telefonnummern"
read_when:
  - Einrichten der Signal-Unterstützung
  - Debugging von Sende- und Empfangsvorgängen in Signal
---

<div id="signal-signal-cli">
  # Signal (signal-cli)
</div>

Status: externe CLI-Integration. Das Gateway kommuniziert über HTTP mittels JSON-RPC und SSE mit `signal-cli`.

<div id="quick-setup-beginner">
  ## Schnellstart (Einsteiger)
</div>

1. Verwende eine **separate Signal-Nummer** für den Bot (empfohlen).
2. Installiere `signal-cli` (Java erforderlich).
3. Verknüpfe das Botgerät und starte den Daemon:
   * `signal-cli link -n "OpenClaw"`
4. Konfiguriere OpenClaw und starte das Gateway.

Minimale Konfiguration:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"]
    }
  }
}
```

<div id="what-it-is">
  ## Was es ist
</div>

* Signal-Kanal über `signal-cli` (nicht eingebettetes libsignal).
* Deterministisches Routing: Antworten gehen immer an Signal zurück.
* DMs nutzen die Hauptsitzung des Agents; Gruppen sind isoliert (`agent:<agentId>:signal:group:<groupId>`).

<div id="config-writes">
  ## Config-Schreibvorgänge
</div>

Standardmäßig ist es Signal erlaubt, Konfigurationsänderungen zu schreiben, die durch `/config set|unset` ausgelöst werden (erfordert `commands.config: true`).

Deaktiviere dies mit:

```json5
{
  channels: { signal: { configWrites: false } }
}
```

<div id="the-number-model-important">
  ## Das Nummernmodell (wichtig)
</div>

* Das Gateway verbindet sich mit einem **Signal-Gerät** (dem `signal-cli`-Konto).
* Wenn du den Bot mit **deinem persönlichen Signal-Konto** nutzt, ignoriert er deine eigenen Nachrichten (Schutz vor Endlosschleifen).
* Für das Szenario „Ich schreibe dem Bot und er antwortet“ verwende eine **separate Bot-Nummer**.

<div id="setup-fast-path">
  ## Einrichtung (Schnellstart)
</div>

1. Installiere `signal-cli` (Java erforderlich).
2. Verknüpfe ein Bot-Konto:
   * `signal-cli link -n "OpenClaw"` und scanne dann den QR-Code in Signal.
3. Konfiguriere Signal und starte das Gateway.

Beispiel:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"]
    }
  }
}
```

Unterstützung mehrerer Accounts: Verwende `channels.signal.accounts` mit kontospezifischer Konfiguration und optionalem `name`. Siehe [`gateway/configuration`](/de/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) für das gemeinsame Konfigurationsmuster.

<div id="external-daemon-mode-httpurl">
  ## Externer Daemon-Modus (httpUrl)
</div>

Wenn du `signal-cli` selbst verwalten möchtest (z.B. wegen langsamer JVM-Kaltstarts, Container-Initialisierung oder geteilten CPUs), starte den Daemon separat und konfiguriere OpenClaw so, dass es darauf zugreift:

```json5
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false
    }
  }
}
```

Dies überspringt Auto-Spawn und die Startwartezeit innerhalb von OpenClaw. Bei langsamen Starts beim Auto-Spawn setze `channels.signal.startupTimeoutMs`.

<div id="access-control-dms-groups">
  ## Zugriffskontrolle (DMs + Gruppen)
</div>

DMs:

* Standard: `channels.signal.dmPolicy = "pairing"`.
* Unbekannte Absender erhalten einen Kopplungscode; Nachrichten werden ignoriert, bis sie genehmigt werden (Codes verfallen nach 1 Stunde).
* Genehmigung über:
  * `openclaw pairing list signal`
  * `openclaw pairing approve signal <CODE>`
* Kopplung ist der standardmäßige Token-Austausch für Signal-DMs. Details: [Pairing](/de/start/pairing)
* Absender, die nur über eine UUID (aus `sourceUuid`) identifiziert werden, werden als `uuid:<id>` in `channels.signal.allowFrom` gespeichert.

Gruppen:

* `channels.signal.groupPolicy = open | allowlist | disabled`.
* `channels.signal.groupAllowFrom` steuert, wer in Gruppen Trigger ausführen kann, wenn `allowlist` gesetzt ist.

<div id="how-it-works-behavior">
  ## Funktionsweise (Verhalten)
</div>

* `signal-cli` läuft als Daemon-Prozess; das Gateway liest Ereignisse über SSE.
* Eingehende Nachrichten werden in den gemeinsamen Channel-Envelope normalisiert.
* Antworten werden immer zurück an dieselbe Nummer oder Gruppe geroutet.

<div id="media-limits">
  ## Medien + Limits
</div>

* Ausgehender Text wird entsprechend `channels.signal.textChunkLimit` (Standardwert 4000) in Blöcke aufgeteilt.
* Optionale Zeilenumbruch-Segmentierung: Setze `channels.signal.chunkMode="newline"`, um vor der Längen-Segmentierung an Leerzeilen (Absatzgrenzen) zu teilen.
* Anhänge werden unterstützt (base64 von `signal-cli` abgerufen).
* Standardgrenze für Medien: `channels.signal.mediaMaxMb` (Standardwert 8).
* Verwende `channels.signal.ignoreAttachments`, um das Herunterladen von Medien zu überspringen.
* Der Verlaufskontext für Gruppenchats verwendet `channels.signal.historyLimit` (oder `channels.signal.accounts.*.historyLimit`) und fällt zurück auf `messages.groupChat.historyLimit`. Setze `0`, um zu deaktivieren (Standardwert 50).

<div id="typing-read-receipts">
  ## Schreibindikatoren + Lesebestätigungen
</div>

* **Schreibindikatoren**: OpenClaw sendet Tipp-Signale über `signal-cli sendTyping` und aktualisiert sie fortlaufend, solange eine Antwort ausgegeben wird.
* **Lesebestätigungen**: Wenn `channels.signal.sendReadReceipts` auf true gesetzt ist, leitet OpenClaw Lesebestätigungen für zulässige DMs weiter.
* `signal-cli` stellt keine Lesebestätigungen für Gruppen zur Verfügung.

<div id="reactions-message-tool">
  ## Reaktionen (message-Tool)
</div>

* Verwende `message action=react` mit `channel=signal`.
* Ziele: Absender in E.164-Form oder UUID (verwende `uuid:<id>` aus der Kopplungsausgabe; eine reine UUID funktioniert ebenfalls).
* `messageId` ist der Signal-Zeitstempel der Nachricht, auf die du reagierst.
* Gruppenreaktionen erfordern `targetAuthor` oder `targetAuthorUuid`.

Beispiele:

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

Konfiguration:

* `channels.signal.actions.reactions`: Reaktionsaktionen aktivieren/deaktivieren (Standard: true).
* `channels.signal.reactionLevel`: `off | ack | minimal | extensive`.
  * `off`/`ack` deaktiviert agent-Reaktionen (Message-Tool `react` führt zu einem Fehler).
  * `minimal`/`extensive` aktiviert agent-Reaktionen und legt den Hinweisgrad fest.
* Kontospezifische Overrides: `channels.signal.accounts.<id>.actions.reactions`, `channels.signal.accounts.<id>.reactionLevel`.

<div id="delivery-targets-clicron">
  ## Zustellziele (CLI/cron)
</div>

* DMs: `signal:+15551234567` (oder im reinen E.164-Format).
* UUID-DMs: `uuid:<id>` (oder reine UUID).
* Gruppen: `signal:group:<groupId>`.
* Benutzernamen: `username:<name>` (falls von deinem Signal-Konto unterstützt).

<div id="configuration-reference-signal">
  ## Konfigurationsreferenz (Signal)
</div>

Vollständige Konfiguration: [Konfiguration](/de/gateway/configuration)

Anbieter-Optionen:

* `channels.signal.enabled`: Aktiviert/deaktiviert den Kanalstart.
* `channels.signal.account`: E.164 für das Bot-Konto.
* `channels.signal.cliPath`: Pfad zu `signal-cli`.
* `channels.signal.httpUrl`: vollständige Daemon-URL (überschreibt Host/Port).
* `channels.signal.httpHost`, `channels.signal.httpPort`: Daemon-Bind-Adresse (Standard 127.0.0.1:8080).
* `channels.signal.autoStart`: automatisches Starten des Daemons (Standard true, wenn `httpUrl` nicht gesetzt ist).
* `channels.signal.startupTimeoutMs`: Start-Timeout in ms (Obergrenze 120000).
* `channels.signal.receiveMode`: `on-start | manual`.
* `channels.signal.ignoreAttachments`: Überspringt das Herunterladen von Attachments.
* `channels.signal.ignoreStories`: Ignoriert Stories vom Daemon.
* `channels.signal.sendReadReceipts`: Leitet Lesebestätigungen weiter.
* `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled` (Standard: pairing).
* `channels.signal.allowFrom`: DM-Allowlist (E.164 oder `uuid:<id>`). `open` erfordert `"*"`. Signal hat keine Benutzernamen; verwende Telefon-/UUID-IDs.
* `channels.signal.groupPolicy`: `open | allowlist | disabled` (Standard: allowlist).
* `channels.signal.groupAllowFrom`: Allowlist für Gruppensender.
* `channels.signal.historyLimit`: Maximale Anzahl von Gruppennachrichten, die als Kontext einbezogen werden (0 deaktiviert).
* `channels.signal.dmHistoryLimit`: DM-History-Limit in Benutzer-Turns. Benutzerspezifische Overrides: `channels.signal.dms["<phone_or_uuid>"].historyLimit`.
* `channels.signal.textChunkLimit`: Größe ausgehender Chunks (Zeichen).
* `channels.signal.chunkMode`: `length` (Standard) oder `newline`, um an Leerzeilen (Abschnittsgrenzen) zu teilen, bevor nach Länge in Chunks aufgeteilt wird.
* `channels.signal.mediaMaxMb`: Obergrenze für eingehende/ausgehende Medien (MB).

Zugehörige globale Optionen:

* `agents.list[].groupChat.mentionPatterns` (Signal unterstützt keine nativen Erwähnungen).
* `messages.groupChat.mentionPatterns` (globaler Fallback).
* `messages.responsePrefix`.