---
title: Slash-Befehle
summary: "Slash-Befehle: Text vs. nativ, Konfiguration und unterstützte Befehle"
read_when:
  - Verwendung oder Konfiguration von Chatbefehlen
  - Fehlerbehebung bei Befehlsrouting oder Berechtigungen
---

<div id="slash-commands">
  # Slash-Befehle
</div>

Befehle werden vom Gateway verarbeitet. Die meisten Befehle müssen als **eigenständige** Nachricht gesendet werden, die mit `/` beginnt.
Der nur für den Host verfügbare Bash-Chatbefehl verwendet `! <cmd>` (mit `/bash <cmd>` als Alias).

Es gibt zwei verwandte Systeme:

* **Befehle**: eigenständige `/...`-Nachrichten.
* **Direktiven**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/exec`, `/model`, `/queue`.
  * Direktiven werden aus der Nachricht entfernt, bevor das Modell sie sieht.
  * In normalen Chat-Nachrichten (nicht nur aus Direktiven bestehend) werden sie als „Inline-Hinweise“ behandelt und übernehmen **keine** Sitzungseinstellungen.
  * In Nachrichten, die nur aus Direktiven bestehen (die Nachricht enthält nur Direktiven), werden sie in der Sitzung übernommen und mit einer Bestätigung beantwortet.
  * Direktiven werden nur für **autorisierte Sender** angewendet (Kanal-Allowlists/Kopplung plus `commands.useAccessGroups`).
    Nicht autorisierte Sender sehen Direktiven als normalen Text.

Es gibt auch einige **Inline-Kurzbefehle** (nur für auf der Allowlist stehende/autorisierte Sender): `/help`, `/commands`, `/status`, `/whoami` (`/id`).
Sie werden sofort ausgeführt, bevor das Modell die Nachricht sieht, und der verbleibende Text durchläuft den normalen Ablauf.

<div id="config">
  ## Konfiguration
</div>

```json5
{
  commands: {
    native: "auto",
    nativeSkills: "auto",
    text: true,
    bash: false,
    bashForegroundMs: 2000,
    config: false,
    debug: false,
    restart: false,
    useAccessGroups: true
  }
}
```

* `commands.text` (Standardwert `true`) aktiviert das Parsen von `/...` in Chat-Nachrichten.
  * Auf Oberflächen ohne native Commands (WhatsApp/WebChat/Signal/iMessage/Google Chat/MS Teams) funktionieren Text-Commands weiterhin, selbst wenn du dies auf `false` setzt.
* `commands.native` (Standardwert `"auto"`) registriert native Commands.
  * Auto: aktiviert für Discord/Telegram; deaktiviert für Slack (bis du Slash-Commands hinzufügst); ignoriert für Anbieter ohne native Unterstützung.
  * Setze `channels.discord.commands.native`, `channels.telegram.commands.native` oder `channels.slack.commands.native`, um dies pro Anbieter zu überschreiben (bool oder `"auto"`).
  * `false` löscht beim Start zuvor registrierte Commands auf Discord/Telegram. Slack-Commands werden in der Slack-App verwaltet und nicht automatisch entfernt.
* `commands.nativeSkills` (Standardwert `"auto"`) registriert **Skill**-Commands nativ, wenn unterstützt.
  * Auto: aktiviert für Discord/Telegram; deaktiviert für Slack (Slack erfordert das Anlegen eines Slash-Commands pro Skill).
  * Setze `channels.discord.commands.nativeSkills`, `channels.telegram.commands.nativeSkills` oder `channels.slack.commands.nativeSkills`, um dies pro Anbieter zu überschreiben (bool oder `"auto"`).
* `commands.bash` (Standardwert `false`) aktiviert `! <cmd>` zum Ausführen von Shell-Befehlen auf dem Host (`/bash <cmd>` ist ein Alias; erfordert `tools.elevated`-Allowlists).
* `commands.bashForegroundMs` (Standardwert `2000`) steuert, wie lange bash wartet, bevor in den Hintergrundmodus gewechselt wird (`0` verschiebt sofort in den Hintergrund).
* `commands.config` (Standardwert `false`) aktiviert `/config` (liest/schreibt `openclaw.json`).
* `commands.debug` (Standardwert `false`) aktiviert `/debug` (Überschreibungen nur zur Laufzeit).
* `commands.useAccessGroups` (Standardwert `true`) erzwingt Allowlists/Richtlinien für Commands.

<div id="command-list">
  ## Befehlsliste
</div>

Text + native (falls aktiviert):

* `/help`
* `/commands`
* `/skill <name> [input]` (führt einen Skill anhand seines Namens aus)
* `/status` (zeigt den aktuellen Status; enthält Anbieter-Nutzung/Kontingent für den aktuellen Modellanbieter, falls verfügbar)
* `/allowlist` (Allowlist-Einträge auflisten/hinzufügen/entfernen)
* `/approve <id> allow-once|allow-always|deny` (Ausführungsfreigabe-Aufforderungen beantworten)
* `/context [list|detail|json]` (erklärt „Kontext“; `detail` zeigt Größe je Datei + Tool + Skill + System-Prompt)
* `/whoami` (zeigt deine Sender-ID; Alias: `/id`)
* `/subagents list|stop|log|info|send` (Sub-Agent-Ausführungen für die aktuelle Sitzung inspizieren, stoppen, protokollieren oder Nachrichten senden)
* `/config show|get|set|unset` (Konfiguration dauerhaft auf Datenträger speichern (nur Besitzer); erfordert `commands.config: true`)
* `/debug show|set|unset|reset` (Laufzeit-Overrides (nur Besitzer); erfordert `commands.debug: true`)
* `/usage off|tokens|full|cost` (Nutzungs-Fußzeile pro Antwort oder lokale Kostenzusammenfassung)
* `/tts off|always|inbound|tagged|status|provider|limit|summary|audio` (TTS steuern; siehe [/tts](/de/tts))
  * Discord: nativer Befehl ist `/voice` (Discord reserviert `/tts`); Text `/tts` funktioniert weiterhin.
* `/stop`
* `/restart`
* `/dock-telegram` (Alias: `/dock_telegram`) (Antworten auf Telegram umstellen)
* `/dock-discord` (Alias: `/dock_discord`) (Antworten auf Discord umstellen)
* `/dock-slack` (Alias: `/dock_slack`) (Antworten auf Slack umstellen)
* `/activation mention|always` (nur Gruppen)
* `/send on|off|inherit` (nur Besitzer)
* `/reset` oder `/new [model]` (optionaler Modell-Hinweis; Rest wird durchgereicht)
* `/think <off|minimal|low|medium|high|xhigh>` (dynamische Auswahl je nach Modell/Anbieter; Aliasse: `/thinking`, `/t`)
* `/verbose on|full|off` (Alias: `/v`)
* `/reasoning on|off|stream` (Alias: `/reason`; wenn aktiv, wird eine separate Nachricht mit Präfix `Reasoning:` gesendet; `stream` = nur Telegram-Entwurf)
* `/elevated on|off|ask|full` (Alias: `/elev`; `full` überspringt Ausführungsfreigaben)
* `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>` (sende `/exec`, um den aktuellen Status anzuzeigen)
* `/model <name>` (Alias: `/models`; oder `/<alias>` aus `agents.defaults.models.*.alias`)
* `/queue <mode>` (plus Optionen wie `debounce:2s cap:25 drop:summarize`; `/queue` senden, um aktuelle Einstellungen anzuzeigen)
* `/bash <command>` (nur Host; Alias für `! <command>`; erfordert `commands.bash: true` + `tools.elevated`-Allowlist-Einträge)

Nur Text:

* `/compact [instructions]` (siehe [/concepts/compaction](/de/concepts/compaction))
* `! <command>` (nur Host; jeweils nur ein Befehl; verwende `!poll` + `!stop` für lang andauernde Jobs)
* `!poll` (Ausgabe/Status prüfen; akzeptiert optional `sessionId`; `/bash poll` funktioniert ebenfalls)
* `!stop` (den laufenden Bash-Job stoppen; akzeptiert optional `sessionId`; `/bash stop` funktioniert ebenfalls)

Hinweise:

* Befehle akzeptieren optional einen Doppelpunkt `:` zwischen Befehl und Argumenten (z. B. `/think: high`, `/send: on`, `/help:`).
* `/new <model>` akzeptiert einen Modell-Alias, `provider/model` oder einen Anbieter-Namen (unscharfe Zuordnung); wenn kein Treffer gefunden wird, wird der Text als Nachrichteninhalt behandelt.
* Für eine vollständige Aufschlüsselung der Nutzung pro Anbieter verwende `openclaw status --usage`.
* `/allowlist add|remove` erfordert `commands.config=true` und beachtet den Channel-Wert `configWrites`.
* `/usage` steuert die Nutzungsfußzeile pro Antwort; `/usage cost` gibt eine lokale Kostenzusammenfassung aus den OpenClaw-Sitzungsprotokollen aus.
* `/restart` ist standardmäßig deaktiviert; setze `commands.restart: true`, um ihn zu aktivieren.
* `/verbose` ist für Debugging und zusätzliche Transparenz gedacht; lasse es bei normaler Nutzung **aus**.
* `/reasoning` (und `/verbose`) sind in Gruppenkontexten riskant: Sie können interne Überlegungen oder Tool-Ausgaben offenlegen, die du nicht freigeben wolltest. Lasse sie vorzugsweise deaktiviert, insbesondere in Gruppenchats.
* **Schnellpfad:** Nachrichten, die nur aus einem Befehl bestehen und von Absendern aus der Allowlist stammen, werden sofort verarbeitet (Umgehung von Queue + Modell).
* **Gruppen-Erwähnungs-Gating:** Nachrichten, die nur aus einem Befehl bestehen und von Absendern aus der Allowlist stammen, umgehen Erwähnungsanforderungen.
* **Inline-Shortcuts (nur Absender aus der Allowlist):** Bestimmte Befehle funktionieren auch, wenn sie in eine normale Nachricht eingebettet sind, und werden entfernt, bevor das Modell den verbleibenden Text sieht.
  * Beispiel: `hey /status` löst eine Statusantwort aus, und der restliche Text läuft durch den normalen Ablauf weiter.
* Aktuell: `/help`, `/commands`, `/status`, `/whoami` (`/id`).
* Nicht autorisierte reine Befehlsnachrichten werden stillschweigend ignoriert, und Inline-`/...`-Tokens werden als normaler Text behandelt.
* **Skill-Befehle:** `user-invocable`-Fähigkeiten werden als Slash-Befehle bereitgestellt. Namen werden auf `a-z0-9_` bereinigt (max. 32 Zeichen); Kollisionen erhalten numerische Suffixe (z. B. `_2`).
  * `/skill <name> [input]` führt eine Fähigkeit per Name aus (nützlich, wenn native Beschränkungen für Befehle eigene Befehle pro Fähigkeit verhindern).
  * Standardmäßig werden Skill-Befehle als normale Anfrage an das Modell weitergeleitet.
  * Fähigkeiten können optional `command-dispatch: tool` deklarieren, um den Befehl direkt an ein Tool zu leiten (deterministisch, ohne Modell).
  * Beispiel: `/prose` (OpenProse Plugin) – siehe [OpenProse](/de/prose).
* **Native Befehlsargumente:** Discord verwendet Autovervollständigung für dynamische Optionen (und Schaltflächenmenüs, wenn du erforderliche Argumente weglässt). Telegram und Slack zeigen ein Schaltflächenmenü an, wenn ein Befehl Auswahlmöglichkeiten unterstützt und du das Argument weglässt.

<div id="usage-surfaces-what-shows-where">
  ## Nutzungsansichten (was wo angezeigt wird)
</div>

* **Anbieter-Nutzung/-Kontingent** (Beispiel: „Claude 80 % verbleibend“) wird in `/status` für den aktuellen Modellanbieter angezeigt, wenn Nutzungs-Tracking aktiviert ist.
* **Tokens/Kosten pro Antwort** werden über `/usage off|tokens|full` gesteuert (an normale Antworten angehängt).
* `/model status` bezieht sich auf **Modelle/Auth/Endpunkte**, nicht auf die Nutzung.

<div id="model-selection-model">
  ## Modellauswahl (`/model`)
</div>

`/model` ist als Direktive implementiert.

Beispiele:

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model opus@anthropic:default
/model status
```

Hinweise:

* `/model` und `/model list` zeigen eine kompakte, nummerierte Auswahlliste (Modellfamilie + verfügbare Anbieter).
* `/model <#>` wählt einen Eintrag aus dieser Liste aus (und bevorzugt nach Möglichkeit den aktuellen Anbieter).
* `/model status` zeigt eine Detailansicht, einschließlich des konfigurierten Anbieter-Endpunkts (`baseUrl`) und des API-Modus (`api`), sofern verfügbar.

<div id="debug-overrides">
  ## Debug-Overrides
</div>

`/debug` ermöglicht es dir, **nur zur Laufzeit wirksame** Konfigurations-Overrides zu setzen (im Speicher, nicht auf der Festplatte). Nur für Owner. Standardmäßig deaktiviert; aktiviere es mit `commands.debug: true`.

Beispiele:

```
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug set channels.whatsapp.allowFrom=["+1555","+4477"]
/debug unset messages.responsePrefix
/debug reset
```

Hinweise:

* Overrides gelten sofort für neue Konfigurations-Lesevorgänge, werden aber **nicht** in `openclaw.json` geschrieben.
* Verwende `/debug reset`, um alle Overrides zu löschen und zur auf der Festplatte gespeicherten Konfiguration zurückzukehren.

<div id="config-updates">
  ## Konfigurationsupdates
</div>

`/config` schreibt in deine Konfigurationsdatei auf dem Datenträger (`openclaw.json`). Nur für Besitzer:innen. Standardmäßig deaktiviert; aktiviere es mit `commands.config: true`.

Beispiele:

```
/config show
/config show messages.responsePrefix
/config get messages.responsePrefix
/config set messages.responsePrefix="[openclaw]"
/config unset messages.responsePrefix
```

Hinweise:

* Die Konfiguration wird validiert, bevor sie geschrieben wird; ungültige Änderungen werden abgelehnt.
* `/config`-Aktualisierungen bleiben über Neustarts hinweg erhalten.

<div id="surface-notes">
  ## Notizen zur Oberfläche
</div>

* **Textbefehle** laufen in der normalen Chat-Sitzung (DMs verwenden gemeinsam `main`, Gruppen haben ihre eigene Sitzung).
* **Native Befehle** verwenden isolierte Sitzungen:
  * Discord: `agent:<agentId>:discord:slash:<userId>`
  * Slack: `agent:<agentId>:slack:slash:<userId>` (Präfix konfigurierbar über `channels.slack.slashCommand.sessionPrefix`)
  * Telegram: `telegram:slash:<userId>` (zielt über `CommandTargetSessionKey` auf die Chat-Sitzung)
* **`/stop`** zielt auf die aktive Chat-Sitzung, damit die aktuelle Ausführung abgebrochen werden kann.
* **Slack:** `channels.slack.slashCommand` wird weiterhin für einen einzelnen `/openclaw`-artigen Befehl unterstützt. Wenn du `commands.native` aktivierst, musst du für jeden integrierten Befehl einen eigenen Slack-Slash-Befehl anlegen (gleiche Namen wie `/help`). Befehlsargument-Menüs für Slack werden als ephemere Block-Kit-Schaltflächen ausgeliefert.