---
title: ClawHub
summary: "Guida a ClawHub: registro pubblico di abilità + flussi di lavoro CLI"
read_when:
  - Presentare ClawHub a nuovi utenti
  - Installare, cercare o pubblicare abilità
  - Spiegare i flag della CLI di ClawHub e il comportamento di sincronizzazione
---

<div id="clawhub">
  # ClawHub
</div>

ClawHub è **il registro pubblico delle abilità di OpenClaw**. È un servizio gratuito: tutte le abilità sono pubbliche, aperte e visibili a chiunque, per condividerle e riutilizzarle. Un&#39;abilità è semplicemente una cartella con un file `SKILL.md` (più eventuali file di testo di supporto). Puoi esplorare le abilità nella web app oppure usare la CLI per cercare, installare, aggiornare e pubblicare abilità.

Sito: [clawhub.com](https://clawhub.com)

<div id="who-this-is-for-beginner-friendly">
  ## A chi è rivolto (adatto ai principianti)
</div>

Se vuoi aggiungere nuove funzionalità al tuo agente OpenClaw, ClawHub è il modo più semplice per trovare e installare abilità. Non devi sapere come funziona il backend. Puoi:

* Cercare abilità usando il linguaggio naturale.
* Installare un&#39;abilità nel tuo spazio di lavoro.
* Aggiornare le abilità in seguito con un solo comando.
* Eseguire il backup delle tue abilità pubblicandole.

<div id="quick-start-non-technical">
  ## Avvio rapido (non tecnico)
</div>

1. Installa la CLI (vedi la sezione successiva).
2. Cerca qualcosa che ti serve:
   * `clawhub search "calendar"`
3. Installa una skill:
   * `clawhub install <skill-slug>`
4. Avvia una nuova sessione OpenClaw in modo che rilevi la nuova skill.

<div id="install-the-cli">
  ## Installa la CLI
</div>

Scegline uno:

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```

<div id="how-it-fits-into-openclaw">
  ## Come si integra in OpenClaw
</div>

Per impostazione predefinita, la CLI installa le abilità in `./skills` all&#39;interno della directory di lavoro corrente. Se è configurato uno spazio di lavoro di OpenClaw, `clawhub` utilizza quello spazio di lavoro come fallback, a meno che tu non sovrascriva `--workdir` (o `CLAWHUB_WORKDIR`). OpenClaw carica le abilità dello spazio di lavoro da `<workspace>/skills` e le utilizzerà nella **prossima** sessione. Se usi già `~/.openclaw/skills` o abilità incluse nel pacchetto, le abilità dello spazio di lavoro hanno la precedenza.

Per ulteriori dettagli su come le abilità vengono caricate, condivise e soggette a controlli di accesso, vedi
[Skills](/it/tools/skills).

<div id="what-the-service-provides-features">
  ## Cosa offre il servizio (funzionalità)
</div>

* **Navigazione pubblica** delle abilità e del relativo contenuto `SKILL.md`.
* **Ricerca** basata su embeddings (ricerca vettoriale), non solo parole chiave.
* **Versionamento** con semver, changelog e tag (incluso `latest`).
* **Download** come archivio zip per ogni versione.
* **Stelle e commenti** per il feedback della community.
* Hook di **moderazione** per approvazioni e verifiche.
* **API adatta all’uso via CLI** per automazione e scripting.

<div id="cli-commands-and-parameters">
  ## Comandi e parametri CLI
</div>

Opzioni globali (si applicano a tutti i comandi):

* `--workdir <dir>`: Directory di lavoro (predefinita: directory corrente; in assenza usa lo spazio di lavoro OpenClaw).
* `--dir <dir>`: Directory delle abilità, relativa a workdir (predefinita: `skills`).
* `--site <url>`: URL di base del sito (login tramite browser).
* `--registry <url>`: URL di base della Registry API.
* `--no-input`: Disabilita i prompt (modalità non interattiva).
* `-V, --cli-version`: Stampa la versione della CLI.

Auth:

* `clawhub login` (flusso tramite browser) oppure `clawhub login --token <token>`
* `clawhub logout`
* `clawhub whoami`

Opzioni:

* `--token <token>`: Incolla un API token.
* `--label <label>`: Etichetta memorizzata per i token di login via browser (predefinita: `CLI token`).
* `--no-browser`: Non aprire il browser (richiede `--token`).

Search:

* `clawhub search "query"`
* `--limit <n>`: Numero massimo di risultati.

Install:

* `clawhub install <slug>`
* `--version <version>`: Installa una versione specifica.
* `--force`: Sovrascrive se la cartella esiste già.

Update:

* `clawhub update <slug>`
* `clawhub update --all`
* `--version <version>`: Aggiorna a una versione specifica (solo per un singolo slug).
* `--force`: Sovrascrive quando i file locali non corrispondono a nessuna versione pubblicata.

List:

* `clawhub list` (legge `.clawhub/lock.json`)

Publish:

* `clawhub publish <path>`
* `--slug <slug>`: Slug della skill.
* `--name <name>`: Nome visualizzato.
* `--version <version>`: Versione Semver.
* `--changelog <text>`: Testo del changelog (può essere vuoto).
* `--tags <tags>`: Tag separati da virgola (predefinito: `latest`).

Delete/undelete (solo owner/admin):

* `clawhub delete <slug> --yes`
* `clawhub undelete <slug> --yes`

Sync (scansiona abilità locali + pubblica nuove/aggiornate):

* `clawhub sync`
* `--root <dir...>`: Directory radice aggiuntive da scansionare.
* `--all`: Carica tutto senza prompt.
* `--dry-run`: Mostra cosa verrebbe caricato.
* `--bump <type>`: `patch|minor|major` per gli aggiornamenti (predefinito: `patch`).
* `--changelog <text>`: Changelog per aggiornamenti non interattivi.
* `--tags <tags>`: Tag separati da virgola (predefinito: `latest`).
* `--concurrency <n>`: Controlli verso il registry (predefinito: 4).

<div id="common-workflows-for-agents">
  ## Flussi di lavoro tipici degli agenti
</div>

<div id="search-for-skills">
  ### Cerca abilità
</div>

```bash
clawhub search "postgres backups"
```

<div id="download-new-skills">
  ### Scarica nuove abilità
</div>

```bash
clawhub install my-skill-pack
```

<div id="update-installed-skills">
  ### Aggiornare le abilità installate
</div>

```bash
clawhub update --all
```

<div id="back-up-your-skills-publish-or-sync">
  ### Esegui il backup delle tue abilità (pubblica o sincronizza)
</div>

Per la cartella di una singola abilità:

```bash
clawhub publish ./my-skill --slug my-skill --name "My Skill" --version 1.0.0 --tags latest
```

Per eseguire la scansione e il backup di più abilità contemporaneamente:

```bash
clawhub sync --all
```

<div id="advanced-details-technical">
  ## Dettagli tecnici avanzati
</div>

<div id="versioning-and-tags">
  ### Versioni e tag
</div>

* Ogni operazione di publish crea una nuova `SkillVersion` con versionamento **semver**.
* I tag (come `latest`) puntano a una versione; spostarli ti permette di ripristinare una versione precedente.
* I changelog vengono associati a ciascuna versione e possono essere vuoti durante la sincronizzazione o la pubblicazione di aggiornamenti.

<div id="local-changes-vs-registry-versions">
  ### Modifiche locali vs versioni del registry
</div>

Gli aggiornamenti confrontano i contenuti delle skill locali con le versioni nel registry utilizzando un hash dei contenuti. Se i file locali non corrispondono a nessuna versione pubblicata, la CLI chiede conferma prima di sovrascrivere (oppure richiede l’uso di `--force` nelle esecuzioni non interattive).

<div id="sync-scanning-and-fallback-roots">
  ### Scansione di sincronizzazione e percorsi di fallback
</div>

`clawhub sync` esegue prima la scansione della directory di lavoro corrente. Se non vengono trovate abilità, ricorre a percorsi legacy noti (ad esempio `~/openclaw/skills` e `~/.openclaw/skills`). Questo consente di individuare installazioni di abilità meno recenti senza dover specificare flag aggiuntivi.

<div id="storage-and-lockfile">
  ### Archiviazione e lockfile
</div>

* Le abilità installate sono registrate in `.clawhub/lock.json` nella tua directory di lavoro.
* I token di autenticazione sono memorizzati nel file di configurazione della CLI di ClawHub (puoi sovrascriverlo impostando `CLAWHUB_CONFIG_PATH`).

<div id="telemetry-install-counts">
  ### Telemetria (conteggio delle installazioni)
</div>

Quando esegui `clawhub sync` mentre hai effettuato l&#39;accesso, la CLI invia un&#39;istantanea minima per calcolare il conteggio delle installazioni. Puoi disabilitare completamente questo comportamento:

```bash
export CLAWHUB_DISABLE_TELEMETRY=1
```

<div id="environment-variables">
  ## Variabili d&#39;ambiente
</div>

* `CLAWHUB_SITE`: Sovrascrive l&#39;URL del sito.
* `CLAWHUB_REGISTRY`: Sovrascrive l&#39;URL dell&#39;API del registro.
* `CLAWHUB_CONFIG_PATH`: Sovrascrive il percorso in cui la CLI archivia token e configurazione.
* `CLAWHUB_WORKDIR`: Sovrascrive la directory di lavoro predefinita.
* `CLAWHUB_DISABLE_TELEMETRY=1`: Disabilita la telemetria per `sync`.