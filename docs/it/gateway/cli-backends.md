---
title: Backend CLI
summary: "Backend CLI: fallback solo testuale tramite CLI di AI locali"
read_when:
  - Vuoi un fallback affidabile quando i provider di API non funzionano
  - Stai eseguendo Claude Code CLI o altre CLI di AI locali e vuoi riutilizzarle
  - Hai bisogno di un flusso solo testuale, senza strumenti, che supporti comunque le sessioni e le immagini
---

<div id="cli-backends-fallback-runtime">
  # Backend CLI (runtime di fallback)
</div>

OpenClaw può eseguire **CLI di AI locali** come **fallback solo testo** quando i provider di API non sono disponibili,
soggetti a rate limiting o si comportano temporaneamente in modo anomalo. Questo comportamento è intenzionalmente conservativo:

- **Gli strumenti sono disabilitati** (nessuna chiamata a strumenti).
- **Testo in → testo fuori** (affidabile).
- **Le sessioni sono supportate** (quindi i turni successivi restano coerenti).
- **Le immagini possono essere inoltrate** se la CLI accetta percorsi di immagini.

Questo è progettato come **rete di sicurezza** piuttosto che come percorso principale. Usalo quando vuoi avere risposte testuali che “funzionano sempre” senza fare affidamento su API esterne.

<div id="beginner-friendly-quick-start">
  ## Avvio rapido per principianti
</div>

Puoi usare Claude Code CLI **senza alcuna configurazione** (OpenClaw fornisce una configurazione predefinita integrata):

```bash
openclaw agent --message "hi" --model claude-cli/opus-4.5
```

Anche Codex CLI funziona subito, pronta all&#39;uso:

```bash
openclaw agent --message "hi" --model codex-cli/gpt-5.2-codex
```

Se il tuo Gateway viene eseguito tramite launchd/systemd e PATH è minimale, aggiungi solo il percorso completo del comando:

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude"
        }
      }
    }
  }
}
```

Ecco fatto. Non servono chiavi né configurazioni di autenticazione aggiuntive oltre a quelle della CLI stessa.


<div id="using-it-as-a-fallback">
  ## Usarlo come fallback
</div>

Aggiungi un backend CLI all&#39;elenco di fallback in modo che venga eseguito solo quando i modelli primari falliscono:

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: [
          "claude-cli/opus-4.5"
        ]
      },
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "claude-cli/opus-4.5": {}
      }
    }
  }
}
```

Note:

* Se utilizzi `agents.defaults.models` (lista di autorizzati), devi includere `claude-cli/...`.
* Se il provider primario ha un errore (auth, limiti di frequenza, timeout), OpenClaw proverà
  il backend CLI come fallback successivo.


<div id="configuration-overview">
  ## Panoramica della configurazione
</div>

Tutti i backend CLI risiedono in:

```
agents.defaults.cliBackends
```

Ogni voce è indicizzata tramite un **ID del provider** (ad es. `claude-cli`, `my-cli`).
L&#39;ID del provider diventa la parte sinistra del riferimento al modello:

```
<provider>/<model>
```


<div id="example-configuration">
  ### Esempio di configurazione
</div>

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude"
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          input: "arg",
          modelArg: "--model",
          modelAliases: {
            "claude-opus-4-5": "opus",
            "claude-sonnet-4-5": "sonnet"
          },
          sessionArg: "--session",
          sessionMode: "existing",
          sessionIdFields: ["session_id", "conversation_id"],
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
          serialize: true
        }
      }
    }
  }
}
```


<div id="how-it-works">
  ## Come funziona
</div>

1) **Seleziona un backend** in base al prefisso del provider (`claude-cli/...`).
2) **Costruisce un prompt di sistema** usando lo stesso prompt OpenClaw e il contesto dello spazio di lavoro.
3) **Esegue la CLI** con un ID di sessione (se supportato), in modo che la cronologia rimanga coerente.
4) **Analizza l'output** (JSON o testo semplice) e restituisce il testo finale.
5) **Mantiene gli ID di sessione** per ogni backend, in modo che le richieste successive riutilizzino la stessa sessione CLI.

<div id="sessions">
  ## Sessioni
</div>

- Se la CLI supporta le sessioni, imposta `sessionArg` (ad esempio `--session-id`) oppure
  `sessionArgs` (segnaposto `{sessionId}`) quando l’id deve essere inserito
  in più flag.
- Se la CLI utilizza un **sottocomando resume** con flag diversi, imposta
  `resumeArgs` (sostituisce `args` durante il resume) e facoltativamente `resumeOutput`
  (per resume non-JSON).
- `sessionMode`:
  - `always`: invia sempre un id di sessione (nuovo UUID se nessuno è memorizzato).
  - `existing`: invia un id di sessione solo se ne è stato memorizzato uno in precedenza.
  - `none`: non inviare mai un id di sessione.

<div id="images-pass-through">
  ## Immagini (pass-through)
</div>

Se la tua CLI accetta percorsi di file immagine, imposta `imageArg`:

```json5
imageArg: "--image",
imageMode: "repeat"
```

OpenClaw scrive le immagini base64 in file temporanei. Se `imageArg` è impostato, tali
percorsi vengono passati alla CLI come argomenti. Se `imageArg` non è impostato, OpenClaw aggiunge i
percorsi dei file al prompt (iniezione del percorso), il che è sufficiente per le CLI che caricano
automaticamente i file locali da percorsi in chiaro (comportamento della Claude Code CLI).


<div id="inputs-outputs">
  ## Input / output
</div>

- `output: "json"` (predefinito) prova ad analizzare il JSON ed estrarre testo + ID della sessione.
- `output: "jsonl"` analizza flussi JSONL (Codex CLI `--json`) ed estrae
  l'ultimo messaggio dell'agente più `thread_id`, quando presente.
- `output: "text"` tratta stdout come risposta finale.

Modalità di input:

- `input: "arg"` (predefinito) passa il prompt come ultimo argomento della CLI.
- `input: "stdin"` invia il prompt tramite stdin.
- Se il prompt è molto lungo e `maxPromptArgChars` è impostato, viene utilizzato stdin.

<div id="defaults-built-in">
  ## Predefiniti (integrati)
</div>

OpenClaw fornisce una configurazione predefinita per `claude-cli`:

- `command: "claude"`
- `args: ["-p", "--output-format", "json", "--dangerously-skip-permissions"]`
- `resumeArgs: ["-p", "--output-format", "json", "--dangerously-skip-permissions", "--resume", "{sessionId}"]`
- `modelArg: "--model"`
- `systemPromptArg: "--append-system-prompt"`
- `sessionArg: "--session-id"`
- `systemPromptWhen: "first"`
- `sessionMode: "always"`

OpenClaw fornisce anche una configurazione predefinita per `codex-cli`:

- `command: "codex"`
- `args: ["exec","--json","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
- `resumeArgs: ["exec","resume","{sessionId}","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
- `output: "jsonl"`
- `resumeOutput: "text"`
- `modelArg: "--model"`
- `imageArg: "--image"`
- `sessionMode: "existing"`

Sostituiscile solo se necessario (caso comune: percorso assoluto di `command`).

<div id="limitations">
  ## Limitazioni
</div>

- **Nessun strumento OpenClaw** (il backend CLI non riceve mai chiamate agli strumenti). Alcune CLI
  possono comunque eseguire i propri strumenti di agente.
- **Nessuno streaming** (l’output della CLI viene raccolto e poi restituito).
- **Gli output strutturati** dipendono dal formato JSON della CLI.
- **Le sessioni Codex CLI** vengono riprese tramite output testuale (nessun JSONL), che è meno
  strutturato rispetto all’esecuzione iniziale con `--json`. Le sessioni OpenClaw continuano a funzionare
  normalmente.

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

- **CLI non trovata**: imposta `command` su un percorso assoluto.
- **Nome modello errato**: usa `modelAliases` per mappare `provider/model` → modello CLI.
- **Nessuna continuità di sessione**: assicurati che `sessionArg` sia impostato e che `sessionMode` non sia
  `none` (attualmente Codex CLI non può riprendere le sessioni con output JSON).
- **Immagini ignorate**: imposta `imageArg` (e verifica che la CLI supporti i percorsi di file).