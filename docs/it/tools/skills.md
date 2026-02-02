---
title: Abilità
summary: "Abilità: gestite vs nello spazio di lavoro, regole di gating e collegamento di config/variabili d'ambiente"
read_when:
  - Aggiunta o modifica delle abilità
  - Modifica del gating o delle regole di caricamento delle abilità
---

<div id="skills-openclaw">
  # Abilità (OpenClaw)
</div>

OpenClaw utilizza cartelle di abilità **compatibili con [AgentSkills](https://agentskills.io)** per insegnare all&#39;agente come usare gli strumenti. Ogni abilità è una directory che contiene un file `SKILL.md` con front matter YAML e istruzioni. OpenClaw carica le **abilità incluse nel pacchetto** più eventuali override locali facoltativi e le filtra durante il caricamento in base all&#39;ambiente, alla configurazione e alla presenza dei binari.

<div id="locations-and-precedence">
  ## Posizioni e precedenza
</div>

Le abilità vengono caricate da **tre** posizioni:

1. **Abilità integrate**: fornite con l&#39;installazione (pacchetto npm o OpenClaw.app)
2. **Abilità gestite/locali**: `~/.openclaw/skills`
3. **Abilità dello spazio di lavoro**: `<workspace>/skills`

Se un nome di abilità va in conflitto, la precedenza è:

`<workspace>/skills` (massima) → `~/.openclaw/skills` → abilità integrate (minima)

Inoltre, puoi configurare cartelle aggiuntive per le abilità (precedenza più bassa) tramite
`skills.load.extraDirs` in `~/.openclaw/openclaw.json`.

<div id="per-agent-vs-shared-skills">
  ## Abilità per-agente vs condivise
</div>

In configurazioni **multi-agente**, ogni agente ha il proprio spazio di lavoro. Questo significa che:

* Le **abilità per-agente** si trovano in `<workspace>/skills` solo per quell&#39;agente.
* Le **abilità condivise** si trovano in `~/.openclaw/skills` (gestite/locali) e sono visibili
  a **tutti gli agenti** sulla stessa macchina.
* Le **cartelle condivise** possono anche essere aggiunte tramite `skills.load.extraDirs` (precedenza
  più bassa) se vuoi un pacchetto di abilità comune utilizzato da più agenti.

Se lo stesso nome di abilità esiste in più di una posizione, si applica la solita precedenza:
lo spazio di lavoro ha la precedenza, poi le abilità gestite/locali, poi quelle incluse nel bundle.

<div id="plugins-skills">
  ## Plugin + abilità
</div>

I plugin possono fornire le proprie abilità dichiarando le directory `skills` in
`openclaw.plugin.json` (percorsi relativi alla root del plugin). Le abilità del plugin vengono caricate
quando il plugin è abilitato e seguono le normali regole di precedenza delle abilità.
Puoi vincolarne l’utilizzo tramite `metadata.openclaw.requires.config` nella voce di configurazione
del plugin. Vedi [Plugins](/it/plugin) per scoperta/configurazione e [Tools](/it/tools) per la
superficie di strumenti che queste abilità espongono.

<div id="clawhub-install-sync">
  ## ClawHub (installazione + sincronizzazione)
</div>

ClawHub è il registro pubblico delle abilità per OpenClaw. Esploralo su
https://clawhub.com. Usalo per scoprire, installare, aggiornare ed eseguire il backup delle abilità.
Guida completa: [ClawHub](/it/tools/clawhub).

Flussi comuni:

* Installa un&#39;abilità nel tuo spazio di lavoro:
  * `clawhub install <skill-slug>`
* Aggiorna tutte le abilità installate:
  * `clawhub update --all`
* Sincronizza (scansione + pubblicazione degli aggiornamenti):
  * `clawhub sync --all`

Per impostazione predefinita, `clawhub` installa in `./skills` nella tua directory di lavoro
corrente (oppure, in alternativa, usa lo spazio di lavoro OpenClaw configurato). OpenClaw lo
rileva come `<workspace>/skills` alla sessione successiva.

<div id="security-notes">
  ## Note sulla sicurezza
</div>

* Considera le abilità di terze parti come **codice attendibile**. Leggile prima di abilitarle.
* Preferisci esecuzioni in sandbox per input non attendibili e strumenti rischiosi. Consulta [Sandboxing](/it/gateway/sandboxing).
* `skills.entries.*.env` e `skills.entries.*.apiKey` iniettano segreti nel processo **host**
  per quel turno di esecuzione dell&#39;agente (non nella sandbox). Non includere i segreti nei prompt o nei log.
* Per un modello di minaccia più ampio e le relative checklist, consulta [Security](/it/gateway/security).

<div id="format-agentskills-pi-compatible">
  ## Formato (AgentSkills + compatibile con Pi)
</div>

`SKILL.md` deve contenere almeno:

```markdown
---
name: nano-banana-pro
description: Genera o modifica immagini tramite Gemini 3 Pro Image
---
```

Note:

* Seguiamo la specifica AgentSkills per layout e intent.
* Il parser usato dall&#39;agente incorporato supporta solo chiavi di frontmatter **su singola riga**.
* `metadata` deve essere un **oggetto JSON su singola riga**.
* Usa `{baseDir}` nelle istruzioni per fare riferimento al percorso della cartella della skill.
* Chiavi di frontmatter opzionali:
  * `homepage` — URL mostrato come “Sito web” nella macOS Skills UI (supportato anche tramite `metadata.openclaw.homepage`).
  * `user-invocable` — `true|false` (predefinito: `true`). Quando è `true`, la skill è esposta come comando slash per l’utente.
  * `disable-model-invocation` — `true|false` (predefinito: `false`). Quando è `true`, la skill è esclusa dal prompt del modello (resta comunque disponibile tramite invocazione da parte dell’utente).
  * `command-dispatch` — `tool` (opzionale). Quando impostato su `tool`, il comando slash bypassa il modello e viene inviato direttamente a uno strumento.
  * `command-tool` — nome dello strumento da invocare quando `command-dispatch: tool` è impostato.
  * `command-arg-mode` — `raw` (predefinito). Per il dispatch allo strumento, inoltra allo strumento la stringa di argomenti in forma grezza (nessun parsing da parte del core).

    Lo strumento viene invocato con i parametri:
    `{ command: "<raw args>", commandName: "<slash command>", skillName: "<skill name>" }`.

<div id="gating-load-time-filters">
  ## Gating (filtri in fase di caricamento)
</div>

OpenClaw **filtra le abilità in fase di caricamento** usando `metadata` (JSON in una singola riga):

```markdown
---
name: nano-banana-pro
description: Genera o modifica immagini tramite Gemini 3 Pro Image
metadata: {"openclaw":{"requires":{"bins":["uv"],"env":["GEMINI_API_KEY"],"config":["browser.enabled"]},"primaryEnv":"GEMINI_API_KEY"}}
---
```

Campi sotto `metadata.openclaw`:

* `always: true` — includi sempre l&#39;abilità (ignora le altre condizioni/gate).
* `emoji` — emoji facoltativo usato dalla macOS Skills UI.
* `homepage` — URL facoltativo mostrato come “Website” nella macOS Skills UI.
* `os` — elenco facoltativo di piattaforme (`darwin`, `linux`, `win32`). Se impostato, l&#39;abilità è idonea solo su quei sistemi operativi.
* `requires.bins` — elenco; ognuno deve esistere nel `PATH`.
* `requires.anyBins` — elenco; almeno uno deve esistere nel `PATH`.
* `requires.env` — elenco; la variabile di ambiente deve esistere **oppure** essere fornita nella configurazione.
* `requires.config` — elenco di percorsi in `openclaw.json` che devono risultare truthy (valutati come true).
* `primaryEnv` — nome della variabile di ambiente associata a `skills.entries.<name>.apiKey`.
* `install` — array facoltativo di specifiche di installazione usate dalla macOS Skills UI (brew/node/go/uv/download).

Nota sul sandboxing:

* `requires.bins` viene verificato sull’**host** al momento del caricamento dell&#39;abilità.
* Se un agente è in sandbox, il binario deve esistere anche **all&#39;interno del container**.
  Installalo tramite `agents.defaults.sandbox.docker.setupCommand` (o un&#39;immagine personalizzata).
  `setupCommand` viene eseguito una volta dopo la creazione del container.
  Le installazioni di pacchetti richiedono anche network egress, un filesystem di root scrivibile e un utente root nella sandbox.
  Esempio: l&#39;abilità `summarize` (`skills/summarize/SKILL.md`) necessita della CLI `summarize`
  nel container della sandbox per essere eseguita lì.

Esempio di installer:

```markdown
---
name: gemini
description: Use Gemini CLI for coding assistance and Google search lookups.
metadata: {"openclaw":{"emoji":"♊️","requires":{"bins":["gemini"]},"install":[{"id":"brew","kind":"brew","formula":"gemini-cli","bins":["gemini"],"label":"Install Gemini CLI (brew)"}]}}
---
```

Note:

* Se sono elencati più installer, il Gateway sceglie **una sola** opzione preferita (brew quando disponibile, altrimenti node).
* Se tutti gli installer sono `download`, OpenClaw elenca ogni voce così puoi vedere gli artefatti disponibili.
* Le specifiche dell&#39;installer possono includere `os: ["darwin"|"linux"|"win32"]` per filtrare le opzioni in base alla piattaforma.
* Le installazioni Node rispettano `skills.install.nodeManager` in `openclaw.json` (predefinito: npm; opzioni: npm/pnpm/yarn/bun).
  Questo influisce solo sulle **installazioni delle abilità**; il runtime del Gateway deve comunque essere Node
  (Bun non è raccomandato per WhatsApp/Telegram).
* Installazioni Go: se `go` manca e `brew` è disponibile, il Gateway installa prima Go tramite Homebrew e imposta `GOBIN` sulla `bin` di Homebrew quando possibile.
* Installazioni tramite download: `url` (obbligatorio), `archive` (`tar.gz` | `tar.bz2` | `zip`), `extract` (predefinito: automatico quando viene rilevato un archivio), `stripComponents`, `targetDir` (predefinito: `~/.openclaw/tools/<skillKey>`).

Se non è presente alcun `metadata.openclaw`, l&#39;abilità è sempre idonea (a meno che
non sia disabilitata nella configurazione o bloccata da `skills.allowBundled` per le abilità incluse di default).

<div id="config-overrides-openclawopenclawjson">
  ## Override della configurazione (`~/.openclaw/openclaw.json`)
</div>

Le abilità incluse/gestite possono essere attivate o disattivate e configurate tramite valori di variabili d&#39;ambiente:

```json5
{
  skills: {
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE"
        },
        config: {
          endpoint: "https://example.invalid",
          model: "nano-pro"
        }
      },
      peekaboo: { enabled: true },
      sag: { enabled: false }
    }
  }
}
```

Nota: se il nome dell&#39;abilità contiene dei trattini, racchiudi la chiave tra virgolette (JSON5 consente chiavi tra virgolette).

Per impostazione predefinita, le chiavi di configurazione corrispondono al **nome dell&#39;abilità**. Se un&#39;abilità definisce
`metadata.openclaw.skillKey`, usa quella chiave sotto `skills.entries`.

Regole:

* `enabled: false` disabilita l&#39;abilità anche se è inclusa/installata.
* `env`: iniettato **solo se** la variabile non è già impostata nel processo.
* `apiKey`: scorciatoia per le abilità che dichiarano `metadata.openclaw.primaryEnv`.
* `config`: contenitore opzionale per campi personalizzati per singola abilità; le chiavi personalizzate devono risiedere qui.
* `allowBundled`: lista di autorizzati opzionale solo per le abilità **bundled**. Se impostata, solo
  le abilità bundled nell&#39;elenco sono abilitate (le abilità gestite/di spazio di lavoro non vengono influenzate).

<div id="environment-injection-per-agent-run">
  ## Iniezione dell&#39;ambiente (per esecuzione di agente)
</div>

Quando ha inizio l&#39;esecuzione di un agente, OpenClaw:

1. Legge i metadati delle abilità.
2. Applica eventuali `skills.entries.<key>.env` o `skills.entries.<key>.apiKey` a
   `process.env`.
3. Costruisce il system prompt con le abilità **idonee**.
4. Ripristina l&#39;ambiente originale al termine dell&#39;esecuzione.

Questo è **limitato all&#39;esecuzione dell&#39;agente**, non a un ambiente shell globale.

<div id="session-snapshot-performance">
  ## Snapshot di sessione (prestazioni)
</div>

OpenClaw esegue uno snapshot delle abilità idonee **all’avvio di una sessione** e riutilizza quell’elenco per i turni successivi nella stessa sessione. Le modifiche alle abilità o alla configurazione hanno effetto a partire dalla sessione nuova successiva.

Le abilità possono anche essere aggiornate a sessione in corso quando il watcher delle abilità è abilitato o quando viene rilevato un nuovo nodo remoto idoneo (vedi sotto). Puoi considerarlo come un **hot reload**: l’elenco aggiornato viene utilizzato al turno successivo dell’agente.

<div id="remote-macos-nodes-linux-gateway">
  ## nodi macOS remoti (Gateway su Linux)
</div>

Se il Gateway è in esecuzione su Linux ma un **nodo macOS** è connesso **con `system.run` consentito** (approvazioni di sicurezza per Exec non impostate su `deny`), OpenClaw può considerare le abilità solo macOS come utilizzabili quando i binari richiesti sono presenti su quel nodo. L&#39;agente dovrebbe eseguire queste abilità tramite lo strumento `nodes` (in genere `nodes.run`).

Questo dipende dal fatto che il nodo dichiari il supporto ai comandi e da una verifica dei binari tramite `system.run`. Se in seguito il nodo macOS va offline, le abilità restano visibili; le invocazioni possono fallire finché il nodo non si riconnette.

<div id="skills-watcher-auto-refresh">
  ## Monitor delle abilità (aggiornamento automatico)
</div>

Per impostazione predefinita, OpenClaw monitora le cartelle delle abilità e aggiorna lo snapshot delle abilità ogni volta che i file `SKILL.md` cambiano. Configura questo comportamento in `skills.load`:

```json5
{
  skills: {
    load: {
      watch: true,
      watchDebounceMs: 250
    }
  }
}
```

<div id="token-impact-skills-list">
  ## Impatto sui token (elenco delle abilità)
</div>

Quando le abilità sono idonee, OpenClaw inserisce un elenco XML compatto delle abilità disponibili nel prompt di sistema (tramite `formatSkillsForPrompt` in `pi-coding-agent`). Il costo è deterministico:

* **Overhead di base (solo quando ≥1 abilità):** 195 caratteri.
* **Per abilità:** 97 caratteri + la lunghezza dei valori di `<name>`, `<description>` e `<location>` dopo l’escape XML.

Formula (caratteri):

```
total = 195 + Σ (97 + len(name_escaped) + len(description_escaped) + len(location_escaped))
```

Note:

* L’escaping XML converte `& < > " '` in entità (`&amp;`, `&lt;`, ecc.), aumentando la lunghezza del testo.
* Il conteggio dei token varia in base al tokenizzatore del modello. Una stima approssimativa in stile OpenAI è di ~4 caratteri/token, quindi **97 caratteri ≈ 24 token** per ogni skill, più le lunghezze effettive dei tuoi campi.

<div id="managed-skills-lifecycle">
  ## Ciclo di vita delle abilità gestite
</div>

OpenClaw fornisce un set di abilità di base come **abilità incluse nel pacchetto** parte
dell&#39;installazione (pacchetto npm o OpenClaw.app). `~/.openclaw/skills` esiste per
le sostituzioni locali (ad esempio, bloccare/applicare una patch a un&#39;abilità senza modificare la copia inclusa). Le abilità dello spazio di lavoro sono di proprietà dell&#39;utente e hanno la precedenza su entrambe in caso di conflitti di nome.

<div id="config-reference">
  ## Riferimento alla configurazione
</div>

Per lo schema di configurazione completo, vedi [Configurazione delle abilità](/it/tools/skills-config).

<div id="looking-for-more-skills">
  ## Cerchi altre abilità?
</div>

Sfoglia https://clawhub.com.

***