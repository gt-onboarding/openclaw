---
title: Canali di sviluppo
summary: "Canali stable, beta e dev: semantica, cambio di canale ed etichettatura"
read_when:
  - Vuoi passare tra i canali stable/beta/dev
  - Stai creando tag o pubblicando versioni pre-release
---

<div id="development-channels">
  # Canali di sviluppo
</div>

Ultimo aggiornamento: 2026-01-21

OpenClaw distribuisce tre canali di aggiornamento:

- **stable**: npm dist-tag `latest`.
- **beta**: npm dist-tag `beta` (build in fase di test).
- **dev**: testa in continua evoluzione di `main` (git). npm dist-tag: `dev` (quando pubblicato).

Distribuiamo le build su **beta**, le testiamo, quindi **promuoviamo una build validata a `latest`**
senza cambiare il numero di versione — i dist-tag sono la fonte di verità per le installazioni npm.

<div id="switching-channels">
  ## Cambio canale
</div>

Git checkout:

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

* `stable`/`beta` effettua il checkout dell&#39;ultimo tag corrispondente (spesso lo stesso tag).
* `dev` passa a `main` ed esegue il rebase su upstream.

Installazione globale con npm/pnpm:

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

Questo viene aggiornato tramite il corrispondente dist-tag npm (`latest`, `beta`, `dev`).

Quando cambi **esplicitamente** canale con `--channel`, OpenClaw allinea anche
il metodo di installazione:

* `dev` effettua un checkout git (predefinito `~/openclaw`, sovrascrivibile con `OPENCLAW_GIT_DIR`),
  lo aggiorna e installa la CLI globale da quel checkout.
* `stable`/`beta` installa da npm usando il dist-tag corrispondente.

Suggerimento: se vuoi stabile + dev in parallelo, mantieni due cloni e punta il tuo Gateway a quello stabile.


<div id="plugins-and-channels">
  ## Plugin e canali
</div>

Quando cambi canale con `openclaw update`, OpenClaw sincronizza anche le sorgenti dei plugin:

- `dev` privilegia i plugin inclusi nel checkout di Git.
- `stable` e `beta` ripristinano i pacchetti plugin installati con npm.

<div id="tagging-best-practices">
  ## Buone pratiche di tagging
</div>

- Applica un tag alle release su cui vuoi che i checkout di Git si posizionino (`vYYYY.M.D` o `vYYYY.M.D-<patch>`).
- Mantieni i tag immutabili: non spostare né riutilizzare mai un tag.
- I dist-tag di npm restano la fonte di verità per le installazioni con npm:
  - `latest` → stabile
  - `beta` → build candidata
  - `dev` → snapshot di main (opzionale)

<div id="macos-app-availability">
  ## Disponibilità dell’app macOS
</div>

Le build beta e di sviluppo potrebbero **non** includere una versione dell’app macOS. Va bene così:

- Il tag git e il dist-tag npm possono comunque essere pubblicati.
- Segnala “nessuna build macOS per questa beta” nelle note di rilascio o nel changelog.