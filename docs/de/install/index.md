---
title: Installieren
summary: "OpenClaw installieren (empfohlener Installer, globale Installation oder aus dem Quellcode)"
read_when:
  - Du installierst OpenClaw
  - Du möchtest OpenClaw von GitHub installieren
---

<div id="install">
  # Installation
</div>

Verwende den Installer, es sei denn, du hast einen guten Grund dagegen. Er richtet die CLI ein und führt das Onboarding durch.

<div id="quick-install-recommended">
  ## Schnellinstallation (empfohlen)
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Windows (PowerShell):

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

Nächster Schritt (falls Sie das Onboarding übersprungen haben):

```bash
openclaw onboard --install-daemon
```

<div id="system-requirements">
  ## Systemanforderungen
</div>

* **Node &gt;=22**
* macOS, Linux oder Windows über WSL2
* `pnpm` nur, wenn du aus dem Quellcode baust

<div id="choose-your-install-path">
  ## Wählen Sie Ihren Installationsweg
</div>

<div id="1-installer-script-recommended">
  ### 1) Installationsskript (empfohlen)
</div>

Installiert `openclaw` global über npm und führt das Onboarding durch.

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Installer-Flags:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --help
```

Details: [Interna des Installers](/de/install/installer).

Nicht interaktiv (Onboarding überspringen):

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --no-onboard
```

<div id="2-global-install-manual">
  ### 2) Globale Installation (manuell)
</div>

Wenn du Node.js bereits installiert hast:

```bash
npm install -g openclaw@latest
```

Wenn du `libvips` systemweit installiert hast (unter macOS häufig über Homebrew) und die Installation von `sharp` fehlschlägt, erzwinge die Verwendung vorkompilierter Binärdateien:

```bash
SHARP_IGNORE_GLOBAL_LIBVIPS=1 npm install -g openclaw@latest
```

Wenn du `sharp: Please add node-gyp to your dependencies` siehst, installiere entweder die Build-Tools (macOS: Xcode CLT + `npm install -g node-gyp`) oder verwende den oben beschriebenen Workaround `SHARP_IGNORE_GLOBAL_LIBVIPS=1`, um den nativen Build zu überspringen.

Oder:

```bash
pnpm add -g openclaw@latest
```

Anschließend:

```bash
openclaw onboard --install-daemon
```

<div id="3-from-source-contributorsdev">
  ### 3) Aus dem Quellcode (contributors/dev)
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # installiert UI-Abhängigkeiten beim ersten Ausführen automatisch
pnpm build
openclaw onboard --install-daemon
```

Tipp: Falls du openclaw noch nicht global installiert hast, führe Repo-Befehle mit `pnpm openclaw ...` aus.

<div id="4-other-install-options">
  ### 4) Weitere Installationsoptionen
</div>

* Docker: [Docker](/de/install/docker)
* Nix: [Nix](/de/install/nix)
* Ansible: [Ansible](/de/install/ansible)
* Bun (nur CLI): [Bun](/de/install/bun)

<div id="after-install">
  ## Nach der Installation
</div>

* Führe das Onboarding aus: `openclaw onboard --install-daemon`
* Führe einen Schnelltest durch: `openclaw doctor`
* Prüfe den Gateway-Status: `openclaw status` + `openclaw health`
* Öffne das Dashboard: `openclaw dashboard`

<div id="install-method-npm-vs-git-installer">
  ## Installationsmethode: npm vs git (Installer)
</div>

Der Installer unterstützt zwei Methoden:

* `npm` (Standard): `npm install -g openclaw@latest`
* `git`: Von GitHub klonen/builden und aus dem ausgecheckten Quellcode ausführen

<div id="cli-flags">
  ### CLI-Flags
</div>

```bash
# Explizites npm
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method npm

# Installation von GitHub (Quellcode-Checkout)
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

Häufig verwendete Flags:

* `--install-method npm|git`
* `--git-dir <path>` (Standardpfad: `~/openclaw`)
* `--no-git-update` (`git pull` bei Verwendung eines vorhandenen Checkouts überspringen)
* `--no-prompt` (Prompts deaktivieren; erforderlich in CI/Automatisierung)
* `--dry-run` (ausgeben, was durchgeführt würde; keine Änderungen vornehmen)
* `--no-onboard` (Onboarding überspringen)

<div id="environment-variables">
  ### Umgebungsvariablen
</div>

Entsprechende Umgebungsvariablen (nützlich für die Automatisierung):

* `OPENCLAW_INSTALL_METHOD=git|npm`
* `OPENCLAW_GIT_DIR=...`
* `OPENCLAW_GIT_UPDATE=0|1`
* `OPENCLAW_NO_PROMPT=1`
* `OPENCLAW_DRY_RUN=1`
* `OPENCLAW_NO_ONBOARD=1`
* `SHARP_IGNORE_GLOBAL_LIBVIPS=0|1` (Standardwert: `1`; verhindert, dass `sharp` gegen die systemweite libvips-Bibliothek gebaut wird)

<div id="troubleshooting-openclaw-not-found-path">
  ## Fehlerbehebung: `openclaw` nicht gefunden (PATH)
</div>

Schnelle Diagnose:

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

Wenn `$(npm prefix -g)/bin` (macOS/Linux) oder `$(npm prefix -g)` (Windows) **nicht** in der Ausgabe von `echo "$PATH"` enthalten ist, kann deine Shell globale npm-Binaries (einschließlich `openclaw`) nicht finden.

Lösung: Füge es deinem Shell-Startskript hinzu (zsh: `~/.zshrc`, bash: `~/.bashrc`):

```bash
# macOS / Linux
export PATH="$(npm prefix -g)/bin:$PATH"
```

Unter Windows fügst du die Ausgabe von `npm prefix -g` zu deinem PATH hinzu.

Öffne anschließend ein neues Terminalfenster (oder führe `rehash` in zsh bzw. `hash -r` in bash aus).

<div id="update-uninstall">
  ## Aktualisierung / Deinstallation
</div>

* Aktualisierungen: [Aktualisieren](/de/install/updating)
* Migration auf einen neuen Rechner: [Migrieren](/de/install/migrating)
* Deinstallation: [Deinstallieren](/de/install/uninstall)