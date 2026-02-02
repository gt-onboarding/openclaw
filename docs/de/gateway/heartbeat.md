---
title: Herzschlag
summary: "Herzschlag-Polling-Nachrichten und Benachrichtigungsregeln"
read_when:
  - Wenn du die Herzschlagfrequenz oder die zugehörigen Nachrichten anpassen möchtest
  - Wenn du zwischen Herzschlag und Cron für geplante Tasks wählen musst
---

<div id="heartbeat-gateway">
  # Herzschlag (Gateway)
</div>

> **Herzschlag vs Cron?** Siehe [Cron vs Heartbeat](/de/automation/cron-vs-heartbeat) für Hinweise, wann Sie welches verwenden sollten.

Herzschlag führt **periodische Interaktionen des agents** in der Hauptsitzung aus, damit das Modell
alles aufzeigen kann, was Aufmerksamkeit benötigt, ohne Sie dabei zu überfluten.

<div id="quick-start-beginner">
  ## Schnellstart (Einsteiger)
</div>

1. Lass Herzschläge aktiviert (Standard ist `30m` bzw. `1h` für Anthropic OAuth/setup-token) oder lege dein eigenes Intervall fest.
2. Lege eine kleine `HEARTBEAT.md`-Checkliste im agent-arbeitsbereich an (optional, aber empfohlen).
3. Entscheide, wohin Herzschlag-Nachrichten gesendet werden sollen (`target: "last"` ist der Standardwert).
4. Optional: Aktiviere die Ausgabe des Herzschlag-Reasonings für mehr Transparenz.
5. Optional: Beschränke Herzschläge auf aktive Stunden (Ortszeit).

Beispielkonfiguration:

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
        // activeHours: { start: "08:00", end: "24:00" },
        // includeReasoning: true, // optional: separate `Reasoning:`-Nachricht ebenfalls senden
      }
    }
  }
}
```

<div id="defaults">
  ## Standardwerte
</div>

* Intervall: `30m` (oder `1h`, wenn Anthropic OAuth/setup-token als Authentifizierungsmodus erkannt wird). Setzen Sie `agents.defaults.heartbeat.every` oder pro Agent `agents.list[].heartbeat.every`; verwenden Sie `0m`, um zu deaktivieren.
* Prompt-Text (konfigurierbar über `agents.defaults.heartbeat.prompt`):
  `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`
* Der Herzschlag-Prompt wird **wortwörtlich** als Benutzernachricht gesendet. Der Systemprompt enthält einen „Heartbeat“-Abschnitt und die Ausführung wird intern markiert.
* Aktive Stunden (`heartbeat.activeHours`) werden in der konfigurierten Zeitzone berücksichtigt. Außerhalb dieses Zeitfensters werden Herzschläge bis zum nächsten Tick innerhalb des Fensters übersprungen.

<div id="what-the-heartbeat-prompt-is-for">
  ## Wofür die Herzschlag-Prompt gedacht ist
</div>

Die Standard-Prompt ist absichtlich breit formuliert:

* **Hintergrundaufgaben**: „Consider outstanding tasks“ stupst den Agent an, offene
  Folgeaufgaben (Posteingang, Kalender, Erinnerungen, anstehende/eingereihte Arbeit) zu prüfen
  und Dringendes hervorzuheben.
* **Menschlicher Check-in**: „Checkup sometimes on your human during day time“ stupst
  gelegentliche, kurze „Brauchst du gerade etwas?“-Nachrichten an, vermeidet aber
  Spam in der Nacht, indem deine konfigurierte lokale Zeitzone verwendet wird (siehe
  [/concepts/timezone](/de/concepts/timezone)).

Wenn du möchtest, dass ein Herzschlag etwas sehr Spezifisches tut (z. B. „check Gmail PubSub
stats“ oder „verify gateway health“), setze `agents.defaults.heartbeat.prompt` (oder
`agents.list[].heartbeat.prompt`) auf einen benutzerdefinierten Text (wird unverändert gesendet).

<div id="response-contract">
  ## Response-Contract
</div>

* Wenn nichts Aufmerksamkeit erfordert, antworte mit **`HEARTBEAT_OK`**.
* Während Herzschlag-Läufen behandelt OpenClaw `HEARTBEAT_OK` als Ack, wenn es
  **am Anfang oder Ende** der Antwort erscheint. Das Token wird entfernt und die Antwort
  verworfen, wenn der verbleibende Inhalt **≤ `ackMaxChars`** ist (Standard: 300).
* Wenn `HEARTBEAT_OK` in der **Mitte** einer Antwort erscheint, wird es nicht
  speziell behandelt.
* Für Alerts **kein** `HEARTBEAT_OK` einfügen; gib nur den Alert-Text zurück.

Außerhalb von Herzschlag-Läufen werden verirrte `HEARTBEAT_OK` am Anfang/Ende einer Nachricht entfernt
und protokolliert; eine Nachricht, die nur aus `HEARTBEAT_OK` besteht, wird verworfen.

<div id="config">
  ## Konfiguration
</div>

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",           // default: 30m (0m disables)
        model: "anthropic/claude-opus-4-5",
        includeReasoning: false, // default: false (deliver separate Reasoning: message when available)
        target: "last",         // last | none | <channel id> (core or plugin, e.g. "bluebubbles")
        to: "+15551234567",     // optional channel-specific override
        prompt: "Lies HEARTBEAT.md, falls vorhanden (Arbeitsbereich-Kontext). Befolge es strikt. Leite keine alten Aufgaben aus früheren Chats ab oder wiederhole sie. Falls nichts Aufmerksamkeit erfordert, antworte mit HEARTBEAT_OK.",
        ackMaxChars: 300         // max chars allowed after HEARTBEAT_OK
      }
    }
  }
}
```

<div id="scope-and-precedence">
  ### Geltungsbereich und Priorität
</div>

* `agents.defaults.heartbeat` legt das globale Herzschlag-Verhalten fest.
* `agents.list[].heartbeat` wird darüber angewendet; wenn ein Agent einen `heartbeat`-Block hat, führen **nur diese Agenten** Herzschläge aus.
* `channels.defaults.heartbeat` legt Sichtbarkeits-Standardwerte für alle Kanäle fest.
* `channels.<channel>.heartbeat` überschreibt die Kanal-Standardwerte.
* `channels.<channel>.accounts.<id>.heartbeat` (Kanäle mit mehreren Accounts) überschreibt die kanalspezifischen Einstellungen.

<div id="per-agent-heartbeats">
  ### Herzschläge pro Agent
</div>

Wenn ein Eintrag in `agents.list[]` einen `heartbeat`-Block enthält, **laufen nur für diese Agenten**
Herzschläge. Der agent-spezifische Block wird über `agents.defaults.heartbeat` gelegt
(so kannst du gemeinsame Voreinstellungen einmal definieren und pro Agent überschreiben).

Beispiel: zwei Agenten, nur für den zweiten Agent laufen Herzschläge.

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last"
      }
    },
    list: [
      { id: "main", default: true },
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "whatsapp",
          to: "+15551234567",
          prompt: "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK."
        }
      }
    ]
  }
}
```

<div id="field-notes">
  ### Feldnotizen
</div>

* `every`: Herzschlagintervall (Duration-String; Standardeinheit = Minuten).
* `model`: optionaler Modell-Override für Herzschlag-Ausführungen (`provider/model`).
* `includeReasoning`: wenn aktiviert, wird auch die separate `Reasoning:`-Nachricht zugestellt, wenn verfügbar (gleiche Struktur wie `/reasoning on`).
* `session`: optionaler Sitzungsschlüssel für Herzschlag-Ausführungen.
  * `main` (Standard): Hauptsitzung des Agents.
  * expliziter Sitzungsschlüssel (aus `openclaw sessions --json` oder der [sessions CLI](/de/cli/sessions) kopieren).
  * Sitzungsschlüsselformate: siehe [Sessions](/de/concepts/session) und [Groups](/de/concepts/groups).
* `target`:
  * `last` (Standard): an den zuletzt verwendeten externen Kanal zustellen.
  * expliziter Kanal: `whatsapp` / `telegram` / `discord` / `googlechat` / `slack` / `msteams` / `signal` / `imessage`.
  * `none`: den Herzschlag ausführen, aber **nicht extern zustellen**.
* `to`: optionaler Empfänger-Override (kanalspezifische ID, z. B. E.164 für WhatsApp oder eine Telegram-Chat-ID).
* `prompt`: überschreibt den Standard-Prompttext (wird nicht zusammengeführt).
* `ackMaxChars`: maximal zulässige Anzahl an Zeichen nach `HEARTBEAT_OK` vor der Zustellung.

<div id="delivery-behavior">
  ## Zustellverhalten
</div>

* Herzschläge laufen standardmäßig in der Hauptsitzung des Agents (`agent:<id>:<mainKey>`),
  oder `global`, wenn `session.scope = "global"` ist. Setze `session`, um auf eine
  spezifische Kanalsitzung (Discord/WhatsApp/etc.) umzuschalten.
* `session` beeinflusst nur den Ausführungskontext; die Zustellung wird durch `target` und `to` gesteuert.
* Um an einen bestimmten Kanal bzw. Empfänger zu senden, setze `target` + `to`. Mit
  `target: "last"` verwendet die Zustellung den letzten externen Kanal für diese Sitzung.
* Wenn die Hauptwarteschlange ausgelastet ist, wird der Herzschlag übersprungen und später erneut ausgeführt.
* Wenn `target` zu keinem externen Ziel aufgelöst wird, läuft die Ausführung trotzdem, aber es
  wird keine ausgehende Nachricht gesendet.
* Reine Herzschlag-Antworten halten die Sitzung **nicht** aktiv; das letzte `updatedAt`
  wird wiederhergestellt, sodass der Leerlauf-Timeout normal greift.

<div id="visibility-controls">
  ## Sichtbarkeitseinstellungen
</div>

Standardmäßig werden `HEARTBEAT_OK`-Bestätigungen unterdrückt, während Alarmmeldungen
zugestellt werden. Du kannst dies pro Kanal oder pro Konto anpassen:

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false      # Hide HEARTBEAT_OK (default)
      showAlerts: true   # Show alert messages (default)
      useIndicator: true # Emit indicator events (default)
  telegram:
    heartbeat:
      showOk: true       # Show OK acknowledgments on Telegram
  whatsapp:
    accounts:
      work:
        heartbeat:
          showAlerts: false # Alarmzustellung für dieses Konto unterdrücken
```

Vorrang: pro Konto → pro Kanal → Kanal-Standardwerte → eingebaute Standardwerte.

<div id="what-each-flag-does">
  ### Was jedes Flag bewirkt
</div>

* `showOk`: sendet eine `HEARTBEAT_OK`-Bestätigung, wenn das Modell eine reine OK-Antwort zurückgibt.
* `showAlerts`: sendet den Alert-Inhalt, wenn das Modell eine Nicht-OK-Antwort zurückgibt.
* `useIndicator`: löst Indikator-Ereignisse für UI-Statusoberflächen aus.

Wenn **alle drei** auf false gesetzt sind, überspringt OpenClaw den Herzschlag-Durchlauf vollständig (kein Modellaufruf).

<div id="per-channel-vs-per-account-examples">
  ### Beispiele auf Kanal- vs. Kontoebene
</div>

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false
      showAlerts: true
      useIndicator: true
  slack:
    heartbeat:
      showOk: true # alle Slack-Accounts
    accounts:
      ops:
        heartbeat:
          showAlerts: false # Warnungen nur für den ops-Account unterdrücken
  telegram:
    heartbeat:
      showOk: true
```

<div id="common-patterns">
  ### Häufige Muster
</div>

| Ziel | Konfiguration |
| --- | --- |
| Standardverhalten (stille OKs, Warnungen an) | *(keine Konfiguration erforderlich)* |
| Vollständig stumm (keine Nachrichten, kein Indikator) | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: false }` |
| Nur Indikator (keine Nachrichten) | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: true }` |
| OKs nur in einem Kanal | `channels.telegram.heartbeat: { showOk: true }` |

<div id="heartbeatmd-optional">
  ## HEARTBEAT.md (optional)
</div>

Wenn im arbeitsbereich eine `HEARTBEAT.md`‑Datei vorhanden ist, weist der Standard‑Prompt den agent an, sie zu lesen. Stell sie dir als deine „Herzschlag‑Checkliste“ vor: klein, stabil und sicher, alle 30 Minuten einzubinden.

Wenn `HEARTBEAT.md` existiert, aber effektiv leer ist (nur Leerzeilen und Markdown‑Überschriften wie `# Heading`), überspringt OpenClaw den Herzschlag‑Durchlauf, um api‑Aufrufe zu sparen. Wenn die Datei fehlt, läuft der Herzschlag trotzdem und das Modell entscheidet, was zu tun ist.

Halte sie möglichst klein (kurze Checkliste oder Erinnerungen), um ein Aufblähen des Prompts zu vermeiden.

Beispiel `HEARTBEAT.md`:

```md
# Herzschlag-Checkliste

- Schneller Scan: etwas Dringendes in den Posteingängen?
- Falls es tagsüber ist, führe einen leichtgewichtigen Check-in durch, wenn nichts anderes ansteht.
- Falls eine Aufgabe blockiert ist, notiere *was fehlt* und frage Peter beim nächsten Mal.
```

<div id="can-the-agent-update-heartbeatmd">
  ### Kann der agent `HEARTBEAT.md` aktualisieren?
</div>

Ja – wenn du ihn darum bittest.

`HEARTBEAT.md` ist nur eine normale Datei im arbeitsbereich des agents, daher kannst du dem
agent (in einem normalen Chat) zum Beispiel sagen:

* „Aktualisiere `HEARTBEAT.md`, um eine tägliche Kalenderprüfung hinzuzufügen.“
* „Schreibe `HEARTBEAT.md` neu, damit es kürzer ist und sich auf Inbox-Nachverfolgungen konzentriert.“

Wenn du möchtest, dass dies proaktiv geschieht, kannst du auch eine explizite Zeile in
deinen Herzschlag-Prompt aufnehmen, etwa: „Wenn die Checkliste veraltet ist, aktualisiere `HEARTBEAT.md`
mit einer besseren Version.“

Hinweis zur Sicherheit: Lege keine Geheimnisse (API-Schlüssel, Telefonnummern, private Tokens) in
`HEARTBEAT.md` ab – es wird Teil des Prompt-Kontexts.

<div id="manual-wake-on-demand">
  ## Manuelles Auslösen (bei Bedarf)
</div>

Du kannst ein Systemereignis in die Warteschlange stellen und einen sofortigen Herzschlag auslösen mit:

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
```

Wenn mehrere Agenten `heartbeat` konfiguriert haben, führt ein manuelles Wecken die Herzschläge all dieser Agenten sofort aus.

Verwende `--mode next-heartbeat`, um auf den nächsten geplanten Tick zu warten.

<div id="reasoning-delivery-optional">
  ## Reasoning-Ausgabe (optional)
</div>

Standardmäßig liefern Herzschläge nur das finale „Antwort“-Payload.

Wenn du mehr Transparenz möchtest, aktiviere:

* `agents.defaults.heartbeat.includeReasoning: true`

Wenn diese Option aktiviert ist, liefern Herzschläge zusätzlich eine separate Nachricht mit dem Präfix
`Reasoning:` (gleiche Struktur wie `/reasoning on`). Das kann hilfreich sein, wenn der agent
mehrere Sitzungen/Kodizes verwaltet und du sehen willst, warum er dich anpingt
— kann aber auch mehr interne Details preisgeben, als du möchtest. In Gruppenchats solltest du es in der Regel deaktiviert lassen.

<div id="cost-awareness">
  ## Kostenbewusstsein
</div>

Herzschläge führen vollständige agent-Durchläufe aus. Kürzere Intervalle verbrauchen
mehr Token. Halte `HEARTBEAT.md` klein und erwäge ein günstigeres `model` oder
`target: "none"`, wenn du nur interne Status-Updates möchtest.