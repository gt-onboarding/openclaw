---
title: Nix
summary: "Installiere OpenClaw deklarativ mit Nix"
read_when:
  - Du möchtest reproduzierbare, rollback-fähige Installationen
  - Du verwendest bereits Nix/NixOS/Home Manager
  - Du möchtest, dass alles vollständig gepinnt und deklarativ verwaltet wird
---

<div id="nix-installation">
  # Nix-Installation
</div>

Die empfohlene Methode, OpenClaw mit Nix zu betreiben, ist über **[nix-openclaw](https://github.com/openclaw/nix-openclaw)** – ein vollständig ausgestattetes Home-Manager-Modul.

<div id="quick-start">
  ## Schnellstart
</div>

Füge das in deinen KI-Agenten (Claude, Cursor, etc.) ein:

```text
Ich möchte nix-openclaw auf meinem Mac einrichten.
Repository: github:openclaw/nix-openclaw

Was ich von dir brauche:
1. Prüfe, ob Determinate Nix installiert ist (falls nicht, installiere es)
2. Erstelle ein lokales Flake unter ~/code/openclaw-local mit templates/agent-first/flake.nix
3. Hilf mir, einen Telegram-Bot zu erstellen (@BotFather) und meine Chat-ID zu erhalten (@userinfobot)
4. Richte Secrets ein (Bot-Token, Anthropic-Schlüssel) – einfache Dateien unter ~/.secrets/ sind in Ordnung
5. Fülle die Template-Platzhalter aus und führe home-manager switch aus
6. Verifiziere: launchd läuft, Bot antwortet auf Nachrichten

Siehe die nix-openclaw README für Moduloptionen.
```

> **📦 Vollständige Anleitung: [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)**
>
> Das nix-openclaw-Repository ist die verbindliche Referenz für die Nix-Installation. Diese Seite bietet nur einen kurzen Überblick.

<div id="what-you-get">
  ## Was du bekommst
</div>

* Gateway + macOS-App + Tools (Whisper, Spotify, Kameras) – alles gepinnt
* launchd-Dienst, der Neustarts übersteht
* Plugin-System mit deklarativer Konfiguration
* Sofortiges Rollback: `home-manager switch --rollback`

***

<div id="nix-mode-runtime-behavior">
  ## Laufzeitverhalten im Nix-Modus
</div>

Wenn `OPENCLAW_NIX_MODE=1` gesetzt ist (automatisch mit nix-openclaw):

OpenClaw unterstützt einen **Nix-Modus**, der für eine deterministische Konfiguration sorgt und automatische Installationsabläufe deaktiviert.
Aktiviere ihn, indem du exportierst:

```bash
OPENCLAW_NIX_MODE=1
```

Unter macOS übernimmt die GUI-App Shell-Umgebungsvariablen nicht automatisch. Du kannst
den Nix-Modus auch per defaults aktivieren:

```bash
defaults write bot.molt.mac openclaw.nixMode -bool true
```

<div id="config-state-paths">
  ### Konfigurations- und Zustandspfade
</div>

OpenClaw liest die JSON5-Konfiguration aus `OPENCLAW_CONFIG_PATH` und speichert veränderliche Daten in `OPENCLAW_STATE_DIR`.

* `OPENCLAW_STATE_DIR` (Standard: `~/.openclaw`)
* `OPENCLAW_CONFIG_PATH` (Standard: `$OPENCLAW_STATE_DIR/openclaw.json`)

Wenn du OpenClaw unter Nix betreibst, setze diese Variablen explizit auf von Nix verwaltete Pfade, damit Laufzeitzustand und Konfiguration außerhalb des unveränderlichen Stores bleiben.

<div id="runtime-behavior-in-nix-mode">
  ### Laufzeitverhalten im Nix-Modus
</div>

* Abläufe für automatische Installation und Selbstaktualisierung sind deaktiviert
* Fehlende Abhängigkeiten lösen Nix-spezifische Hinweise zur Fehlerbehebung aus
* Die UI zeigt ein schreibgeschütztes Nix-Modus-Banner an, wenn vorhanden

<div id="packaging-note-macos">
  ## Hinweis zur Paketierung (macOS)
</div>

Der macOS-Paketierungsprozess erwartet eine stabile Info.plist-Vorlage unter:

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) kopiert diese Vorlage in das App-Bundle und passt dynamische Felder an
(Bundle-ID, Version/Build, Git-SHA, Sparkle-Schlüssel). Dadurch bleibt die plist für SwiftPM-Packaging
und Nix-Builds deterministisch (diese sind nicht auf eine vollständige Xcode-Toolchain angewiesen).

<div id="related">
  ## Verwandt
</div>

* [nix-openclaw](https://github.com/openclaw/nix-openclaw) — vollständige Anleitung zur Einrichtung
* [Wizard](/de/start/wizard) — CLI-Einrichtung ohne Nix
* [Docker](/de/install/docker) — containerbasierte Einrichtung