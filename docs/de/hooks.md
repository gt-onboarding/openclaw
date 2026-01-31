---
title: Hooks
summary: "Hooks: Ereignisgesteuerte Automatisierung für Befehle und Lebenszyklusereignisse"
read_when:
  - Du möchtest ereignisgesteuerte Automatisierung für /new, /reset, /stop und Lebenszyklusereignisse von agents
  - Du möchtest Hooks erstellen, installieren oder debuggen
---

<div id="hooks">
  # Hooks
</div>

Hooks stellen ein erweiterbares, ereignisgesteuertes System bereit, um Aktionen als Reaktion auf Agent-Befehle und ‑Ereignisse zu automatisieren. Hooks werden automatisch in Verzeichnissen gefunden und können über CLI-Befehle verwaltet werden, ähnlich wie Fähigkeiten in OpenClaw gehandhabt werden.

<div id="getting-oriented">
  ## Orientierung
</div>

Hooks sind kleine Skripte, die ausgeführt werden, wenn etwas passiert. Es gibt zwei Arten:

* **Hooks** (diese Seite): laufen im Gateway, wenn agent-Ereignisse ausgelöst werden, z. B. `/new`, `/reset`, `/stop` oder Lifecycle-Ereignisse.
* **Webhooks**: externe HTTP-Webhooks, mit denen andere Systeme Arbeit in OpenClaw auslösen können. Siehe [Webhook Hooks](/de/automation/webhook) oder verwende `openclaw webhooks` für Gmail-Hilfsbefehle.

Hooks können auch in Plugins gebündelt sein; siehe [Plugins](/de/plugin#plugin-hooks).

Typische Anwendungsfälle:

* Einen Memory-Snapshot speichern, wenn du eine Sitzung zurücksetzt
* Einen Audit-Trail von Befehlen für Troubleshooting oder Compliance führen
* Nachgelagerte Automatisierung auslösen, wenn eine Sitzung startet oder endet
* Dateien in den Arbeitsbereich des agent schreiben oder externe APIs aufrufen, wenn Ereignisse eintreten

Wenn du eine kleine TypeScript-Funktion schreiben kannst, kannst du auch einen Hook schreiben. Hooks werden automatisch erkannt, und du aktivierst oder deaktivierst sie über die CLI.

<div id="overview">
  ## Übersicht
</div>

Das Hooks-System ermöglicht dir:

* Sitzungs­kontext beim Ausführen von `/new` im Speicher zu speichern
* Alle Befehle für Audits zu protokollieren
* Benutzerdefinierte Automatisierungen bei Agent-Lebenszyklus-Ereignissen auszulösen
* OpenClaws Verhalten zu erweitern, ohne den Kerncode zu ändern

<div id="getting-started">
  ## Erste Schritte
</div>

<div id="bundled-hooks">
  ### Mitgelieferte Hooks
</div>

OpenClaw wird mit vier Hooks ausgeliefert, die automatisch erkannt werden:

* **💾 session-memory**: Speichert Sitzungs­kontext in deinem agent-arbeitsbereich (Standard `~/.openclaw/workspace/memory/`), wenn du `/new` ausführst
* **📝 command-logger**: Protokolliert alle Befehlsereignisse nach `~/.openclaw/logs/commands.log`
* **🚀 boot-md**: Führt `BOOT.md` aus, wenn das Gateway startet (erfordert aktivierte interne Hooks)
* **😈 soul-evil**: Tauscht injizierten `SOUL.md`-Inhalt mit `SOUL_EVIL.md` während eines Purge-Fensters oder nach dem Zufallsprinzip aus

Verfügbare Hooks auflisten:

```bash
openclaw hooks list
```

Einen Hook aktivieren:

```bash
openclaw hooks enable session-memory
```

Hook-Status überprüfen:

```bash
openclaw hooks check
```

Detaillierte Informationen abrufen:

```bash
openclaw hooks info session-memory
```

<div id="onboarding">
  ### Onboarding
</div>

Während des Onboarding-Vorgangs (`openclaw onboard`) wirst du aufgefordert, empfohlene Hooks zu aktivieren. Der Assistent ermittelt automatisch passende Hooks und bietet sie zur Auswahl an.

<div id="hook-discovery">
  ## Hook-Erkennung
</div>

Hooks werden automatisch aus drei Verzeichnissen erkannt (in absteigender Priorität):

1. **Arbeitsbereich-Hooks**: `<workspace>/hooks/` (agent-spezifisch, höchste Priorität)
2. **Verwaltete Hooks**: `~/.openclaw/hooks/` (vom Benutzer installiert, zwischen Arbeitsbereichen geteilt)
3. **Mitgelieferte Hooks**: `<openclaw>/dist/hooks/bundled/` (mit OpenClaw ausgeliefert)

Verwaltete Hook-Verzeichnisse können entweder ein **einzelner Hook** oder ein **Hook-Pack** (Paketverzeichnis) sein.

Jeder Hook ist ein Verzeichnis, das Folgendes enthält:

```
my-hook/
├── HOOK.md          # Metadaten + Dokumentation
└── handler.ts       # Handler-Implementierung
```

<div id="hook-packs-npmarchives">
  ## Hook-Pakete (npm/Archive)
</div>

Hook-Pakete sind Standard-npm-Pakete, die einen oder mehrere Hooks über `openclaw.hooks` in
`package.json` exportieren. Installieren Sie sie mit:

```bash
openclaw hooks install <path-or-spec>
```

Beispiel für `package.json`:

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "openclaw": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

Jeder Eintrag verweist auf ein Hook-Verzeichnis, das `HOOK.md` und `handler.ts` (oder `index.ts`) enthält.
Hook-Pakete können Abhängigkeiten mitliefern; diese werden unter `~/.openclaw/hooks/<id>` installiert.

<div id="hook-structure">
  ## Struktur von Hooks
</div>

<div id="hookmd-format">
  ### HOOK.md-Format
</div>

Die Datei `HOOK.md` enthält Metadaten als YAML-Frontmatter sowie eine Markdown-Dokumentation:

```markdown
---
name: my-hook
description: "Short description of what this hook does"
homepage: https://docs.openclaw.ai/hooks#my-hook
metadata: {"openclaw":{"emoji":"🔗","events":["command:new"],"requires":{"bins":["node"]}}}
---

# My Hook

Detailed documentation goes here...

## What It Does

- Listens for `/new` commands
- Performs some action
- Logs the result

## Requirements

- Node.js must be installed

## Configuration

No configuration needed.
```

<div id="metadata-fields">
  ### Metadatenfelder
</div>

Das Objekt `metadata.openclaw` unterstützt:

* **`emoji`**: Anzeige-Emoji für die CLI (z. B. `"💾"`)
* **`events`**: Array von Ereignissen, die abonniert werden (z. B. `["command:new", "command:reset"]`)
* **`export`**: Zu verwendender benannter Export (Standardwert ist `"default"`)
* **`homepage`**: Dokumentations-URL
* **`requires`**: Optionale Anforderungen
  * **`bins`**: Erforderliche Binaries im PATH (z. B. `["git", "node"]`)
  * **`anyBins`**: Mindestens eines dieser Binaries muss vorhanden sein
  * **`env`**: Erforderliche Umgebungsvariablen
  * **`config`**: Erforderliche Konfigurationspfade (z. B. `["workspace.dir"]`)
  * **`os`**: Erforderliche Plattformen (z. B. `["darwin", "linux"]`)
* **`always`**: Eignungsprüfungen überspringen (Boolean)
* **`install`**: Installationsmethoden (für gebündelte Hooks: `[{"id":"bundled","kind":"bundled"}]`)

<div id="handler-implementation">
  ### Handler-Implementierung
</div>

Die Datei `handler.ts` exportiert eine Funktion `HookHandler`:

```typescript
import type { HookHandler } from '../../src/hooks/hooks.js';

const myHandler: HookHandler = async (event) => {
  // Nur bei 'new'-Befehl auslösen
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  console.log(`[my-hook] New command triggered`);
  console.log(`  Sitzung: ${event.sessionKey}`);
  console.log(`  Timestamp: ${event.timestamp.toISOString()}`);

  // Ihre benutzerdefinierte Logik hier

  // Optional: Nachricht an Benutzer senden
  event.messages.push('✨ My hook executed!');
};

export default myHandler;
```

<div id="event-context">
  #### Ereigniskontext
</div>

Jedes Ereignis umfasst:

```typescript
{
  type: 'command' | 'session' | 'agent' | 'gateway',
  action: string,              // e.g., 'new', 'reset', 'stop'
  sessionKey: string,          // Session identifier
  timestamp: Date,             // When the event occurred
  messages: string[],          // Nachrichten hier hinzufügen, um sie an den Benutzer zu senden
  context: {
    sessionEntry?: SessionEntry,
    sessionId?: string,
    sessionFile?: string,
    commandSource?: string,    // e.g., 'whatsapp', 'telegram'
    senderId?: string,
    workspaceDir?: string,
    bootstrapFiles?: WorkspaceBootstrapFile[],
    cfg?: OpenClawConfig
  }
}
```

<div id="event-types">
  ## Event-Typen
</div>

<div id="command-events">
  ### Befehlsereignisse
</div>

Ausgelöst, wenn agent-Befehle abgesetzt werden:

* **`command`**: Alle Befehlsereignisse (allgemeiner Listener)
* **`command:new`**: Wenn der Befehl `/new` abgesetzt wird
* **`command:reset`**: Wenn der Befehl `/reset` abgesetzt wird
* **`command:stop`**: Wenn der Befehl `/stop` abgesetzt wird

<div id="agent-events">
  ### Agent-Ereignisse
</div>

* **`agent:bootstrap`**: Bevor Bootstrap-Dateien für den Arbeitsbereich eingefügt werden (Hooks können `context.bootstrapFiles` modifizieren)

<div id="gateway-events">
  ### Gateway-Ereignisse
</div>

Ausgelöst, wenn das Gateway startet:

* **`gateway:startup`**: Nachdem die Kanäle gestartet wurden und Hooks geladen sind

<div id="tool-result-hooks-plugin-api">
  ### Toolergebnis-Hooks (Plugin-API)
</div>

Diese Hooks sind keine Event-Stream-Listener; sie ermöglichen Plugins, Toolergebnisse synchron anzupassen, bevor OpenClaw sie speichert.

* **`tool_result_persist`**: transformiert Toolergebnisse, bevor sie in das Sitzungstranskript geschrieben werden. Muss synchron sein; gibt die aktualisierte Toolergebnis-Payload zurück oder `undefined`, um sie unverändert zu lassen. Siehe [Agent Loop](/de/concepts/agent-loop).

<div id="future-events">
  ### Zukünftige Ereignisse
</div>

Geplante Ereignistypen:

* **`session:start`**: Wenn eine neue Sitzung beginnt
* **`session:end`**: Wenn eine Sitzung endet
* **`agent:error`**: Wenn ein Agent auf einen Fehler stößt
* **`message:sent`**: Wenn eine Nachricht gesendet wird
* **`message:received`**: Wenn eine Nachricht empfangen wird

<div id="creating-custom-hooks">
  ## Eigene Hooks erstellen
</div>

<div id="1-choose-location">
  ### 1. Speicherort wählen
</div>

* **Arbeitsbereich-Hooks** (`<workspace>/hooks/`): Pro Agent, höchste Priorität
* **Verwaltete Hooks** (`~/.openclaw/hooks/`): Gemeinsam für alle Arbeitsbereiche

<div id="2-create-directory-structure">
  ### 2. Verzeichnisstruktur anlegen
</div>

```bash
mkdir -p ~/.openclaw/hooks/my-hook
cd ~/.openclaw/hooks/my-hook
```

<div id="3-create-hookmd">
  ### 3. HOOK.md erstellen
</div>

```markdown
---
name: my-hook
description: "Does something useful"
metadata: {"openclaw":{"emoji":"🎯","events":["command:new"]}}
---

# My Custom Hook

This hook does something useful when you issue `/new`.
```

<div id="4-create-handlerts">
  ### 4. handler.ts erstellen
</div>

```typescript
import type { HookHandler } from '../../src/hooks/hooks.js';

const handler: HookHandler = async (event) => {
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  console.log('[my-hook] Running!');
  // Deine Logik hier
};

export default handler;
```

<div id="5-enable-and-test">
  ### 5. Aktivieren und Testen
</div>

```bash
# Hook-Erkennung überprüfen
openclaw hooks list

# Hook aktivieren
openclaw hooks enable my-hook

# Gateway-Prozess neu starten (Neustart der Menüleisten-App unter macOS oder Neustart des Dev-Prozesses)

# Event auslösen
# /new über den Messaging-Kanal senden
```

<div id="configuration">
  ## Konfiguration
</div>

<div id="new-config-format-recommended">
  ### Neues Konfigurationsformat (empfohlen)
</div>

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "session-memory": { "enabled": true },
        "command-logger": { "enabled": false }
      }
    }
  }
}
```

<div id="per-hook-configuration">
  ### Hook-spezifische Konfiguration
</div>

Hooks können eine individuelle Konfiguration haben:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "my-hook": {
          "enabled": true,
          "env": {
            "MY_CUSTOM_VAR": "value"
          }
        }
      }
    }
  }
}
```

<div id="extra-directories">
  ### Zusätzliche Verzeichnisse
</div>

Hooks aus zusätzlichen Verzeichnissen laden:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "load": {
        "extraDirs": ["/path/to/more/hooks"]
      }
    }
  }
}
```

<div id="legacy-config-format-still-supported">
  ### Legacy-Konfigurationsformat (weiterhin unterstützt)
</div>

Das alte Konfigurationsformat funktioniert weiterhin aus Gründen der Abwärtskompatibilität:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts",
          "export": "default"
        }
      ]
    }
  }
}
```

**Migration**: Verwenden Sie für neue Hooks das neue Discovery-basierte System. Legacy-Handler werden nach den verzeichnisbasierten Hooks geladen.

<div id="cli-commands">
  ## CLI-Befehle
</div>

<div id="list-hooks">
  ### Hooks auflisten
</div>

```bash
# List all hooks
openclaw hooks list

# Show only eligible hooks
openclaw hooks list --eligible

# Ausführliche Ausgabe (fehlende Anforderungen anzeigen)
openclaw hooks list --verbose

# JSON output
openclaw hooks list --json
```

<div id="hook-information">
  ### Informationen zum Hook
</div>

```bash
# Detaillierte Informationen über einen Hook anzeigen
openclaw hooks info session-memory

# JSON output
openclaw hooks info session-memory --json
```

<div id="check-eligibility">
  ### Voraussetzungen prüfen
</div>

```bash
# Berechtigungsübersicht anzeigen
openclaw hooks check

# JSON output
openclaw hooks check --json
```

<div id="enabledisable">
  ### Aktivieren/Deaktivieren
</div>

```bash
# Einen Hook aktivieren
openclaw hooks enable session-memory

# Einen Hook deaktivieren
openclaw hooks disable command-logger
```

## Integrierte Hooks

<div id="session-memory">
  ### session-memory
</div>

Speichert den Sitzungskontext im Memory-Verzeichnis, wenn du den Befehl `/new` ausführst.

**Events**: `command:new`

**Requirements**: `workspace.dir` muss konfiguriert sein

**Output**: `<workspace>/memory/YYYY-MM-DD-slug.md` (Standard: `~/.openclaw/workspace`)

**What it does**:

1. Verwendet den Sitzungs-Eintrag vor dem Reset, um das richtige Transkript zu finden
2. Extrahiert die letzten 15 Zeilen der Unterhaltung
3. Verwendet ein LLM, um einen beschreibenden Dateinamen-Slug zu erzeugen
4. Speichert Sitzungsmetadaten in einer datierten Memory-Datei

**Example output**:

```markdown
# Sitzung: 2026-01-16 14:30:00 UTC

- **Session Key**: agent:main:main
- **Session ID**: abc123def456
- **Source**: telegram
```

**Beispiele für Dateinamen**:

* `2026-01-16-vendor-pitch.md`
* `2026-01-16-api-design.md`
* `2026-01-16-1430.md` (Fallback-Zeitstempel, falls die Slug-Generierung fehlschlägt)

**Aktivieren**:

```bash
openclaw hooks enable session-memory
```

<div id="command-logger">
  ### command-logger
</div>

Protokolliert alle Befehlsereignisse in eine zentrale Audit-Datei.

**Ereignisse**: `command`

**Voraussetzungen**: Keine

**Ausgabe**: `~/.openclaw/logs/commands.log`

**Was er macht**:

1. Erfasst Ereignisdetails (Befehlsaktion, Zeitstempel, Sitzungsschlüssel, Absender-ID, Quelle)
2. Hängt Einträge im JSONL-Format an die Logdatei an
3. Läuft geräuschlos im Hintergrund

**Beispiel-Logeinträge**:

```jsonl
{"timestamp":"2026-01-16T14:30:00.000Z","action":"new","sessionKey":"agent:main:main","senderId":"+1234567890","source":"telegram"}
{"timestamp":"2026-01-16T15:45:22.000Z","action":"stop","sessionKey":"agent:main:main","senderId":"user@example.com","source":"whatsapp"}
```

**Protokolle anzeigen**:

```bash
# View recent commands
tail -n 20 ~/.openclaw/logs/commands.log

# Formatierte Ausgabe mit jq
cat ~/.openclaw/logs/commands.log | jq .

# Filter by action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**Aktivieren**:

```bash
openclaw hooks enable command-logger
```

<div id="soul-evil">
  ### soul-evil
</div>

Tauscht den injizierten `SOUL.md`‑Inhalt während eines Bereinigungsfensters oder zufällig mit `SOUL_EVIL.md` aus.

**Events**: `agent:bootstrap`

**Docs**: [SOUL Evil Hook](/de/hooks/soul-evil)

**Output**: Es werden keine Dateien geschrieben; die Ersetzungen erfolgen ausschließlich im Speicher.

**Enable**:

```bash
openclaw hooks enable soul-evil
```

**Config**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 0.1,
          "purge": { "at": "21:00", "duration": "15m" }
        }
      }
    }
  }
}
```

<div id="boot-md">
  ### boot-md
</div>

Führt `BOOT.md` aus, wenn das Gateway startet (nachdem die Kanäle gestartet wurden).
Interne Hooks müssen aktiviert sein, damit dies ausgeführt wird.

**Events**: `gateway:startup`

**Requirements**: `workspace.dir` muss konfiguriert sein

**What it does**:

1. Liest `BOOT.md` aus deinem Arbeitsbereich
2. Führt die Anweisungen über den Agent-Runner aus
3. Sendet alle angeforderten ausgehenden Nachrichten über das Message-Tool

**Enable**:

```bash
openclaw hooks enable boot-md
```

<div id="best-practices">
  ## Best Practices
</div>

<div id="keep-handlers-fast">
  ### Handler schlank halten
</div>

Hooks werden während der Befehlsverarbeitung ausgeführt. Halte sie möglichst leichtgewichtig:

```typescript
// ✓ Gut - asynchrone Arbeit, kehrt sofort zurück
const handler: HookHandler = async (event) => {
  void processInBackground(event); // Fire and forget
};

// ✗ Bad - blocks command processing
const handler: HookHandler = async (event) => {
  await slowDatabaseQuery(event);
  await evenSlowerAPICall(event);
};
```

<div id="handle-errors-gracefully">
  ### Fehler robust behandeln
</div>

Kapsle riskante Operationen immer ein:

```typescript
const handler: HookHandler = async (event) => {
  try {
    await riskyOperation(event);
  } catch (err) {
    console.error('[my-handler] Failed:', err instanceof Error ? err.message : String(err));
    // Nicht werfen – andere Handler laufen lassen
  }
};
```

<div id="filter-events-early">
  ### Ereignisse frühzeitig filtern
</div>

Gib frühzeitig zurück, wenn das Ereignis nicht relevant ist:

```typescript
const handler: HookHandler = async (event) => {
  // Nur 'new'-Befehle verarbeiten
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  // Your logic here
};
```

<div id="use-specific-event-keys">
  ### Spezifische Ereignisschlüssel verwenden
</div>

Gib nach Möglichkeit konkrete Ereignisse in den Metadaten an:

```yaml
metadata: {"openclaw":{"events":["command:new"]}}  # Spezifisch
```

Statt:

```yaml
metadata: {"openclaw":{"events":["command"]}}      # Allgemein - mehr Overhead
```

<div id="debugging">
  ## Debugging
</div>

<div id="enable-hook-logging">
  ### Hook-Logging aktivieren
</div>

Das Gateway protokolliert das Laden der Hooks beim Start:

```
Registered hook: session-memory -> command:new
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

<div id="check-discovery">
  ### Discovery prüfen
</div>

Liste alle gefundenen Hooks auf:

```bash
openclaw hooks list --verbose
```

<div id="check-registration">
  ### Registrierung überprüfen
</div>

Logge in deinem Handler, wenn er aufgerufen wird:

```typescript
const handler: HookHandler = async (event) => {
  console.log('[my-handler] Triggered:', event.type, event.action);
  // Ihre Logik hier
};
```

<div id="verify-eligibility">
  ### Berechtigung prüfen
</div>

Prüfe, warum ein Hook nicht zulässig ist:

```bash
openclaw hooks info my-hook
```

Prüfe die Ausgabe auf fehlende Anforderungen.

<div id="testing">
  ## Testen
</div>

<div id="gateway-logs">
  ### Gateway-Logs
</div>

Überwache die Gateway-Logs, um die Ausführung von Hooks nachzuvollziehen:

```bash
# macOS
./scripts/clawlog.sh -f

# Andere Plattformen
tail -f ~/.openclaw/gateway.log
```

<div id="test-hooks-directly">
  ### Hooks direkt testen
</div>

Handler isoliert testen:

```typescript
import { test } from 'vitest';
import { createHookEvent } from './src/hooks/hooks.js';
import myHandler from './hooks/my-hook/handler.js';

test('my handler works', async () => {
  const event = createHookEvent('command', 'new', 'test-session', {
    foo: 'bar'
  });

  await myHandler(event);

  // Seiteneffekte prüfen
});
```

<div id="architecture">
  ## Architektur
</div>

<div id="core-components">
  ### Zentrale Komponenten
</div>

* **`src/hooks/types.ts`**: Typdefinitionen
* **`src/hooks/workspace.ts`**: Verzeichnisse scannen und laden
* **`src/hooks/frontmatter.ts`**: Parsen der HOOK.md-Metadaten
* **`src/hooks/config.ts`**: Prüfung der Ausführungsbedingungen
* **`src/hooks/hooks-status.ts`**: Status-Reporting
* **`src/hooks/loader.ts`**: Dynamischer Modullader
* **`src/cli/hooks-cli.ts`**: CLI-Befehle
* **`src/gateway/server-startup.ts`**: Lädt Hooks beim Gateway-Start
* **`src/auto-reply/reply/commands-core.ts`**: Löst Befehlsereignisse aus

<div id="discovery-flow">
  ### Discovery-Ablauf
</div>

```
Gateway startup
    ↓
Scan directories (arbeitsbereich → managed → bundled)
    ↓
Parse HOOK.md files
    ↓
Check eligibility (bins, env, config, os)
    ↓
Load handlers from eligible hooks
    ↓
Register handlers for events
```

<div id="event-flow">
  ### Ereignisablauf
</div>

```
User sends /new
    ↓
Command validation
    ↓
Create hook event
    ↓
Trigger hook (all registered handlers)
    ↓
Command processing continues
    ↓
Session reset
```

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

<div id="hook-not-discovered">
  ### Hook nicht gefunden
</div>

1. Verzeichnisstruktur prüfen:
   ```bash
   ls -la ~/.openclaw/hooks/my-hook/
   # Sollte anzeigen: HOOK.md, handler.ts
   ```

2. HOOK.md-Format überprüfen:
   ```bash
   cat ~/.openclaw/hooks/my-hook/HOOK.md
   # Sollte YAML-Frontmatter mit Name und Metadaten enthalten
   ```

3. Alle erkannten Hooks auflisten:
   ```bash
   openclaw hooks list
   ```

<div id="hook-not-eligible">
  ### Hook nicht zulässig
</div>

Prüfen Sie die Voraussetzungen:

```bash
openclaw hooks info my-hook
```

Prüfe, ob Folgendes fehlt:

* Binaries (PATH prüfen)
* Umgebungsvariablen
* Konfigurationswerte
* Kompatibilität mit dem Betriebssystem

<div id="hook-not-executing">
  ### Hook wird nicht ausgeführt
</div>

1. Überprüfe, ob der Hook aktiviert ist:
   ```bash
   openclaw hooks list
   # Sollte ein ✓ neben aktivierten Hooks anzeigen
   ```

2. Starte deinen Gateway-Prozess neu, damit die Hooks neu geladen werden.

3. Prüfe die Gateway-Logs auf Fehler:
   ```bash
   ./scripts/clawlog.sh | grep hook
   ```

<div id="handler-errors">
  ### Handler-Fehler
</div>

Prüfen Sie auf TypeScript- oder Import-Fehler:

```bash
# Import direkt testen
node -e "import('./path/to/handler.ts').then(console.log)"
```

<div id="migration-guide">
  ## Migrationsleitfaden
</div>

<div id="from-legacy-config-to-discovery">
  ### Von Legacy-Konfiguration zu Discovery
</div>

**Vorher**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts"
        }
      ]
    }
  }
}
```

**Nachher**:

1. Hook-Verzeichnis erstellen:
   ```bash
   mkdir -p ~/.openclaw/hooks/my-hook
   mv ./hooks/handlers/my-handler.ts ~/.openclaw/hooks/my-hook/handler.ts
   ```

2. HOOK.md erstellen:
   ```markdown
   ---
   name: my-hook
   description: "My custom hook"
   metadata: {"openclaw":{"emoji":"🎯","events":["command:new"]}}
   ---

   # My Hook

   Führt etwas Nützliches aus.
   ```

3. Konfiguration aktualisieren:
   ```json
   {
     "hooks": {
       "internal": {
         "enabled": true,
         "entries": {
           "my-hook": { "enabled": true }
         }
       }
     }
   }
   ```

4. Gateway-Prozess prüfen und neu starten:
   ```bash
   openclaw hooks list
   # Sollte anzeigen: 🎯 my-hook ✓
   ```

**Vorteile der Migration**:

* Automatische Erkennung
* CLI-Verwaltung
* Eignungsprüfung
* Bessere Dokumentation
* Konsistente Struktur

<div id="see-also">
  ## Siehe auch
</div>

* [CLI-Referenz: Hooks](/de/cli/hooks)
* [Bundled Hooks-README](https://github.com/openclaw/openclaw/tree/main/src/hooks/bundled)
* [Webhook-Hooks](/de/automation/webhook)
* [Konfiguration](/de/gateway/configuration#hooks)