---
title: Migrazione
summary: "Trasferire (migrare) un'installazione di OpenClaw da una macchina a un'altra"
read_when:
  - Stai trasferendo OpenClaw su un nuovo laptop/server
  - Vuoi conservare le sessioni, l'autenticazione e gli accessi ai canali (WhatsApp, ecc.)
---

<div id="migrating-openclaw-to-a-new-machine">
  # Migrare OpenClaw su una nuova macchina
</div>

Questa guida descrive come migrare un Gateway OpenClaw da una macchina a un&#39;altra **senza ripetere l&#39;onboarding**.

Concettualmente, la migrazione è semplice:

* Copia la **directory di stato** (`$OPENCLAW_STATE_DIR`, valore predefinito: `~/.openclaw/`) — include configurazione, autenticazione, sessioni e stato dei canali.
* Copia il tuo **spazio di lavoro** (`~/.openclaw/workspace/` per impostazione predefinita) — include i file del tuo agente (memoria, prompt, ecc.).

Ma ci sono trappole comuni legate a **profili**, **permessi** e **copie parziali**.

<div id="before-you-start-what-you-are-migrating">
  ## Prima di iniziare (cosa stai per migrare)
</div>

<div id="1-identify-your-state-directory">
  ### 1) Identifica la tua directory di stato
</div>

La maggior parte delle installazioni usa il valore predefinito:

* **Directory di stato:** `~/.openclaw/`

Ma può essere diversa se usi:

* `--profile <name>` (spesso diventa `~/.openclaw-<profile>/`)
* `OPENCLAW_STATE_DIR=/some/path`

Se non ne sei sicuro, esegui questo sul **vecchio** computer:

```bash
openclaw status
```

Cerca le menzioni di `OPENCLAW_STATE_DIR` / profilo nell&#39;output. Se esegui più gateway, ripeti per ogni profilo.

<div id="2-identify-your-workspace">
  ### 2) Identifica il tuo spazio di lavoro
</div>

Valori predefiniti comuni:

* `~/.openclaw/workspace/` (spazio di lavoro consigliato)
* una cartella personalizzata che hai creato

Il tuo spazio di lavoro è la posizione in cui si trovano file come `MEMORY.md`, `USER.md` e `memory/*.md`.

<div id="3-understand-what-you-will-preserve">
  ### 3) Comprendere cosa verrà conservato
</div>

Se copi **sia** la directory di stato che lo spazio di lavoro, mantieni:

* Configurazione del Gateway (`openclaw.json`)
* Profili di autenticazione / chiavi API / token OAuth
* Cronologia delle sessioni + stato dell&#39;agente
* Stato dei canali (ad es. login/sessione WhatsApp)
* I file del tuo spazio di lavoro (memoria, note sulle abilità, ecc.)

Se copi **solo** lo spazio di lavoro (ad es. tramite Git), **non** verranno conservati:

* sessioni
* credenziali
* accessi ai canali

Questi si trovano in `$OPENCLAW_STATE_DIR`.

<div id="migration-steps-recommended">
  ## Passaggi per la migrazione (consigliati)
</div>

<div id="step-0-make-a-backup-old-machine">
  ### Passaggio 0 — Esegui un backup (vecchia macchina)
</div>

Sulla **vecchia** macchina, arresta prima il Gateway per evitare che i file cambino durante la copia:

```bash
openclaw gateway stop
```

(Facoltativo ma consigliato) archivia la directory di stato e lo spazio di lavoro:

```bash
# Modifica i percorsi se utilizzi un profilo o posizioni personalizzate
cd ~
tar -czf openclaw-state.tgz .openclaw

tar -czf openclaw-workspace.tgz .openclaw/workspace
```

Se hai più profili o directory di stato (ad esempio `~/.openclaw-main`, `~/.openclaw-work`), archivia ciascuna directory.

<div id="step-1-install-openclaw-on-the-new-machine">
  ### Passaggio 1 — Installa OpenClaw sulla nuova macchina
</div>

Sulla **nuova** macchina, installa la CLI (e Node, se necessario):

* Consulta: [Installazione](/it/install)

In questa fase, va bene se la procedura di onboarding crea una nuova directory `~/.openclaw/` — la sovrascriverai nel passaggio successivo.

<div id="step-2-copy-the-state-dir-workspace-to-the-new-machine">
  ### Passaggio 2 — Copia la directory di stato e lo spazio di lavoro sulla nuova macchina
</div>

Copia **entrambi**:

* `$OPENCLAW_STATE_DIR` (predefinito `~/.openclaw/`)
* il tuo spazio di lavoro (predefinito `~/.openclaw/workspace/`)

Approcci comuni:

* `scp` dei tarball e successiva estrazione
* `rsync -a` tramite SSH
* unità esterna

Dopo la copia, assicurati che:

* Le directory nascoste siano state incluse (ad es. `.openclaw/`)
* La proprietà dei file sia corretta per l’utente che esegue il Gateway

<div id="step-3-run-doctor-migrations-service-repair">
  ### Passaggio 3 — Esegui Doctor (migrazioni + riparazione dei servizi)
</div>

Sulla **nuova** macchina:

```bash
openclaw doctor
```

Doctor è il comando “sicuro e senza sorprese”. Ripara i servizi, applica le migrazioni di configurazione e avvisa in caso di incongruenze.

Quindi:

```bash
openclaw gateway restart
openclaw status
```

<div id="common-footguns-and-how-to-avoid-them">
  ## Errori comuni (e come evitarli)
</div>

<div id="footgun-profile-state-dir-mismatch">
  ### Trappola: mancata corrispondenza tra profilo e state-dir
</div>

Se eseguivi il vecchio gateway con un profilo (o `OPENCLAW_STATE_DIR`) e il nuovo gateway ne usa uno diverso, vedrai sintomi come:

* modifiche alla configurazione che non hanno effetto
* canali mancanti / disconnessi
* cronologia delle sessioni vuota

Soluzione: esegui il gateway/servizio usando **lo stesso** profilo/state-dir che hai migrato, quindi riesegui:

```bash
openclaw doctor
```

<div id="footgun-copying-only-openclawjson">
  ### Footgun: copiare solo `openclaw.json`
</div>

`openclaw.json` non è sufficiente. Molti provider salvano lo stato in:

* `$OPENCLAW_STATE_DIR/credentials/`
* `$OPENCLAW_STATE_DIR/agents/<agentId>/...`

Migra sempre l&#39;intera cartella `$OPENCLAW_STATE_DIR`.

<div id="footgun-permissions-ownership">
  ### Trappola: permessi / proprietà
</div>

Se hai copiato i file come root o hai cambiato utente, il Gateway potrebbe non riuscire a leggere credenziali/sessioni.

Soluzione: assicurati che la directory dello stato e lo spazio di lavoro siano di proprietà dell&#39;utente che esegue il Gateway.

<div id="footgun-migrating-between-remotelocal-modes">
  ### Trappola: migrare tra modalità remota/locale
</div>

* Se la tua UI (WebUI/TUI) punta a un **Gateway remoto**, l’host remoto gestisce l’archivio delle sessioni e lo spazio di lavoro.
* Migrare il tuo laptop non trasferirà lo stato del Gateway remoto.

Se sei in modalità remota, migra l’**host del Gateway**.

<div id="footgun-secrets-in-backups">
  ### Trappola pericolosa: segreti nei backup
</div>

`$OPENCLAW_STATE_DIR` contiene segreti (chiavi API, token OAuth, credenziali WhatsApp). Gestisci i backup come se fossero segreti di produzione:

* conservali cifrati
* evita di condividerli su canali non sicuri
* ruota le chiavi se sospetti una possibile esposizione

<div id="verification-checklist">
  ## Checklist di verifica
</div>

Sulla nuova macchina, verifica che:

* `openclaw status` mostri il Gateway in esecuzione
* I tuoi canali siano ancora connessi (ad es. WhatsApp non richieda un nuovo pairing)
* La dashboard si apra e mostri le sessioni esistenti
* I file del tuo spazio di lavoro (memory, configs) siano presenti

<div id="related">
  ## Contenuti correlati
</div>

* [Doctor](/it/gateway/doctor)
* [Risoluzione dei problemi del Gateway](/it/gateway/troubleshooting)
* [Dove memorizza i propri dati OpenClaw?](/it/help/faq#where-does-openclaw-store-its-data)