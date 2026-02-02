---
title: Config rigorosa
summary: "Convalida rigorosa della config + migrazioni gestite esclusivamente da doctor"
read_when:
  - Progettazione o implementazione del comportamento di convalida della config
  - Lavoro su migrazioni della config o workflow di doctor
  - Gestione degli schemi di config dei plugin o del gating del caricamento dei plugin
---

<div id="strict-config-validation-doctor-only-migrations">
  # Validazione rigorosa della configurazione (migrazioni solo tramite doctor)
</div>

<div id="goals">
  ## Obiettivi
</div>

- **Rifiutare ovunque chiavi di configurazione sconosciute** (root e annidate).
- **Rifiutare configurazioni dei plugin prive di schema**; non caricare il relativo plugin.
- **Rimuovere la migrazione automatica legacy al caricamento**; le migrazioni vengono eseguite solo tramite doctor.
- **Eseguire automaticamente doctor (dry-run) all’avvio**; se la configurazione non è valida, bloccare i comandi non diagnostici.

<div id="non-goals">
  ## Non-obiettivi
</div>

- Compatibilità retroattiva in fase di caricamento (le chiavi legacy non vengono migrate automaticamente).
- Eliminazione silenziosa delle chiavi non riconosciute.

<div id="strict-validation-rules">
  ## Regole di validazione rigorose
</div>

- La configurazione deve corrispondere esattamente allo schema a ogni livello.
- Le chiavi sconosciute sono considerate errori di validazione (nessun pass-through né alla radice né nei livelli nidificati).
- `plugins.entries.<id>.config` deve essere validato dallo schema del plugin.
  - Se un plugin è privo di schema, **rifiuta il caricamento del plugin** e mostra un errore chiaro.
- Le chiavi `channels.<id>` sconosciute sono errori, a meno che un manifest del plugin dichiari l'ID del canale.
- I manifest dei plugin (`openclaw.plugin.json`) sono obbligatori per tutti i plugin.

<div id="plugin-schema-enforcement">
  ## Applicazione dello schema dei plugin
</div>

- Ogni plugin fornisce uno schema JSON rigoroso per la propria configurazione (direttamente nel manifest).
- Flusso di caricamento del plugin:
  1) Risolvi il manifest e lo schema del plugin (`openclaw.plugin.json`).
  2) Valida la configurazione rispetto allo schema.
  3) Se manca lo schema o la configurazione non è valida: blocca il caricamento del plugin e registra l'errore.
- Il messaggio di errore include:
  - ID del plugin
  - Motivo (schema mancante / configurazione non valida)
  - Percorso(i) che non hanno superato la validazione
- I plugin disabilitati mantengono la loro configurazione, ma Doctor + log segnalano un avviso.

<div id="doctor-flow">
  ## Flusso di Doctor
</div>

- Doctor viene eseguito **ogni volta** che la configurazione viene caricata (dry-run per impostazione predefinita).
- Se la configurazione non è valida:
  - Stampa un riepilogo + errori con azioni eseguibili.
  - Suggerisce di eseguire: `openclaw doctor --fix`.
- `openclaw doctor --fix`:
  - Applica le migrazioni.
  - Rimuove le chiavi sconosciute.
  - Scrive la configurazione aggiornata.

<div id="command-gating-when-config-is-invalid">
  ## Blocco dei comandi (quando la configurazione non è valida)
</div>

Consentiti (solo per diagnostica):

- `openclaw doctor`
- `openclaw logs`
- `openclaw health`
- `openclaw help`
- `openclaw status`
- `openclaw gateway status`

Tutti gli altri comandi devono fallire in modo bloccante con: “Configurazione non valida. Esegui `openclaw doctor --fix`.”

<div id="error-ux-format">
  ## Formato UX degli errori
</div>

- Intestazione di riepilogo unica.
- Sezioni raggruppate:
  - Chiavi sconosciute (percorsi completi)
  - Chiavi legacy / migrazioni necessarie
  - Errori di caricamento dei plugin (ID del plugin + motivo + percorso)

<div id="implementation-touchpoints">
  ## Punti di intervento per l’implementazione
</div>

- `src/config/zod-schema.ts`: rimuovere il passthrough alla radice; oggetti strict ovunque.
- `src/config/zod-schema.providers.ts`: garantire schemi di canale strict.
- `src/config/validation.ts`: generare errore in caso di chiavi sconosciute; non applicare le migrazioni legacy.
- `src/config/io.ts`: rimuovere le migrazioni automatiche legacy; eseguire sempre il doctor in modalità dry-run.
- `src/config/legacy*.ts`: limitare l’utilizzo al solo doctor.
- `src/plugins/*`: aggiungere registro degli schemi + gating.
- Gating dei comandi CLI in `src/cli`.

<div id="tests">
  ## Test
</div>

- Rifiuto di chiavi sconosciute (a livello root e annidate).
- Schema del plugin mancante → caricamento del plugin bloccato con errore esplicito.
- Configurazione non valida → avvio del Gateway bloccato, eccetto per i comandi diagnostici.
- Dry-run automatico di `doctor`; `doctor --fix` scrive la configurazione corretta.