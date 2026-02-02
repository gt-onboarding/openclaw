---
title: Lobster
summary: "Runtime di workflow tipizzati per OpenClaw con gate di approvazione riprendibili."
description: Runtime di workflow tipizzati per OpenClaw — pipeline componibili con gate di approvazione.
read_when:
  - Vuoi workflow deterministici a più passaggi con approvazioni esplicite
  - Hai bisogno di riprendere un workflow senza rieseguire i passaggi precedenti
---

<div id="lobster">
  # Lobster
</div>

Lobster è una shell di workflow che permette a OpenClaw di eseguire sequenze di strumenti a più passaggi come un&#39;unica operazione deterministica, con checkpoint di approvazione espliciti.

<div id="hook">
  ## Hook
</div>

Il tuo assistente può costruire gli strumenti per auto-gestirsi. Chiedi un workflow e, 30 minuti dopo, hai una CLI e pipeline che vengono eseguite come un&#39;unica chiamata. Lobster è il tassello mancante: pipeline deterministiche, approvazioni esplicite e stato riprendibile.

<div id="why">
  ## Perché
</div>

Oggi i workflow complessi richiedono numerose chiamate di andata e ritorno agli strumenti. Ogni chiamata consuma token e l&#39;LLM deve orchestrare ogni singolo passaggio. Lobster sposta tale orchestrazione in un runtime tipizzato:

* **Una chiamata invece di molte**: OpenClaw esegue una chiamata a uno strumento Lobster e ottiene un risultato strutturato.
* **Approvazioni integrate**: Gli effetti collaterali (invio di email, pubblicazione di commenti) bloccano il workflow finché non vengono approvati esplicitamente.
* **Riprendibile**: I workflow bloccati restituiscono un token; approva e riprendi senza dover rieseguire tutto.

<div id="why-a-dsl-instead-of-plain-programs">
  ## Perché una DSL invece di normali programmi?
</div>

Lobster è intenzionalmente piccolo. L’obiettivo non è &quot;un nuovo linguaggio&quot;, ma una specifica di pipeline prevedibile e adatta all’uso con l’AI, con approvazioni di prima classe e token di ripresa.

* **L’approve/resume è integrato**: Un normale programma può richiedere input a un umano, ma non può *mettersi in pausa e riprendere* con un token durevole senza che tu ti inventi quel runtime da zero.
* **Determinismo + auditabilità**: Le pipeline sono dati, quindi è facile registrarli nei log, confrontarli (diff), riprodurli e analizzarli.
* **Superficie limitata per l’AI**: Una grammatica minima + piping JSON riducono i percorsi di codice “creativi” e rendono la validazione realistica.
* **Policy di sicurezza integrate**: Timeout, limiti di output, controlli di sandbox e liste di autorizzati sono applicati dal runtime, non da ogni singolo script.
* **Comunque programmabile**: Ogni step può chiamare qualsiasi CLI o script. Se vuoi usare JS/TS, genera file `.lobster` dal codice.

<div id="how-it-works">
  ## Come funziona
</div>

OpenClaw avvia la CLI locale `lobster` in **tool mode** e analizza un envelope JSON dallo stdout.
Se la pipeline viene messa in pausa per un’approvazione, lo strumento restituisce un `resumeToken` per consentirti di riprendere in seguito.

<div id="pattern-small-cli-json-pipes-approvals">
  ## Pattern: CLI minimale + pipeline JSON + approvazioni
</div>

Crea piccoli comandi che parlano JSON, quindi concatenali in un&#39;unica chiamata Lobster. (Nomi di comando di esempio qui sotto — sostituiscili con i tuoi.)

```bash
inbox list --json
inbox categorize --json
inbox apply --json
```

```json
{
  "action": "run",
  "pipeline": "exec --json --shell 'inbox list --json' | exec --stdin json --shell 'inbox categorize --json' | exec --stdin json --shell 'inbox apply --json' | approve --preview-from-stdin --limit 5 --prompt 'Applicare le modifiche?'",
  "timeoutMs": 30000
}
```

Se la pipeline richiede l&#39;approvazione, riprendi l&#39;esecuzione con il token:

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

L&#39;IA attiva il workflow; Lobster esegue i passaggi. I gate di approvazione mantengono gli effetti collaterali espliciti e tracciabili.

Esempio: mappare gli elementi di input in chiamate a tool:

```bash
gog.gmail.search --query 'newer_than:1d' \
  | openclaw.invoke --tool message --action send --each --item-key message --args-json '{"provider":"telegram","to":"..."}'
```

<div id="json-only-llm-steps-llm-task">
  ## Step LLM solo JSON (llm-task)
</div>

Per i workflow che necessitano di uno **step LLM strutturato**, abilita lo strumento
plugin opzionale `llm-task` e invocalo da Lobster. In questo modo il workflow
resta deterministico pur consentendoti di classificare/riassumere/redigere con un modello.

Abilita lo strumento:

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  },
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```

Utilizzalo in una pipeline:

```lobster
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Data l'email in input, restituisci intent e bozza.",
  "input": { "subject": "Hello", "body": "Can you help?" },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```

Consulta [LLM Task](/it/tools/llm-task) per i dettagli e le opzioni di configurazione.

<div id="workflow-files-lobster">
  ## File di workflow (.lobster)
</div>

Lobster può eseguire file di workflow YAML/JSON con i campi `name`, `args`, `steps`, `env`, `condition` e `approval`. Nelle chiamate agli strumenti di OpenClaw, imposta `pipeline` sul percorso del file.

```yaml
name: inbox-triage
args:
  tag:
    default: "family"
steps:
  - id: collect
    command: inbox list --json
  - id: categorize
    command: inbox categorize --json
    stdin: $collect.stdout
  - id: approve
    command: inbox apply --approve
    stdin: $categorize.stdout
    approval: required
  - id: execute
    command: inbox apply --execute
    stdin: $categorize.stdout
    condition: $approve.approved
```

Note:

* `stdin: $step.stdout` e `stdin: $step.json` passano l’output di uno step precedente.
* `condition` (o `when`) può subordinare l’esecuzione degli step a `$step.approved`.

<div id="install-lobster">
  ## Installa Lobster
</div>

Installa la CLI di Lobster **sullo stesso host** che esegue l’OpenClaw Gateway (vedi il [repository di Lobster](https://github.com/openclaw/lobster)) e assicurati che `lobster` sia nel `PATH`.
Se vuoi usare un percorso del binario personalizzato, passa un `lobsterPath` **assoluto** nella chiamata allo strumento.

<div id="enable-the-tool">
  ## Abilita lo strumento
</div>

Lobster è un plugin **opzionale** (non abilitato per impostazione predefinita).

Consigliato (aggiuntivo, sicuro):

```json
{
  "tools": {
    "alsoAllow": ["lobster"]
  }
}
```

Oppure per agente:

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": {
          "alsoAllow": ["lobster"]
        }
      }
    ]
  }
}
```

Evita di usare `tools.allow: ["lobster"]` a meno che tu non intenda eseguire in una modalità restrittiva basata su lista di autorizzati.

Nota: le liste di autorizzati sono opzionali per i plugin facoltativi. Se la tua lista di autorizzati include solo
strumenti plugin (come `lobster`), OpenClaw mantiene abilitati gli strumenti core. Per limitare gli strumenti core,
includi nella lista di autorizzati anche gli strumenti core o i gruppi che vuoi consentire.

<div id="example-email-triage">
  ## Esempio: smistamento delle email
</div>

Senza Lobster:

```
User: "Check my email and draft replies"
→ openclaw calls gmail.list
→ LLM summarizes
→ User: "draft replies to #2 and #5"
→ LLM drafts
→ User: "send #2"
→ openclaw calls gmail.send
(repeat daily, no memory of what was triaged)
```

Con Lobster:

```json
{
  "action": "run",
  "pipeline": "email.triage --limit 20",
  "timeoutMs": 30000
}
```

Restituisce un wrapper JSON (troncato):

```json
{
  "ok": true,
  "status": "needs_approval",
  "output": [{ "summary": "5 need replies, 2 need action" }],
  "requiresApproval": {
    "type": "approval_request",
    "prompt": "Inviare 2 bozze di risposta?",
    "items": [],
    "resumeToken": "..."
  }
}
```

Se l’utente approva → riprendi:

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

Un solo workflow. Deterministico. Sicuro.

<div id="tool-parameters">
  ## Parametri degli strumenti
</div>

<div id="run">
  ### `run`
</div>

Esegui una pipeline in modalità tool.

```json
{
  "action": "run",
  "pipeline": "gog.gmail.search --query 'newer_than:1d' | email.triage",
  "cwd": "/path/to/workspace",
  "timeoutMs": 30000,
  "maxStdoutBytes": 512000
}
```

Esegui un file di workflow con parametri:

```json
{
  "action": "run",
  "pipeline": "/path/to/inbox-triage.lobster",
  "argsJson": "{\"tag\":\"family\"}"
}
```

<div id="resume">
  ### `resume`
</div>

Continua l&#39;esecuzione di un flusso di lavoro interrotto dopo l&#39;approvazione.

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

<div id="optional-inputs">
  ### Input opzionali
</div>

* `lobsterPath`: Percorso assoluto del binario di Lobster (ometti per utilizzare `PATH`).
* `cwd`: Directory di lavoro per la pipeline (valore predefinito: la directory di lavoro del processo corrente).
* `timeoutMs`: Interrompe il sottoprocesso se supera questa durata (predefinito: 20000).
* `maxStdoutBytes`: Interrompe il sottoprocesso se l&#39;output su stdout supera questa dimensione (predefinito: 512000).
* `argsJson`: Stringa JSON passata a `lobster run --args-json` (solo per file di workflow).

<div id="output-envelope">
  ## Busta di output
</div>

Lobster restituisce una busta JSON con uno di tre stati:

* `ok` → completato correttamente
* `needs_approval` → in pausa; `requiresApproval.resumeToken` è necessario per riprendere
* `cancelled` → esplicitamente negato o annullato

Lo strumento rende disponibile la busta sia in `content` (JSON formattato) sia in `details` (oggetto grezzo).

<div id="approvals">
  ## Approvazioni
</div>

Se `requiresApproval` è presente, analizza il prompt e decidi:

* `approve: true` → riprendi ed esegui gli effetti collaterali
* `approve: false` → annulla e finalizza il workflow

Usa `approve --preview-from-stdin --limit N` per allegare un&#39;anteprima JSON alle richieste di approvazione senza bisogno di codice collante jq/heredoc personalizzato. I token di ripresa ora sono compatti: Lobster memorizza lo stato di ripresa del workflow nella propria directory di stato e restituisce una piccola chiave di token.

<div id="openprose">
  ## OpenProse
</div>

OpenProse si integra bene con Lobster: usa `/prose` per orchestrare la preparazione multi-agente, quindi esegui una pipeline Lobster per approvazioni deterministiche. Se un programma Prose necessita di Lobster, consenti lo strumento `lobster` per i sottoagenti tramite `tools.subagents.tools`. Vedi [OpenProse](/it/prose).

<div id="safety">
  ## Sicurezza
</div>

* **Solo sottoprocesso locale** — nessuna chiamata di rete effettuata direttamente dal plugin.
* **Nessun secret** — Lobster non gestisce OAuth; invoca gli strumenti di OpenClaw che se ne occupano.
* **Compatibile con sandbox** — disabilitato quando il contesto dello strumento è in sandbox.
* **Rafforzato** — `lobsterPath` deve essere assoluto se specificato; vengono applicati timeout e limiti all&#39;output.

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

* **`lobster subprocess timed out`** → aumenta `timeoutMs` oppure suddividi una pipeline troppo lunga.
* **`lobster output exceeded maxStdoutBytes`** → aumenta `maxStdoutBytes` o riduci la dimensione dell&#39;output.
* **`lobster returned invalid JSON`** → assicurati che la pipeline venga eseguita in modalità tool e produca solo JSON.
* **`lobster failed (code …)`** → esegui la stessa pipeline in un terminale per esaminare stderr.

<div id="learn-more">
  ## Per saperne di più
</div>

* [Plugin](/it/plugin)
* [Sviluppo di strumenti per plugin](/it/plugins/agent-tools)

<div id="case-study-community-workflows">
  ## Caso di studio: workflow della community
</div>

Un esempio pubblico: una CLI da “second brain” + pipeline Lobster che gestiscono tre vault Markdown (personale, partner, condiviso). La CLI emette JSON per statistiche, elenchi dell&#39;inbox e scansioni di elementi obsoleti; Lobster concatena questi comandi in workflow come `weekly-review`, `inbox-triage`, `memory-consolidation` e `shared-task-sync`, ciascuno con gate di approvazione. L&#39;IA gestisce la valutazione (categorizzazione) quando disponibile e, in caso contrario, ricorre a regole deterministiche.

* Thread: https://x.com/plattenschieber/status/2014508656335770033
* Repo: https://github.com/bloomedai/brain-cli