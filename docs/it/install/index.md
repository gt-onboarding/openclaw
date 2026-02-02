---
title: Installazione
summary: "Installa OpenClaw (installer consigliato, installazione globale o da sorgente)"
read_when:
  - Stai installando OpenClaw
  - Vuoi installare OpenClaw da GitHub
---

<div id="install">
  # Installazione
</div>

Usa l&#39;installer a meno che tu non abbia un motivo per non farlo. Configura la CLI ed esegue la procedura di onboarding.

<div id="quick-install-recommended">
  ## Installazione rapida (consigliata)
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Windows (PowerShell):

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

Passaggio successivo (se hai saltato la configurazione iniziale):

```bash
openclaw onboard --install-daemon
```

<div id="system-requirements">
  ## Requisiti di sistema
</div>

* **Node &gt;=22**
* macOS, Linux o Windows tramite WSL2
* `pnpm` solo se compili dai sorgenti

<div id="choose-your-install-path">
  ## Scegli il metodo di installazione
</div>

<div id="1-installer-script-recommended">
  ### 1) Script di installazione (consigliato)
</div>

Installa `openclaw` a livello globale tramite npm ed esegue la procedura di onboarding iniziale.

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Flag dell&#39;installer:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --help
```

Dettagli: [Internals dell&#39;installer](/it/install/installer).

Modalità non interattiva (salta l&#39;onboarding):

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --no-onboard
```

<div id="2-global-install-manual">
  ### 2) Installazione globale (manuale)
</div>

Se hai già installato Node:

```bash
npm install -g openclaw@latest
```

Se hai libvips installato a livello di sistema (comune su macOS tramite Homebrew) e l&#39;installazione di `sharp` non riesce, forza l&#39;uso dei binari precompilati:

```bash
SHARP_IGNORE_GLOBAL_LIBVIPS=1 npm install -g openclaw@latest
```

Se visualizzi `sharp: Please add node-gyp to your dependencies`, installa gli strumenti di compilazione (macOS: Xcode CLT + `npm install -g node-gyp`) oppure usa la soluzione alternativa `SHARP_IGNORE_GLOBAL_LIBVIPS=1` indicata sopra per evitare la compilazione nativa.

Oppure:

```bash
pnpm add -g openclaw@latest
```

Poi:

```bash
openclaw onboard --install-daemon
```

<div id="3-from-source-contributorsdev">
  ### 3) Da sorgente (contributor/dev)
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # installa automaticamente le dipendenze della UI alla prima esecuzione
pnpm build
openclaw onboard --install-daemon
```

Suggerimento: se non hai ancora installato openclaw globalmente, esegui i comandi del repo tramite `pnpm openclaw ...`.

<div id="4-other-install-options">
  ### 4) Altre opzioni di installazione
</div>

* Docker: [Docker](/it/install/docker)
* Nix: [Nix](/it/install/nix)
* Ansible: [Ansible](/it/install/ansible)
* Bun (solo CLI): [Bun](/it/install/bun)

<div id="after-install">
  ## Dopo l&#39;installazione
</div>

* Esegui la procedura di onboarding: `openclaw onboard --install-daemon`
* Esegui un controllo rapido: `openclaw doctor`
* Verifica lo stato del Gateway: `openclaw status` + `openclaw health`
* Apri la dashboard: `openclaw dashboard`

<div id="install-method-npm-vs-git-installer">
  ## Metodo di installazione: npm vs git (installer)
</div>

Il programma di installazione supporta due metodi:

* `npm` (predefinito): `npm install -g openclaw@latest`
* `git`: clonare/compilare da GitHub ed eseguire da una checkout del codice sorgente

<div id="cli-flags">
  ### Flag CLI
</div>

```bash
# npm esplicito
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method npm

# Installa da GitHub (checkout del sorgente)
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

Opzioni comuni:

* `--install-method npm|git`
* `--git-dir &lt;path&gt;` (predefinito: `~/openclaw`)
* `--no-git-update` (salta `git pull` quando utilizzi una copia locale esistente)
* `--no-prompt` (disabilita i prompt; richiesto in CI/automazione)
* `--dry-run` (mostra cosa accadrebbe; non apporta modifiche)
* `--no-onboard` (salta la procedura di onboarding iniziale)

<div id="environment-variables">
  ### Variabili di ambiente
</div>

Variabili di ambiente equivalenti (utili per l&#39;automazione):

* `OPENCLAW_INSTALL_METHOD=git|npm`
* `OPENCLAW_GIT_DIR=...`
* `OPENCLAW_GIT_UPDATE=0|1`
* `OPENCLAW_NO_PROMPT=1`
* `OPENCLAW_DRY_RUN=1`
* `OPENCLAW_NO_ONBOARD=1`
* `SHARP_IGNORE_GLOBAL_LIBVIPS=0|1` (valore predefinito: `1`; evita che `sharp` venga compilato contro la libvips di sistema)

<div id="troubleshooting-openclaw-not-found-path">
  ## Risoluzione problemi: `openclaw` non trovato (PATH)
</div>

Verifica rapida:

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

Se `$(npm prefix -g)/bin` (macOS/Linux) o `$(npm prefix -g)` (Windows) **non** compare nell&#39;output di `echo "$PATH"`, la tua shell non riesce a trovare i binari globali di npm (incluso `openclaw`).

Soluzione: aggiungilo al file di avvio della tua shell (zsh: `~/.zshrc`, bash: `~/.bashrc`):

```bash
# macOS / Linux
export PATH="$(npm prefix -g)/bin:$PATH"
```

Su Windows, aggiungi l&#39;output di `npm prefix -g` alla variabile PATH.

Quindi apri un nuovo terminale (oppure esegui `rehash` in zsh / `hash -r` in bash).

<div id="update-uninstall">
  ## Aggiornamento / disinstallazione
</div>

* Aggiornamento: [Aggiornare](/it/install/updating)
* Migrazione su una nuova macchina: [Migrazione](/it/install/migrating)
* Disinstallazione: [Disinstallare](/it/install/uninstall)