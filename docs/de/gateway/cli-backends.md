---
title: CLI-Backends
summary: "CLI-Backends: rein textbasierte Fallback-Option über lokale KI-CLIs"
read_when:
  - du eine zuverlässige Fallback-Ebene brauchst, wenn API-Anbieter ausfallen
  - du die Claude Code CLI oder andere lokale KI-CLIs ausführst und sie wiederverwenden möchtest
  - du einen rein textbasierten, tool-freien Weg brauchst, der trotzdem Sitzungen und Bilder unterstützt
---

<div id="cli-backends-fallback-runtime">
  # CLI-Backends (Fallback-Runtime)
</div>

OpenClaw kann **lokale KI-CLIs** als **rein textbasierten Fallback** ausführen, wenn API-Anbieter ausfallen, ratebegrenzt sind oder sich vorübergehend fehlerhaft verhalten. Dieses Verhalten ist bewusst konservativ:

- **Tools sind deaktiviert** (keine Tool-Aufrufe).
- **Text rein → Text raus** (zuverlässig).
- **Sitzungen werden unterstützt** (damit spätere Nachrichten kohärent bleiben).
- **Bilder können durchgereicht werden**, wenn die CLI Bildpfade akzeptiert.

Dies ist als **Sicherheitsnetz** und nicht als primärer Ausführungsweg gedacht. Verwende es, wenn du „funktioniert immer“-Textantworten möchtest, ohne dich auf externe APIs zu verlassen.

<div id="beginner-friendly-quick-start">
  ## Schnellstart für Einsteiger
</div>

Du kannst Claude Code CLI **ohne Konfiguration** verwenden (OpenClaw bringt eine integrierte Standardkonfiguration mit):

```bash
openclaw agent --message "hi" --model claude-cli/opus-4.5
```

Codex CLI funktioniert ebenfalls out of the box:

```bash
openclaw agent --message "hi" --model codex-cli/gpt-5.2-codex
```

Wenn dein Gateway unter launchd/systemd läuft und PATH nur minimal gesetzt ist, füge nur den Pfad zum Befehl hinzu:

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude"
        }
      }
    }
  }
}
```

Das war’s. Keine Keys, keine zusätzliche Authentifizierungskonfiguration erforderlich, außer der CLI selbst.


<div id="using-it-as-a-fallback">
  ## Verwendung als Fallback
</div>

Füge ein CLI-Backend zu deiner Fallback-Liste hinzu, sodass es nur ausgeführt wird, wenn die primären Modelle ausfallen:

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: [
          "claude-cli/opus-4.5"
        ]
      },
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "claude-cli/opus-4.5": {}
      }
    }
  }
}
```

Hinweise:

* Wenn du `agents.defaults.models` (Allowlist) verwendest, musst du `claude-cli/...` angeben.
* Wenn der primäre Anbieter fehlschlägt (Authentifizierung, Rate-Limits, Timeouts), versucht OpenClaw
  anschließend, das CLI-Backend zu verwenden.


<div id="configuration-overview">
  ## Konfigurationsübersicht
</div>

Alle CLI-Backends liegen unter:

```
agents.defaults.cliBackends
```

Jedem Eintrag ist eine **Anbieter-ID** zugeordnet (z. B. `claude-cli`, `my-cli`).
Die Anbieter-ID bildet die linke Seite deiner Modellreferenz:

```
<provider>/<model>
```


<div id="example-configuration">
  ### Beispielkonfiguration
</div>

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude"
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          input: "arg",
          modelArg: "--model",
          modelAliases: {
            "claude-opus-4-5": "opus",
            "claude-sonnet-4-5": "sonnet"
          },
          sessionArg: "--session",
          sessionMode: "existing",
          sessionIdFields: ["session_id", "conversation_id"],
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
          serialize: true
        }
      }
    }
  }
}
```


<div id="how-it-works">
  ## Funktionsweise
</div>

1) **Wählt ein Backend aus** auf Grundlage des Anbieter-Präfixes (`claude-cli/...`).
2) **Erstellt einen System-Prompt** mithilfe desselben OpenClaw-Prompts und des Arbeitsbereichs-Kontexts.
3) **Führt die CLI aus** mit einer Sitzungs-ID (falls unterstützt), damit der Verlauf konsistent bleibt.
4) **Parst die Ausgabe** (JSON oder Klartext) und gibt den endgültigen Text zurück.
5) **Speichert Sitzungs-IDs** pro Backend, sodass Folgeanfragen dieselbe CLI-Sitzung wiederverwenden.

<div id="sessions">
  ## Sitzungen
</div>

- Wenn die CLI Sitzungen unterstützt, konfiguriere `sessionArg` (z. B. `--session-id`) oder
  `sessionArgs` (Platzhalter `{sessionId}`), wenn die ID in mehrere Flags
  eingesetzt werden muss.
- Wenn die CLI ein **Subcommand `resume`** mit abweichenden Flags verwendet, konfiguriere
  `resumeArgs` (ersetzt `args` beim Fortsetzen) und optional `resumeOutput`
  (für Fortsetzungen ohne JSON-Ausgabe).
- `sessionMode`:
  - `always`: immer eine Sitzungs-ID senden (neue UUID, falls keine gespeichert ist).
  - `existing`: nur eine Sitzungs-ID senden, wenn zuvor eine gespeichert wurde.
  - `none`: niemals eine Sitzungs-ID senden.

<div id="images-pass-through">
  ## Bilder (Durchleitung)
</div>

Wenn deine CLI Bildpfade akzeptiert, setze `imageArg`:

```json5
imageArg: "--image",
imageMode: "repeat"
```

OpenClaw schreibt Base64-kodierte Bilder in temporäre Dateien. Wenn `imageArg` gesetzt ist, werden diese Pfade als CLI-Argumente übergeben. Wenn `imageArg` fehlt, hängt OpenClaw die Dateipfade an den Prompt an (Pfad-Injection). Das reicht für CLIs, die lokale Dateien automatisch aus reinen Pfadangaben laden (Verhalten der Claude Code CLI).


<div id="inputs-outputs">
  ## Eingaben / Ausgaben
</div>

- `output: "json"` (Standard) versucht, JSON zu parsen und Text + Sitzungs-ID zu extrahieren.
- `output: "jsonl"` parst JSONL-Streams (Codex CLI `--json`) und extrahiert die
  letzte Nachricht des agents plus `thread_id`, falls vorhanden.
- `output: "text"` behandelt stdout als die endgültige Antwort.

Eingabemodi:

- `input: "arg"` (Standard) übergibt den Prompt als letztes CLI-Argument.
- `input: "stdin"` sendet den Prompt über stdin.
- Wenn der Prompt sehr lang ist und `maxPromptArgChars` gesetzt ist, wird stdin verwendet.

<div id="defaults-built-in">
  ## Standardwerte (integriert)
</div>

OpenClaw liefert eine Standardeinstellung für `claude-cli` mit:

- `command: "claude"`
- `args: ["-p", "--output-format", "json", "--dangerously-skip-permissions"]`
- `resumeArgs: ["-p", "--output-format", "json", "--dangerously-skip-permissions", "--resume", "{sessionId}"]`
- `modelArg: "--model"`
- `systemPromptArg: "--append-system-prompt"`
- `sessionArg: "--session-id"`
- `systemPromptWhen: "first"`
- `sessionMode: "always"`

OpenClaw liefert außerdem eine Standardeinstellung für `codex-cli` mit:

- `command: "codex"`
- `args: ["exec","--json","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
- `resumeArgs: ["exec","resume","{sessionId}","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
- `output: "jsonl"`
- `resumeOutput: "text"`
- `modelArg: "--model"`
- `imageArg: "--image"`
- `sessionMode: "existing"`

Überschreiben Sie diese Werte nur bei Bedarf (häufig: absoluter Pfad für `command`).

<div id="limitations">
  ## Einschränkungen
</div>

- **Keine OpenClaw-Tools** (das CLI-Backend erhält niemals Tool-Aufrufe). Einige CLIs
  können dennoch ihre eigenen agent-Tools ausführen.
- **Kein Streaming** (CLI-Ausgabe wird gesammelt und anschließend zurückgegeben).
- **Strukturierte Ausgaben** hängen vom JSON-Format der CLI ab.
- **Codex-CLI-Sitzungen** werden über Textausgabe (kein JSONL) fortgesetzt, die weniger
  strukturiert ist als der ursprüngliche `--json`-Durchlauf. OpenClaw-Sitzungen funktionieren weiterhin
  normal.

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

- **CLI nicht gefunden**: setze `command` auf einen vollständigen Pfad.
- **Falscher Modellname**: verwende `modelAliases`, um `provider/model` → CLI-Modell zuzuordnen.
- **Keine Sitzungsfortsetzung**: stelle sicher, dass `sessionArg` gesetzt ist und `sessionMode` nicht
  `none` ist (Codex CLI kann Sitzungen bei JSON-Ausgabe derzeit nicht fortsetzen).
- **Bilder werden ignoriert**: setze `imageArg` (und prüfe, ob die CLI Dateipfade unterstützt).