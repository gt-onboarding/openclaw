---
title: Contesto
summary: "Contesto: cosa vede il modello, come viene costruito e come ispezionarlo"
read_when:
  - Vuoi capire cosa significa “contesto” in OpenClaw
  - Stai effettuando il debug per capire perché il modello “sa” qualcosa (o lo ha dimenticato)
  - Vuoi ridurre l’overhead del contesto (/context, /status, /compact)
---

<div id="context">
  # Contesto
</div>

Il “contesto” è **tutto ciò che OpenClaw invia al modello per una run**. È limitato dalla **finestra di contesto** (limite di token) del modello.

Modello mentale di base:

* **System prompt** (costruito da OpenClaw): regole, strumenti, elenco di abilità, ora/tempo di esecuzione e file dello spazio di lavoro iniettati.
* **Cronologia della conversazione**: i tuoi messaggi + i messaggi dell’assistente per questa sessione.
* **Chiamate/risultati degli strumenti + allegati**: output dei comandi, lettura dei file, immagini/audio, ecc.

Il contesto *non è la stessa cosa* della “memoria”: la memoria può essere salvata su disco e ricaricata in seguito; il contesto è ciò che si trova all’interno della finestra di contesto corrente del modello.

<div id="quick-start-inspect-context">
  ## Guida rapida (ispeziona il contesto)
</div>

* `/status` → vista rapida “quanto è piena la mia finestra?” + impostazioni di sessione.
* `/context list` → cosa viene iniettato + dimensioni approssimative (per file + totali).
* `/context detail` → analisi più approfondita: dimensioni per file, per schema di tool, per voce di skill e dimensione del system prompt.
* `/usage tokens` → aggiunge un footer con l’utilizzo di token per risposta alle risposte normali.
* `/compact` → riassume lo storico più vecchio in una voce compatta per liberare spazio nella finestra.

Vedi anche: [Slash commands](/it/tools/slash-commands), [Uso dei token e costi](/it/token-use), [Compattazione](/it/concepts/compaction).

<div id="example-output">
  ## Esempio di output
</div>

I valori variano in base al modello, al provider, alla policy degli strumenti e al contenuto del tuo spazio di lavoro.

<div id="context-list">
  ### `/context list`
</div>

```
🧠 Dettaglio del contesto
Spazio di lavoro: <workspaceDir>
Bootstrap max/file: 20.000 caratteri
Sandbox: mode=non-main sandboxed=false
Prompt di sistema (esecuzione): 38.412 caratteri (~9.603 tok) (Contesto progetto 23.901 caratteri (~5.976 tok))

File dello spazio di lavoro iniettati:
- AGENTS.md: OK | grezzo 1,742 caratteri (~436 tok) | iniettato 1,742 caratteri (~436 tok)
- SOUL.md: OK | grezzo 912 caratteri (~228 tok) | iniettato 912 caratteri (~228 tok)
- TOOLS.md: TRONCATO | grezzo 54,210 caratteri (~13,553 tok) | iniettato 20,962 caratteri (~5,241 tok)
- IDENTITY.md: OK | grezzo 211 caratteri (~53 tok) | iniettato 211 caratteri (~53 tok)
- USER.md: OK | grezzo 388 caratteri (~97 tok) | iniettato 388 caratteri (~97 tok)
- HEARTBEAT.md: MANCANTE | grezzo 0 | iniettato 0
- BOOTSTRAP.md: OK | grezzo 0 caratteri (~0 tok) | iniettato 0 caratteri (~0 tok)

Elenco abilità (testo prompt di sistema): 2.184 caratteri (~546 tok) (12 abilità)
Tools: read, edit, write, exec, process, browser, message, sessions_send, …
Elenco strumenti (testo prompt di sistema): 1.032 caratteri (~258 tok)
Schemi strumenti (JSON): 31.988 caratteri (~7.997 tok) (conteggiati nel contesto; non mostrati come testo)
Strumenti: (come sopra)

Token di sessione (in cache): 14.250 totali / ctx=32.000
```

<div id="context-detail">
  ### `/context detail`
</div>

```
🧠 Context breakdown (detailed)
…
Top skills (prompt entry size):
- frontend-design: 412 chars (~103 tok)
- oracle: 401 chars (~101 tok)
… (+10 more skills)

Top tools (schema size):
- browser: 9,812 chars (~2,453 tok)
- exec: 6,240 chars (~1,560 tok)
… (+N more tools)
```

<div id="what-counts-toward-the-context-window">
  ## Cosa viene conteggiato nella finestra di contesto
</div>

Tutto ciò che il modello riceve viene conteggiato, inclusi:

* System prompt (tutte le sezioni).
* Cronologia della conversazione.
* Chiamate agli strumenti e relativi risultati.
* Allegati/trascrizioni (immagini/audio/file).
* Riepiloghi di compattazione e artefatti di pruning.
* “Wrapper” del provider o intestazioni nascoste (non visibili, ma comunque conteggiati).

<div id="how-openclaw-builds-the-system-prompt">
  ## Come OpenClaw genera il system prompt
</div>

Il system prompt è **controllato da OpenClaw** e viene ricostruito a ogni esecuzione. Include:

* Elenco degli strumenti + brevi descrizioni.
* Elenco delle abilità (solo metadati; vedi sotto).
* Percorso dello spazio di lavoro.
* Ora (UTC + ora utente convertita se configurata).
* Metadati di runtime (host/OS/modello/thinking).
* File di bootstrap dello spazio di lavoro inseriti in **Project Context**.

Dettagli completi: [System Prompt](/it/concepts/system-prompt).

<div id="injected-workspace-files-project-context">
  ## File dello spazio di lavoro iniettati (Contesto del progetto)
</div>

Per impostazione predefinita, OpenClaw inietta un insieme fisso di file dello spazio di lavoro (se presenti):

* `AGENTS.md`
* `SOUL.md`
* `TOOLS.md`
* `IDENTITY.md`
* `USER.md`
* `HEARTBEAT.md`
* `BOOTSTRAP.md` (solo al primo avvio)

I file di grandi dimensioni vengono troncati, file per file, utilizzando `agents.defaults.bootstrapMaxChars` (predefinito `20000` caratteri). `/context` mostra le dimensioni **originali vs iniettate** e se è stato effettuato un troncamento.

<div id="skills-whats-injected-vs-loaded-on-demand">
  ## Abilità: cosa viene iniettato e cosa caricato on‑demand
</div>

Il prompt di sistema include un **elenco di abilità** compatto (nome + descrizione + posizione). Questo elenco comporta un overhead reale.

Le istruzioni dell’abilità *non* sono incluse per impostazione predefinita. Si prevede che il modello esegua il comando `read` sul file `SKILL.md` dell’abilità **solo quando necessario**.

<div id="tools-there-are-two-costs">
  ## Strumenti: ci sono due voci di costo
</div>

Gli strumenti influenzano il contesto in due modi:

1. **Testo dell’elenco degli strumenti** nel system prompt (quello che vedi come “Tooling”).
2. **Schemi degli strumenti** (JSON). Questi vengono inviati al modello in modo che possa chiamare gli strumenti. Contribuiscono al contesto anche se non li vedi come testo in chiaro.

`/context detail` suddivide gli schemi degli strumenti più grandi, così puoi vedere cosa incide di più.

<div id="commands-directives-and-inline-shortcuts">
  ## Comandi, direttive e “scorciatoie inline”
</div>

I comandi slash sono gestiti dal Gateway. Esistono alcuni comportamenti diversi:

* **Comandi standalone**: un messaggio che contiene solo `/...` viene eseguito come comando.
* **Direttive**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/model`, `/queue` vengono rimosse prima che il modello veda il messaggio.
  * I messaggi contenenti solo direttive rendono persistenti le impostazioni della sessione.
  * Le direttive inline in un messaggio normale agiscono come suggerimenti per il singolo messaggio.
* **Scorciatoie inline** (solo mittenti nella lista di autorizzati): alcuni token `/...` all&#39;interno di un messaggio normale possono essere eseguiti immediatamente (esempio: “hey /status”) e vengono rimossi prima che il modello veda il testo rimanente.

Dettagli: [Slash commands](/it/tools/slash-commands).

<div id="sessions-compaction-and-pruning-what-persists">
  ## Sessioni, compattazione e pruning (cosa persiste)
</div>

Quello che persiste tra i messaggi dipende dal meccanismo:

* La **cronologia normale** persiste nel registro della sessione finché non viene compattata/soggetta a pruning in base alle policy.
* La **compattazione** salva un riepilogo nel registro e mantiene intatti i messaggi recenti.
* Il **pruning** rimuove i vecchi risultati degli strumenti dal prompt *in-memory* per un&#39;esecuzione, ma non riscrive il registro.

Documentazione: [Sessione](/it/concepts/session), [Compattazione](/it/concepts/compaction), [Pruning della sessione](/it/concepts/session-pruning).

<div id="what-context-actually-reports">
  ## Cosa riporta effettivamente `/context`
</div>

`/context` preferisce l’ultimo report del system prompt **generato da una run** quando è disponibile:

* `System prompt (run)` = acquisito dall’ultima run incorporata (abilitata ai tool) e conservato nell’archivio delle sessioni.
* `System prompt (estimate)` = calcolato al volo quando non esiste alcun report di run (o quando si esegue tramite un backend CLI che non genera il report).

In entrambi i casi, riporta dimensioni e principali contributi; **non** effettua il dump completo del system prompt né degli schemi dei tool.