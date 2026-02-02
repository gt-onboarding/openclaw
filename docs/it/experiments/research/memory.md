---
title: Memoria
summary: "Note di ricerca: sistema di memoria offline per gli spazi di lavoro Clawd (Markdown come sorgente di verità + indice derivato)"
read_when:
  - Progettare la memoria dello spazio di lavoro (~/.openclaw/workspace) oltre i log Markdown quotidiani
  - Decidere: CLI autonoma vs integrazione profonda con OpenClaw
  - Aggiungere richiamo offline + riflessione (retain/recall/reflect)
---

<div id="workspace-memory-v2-offline-research-notes">
  # Memoria dello spazio di lavoro v2 (offline): note di ricerca
</div>

Obiettivo: uno spazio di lavoro di tipo Clawd (`agents.defaults.workspace`, predefinito `~/.openclaw/workspace`) in cui la “memoria” è archiviata come un file Markdown al giorno (`memory/YYYY-MM-DD.md`), più un piccolo insieme di file stabili (ad esempio `memory.md`, `SOUL.md`).

Questo documento propone un'architettura di memoria **offline-first** che mantiene il Markdown come fonte di verità canonica e revisionabile, ma aggiunge un **richiamo strutturato** (ricerca, riepiloghi di entità, aggiornamenti del grado di confidenza) tramite un indice derivato.

<div id="why-change">
  ## Perché cambiare?
</div>

L’attuale impostazione (un file al giorno) è eccellente per:

- journaling “append-only”
- modifica manuale
- durabilità + tracciabilità con git
- cattura a basso attrito (“basta scriverlo”)

È debole per:

- recupero con alta capacità di richiamo (“cosa abbiamo deciso su X?”, “quando abbiamo provato Y l’ultima volta?”)
- risposte incentrate sulle entità (“parlami di Alice / The Castle / warelay”) senza dover rileggere molti file
- stabilità di opinioni/preferenze (e le evidenze quando cambiano)
- vincoli temporali (“cosa era vero nel novembre 2025?”) e risoluzione dei conflitti

<div id="design-goals">
  ## Obiettivi di progettazione
</div>

- **Offline**: funziona senza connessione di rete; può girare su laptop/Castle; nessuna dipendenza dal cloud.
- **Spiegabile**: gli elementi recuperati devono essere attribuibili (file + posizione) e separabili dall'inferenza.
- **A bassa cerimonia**: il logging quotidiano rimane in Markdown, senza pesante lavoro di modellazione dello schema.
- **Incrementale**: la v1 è utile anche solo con FTS (ricerca full-text); semantica/vettori e grafi sono upgrade opzionali.
- **Adatto agli agenti**: rende semplice il “richiamo entro i budget di token” (restituisce piccoli pacchetti di fatti).

<div id="north-star-model-hindsight-letta">
  ## Modello guida (North star) (Hindsight × Letta)
</div>

Due componenti da combinare:

1) **Ciclo di controllo in stile Letta/MemGPT**

- mantieni un piccolo “core” sempre nel contesto (persona + fatti chiave sull’utente)
- tutto il resto è fuori contesto e viene recuperato tramite strumenti
- le scritture in memoria sono chiamate esplicite a strumenti (append/replace/insert), rese persistenti e poi re-iniettate al turno successivo

2) **Substrato di memoria in stile Hindsight**

- separa ciò che è osservato da ciò che si ritiene vero e da ciò che è riassunto
- supporta retain/recall/reflect
- opinioni con un livello di confidenza che può evolvere con le evidenze
- recupero sensibile alle entità + query temporali (anche senza grafi di conoscenza completi)

<div id="proposed-architecture-markdown-source-of-truth-derived-index">
  ## Architettura proposta (Markdown come fonte di verità + indice derivato)
</div>

<div id="canonical-store-git-friendly">
  ### Archivio canonico (adatto a Git)
</div>

Mantieni `~/.openclaw/workspace` come memoria canonica in formato leggibile dalle persone.

Layout suggerito per lo spazio di lavoro:

```
~/.openclaw/workspace/
  memory.md                    # piccolo: fatti durevoli + preferenze (essenziale)
  memory/
    YYYY-MM-DD.md              # registro giornaliero (append; narrativo)
  bank/                        # pagine di memoria "tipizzate" (stabili, revisionabili)
    world.md                   # fatti oggettivi sul mondo
    experience.md              # cosa ha fatto l'agente (prima persona)
    opinions.md                # preferenze/giudizi soggettivi + confidenza + puntatori alle evidenze
    entities/
      Peter.md
      The-Castle.md
      warelay.md
      ...
```

Note:

* **Il registro giornaliero resta il registro giornaliero**. Non c’è bisogno di trasformarlo in JSON.
* I file `bank/` sono **curati**, prodotti da job di reflection e possono comunque essere modificati manualmente.
* `memory.md` rimane “piccolo e ‘core’”: le cose che vuoi che Clawd veda in ogni sessione.


<div id="derived-store-machine-recall">
  ### Archivio derivato (richiamo automatico)
</div>

Aggiungi un indice derivato all&#39;interno dello spazio di lavoro (non necessariamente tracciato in Git):

```
~/.openclaw/workspace/.memory/index.sqlite
```

Supportalo con:

* uno schema SQLite per i fatti + i collegamenti tra entità + i metadati delle opinioni
* SQLite **FTS5** per il richiamo lessicale (veloce, leggero, offline)
* tabella opzionale di embeddings per il richiamo semantico (comunque offline)

L&#39;indice è sempre **ricostruibile a partire dal Markdown**.


<div id="retain-recall-reflect-operational-loop">
  ## Memorizzare / Richiamare / Riflettere (ciclo operativo)
</div>

<div id="retain-normalize-daily-logs-into-facts">
  ### Retain: normalizza i log giornalieri in “fatti”
</div>

L’intuizione chiave di Hindsight che conta qui: memorizza **fatti narrativi e autosufficienti**, non micro-frammenti.

Regola pratica per `memory/YYYY-MM-DD.md`:

* a fine giornata (o durante), aggiungi una sezione `## Retain` con 2–5 punti elenco che siano:
  * narrativi (contesto tra i turni di dialogo preservato)
  * autosufficienti (hanno senso da soli anche in seguito)
  * taggati con tipo + menzioni di entità

Esempio:

```
## Mantieni
- W @Peter: Attualmente a Marrakech (27 nov–1 dic 2025) per il compleanno di Andy.
- B @warelay: Ho risolto il crash WS di Baileys avvolgendo i gestori connection.update in try/catch (vedi memory/2025-11-27.md).
- O(c=0.95) @Peter: Preferisce risposte concise (&lt;1500 caratteri) su WhatsApp; i contenuti lunghi vanno nei file.
```

Parsing minimo:

* Prefisso di tipo: `W` (world/mondo), `B` (experience/biographical/esperienziale/biografico), `O` (opinion/opinione), `S` (observation/summary/osservazione/sintesi; di solito generata)
* Entità: `@Peter`, `@warelay`, ecc. (gli slug mappano a `bank/entities/*.md`)
* Attendibilità dell’opinione: `O(c=0.0..1.0)` opzionale

Se non vuoi che gli autori ci pensino: il job reflect può dedurre questi elementi dal resto del log, ma avere una sezione esplicita `## Retain` è la “leva di qualità” più semplice da usare.


<div id="recall-queries-over-the-derived-index">
  ### Recall: query sull'indice derivato
</div>

Recall deve supportare:

- **lessicale**: "trova termini / nomi / comandi esatti" (FTS5)
- **entità**: "parlami di X" (pagine delle entità + fatti collegati alle entità)
- **temporale**: "che cosa è successo intorno al 27 nov" / "dalla scorsa settimana"
- **opinione**: "che cosa preferisce Peter?" (con livello di confidenza + evidenze)

Il formato di output deve essere adatto all'agente e includere le fonti:

- `kind` (`world|experience|opinion|observation`)
- `timestamp` (giorno di origine, oppure intervallo temporale estratto se presente)
- `entities` (`["Peter","warelay"]`)
- `content` (il fatto in forma narrativa)
- `source` (`memory/2025-11-27.md#L12` ecc)

<div id="reflect-produce-stable-pages-update-beliefs">
  ### Reflect: produce stable pages + update beliefs
</div>

La riflessione è un job pianificato (giornaliero o heartbeat `ultrathink`) che:

- aggiorna `bank/entities/*.md` a partire dai fatti recenti (riassunti di entità)
- aggiorna il grado di confidenza in `bank/opinions.md` in base a rinforzi/contraddizioni
- opzionalmente propone modifiche a `memory.md` (fatti durevoli quasi “core”)

Evoluzione delle opinioni (semplice, spiegabile):

- ogni opinione ha:
  - statement
  - confidence `c ∈ [0,1]`
  - last_updated
  - evidence links (ID di fatti a supporto + in contraddizione)
- quando arrivano nuovi fatti:
  - trova le opinioni candidate per sovrapposizione tra entità + similarità (prima FTS, poi embeddings)
  - aggiorna la confidenza con piccoli delta; grandi salti richiedono una forte contraddizione + prove ripetute

<div id="cli-integration-standalone-vs-deep-integration">
  ## Integrazione della CLI: standalone vs integrazione profonda
</div>

Raccomandazione: **integrazione profonda in OpenClaw**, ma mantieni una libreria core separabile.

<div id="why-integrate-into-openclaw">
  ### Perché integrare con OpenClaw?
</div>

- OpenClaw conosce già:
  - il percorso dello spazio di lavoro (`agents.defaults.workspace`)
  - il modello di sessione + gli heartbeat
  - le convenzioni di logging e troubleshooting
- Vuoi che sia l'agente stesso a invocare gli strumenti:
  - `openclaw memory recall "…" --k 25 --since 30d`
  - `openclaw memory reflect --since 7d`

<div id="why-still-split-a-library">
  ### Perché suddividere comunque una libreria?
</div>

- mantenere la logica della memoria testabile senza Gateway/runtime
- riutilizzarla in altri contesti (script locali, futura app desktop, ecc.)

Struttura:
La strumentazione per la memoria è pensata come un piccolo livello costituito da CLI + libreria, ma al momento questo è solo esplorativo.

<div id="s-collide-suco-when-to-use-it-research">
  ## “S-Collide” / SuCo: quando usarlo (ricerca)
</div>

Se “S-Collide” si riferisce a **SuCo (Subspace Collision)**: è un approccio di retrieval ANN che punta a un forte compromesso tra richiamo e latenza usando collisioni apprese/strutturate in sottospazi (paper: arXiv 2411.14754, 2024).

Indicazioni pratiche per `~/.openclaw/workspace`:

- **non partire** da SuCo.
- inizia con SQLite FTS + (opzionali) embedding semplici; otterrai subito la maggior parte dei miglioramenti di UX.
- prendi in considerazione soluzioni tipo SuCo/HNSW/ScaNN solo quando:
  - il corpus è grande (decine/centinaia di migliaia di chunk)
  - la ricerca a forza bruta sugli embedding diventa troppo lenta
  - la qualità del richiamo è significativamente limitata dalla ricerca lessicale

Alternative adatte all’uso offline (in ordine di complessità crescente):

- SQLite FTS5 + filtri di metadati (zero ML)
- Embedding + forza bruta (funziona sorprendentemente bene se il numero di chunk è basso)
- Indice HNSW (comune, robusto; richiede un binding di libreria)
- SuCo (di livello research; interessante se esiste un’implementazione solida che puoi incorporare)

Questione aperta:

- qual è il **miglior** modello di embedding offline per la “memoria dell’assistente personale” sulle tue macchine (laptop + desktop)?
  - se hai già Ollama: genera embedding con un modello locale; altrimenti includi un piccolo modello di embedding nella toolchain.

<div id="smallest-useful-pilot">
  ## Il più piccolo progetto pilota utile
</div>

Se vuoi una versione minima ma comunque utile:

- Aggiungi le pagine di entità `bank/` e una sezione `## Retain` nei log giornalieri.
- Usa SQLite FTS per il richiamo con citazioni (percorso + numeri di riga).
- Aggiungi gli embeddings solo se la qualità del richiamo o la scala lo richiedono.

<div id="references">
  ## Riferimenti
</div>

- Concetti di Letta / MemGPT: “core memory blocks” + “archival memory” + memoria auto-modificante guidata da tool.
- Hindsight Technical Report: “retain / recall / reflect”, memoria a quattro reti, estrazione narrativa dei fatti, evoluzione del grado di confidenza nelle opinioni.
- SuCo: arXiv 2411.14754 (2024): “Subspace Collision” per la ricerca approssimata dei vicini più prossimi.