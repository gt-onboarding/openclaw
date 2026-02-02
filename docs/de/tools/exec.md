---
title: Exec
summary: "Verwendung des Exec-Tools, stdin-Modi und TTY-Unterstützung"
read_when:
  - Exec-Tool verwenden oder anpassen
  - stdin- oder TTY-Verhalten debuggen
---

<div id="exec-tool">
  # Exec-Tool
</div>

Führt Shell-Befehle im Arbeitsbereich aus. Unterstützt Vordergrund- und Hintergrundausführung über `process`.
Wenn `process` nicht erlaubt ist, läuft `exec` synchron und ignoriert `yieldMs`/`background`.
Hintergrund-Sitzungen sind pro agent isoliert; `process` sieht nur Sitzungen desselben agent.

<div id="parameters">
  ## Parameter
</div>

* `command` (erforderlich)
* `workdir` (Standard: cwd)
* `env` (Key/Value-Überschreibungen)
* `yieldMs` (Standard 10000): automatisches Verschieben in den Hintergrund nach einer Verzögerung
* `background` (bool): sofort im Hintergrund ausführen
* `timeout` (Sekunden, Standard 1800): bei Ablauf beenden
* `pty` (bool): in einem Pseudo-Terminal ausführen, wenn verfügbar (reine TTY-CLIs, Coding-Agenten, Terminal-UIs)
* `host` (`sandbox | gateway | node`): Ausführungsort
* `security` (`deny | allowlist | full`): Durchsetzungsmodus für `gateway`/`node`
* `ask` (`off | on-miss | always`): Genehmigungsabfragen für `gateway`/`node`
* `node` (string): Knoten-ID/-Name für `host=node`
* `elevated` (bool): erhöhten Modus anfordern (Gateway-Host); `security=full` wird nur erzwungen, wenn `elevated` letztlich zu `full` aufgelöst wird

Hinweise:

* `host` ist standardmäßig `sandbox`.
* `elevated` wird ignoriert, wenn Sandboxing deaktiviert ist (exec läuft dann bereits auf dem Host).
* Genehmigungen für `gateway`/`node` werden durch `~/.openclaw/exec-approvals.json` gesteuert.
* `node` erfordert einen gepaarten Knoten (Companion-App oder Headless-Knotenhost).
* Wenn mehrere Knoten verfügbar sind, setze `exec.node` oder `tools.exec.node`, um einen auszuwählen.
* Auf Nicht-Windows-Hosts verwendet exec `SHELL`, wenn gesetzt; wenn `SHELL` `fish` ist, wird `bash` (oder `sh`)
  aus `PATH` bevorzugt, um mit fish inkompatible Skripte zu vermeiden, und es wird auf `SHELL` zurückgegriffen, falls beide nicht vorhanden sind.
* Wichtig: Sandboxing ist **standardmäßig deaktiviert**. Wenn Sandboxing deaktiviert ist, läuft `host=sandbox` direkt auf
  dem Gateway-Host (kein Container) und **erfordert keine Genehmigungen**. Um Genehmigungen zu erzwingen, führe mit
  `host=gateway` aus und konfiguriere Exec-Genehmigungen (oder aktiviere Sandboxing).

<div id="config">
  ## Konfiguration
</div>

* `tools.exec.notifyOnExit` (Standard: true): Wenn auf true gesetzt, stellen im Hintergrund ausgeführte Exec-Sitzungen ein Systemereignis in die Warteschlange und fordern beim Beenden einen Herzschlag an.
* `tools.exec.approvalRunningNoticeMs` (Standard: 10000): Gibt eine einzelne „läuft“-Benachrichtigung aus, wenn eine approval-gesteuerte Exec-Ausführung länger als diesen Wert läuft (0 deaktiviert).
* `tools.exec.host` (Standard: `sandbox`)
* `tools.exec.security` (Standard: `deny` für die sandbox, `allowlist` für Gateway + knoten, wenn nicht gesetzt)
* `tools.exec.ask` (Standard: `on-miss`)
* `tools.exec.node` (Standard: nicht gesetzt)
* `tools.exec.pathPrepend`: Liste von Verzeichnissen, die Exec-Ausführungen vor `PATH` vorangestellt werden.
* `tools.exec.safeBins`: Nur-stdin-fähige, als sicher eingestufte Binaries, die ohne explizite Allowlist-Einträge ausgeführt werden können.

Beispiel:

```json5
{
  tools: {
    exec: {
      pathPrepend: ["~/bin", "/opt/oss/bin"]
    }
  }
}
```

<div id="path-handling">
  ### PATH-Behandlung
</div>

* `host=gateway`: übernimmt den `PATH` deiner Login-Shell in die Exec-Umgebung (sofern der Exec-Aufruf
  nicht bereits `env.PATH` setzt). Der Daemon selbst läuft weiterhin mit einem minimalen `PATH`:
  * macOS: `/opt/homebrew/bin`, `/usr/local/bin`, `/usr/bin`, `/bin`
  * Linux: `/usr/local/bin`, `/usr/bin`, `/bin`
* `host=sandbox`: führt `sh -lc` (Login-Shell) im Container aus, daher kann `/etc/profile` den `PATH` zurücksetzen.
  OpenClaw stellt `env.PATH` nach dem Laden des Profils über eine interne Umgebungsvariable voran (keine Shell-Interpolation);
  `tools.exec.pathPrepend` gilt hier ebenfalls.
* `host=node`: nur die von dir übergebenen Env-Overrides werden an den Knoten gesendet. `tools.exec.pathPrepend` gilt nur,
  wenn der Exec-Aufruf bereits `env.PATH` setzt. Headless-Knoten-Hosts akzeptieren `PATH` nur, wenn er den Host-`PATH`
  des Knotens vorn anstellt (kein Ersetzen). macOS-Knoten verwerfen `PATH`-Overrides vollständig.

Knotenbindung pro agent (verwende den agent-Listenindex in der Konfiguration):

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

Control UI: Die Registerkarte „Knoten“ enthält ein kleines Panel „Exec node binding“ mit denselben Einstellungen.

<div id="session-overrides-exec">
  ## Sitzungs-Overrides (`/exec`)
</div>

Verwende `/exec`, um **sitzungsspezifische** Standardwerte für `host`, `security`, `ask` und `knoten` festzulegen.
Sende `/exec` ohne Argumente, um die aktuellen Werte anzuzeigen.

Beispiel:

```
/exec host=gateway security=allowlist ask=on-miss node=mac-1
```

<div id="authorization-model">
  ## Autorisierungsmodell
</div>

`/exec` wird nur für **autorisierte Absender** berücksichtigt (Channel-Allowlists/Kopplung plus `commands.useAccessGroups`).
Es aktualisiert **nur den Sitzungszustand** und schreibt keine Konfiguration. Um `exec` hart zu deaktivieren, unterbinde es über die Tool-Policy (`tools.deny: ["exec"]` oder pro agent). Host-Freigaben gelten weiterhin, es sei denn, du setzt explizit `security=full` und `ask=off`.

<div id="exec-approvals-companion-app-node-host">
  ## Exec-Genehmigungen (Companion-App / Knoten-Host)
</div>

Sandboxed agents können eine Genehmigung pro Anfrage erfordern, bevor `exec` auf dem Gateway oder dem Knoten-Host ausgeführt wird.
Siehe [Exec approvals](/de/tools/exec-approvals) für Richtlinie, Allowlist und UI-Flow.

Wenn Genehmigungen erforderlich sind, gibt das `exec`-Tool sofort mit
`status: "approval-pending"` und einer Genehmigungs-ID zurück. Sobald die Ausführung genehmigt (oder abgelehnt / abgelaufen) ist,
sendet das Gateway Systemereignisse (`Exec finished` / `Exec denied`). Wenn der Befehl nach `tools.exec.approvalRunningNoticeMs`
immer noch läuft, wird eine einzelne `Exec running`-Benachrichtigung gesendet.

<div id="allowlist-safe-bins">
  ## Allowlist + sichere Binaries
</div>

Die Durchsetzung der Allowlist erfolgt nur für **aufgelöste Pfade zu Binärdateien** (keine Basename-Vergleiche). Wenn
`security=allowlist` gesetzt ist, werden Shell-Befehle nur dann automatisch erlaubt, wenn jedes Pipeline-Segment
in der Allowlist enthalten oder ein sicheres Binary ist. Verkettungen (`;`, `&&`, `||`) und Umleitungen werden im
Allowlist-Modus abgelehnt.

<div id="examples">
  ## Beispiele
</div>

Vordergrund:

```json
{"tool":"exec","command":"ls -la"}
```

Hintergrund + Umfrage:

```json
{"tool":"exec","command":"npm run build","yieldMs":1000}
{"tool":"process","action":"poll","sessionId":"<id>"}
```

Tasten senden (im tmux-Stil):

```json
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Enter"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["C-c"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Up","Up","Enter"]}
```

Absenden (nur CR Senden):

```json
{"tool":"process","action":"submit","sessionId":"<id>"}
```

Einfügen (standardmäßig in eckige Klammern gesetzt):

```json
{"tool":"process","action":"paste","sessionId":"<id>","text":"line1\nline2\n"}
```

<div id="apply_patch-experimental">
  ## apply_patch (experimentell)
</div>

`apply_patch` ist ein Unterwerkzeug von `exec` für strukturierte Änderungen an mehreren Dateien.
Aktiviere es explizit:

```json5
{
  tools: {
    exec: {
      applyPatch: { enabled: true, allowModels: ["gpt-5.2"] }
    }
  }
}
```

Hinweise:

* Nur verfügbar für OpenAI- und OpenAI-Codex-Modelle.
* Die Tool-Richtlinie gilt weiterhin; `allow: ["exec"]` erlaubt implizit auch `apply_patch`.
* Die Konfiguration befindet sich unter `tools.exec.applyPatch`.
