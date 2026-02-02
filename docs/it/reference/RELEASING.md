---
title: RILASCIO
summary: "Checklist di rilascio passo-passo per npm + app macOS"
read_when:
  - Quando prepari un nuovo rilascio npm
  - Quando prepari un nuovo rilascio dell'app macOS
  - Quando verifichi i metadati prima della pubblicazione
---

<div id="release-checklist-npm-macos">
  # Checklist di rilascio (npm + macOS)
</div>

Usa `pnpm` (Node 22+) dalla directory principale del repository. Mantieni l&#39;albero di lavoro pulito prima di creare il tag e pubblicare.

<div id="operator-trigger">
  ## Attivazione da parte dell&#39;operatore
</div>

Quando l&#39;operatore dice &quot;release&quot;, esegui immediatamente questo check preliminare (nessuna domanda aggiuntiva a meno che qualcosa blocchi):

* Leggi questo documento e `docs/platforms/mac/release.md`.
* Carica le variabili d&#39;ambiente da `~/.profile` e conferma che `SPARKLE_PRIVATE_KEY_FILE` + le variabili di App Store Connect siano impostate (`SPARKLE_PRIVATE_KEY_FILE` dovrebbe risiedere in `~/.profile`).
* Usa le chiavi Sparkle da `~/Library/CloudStorage/Dropbox/Backup/Sparkle` se necessario.

1. **Versione &amp; metadati**

* [ ] Incrementa la versione in `package.json` (ad es. `2026.1.29`).
* [ ] Esegui `pnpm plugins:sync` per allineare le versioni dei pacchetti delle estensioni + i changelog.
* [ ] Aggiorna le stringhe di versione della CLI: [`src/cli/program.ts`](https://github.com/openclaw/openclaw/blob/main/src/cli/program.ts) e il Baileys user agent in [`src/provider-web.ts`](https://github.com/openclaw/openclaw/blob/main/src/provider-web.ts).
* [ ] Conferma i metadati del pacchetto (name, description, repository, keywords, license) e che la mappa `bin` punti a [`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs) per `openclaw`.
* [ ] Se le dipendenze sono cambiate, esegui `pnpm install` in modo che `pnpm-lock.yaml` sia aggiornato.

2. **Build &amp; artefatti**

* [ ] Se gli input A2UI sono cambiati, esegui `pnpm canvas:a2ui:bundle` e fai commit di ogni [`src/canvas-host/a2ui/a2ui.bundle.js`](https://github.com/openclaw/openclaw/blob/main/src/canvas-host/a2ui/a2ui.bundle.js) aggiornato.
* [ ] `pnpm run build` (rigenera `dist/`).
* [ ] Verifica che il campo `files` del pacchetto npm includa tutte le cartelle `dist/*` richieste (in particolare `dist/node-host/**` e `dist/acp/**` per il nodo headless + ACP CLI).
* [ ] Conferma che `dist/build-info.json` esista e includa l&#39;hash `commit` previsto (il banner della CLI lo usa per le installazioni da npm).
* [ ] Opzionale: `npm pack --pack-destination /tmp` dopo la build; ispeziona il contenuto del tarball e tienilo a portata di mano per la release su GitHub (non fare **commit**).

3. **Changelog &amp; documentazione**

* [ ] Aggiorna `CHANGELOG.md` con le novità principali rivolte agli utenti (crea il file se manca); mantieni le voci rigorosamente in ordine decrescente per versione.
* [ ] Assicurati che gli esempi e i flag del README corrispondano al comportamento attuale della CLI (in particolare nuovi comandi o opzioni).

4. **Validazione**

* [ ] `pnpm lint`
* [ ] `pnpm test` (o `pnpm test:coverage` se ti serve l&#39;output di coverage)
* [ ] `pnpm run build` (ultimo sanity check dopo i test)
* [ ] `pnpm release:check` (verifica i contenuti del pacchetto npm)
* [ ] `OPENCLAW_INSTALL_SMOKE_SKIP_NONROOT=1 pnpm test:install:smoke` (smoke test di installazione Docker, percorso rapido; richiesto prima della release)
  * Se la release npm immediatamente precedente è nota come difettosa, imposta `OPENCLAW_INSTALL_SMOKE_PREVIOUS=<last-good-version>` o `OPENCLAW_INSTALL_SMOKE_SKIP_PREVIOUS=1` per la fase di preinstall.
* [ ] (Opzionale) Smoke test completo dell&#39;installer (aggiunge copertura non-root + CLI): `pnpm test:install:smoke`
* [ ] (Opzionale) Installer E2E (Docker, esegue `curl -fsSL https://openclaw.bot/install.sh | bash`, effettua l&#39;onboarding, poi esegue vere chiamate agli strumenti):
  * `pnpm test:install:e2e:openai` (richiede `OPENAI_API_KEY`)
  * `pnpm test:install:e2e:anthropic` (richiede `ANTHROPIC_API_KEY`)
  * `pnpm test:install:e2e` (richiede entrambe le chiavi; esegue entrambi i provider)
* [ ] (Opzionale) Verifica a campione il Gateway web se le tue modifiche influiscono sui percorsi di invio/ricezione.

5. **App macOS (Sparkle)**

* [ ] Compila e firma l&#39;app macOS, quindi crea un archivio ZIP per la distribuzione.
* [ ] Genera lo Sparkle appcast (note HTML tramite [`scripts/make_appcast.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/make_appcast.sh)) e aggiorna `appcast.xml`.
* [ ] Tieni lo ZIP dell&#39;app (e l&#39;eventuale ZIP dSYM) pronto per essere allegato alla release GitHub.
* [ ] Segui [macOS release](/it/platforms/mac/release) per i comandi esatti e le variabili d&#39;ambiente richieste.
  * `APP_BUILD` deve essere numerico e strettamente crescente (nessun `-beta`) in modo che Sparkle confronti correttamente le versioni.
  * Se effettui la notarizzazione, usa il profilo del portachiavi `openclaw-notary` creato dalle variabili d&#39;ambiente API di App Store Connect (vedi [macOS release](/it/platforms/mac/release)).

6. **Pubblica (npm)**

* [ ] Verifica che lo stato di git sia pulito; effettua commit e push se necessario.
* [ ] Esegui `npm login` (verifica la 2FA) se necessario.
* [ ] Esegui `npm publish --access public` (usa `--tag beta` per le pre-release).
* [ ] Verifica il registry npm: `npm view openclaw version`, `npm view openclaw dist-tags` e `npx -y openclaw@X.Y.Z --version` (o `--help`).

<div id="troubleshooting-notes-from-200-beta2-release">
  ### Risoluzione dei problemi (note dalla release 2.0.0-beta2)
</div>

* **`npm pack/publish` si blocca o produce un tarball enorme**: il bundle dell&#39;app macOS in `dist/OpenClaw.app` (e gli zip della release) viene incluso nel pacchetto. Correggi limitando esplicitamente i contenuti da pubblicare tramite il campo `files` in `package.json` (includi le sottodirectory di `dist`, la documentazione, le abilità; escludi i bundle delle app). Verifica con `npm pack --dry-run` che `dist/OpenClaw.app` non sia elencato.
* **Loop di autenticazione web npm per i dist-tag**: usa l&#39;autenticazione legacy per ottenere il prompt OTP:
  * `NPM_CONFIG_AUTH_TYPE=legacy npm dist-tag add openclaw@X.Y.Z latest`
* **Verifica `npx` non riuscita con `ECOMPROMISED: Lock compromised`**: riprova con una cache nuova:
  * `NPM_CONFIG_CACHE=/tmp/npm-cache-$(date +%s) npx -y openclaw@X.Y.Z --version`
* **Il tag deve essere riassegnato dopo una correzione tardiva**: forza l&#39;aggiornamento e fai push del tag, quindi verifica che gli asset della release GitHub corrispondano ancora:
  * `git tag -f vX.Y.Z && git push -f origin vX.Y.Z`

7. **GitHub release + appcast**

* [ ] Tagga e fai push: `git tag vX.Y.Z && git push origin vX.Y.Z` (oppure `git push --tags`).
* [ ] Crea/aggiorna la release GitHub per `vX.Y.Z` con **titolo `openclaw X.Y.Z`** (non solo il tag); il corpo deve includere l&#39;intera sezione del changelog per quella versione (Highlights + Changes + Fixes), in linea (nessun link nudo), e **non deve ripetere il titolo all&#39;interno del corpo**.
* [ ] Allega gli artefatti: tarball di `npm pack` (opzionale), `OpenClaw-X.Y.Z.zip` e `OpenClaw-X.Y.Z.dSYM.zip` (se generato).
* [ ] Effettua il commit di `appcast.xml` aggiornato e fai push (Sparkle legge dal branch `main`).
* [ ] Da una directory temporanea pulita (senza `package.json`), esegui `npx -y openclaw@X.Y.Z send --help` per confermare che installazione e entrypoint della CLI funzionino.
* [ ] Annuncia/condividi le note di release.

<div id="plugin-publish-scope-npm">
  ## Ambito di pubblicazione dei plugin (npm)
</div>

Pubbliciamo solo i **plugin npm esistenti** all&#39;interno dello scope `@openclaw/*`. I plugin
inclusi che non sono su npm restano **solo nell&#39;albero su disco** (comunque forniti in
`extensions/**`).

Procedura per ricavare l&#39;elenco:

1. Esegui `npm search @openclaw --json` e recupera i nomi dei package.
2. Confrontali con i nomi in `extensions/*/package.json`.
3. Pubblica solo l&#39;**intersezione** (già presenti su npm).

Elenco attuale dei plugin npm (aggiornare se necessario):

* @openclaw/bluebubbles
* @openclaw/diagnostics-otel
* @openclaw/discord
* @openclaw/lobster
* @openclaw/matrix
* @openclaw/msteams
* @openclaw/nextcloud-talk
* @openclaw/nostr
* @openclaw/voice-call
* @openclaw/zalo
* @openclaw/zalouser

Le note di rilascio devono anche evidenziare i **nuovi plugin opzionali inclusi**
che **non sono attivi di default** (esempio: `tlon`).