---
title: Prosa
summary: "OpenProse: workflow .prose, slash command e gestione dello stato in OpenClaw"
read_when:
  - Vuoi eseguire o scrivere workflow .prose
  - Vuoi abilitare il plugin OpenProse
  - Hai bisogno di comprendere la memorizzazione dello stato
---

<div id="openprose">
  # OpenProse
</div>

OpenProse è un formato di workflow portatile, incentrato su markdown, per orchestrare sessioni di intelligenza artificiale. In OpenClaw viene distribuito come plugin che installa un pacchetto di skill OpenProse e il comando slash `/prose`. I programmi risiedono in file `.prose` e possono avviare più sotto-agenti con flusso di controllo esplicito.

Sito ufficiale: https://www.prose.md

<div id="what-it-can-do">
  ## Cosa può fare
</div>

* Ricerca e sintesi multi-agente con parallelismo esplicito.
* Flussi di lavoro ripetibili e sicuri rispetto alle approvazioni (code review, triage degli incidenti, pipeline di contenuti).
* Programmi `.prose` riutilizzabili che puoi eseguire sui runtime di agente supportati.

<div id="install-enable">
  ## Installazione + abilitazione
</div>

I plugin forniti in bundle sono disabilitati per impostazione predefinita. Abilita OpenProse:

```bash
openclaw plugins enable open-prose
```

Riavvia il Gateway dopo aver abilitato il plugin.

Checkout locale di sviluppo: `openclaw plugins install ./extensions/open-prose`

Documentazione correlata: [Plugin](/it/plugin), [Manifest del plugin](/it/plugins/manifest), [Abilità](/it/tools/skills).

<div id="slash-command">
  ## Comando slash
</div>

OpenProse registra `/prose` come comando di skill che l&#39;utente può invocare. Instrada alle istruzioni della VM di OpenProse e utilizza gli strumenti di OpenClaw sotto il cofano.

Comandi comuni:

```
/prose help
/prose run <file.prose>
/prose run <handle/slug>
/prose run <https://example.com/file.prose>
/prose compile <file.prose>
/prose examples
/prose update
```

<div id="example-a-simple-prose-file">
  ## Esempio: un semplice file `.prose`
</div>

```prose
# Research + synthesis with two agents running in parallel.

input topic: "What should we research?"

agent researcher:
  model: sonnet
  prompt: "You research thoroughly and cite sources."

agent writer:
  model: opus
  prompt: "You write a concise summary."

parallel:
  findings = session: researcher
    prompt: "Research {topic}."
  draft = session: writer
    prompt: "Summarize {topic}."

session "Merge the findings + draft into a final answer."
context: { findings, draft }
```

<div id="file-locations">
  ## Posizione dei file
</div>

OpenProse mantiene lo stato nella directory `.prose/` nel tuo spazio di lavoro:

```
.prose/
├── .env
├── runs/
│   └── {YYYYMMDD}-{HHMMSS}-{random}/
│       ├── program.prose
│       ├── state.md
│       ├── bindings/
│       └── agents/
└── agents/
```

Gli agenti persistenti a livello utente sono memorizzati in:

```
~/.prose/agents/
```

<div id="state-modes">
  ## Modalità di stato
</div>

OpenProse supporta più backend di stato:

* **filesystem** (predefinito): `.prose/runs/...`
* **in-context**: archivio temporaneo, per programmi piccoli
* **sqlite** (sperimentale): richiede il binario `sqlite3`
* **postgres** (sperimentale): richiede `psql` e una stringa di connessione

Note:

* sqlite/postgres sono facoltativi e sperimentali.
* Le credenziali postgres finiscono nei log dei sub‑agenti; usa un database dedicato con privilegi minimi.

<div id="remote-programs">
  ## Programmi remoti
</div>

`/prose run <handle/slug>` viene risolto in `https://p.prose.md/<handle>/<slug>`.
Gli URL diretti vengono recuperati così come sono. Questo usa lo strumento `web_fetch` (oppure `exec` per le richieste POST).

<div id="openclaw-runtime-mapping">
  ## Mappatura del runtime di OpenClaw
</div>

I programmi OpenProse vengono mappati sulle primitive di OpenClaw:

| Concetto OpenProse | Strumento OpenClaw |
| --- | --- |
| Crea sessione / strumento Task | `sessions_spawn` |
| Lettura/scrittura file | `read` / `write` |
| Fetch dal web | `web_fetch` |

Se la tua lista di autorizzati degli strumenti li blocca, i programmi OpenProse non funzioneranno. Consulta [Configurazione delle abilità](/it/tools/skills-config).

<div id="security-approvals">
  ## Sicurezza + approvazioni
</div>

Tratta i file `.prose` come codice. Controllali prima di eseguirli. Usa le liste di autorizzati degli strumenti di OpenClaw e i gate di approvazione per controllare gli effetti collaterali.

Per workflow deterministici con approvazione obbligatoria, consulta [Lobster](/it/tools/lobster).