---
title: Problema con Node
summary: Note e workaround per il crash di Node + tsx "__name is not a function"
read_when:
  - Per eseguire il debug di script di sviluppo solo-Node o dei problemi in modalità watch
  - Per analizzare i crash del loader tsx/esbuild in OpenClaw
---

<div id="node-tsx-__name-is-not-a-function-crash">
  # Arresto anomalo di Node + tsx "__name is not a function"
</div>

<div id="summary">
  ## Riepilogo
</div>

L&#39;esecuzione di OpenClaw tramite Node con `tsx` fallisce all&#39;avvio con il seguente errore:

```
[openclaw] Avvio della CLI non riuscito: TypeError: __name is not a function
    at createSubsystemLogger (.../src/logging/subsystem.ts:203:25)
    at .../src/agents/auth-profiles/constants.ts:25:20
```

Questo problema è iniziato dopo il passaggio degli script di sviluppo da Bun a `tsx` (commit `2871657e`, 2026-01-06). Lo stesso percorso di runtime funzionava con Bun.


<div id="environment">
  ## Ambiente
</div>

- Node: v25.x (osservato su v25.3.0)
- tsx: 4.21.0
- OS: macOS (probabilmente riproducibile anche su altre piattaforme che eseguono Node 25)

<div id="repro-node-only">
  ## Riproduzione (solo Node.js)
</div>

```bash
# nella root del repository
node --version
pnpm install
node --import tsx src/entry.ts status
```


<div id="minimal-repro-in-repo">
  ## Esempio minimo riproducibile nel repository
</div>

```bash
node --import tsx scripts/repro/tsx-name-repro.ts
```


<div id="node-version-check">
  ## Verifica della versione di Node
</div>

- Node 25.3.0: fallisce
- Node 22.22.0 (Homebrew `node@22`): fallisce
- Node 24: non è ancora installato qui; da verificare

<div id="notes-hypothesis">
  ## Note / ipotesi
</div>

- `tsx` usa esbuild per trasformare TS/ESM. L’opzione `keepNames` di esbuild emette un helper `__name` e avvolge le definizioni di funzione con `__name(...)`.
- Il crash indica che `__name` esiste ma non è una funzione a runtime, il che implica che l’helper manca o è stato sovrascritto per questo modulo nel percorso di caricamento di Node 25.
- Problemi simili con l’helper `__name` sono stati segnalati in altri consumatori di esbuild quando l’helper manca o viene riscritto.

<div id="regression-history">
  ## Cronologia delle regressioni
</div>

- `2871657e` (2026-01-06): gli script sono stati cambiati da Bun a tsx per rendere Bun opzionale.
- Prima di allora (percorso con Bun), `openclaw status` e `gateway:watch` funzionavano.

<div id="workarounds">
  ## Soluzioni alternative
</div>

- Usa Bun per gli script di sviluppo (attuale ripristino temporaneo).
- Usa Node + tsc in modalità watch, quindi esegui l'output compilato:
  ```bash
  pnpm exec tsc --watch --preserveWatchOutput
  node --watch openclaw.mjs status
  ```
- Confermato localmente: `pnpm exec tsc -p tsconfig.json` + `node openclaw.mjs status` funzionano su Node 25.
- Disabilita `keepNames` di esbuild nel TS loader se possibile (evita l'inserimento dell'helper `__name`); tsx al momento non espone questa opzione.
- Prova Node LTS (22/24) con `tsx` per verificare se il problema è specifico di Node 25.

<div id="references">
  ## Riferimenti
</div>

- https://opennext.js.org/cloudflare/howtos/keep_names
- https://esbuild.github.io/api/#keep-names
- https://github.com/evanw/esbuild/issues/1031

<div id="next-steps">
  ## Prossimi passi
</div>

- Riproduci il problema su Node 22/24 per confermare la regressione su Node 25.
- Prova la versione nightly di `tsx` oppure blocca a una versione precedente se esiste una regressione nota.
- Se il problema si riproduce su Node LTS, invia una segnalazione upstream con una repro minimale e lo stack trace di `__name`.