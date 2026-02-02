---
title: Hook
summary: "Riferimento CLI per `openclaw hooks` (hook degli agenti)"
read_when:
  - Vuoi gestire gli hook degli agenti
  - Vuoi installare o aggiornare gli hook
---

<div id="openclaw-hooks">
  # `openclaw hooks`
</div>

Gestisci gli hook degli agenti (automazioni basate su eventi per comandi come `/new`, `/reset` e l&#39;avvio del Gateway).

Correlati:

* Hook: [Hooks](/it/hooks)
* Hook dei plugin: [Plugins](/it/plugin#plugin-hooks)

<div id="list-all-hooks">
  ## Elenca tutti gli hook
</div>

```bash
openclaw hooks list
```

Elenca tutti gli hook individuati nelle directory dello spazio di lavoro, gestite e fornite a corredo.

**Opzioni:**

* `--eligible`: Mostra solo gli hook idonei (requisiti soddisfatti)
* `--json`: Output in formato JSON
* `-v, --verbose`: Mostra informazioni dettagliate, inclusi i requisiti non soddisfatti

**Esempio di output:**

```
Hooks (4/4 pronti)

Pronti:
  🚀 boot-md ✓ - Esegui BOOT.md all'avvio del gateway
  📝 command-logger ✓ - Registra tutti gli eventi dei comandi in un file di audit centralizzato
  💾 session-memory ✓ - Salva il contesto della sessione in memoria quando viene emesso il comando /new
  😈 soul-evil ✓ - Scambia il contenuto SOUL iniettato durante una finestra di purge o casualmente
```

**Esempio (dettagliato):**

```bash
openclaw hooks list --verbose
```

Mostra i requisiti non soddisfatti per gli hook non idonei.

**Esempio (JSON):**

```bash
openclaw hooks list --json
```

Restituisce JSON strutturato per l&#39;uso da parte di programmi.

<div id="get-hook-information">
  ## Visualizzare le informazioni di un hook
</div>

```bash
openclaw hooks info <name>
```

Mostra informazioni dettagliate su uno specifico hook.

**Argomenti:**

* `<name>`: Nome dell&#39;hook (ad es. `session-memory`)

**Opzioni:**

* `--json`: Output in formato JSON

**Esempio:**

```bash
openclaw hooks info session-memory
```

**Output:**

```
💾 session-memory ✓ Ready

Salva il contesto della sessione nella memoria quando viene emesso il comando /new

Details:
  Source: openclaw-bundled
  Path: /path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  Handler: /path/to/openclaw/hooks/bundled/session-memory/handler.ts
  Homepage: https://docs.openclaw.ai/hooks#session-memory
  Events: command:new

Requirements:
  Config: ✓ workspace.dir
```

<div id="check-hooks-eligibility">
  ## Controllare l&#39;idoneità degli hook
</div>

```bash
openclaw hooks check
```

Mostra un riepilogo dello stato di idoneità dei hook (quanti sono pronti e quanti no).

**Opzioni:**

* `--json`: Stampa in formato JSON

**Esempio di output:**

```
Hooks Status

Total hooks: 4
Ready: 4
Not ready: 0
```

<div id="enable-a-hook">
  ## Abilitare un hook
</div>

```bash
openclaw hooks enable <name>
```

Abilita un hook specifico aggiungendolo alla tua configurazione (`~/.openclaw/config.json`).

**Nota:** Gli hook gestiti dai plugin riportano `plugin:<id>` in `openclaw hooks list` e
non possono essere abilitati/disabilitati qui. Abilita/disabilita invece il plugin.

**Argomenti:**

* `<name>`: Nome dell’hook (ad es. `session-memory`)

**Esempio:**

```bash
openclaw hooks enable session-memory
```

**Output:**

```
✓ Hook abilitato: 💾 session-memory
```

**Cosa fa:**

* Verifica se l&#39;hook esiste ed è idoneo
* Aggiorna `hooks.internal.entries.<name>.enabled = true` nella tua configurazione
* Salva la configurazione su disco

**Dopo averlo abilitato:**

* Riavvia il Gateway per ricaricare gli hook (riavvia l&#39;app della barra dei menu su macOS oppure riavvia il processo del tuo Gateway in sviluppo).

<div id="disable-a-hook">
  ## Disabilitare un hook
</div>

```bash
openclaw hooks disable <name>
```

Disabilita uno specifico hook aggiornando la configurazione.

**Argomenti:**

* `<name>`: Nome dell&#39;hook (ad es. `command-logger`)

**Esempio:**

```bash
openclaw hooks disable command-logger
```

**Output:**

```
⏸ Disabled hook: 📝 command-logger
```

**Dopo averli disattivati:**

* Riavvia il Gateway per ricaricare gli hook

<div id="install-hooks">
  ## Installazione degli hook
</div>

```bash
openclaw hooks install <path-or-spec>
```

Installa un pacchetto di hook da una directory/archivio locale o da npm.

**Cosa fa:**

* Copia il pacchetto di hook in `~/.openclaw/hooks/<id>`
* Abilita gli hook installati in `hooks.internal.entries.*`
* Registra l&#39;installazione in `hooks.internal.installs`

**Opzioni:**

* `-l, --link`: Collega una directory locale invece di copiarla (la aggiunge a `hooks.internal.load.extraDirs`)

**Archivi supportati:** `.zip`, `.tgz`, `.tar.gz`, `.tar`

**Esempi:**

```bash
# Local directory
openclaw hooks install ./my-hook-pack

# Local archive
openclaw hooks install ./my-hook-pack.zip

# NPM package
openclaw hooks install @openclaw/my-hook-pack

# Link a local directory without copying
openclaw hooks install -l ./my-hook-pack
```

<div id="update-hooks">
  ## Hook di aggiornamento
</div>

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

Aggiorna i pacchetti di hook installati (solo per installazioni tramite npm).

**Opzioni:**

* `--all`: aggiorna tutti i pacchetti di hook tracciati
* `--dry-run`: mostra le modifiche che verrebbero applicate senza scrivere nulla

<div id="bundled-hooks">
  ## Hook predefiniti
</div>

<div id="session-memory">
  ### session-memory
</div>

Salva il contesto della sessione in memoria quando esegui il comando `/new`.

**Attiva:**

```bash
openclaw hooks enable session-memory
```

**Output:** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md`

**Vedi:** [documentazione di session-memory](/it/hooks#session-memory)

<div id="command-logger">
  ### command-logger
</div>

Registra tutti gli eventi di comando in un file di audit centralizzato.

**Abilitazione:**

```bash
openclaw hooks enable command-logger
```

**Output:** `~/.openclaw/logs/commands.log`

**Visualizza i log:**

```bash
# Comandi recenti
tail -n 20 ~/.openclaw/logs/commands.log

# Stampa formattata
cat ~/.openclaw/logs/commands.log | jq .

# Filtra per azione
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**Vedi anche:** [documentazione di command-logger](/it/hooks#command-logger)

<div id="soul-evil">
  ### soul-evil
</div>

Sostituisce il contenuto iniettato di `SOUL.md` con `SOUL_EVIL.md` durante una finestra di purge o in modo casuale.

**Attiva:**

```bash
openclaw hooks enable soul-evil
```

**Vedi anche:** [SOUL Evil Hook](/it/hooks/soul-evil)

<div id="boot-md">
  ### boot-md
</div>

Esegue `BOOT.md` quando il Gateway si avvia (dopo l&#39;avvio dei canali).

**Eventi**: `gateway:startup`

**Abilita**:

```bash
openclaw hooks enable boot-md
```

**Consulta:** [documentazione di boot-md](/it/hooks#boot-md)
