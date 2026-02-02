---
title: OAuth
summary: "OAuth in OpenClaw: scambio di token, archiviazione e pattern multi-account"
read_when:
  - Vuoi comprendere il processo OAuth end-to-end in OpenClaw
  - Riscontri problemi di invalidazione dei token o di logout
  - Vuoi configurare flussi di autenticazione OAuth o con setup-token
  - Vuoi gestire più account o l'instradamento dei profili
---

<div id="oauth">
  # OAuth
</div>

OpenClaw supporta l’“autenticazione tramite abbonamento” via OAuth per i provider che la offrono (in particolare **OpenAI Codex (ChatGPT OAuth)**). Per le sottoscrizioni Anthropic, utilizza il flusso **setup-token**. Questa pagina spiega:

* come funziona lo **scambio di token** OAuth (PKCE)
* dove i token vengono **memorizzati** (e perché)
* come gestire **più account** (profili + override per singola sessione)

OpenClaw supporta anche **plugin dei provider** che includono i propri flussi OAuth o di chiave API.
Eseguili tramite:

```bash
openclaw models auth login --provider <id>
```

<div id="the-token-sink-why-it-exists">
  ## Il token sink (perché esiste)
</div>

I provider OAuth in genere generano un **nuovo refresh token** durante i flussi di login/refresh. Alcuni provider (o client OAuth) possono invalidare i vecchi refresh token quando ne viene emesso uno nuovo per lo stesso utente/app.

Sintomo pratico:

* effettui il login tramite OpenClaw *e* tramite Claude Code / Codex CLI → uno dei due in modo casuale viene “disconnesso” in un secondo momento

Per ridurre questo problema, OpenClaw tratta `auth-profiles.json` come un **token sink**:

* il runtime legge le credenziali da **un’unica posizione**
* possiamo mantenere più profili e instradarli in modo deterministico

<div id="storage-where-tokens-live">
  ## Storage (dove sono memorizzati i token)
</div>

I segreti sono memorizzati **per agente**:

* Profili di autenticazione (OAuth + API key): `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
* Cache di runtime (gestita automaticamente; non modificarla): `~/.openclaw/agents/<agentId>/agent/auth.json`

File legacy usato solo per l’import (ancora supportato, ma non è l’archivio principale):

* `~/.openclaw/credentials/oauth.json` (importato in `auth-profiles.json` al primo utilizzo)

Tutti i percorsi sopra indicati rispettano anche `$OPENCLAW_STATE_DIR` (sovrascrittura della directory di stato). Riferimento completo: [/gateway/configuration](/it/gateway/configuration#auth-storage-oauth--api-keys)

<div id="anthropic-setup-token-subscription-auth">
  ## Anthropic setup-token (autenticazione tramite abbonamento)
</div>

Esegui `claude setup-token` su qualsiasi macchina, quindi incolla il token in OpenClaw:

```bash
openclaw models auth setup-token --provider anthropic
```

Se hai generato il token altrove, incollalo qui manualmente:

```bash
openclaw models auth paste-token --provider anthropic
```

Verifica:

```bash
openclaw models status
```

<div id="oauth-exchange-how-login-works">
  ## Scambio OAuth (come funziona l&#39;accesso)
</div>

I flussi di accesso interattivi di OpenClaw sono implementati in `@mariozechner/pi-ai` e integrati nei wizard/comandi.

<div id="anthropic-claude-promax-setup-token">
  ### Anthropic (Claude Pro/Max) setup-token
</div>

Struttura del flusso:

1. esegui `claude setup-token`
2. incolla il token in OpenClaw
3. salvalo come profilo di autenticazione tramite token (senza refresh)

Il percorso guidato è `openclaw onboard` → opzione di autenticazione `setup-token` (Anthropic).

<div id="openai-codex-chatgpt-oauth">
  ### OpenAI Codex (ChatGPT OAuth)
</div>

Schema del flusso (PKCE):

1. genera il PKCE verifier/challenge + uno `state` casuale
2. apri `https://auth.openai.com/oauth/authorize?...`
3. prova a intercettare il callback su `http://127.0.0.1:1455/auth/callback`
4. se non è possibile effettuare il bind del callback (o sei remoto/headless), incolla l&#39;URL/codice di redirect
5. effettua lo scambio su `https://auth.openai.com/oauth/token`
6. estrai `accountId` dal token di accesso e memorizza `{ access, refresh, expires, accountId }`

Il percorso del wizard è `openclaw onboard` → opzione di autenticazione `openai-codex`.

<div id="refresh-expiry">
  ## Refresh + scadenza
</div>

I profili memorizzano un timestamp `expires`.

In fase di esecuzione:

* se `expires` è nel futuro → usa il token di accesso memorizzato
* se è scaduto → esegui il refresh (con un file lock) e sovrascrivi le credenziali memorizzate

La procedura di refresh è automatica; in genere non devi gestire i token manualmente.

<div id="multiple-accounts-profiles-routing">
  ## Più account (profili) + instradamento
</div>

Due pattern possibili:

<div id="1-preferred-separate-agents">
  ### 1) Consigliato: agenti separati
</div>

Se vuoi che “personale” e “lavoro” non interagiscano mai, usa agenti isolati (sessioni + credenziali + spazio di lavoro separati):

```bash
openclaw agents add work
openclaw agents add personal
```

Quindi configura l’autenticazione per ogni agente (wizard) e instrada le chat verso l’agente corretto.

<div id="2-advanced-multiple-profiles-in-one-agent">
  ### 2) Avanzato: più profili in un unico agente
</div>

`auth-profiles.json` supporta più ID di profilo per lo stesso provider.

Scegli quale profilo usare:

* globalmente tramite l&#39;ordinamento della configurazione (`auth.order`)
* per sessione tramite `/model ...@<profileId>`

Esempio (override della sessione):

* `/model Opus@anthropic:work`

Come vedere quali ID di profilo sono disponibili:

* `openclaw channels list --json` (mostra `auth[]`)

Documentazione correlata:

* [/concepts/model-failover](/it/concepts/model-failover) (regole di rotazione + cooldown)
* [/tools/slash-commands](/it/tools/slash-commands) (interfaccia dei comandi)