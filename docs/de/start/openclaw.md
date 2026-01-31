---
title: Openclaw
summary: "End-to-End-Anleitung für den Betrieb von OpenClaw als persönlichem Assistenten mit Sicherheitshinweisen"
read_when:
  - Beim Onboarding einer neuen Assistant-Instanz
  - Beim Überprüfen der Auswirkungen auf Sicherheit und Berechtigungen
---

<div id="building-a-personal-assistant-with-openclaw">
  # Erstellen eines persönlichen Assistenten mit OpenClaw
</div>

OpenClaw ist ein WhatsApp- + Telegram- + Discord- + iMessage-Gateway für **Pi**-Agenten. Plugins fügen Mattermost-Unterstützung hinzu. Dieser Leitfaden beschreibt das Setup für einen „persönlichen Assistenten“: eine dedizierte WhatsApp-Nummer, die sich wie dein dauerhaft erreichbarer Agent verhält.

<div id="safety-first">
  ## ⚠️ Sicherheit zuerst
</div>

Du versetzt einen agent in die Lage:

* Befehle auf deinem Rechner auszuführen (abhängig von deinem Pi-Tool-Setup)
* Dateien in deinem arbeitsbereich zu lesen und zu schreiben
* Nachrichten über WhatsApp/Telegram/Discord/Mattermost (Plugin) nach außen zu senden

Starte konservativ:

* Setze immer `channels.whatsapp.allowFrom` (führe auf deinem persönlichen Mac niemals ein Setup aus, das aus dem gesamten Internet erreichbar ist).
* Verwende eine eigene WhatsApp-Nummer für den Assistenten.
* Der Herzschlag läuft jetzt standardmäßig alle 30 Minuten. Deaktiviere ihn, bis du dem Setup vertraust, indem du `agents.defaults.heartbeat.every: "0m"` setzt.

<div id="prerequisites">
  ## Voraussetzungen
</div>

* Node **22+**
* OpenClaw im PATH verfügbar (empfohlen: global installiert)
* Eine zweite Telefonnummer (SIM/eSIM/Prepaid) für den Assistenten

```bash
npm install -g openclaw@latest
# oder: pnpm add -g openclaw@latest
```

Vom Quellgerät (Entwicklung):

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # installiert UI-Abhängigkeiten beim ersten Ausführen automatisch
pnpm build
pnpm link --global
```

<div id="the-two-phone-setup-recommended">
  ## Das Zwei-Smartphone-Setup (empfohlen)
</div>

So soll es aussehen:

```
Your Phone (personal)          Second Phone (assistant)
┌─────────────────┐           ┌─────────────────┐
│  Your WhatsApp  │  ──────▶  │  Assistant WA   │
│  +1-555-YOU     │  message  │  +1-555-ASSIST  │
└─────────────────┘           └────────┬────────┘
                                       │ linked via QR
                                       ▼
                              ┌─────────────────┐
                              │  Your Mac       │
                              │  (openclaw)      │
                              │    Pi agent     │
                              └─────────────────┘
```

Wenn du dein persönliches WhatsApp mit OpenClaw verknüpfst, wird jede Nachricht an dich zu „Agent-Input“. Das möchtest du in der Regel nicht.

<div id="5-minute-quick-start">
  ## 5-Minuten-Schnellstart
</div>

1. WhatsApp Web koppeln (zeigt einen QR-Code; mit dem Telefon des Assistenten scannen):

```bash
openclaw channels login
```

2. Starte das Gateway (und lasse es laufen):

```bash
openclaw gateway --port 18789
```

3. Erstelle eine minimale Konfigurationsdatei unter `~/.openclaw/openclaw.json`:

```json5
{
  channels: { whatsapp: { allowFrom: ["+15555550123"] } }
}
```

Sende jetzt eine Nachricht von deinem in der Allowlist eingetragenen Telefon an die Assistenten-Nummer.

Wenn das Onboarding abgeschlossen ist, öffnen wir automatisch das Dashboard mit deinem Gateway-Token und geben den tokenisierten Link aus. Um das Dashboard später erneut zu öffnen: `openclaw dashboard`.

<div id="give-the-agent-a-workspace-agents">
  ## Gib dem Agent einen Arbeitsbereich (AGENTS)
</div>

OpenClaw liest Bedienungsanweisungen und „Speicher“ aus seinem Arbeitsbereichsverzeichnis ein.

Standardmäßig verwendet OpenClaw `~/.openclaw/workspace` als Agent-Arbeitsbereich und erstellt ihn (plus die Startdateien `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`) automatisch bei der Einrichtung bzw. beim ersten Lauf eines Agenten. `BOOTSTRAP.md` wird nur erstellt, wenn der Arbeitsbereich komplett neu ist (diese Datei sollte nicht wieder auftauchen, nachdem du sie gelöscht hast).

Hinweis: Behandle diesen Ordner wie das „Gedächtnis“ von OpenClaw und mach ihn zu einem Git-Repository (idealerweise privat), damit deine `AGENTS.md`- und Speicherdateien gesichert sind. Wenn Git installiert ist, werden komplett neue Arbeitsbereiche automatisch initialisiert.

```bash
openclaw setup
```

Vollständiges Layout des Arbeitsbereichs und Backup-Leitfaden: [Agent-Arbeitsbereich](/de/concepts/agent-workspace)
Memory-Workflow: [Memory](/de/concepts/memory)

Optional: Wähle einen anderen Arbeitsbereich mit `agents.defaults.workspace` (unterstützt `~`).

```json5
{
  agent: {
    workspace: "~/.openclaw/workspace"
  }
}
```

Wenn du deine eigenen Arbeitsbereichsdateien bereits aus einem Repository auslieferst, kannst du die Erstellung der Bootstrap-Dateien vollständig deaktivieren:

```json5
{
  agent: {
    skipBootstrap: true
  }
}
```

<div id="the-config-that-turns-it-into-an-assistant">
  ## Die Konfiguration, die es in „einen Assistenten“ verwandelt
</div>

OpenClaw ist standardmäßig als guter Assistent vorkonfiguriert, aber in der Regel möchtest du Folgendes anpassen:

* Persona/Anweisungen in `SOUL.md`
* Denk-Standardeinstellungen (falls gewünscht)
* Herzschläge (sobald du ihm vertraust)

Beispiel:

```json5
{
  logging: { level: "info" },
  agent: {
    model: "anthropic/claude-opus-4-5",
    workspace: "~/.openclaw/workspace",
    thinkingDefault: "high",
    timeoutSeconds: 1800,
    // Zunächst mit 0 starten; später aktivieren.
    heartbeat: { every: "0m" }
  },
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true }
      }
    }
  },
  routing: {
    groupChat: {
      mentionPatterns: ["@openclaw", "openclaw"]
    }
  },
  session: {
    scope: "per-sender",
    resetTriggers: ["/new", "/reset"],
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 10080
    }
  }
}
```

<div id="sessions-and-memory">
  ## Sitzungen und Speicher
</div>

* Sitzungsdateien: `~/.openclaw/agents/<agentId>/sessions/{{SessionId}}.jsonl`
* Sitzungsmetadaten (Tokenverbrauch, letzte Route usw.): `~/.openclaw/agents/<agentId>/sessions/sessions.json` (Legacy: `~/.openclaw/sessions/sessions.json`)
* `/new` oder `/reset` startet eine neue Sitzung für diesen Chat (konfigurierbar über `resetTriggers`). Wenn der Befehl allein gesendet wird, antwortet der agent mit einer kurzen Begrüßung zur Bestätigung des Zurücksetzens.
* `/compact [instructions]` verdichtet den Sitzungskontext und meldet das verbleibende Kontextbudget.

<div id="heartbeats-proactive-mode">
  ## Herzschläge (proaktiver Modus)
</div>

Standardmäßig führt OpenClaw alle 30 Minuten einen Herzschlag mit folgendem Prompt aus:
`Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`
Setze `agents.defaults.heartbeat.every: "0m"` zum Deaktivieren.

* Wenn `HEARTBEAT.md` existiert, aber effektiv leer ist (nur Leerzeilen und Markdown-Überschriften wie `# Heading`), überspringt OpenClaw den Herzschlag-Durchlauf, um API-Aufrufe zu sparen.
* Wenn die Datei fehlt, wird der Herzschlag trotzdem ausgeführt und das Modell entscheidet, was zu tun ist.
* Wenn der Agent mit `HEARTBEAT_OK` antwortet (optional mit kurzem Padding; siehe `agents.defaults.heartbeat.ackMaxChars`), unterdrückt OpenClaw die ausgehende Zustellung für diesen Herzschlag.
* Herzschläge laufen als vollständige Agent-Turns – kürzere Intervalle verbrauchen mehr Tokens.

```json5
{
  agent: {
    heartbeat: { every: "30m" }
  }
}
```

<div id="media-in-and-out">
  ## Medien-Ein- und -Ausgabe
</div>

Eingehende Anhänge (Bilder/Audio/Dokumente) kannst du deinem Befehl über Templates zur Verfügung stellen:

* `{{MediaPath}}` (lokaler temporärer Dateipfad)
* `{{MediaUrl}}` (Pseudo-URL)
* `{{Transcript}}` (falls Audio-Transkription aktiviert ist)

Ausgehende Anhänge des agents: Füge `MEDIA:<path-or-url>` in einer eigenen Zeile ein (keine Leerzeichen). Beispiel:

```
Hier ist der Screenshot.
MEDIA:/tmp/screenshot.png
```

OpenClaw extrahiert sie und sendet sie als Mediendateien zusammen mit dem Text.

<div id="operations-checklist">
  ## Betriebs-Checkliste
</div>

```bash
openclaw status          # lokaler Status (Zugangsdaten, Sitzungen, Ereignisse in der Warteschlange)
openclaw status --all    # vollständige Diagnose (schreibgeschützt, kopierbar)
openclaw status --deep   # fügt Gateway-Statusprüfungen hinzu (Telegram + Discord)
openclaw health --json   # Gateway-Status-Snapshot (WS)
```

Logs befinden sich unter `/tmp/openclaw/` (standardmäßig: `openclaw-YYYY-MM-DD.log`).

<div id="next-steps">
  ## Nächste Schritte
</div>

* WebChat: [WebChat](/de/web/webchat)
* Gateway-Betrieb: [Gateway-Runbook](/de/gateway)
* Cron + Aufwecken: [Cron-Jobs](/de/automation/cron-jobs)
* Begleit-App in der macOS-Menüleiste: [OpenClaw macOS app](/de/platforms/macos)
* iOS-Knoten-App: [iOS app](/de/platforms/ios)
* Android-Knoten-App: [Android app](/de/platforms/android)
* Windows-Status: [Windows (WSL2)](/de/platforms/windows)
* Linux-Status: [Linux app](/de/platforms/linux)
* Sicherheit: [Security](/de/gateway/security)