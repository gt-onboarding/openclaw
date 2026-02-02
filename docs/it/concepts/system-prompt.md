---
title: Prompt di sistema
summary: "Cosa contiene il prompt di sistema di OpenClaw e come viene assemblato"
read_when:
  - Quando modifichi il testo del prompt di sistema, l'elenco degli strumenti o le sezioni relative a tempo/heartbeat
  - Quando modifichi il bootstrap dello spazio di lavoro o il comportamento di iniezione delle abilità
---

<div id="system-prompt">
  # Prompt di sistema
</div>

OpenClaw genera un prompt di sistema personalizzato per ogni esecuzione di un agente. Il prompt è **di proprietà di OpenClaw** e non utilizza il prompt predefinito di p-coding-agent.

Il prompt è assemblato da OpenClaw e iniettato in ogni esecuzione dell&#39;agente.

<div id="structure">
  ## Struttura
</div>

Il prompt è deliberatamente compatto e utilizza sezioni fisse:

* **Tooling**: elenco attuale degli strumenti + brevi descrizioni.
* **Skills** (quando disponibili): indica al modello come caricare le istruzioni delle abilità su richiesta.
* **OpenClaw Self-Update**: come eseguire `config.apply` e `update.run`.
* **Workspace**: spazio di lavoro/directory di lavoro (`agents.defaults.workspace`).
* **Documentation**: percorso locale alla documentazione di OpenClaw (repo o pacchetto npm) e quando leggerla.
* **Workspace Files (injected)**: indica che i file di bootstrap sono inclusi di seguito.
* **Sandbox** (quando abilitata): indica il runtime in sandbox, i percorsi della sandbox e se è disponibile l&#39;esecuzione con privilegi elevati.
* **Current Date &amp; Time**: orario locale dell’utente, fuso orario e formato dell’ora.
* **Reply Tags**: sintassi opzionale dei tag di risposta per i provider supportati.
* **Heartbeats**: prompt dell’heartbeat e comportamento di ack/acknowledgment.
* **Runtime**: host, OS, nodo, modello, radice della repo (quando rilevata), livello di ragionamento (una riga).
* **Reasoning**: livello di visibilità corrente + suggerimento per il toggle /reasoning.

<div id="prompt-modes">
  ## Modalità di prompt
</div>

OpenClaw può generare prompt di sistema più brevi per i sotto‑agenti. Il runtime imposta un
`promptMode` per ogni esecuzione (non è una configurazione esposta all&#39;utente):

* `full` (predefinito): include tutte le sezioni sopra.
* `minimal`: usato per i sotto‑agenti; omette **Skills**, **Memory Recall**, **OpenClaw
  Self-Update**, **Model Aliases**, **User Identity**, **Reply Tags**,
  **Messaging**, **Silent Replies** e **Heartbeats**. Tooling, Spazio di lavoro,
  Sandbox, Data e ora correnti (quando note), Runtime e contesto iniettato restano
  disponibili.
* `none`: restituisce solo la riga di identità di base.

Quando `promptMode=minimal`, i prompt aggiuntivi iniettati sono etichettati come **Subagent
Context** invece di **Group Chat Context**.

<div id="workspace-bootstrap-injection">
  ## Iniezione di bootstrap dello spazio di lavoro
</div>

I file di bootstrap vengono ridotti e aggiunti sotto **Project Context** in modo che il modello veda il contesto di identità e profilo senza aver bisogno di `read` espliciti:

* `AGENTS.md`
* `SOUL.md`
* `TOOLS.md`
* `IDENTITY.md`
* `USER.md`
* `HEARTBEAT.md`
* `BOOTSTRAP.md` (solo negli spazi di lavoro completamente nuovi)

I file di grandi dimensioni vengono troncati con un marcatore. La dimensione massima per file è controllata da
`agents.defaults.bootstrapMaxChars` (predefinito: 20000). Per i file mancanti viene iniettato un
breve marcatore di file mancante.

Gli hook interni possono intercettare questo passaggio tramite `agent:bootstrap` per modificare o sostituire
i file di bootstrap iniettati (ad esempio sostituendo `SOUL.md` con un’identità/persona alternativa).

Per ispezionare quanto contribuisce ciascun file iniettato (contenuto grezzo vs contenuto iniettato, troncamento, più overhead dello schema degli strumenti), usa `/context list` o `/context detail`. Vedi [Context](/it/concepts/context).

<div id="time-handling">
  ## Gestione di data e ora
</div>

Il prompt di sistema include una sezione dedicata **Current Date &amp; Time** quando il
fuso orario dell&#39;utente è noto. Per mantenere stabile la cache del prompt, ora include
solo il **fuso orario** (senza orologio dinamico né formato dell&#39;ora).

Usa `session_status` quando l&#39;agente ha bisogno dell&#39;ora corrente; la scheda di stato
include una riga con la marca temporale.

Configura con:

* `agents.defaults.userTimezone`
* `agents.defaults.timeFormat` (`auto` | `12` | `24`)

Consulta [Date &amp; Time](/it/date-time) per i dettagli completi sul comportamento.

<div id="skills">
  ## Abilità
</div>

Quando sono presenti abilità idonee, OpenClaw inserisce un elenco compatto di
**abilità disponibili** (`formatSkillsForPrompt`) che include il **percorso del
file** per ciascuna abilità. Il prompt istruisce il modello a usare `read` per
caricare il file SKILL.md nella posizione indicata (spazio di lavoro, gestito o
in bundle). Se non ci sono abilità idonee, la sezione Abilità viene omessa.

```
<available_skills>
  <skill>
    <name>...</name>
    <description>...</description>
    <location>...</location>
  </skill>
</available_skills>
```

Questo mantiene il prompt di base compatto pur consentendo l&#39;uso mirato delle skill.

<div id="documentation">
  ## Documentazione
</div>

Quando è disponibile, il prompt di sistema include una sezione **Documentazione** che fa riferimento alla
directory locale della documentazione di OpenClaw (sia `docs/` nello spazio di lavoro del repository sia la documentazione del pacchetto npm
inclusa) e segnala anche il mirror pubblico, il repository sorgente, il server Discord della community e
ClawHub (https://clawhub.com) per l’individuazione delle abilità. Il prompt istruisce il modello a consultare prima la documentazione locale
per il comportamento, i comandi, la configurazione o l’architettura di OpenClaw e a eseguire
`openclaw status` direttamente quando possibile (chiedendo all’utente solo quando non ha accesso).