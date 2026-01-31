---
title: Tools
summary: "Agent-Tool-Oberfläche in OpenClaw (Browser, Canvas, Knoten, Nachrichten, Cron), die die bisherigen `openclaw-*`-Fähigkeiten ersetzt"
read_when:
  - Hinzufügen oder Anpassen von Agent-Tools
  - Ausmustern oder Ändern der `openclaw-*`-Fähigkeiten
---

<div id="tools-openclaw">
  # Tools (OpenClaw)
</div>

OpenClaw stellt **erstklassige Agent-Tools** für Browser, Canvas, Knoten und Cron bereit.
Diese ersetzen die alten `openclaw-*` Fähigkeiten: Die Tools sind typisiert, ohne Shelling,
und der Agent sollte sie direkt verwenden.

<div id="disabling-tools">
  ## Tools deaktivieren
</div>

Du kannst Tools global über `tools.allow` / `tools.deny` in `openclaw.json` zulassen oder verbieten
(`deny` hat Vorrang). Dadurch wird verhindert, dass nicht zugelassene Tools an Modellanbieter gesendet werden.

```json5
{
  tools: { deny: ["browser"] }
}
```

Hinweise:

* Die Übereinstimmung erfolgt ohne Beachtung der Groß- und Kleinschreibung.
* `*`-Wildcards werden unterstützt (`"*"` bedeutet alle Tools).
* Wenn `tools.allow` nur unbekannte oder nicht geladene Plugin-Tool-Namen referenziert, protokolliert OpenClaw eine Warnung und ignoriert die Allowlist, damit Core-Tools verfügbar bleiben.

<div id="tool-profiles-base-allowlist">
  ## Tool-Profile (Basis-Allowlist)
</div>

`tools.profile` setzt eine **Basis-Tool-Allowlist** vor `tools.allow`/`tools.deny`.
Override pro Agent: `agents.list[].tools.profile`.

Profile:

* `minimal`: nur `session_status`
* `coding`: `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
* `messaging`: `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
* `full`: keine Einschränkung (entspricht nicht gesetzt)

Beispiel (standardmäßig nur Messaging, zusätzlich auch Slack- und Discord-Tools erlauben):

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"]
  }
}
```

Beispiel (Coding-Profil, aber `exec`/`process` überall untersagen):

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"]
  }
}
```

Beispiel (globales Coding-Profil, reiner Messaging-Support-agent):

```json5
{
  tools: { profile: "coding" },
  agents: {
    list: [
      {
        id: "support",
        tools: { profile: "messaging", allow: ["slack"] }
      }
    ]
  }
}
```

<div id="provider-specific-tool-policy">
  ## Anbieterspezifische Tool-Richtlinie
</div>

Verwende `tools.byProvider`, um Tools für bestimmte anbieter
(oder ein einzelnes `provider/model`) **weiter einzuschränken**,
ohne deine globalen Defaults zu ändern.
Agent-spezifische Überschreibung: `agents.list[].tools.byProvider`.

Dies wird **nach** dem Basis-Tool-Profil und **vor** Allow-/Deny-Listen angewendet
und kann die Tool-Menge daher nur weiter einschränken.
Anbieter-Schlüssel akzeptieren entweder `provider` (z. B. `google-antigravity`) oder
`provider/model` (z. B. `openai/gpt-5.2`).

Beispiel (globales Coding-Profil beibehalten, aber minimale Tools für Google Antigravity):

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" }
    }
  }
}
```

Beispiel (anbieter- bzw. modellspezifische Allowlist für einen unzuverlässigen Endpunkt):

```json5
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] }
    }
  }
}
```

Beispiel (agent-spezifische Anpassung für einen einzelnen Anbieter):

```json5
{
  agents: {
    list: [
      {
        id: "support",
        tools: {
          byProvider: {
            "google-antigravity": { allow: ["message", "sessions_list"] }
          }
        }
      }
    ]
  }
}
```

<div id="tool-groups-shorthands">
  ## Tool-Gruppen (Kurzformen)
</div>

Tool-Richtlinien (global, agent, sandbox) unterstützen Einträge vom Typ `group:*`, die sich zu mehreren Tools auflösen.
Verwende diese in `tools.allow` / `tools.deny`.

Verfügbare Gruppen:

* `group:runtime`: `exec`, `bash`, `process`
* `group:fs`: `read`, `write`, `edit`, `apply_patch`
* `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory`: `memory_search`, `memory_get`
* `group:web`: `web_search`, `web_fetch`
* `group:ui`: `browser`, `canvas`
* `group:automation`: `cron`, `gateway`
* `group:messaging`: `message`
* `group:nodes`: `nodes`
* `group:openclaw`: alle integrierten OpenClaw-Tools (ohne Anbieter-Plugins)

Beispiel (nur Datei-Tools + Browser erlauben):

```json5
{
  tools: {
    allow: ["group:fs", "browser"]
  }
}
```

<div id="plugins-tools">
  ## Plugins + Tools
</div>

Plugins können **zusätzliche Tools** und CLI-Befehle über den Kernumfang hinaus registrieren.
Siehe [Plugins](/de/plugin) für Installation und Konfiguration sowie [Fähigkeiten](/de/tools/skills) für Details dazu,
wie Hinweise zur Tool-Nutzung in Prompts eingefügt werden. Einige Plugins liefern eigene Fähigkeiten
zusammen mit Tools aus (zum Beispiel das voice-call Plugin).

Optionale Plugin-Tools:

* [Lobster](/de/tools/lobster): typisierte Workflow-Laufzeitumgebung mit wiederaufnehmbaren Genehmigungen (erfordert die Lobster CLI auf dem Gateway-Host).
* [LLM Task](/de/tools/llm-task): ausschließlich JSON-basierter LLM-Schritt für strukturierte Workflow-Ausgaben (optionale Schema-Validierung).

<div id="tool-inventory">
  ## Tool-Übersicht
</div>

<div id="apply_patch">
  ### `apply_patch`
</div>

Wende strukturierte Patches auf eine oder mehrere Dateien an. Verwende dieses Tool für Änderungen mit mehreren Hunks.
Experimentell: aktiviere über `tools.exec.applyPatch.enabled` (nur OpenAI-Modelle).

<div id="exec">
  ### `exec`
</div>

Führe Shell-Befehle im Arbeitsbereich aus.

Kernparameter:

* `command` (erforderlich)
* `yieldMs` (automatisches Verschieben in den Hintergrund nach Timeout, Standardwert 10000)
* `background` (sofortiger Hintergrundmodus)
* `timeout` (Sekunden; beendet den Prozess bei Überschreitung, Standardwert 1800)
* `elevated` (bool; auf dem Host ausführen, wenn erhöhter Modus aktiviert/erlaubt ist; ändert das Verhalten nur, wenn der agent in einer sandbox läuft)
* `host` (`sandbox | gateway | node`)
* `security` (`deny | allowlist | full`)
* `ask` (`off | on-miss | always`)
* `node` (Knoten-ID/-Name für `host=node`)
* Du brauchst ein echtes TTY? Setze `pty: true`.

Hinweise:

* Gibt `status: "running"` mit einer `sessionId` zurück, wenn in den Hintergrund verschoben.
* Verwende `process`, um Hintergrund-Sitzungen abzufragen/protokollieren/zu schreiben/zu beenden/aufzuräumen.
* Wenn `process` nicht erlaubt ist, läuft `exec` synchron und ignoriert `yieldMs`/`background`.
* `elevated` wird durch `tools.elevated` plus alle `agents.list[].tools.elevated`-Overrides gesteuert (beide müssen erlauben) und ist ein Alias für `host=gateway` + `security=full`.
* `elevated` ändert das Verhalten nur, wenn der agent in einer sandbox läuft (ansonsten ohne Effekt/No-Op).
* `host=node` kann eine macOS-Companion-App oder einen Headless-Knotenhost (`openclaw node run`) ansprechen.
* Gateway-/Knoten-Genehmigungen und Allowlists: [Exec approvals](/de/tools/exec-approvals).

<div id="process">
  ### `process`
</div>

Verwalte Hintergrund-Exec-Sitzungen.

Zentrale Aktionen:

* `list`, `poll`, `log`, `write`, `kill`, `clear`, `remove`

Hinweise:

* `poll` gibt neue Ausgabe und den Exit-Status zurück, wenn der Vorgang abgeschlossen ist.
* `log` unterstützt zeilenbasiertes `offset`/`limit` (lasse `offset` weg, um die letzten N Zeilen abzurufen).
* `process` ist pro agent abgegrenzt; Sitzungen anderer Agenten sind nicht sichtbar.

<div id="web_search">
  ### `web_search`
</div>

Durchsuche das Web mit der Brave Search API.

Kernparameter:

* `query` (erforderlich)
* `count` (1–10; Standardwert aus `tools.web.search.maxResults`)

Hinweise:

* Erfordert einen Brave-API-Schlüssel (empfohlen: `openclaw configure --section web` oder `BRAVE_API_KEY` setzen).
* Aktivieren über `tools.web.search.enabled`.
* Antworten werden zwischengespeichert (standardmäßig 15 Minuten).
* Siehe [Web-Tools](/de/tools/web) für die Einrichtung.

<div id="web_fetch">
  ### `web_fetch`
</div>

Ruft lesbare Inhalte von einer URL ab und extrahiert sie (HTML → Markdown/Text).

Kernparameter:

* `url` (erforderlich)
* `extractMode` (`markdown` | `text`)
* `maxChars` (schneidet sehr lange Seiten ab)

Hinweise:

* Aktiviere über `tools.web.fetch.enabled`.
* Responses werden zwischengespeichert (standardmäßig 15 Min.).
* Für JavaScript-lastige Seiten das `browser`-Tool bevorzugen.
* Siehe [Web-Tools](/de/tools/web) für die Einrichtung.
* Siehe [Firecrawl](/de/tools/firecrawl) für das optionale Anti-Bot-Fallback.

<div id="browser">
  ### `browser`
</div>

Steuere den dedizierten, von OpenClaw verwalteten Browser.

Zentrale Aktionen:

* `status`, `start`, `stop`, `tabs`, `open`, `focus`, `close`
* `snapshot` (aria/ai)
* `screenshot` (gibt Bildblock + `MEDIA:<path>` zurück)
* `act` (UI-Aktionen: click/type/press/hover/drag/select/fill/resize/wait/evaluate)
* `navigate`, `console`, `pdf`, `upload`, `dialog`

Profilverwaltung:

* `profiles` — alle Browser-Profile mit Status auflisten
* `create-profile` — neues Profil mit automatisch zugewiesenem Port (oder `cdpUrl`) erstellen
* `delete-profile` — Browser stoppen, Benutzerdaten löschen, aus der Config entfernen (nur lokal)
* `reset-profile` — verwaisten Prozess auf dem Port des Profils beenden (nur lokal)

Gängige Parameter:

* `profile` (optional; Standard ist `browser.defaultProfile`)
* `target` (`sandbox` | `host` | `node`)
* `node` (optional; wählt eine spezifische node-ID bzw. einen node-Namen)

Hinweise:

* Erfordert `browser.enabled=true` (Standard ist `true`; auf `false` setzen zum Deaktivieren).
* Alle Aktionen akzeptieren einen optionalen `profile`-Parameter zur Unterstützung mehrerer Instanzen.
* Wenn `profile` weggelassen wird, wird `browser.defaultProfile` verwendet (Standard ist „chrome“).
* Profilnamen: nur Kleinbuchstaben, Ziffern und Bindestriche (max. 64 Zeichen).
* Portbereich: 18800–18899 (~100 Profile maximal).
* Remote-Profile sind attach-only (kein start/stop/reset).
* Wenn ein browserfähiger node verbunden ist, kann das Tool automatisch dorthin routen (außer du fixierst `target`).
* `snapshot` verwendet standardmäßig `ai`, wenn Playwright installiert ist; verwende `aria` für den Accessibility-Tree.
* `snapshot` unterstützt außerdem Role-Snapshot-Optionen (`interactive`, `compact`, `depth`, `selector`), die Referenzen wie `e12` zurückgeben.
* `act` erfordert eine `ref` aus `snapshot` (numerisches `12` aus AI-Snapshots oder `e12` aus Role-Snapshots); verwende `evaluate` für seltene Fälle, in denen du CSS-Selektoren benötigst.
* Vermeide standardmäßig `act` → `wait`; setze es nur in Ausnahmefällen ein (wenn kein verlässlicher UI-Status zum Warten vorhanden ist).
* `upload` kann optional eine `ref` übergeben, um nach dem Aktivieren automatisch zu klicken.
* `upload` unterstützt außerdem `inputRef` (ARIA-Ref) oder `element` (CSS-Selektor), um `<input type="file">` direkt zu setzen.

<div id="canvas">
  ### `canvas`
</div>

Steuere das Knoten-Canvas (present, eval, snapshot, A2UI).

Kernaktionen:

* `present`, `hide`, `navigate`, `eval`
* `snapshot` (liefert Bildblock + `MEDIA:<path>`)
* `a2ui_push`, `a2ui_reset`

Hinweise:

* Nutzt intern Gateway-`node.invoke`.
* Wenn kein `node` angegeben ist, wählt das Tool einen Standard aus (einzelner verbundener Knoten oder lokaler macOS-Knoten).
* A2UI ist nur in v0.8 verfügbar (kein `createSurface`); die CLI weist v0.9-JSONL mit Zeilenfehlern zurück.
* Schneller Smoke-Test: `openclaw nodes canvas a2ui push --node <id> --text "Hello from A2UI"`.

<div id="nodes">
  ### `nodes`
</div>

Gekoppelte Knoten entdecken und ansteuern; Benachrichtigungen senden; Kamera/Bildschirm erfassen.

Kernaktionen:

* `status`, `describe`
* `pending`, `approve`, `reject` (Kopplung)
* `notify` (macOS `system.notify`)
* `run` (macOS `system.run`)
* `camera_snap`, `camera_clip`, `screen_record`
* `location_get`

Hinweise:

* Kamera-/Bildschirmbefehle erfordern, dass die Knoten-App im Vordergrund ist.
* Bilder liefern Bildblöcke + `MEDIA:<path>`.
* Videos liefern `FILE:<path>` (mp4).
* Standort liefert eine JSON-Payload (lat/lon/accuracy/timestamp).
* `run`-Parameter: `command`-argv-Array; optionale `cwd`, `env` (`KEY=VAL`), `commandTimeoutMs`, `invokeTimeoutMs`, `needsScreenRecording`.

Beispiel (`run`):

```json
{
  "action": "run",
  "node": "office-mac",
  "command": ["echo", "Hello"],
  "env": ["FOO=bar"],
  "commandTimeoutMs": 12000,
  "invokeTimeoutMs": 45000,
  "needsScreenRecording": false
}
```

<div id="image">
  ### `image`
</div>

Analysiere ein Bild mit dem konfigurierten Bildmodell.

Kernparameter:

* `image` (erforderlicher Pfad oder URL)
* `prompt` (optional; Standardwert ist „Describe the image.“)
* `model` (optional, Override)
* `maxBytesMb` (optionale Größenobergrenze)

Hinweise:

* Nur verfügbar, wenn `agents.defaults.imageModel` konfiguriert ist (primär oder mit Fallbacks) oder wenn ein implizites Bildmodell aus deinem Standardmodell plus konfigurierter Auth abgeleitet werden kann (Best-Effort-kopplung).
* Verwendet das Bildmodell direkt (unabhängig vom Haupt-Chatmodell).

<div id="message">
  ### `message`
</div>

Sende Nachrichten und Kanalaktionen über Discord/Google Chat/Slack/Telegram/WhatsApp/Signal/iMessage/MS Teams.

Kernaktionen:

* `send` (Text + optionale Medien; MS Teams unterstützt außerdem `card` für Adaptive Cards)
* `poll` (WhatsApp-/Discord-/MS-Teams-Umfragen)
* `react` / `reactions` / `read` / `edit` / `delete`
* `pin` / `unpin` / `list-pins`
* `permissions`
* `thread-create` / `thread-list` / `thread-reply`
* `search`
* `sticker`
* `member-info` / `role-info`
* `emoji-list` / `emoji-upload` / `sticker-upload`
* `role-add` / `role-remove`
* `channel-info` / `channel-list`
* `voice-status`
* `event-list` / `event-create`
* `timeout` / `kick` / `ban`

Hinweise:

* `send` routet WhatsApp über das Gateway; andere Kanäle werden direkt angesprochen.
* `poll` nutzt das Gateway für WhatsApp und MS Teams; Discord-Umfragen gehen direkt.
* Wenn ein `message`-Toolaufruf an eine aktive Chat-Sitzung gebunden ist, werden `send`-Aufrufe auf das Ziel dieser Sitzung beschränkt, um Cross-Context-Leaks zu vermeiden.

<div id="cron">
  ### `cron`
</div>

Verwaltung von Gateway-Cronjobs und Wakeups.

Kernaktionen:

* `status`, `list`
* `add`, `update`, `remove`, `run`, `runs`
* `wake` (Systemereignis in die Warteschlange stellen + optional mit sofortigem Herzschlag)

Hinweise:

* `add` erwartet ein vollständiges Cronjob-Objekt (dasselbe Schema wie beim `cron.add`-RPC).
* `update` verwendet `{ id, patch }`.

<div id="gateway">
  ### `gateway`
</div>

Neustarten oder Updates auf den laufenden Gateway-Prozess anwenden (in-place).

Zentrale Aktionen:

* `restart` (führt Autorisierung durch + sendet `SIGUSR1` für einen Neustart im selben Prozess; `openclaw gateway`-Neustart in-place)
* `config.get` / `config.schema`
* `config.apply` (Konfiguration validieren + schreiben + neu starten + aufwecken)
* `config.patch` (partielles Update zusammenführen + neu starten + aufwecken)
* `update.run` (Update ausführen + neu starten + aufwecken)

Hinweise:

* Verwende `delayMs` (Standard ist 2000), um zu vermeiden, dass eine laufende Antwort unterbrochen wird.
* `restart` ist standardmäßig deaktiviert; aktiviere es mit `commands.restart: true`.

<div id="sessions_list-sessions_history-sessions_send-sessions_spawn-session_status">
  ### `sessions_list` / `sessions_history` / `sessions_send` / `sessions_spawn` / `session_status`
</div>

Sitzungen auflisten, Transkripthistorie einsehen oder an eine andere Sitzung senden.

Zentrale Parameter:

* `sessions_list`: `kinds?`, `limit?`, `activeMinutes?`, `messageLimit?` (0 = keine Begrenzung)
* `sessions_history`: `sessionKey` (oder `sessionId`), `limit?`, `includeTools?`
* `sessions_send`: `sessionKey` (oder `sessionId`), `message`, `timeoutSeconds?` (0 = fire-and-forget)
* `sessions_spawn`: `task`, `label?`, `agentId?`, `model?`, `runTimeoutSeconds?`, `cleanup?`
* `session_status`: `sessionKey?` (Standard ist die aktuelle Sitzung; akzeptiert `sessionId`), `model?` (`default` hebt eine Überschreibung auf)

Hinweise:

* `main` ist der kanonische Direkt‑Chat‑Schlüssel; global/unknown sind verborgen.
* `messageLimit > 0` holt die letzten N Nachrichten pro Sitzung (Tool‑Nachrichten werden gefiltert).
* `sessions_send` wartet auf den endgültigen Abschluss, wenn `timeoutSeconds > 0`.
* Zustellung/Announce erfolgt nach Abschluss und ist Best‑Effort; `status: "ok"` bestätigt, dass der Agent‑Lauf beendet ist, nicht dass das Announce zugestellt wurde.
* `sessions_spawn` startet einen untergeordneten Agent‑Lauf und postet eine Announce‑Antwort zurück in den anfragenden Chat.
* `sessions_spawn` ist nicht blockierend und gibt sofort `status: "accepted"` zurück.
* `sessions_send` führt einen Reply‑Back‑Ping‑Pong aus (antworte mit `REPLY_SKIP`, um zu stoppen; maximale Anzahl an Turns über `session.agentToAgent.maxPingPongTurns`, 0–5).
* Nach dem Ping‑Pong führt der Ziel‑Agent einen **Announce‑Schritt** aus; antworte mit `ANNOUNCE_SKIP`, um das Announcement zu unterdrücken.

<div id="agents_list">
  ### `agents_list`
</div>

Listet die Agent-IDs auf, die die aktuelle Sitzung mit `sessions_spawn` ansprechen darf.

Hinweise:

* Das Ergebnis ist auf per-Agent-Allowlists (`agents.list[].subagents.allowAgents`) beschränkt.
* Wenn `["*"]` konfiguriert ist, umfasst das Tool alle konfigurierten Agenten und setzt `allowAny: true`.

<div id="parameters-common">
  ## Parameter (allgemein)
</div>

Gateway-basierte Tools (`canvas`, `nodes`, `cron`):

* `gatewayUrl` (Standardwert: `ws://127.0.0.1:18789`)
* `gatewayToken` (falls Authentifizierung aktiviert ist)
* `timeoutMs`

Browser-Tool:

* `profile` (optional; Standard ist `browser.defaultProfile`)
* `target` (`sandbox` | `host` | `node`)
* `node` (optional; eine bestimmte Knoten-ID bzw. einen Namen festlegen)

<div id="recommended-agent-flows">
  ## Empfohlene Agent-Flows
</div>

Browser-Automatisierung:

1. `browser` → `status` / `start`
2. `snapshot` (ai oder aria)
3. `act` (click/type/press)
4. `screenshot`, wenn du eine visuelle Bestätigung benötigst

Canvas-Rendering:

1. `canvas` → `present`
2. `a2ui_push` (optional)
3. `snapshot`

Knoten-Ansteuerung:

1. `nodes` → `status`
2. `describe` für den ausgewählten Knoten
3. `notify` / `run` / `camera_snap` / `screen_record`

<div id="safety">
  ## Sicherheit
</div>

* Vermeide direkten Aufruf von `system.run`; führe `nodes` → `run` nur mit ausdrücklicher Zustimmung der Nutzer aus.
* Respektiere die Einwilligung der Nutzer für Kamera- und Bildschirmaufnahmen.
* Verwende `status/describe`, um Berechtigungen zu verifizieren, bevor du Medienbefehle ausführst.

<div id="how-tools-are-presented-to-the-agent">
  ## Wie Tools dem agent präsentiert werden
</div>

Tools werden in zwei parallelen Kanälen bereitgestellt:

1. **System-Prompt-Text**: eine menschenlesbare Liste plus Hinweise.
2. **Tool-Schema**: die strukturierten Funktionsdefinitionen, die an die Modell-API gesendet werden.

Das bedeutet, der agent sieht sowohl „welche Tools existieren“ als auch „wie man sie aufruft“. Wenn ein Tool
nicht im System-Prompt oder im Schema vorkommt, kann das Modell es nicht aufrufen.