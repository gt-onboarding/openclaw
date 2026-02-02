---
title: Firma
summary: "Procedura di firma per build di debug macOS generate dagli script di packaging"
read_when:
  - Compilazione o firma di build di debug macOS
---

<div id="mac-signing-debug-builds">
  # mac signing (debug builds)
</div>

Questa app viene di solito generata tramite [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh), che ora:

* imposta un identificatore di bundle di debug stabile: `ai.openclaw.mac.debug`
* scrive l&#39;Info.plist con quel bundle id (sovrascrivibile tramite `BUNDLE_ID=...`)
* chiama [`scripts/codesign-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/codesign-mac-app.sh) per firmare il binario principale e il bundle dell&#39;app, così che macOS tratti ogni rebuild come lo stesso bundle firmato e mantenga i permessi TCC (notifiche, accessibilità, registrazione schermo, microfono, sintesi vocale). Per permessi stabili, usa una vera identità di firma; l&#39;ad-hoc è opzionale e fragile (vedi [macOS permissions](/it/platforms/mac/permissions)).
* usa `CODESIGN_TIMESTAMP=auto` come impostazione predefinita; abilita timestamp attendibili per le firme Developer ID. Imposta `CODESIGN_TIMESTAMP=off` per saltare il timestamping (build di debug offline).
* inietta metadati di build in Info.plist: `OpenClawBuildTimestamp` (UTC) e `OpenClawGitCommit` (short hash), così che il pannello Informazioni possa mostrare build, git e canale debug/release.
* **Il packaging richiede Node 22+**: lo script esegue le build TS e la build della Control UI.
* legge `SIGN_IDENTITY` dall&#39;ambiente. Aggiungi `export SIGN_IDENTITY="Apple Development: Your Name (TEAMID)"` (o il tuo certificato Developer ID Application) al file rc della tua shell per firmare sempre con il tuo certificato. La firma ad-hoc richiede un opt-in esplicito tramite `ALLOW_ADHOC_SIGNING=1` o `SIGN_IDENTITY="-"` (non raccomandato per i test dei permessi).
* esegue un audit del Team ID dopo la firma e fallisce se qualsiasi Mach-O all&#39;interno del bundle dell&#39;app è firmato con un Team ID diverso. Imposta `SKIP_TEAM_ID_CHECK=1` per ignorare questo controllo.

<div id="usage">
  ## Utilizzo
</div>

```bash
# dalla root del repository
scripts/package-mac-app.sh               # seleziona automaticamente l'identità; errore se non ne viene trovata nessuna
SIGN_IDENTITY="Developer ID Application: Your Name" scripts/package-mac-app.sh   # certificato reale
ALLOW_ADHOC_SIGNING=1 scripts/package-mac-app.sh    # ad-hoc (i permessi non persisteranno)
SIGN_IDENTITY="-" scripts/package-mac-app.sh        # ad-hoc esplicito (stessa limitazione)
DISABLE_LIBRARY_VALIDATION=1 scripts/package-mac-app.sh   # workaround solo per sviluppo per mancata corrispondenza del Sparkle Team ID
```

<div id="ad-hoc-signing-note">
  ### Nota sulla firma ad-hoc
</div>

Quando firmi con `SIGN_IDENTITY="-"` (ad-hoc), lo script disabilita automaticamente l&#39;**Hardened Runtime** (`--options runtime`). Questo è necessario per evitare arresti anomali quando l&#39;app tenta di caricare framework integrati (come Sparkle) che non condividono lo stesso Team ID. Le firme ad-hoc interrompono anche la persistenza delle autorizzazioni TCC; vedi [macOS permissions](/it/platforms/mac/permissions) per i passaggi di ripristino.

<div id="build-metadata-for-about">
  ## Metadati di build per la sezione Informazioni
</div>

`package-mac-app.sh` inserisce nel bundle:

* `OpenClawBuildTimestamp`: data/ora in formato ISO8601 UTC al momento della creazione del pacchetto
* `OpenClawGitCommit`: hash git corto (oppure `unknown` se non disponibile)

La scheda Informazioni legge queste chiavi per mostrare versione, data di build, commit git e se si tratta di una build di debug (tramite `#if DEBUG`). Esegui di nuovo lo script di packaging per aggiornare questi valori dopo le modifiche al codice.

<div id="why">
  ## Perché
</div>

I permessi TCC sono legati al bundle identifier *e* alla firma del codice. Le build di debug non firmate, con UUID variabile, facevano sì che macOS “dimenticasse” le autorizzazioni dopo ogni ricompilazione. Firmare i binari (ad‑hoc per impostazione predefinita) e mantenere un bundle id/percorso fisso (`dist/OpenClaw.app`) preserva le autorizzazioni tra una build e l’altra, in linea con l’approccio usato da VibeTunnel.