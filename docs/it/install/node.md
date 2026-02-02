---
title: "Node.js + npm (PATH corretto)"
summary: "Verifica dell’installazione di Node.js + npm: versioni, PATH e installazioni globali"
read_when:
  - "Hai installato OpenClaw ma `openclaw` riporta “command not found”"
  - "Stai configurando Node.js/npm su una nuova macchina"
  - "npm install -g ... ha esito negativo per problemi di permessi o di PATH"
---

<div id="nodejs-npm-path-sanity">
  # Node.js + npm (verifica PATH)
</div>

Il requisito minimo di runtime per OpenClaw è **Node 22+**.

Se riesci a eseguire `npm install -g openclaw@latest` ma poi vedi `openclaw: command not found`, quasi sempre si tratta di un problema di **PATH**: la directory in cui npm inserisce i binari globali non è presente nel PATH della tua shell.

<div id="quick-diagnosis">
  ## Diagnosi rapida
</div>

Esegui:

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

Se `$(npm prefix -g)/bin` (macOS/Linux) o `$(npm prefix -g)` (Windows) **non** compare nell’output di `echo "$PATH"`, la tua shell non è in grado di trovare i binari globali di npm (incluso `openclaw`).


<div id="fix-put-npms-global-bin-dir-on-path">
  ## Soluzione: aggiungi la directory bin globale di npm al PATH
</div>

1. Individua il prefisso globale di npm:

```bash
npm prefix -g
```

2. Aggiungi la directory bin globale di npm al file di inizializzazione della shell:

* zsh: `~/.zshrc`
* bash: `~/.bashrc`

Esempio (sostituisci il percorso con l’output di `npm prefix -g`):

```bash
# macOS / Linux
export PATH="/path/from/npm/prefix/bin:$PATH"
```

Quindi apri un **nuovo terminale** (oppure esegui `rehash` in zsh / `hash -r` in bash).

Su Windows, aggiungi l&#39;output di `npm prefix -g` alla variabile PATH.


<div id="fix-avoid-sudo-npm-install-g-permission-errors-linux">
  ## Correzione: evita `sudo npm install -g` / errori di permesso (Linux)
</div>

Se `npm install -g ...` fallisce con errore `EACCES`, imposta il prefisso globale di npm su una directory in cui l’utente abbia permessi di scrittura:

```bash
mkdir -p "$HOME/.npm-global"
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

Rendi permanente la riga `export PATH=...` nel file di inizializzazione della shell.


<div id="recommended-node-install-options">
  ## Opzioni consigliate per l’installazione di Node
</div>

Avrai meno sorprese se Node/npm sono installati in modo tale da:

- mantenere Node aggiornato (22+)
- rendere la directory bin globale di npm stabile e presente nel PATH nelle nuove shell

Scelte comuni:

- macOS: Homebrew (`brew install node`) o un gestore di versioni
- Linux: il tuo gestore di versioni preferito o un’installazione supportata dalla distribuzione che fornisca Node 22+
- Windows: installer ufficiale di Node, `winget` o un gestore di versioni di Node per Windows

Se usi un gestore di versioni (nvm/fnm/asdf/etc.), assicurati che venga inizializzato nella shell che usi quotidianamente (zsh o bash), in modo che il PATH che imposta sia presente quando esegui gli installer.