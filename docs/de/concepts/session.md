---
title: Sitzung
summary: "Regeln, Schlüssel und Persistenz für das Sitzungsmanagement in Chats"
read_when:
  - Bei Änderungen an der Sitzungsbehandlung oder -speicherung
---

<div id="session-management">
  # Sitzungsverwaltung
</div>

OpenClaw behandelt **eine Direktchat-Sitzung pro agent** als primär. Direktchats werden zu `agent:<agentId>:<mainKey>` (Standard: `main`) zusammengefasst, während Gruppen-/Kanal-Chats eigene Schlüssel erhalten. `session.mainKey` wird berücksichtigt.

Verwende `session.dmScope`, um zu steuern, wie **Direktnachrichten** gruppiert werden:

* `main` (Standard): Alle DMs teilen sich die Hauptsitzung für durchgängigen Verlauf.
* `per-peer`: Isolierung nach Absender-ID über Kanäle hinweg.
* `per-channel-peer`: Isolierung nach Kanal + Absender (empfohlen für Postfächer mit mehreren Nutzenden).
* `per-account-channel-peer`: Isolierung nach Konto + Kanal + Absender (empfohlen für Postfächer mit mehreren Konten).

Verwende `session.identityLinks`, um anbieterpräfixierte Peer-IDs einer kanonischen Identität zuzuordnen, sodass dieselbe Person bei Verwendung von `per-peer`, `per-channel-peer` oder `per-account-channel-peer` kanalübergreifend dieselbe DM-Sitzung verwendet.

<div id="gateway-is-the-source-of-truth">
  ## Gateway ist die maßgebliche Quelle der Wahrheit
</div>

Der gesamte Sitzungszustand gehört **dem Gateway** (der „Master“-OpenClaw-Instanz). UI-Clients (macOS-App, WebChat usw.) müssen das Gateway nach Sitzungslisten und Token-Anzahlen abfragen, statt lokale Dateien zu lesen.

* Im **Remote-Modus** befindet sich der für dich relevante Session-Store auf dem entfernten Gateway-Host, nicht auf deinem Mac.
* In UIs angezeigte Token-Anzahlen stammen aus den Store-Feldern des Gateways (`inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`). Clients parsen keine JSONL-Transkripte, um Gesamtwerte nachträglich zu „korrigieren“.

<div id="where-state-lives">
  ## Wo der Zustand gespeichert wird
</div>

* Auf dem **Gateway-Host**:
  * Store-Datei: `~/.openclaw/agents/<agentId>/sessions/sessions.json` (pro Agent).
* Transkripte: `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl` (Telegram-Topic-Sitzungen verwenden `.../<SessionId>-topic-<threadId>.jsonl`).
* Der Store ist eine Map `sessionKey -> { sessionId, updatedAt, ... }`. Das Löschen von Einträgen ist unbedenklich; sie werden bei Bedarf neu erstellt.
* Gruppeneinträge können `displayName`, `channel`, `subject`, `room` und `space` enthalten, um Sitzungen in UIs zu kennzeichnen.
* Sitzungseinträge enthalten `origin`-Metadaten (Label + Routing-Hinweise), damit UIs erklären können, woher eine Sitzung stammt.
* OpenClaw liest **keine** alten Pi/Tau-Sitzungsordner.

<div id="session-pruning">
  ## Sitzungsbereinigung
</div>

OpenClaw entfernt standardmäßig **alte Tool-Ergebnisse** aus dem In-Memory-Kontext unmittelbar vor LLM-Aufrufen.
Dies überschreibt die JSONL-Historie **nicht**. Siehe [/concepts/session-pruning](/de/concepts/session-pruning).

<div id="pre-compaction-memory-flush">
  ## Speicher-Flush vor der Kompaktierung
</div>

Wenn eine Sitzung kurz vor der automatischen Kompaktierung steht, kann OpenClaw
einen **stillen Speicher-Flush** ausführen, der das Modell daran erinnert,
dauerhafte Notizen auf den Datenträger zu schreiben. Dies wird nur ausgeführt,
wenn der Arbeitsbereich beschreibbar ist. Siehe [Speicher](/de/concepts/memory) und
[Kompaktierung](/de/concepts/compaction).

<div id="mapping-transports-session-keys">
  ## Zuordnung von Transporten → Sitzungsschlüsseln
</div>

* Direkte Chats folgen `session.dmScope` (Standard `main`).
  * `main`: `agent:<agentId>:<mainKey>` (Kontinuität über Geräte/Kanäle hinweg).
    * Mehrere Telefonnummern und Kanäle können demselben Agent-Hauptschlüssel zugeordnet werden; sie fungieren als Transportwege in eine Unterhaltung.
  * `per-peer`: `agent:<agentId>:dm:<peerId>`.
  * `per-channel-peer`: `agent:<agentId>:<channel>:dm:<peerId>`.
  * `per-account-channel-peer`: `agent:<agentId>:<channel>:<accountId>:dm:<peerId>` (`accountId` ist standardmäßig `default`).
  * Wenn `session.identityLinks` mit einer anbieterpräfigierten Peer-ID übereinstimmt (zum Beispiel `telegram:123`), ersetzt der kanonische Schlüssel `<peerId>`, sodass dieselbe Person kanalübergreifend dieselbe Sitzung verwendet.
* Gruppenchats isolieren Zustand: `agent:<agentId>:<channel>:group:<id>` (Räume/Kanäle verwenden `agent:<agentId>:<channel>:channel:<id>`).
  * Telegram-Forenthemen hängen zur Isolation `:topic:<threadId>` an die Gruppen-ID an.
  * Legacy-Schlüssel `group:<id>` werden für Migration weiterhin erkannt.
* Eingehende Kontexte können weiterhin `group:<id>` verwenden; der Kanal wird aus `Provider` abgeleitet und in die kanonische Form `agent:<agentId>:<channel>:group:<id>` normalisiert.
* Weitere Quellen:
  * Cron-Jobs: `cron:<job.id>`
  * Webhooks: `hook:<uuid>` (sofern nicht explizit vom Hook gesetzt)
  * Knoten-Läufe: `node-<nodeId>`

<div id="lifecycle">
  ## Lebenszyklus
</div>

* Reset-Policy: Sitzungen werden wiederverwendet, bis sie ablaufen; das Ablaufdatum wird bei der nächsten eingehenden Nachricht ausgewertet.
* Täglicher Reset: Standard ist **4:00 Uhr Ortszeit auf dem Gateway-Host**. Eine Sitzung gilt als veraltet, sobald ihre letzte Aktualisierung vor der jüngsten täglichen Reset-Zeit liegt.
* Inaktivitäts-Reset (optional): `idleMinutes` fügt ein gleitendes Inaktivitätsfenster hinzu. Wenn täglicher und Inaktivitäts-Reset beide konfiguriert sind, erzwingt **der zuerst ablaufende** eine neue Sitzung.
* Legacy „nur Inaktivität“: Wenn du `session.idleMinutes` ohne jegliche `session.reset`-/`resetByType`-Konfiguration setzt, bleibt OpenClaw aus Gründen der Abwärtskompatibilität im Nur-Inaktivitäts-Modus.
* Überschreibungen pro Typ (optional): `resetByType` erlaubt dir, die Policy für `dm`-, `group`- und `thread`-Sitzungen zu überschreiben (thread = Slack-/Discord-Threads, Telegram-Themen, Matrix-Threads, wenn vom Connector bereitgestellt).
* Überschreibungen pro Kanal (optional): `resetByChannel` überschreibt die Reset-Policy für einen Kanal (gilt für alle Sitzungstypen für diesen Kanal und hat Vorrang vor `reset`/`resetByType`).
* Reset-Trigger: Exaktes `/new` oder `/reset` (plus alle zusätzlichen in `resetTriggers`) startet eine neue `sessionId` und lässt den Rest der Nachricht unverändert durch. `/new <model>` akzeptiert einen Modell-Alias, `provider/model` oder einen Anbieter-Namen (unscharfe Übereinstimmung), um das Modell der neuen Sitzung zu setzen. Wenn `/new` oder `/reset` allein gesendet wird, führt OpenClaw einen kurzen „Hallo“-Begrüßungsdurchlauf aus, um den Reset zu bestätigen.
* Manueller Reset: Lösche bestimmte Schlüssel aus dem Store oder entferne das JSONL-Transkript; die nächste Nachricht legt sie neu an.
* Isolierte Cron-Jobs erzeugen immer eine neue `sessionId` pro Lauf (keine Inaktivitäts-Wiederverwendung).

<div id="send-policy-optional">
  ## Sende-Richtlinie (optional)
</div>

Blockiere die Zustellung für bestimmte Sitzungstypen, ohne einzelne IDs anzugeben.

```json5
{
  session: {
    sendPolicy: {
      rules: [
        { action: "deny", match: { channel: "discord", chatType: "group" } },
        { action: "deny", match: { keyPrefix: "cron:" } }
      ],
      default: "allow"
    }
  }
}
```

Laufzeit-Override (nur Eigentümer):

* `/send on` → für diese Sitzung erlauben
* `/send off` → für diese Sitzung verweigern
* `/send inherit` → Override aufheben und Konfigurationsregeln verwenden
  Sende diese als eigenständige Nachrichten, damit sie wirksam werden.

<div id="configuration-optional-rename-example">
  ## Konfiguration (Beispiel für optionales Umbenennen)
</div>

```json5
// ~/.openclaw/openclaw.json
{
  session: {
    scope: "per-sender",      // Gruppenschlüssel getrennt halten
    dmScope: "main",          // DM-Kontinuität (für gemeinsame Postfächer auf per-channel-peer/per-account-channel-peer setzen)
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"]
    },
    reset: {
      // Standardwerte: mode=daily, atHour=4 (lokale Zeit des Gateway-Hosts).
      // Falls Sie auch idleMinutes setzen, gilt der zuerst ablaufende Wert.
      mode: "daily",
      atHour: 4,
      idleMinutes: 120
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      dm: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 }
    },
    resetByChannel: {
      discord: { mode: "idle", idleMinutes: 10080 }
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    mainKey: "main",
  }
}
```

<div id="inspecting">
  ## Überprüfen
</div>

* `openclaw status` — zeigt den Speicherpfad und die neuesten Sitzungen.
* `openclaw sessions --json` — gibt alle Einträge aus (mit `--active <minutes>` filtern).
* `openclaw gateway call sessions.list --params '{}'` — ruft Sitzungen vom laufenden Gateway ab (verwende `--url`/`--token` für den Zugriff auf ein entferntes Gateway).
* Sende `/status` als eigenständige Nachricht im Chat, um zu sehen, ob der agent erreichbar ist, wie viel des Sitzungs­kontexts genutzt wird, welche Thinking-/Verbose-Schalter aktiv sind und wann deine WhatsApp-Web-Anmeldedaten zuletzt aktualisiert wurden (hilft, Neuverknüpfungsbedarf zu erkennen).
* Sende `/context list` oder `/context detail`, um zu sehen, was sich im System-Prompt und in den injizierten Arbeitsbereichsdateien befindet (und welche Elemente am stärksten zum Kontext beitragen).
* Sende `/stop` als eigenständige Nachricht, um den aktuellen Lauf abzubrechen, geplante Folgeaktionen für diese Sitzung zu löschen und alle von ihr erzeugten Sub-agent-Läufe zu stoppen (die Antwort enthält die Anzahl der gestoppten Läufe).
* Sende `/compact` (optionale Anweisungen) als eigenständige Nachricht, um älteren Kontext zusammenzufassen und Platz im Kontextfenster freizugeben. Siehe [/concepts/compaction](/de/concepts/compaction).
* JSONL-Transkripte können direkt geöffnet werden, um vollständige Dialogschritte zu überprüfen.

<div id="tips">
  ## Tipps
</div>

* Reserviere den primären Schlüssel ausschließlich für 1:1-Traffic; lass Gruppen ihre eigenen Schlüssel behalten.
* Wenn du das Aufräumen automatisierst, lösche einzelne Schlüssel statt den gesamten Store, um Kontext an anderer Stelle beizubehalten.

<div id="session-origin-metadata">
  ## Sitzungsursprungs-Metadaten
</div>

Jeder Sitzungseintrag protokolliert bestmöglich, woher er stammt, in `origin`:

* `label`: menschenlesbares Label (abgeleitet aus Gesprächslabel + Gruppenbetreff/-kanal)
* `provider`: normalisierte Kanal-ID (einschließlich Erweiterungen)
* `from`/`to`: rohe Routing-IDs aus dem eingehenden Umschlag
* `accountId`: Anbieter-Konto-ID (bei mehreren Konten)
* `threadId`: Thread-/Themen-ID, wenn der Kanal dies unterstützt
  Die Ursprungsfelder werden für Direktnachrichten, Kanäle und Gruppen befüllt. Wenn ein
  Connector nur das Zustellrouting aktualisiert (zum Beispiel, um eine DM-Hauptsitzung
  aktuell zu halten), sollte er trotzdem eingehenden Kontext bereitstellen, damit die Sitzung ihre
  erklärenden Metadaten behält. Erweiterungen können dies tun, indem sie `ConversationLabel`,
  `GroupSubject`, `GroupChannel`, `GroupSpace` und `SenderName` im eingehenden
  Kontext senden und `recordSessionMetaFromInbound` aufrufen (oder denselben Kontext
  an `updateLastRoute` übergeben).