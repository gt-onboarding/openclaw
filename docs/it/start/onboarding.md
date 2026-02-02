---
title: Onboarding
summary: "Flusso di onboarding al primo avvio per OpenClaw (app macOS)"
read_when:
  - Durante la progettazione dell'assistente di onboarding per macOS
  - Durante l'implementazione della configurazione di autenticazione o identità
---

<div id="onboarding-macos-app">
  # Onboarding (app per macOS)
</div>

Questo documento descrive il flusso di onboarding **attuale** al primo avvio. L&#39;obiettivo è offrire un&#39;esperienza fluida del “giorno 0”: scegliere dove eseguire il Gateway, configurare l&#39;autenticazione, eseguire la procedura guidata e lasciare che l&#39;agente si inizializzi autonomamente.

<div id="page-order-current">
  ## Ordine delle pagine (attuale)
</div>

1. Benvenuto + avviso di sicurezza
2. **Selezione del Gateway** (Locale / Remoto / Configura in un secondo momento)
3. **Autenticazione (Anthropic OAuth)** — solo in locale
4. **Procedura guidata di configurazione** (gestita dal Gateway)
5. **Autorizzazioni** (prompt TCC)
6. **CLI** (opzionale)
7. **Chat di onboarding** (sessione dedicata)
8. Tutto pronto

<div id="1-local-vs-remote">
  ## 1) Locale vs remoto
</div>

Dove viene eseguito il **Gateway**?

* **Locale (questo Mac):** l&#39;onboarding può eseguire i flussi OAuth e scrivere le credenziali
  in locale.
* **Remoto (via SSH/Tailnet):** l&#39;onboarding **non** esegue OAuth in locale;
  le credenziali devono esistere sull&#39;host del Gateway.
* **Configura più tardi:** salta la configurazione e lascia l&#39;app non configurata.

Suggerimento per l&#39;autenticazione del Gateway:

* La procedura guidata ora genera un **token** anche per il loopback, quindi i client WS locali devono autenticarsi.
* Se disabiliti l&#39;autenticazione, qualsiasi processo locale può connettersi; usala solo su macchine completamente attendibili.
* Usa un **token** per l&#39;accesso multi‑macchina o per connessioni non‑loopback.

<div id="2-local-only-auth-anthropic-oauth">
  ## 2) Autenticazione solo in locale (Anthropic OAuth)
</div>

L&#39;app macOS supporta Anthropic OAuth (Claude Pro/Max). Il flusso è il seguente:

* Apre il browser per OAuth (PKCE)
* Chiede all&#39;utente di incollare il valore `code#state`
* Scrive le credenziali in `~/.openclaw/credentials/oauth.json`

Altri provider (OpenAI, API personalizzate) sono al momento configurati tramite variabili
d&#39;ambiente o file di configurazione.

<div id="3-setup-wizard-gatewaydriven">
  ## 3) Procedura guidata di configurazione (gestita dal Gateway)
</div>

L&#39;app può eseguire la stessa procedura guidata di configurazione disponibile nella CLI. In questo modo l&#39;onboarding rimane sincronizzato con il comportamento del Gateway ed evita di duplicare la logica in SwiftUI.

<div id="4-permissions">
  ## 4) Autorizzazioni
</div>

Per la procedura di onboarding sono necessarie le seguenti autorizzazioni TCC:

* Notifiche
* Accessibilità
* Registrazione schermo
* Microfono / Riconoscimento vocale
* Automazione (AppleScript)

<div id="5-cli-optional">
  ## 5) CLI (facoltativo)
</div>

L&#39;app può installare la CLI globale `openclaw` tramite npm/pnpm, così che i flussi di lavoro da terminale e le attività launchd siano subito pronti all&#39;uso.

<div id="6-onboarding-chat-dedicated-session">
  ## 6) Chat di onboarding (sessione dedicata)
</div>

Dopo la configurazione, l&#39;app apre una sessione di chat di onboarding dedicata, in modo che l&#39;agente possa
presentarsi e guidarti nei passaggi successivi. In questo modo le istruzioni del primo avvio rimangono separate
dalla tua normale conversazione.

<div id="agent-bootstrap-ritual">
  ## Rituale di bootstrap dell&#39;Agente
</div>

Alla prima esecuzione dell&#39;agente, OpenClaw esegue il bootstrap di uno spazio di lavoro (predefinito `~/.openclaw/workspace`):

* Inizializza `AGENTS.md`, `BOOTSTRAP.md`, `IDENTITY.md`, `USER.md`
* Esegue un breve rituale di domande e risposte (una domanda alla volta)
* Scrive identità e preferenze in `IDENTITY.md`, `USER.md`, `SOUL.md`
* Rimuove `BOOTSTRAP.md` al termine, in modo che venga eseguito una sola volta

<div id="optional-gmail-hooks-manual">
  ## Facoltativo: hook Gmail (manuale)
</div>

Attualmente la configurazione di Gmail Pub/Sub va eseguita manualmente. Usa:

```bash
openclaw webhooks gmail setup --account you@gmail.com
```

Vedi [/automation/gmail-pubsub](/it/automation/gmail-pubsub) per maggiori dettagli.

<div id="remote-mode-notes">
  ## Note sulla modalità remota
</div>

Quando il Gateway viene eseguito su un&#39;altra macchina, le credenziali e i file dello spazio di lavoro risiedono
**su quell&#39;host**. Se ti serve OAuth in modalità remota, crea:

* `~/.openclaw/credentials/oauth.json`
* `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

sull&#39;host del Gateway.