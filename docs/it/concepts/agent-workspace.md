---
title: Spazio di lavoro dell'Agente
summary: "Spazio di lavoro dell'agente: posizione, struttura e strategia di backup"
read_when:
  - Devi spiegare lo spazio di lavoro di un agente o la sua struttura dei file
  - Vuoi eseguire il backup o migrare lo spazio di lavoro di un agente
---

<div id="agent-workspace">
  # Spazio di lavoro dell&#39;Agente
</div>

Lo spazio di lavoro è la &quot;casa&quot; dell&#39;agente. È l&#39;unica directory di lavoro usata
per gli strumenti per i file e per il contesto dello spazio di lavoro. Mantienilo
privato e trattalo come memoria.

Questo è distinto da `~/.openclaw/`, che memorizza configurazioni, credenziali e
sessioni.

**Importante:** lo spazio di lavoro è la **directory di lavoro corrente (cwd) predefinita**,
non una sandbox con isolamento rigido. Gli strumenti interpretano i percorsi relativi
rispetto allo spazio di lavoro, ma i percorsi assoluti possono comunque raggiungere
altre parti dell&#39;host, a meno che la sandbox non sia abilitata. Se hai bisogno di
isolamento, usa [`agents.defaults.sandbox`](/it/gateway/sandboxing) (e/o una configurazione
di sandbox per singolo agente). Quando la sandbox è abilitata e `workspaceAccess`
non è `"rw"`, gli strumenti operano all&#39;interno di uno spazio di lavoro della sandbox
in `~/.openclaw/sandboxes`, non nel tuo spazio di lavoro sull&#39;host.

<div id="default-location">
  ## Posizione predefinita
</div>

* Valore predefinito: `~/.openclaw/workspace`
* Se `OPENCLAW_PROFILE` è impostata e non è `"default"`, il valore predefinito diventa
  `~/.openclaw/workspace-<profile>`.
* Puoi sovrascrivere questa impostazione in `~/.openclaw/openclaw.json`:

```json5
{
  agent: {
    workspace: "~/.openclaw/workspace"
  }
}
```

`openclaw onboard`, `openclaw configure` o `openclaw setup` creeranno lo
spazio di lavoro e, se mancanti, genereranno i file di bootstrap.

Se gestisci già direttamente i file dello spazio di lavoro, puoi disattivare la
creazione dei file di bootstrap:

```json5
{ agent: { skipBootstrap: true } }
```

<div id="extra-workspace-folders">
  ## Cartelle aggiuntive dello spazio di lavoro
</div>

Le installazioni più vecchie potrebbero aver creato `~/openclaw`. Mantenere più
directory dello spazio di lavoro può causare comportamenti confusi relativi
all&#39;autenticazione o allo stato, perché è attivo solo uno spazio di lavoro alla volta.

**Raccomandazione:** mantieni un unico spazio di lavoro attivo. Se non usi più
le cartelle aggiuntive, archiviale o spostale nel Cestino (per esempio `trash ~/openclaw`).
Se mantieni intenzionalmente più spazi di lavoro, assicurati che
`agents.defaults.workspace` punti a quello attivo.

`openclaw doctor` mostra un avviso quando rileva directory di spazio di lavoro aggiuntive.

<div id="workspace-file-map-what-each-file-means">
  ## Mappa dei file dello spazio di lavoro (significato di ciascun file)
</div>

Questi sono i file standard che OpenClaw si aspetta di trovare all&#39;interno dello spazio di lavoro:

* `AGENTS.md`
  * Istruzioni operative per l&#39;agente e su come dovrebbe usare la memoria.
  * Caricato all&#39;inizio di ogni sessione.
  * Buon posto per regole, priorità e dettagli su &quot;come comportarsi&quot;.

* `SOUL.md`
  * Persona, tono e limiti.
  * Caricato a ogni sessione.

* `USER.md`
  * Chi è l&#39;utente e come rivolgersi a lui/lei.
  * Caricato a ogni sessione.

* `IDENTITY.md`
  * Nome, stile e emoji dell&#39;agente.
  * Creato/aggiornato durante il rituale di bootstrap.

* `TOOLS.md`
  * Note sui tuoi strumenti locali e sulle convenzioni.
  * Non controlla la disponibilità degli strumenti; è solo una guida.

* `HEARTBEAT.md`
  * Piccola checklist opzionale per le esecuzioni di heartbeat.
  * Tienilo breve per evitare un consumo eccessivo di token.

* `BOOT.md`
  * Checklist di avvio opzionale eseguita al riavvio del Gateway quando gli hook interni sono abilitati.
  * Tienilo breve; usa lo strumento dei messaggi per gli invii in uscita.

* `BOOTSTRAP.md`
  * Rituale di prima esecuzione una tantum.
  * Creato solo per uno spazio di lavoro completamente nuovo.
  * Eliminalo dopo che il rituale è stato completato.

* `memory/YYYY-MM-DD.md`
  * Log di memoria giornaliero (un file per giorno).
  * Si consiglia di leggere oggi + ieri all&#39;avvio della sessione.

* `MEMORY.md` (opzionale)
  * Memoria a lungo termine curata.
  * Caricalo solo nella sessione principale privata (non in contesti condivisi/di gruppo).

Vedi [Memory](/it/concepts/memory) per il workflow e il flush automatico della memoria.

* `skills/` (opzionale)
  * Abilità specifiche dello spazio di lavoro.
  * Sovrascrive le abilità gestite/in bundle quando i nomi coincidono.

* `canvas/` (opzionale)
  * File UI canvas per i display dei nodi (per esempio `canvas/index.html`).

Se manca qualche file di bootstrap, OpenClaw inserisce un marcatore di &quot;file mancante&quot; nella
sessione e continua. I file di bootstrap di grandi dimensioni vengono troncati quando vengono inseriti;
regola il limite con `agents.defaults.bootstrapMaxChars` (predefinito: 20000).
`openclaw setup` può ricreare i valori predefiniti mancanti senza sovrascrivere i file
esistenti.

<div id="what-is-not-in-the-workspace">
  ## Cosa NON fa parte dello spazio di lavoro
</div>

Questi elementi si trovano in `~/.openclaw/` e NON devono essere inclusi nel repo dello spazio di lavoro:

* `~/.openclaw/openclaw.json` (configurazione)
* `~/.openclaw/credentials/` (token OAuth, chiavi API)
* `~/.openclaw/agents/<agentId>/sessions/` (trascrizioni delle sessioni + metadati)
* `~/.openclaw/skills/` (abilità gestite)

Se devi migrare sessioni o configurazioni, copiale separatamente e mantienile
fuori dal controllo di versione.

<div id="git-backup-recommended-private">
  ## Backup Git (consigliato, privato)
</div>

Considera lo spazio di lavoro come memoria privata. Inseriscilo in una repository Git **privata** in modo che sia
sottoposto a backup e possa essere ripristinato.

Esegui questi passaggi sulla macchina su cui è in esecuzione il Gateway (ovvero dove
si trova lo spazio di lavoro).

<div id="1-initialize-the-repo">
  ### 1) Inizializza il repo
</div>

Se git è installato, i nuovi spazi di lavoro vengono inizializzati automaticamente. Se questo
spazio di lavoro non è già un repo, esegui:

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md SOUL.md TOOLS.md IDENTITY.md USER.md HEARTBEAT.md memory/
git commit -m "Aggiungi spazio di lavoro dell'agente"
```

<div id="2-add-a-private-remote-beginner-friendly-options">
  ### 2) Aggiungi un remote privato (opzioni per principianti)
</div>

Opzione A: GitHub Web UI

1. Crea un nuovo repository **privato** su GitHub.
2. Non inizializzarlo con un README (per evitare conflitti di merge).
3. Copia l&#39;URL HTTPS del remote.
4. Aggiungi il remote e fai il push:

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

Opzione B: GitHub CLI (`gh`)

```bash
gh auth login
gh repo create openclaw-workspace --private --source . --remote origin --push
```

Opzione C: UI web di GitLab

1. Crea un nuovo repository **privato** su GitLab.
2. Non inizializzare il repository con un README (evita conflitti di merge).
3. Copia l&#39;URL HTTPS del remote.
4. Aggiungi il remote e fai il push:

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

<div id="3-ongoing-updates">
  ### 3) Aggiornamenti continui
</div>

```bash
git status
git add .
git commit -m "Aggiorna memoria"
git push
```

<div id="do-not-commit-secrets">
  ## Non eseguire commit di segreti
</div>

Anche in un repository privato, evita di archiviare segreti nello spazio di lavoro:

* Chiavi API, token OAuth, password o credenziali private.
* Qualsiasi cosa sotto `~/.openclaw/`.
* Dump grezzi di chat o allegati sensibili.

Se devi memorizzare riferimenti sensibili, usa dei segnaposto e conserva il vero
segreto altrove (password manager, variabili d&#39;ambiente o `~/.openclaw/`).

`.gitignore` iniziale consigliato:

```gitignore
.DS_Store
.env
**/*.key
**/*.pem
**/secrets*
```

<div id="moving-the-workspace-to-a-new-machine">
  ## Spostare lo spazio di lavoro su una nuova macchina
</div>

1. Clona il repo nel percorso desiderato (predefinito `~/.openclaw/workspace`).
2. Imposta `agents.defaults.workspace` su quel percorso in `~/.openclaw/openclaw.json`.
3. Esegui `openclaw setup --workspace <path>` per inizializzare eventuali file mancanti.
4. Se hai bisogno delle sessioni, copia `~/.openclaw/agents/<agentId>/sessions/` dalla
   vecchia macchina, separatamente.

<div id="advanced-notes">
  ## Note avanzate
</div>

* L&#39;instradamento multi-agente può usare spazi di lavoro diversi per ogni agente. Consulta
  [Instradamento dei canali](/it/concepts/channel-routing) per la configurazione dell&#39;instradamento.
* Se `agents.defaults.sandbox` è abilitato, le sessioni non principali possono usare spazi di lavoro della sandbox per singola sessione sotto `agents.defaults.sandbox.workspaceRoot`.