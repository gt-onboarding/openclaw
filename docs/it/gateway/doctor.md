---
title: Doctor
summary: "Comando Doctor: controlli di integrità, migrazioni di configurazione e azioni correttive"
read_when:
  - Aggiunta o modifica delle migrazioni di Doctor
  - Introduzione di modifiche di configurazione incompatibili con le versioni precedenti
---

<div id="doctor">
  # Doctor
</div>

`openclaw doctor` è lo strumento di riparazione e migrazione per OpenClaw. Corregge configurazioni e stati obsoleti, verifica lo stato di salute del sistema e fornisce istruzioni operative per la riparazione.

<div id="quick-start">
  ## Guida rapida
</div>

```bash
openclaw doctor
```

<div id="headless-automation">
  ### Modalità headless / automazione
</div>

```bash
openclaw doctor --yes
```

Accetta i valori predefiniti senza chiedere conferma (inclusi i passaggi di riavvio/servizio/riparazione della sandbox, se applicabili).

```bash
openclaw doctor --repair
```

Applica automaticamente le correzioni consigliate, senza richiedere conferma (correzioni + riavvii quando è sicuro farlo).

```bash
openclaw doctor --repair --force
```

Applica anche interventi di riparazione aggressivi (sovrascrive le configurazioni personalizzate del supervisor).

```bash
openclaw doctor --non-interactive
```

Esegue senza prompt interattivi e applica solo migrazioni sicure (normalizzazione della configurazione + spostamenti dello stato su disco). Ignora le azioni di riavvio/servizio/sandbox che richiedono conferma umana.
Le migrazioni dello stato legacy vengono eseguite automaticamente quando vengono rilevate.

```bash
openclaw doctor --deep
```

Scansiona i servizi di sistema alla ricerca di installazioni aggiuntive del Gateway (launchd/systemd/schtasks).

Se vuoi rivedere le modifiche prima di salvarle, apri prima il file di configurazione:

```bash
cat ~/.openclaw/openclaw.json
```

<div id="what-it-does-summary">
  ## Cosa fa (riepilogo)
</div>

* Aggiornamento pre-flight opzionale per installazioni git (solo interattive).
* Verifica dell’aggiornamento del protocollo UI (ricostruisce la Control UI quando lo schema del protocollo è più recente).
* Health check + prompt di riavvio.
* Riepilogo dello stato delle abilità (idonee/mancanti/bloccate).
* Normalizzazione della configurazione per valori legacy.
* Avvisi di override del provider OpenCode Zen (`models.providers.opencode`).
* Migrazione dello stato legacy su disco (sessioni/directory agente/auth WhatsApp).
* Verifiche di integrità dello stato e dei permessi (sessioni, trascrizioni, directory di stato).
* Verifiche dei permessi dei file di configurazione (chmod 600) quando eseguito in locale.
* Stato di salute dell&#39;autenticazione dei modelli: controlla la scadenza OAuth, può aggiornare i token in scadenza e segnala gli stati di cooldown/disabilitato dei profili di autenticazione.
* Rilevamento di directory di spazio di lavoro aggiuntive (`~/openclaw`).
* Ripristino dell&#39;immagine sandbox quando il sandboxing è abilitato.
* Migrazione dei servizi legacy e rilevamento di Gateway aggiuntivi.
* Verifiche del runtime del Gateway (servizio installato ma non in esecuzione; etichetta launchd in cache).
* Avvisi sullo stato dei canali (rilevati dal Gateway in esecuzione).
* Audit della configurazione del supervisore (launchd/systemd/schtasks) con riparazione opzionale.
* Verifiche delle best practice del runtime del Gateway (Node vs Bun, percorsi del version manager).
* Diagnostica sulle collisioni di porta del Gateway (predefinita `18789`).
* Avvisi di sicurezza per policy di DM `open`.
* Avvisi di autenticazione del Gateway quando `gateway.auth.token` non è impostato (modalità locale; offre la generazione del token).
* Verifica di systemd linger su Linux.
* Controlli sulle installazioni da sorgente (mismatch dello spazio di lavoro pnpm, asset UI mancanti, binario tsx mancante).
* Scrive la configurazione aggiornata + i metadati del wizard.

<div id="detailed-behavior-and-rationale">
  ## Funzionamento dettagliato e motivazioni
</div>

<div id="0-optional-update-git-installs">
  ### 0) Aggiornamento opzionale (installazioni da git)
</div>

Se si tratta di una checkout Git e `doctor` viene eseguito in modalità interattiva, verrà proposta
l’esecuzione di un aggiornamento (fetch/rebase/build) prima di avviare `doctor`.

<div id="1-config-normalization">
  ### 1) Normalizzazione della configurazione
</div>

Se la configurazione contiene strutture di valori legacy (ad esempio `messages.ackReaction`
senza un override specifico per canale), doctor le normalizza nello schema
corrente.

<div id="2-legacy-config-key-migrations">
  ### 2) Migrazioni delle chiavi di configurazione legacy
</div>

Quando la configurazione contiene chiavi deprecate, gli altri comandi non vengono eseguiti e ti chiedono
di eseguire `openclaw doctor`.

Doctor eseguirà le seguenti operazioni:

* Spiega quali chiavi legacy sono state trovate.
* Mostra la migrazione che ha applicato.
* Riscrive `~/.openclaw/openclaw.json` con lo schema aggiornato.

Il Gateway esegue inoltre automaticamente le migrazioni di doctor all&#39;avvio quando rileva
un formato di configurazione legacy, così le configurazioni obsolete vengono ripristinate senza intervento manuale.

Migrazioni attuali:

* `routing.allowFrom` → `channels.whatsapp.allowFrom`
* `routing.groupChat.requireMention` → `channels.whatsapp/telegram/imessage.groups."*".requireMention`
* `routing.groupChat.historyLimit` → `messages.groupChat.historyLimit`
* `routing.groupChat.mentionPatterns` → `messages.groupChat.mentionPatterns`
* `routing.queue` → `messages.queue`
* `routing.bindings` → top-level `bindings`
* `routing.agents`/`routing.defaultAgentId` → `agents.list` + `agents.list[].default`
* `routing.agentToAgent` → `tools.agentToAgent`
* `routing.transcribeAudio` → `tools.media.audio.models`
* `bindings[].match.accountID` → `bindings[].match.accountId`
* `identity` → `agents.list[].identity`
* `agent.*` → `agents.defaults` + `tools.*` (tools/elevated/exec/sandbox/subagents)
* `agent.model`/`allowedModels`/`modelAliases`/`modelFallbacks`/`imageModelFallbacks`
  → `agents.defaults.models` + `agents.defaults.model.primary/fallbacks` + `agents.defaults.imageModel.primary/fallbacks`

<div id="2b-opencode-zen-provider-overrides">
  ### 2b) Override del provider OpenCode Zen
</div>

Se hai aggiunto manualmente `models.providers.opencode` (o `opencode-zen`),
questo sostituisce il catalogo OpenCode Zen integrato fornito da `@mariozechner/pi-ai`. Questo può forzare tutti i modelli su un’unica API oppure azzerare i costi. Doctor ti avvisa così puoi rimuovere l’override e ripristinare l’instradamento API per singolo modello e i relativi costi.

<div id="3-legacy-state-migrations-disk-layout">
  ### 3) Migrazioni dello stato legacy (layout su disco)
</div>

Doctor può migrare i layout legacy presenti su disco nella struttura corrente:

* Archivio delle sessioni + trascrizioni:
  * da `~/.openclaw/sessions/` a `~/.openclaw/agents/<agentId>/sessions/`
* Directory dell&#39;agente:
  * da `~/.openclaw/agent/` a `~/.openclaw/agents/<agentId>/agent/`
* Stato di autenticazione WhatsApp (Baileys):
  * dal percorso legacy `~/.openclaw/credentials/*.json` (eccetto `oauth.json`)
  * a `~/.openclaw/credentials/whatsapp/<accountId>/...` (ID account predefinito: `default`)

Queste migrazioni sono best-effort e idempotenti; Doctor emetterà avvisi se
lascia eventuali cartelle legacy come backup. Il Gateway/CLI esegue inoltre automaticamente la migrazione
delle sessioni legacy e della directory dell&#39;agente all&#39;avvio, in modo che cronologia/auth/modelli finiscano nel
percorso specifico per ciascun agente senza dover eseguire manualmente Doctor. La migrazione dell&#39;autenticazione WhatsApp è volutamente
eseguita solo tramite `openclaw doctor`.

<div id="4-state-integrity-checks-session-persistence-routing-and-safety">
  ### 4) Controlli di integrità dello stato (persistenza delle sessioni, instradamento e sicurezza)
</div>

La directory dello stato è il tronco encefalico operativo. Se scompare, perdi
sessioni, credenziali, log e configurazione (a meno che tu non abbia backup altrove).

Controlli eseguiti da Doctor:

* **Directory dello stato mancante**: avvisa di una perdita catastrofica dello stato, suggerisce di ricreare
  la directory e ti ricorda che non può recuperare i dati mancanti.
* **Permessi sulla directory dello stato**: verifica che sia scrivibile; offre di riparare i permessi
  (ed emette un suggerimento `chown` quando viene rilevata una mancata corrispondenza tra proprietario e gruppo).
* **Directory delle sessioni mancanti**: `sessions/` e la directory dello store delle sessioni sono
  necessarie per conservare la cronologia ed evitare errori `ENOENT`.
* **Mancata corrispondenza dei transcript**: avvisa quando le voci recenti di una sessione hanno file
  di transcript mancanti.
* **Sessione principale “JSONL a 1 riga”**: segnala quando il transcript principale ha una sola
  riga (la cronologia non si sta accumulando).
* **Directory dello stato multiple**: avvisa quando esistono più cartelle `~/.openclaw` tra
  directory home diverse o quando `OPENCLAW_STATE_DIR` punta altrove (la cronologia può
  dividersi tra installazioni).
* **Promemoria modalità remota**: se `gateway.mode=remote`, Doctor ti ricorda di eseguirlo
  sull&#39;host remoto (lo stato risiede lì).
* **Permessi sul file di configurazione**: avvisa se `~/.openclaw/openclaw.json` è
  leggibile da gruppo/altri utenti e offre di restringere a `600`.

<div id="5-model-auth-health-oauth-expiry">
  ### 5) Integrità dell&#39;autenticazione del modello (scadenza OAuth)
</div>

Doctor ispeziona i profili OAuth nello store di autenticazione, avvisa quando i
token stanno per scadere/sono scaduti e può aggiornarli quando è sicuro farlo. Se
il profilo Anthropic Claude Code è obsoleto, suggerisce di eseguire
`claude setup-token` (o di incollare un setup-token).
Le richieste di aggiornamento compaiono solo in esecuzione interattiva (TTY);
`--non-interactive` salta i tentativi di aggiornamento.

Doctor segnala anche i profili di autenticazione temporaneamente inutilizzabili a causa di:

* brevi periodi di cooldown (rate limit/timeout/errori di autenticazione)
* disabilitazioni più lunghe (problemi di fatturazione/credito)

<div id="6-hooks-model-validation">
  ### 6) Validazione del modello per gli hook
</div>

Se `hooks.gmail.model` è configurato, doctor convalida il riferimento al modello rispetto al
catalogo e alla lista di autorizzati e avvisa quando non è risolvibile o non è consentito.

<div id="7-sandbox-image-repair">
  ### 7) Ripristino delle immagini sandbox
</div>

Quando la sandbox è abilitata, doctor controlla le immagini Docker e propone di crearne una nuova
o passare a nomi legacy se l&#39;immagine corrente non è disponibile.

<div id="8-gateway-service-migrations-and-cleanup-hints">
  ### 8) Migrazioni del servizio Gateway e suggerimenti di pulizia
</div>

Doctor rileva servizi gateway legacy (launchd/systemd/schtasks) e
offre di rimuoverli e installare il servizio OpenClaw usando la porta
Gateway corrente. Può anche eseguire una scansione alla ricerca di servizi aggiuntivi
simili al Gateway e stampare suggerimenti di pulizia.
I servizi Gateway OpenClaw con nome di profilo sono considerati di prima classe e non
vengono contrassegnati come &quot;extra&quot;.

<div id="9-security-warnings">
  ### 9) Avvisi di sicurezza
</div>

Doctor emette avvisi quando un provider è configurato su open per i messaggi diretti (DM) senza una lista di autorizzati (cioè accetta messaggi da chiunque senza restrizioni), o
quando una policy è configurata in modo pericoloso.

<div id="10-systemd-linger-linux">
  ### 10) systemd linger (Linux)
</div>

Quando viene eseguito come servizio utente systemd, doctor verifica che il lingering sia abilitato affinché il Gateway rimanga in esecuzione dopo il logout.

<div id="11-skills-status">
  ### 11) Stato delle abilità
</div>

Doctor mostra un riepilogo sintetico delle abilità idonee/mancanti/bloccate per lo
spazio di lavoro corrente.

<div id="12-gateway-auth-checks-local-token">
  ### 12) Verifiche di autenticazione del Gateway (token locale)
</div>

Doctor segnala quando `gateway.auth` manca su un Gateway locale e propone
di generare un token. Usa `openclaw doctor --generate-gateway-token` per
forzare la creazione del token nelle automazioni.

<div id="13-gateway-health-check-restart">
  ### 13) Verifica stato del Gateway + riavvio
</div>

Doctor esegue un controllo dello stato e propone di riavviare il Gateway quando
rileva che non è in uno stato di salute corretto.

<div id="14-channel-status-warnings">
  ### 14) Avvisi sullo stato dei canali
</div>

Se il Gateway funziona correttamente, `doctor` esegue un controllo sullo stato dei canali e
riporta avvisi con i relativi suggerimenti di correzione.

<div id="15-supervisor-config-audit-repair">
  ### 15) Audit e riparazione della configurazione del supervisor
</div>

Doctor verifica la configurazione del supervisor installata (launchd/systemd/schtasks) per
individuare impostazioni predefinite mancanti o obsolete (ad esempio le dipendenze `network-online` di systemd e il
ritardo di riavvio). Quando trova una discrepanza, raccomanda un aggiornamento e può
riscrivere il file di servizio/attività in base alle impostazioni predefinite correnti.

Note:

* `openclaw doctor` chiede conferma prima di riscrivere la configurazione del supervisor.
* `openclaw doctor --yes` accetta i prompt di riparazione predefiniti.
* `openclaw doctor --repair` applica le correzioni consigliate senza prompt.
* `openclaw doctor --repair --force` sovrascrive le configurazioni personalizzate del supervisor.
* Puoi sempre forzare una riscrittura completa tramite `openclaw gateway install --force`.

<div id="16-gateway-runtime-port-diagnostics">
  ### 16) Diagnostica del runtime del Gateway e della porta
</div>

Doctor ispeziona il runtime del servizio (PID, ultimo codice di uscita) e avvisa quando
il servizio è installato ma non è effettivamente in esecuzione. Verifica inoltre eventuali conflitti
sulla porta del Gateway (predefinita `18789`) e segnala le cause probabili (Gateway già
in esecuzione, tunnel SSH).

<div id="17-gateway-runtime-best-practices">
  ### 17) Best practice per il runtime del Gateway
</div>

Doctor emette un avviso quando il servizio Gateway viene eseguito su Bun o su un percorso Node gestito da un gestore di versioni
(`nvm`, `fnm`, `volta`, `asdf`, ecc.). I canali WhatsApp e Telegram richiedono Node
e i percorsi gestiti da un gestore di versioni possono smettere di funzionare dopo gli aggiornamenti perché il servizio non
esegue gli script di inizializzazione della shell. Quando possibile, Doctor propone di migrare a un’installazione di Node a livello di sistema
(Homebrew/apt/choco).

<div id="18-config-write-wizard-metadata">
  ### 18) Scrittura della configurazione + metadati del wizard
</div>

Doctor salva tutte le modifiche alla configurazione e aggiunge i metadati del wizard per registrare l&#39;esecuzione di Doctor.

<div id="19-workspace-tips-backup-memory-system">
  ### 19) Suggerimenti per lo spazio di lavoro (backup + sistema di memoria)
</div>

Doctor suggerisce un sistema di memoria per lo spazio di lavoro quando non è presente e mostra un suggerimento per il backup
se lo spazio di lavoro non è già tracciato con Git.

Consulta [/concepts/agent-workspace](/it/concepts/agent-workspace) per una guida completa alla
struttura dello spazio di lavoro e al backup tramite Git (consigliato l&#39;uso di repository privati GitHub o GitLab).