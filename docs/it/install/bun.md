---
title: Bun
summary: "Workflow Bun (sperimentale): installazioni e trappole rispetto a pnpm"
read_when:
  - Vuoi il ciclo di sviluppo locale più veloce possibile (bun + watch)
  - Ti imbatti in problemi con gli script di install/patch/lifecycle di Bun
---

<div id="bun-experimental">
  # Bun (sperimentale)
</div>

Obiettivo: eseguire questo repository con **Bun** (opzionale, sconsigliato per WhatsApp/Telegram)
senza discostarsi dai flussi di lavoro pnpm.

⚠️ **Sconsigliato per il runtime del Gateway** (bug con WhatsApp/Telegram). Usa Node in produzione.

<div id="status">
  ## Stato
</div>

- Bun è un runtime locale opzionale per eseguire TypeScript direttamente (`bun run …`, `bun --watch …`).
- `pnpm` è lo strumento predefinito per le build e rimane pienamente supportato (ed è utilizzato da alcuni strumenti della documentazione).
- Bun non utilizza `pnpm-lock.yaml` e lo ignora.

<div id="install">
  ## Installazione
</div>

Impostazione predefinita:

```sh
bun install
```

Nota: `bun.lock`/`bun.lockb` sono ignorati da git, quindi non si generano modifiche nel repository in ogni caso. Se vuoi evitare *qualsiasi scrittura del lockfile*:

```sh
bun install --no-save
```


<div id="build-test-bun">
  ## Build / test (Bun)
</div>

```sh
bun run build
bun run vitest run
```


<div id="bun-lifecycle-scripts-blocked-by-default">
  ## Script di lifecycle di Bun (bloccati per impostazione predefinita)
</div>

Bun può bloccare gli script di lifecycle delle dipendenze a meno che non siano esplicitamente considerati attendibili (`bun pm untrusted` / `bun pm trust`).
Per questo repository, gli script comunemente bloccati non sono necessari:

* `@whiskeysockets/baileys` `preinstall`: verifica che la versione major di Node sia &gt;= 20 (noi eseguiamo Node 22+).
* `protobufjs` `postinstall`: emette avvisi su schemi di versioni incompatibili (nessun artefatto di build).

Se incontri un reale problema di runtime che richiede questi script, contrassegnali esplicitamente come attendibili:

```sh
bun pm trust @whiskeysockets/baileys protobufjs
```


<div id="caveats">
  ## Avvertenze
</div>

- Alcuni script fanno ancora riferimento esplicito a pnpm (ad esempio `docs:build`, `ui:*`, `protocol:check`). Per ora eseguili tramite pnpm.