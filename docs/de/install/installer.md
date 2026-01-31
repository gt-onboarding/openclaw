---
title: Installer
summary: "Funktionsweise der Installer-Skripte (install.sh + install-cli.sh), Flags und Automatisierung"
read_when:
  - Du möchtest `openclaw.bot/install.sh` verstehen
  - Du möchtest Installationen automatisieren (CI / headless)
  - Du möchtest aus einem GitHub-Checkout installieren
---

<div id="installer-internals">
  # Interna des Installers
</div>

OpenClaw wird mit zwei Installationsskripten ausgeliefert (bereitgestellt von `openclaw.ai`):

* `https://openclaw.bot/install.sh` — „empfohlener“ Installer (standardmäßig globale npm-Installation; kann auch aus einem GitHub-Checkout installieren)
* `https://openclaw.bot/install-cli.sh` — CLI-Installer, der ohne Root-Rechte auskommt (installiert in ein Präfixverzeichnis mit eigenem Node.js)
* `https://openclaw.ai/install.ps1` — Windows PowerShell-Installer (standardmäßig npm; optionale Git-Installation)

Um die aktuellen Flags bzw. das aktuelle Verhalten zu sehen, führe aus:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --help
```

Windows-(PowerShell)-Hilfe:

```powershell
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -?
```

Wenn das Installationsprogramm zwar durchläuft, `openclaw` in einem neuen Terminal aber nicht gefunden wird, liegt das normalerweise an einem Node-/npm-PATH-Problem. Siehe: [Installation](/de/install#nodejs--npm-path-sanity).

<div id="installsh-recommended">
  ## install.sh (empfohlen)
</div>

Was es auf hoher Ebene macht:

* Erkennt das Betriebssystem (macOS / Linux / WSL).
* Stellt sicher, dass Node.js **22+** installiert ist (macOS über Homebrew; Linux über NodeSource).
* Wählt die Installationsmethode:
  * `npm` (Standard): `npm install -g openclaw@latest`
  * `git`: Klont/baut einen Quellcode-Checkout und installiert ein Wrapper-Skript
* Unter Linux: Vermeidet globale npm-Berechtigungsfehler, indem bei Bedarf das npm-Prefix auf `~/.npm-global` umgestellt wird.
* Beim Upgrade einer bestehenden Installation: führt `openclaw doctor --non-interactive` aus (nach Möglichkeit).
* Für git-Installationen: führt `openclaw doctor --non-interactive` nach Installation/Update aus (nach Möglichkeit).
* Umgeht typische `sharp`-Native-Installationsprobleme, indem standardmäßig `SHARP_IGNORE_GLOBAL_LIBVIPS=1` gesetzt wird (verhindert das Bauen gegen die systemweite libvips).

Wenn du *möchtest*, dass `sharp` gegen eine global installierte libvips linkt (oder du debuggen willst), setze:

```bash
SHARP_IGNORE_GLOBAL_LIBVIPS=0 curl -fsSL https://openclaw.bot/install.sh | bash
```

<div id="discoverability-git-install-prompt">
  ### Auffindbarkeit / „git install“-Aufforderung
</div>

Wenn du das Installationsskript **bereits in einem OpenClaw-Quellcode-Checkout** ausführst (erkannt über `package.json` + `pnpm-workspace.yaml`), erscheint folgende Abfrage:

* dieses Checkout aktualisieren und verwenden (`git`)
* oder zur globalen npm-Installation migrieren (`npm`)

In nicht-interaktiven Kontexten (kein TTY / `--no-prompt`) musst du `--install-method git|npm` übergeben (oder `OPENCLAW_INSTALL_METHOD` setzen), andernfalls beendet sich das Skript mit Exit-Code `2`.

<div id="why-git-is-needed">
  ### Warum Git benötigt wird
</div>

Git wird für den `--install-method git` Pfad benötigt (clone / pull).

Für `npm`-Installationen ist Git *meistens* nicht erforderlich, aber in einigen Umgebungen wird es dennoch gebraucht (z. B. wenn ein Paket oder eine Abhängigkeit über eine Git-URL bezogen wird). Der Installer stellt derzeit sicher, dass Git verfügbar ist, um `spawn git ENOENT`-Überraschungen auf frischen Distributionen zu vermeiden.

<div id="why-npm-hits-eacces-on-fresh-linux">
  ### Warum npm auf einem frisch installierten Linux-System `EACCES` auslöst
</div>

In einigen Linux-Setups (insbesondere nach der Installation von Node über den System-Paketmanager oder NodeSource) verweist das globale Präfix von npm auf einen von root verwalteten Pfad. Dann schlägt `npm install -g ...` mit `EACCES`- / `mkdir`-Berechtigungsfehlern fehl.

`install.sh` umgeht dieses Problem, indem es das Präfix umstellt auf:

* `~/.npm-global` (und es zu `PATH` in `~/.bashrc` / `~/.zshrc` hinzufügt, sofern vorhanden)

<div id="install-clish-non-root-cli-installer">
  ## install-cli.sh (CLI-Installer ohne Root-Rechte)
</div>

Dieses Skript installiert `openclaw` unter einem Präfix (Standard: `~/.openclaw`) und richtet außerdem eine dedizierte Node-Laufzeitumgebung unter diesem Präfix ein, sodass es auf Systemen funktioniert, auf denen du die Systeminstallation von Node/npm nicht verändern möchtest.

Hilfe:

```bash
curl -fsSL https://openclaw.bot/install-cli.sh | bash -s -- --help
```

<div id="installps1-windows-powershell">
  ## install.ps1 (Windows PowerShell)
</div>

Was es macht (auf hoher Ebene):

* Stellt sicher, dass Node.js **22+** installiert ist (winget/Chocolatey/Scoop oder manuell).
* Ermöglicht die Wahl der Installationsmethode:
  * `npm` (Standard): `npm install -g openclaw@latest`
  * `git`: Klont/baut ein Quellcode-Checkout und installiert ein Wrapper-Skript
* Führt `openclaw doctor --non-interactive` bei Upgrades und Git-Installationen aus (soweit möglich).

Beispiele:

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex -InstallMethod git
```

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex -InstallMethod git -GitDir "C:\\openclaw"
```

Umgebungsvariablen:

* `OPENCLAW_INSTALL_METHOD=git|npm`
* `OPENCLAW_GIT_DIR=...`

Git-Voraussetzung:

Wenn du `-InstallMethod git` wählst und Git fehlt, gibt das Installationsskript den
Git-for-Windows-Link (`https://git-scm.com/download/win`) aus und beendet sich.

Häufige Windows-Probleme:

* **npm error spawn git / ENOENT**: Installiere Git for Windows und öffne PowerShell erneut, dann führe den Installer noch einmal aus.
* **&quot;openclaw&quot; wird nicht erkannt**: Dein globaler npm-Bin-Ordner ist nicht im PATH. Die meisten Systeme verwenden
  `%AppData%\\npm`. Du kannst auch `npm config get prefix` ausführen und `\\bin` zum PATH hinzufügen, dann PowerShell erneut öffnen.
