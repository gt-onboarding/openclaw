---
title: Refactoring des ausgehenden Sitzungs-Mirroring (Issue #1520)
description: Erfassung von Notizen, Entscheidungen, Tests und offenen Punkten zum Refactoring des ausgehenden Sitzungs-Mirroring.
---

<div id="outbound-session-mirroring-refactor-issue-1520">
  # Refactoring der Spiegelung ausgehender Sitzungen (Issue #1520)
</div>

<div id="status">
  ## Status
</div>

- In Bearbeitung.
- Core- und Plugin-Channel-Routing für ausgehendes Mirroring wurde aktualisiert.
- Gateway send ermittelt nun die Ziel-Sitzung, wenn sessionKey nicht angegeben wird.

<div id="context">
  ## Kontext
</div>

Ausgehende Send-Vorgänge wurden in die *aktuelle* Agent-Sitzung (Tool-Sitzungsschlüssel) gespiegelt statt in die Sitzung des Zielkanals. Eingehendes Routing verwendet Kanal-/Peer-Sitzungsschlüssel, daher landeten ausgehende Antworten in der falschen Sitzung und für Erstkontakte fehlten häufig Sitzungseinträge.

<div id="goals">
  ## Ziele
</div>

- Ausgehende Nachrichten im Sitzungsschlüssel des Zielkanals spiegeln.
- Sitzungseinträge für ausgehende Nachrichten erstellen, wenn sie fehlen.
- Thread-/Themenscope mit eingehenden Sitzungsschlüsseln abgeglichen halten.
- Zentrale Kanäle plus mitgelieferte Erweiterungen abdecken.

<div id="implementation-summary">
  ## Implementierungszusammenfassung
</div>

- Neuer Helper für ausgehendes Sitzungsrouting:
  - `src/infra/outbound/outbound-session.ts`
  - `resolveOutboundSessionRoute` baut den Ziel-`sessionKey` mithilfe von `buildAgentSessionKey` (dmScope + identityLinks).
  - `ensureOutboundSessionEntry` schreibt einen minimalen `MsgContext` über `recordSessionMetaFromInbound`.
- `runMessageAction` (Senden) leitet den Ziel-`sessionKey` ab und übergibt ihn zur Spiegelung an `executeSendAction`.
- `message-tool` spiegelt nicht mehr direkt, sondern löst nur noch `agentId` aus dem aktuellen `sessionKey` auf.
- Der Plugin-Sendepfad spiegelt über `appendAssistantMessageToSessionTranscript` mithilfe des abgeleiteten `sessionKey`.
- Gateway-Senden leitet einen Ziel-`sessionKey` ab, wenn keiner angegeben ist (Standard-Agent), und stellt sicher, dass ein Sitzungseintrag existiert.

<div id="threadtopic-handling">
  ## Thread-/Themenverarbeitung
</div>

- Slack: replyTo/threadId -> `resolveThreadSessionKeys` (Suffix).
- Discord: threadId/replyTo -> `resolveThreadSessionKeys` mit `useSuffix=false`, um eingehende Nachrichten abzugleichen (Thread-Channel-ID scoped die Sitzung bereits).
- Telegram: Themen-IDs werden über `buildTelegramGroupPeerId` auf `chatId:topic:<id>` gemappt.

<div id="extensions-covered">
  ## Abgedeckte Erweiterungen
</div>

- Matrix, MS Teams, Mattermost, BlueBubbles, Nextcloud Talk, Zalo, Zalo Personal, Nostr, Tlon.
- Hinweise:
  - Mattermost-Ziele entfernen jetzt `@` für das Routing von DM-Sitzungsschlüsseln.
  - Zalo Personal verwendet den DM-Peer-Typ für 1:1-Ziele (wird nur als Gruppe behandelt, wenn `group:` vorhanden ist).
  - BlueBubbles-Gruppenziele entfernen `chat_*`-Präfixe, um eingehende Sitzungsschlüssel abzugleichen.
  - Automatisches Thread-Mirroring in Slack vergleicht Channel-IDs ohne Beachtung der Groß-/Kleinschreibung.
  - Gateway send wandelt übergebene Sitzungsschlüssel vor dem Mirroring in Kleinbuchstaben um.

<div id="decisions">
  ## Entscheidungen
</div>

- **Ableitung der Gateway-Sitzung beim Senden**: Wenn `sessionKey` angegeben ist, verwende ihn. Wenn er fehlt, leite einen sessionKey aus Ziel + Standard-agent ab und spiegele dorthin.
- **Erstellung von Sitzungseinträgen**: Verwende immer `recordSessionMetaFromInbound` mit `Provider/From/To/ChatType/AccountId/Originating*`, ausgerichtet auf eingehende Formate.
- **Zielnormalisierung**: Ausgehendes Routing verwendet aufgelöste Ziele (nach `resolveChannelTarget`), wenn verfügbar.
- **Schreibweise von Session-Keys**: Normalisiere Session-Keys beim Schreiben und während Migrationen auf Kleinbuchstaben.

<div id="tests-addedupdated">
  ## Tests hinzugefügt/aktualisiert
</div>

- `src/infra/outbound/outbound-session.test.ts`
  - Slack-Thread-Sitzungsschlüssel.
  - Telegram-Topic-Sitzungsschlüssel.
  - dmScope identityLinks mit Discord.
- `src/agents/tools/message-tool.test.ts`
  - Leitet agentId aus dem Sitzungsschlüssel ab (kein sessionKey durchgereicht).
- `src/gateway/server-methods/send.test.ts`
  - Leitet Sitzungsschlüssel ab, wenn weggelassen, und erstellt einen Sitzungseintrag.

<div id="open-items-follow-ups">
  ## Offene Punkte / Follow-ups
</div>

- Das Voice-Call-Plugin verwendet benutzerdefinierte `voice:<phone>` Sitzungs-Keys. Das Mapping für ausgehende Nachrichten ist hier nicht standardisiert; wenn message-tool Voice-Call-Sendvorgänge unterstützen soll, ein explizites Mapping hinzufügen.
- Prüfen, ob externe Plugins nicht standardisierte `From/To`-Formate verwenden, die über den mitgelieferten Satz hinausgehen.

<div id="files-touched">
  ## Betroffene Dateien
</div>

- `src/infra/outbound/outbound-session.ts`
- `src/infra/outbound/outbound-send-service.ts`
- `src/infra/outbound/message-action-runner.ts`
- `src/agents/tools/message-tool.ts`
- `src/gateway/server-methods/send.ts`
- Tests in:
  - `src/infra/outbound/outbound-session.test.ts`
  - `src/agents/tools/message-tool.test.ts`
  - `src/gateway/server-methods/send.test.ts`