---
title: Debugging
summary: "Debugging-Tools: Watch-Modus, rohe Modellstreams und Tracing von Reasoning-Leakage"
read_when:
  - Du musst rohe Modellausgaben auf Reasoning-Leakage prüfen
  - Du möchtest das Gateway im Watch-Modus ausführen, während du iterierst
  - Du brauchst einen wiederholbaren Debugging-Workflow
---

<div id="debugging">
  # Debugging
</div>

Auf dieser Seite werden Hilfen zum Debuggen von Streaming-Ausgaben beschrieben, insbesondere für Fälle, in denen ein Anbieter Überlegungen in normalen Text einmischt.

<div id="runtime-debug-overrides">
  ## Laufzeit-Debug-Overrides
</div>

Verwende `/debug` im Chat, um **nur zur Laufzeit** gültige Konfigurations-Overrides zu setzen (im Speicher, nicht auf der Festplatte).
`/debug` ist standardmäßig deaktiviert; aktiviere es mit `commands.debug: true`.
Das ist praktisch, wenn du obskure Einstellungen umschalten musst, ohne `openclaw.json` zu bearbeiten.

Beispiele:

```
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug unset messages.responsePrefix
/debug reset
```

`/debug reset` setzt alle Überschreibungen zurück und stellt die Konfiguration von der Festplatte wieder her.


<div id="gateway-watch-mode">
  ## Gateway-Watch-Modus
</div>

Für schnelle Iterationen führe das Gateway mit dem File-Watcher aus:

```bash
pnpm gateway:watch --force
```

Dies entspricht:

```bash
tsx watch src/entry.ts gateway --force
```

Füge beliebige Gateway-CLI-Flags hinter `gateway:watch` hinzu, sie werden bei jedem Neustart weitergereicht.


<div id="dev-profile-dev-gateway-dev">
  ## Dev-Profil + Dev-Gateway (--dev)
</div>

Verwende das Dev-Profil, um den State zu isolieren und ein sicheres, wegwerfbares
Setup fürs Debugging zu starten. Es gibt **zwei** `--dev`‑Flags:

* **Globales `--dev` (Profil):** isoliert den State unter `~/.openclaw-dev` und
  setzt den Standardport des Gateways auf `19001` (zugehörige Ports verschieben sich entsprechend mit).
* **`gateway --dev`: weist das Gateway an, bei fehlender Konfiguration automatisch eine Standard-Config +
  einen Arbeitsbereich zu erstellen** (und BOOTSTRAP.md zu überspringen).

Empfohlener Ablauf (Dev-Profil + Dev-Bootstrap):

```bash
pnpm gateway:dev
OPENCLAW_PROFILE=dev openclaw tui
```

Wenn du noch keine globale Installation hast, führe die CLI über `pnpm openclaw ...` aus.

Was das bewirkt:

1. **Profilisolation** (global `--dev`)
   * `OPENCLAW_PROFILE=dev`
   * `OPENCLAW_STATE_DIR=~/.openclaw-dev`
   * `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
   * `OPENCLAW_GATEWAY_PORT=19001` (Browser/Canvas werden entsprechend verschoben)

2. **Dev-Bootstrap** (`gateway --dev`)
   * Schreibt eine minimale Konfiguration, falls sie fehlt (`gateway.mode=local`, bindet an Loopback).
   * Setzt `agent.workspace` auf den Dev-Arbeitsbereich.
   * Setzt `agent.skipBootstrap=true` (kein BOOTSTRAP.md).
   * Initialisiert die Dateien im Arbeitsbereich, falls sie fehlen:
     `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`.
   * Standardidentität: **C3‑PO** (Protokolldroide).
   * Überspringt Kanal-Anbieter im Dev-Modus (`OPENCLAW_SKIP_CHANNELS=1`).

Reset-Ablauf (frischer Start):

```bash
pnpm gateway:dev:reset
```

Hinweis: `--dev` ist ein **globales** Profil-Flag und wird von einigen Runnern ignoriert.
Wenn du es explizit angeben musst, verwende die Variante mit Umgebungsvariable:

```bash
OPENCLAW_PROFILE=dev openclaw gateway --dev --reset
```

`--reset` löscht Konfiguration, Anmeldedaten, Sitzungen und den Dev-Arbeitsbereich (wobei `trash` und nicht `rm` verwendet wird) und richtet anschließend das Standard-Dev-Setup neu ein.

Tipp: Falls bereits ein Gateway außerhalb der Entwicklungsumgebung läuft (launchd/systemd), stoppen Sie es zuerst:

```bash
openclaw gateway stop
```


<div id="raw-stream-logging-openclaw">
  ## Roh-Stream-Logging (OpenClaw)
</div>

OpenClaw kann den **rohen Assistant-Stream** protokollieren, bevor irgendeine Filterung bzw. Formatierung stattfindet.
Dies ist der beste Weg, um zu sehen, ob Reasoning als Plain-Text-Deltas
(oder als separate Thinking-Blöcke) ankommt.

Aktiviere es über die CLI:

```bash
pnpm gateway:watch --force --raw-stream
```

Optionaler Pfad-Override:

```bash
pnpm gateway:watch --force --raw-stream --raw-stream-path ~/.openclaw/logs/raw-stream.jsonl
```

Entsprechende Umgebungsvariablen:

```bash
OPENCLAW_RAW_STREAM=1
OPENCLAW_RAW_STREAM_PATH=~/.openclaw/logs/raw-stream.jsonl
```

Standarddatei:

`~/.openclaw/logs/raw-stream.jsonl`


<div id="raw-chunk-logging-pi-mono">
  ## Logging von Roh-Chunks (pi-mono)
</div>

Um **rohe, OpenAI-kompatible Chunks** zu erfassen, bevor sie zu Blöcken geparst werden,
stellt pi-mono einen separaten Logger bereit:

```bash
PI_RAW_STREAM=1
```

Optionaler Pfad:

```bash
PI_RAW_STREAM_PATH=~/.pi-mono/logs/raw-openai-completions.jsonl
```

Standarddatei:

`~/.pi-mono/logs/raw-openai-completions.jsonl`

> Hinweis: Dies wird nur von Prozessen ausgegeben, die den
> `openai-completions` Anbieter von pi-mono verwenden.


<div id="safety-notes">
  ## Sicherheitshinweise
</div>

- Roh-Stream-Protokolle können vollständige Prompts, Tool-Ausgaben und Nutzerdaten enthalten.
- Speichere Protokolle nur lokal und lösche sie nach dem Debugging.
- Wenn du Protokolle weitergibst, entferne vorher Secrets und personenbezogene Daten (PII).