---
title: Barra dei menu
summary: "Logica dello stato della barra dei menu e cosa viene mostrato agli utenti"
read_when:
  - Modifica della UI della barra dei menu di macOS o della logica dello stato
---

<div id="menu-bar-status-logic">
  # Logica dello stato della barra dei menu
</div>

<div id="what-is-shown">
  ## Cosa viene mostrato
</div>

- Visualizziamo lo stato di attività corrente dell'agente nell'icona della barra dei menu e nella prima riga di stato del menu.
- Lo stato di salute è nascosto mentre è in corso un'attività; riappare quando tutte le sessioni sono inattive.
- La sezione “Nodi” nel menu elenca solo i **dispositivi** (nodi associati tramite `node.list`), non le voci relative a client/presenza.
- Una sezione “Utilizzo” viene visualizzata sotto Context quando sono disponibili istantanee di utilizzo del provider.

<div id="state-model">
  ## Modello di stato
</div>

- Sessioni: gli eventi arrivano con `runId` (per singola esecuzione) più `sessionKey` nel payload. La sessione "principale" ha la chiave `main`; se assente, usiamo come fallback la sessione aggiornata più di recente.
- Priorità: la principale vince sempre. Se la principale è attiva, il suo stato viene mostrato immediatamente. Se la principale è inattiva, viene mostrata la sessione non principale attiva più di recente. Non effettuiamo switch a metà di un'attività; cambiamo solo quando la sessione corrente diventa inattiva o la principale diventa attiva.
- Tipi di attività:
  - `job`: esecuzione di comandi di alto livello (`state: started|streaming|done|error`).
  - `tool`: `phase: start|result` con `toolName` e `meta/args`.

<div id="iconstate-enum-swift">
  ## enum IconState (Swift)
</div>

- `idle`
- `workingMain(ActivityKind)`
- `workingOther(ActivityKind)`
- `overridden(ActivityKind)` (sovrascrittura di debug)

<div id="activitykind-glyph">
  ### ActivityKind → glifo
</div>

- `exec` → 💻
- `read` → 📄
- `write` → ✍️
- `edit` → 📝
- `attach` → 📎
- predefinito → 🛠️

<div id="visual-mapping">
  ### Mappatura visiva
</div>

- `idle`: critter normale.
- `workingMain`: badge con glifo, tinta piena, animazione delle zampe "al lavoro".
- `workingOther`: badge con glifo, tinta attenuata, nessuna animazione di corsa.
- `overridden`: usa il glifo e la tinta scelti indipendentemente dall'attività.

<div id="status-row-text-menu">
  ## Testo della riga di stato (menu)
</div>

- Quando è in corso un'attività: `<Session role> · <activity label>`
  - Esempi: `Main · exec: pnpm test`, `Other · read: apps/macos/Sources/OpenClaw/AppState.swift`.
- Quando è inattivo: viene visualizzato il riepilogo dello stato di salute.

<div id="event-ingestion">
  ## Ingestione di eventi
</div>

- Origine: eventi `agent` del canale di controllo (`ControlChannel.handleAgentEvent`).
- Campi interpretati:
  - `stream: "job"` con `data.state` per avvio/arresto.
  - `stream: "tool"` con `data.phase`, `name`, `meta`/`args` opzionali.
- Etichette:
  - `exec`: prima riga di `args.command`.
  - `read`/`write`: percorso abbreviato.
  - `edit`: percorso più tipo di modifica dedotto da `meta`/conteggi delle diff.
  - fallback: nome del tool.

<div id="debug-override">
  ## Override di debug
</div>

- Settings ▸ Debug ▸ selettore “Icon override”:
  - `System (auto)` (predefinito)
  - `Working: main` (per tipo di tool)
  - `Working: other` (per tipo di tool)
  - `Idle`
- Salvato tramite `@AppStorage("iconOverride")`; mappato su `IconState.overridden`.

<div id="testing-checklist">
  ## Checklist di test
</div>

- Attiva il job della sessione principale: verifica che l'icona cambi immediatamente e che la riga di stato mostri l'etichetta principale.
- Attiva un job di una sessione non principale mentre la principale è inattiva: icona/stato mostrano la non principale; rimane stabile fino al termine.
- Avvia la principale mentre un'altra è attiva: l'icona passa alla principale istantaneamente.
- Raffiche rapide di strumenti: assicurati che il badge non lampeggi (margine di tolleranza TTL sui risultati degli strumenti).
- La riga Health riappare quando tutte le sessioni sono inattive.