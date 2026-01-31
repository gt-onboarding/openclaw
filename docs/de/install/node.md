---
title: "Node.js + npm (PATH-Grundlagen)"
summary: "Node.js- und npm-Grundlagen: Versionen, PATH und globale Installationen"
read_when:
  - "Du hast OpenClaw installiert, aber `openclaw` ergibt den Fehler „command not found“"
  - "Du richtest Node.js/npm auf einem neuen Rechner ein"
  - "npm install -g ... schlägt wegen Berechtigungs- oder PATH-Problemen fehl"
---

<div id="nodejs-npm-path-sanity">
  # Node.js + npm (PATH-Check)
</div>

OpenClaw benötigt mindestens **Node 22+** zur Ausführung.

Wenn du `npm install -g openclaw@latest` ausführen kannst, später aber `openclaw: command not found` siehst, ist das fast immer ein **PATH**-Problem: Das Verzeichnis, in das npm globale Binaries installiert, ist nicht in deinem Shell-PATH.

<div id="quick-diagnosis">
  ## Schnelldiagnose
</div>

Führe Folgendes aus:

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

Wenn `$(npm prefix -g)/bin` (macOS/Linux) oder `$(npm prefix -g)` (Windows) **nicht** in der Ausgabe von `echo "$PATH"` enthalten ist, findet deine Shell die global installierten npm-Binaries (einschließlich `openclaw`) nicht.


<div id="fix-put-npms-global-bin-dir-on-path">
  ## Lösung: globales npm-Bin-Verzeichnis in den PATH aufnehmen
</div>

1. Finde dein globales npm-Prefix:

```bash
npm prefix -g
```

2. Füge das globale npm-Bin-Verzeichnis zu deinem Shell-Startskript hinzu:

* zsh: `~/.zshrc`
* bash: `~/.bashrc`

Beispiel (ersetze den Pfad durch die Ausgabe von `npm prefix -g`):

```bash
# macOS / Linux
export PATH="/path/from/npm/prefix/bin:$PATH"
```

Öffne anschließend ein **neues Terminal** (oder führe `rehash` in zsh bzw. `hash -r` in bash aus).

Unter Windows musst du die Ausgabe von `npm prefix -g` zu deinem PATH hinzufügen.


<div id="fix-avoid-sudo-npm-install-g-permission-errors-linux">
  ## Fix: vermeide `sudo npm install -g` / Berechtigungsfehler (Linux)
</div>

Wenn `npm install -g ...` mit `EACCES` fehlschlägt, ändere das globale Prefix von npm in ein vom Benutzer schreibbares Verzeichnis:

```bash
mkdir -p "$HOME/.npm-global"
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

Füge die Zeile `export PATH=...` dauerhaft in deiner Shell-Startdatei hinzu.


<div id="recommended-node-install-options">
  ## Empfohlene Optionen zur Node-Installation
</div>

Du wirst die wenigsten Überraschungen haben, wenn Node/npm so installiert sind, dass:

- Node aktuell gehalten wird (22+)
- das globale npm-Bin-Verzeichnis stabil ist und in neuen Shells im PATH liegt

Gängige Optionen:

- macOS: Homebrew (`brew install node`) oder ein Versions-Manager
- Linux: dein bevorzugter Versions-Manager oder eine von der Distribution unterstützte Installation, die Node 22+ bereitstellt
- Windows: offizieller Node-Installer, `winget` oder ein Windows-Node-Versions-Manager

Wenn du einen Versions-Manager (nvm/fnm/asdf/etc.) verwendest, stelle sicher, dass er in der Shell initialisiert wird, die du täglich nutzt (zsh vs. bash), damit der von ihm gesetzte PATH verfügbar ist, wenn du Installer ausführst.