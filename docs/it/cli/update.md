---
title: Aggiornamento
summary: "Riferimento CLI per `openclaw update` (aggiornamento del sorgente relativamente sicuro + riavvio automatico del Gateway)"
read_when:
  - Vuoi aggiornare in modo sicuro un checkout del sorgente
  - Devi capire il comportamento della forma abbreviata `--update`
---

<div id="openclaw-update">
  # `openclaw update`
</div>

Aggiorna OpenClaw in modo sicuro e consente di passare tra i canali stable/beta/dev.

Se hai installato tramite **npm/pnpm** (installazione globale, senza metadati git), gli aggiornamenti avvengono tramite la procedura del gestore di pacchetti descritta in [Updating](/it/install/updating).

<div id="usage">
  ## Utilizzo
</div>

```bash
openclaw update
openclaw update status
openclaw update wizard
openclaw update --channel beta
openclaw update --channel dev
openclaw update --tag beta
openclaw update --no-restart
openclaw update --json
openclaw --update
```

<div id="options">
  ## Opzioni
</div>

* `--no-restart`: evita il riavvio del servizio Gateway dopo un aggiornamento riuscito.
* `--channel <stable|beta|dev>`: imposta il canale di aggiornamento (git + npm; memorizzato nella configurazione).
* `--tag <dist-tag|version>`: sovrascrive il dist-tag o la versione npm solo per questo aggiornamento.
* `--json`: stampa `UpdateRunResult` in formato JSON adatto all’elaborazione automatica.
* `--timeout <seconds>`: timeout per ogni fase (predefinito 1200s).

Nota: i downgrade richiedono conferma perché le versioni precedenti possono rendere non valida la configurazione.

<div id="update-status">
  ## `update status`
</div>

Mostra il canale di aggiornamento attivo e il tag/branch/SHA di git (per le installazioni dal sorgente), oltre alla disponibilità di eventuali aggiornamenti.

```bash
openclaw update status
openclaw update status --json
openclaw update status --timeout 10
```

Opzioni:

* `--json`: stampa lo stato in formato JSON leggibile dalla macchina.
* `--timeout <seconds>`: timeout per i controlli (valore predefinito: 3s).

<div id="update-wizard">
  ## `update wizard`
</div>

Procedura interattiva per scegliere un canale di aggiornamento e confermare se riavviare il Gateway
dopo l&#39;aggiornamento (per impostazione predefinita viene eseguito il riavvio). Se selezioni `dev` senza un checkout Git,
propone di crearne uno.

<div id="what-it-does">
  ## Cosa fa
</div>

Quando passi esplicitamente a un canale (`--channel ...`), OpenClaw mantiene anche
coerente il metodo di installazione:

* `dev` → garantisce un checkout git (predefinito: `~/openclaw`, modificabile con `OPENCLAW_GIT_DIR`),
  lo aggiorna e installa la CLI globale da quel checkout.
* `stable`/`beta` → installa da npm usando il dist-tag corrispondente.

<div id="git-checkout-flow">
  ## Flusso di checkout Git
</div>

Canali:

* `stable`: esegue il checkout dell&#39;ultimo tag non beta, quindi build + doctor.
* `beta`: esegue il checkout dell&#39;ultimo tag `-beta`, quindi build + doctor.
* `dev`: esegue il checkout di `main`, quindi fetch + rebase.

Panoramica generale:

1. Richiede una worktree pulita (nessuna modifica non committata).
2. Passa al canale selezionato (tag o branch).
3. Esegue il fetch da upstream (solo dev).
4. Solo dev: esegue un lint preliminare (preflight) + build TypeScript in una worktree temporanea; se l&#39;ultima revisione fallisce, torna indietro fino a 10 commit per trovare la build più recente che va a buon fine.
5. Esegue il rebase sul commit selezionato (solo dev).
6. Installa le dipendenze (pnpm preferito; npm come fallback).
7. Esegue la build del core e della Control UI.
8. Esegue `openclaw doctor` come controllo finale di “safe update”.
9. Sincronizza i plugin con il canale attivo (dev usa le estensioni incluse nel bundle; stable/beta usa npm) e aggiorna i plugin installati tramite npm.

<div id="update-shorthand">
  ## scorciatoia `--update`
</div>

`openclaw --update` è equivalente a `openclaw update` (utile per shell e script di lancio).

<div id="see-also">
  ## Vedi anche
</div>

* `openclaw doctor` (può eseguire prima l&#39;aggiornamento sui checkout Git)
* [Canali di sviluppo](/it/install/development-channels)
* [Aggiornare](/it/install/updating)
* [Riferimento alla CLI](/it/cli)