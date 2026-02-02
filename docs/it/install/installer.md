---
title: Installer
summary: "Come funzionano gli script di installazione (install.sh + install-cli.sh), flag e automazione"
read_when:
  - Vuoi comprendere `openclaw.bot/install.sh`
  - Vuoi automatizzare le installazioni (CI / senza interfaccia grafica, headless)
  - Vuoi installare da un checkout GitHub
---

<div id="installer-internals">
  # Dettagli interni dell&#39;installer
</div>

OpenClaw fornisce due script di installazione (distribuiti da `openclaw.ai`):

* `https://openclaw.bot/install.sh` — installer “consigliato” (installazione globale tramite npm per impostazione predefinita; può anche installare da un clone GitHub)
* `https://openclaw.bot/install-cli.sh` — installer CLI adatto a contesti senza privilegi di root (installa in un prefisso con il proprio runtime Node.js)
* `https://openclaw.ai/install.ps1` — installer Windows PowerShell (npm per impostazione predefinita; installazione tramite git opzionale)

Per vedere le opzioni/comportamento attuali, esegui:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --help
```

Guida per Windows (PowerShell):

```powershell
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -?
```

Se il programma di installazione termina correttamente ma `openclaw` non viene trovato in un nuovo terminale, di solito è un problema di PATH di Node o npm. Consulta: [Install](/it/install#nodejs--npm-path-sanity).

<div id="installsh-recommended">
  ## install.sh (consigliato)
</div>

Cosa fa (a livello generale):

* Rileva il sistema operativo (macOS / Linux / WSL).
* Verifica la presenza di Node.js **22+** (macOS tramite Homebrew; Linux tramite NodeSource) e la installa/aggiorna se necessario.
* Sceglie il metodo di installazione:
  * `npm` (predefinito): `npm install -g openclaw@latest`
  * `git`: clona/compila una checkout del sorgente e installa uno script wrapper
* Su Linux: evita errori di permessi con l’installazione globale di npm cambiando il prefix di npm in `~/.npm-global` quando necessario.
* Se stai aggiornando un’installazione esistente: esegue `openclaw doctor --non-interactive` (best effort).
* Per le installazioni via git: esegue `openclaw doctor --non-interactive` dopo install/update (best effort).
* Mitiga i problemi comuni di installazione nativa di `sharp` impostando di default `SHARP_IGNORE_GLOBAL_LIBVIPS=1` (evita la compilazione contro la libvips di sistema).

Se *vuoi* che `sharp` si colleghi a una libvips installata globalmente (o stai effettuando il debug), imposta:

```bash
SHARP_IGNORE_GLOBAL_LIBVIPS=0 curl -fsSL https://openclaw.bot/install.sh | bash
```

<div id="discoverability-git-install-prompt">
  ### Individuazione / prompt “git install”
</div>

Se esegui l’installer **mentre ti trovi già all’interno di una checkout del sorgente di OpenClaw** (rilevata tramite `package.json` + `pnpm-workspace.yaml`), viene mostrato questo prompt:

* aggiornare e usare questa checkout (`git`)
* oppure migrare all’installazione globale npm (`npm`)

In contesti non interattivi (assenza di TTY / `--no-prompt`), devi passare `--install-method git|npm` (oppure impostare `OPENCLAW_INSTALL_METHOD`), altrimenti lo script termina con codice `2`.

<div id="why-git-is-needed">
  ### Perché Git è necessario
</div>

Git è richiesto per il percorso `--install-method git` (clone / pull).

Per le installazioni tramite `npm`, Git di solito *non* è richiesto, ma in alcuni ambienti finisce comunque per essere necessario (ad esempio quando un pacchetto o una dipendenza viene scaricato tramite un URL git). Al momento, l’installer verifica che Git sia presente per evitare sorprese tipo `spawn git ENOENT` su distribuzioni appena installate.

<div id="why-npm-hits-eacces-on-fresh-linux">
  ### Perché npm genera `EACCES` su un Linux appena installato
</div>

Su alcune configurazioni Linux (soprattutto dopo aver installato Node tramite il package manager di sistema o NodeSource), il prefisso globale di npm è impostato su un percorso di proprietà dell’utente root. Di conseguenza `npm install -g ...` va in errore con errori di permessi `EACCES` / `mkdir`.

`install.sh` mitiga questo problema cambiando il prefisso in:

* `~/.npm-global` (e aggiungendolo al `PATH` in `~/.bashrc` / `~/.zshrc` quando presenti)

<div id="install-clish-non-root-cli-installer">
  ## install-cli.sh (programma di installazione CLI non-root)
</div>

Questo script installa `openclaw` in un prefisso (predefinito: `~/.openclaw`) e installa anche un runtime Node dedicato sotto quello stesso prefisso, in modo che possa funzionare su macchine in cui non vuoi modificare il Node/npm di sistema.

Guida:

```bash
curl -fsSL https://openclaw.bot/install-cli.sh | bash -s -- --help
```

<div id="installps1-windows-powershell">
  ## install.ps1 (Windows PowerShell)
</div>

Cosa fa (a livello generale):

* Verifica che sia installato Node.js **22+** (via winget/Chocolatey/Scoop o manualmente).
* Scegli il metodo di installazione:
  * `npm` (predefinito): `npm install -g openclaw@latest`
  * `git`: clona/compila un checkout del codice sorgente e installa uno script wrapper
* Esegue `openclaw doctor --non-interactive` durante gli aggiornamenti e le installazioni via git (best effort).

Esempi:

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex -InstallMethod git
```

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex -InstallMethod git -GitDir "C:\\openclaw"
```

Variabili d&#39;ambiente:

* `OPENCLAW_INSTALL_METHOD=git|npm`
* `OPENCLAW_GIT_DIR=...`

Requisito Git:

Se scegli `-InstallMethod git` e Git non è installato, il programma di installazione stamperà il link
Git for Windows (`https://git-scm.com/download/win`) e terminerà.

Problemi comuni su Windows:

* **errore npm spawn git / ENOENT**: installa Git for Windows e riapri PowerShell, quindi esegui di nuovo il programma di installazione.
* **&quot;openclaw&quot; non è riconosciuto**: la cartella bin globale di npm non è presente nel PATH. La maggior parte dei sistemi usa
  `%AppData%\\npm`. Puoi anche eseguire `npm config get prefix` e aggiungere `\\bin` al PATH, quindi riaprire PowerShell.
