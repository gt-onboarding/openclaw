---
title: Gruppennachrichten
summary: "Verhalten und Konfiguration für die Verarbeitung von WhatsApp-Gruppennachrichten (mentionPatterns werden oberflächenübergreifend gemeinsam genutzt)"
read_when:
  - Beim Ändern von Regeln für Gruppennachrichten oder Erwähnungen
---

<div id="group-messages-whatsapp-web-channel">
  # Gruppennachrichten (WhatsApp-Web-Kanal)
</div>

Ziel: Clawd soll in WhatsApp-Gruppen mitlaufen, nur aufwachen, wenn er erwähnt wird, und diesen Thread getrennt von der persönlichen DM-Sitzung halten.

Hinweis: `agents.list[].groupChat.mentionPatterns` wird jetzt auch von Telegram/Discord/Slack/iMessage verwendet; dieses Dokument konzentriert sich auf das WhatsApp-spezifische Verhalten. Für Multi-Agenten-Setups setzt du `agents.list[].groupChat.mentionPatterns` pro Agent (oder verwendest `messages.groupChat.mentionPatterns` als globalen Fallback).

<div id="whats-implemented-2025-12-03">
  ## Was bereits implementiert ist (2025-12-03)
</div>

- Aktivierungsmodi: `mention` (Standard) oder `always`. `mention` erfordert einen Ping (echte WhatsApp-@-Mentions über `mentionedJids`, Regex-Muster oder die E.164 des Bots irgendwo im Text). `always` weckt den agent bei jeder Nachricht, aber er sollte nur antworten, wenn er einen sinnvollen Mehrwert liefern kann; andernfalls gibt er das stille Token `NO_REPLY` zurück. Standardwerte können in der Konfiguration gesetzt werden (`channels.whatsapp.groups`) und pro Gruppe via `/activation` überschrieben werden. Wenn `channels.whatsapp.groups` gesetzt ist, fungiert es auch als Gruppen-Allowlist (füge `"*"` hinzu, um alle zuzulassen).
- Gruppenrichtlinie: `channels.whatsapp.groupPolicy` steuert, ob Gruppennachrichten akzeptiert werden (`open|disabled|allowlist`). `allowlist` verwendet `channels.whatsapp.groupAllowFrom` (Fallback: explizites `channels.whatsapp.allowFrom`). Der Standard ist `allowlist` (blockiert, bis du Absender hinzufügst).
- Sitzungen pro Gruppe: Sitzungsschlüssel sehen aus wie `agent:<agentId>:whatsapp:group:<jid>`, sodass Befehle wie `/verbose on` oder `/think high` (als eigenständige Nachrichten gesendet) auf diese Gruppe beschränkt sind; der persönliche DM-Status bleibt unberührt. Herzschlag-Ereignisse werden für Gruppenthreads übersprungen.
- Kontexteinspeisung: **nur ausstehende** Gruppennachrichten (standardmäßig 50), die *keine* Ausführung ausgelöst haben, werden mit dem Präfix `[Chat messages since your last reply - for context]` versehen, wobei die auslösende Zeile unter `[Current message - respond to this]` steht. Nachrichten, die bereits in der Sitzung sind, werden nicht erneut eingespeist.
- Sichtbarmachen des Absenders: Jede Gruppencharge endet jetzt mit `[from: Sender Name (+E164)]`, damit Pi weiß, wer spricht.
- Flüchtig/Einmalansicht: Wir lösen diese auf, bevor wir Text/Mentions extrahieren, sodass Pings darin weiterhin auslösen.
- Gruppen-Systemprompt: Im ersten Durchlauf einer Gruppensitzung (und immer dann, wenn `/activation` den Modus ändert) fügen wir einen kurzen Text in den Systemprompt ein, etwa `You are replying inside the WhatsApp group "<subject>". Group members: Alice (+44...), Bob (+43...), … Activation: trigger-only … Address the specific sender noted in the message context.` Wenn Metadaten nicht verfügbar sind, teilen wir dem agent trotzdem mit, dass es sich um einen Gruppenchat handelt.

<div id="config-example-whatsapp">
  ## Konfigurationsbeispiel (WhatsApp)
</div>

Füge einen `groupChat`-Block zu `~/.openclaw/openclaw.json` hinzu, damit Display-Name-Pings auch dann funktionieren, wenn WhatsApp das sichtbare `@` im Nachrichtentext entfernt:

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true }
      }
    }
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          historyLimit: 50,
          mentionPatterns: [
            "@?openclaw",
            "\\+?15555550123"
          ]
        }
      }
    ]
  }
}
```

Hinweise:

* Die Regex-Ausdrücke sind nicht Groß-/Kleinschreibungs-sensitiv; sie erfassen sowohl einen Ping über den Anzeigenamen wie `@openclaw` als auch die reine Nummer mit oder ohne `+`/Leerzeichen.
* WhatsApp sendet weiterhin kanonische Erwähnungen über `mentionedJids`, wenn jemand auf den Kontakt tippt, daher wird das Fallback über die Nummer nur selten benötigt, ist aber ein nützliches Sicherheitsnetz.


<div id="activation-command-owner-only">
  ### Aktivierungsbefehl (nur für den Owner)
</div>

Verwende den Gruppenchat-Befehl:

- `/activation mention`
- `/activation always`

Nur die Owner-Nummer (aus `channels.whatsapp.allowFrom` oder die eigene E.164 des Bots, falls nicht gesetzt) kann dies ändern. Sende `/status` als einzelne Nachricht in der Gruppe, um den aktuellen Aktivierungsmodus anzuzeigen.

<div id="how-to-use">
  ## Verwendung
</div>

1) Füge dein WhatsApp-Konto (das, auf dem OpenClaw läuft) zur Gruppe hinzu.
2) Schreibe `@openclaw …` (oder füge die Nummer hinzu). Nur Absender in der Allowlist können es triggern, es sei denn, du setzt `groupPolicy: "open"` (d. h. Nachrichten von allen Nutzern werden ohne Einschränkung akzeptiert).
3) Der Agent-Prompt enthält den aktuellen Gruppenkontext sowie den nachgestellten `[from: …]`-Marker, damit die Antwort an die richtige Person adressiert werden kann.
4) Direktiven auf Sitzungsebene (`/verbose on`, `/think high`, `/new` oder `/reset`, `/compact`) gelten nur für die Sitzung dieser Gruppe; sende sie als eigenständige Nachrichten, damit sie erfasst werden. Deine persönliche DM‑Sitzung bleibt davon unabhängig.

<div id="testing-verification">
  ## Tests / Verifizierung
</div>

- Manuelle Smoke-Tests:
  - Sende ein `@openclaw`-Ping in die Gruppe und bestätige eine Antwort, die den Namen des Absenders enthält.
  - Sende ein zweites Ping und überprüfe, dass der Verlaufsblock enthalten ist und dann im nächsten Durchlauf geleert wird.
- Prüfe die Gateway-Logs (führe mit `--verbose` aus), um `inbound web message`-Einträge zu sehen, die `from: <groupJid>` und das `[from: …]`-Suffix anzeigen.

<div id="known-considerations">
  ## Bekannte Besonderheiten
</div>

- Herzschläge werden für Gruppen absichtlich übersprungen, um störende Broadcasts zu vermeiden.
- Die Echo-Unterdrückung verwendet die kombinierte Batch-Zeichenkette; wenn du denselben Text zweimal ohne Erwähnungen sendest, erhält nur der erste eine Antwort.
- Einträge im Sitzungsspeicher erscheinen als `agent:&lt;agentId&gt;:whatsapp:group:&lt;jid&gt;` im Sitzungsspeicher (standardmäßig `~/.openclaw/agents/&lt;agentId&gt;/sessions/sessions.json`); ein fehlender Eintrag bedeutet lediglich, dass die Gruppe noch keine Ausführung ausgelöst hat.
- Schreibindikatoren in Gruppen folgen `agents.defaults.typingMode` (Standard: `message`, wenn du nicht erwähnt wirst).