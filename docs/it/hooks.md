---
title: Hook
summary: "Hook: automazione basata su eventi per comandi ed eventi di ciclo di vita"
read_when:
  - Vuoi un'automazione basata su eventi per /new, /reset, /stop e per gli eventi di ciclo di vita degli agenti
  - Vuoi creare, installare o eseguire il debug degli hook
---

<div id="hooks">
  # Hooks
</div>

Gli hook forniscono un sistema a eventi estensibile per automatizzare le azioni in risposta ai comandi e agli eventi degli agenti. Gli hook vengono rilevati automaticamente nelle directory e possono essere gestiti tramite comandi CLI, in modo analogo al funzionamento delle abilità in OpenClaw.

<div id="getting-oriented">
  ## Orientarsi
</div>

Gli hook sono piccoli script che vengono eseguiti quando succede qualcosa. Ce ne sono di due tipi:

* **Hooks** (questa pagina): vengono eseguiti all&#39;interno del Gateway quando si verificano eventi dell&#39;agente, come `/new`, `/reset`, `/stop` o altri eventi di ciclo di vita.
* **Webhooks**: webhooks HTTP esterni che permettono ad altri sistemi di avviare operazioni in OpenClaw. Vedi [Webhook Hooks](/it/automation/webhook) oppure usa `openclaw webhooks` per i comandi di supporto a Gmail.

Gli hook possono anche essere inclusi all&#39;interno dei plugin; vedi [Plugins](/it/plugin#plugin-hooks).

Utilizzi comuni:

* Salvare un&#39;istantanea della memoria quando reimposti una sessione
* Mantenere una traccia di audit dei comandi per il troubleshooting o per la conformità
* Attivare automazioni successive quando una sessione inizia o termina
* Scrivere file nello spazio di lavoro dell&#39;agente o chiamare API esterne quando si verificano eventi

Se sai scrivere una piccola funzione TypeScript, puoi scrivere un hook. Gli hook vengono individuati automaticamente e puoi abilitarli o disabilitarli tramite la CLI.

<div id="overview">
  ## Panoramica
</div>

Il sistema di hook ti permette di:

* Salvare il contesto della sessione in memoria quando esegui il comando `/new`
* Registrare tutti i comandi per finalità di audit
* Attivare automazioni personalizzate sugli eventi del ciclo di vita degli agenti
* Estendere il comportamento di OpenClaw senza modificare il codice principale

<div id="getting-started">
  ## Guida introduttiva
</div>

### Hook integrati

OpenClaw fornisce quattro hook predefiniti che vengono rilevati automaticamente:

* **💾 session-memory**: Salva il contesto della sessione nello spazio di lavoro del tuo agente (predefinito `~/.openclaw/workspace/memory/`) quando esegui `/new`
* **📝 command-logger**: Registra tutti gli eventi di comando in `~/.openclaw/logs/commands.log`
* **🚀 boot-md**: Esegue `BOOT.md` all&#39;avvio del Gateway (richiede gli hook interni abilitati)
* **😈 soul-evil**: Sostituisce il contenuto iniettato di `SOUL.md` con `SOUL_EVIL.md` durante una purge window o in maniera casuale

Per elencare gli hook disponibili:

```bash
openclaw hooks list
```

Abilita un hook:

```bash
openclaw hooks enable session-memory
```

Verifica lo stato dell&#39;hook:

```bash
openclaw hooks check
```

Visualizza informazioni dettagliate:

```bash
openclaw hooks info session-memory
```

<div id="onboarding">
  ### Onboarding
</div>

Durante la procedura di onboarding (`openclaw onboard`), ti verrà chiesto di abilitare gli hook consigliati. La procedura guidata rileva automaticamente gli hook idonei e te li mostra per la scelta.

<div id="hook-discovery">
  ## Individuazione degli hook
</div>

Gli hook vengono rilevati automaticamente da tre directory (in ordine di precedenza):

1. **Hook dello spazio di lavoro**: `<workspace>/hooks/` (per singolo agente, massima precedenza)
2. **Hook gestiti**: `~/.openclaw/hooks/` (installati dall&#39;utente, condivisi tra spazi di lavoro)
3. **Hook in bundle**: `<openclaw>/dist/hooks/bundled/` (forniti con OpenClaw)

Le directory degli hook gestiti possono essere un **singolo hook** oppure un **pacchetto di hook** (directory di pacchetto).

Ogni hook è una directory che contiene:

```
my-hook/
├── HOOK.md          # Metadati + documentazione
└── handler.ts       # Implementazione handler
```

<div id="hook-packs-npmarchives">
  ## Pacchetti di hook (npm/archivi)
</div>

I pacchetti di hook sono pacchetti npm standard che esportano uno o più hook tramite `openclaw.hooks` nel file
`package.json`. Installali con:

```bash
openclaw hooks install <path-or-spec>
```

Esempio di `package.json`:

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "openclaw": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

Ogni voce fa riferimento a una directory hook che contiene `HOOK.md` e `handler.ts` (o `index.ts`).
Gli hook pack possono includere dipendenze; queste verranno installate in `~/.openclaw/hooks/<id>`.

<div id="hook-structure">
  ## Struttura degli hook
</div>

<div id="hookmd-format">
  ### Formato di HOOK.md
</div>

Il file `HOOK.md` contiene metadati nel front matter YAML e documentazione in Markdown:

```markdown
---
name: my-hook
description: "Short description of what this hook does"
homepage: https://docs.openclaw.ai/hooks#my-hook
metadata: {"openclaw":{"emoji":"🔗","events":["command:new"],"requires":{"bins":["node"]}}}
---

# My Hook

Detailed documentation goes here...

## What It Does

- Listens for `/new` commands
- Performs some action
- Logs the result

## Requirements

- Node.js must be installed

## Configuration

No configuration needed.
```

<div id="metadata-fields">
  ### Campi di metadati
</div>

L&#39;oggetto `metadata.openclaw` supporta:

* **`emoji`**: Emoji da visualizzare nella CLI (ad es. `"💾"`)
* **`events`**: Array di eventi da gestire (ad es. `["command:new", "command:reset"]`)
* **`export`**: Esportazione nominata da usare (valore predefinito `"default"`)
* **`homepage`**: URL della documentazione
* **`requires`**: Requisiti opzionali
  * **`bins`**: Binari richiesti nel PATH (ad es. `["git", "node"]`)
  * **`anyBins`**: Deve essere presente almeno uno di questi binari
  * **`env`**: Variabili d&#39;ambiente richieste
  * **`config`**: Percorsi di configurazione richiesti (ad es. `["workspace.dir"]`)
  * **`os`**: Piattaforme richieste (ad es. `["darwin", "linux"]`)
* **`always`**: Ignora i controlli sui requisiti (booleano)
* **`install`**: Metodi di installazione (per gli hook inclusi: `[{"id":"bundled","kind":"bundled"}]`)

<div id="handler-implementation">
  ### Implementazione dell&#39;handler
</div>

Il file `handler.ts` esporta una funzione `HookHandler`:

```typescript
import type { HookHandler } from '../../src/hooks/hooks.js';

const myHandler: HookHandler = async (event) => {
  // Only trigger on 'new' command
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  console.log(`[my-hook] New command triggered`);
  console.log(`  Session: ${event.sessionKey}`);
  console.log(`  Timestamp: ${event.timestamp.toISOString()}`);

  // Your custom logic here

  // Optionally send message to user
  event.messages.push('✨ Il mio hook è stato eseguito!');
};

export default myHandler;
```

<div id="event-context">
  #### Contesto dell&#39;evento
</div>

Ogni evento include:

```typescript
{
  type: 'command' | 'session' | 'agent' | 'gateway',
  action: string,              // e.g., 'new', 'reset', 'stop'
  sessionKey: string,          // Session identifier
  timestamp: Date,             // When the event occurred
  messages: string[],          // Inserisci qui i messaggi da inviare all'utente
  context: {
    sessionEntry?: SessionEntry,
    sessionId?: string,
    sessionFile?: string,
    commandSource?: string,    // e.g., 'whatsapp', 'telegram'
    senderId?: string,
    workspaceDir?: string,
    bootstrapFiles?: WorkspaceBootstrapFile[],
    cfg?: OpenClawConfig
  }
}
```

<div id="event-types">
  ## Tipi di eventi
</div>

<div id="command-events">
  ### Eventi di comando
</div>

Attivati quando vengono inviati comandi dell&#39;agente:

* **`command`**: Tutti gli eventi di comando (listener generale)
* **`command:new`**: Quando viene inviato il comando `/new`
* **`command:reset`**: Quando viene inviato il comando `/reset`
* **`command:stop`**: Quando viene inviato il comando `/stop`

<div id="agent-events">
  ### Eventi dell&#39;Agente
</div>

* **`agent:bootstrap`**: Prima che i file di bootstrap dello spazio di lavoro vengano inseriti (gli hook possono modificare `context.bootstrapFiles`)

<div id="gateway-events">
  ### Eventi del Gateway
</div>

Viene emesso all&#39;avvio del Gateway:

* **`gateway:startup`**: dopo l&#39;avvio dei canali e il caricamento degli hook

<div id="tool-result-hooks-plugin-api">
  ### Hook sui risultati degli strumenti (Plugin API)
</div>

Questi hook non sono listener di stream di eventi; consentono ai plugin di modificare in modo sincrono i risultati degli strumenti prima che OpenClaw li memorizzi.

* **`tool_result_persist`**: trasforma i risultati degli strumenti prima che vengano scritti nella trascrizione della sessione. Deve essere sincrono; deve restituire il payload del risultato dello strumento aggiornato oppure `undefined` per mantenerlo invariato. Vedi [Agent Loop](/it/concepts/agent-loop).

<div id="future-events">
  ### Eventi futuri
</div>

Tipi di eventi previsti:

* **`session:start`**: Quando inizia una nuova sessione
* **`session:end`**: Quando termina una sessione
* **`agent:error`**: Quando un agente rileva un errore
* **`message:sent`**: Quando un messaggio viene inviato
* **`message:received`**: Quando un messaggio viene ricevuto

<div id="creating-custom-hooks">
  ## Creare hook personalizzati
</div>

<div id="1-choose-location">
  ### 1. Scegli la posizione
</div>

* **Workspace hooks** (`<workspace>/hooks/`): Per agente, con la precedenza più alta
* **Managed hooks** (`~/.openclaw/hooks/`): Condivisi tra gli spazi di lavoro

<div id="2-create-directory-structure">
  ### 2. Crea la struttura delle directory
</div>

```bash
mkdir -p ~/.openclaw/hooks/my-hook
cd ~/.openclaw/hooks/my-hook
```

<div id="3-create-hookmd">
  ### 3. Crea HOOK.md
</div>

```markdown
---
name: my-hook
description: "Does something useful"
metadata: {"openclaw":{"emoji":"🎯","events":["command:new"]}}
---

# My Custom Hook

This hook does something useful when you issue `/new`.
```

<div id="4-create-handlerts">
  ### 4. Crea il file handler.ts
</div>

```typescript
import type { HookHandler } from '../../src/hooks/hooks.js';

const handler: HookHandler = async (event) => {
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  console.log('[my-hook] Running!');
  // La tua logica qui
};

export default handler;
```

<div id="5-enable-and-test">
  ### 5. Abilita e verifica
</div>

```bash
# Verifica che l'hook sia stato rilevato
openclaw hooks list

# Abilitalo
openclaw hooks enable my-hook

# Riavvia il processo del Gateway (riavvio dell'app dalla barra dei menu su macOS, oppure riavvia il processo di sviluppo)

# Attiva l'evento
# Invia /new tramite il tuo canale di messaggistica
```

<div id="configuration">
  ## Configurazione
</div>

<div id="new-config-format-recommended">
  ### Nuovo formato di configurazione (raccomandato)
</div>

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "session-memory": { "enabled": true },
        "command-logger": { "enabled": false }
      }
    }
  }
}
```

<div id="per-hook-configuration">
  ### Configurazione specifica per hook
</div>

Gli hook possono avere una configurazione personalizzata:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "my-hook": {
          "enabled": true,
          "env": {
            "MY_CUSTOM_VAR": "value"
          }
        }
      }
    }
  }
}
```

<div id="extra-directories">
  ### Directory aggiuntive
</div>

Carica hook da directory aggiuntive:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "load": {
        "extraDirs": ["/path/to/more/hooks"]
      }
    }
  }
}
```

<div id="legacy-config-format-still-supported">
  ### Formato di configurazione legacy (ancora supportato)
</div>

Il vecchio formato di configurazione è ancora supportato per garantire la retrocompatibilità:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts",
          "export": "default"
        }
      ]
    }
  }
}
```

**Migrazione**: Usa il nuovo sistema di discovery per i nuovi hook. Gli handler legacy vengono caricati dopo gli hook basati su directory.

<div id="cli-commands">
  ## Comandi CLI
</div>

<div id="list-hooks">
  ### Elenca gli hook
</div>

```bash
# List all hooks
openclaw hooks list

# Show only eligible hooks
openclaw hooks list --eligible

# Output dettagliato (mostra i requisiti mancanti)
openclaw hooks list --verbose

# JSON output
openclaw hooks list --json
```

<div id="hook-information">
  ### Dettagli hook
</div>

```bash
# Mostra informazioni dettagliate su un hook
openclaw hooks info session-memory

# JSON output
openclaw hooks info session-memory --json
```

<div id="check-eligibility">
  ### Verifica dei requisiti
</div>

```bash
# Mostra riepilogo di idoneità
openclaw hooks check

# JSON output
openclaw hooks check --json
```

<div id="enabledisable">
  ### Abilitare/Disabilitare
</div>

```bash
# Enable a hook
openclaw hooks enable session-memory

# Disabilita un hook
openclaw hooks disable command-logger
```

<div id="bundled-hooks">
  ## Hook predefiniti
</div>

<div id="session-memory">
  ### session-memory
</div>

Salva il contesto della sessione in memoria quando esegui il comando `/new`.

**Eventi**: `command:new`

**Requisiti**: `workspace.dir` deve essere configurato

**Output**: `<workspace>/memory/YYYY-MM-DD-slug.md` (per impostazione predefinita `~/.openclaw/workspace`)

**Cosa fa**:

1. Usa la voce della sessione prima del reset per individuare la trascrizione corretta
2. Estrae le ultime 15 righe della conversazione
3. Usa un LLM per generare uno slug descrittivo per il nome file
4. Salva i metadati della sessione in un file di memoria con data

**Esempio di output**:

```markdown
# Sessione: 2026-01-16 14:30:00 UTC

- **Chiave sessione**: agent:main:main
- **ID sessione**: abc123def456
- **Origine**: telegram
```

**Esempi di nomi di file**:

* `2026-01-16-vendor-pitch.md`
* `2026-01-16-api-design.md`
* `2026-01-16-1430.md` (timestamp di fallback se la generazione dello slug non riesce)

**Abilita**:

```bash
openclaw hooks enable session-memory
```

<div id="command-logger">
  ### command-logger
</div>

Registra tutti gli eventi relativi ai comandi in un file di audit centralizzato.

**Eventi**: `command`

**Requisiti**: Nessuno

**Output**: `~/.openclaw/logs/commands.log`

**Cosa fa**:

1. Acquisisce i dettagli dell&#39;evento (azione del comando, timestamp, chiave di sessione, ID mittente, origine)
2. Accoda i dati al file di log in formato JSONL
3. Viene eseguito in background in modo silenzioso

**Esempi di voci di log**:

```jsonl
{"timestamp":"2026-01-16T14:30:00.000Z","action":"new","sessionKey":"agent:main:main","senderId":"+1234567890","source":"telegram"}
{"timestamp":"2026-01-16T15:45:22.000Z","action":"stop","sessionKey":"agent:main:main","senderId":"user@example.com","source":"whatsapp"}
```

**Visualizza log**:

```bash
# Visualizza i comandi recenti
tail -n 20 ~/.openclaw/logs/commands.log

# Pretty-print with jq
cat ~/.openclaw/logs/commands.log | jq .

# Filter by action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**Attiva**:

```bash
openclaw hooks enable command-logger
```

<div id="soul-evil">
  ### soul-evil
</div>

Sostituisce il contenuto `SOUL.md` iniettato con `SOUL_EVIL.md` durante una finestra di eliminazione o in modo casuale.

**Eventi**: `agent:bootstrap`

**Documentazione**: [SOUL Evil Hook](/it/hooks/soul-evil)

**Output**: Nessun file viene scritto; le sostituzioni avvengono solo in memoria.

**Abilita**:

```bash
openclaw hooks enable soul-evil
```

**Config**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 0.1,
          "purge": { "at": "21:00", "duration": "15m" }
        }
      }
    }
  }
}
```

<div id="boot-md">
  ### boot-md
</div>

Esegue `BOOT.md` all&#39;avvio del Gateway (dopo l&#39;avvio dei canali).
Gli hook interni devono essere abilitati perché questo venga eseguito.

**Eventi**: `gateway:startup`

**Requisiti**: `workspace.dir` deve essere configurato

**Cosa fa**:

1. Legge `BOOT.md` dal tuo spazio di lavoro
2. Esegue le istruzioni tramite il runner dell&#39;agente
3. Invia eventuali messaggi in uscita richiesti tramite lo strumento di messaggistica

**Abilita**:

```bash
openclaw hooks enable boot-md
```

<div id="best-practices">
  ## Migliori pratiche
</div>

<div id="keep-handlers-fast">
  ### Mantieni i gestori rapidi
</div>

Gli hook vengono eseguiti durante l&#39;elaborazione dei comandi. Mantienili leggeri:

```typescript
// ✓ Good - async work, returns immediately
const handler: HookHandler = async (event) => {
  void processInBackground(event); // Fire and forget
};

// ✗ Cattivo - blocca l'elaborazione dei comandi
const handler: HookHandler = async (event) => {
  await slowDatabaseQuery(event);
  await evenSlowerAPICall(event);
};
```

<div id="handle-errors-gracefully">
  ### Gestisci gli errori in modo efficace
</div>

Racchiudi sempre le operazioni rischiose:

```typescript
const handler: HookHandler = async (event) => {
  try {
    await riskyOperation(event);
  } catch (err) {
    console.error('[my-handler] Failed:', err instanceof Error ? err.message : String(err));
    // Non lanciare eccezioni - consenti l'esecuzione degli altri handler
  }
};
```

<div id="filter-events-early">
  ### Filtra gli eventi il prima possibile
</div>

Restituisci immediatamente se l&#39;evento non è rilevante:

```typescript
const handler: HookHandler = async (event) => {
  // Gestisci solo i comandi 'new'
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  // Inserisci qui la tua logica
};
```

<div id="use-specific-event-keys">
  ### Usa chiavi evento specifiche
</div>

Quando possibile, indica nei metadati le singole chiavi evento specifiche:

```yaml
metadata: {"openclaw":{"events":["command:new"]}}  # Specifico
```

Invece di:

```yaml
metadata: {"openclaw":{"events":["command"]}}      # Generico - overhead maggiore
```

<div id="debugging">
  ## Debug
</div>

<div id="enable-hook-logging">
  ### Abilitare la registrazione nei log degli hook
</div>

Il Gateway registra nei log il caricamento degli hook all&#39;avvio:

```
Registered hook: session-memory -> command:new
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

<div id="check-discovery">
  ### Verificare il rilevamento
</div>

Elenca tutti gli hook rilevati:

```bash
openclaw hooks list --verbose
```

<div id="check-registration">
  ### Verifica della registrazione
</div>

Nel tuo handler, registra un messaggio di log quando viene invocato:

```typescript
const handler: HookHandler = async (event) => {
  console.log('[my-handler] Triggered:', event.type, event.action);
  // La tua logica qui
};
```

<div id="verify-eligibility">
  ### Verifica idoneità
</div>

Controlla perché un hook non è idoneo:

```bash
openclaw hooks info my-hook
```

Verifica la presenza di requisiti mancanti nell&#39;output.

<div id="testing">
  ## Test
</div>

<div id="gateway-logs">
  ### Log del Gateway
</div>

Monitora i log del Gateway per verificare l&#39;esecuzione degli hook:

```bash
# macOS
./scripts/clawlog.sh -f

# Altre piattaforme
tail -f ~/.openclaw/gateway.log
```

<div id="test-hooks-directly">
  ### Testa direttamente gli hook
</div>

Testa i tuoi handler in isolamento:

```typescript
import { test } from 'vitest';
import { createHookEvent } from './src/hooks/hooks.js';
import myHandler from './hooks/my-hook/handler.js';

test('il mio handler funziona', async () => {
  const event = createHookEvent('command', 'new', 'test-session', {
    foo: 'bar'
  });

  await myHandler(event);

  // Assert side effects
});
```

<div id="architecture">
  ## Architettura
</div>

<div id="core-components">
  ### Componenti principali
</div>

* **`src/hooks/types.ts`**: Definizioni di tipi
* **`src/hooks/workspace.ts`**: Scansione e caricamento delle directory
* **`src/hooks/frontmatter.ts`**: Parsing dei metadati di HOOK.md
* **`src/hooks/config.ts`**: Verifica dei criteri di idoneità
* **`src/hooks/hooks-status.ts`**: Segnalazione dello stato
* **`src/hooks/loader.ts`**: Caricatore dinamico di moduli
* **`src/cli/hooks-cli.ts`**: Comandi CLI
* **`src/gateway/server-startup.ts`**: Carica gli hook all&#39;avvio del Gateway
* **`src/auto-reply/reply/commands-core.ts`**: Genera eventi di comando

<div id="discovery-flow">
  ### Flusso di individuazione
</div>

```
Gateway startup
    ↓
Scan directories (workspace → managed → bundled)
    ↓
Parse HOOK.md files
    ↓
Check eligibility (bins, env, config, os)
    ↓
Load handlers from eligible hooks
    ↓
Register handlers for events
```

<div id="event-flow">
  ### Flusso degli eventi
</div>

```
User sends /new
    ↓
Command validation
    ↓
Create hook event
    ↓
Trigger hook (all registered handlers)
    ↓
Command processing continues
    ↓
Session reset
```

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

<div id="hook-not-discovered">
  ### Hook Non Rilevato
</div>

1. Controlla la struttura della directory:
   ```bash
   ls -la ~/.openclaw/hooks/my-hook/
   # Dovrebbe mostrare: HOOK.md, handler.ts
   ```

2. Verifica il formato di HOOK.md:
   ```bash
   cat ~/.openclaw/hooks/my-hook/HOOK.md
   # Dovrebbe avere una front matter YAML con nome e metadati
   ```

3. Elenca tutti gli hook rilevati:
   ```bash
   openclaw hooks list
   ```

<div id="hook-not-eligible">
  ### Hook non idoneo
</div>

Controlla i requisiti:

```bash
openclaw hooks info my-hook
```

Verifica che non manchino:

* Eseguibili (controlla il PATH)
* Variabili di ambiente
* Valori di configurazione
* Compatibilità con il sistema operativo

<div id="hook-not-executing">
  ### Hook non viene eseguito
</div>

1. Verifica che l&#39;hook sia abilitato:
   ```bash
   openclaw hooks list
   # Dovrebbe mostrare ✓ accanto agli hook abilitati
   ```

2. Riavvia il processo del Gateway in modo che gli hook vengano ricaricati.

3. Controlla i log del Gateway per eventuali errori:
   ```bash
   ./scripts/clawlog.sh | grep hook
   ```

<div id="handler-errors">
  ### Errori dell&#39;handler
</div>

Controlla la presenza di errori di TypeScript o di importazione:

```bash
# Testa l'importazione direttamente
node -e "import('./path/to/handler.ts').then(console.log)"
```

<div id="migration-guide">
  ## Guida alla migrazione
</div>

<div id="from-legacy-config-to-discovery">
  ### Dalla configurazione legacy a Discovery
</div>

**Prima**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts"
        }
      ]
    }
  }
}
```

**Dopo**:

1. Crea la directory per l&#39;hook:
   ```bash
   mkdir -p ~/.openclaw/hooks/my-hook
   mv ./hooks/handlers/my-handler.ts ~/.openclaw/hooks/my-hook/handler.ts
   ```

2. Crea HOOK.md:
   ```markdown
   ---
   name: my-hook
   description: "Il mio hook personalizzato"
   metadata: {"openclaw":{"emoji":"🎯","events":["command:new"]}}
   ---

   # Il mio Hook

   Fa qualcosa di utile.
   ```

3. Aggiorna la configurazione:
   ```json
   {
     "hooks": {
       "internal": {
         "enabled": true,
         "entries": {
           "my-hook": { "enabled": true }
         }
       }
     }
   }
   ```

4. Verifica e riavvia il processo del Gateway:
   ```bash
   openclaw hooks list
   # Dovrebbe mostrare: 🎯 my-hook ✓
   ```

**Vantaggi della migrazione**:

* Scoperta automatica
* Gestione tramite CLI
* Verifica dell&#39;idoneità
* Documentazione migliore
* Struttura coerente

<div id="see-also">
  ## Vedi anche
</div>

* [Riferimento CLI: hook](/it/cli/hooks)
* [README hook integrati](https://github.com/openclaw/openclaw/tree/main/src/hooks/bundled)
* [Hook webhook](/it/automation/webhook)
* [Configurazione](/it/gateway/configuration#hooks)