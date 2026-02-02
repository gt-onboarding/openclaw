---
title: Compattazione
summary: "Finestra di contesto e compattazione: come OpenClaw mantiene le sessioni entro i limiti del modello"
read_when:
  - Vuoi capire la compattazione automatica e /compact
  - Stai eseguendo il debug di sessioni lunghe che raggiungono i limiti di contesto
---

<div id="context-window-compaction">
  # Finestra di contesto e compattazione
</div>

Ogni modello ha una **finestra di contesto** (numero massimo di token che può elaborare). Le chat di lunga durata accumulano messaggi e risultati degli strumenti; quando la finestra si avvicina al limite, OpenClaw **compatta** la cronologia più vecchia per restare entro i limiti.

<div id="what-compaction-is">
  ## Che cos’è la compattazione
</div>

La compattazione **riassume le conversazioni più vecchie** in una voce di riepilogo compatta e mantiene intatti i messaggi recenti. Il riepilogo viene memorizzato nella cronologia della sessione, quindi le richieste future usano:

* Il riepilogo di compattazione
* I messaggi recenti successivi al punto di compattazione

La compattazione **persiste** nella cronologia JSONL della sessione.

<div id="configuration">
  ## Configurazione
</div>

Consulta [Configurazione e modalità di compattazione](/it/concepts/compaction) per le impostazioni di `agents.defaults.compaction`.

<div id="auto-compaction-default-on">
  ## Compattazione automatica (attiva per impostazione predefinita)
</div>

Quando una sessione si avvicina o supera la finestra di contesto del modello, OpenClaw attiva la compattazione automatica e può riprovare la richiesta originale utilizzando il contesto compattato.

Potrai vedere:

* `🧹 Auto-compaction complete` in modalità dettagliata
* `/status` che mostra `🧹 Compactions: <count>`

Prima della compattazione, OpenClaw può eseguire un passaggio di **silent memory flush** per archiviare
note persistenti su disco. Per dettagli e configurazione, vedi [Memory](/it/concepts/memory).

<div id="manual-compaction">
  ## Compattazione manuale
</div>

Utilizza `/compact` (eventualmente con istruzioni aggiuntive) per forzare un ciclo di compattazione:

```
/compact Focus on decisions and open questions
```

<div id="context-window-source">
  ## Origine della finestra di contesto
</div>

La finestra di contesto è specifica del modello. OpenClaw utilizza la definizione del modello dal catalogo del provider configurato per determinarne i limiti.

<div id="compaction-vs-pruning">
  ## Compattazione vs pruning
</div>

* **Compattazione**: riassume e **salva in modo persistente** in JSONL.
* **Pruning della sessione**: rimuove solo i **risultati degli strumenti** più vecchi, **in memoria**, a ogni richiesta.

Vedi [/concepts/session-pruning](/it/concepts/session-pruning) per i dettagli sul pruning.

<div id="tips">
  ## Suggerimenti
</div>

* Usa `/compact` quando le sessioni sembrano “stanche” o il contesto è eccessivamente gonfio.
* I risultati molto estesi degli strumenti vengono già troncati; una potatura ulteriore può ridurre ancora di più l&#39;accumulo dei risultati degli strumenti.
* Se ti serve una “lavagna pulita”, `/new` o `/reset` avvia una nuova sessione con un nuovo id.