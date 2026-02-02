---
title: Autenticazione
summary: "Autenticazione dei modelli: OAuth, chiavi API e setup-token"
read_when:
  - Eseguire il debug dell'autenticazione dei modelli o della scadenza di OAuth
  - Documentare l'autenticazione o l'archiviazione delle credenziali
---

<div id="authentication">
  # Autenticazione
</div>

OpenClaw supporta OAuth e chiavi API per i provider di modelli. Per gli account
Anthropic, si consiglia di usare una **chiave API**. Per l&#39;accesso con
abbonamento a Claude, usa il token a lunga durata creato da `claude setup-token`.

Consulta [/concepts/oauth](/it/concepts/oauth) per il flusso OAuth completo e lo
schema di archiviazione.

<div id="recommended-anthropic-setup-api-key">
  ## Configurazione consigliata di Anthropic (API key)
</div>

Se usi direttamente Anthropic, utilizza una chiave API.

1. Crea una chiave API nella console Anthropic.
2. Impostala sull’**host del Gateway** (la macchina che esegue `openclaw gateway`).

```bash
export ANTHROPIC_API_KEY="..."
openclaw models status
```

3. Se il Gateway viene eseguito come servizio systemd/launchd, è preferibile inserire la chiave in
   `~/.openclaw/.env` in modo che il demone possa leggerla:

```bash
cat >> ~/.openclaw/.env <<'EOF'
ANTHROPIC_API_KEY=...
EOF
```

Quindi riavvia il demone (oppure riavvia il processo Gateway) e controlla nuovamente:

```bash
openclaw models status
openclaw doctor
```

Se preferisci non gestire direttamente le variabili d&#39;ambiente, la procedura guidata di onboarding può salvare
le chiavi API per l&#39;uso da parte del demone: `openclaw onboard`.

Consulta [Help](/it/help) per i dettagli sull&#39;ereditarietà delle variabili d&#39;ambiente (`env.shellEnv`,
`~/.openclaw/.env`, systemd/launchd).

<div id="anthropic-setup-token-subscription-auth">
  ## Anthropic: setup-token (autenticazione tramite abbonamento)
</div>

Per Anthropic, il metodo consigliato è una **chiave API**. Se stai usando un abbonamento
Claude, è supportato anche il flusso setup-token. Eseguilo sull’**host del Gateway**:

```bash
claude setup-token
```

Quindi incollalo in OpenClaw:

```bash
openclaw models auth setup-token --provider anthropic
```

Se il token è stato generato su un&#39;altra macchina, incollalo manualmente:

```bash
openclaw models auth paste-token --provider anthropic
```

Se vedi comparire un errore Anthropic del tipo:

```
Questa credenziale è autorizzata solo per l'uso con Claude Code e non può essere utilizzata per altre richieste API.
```

…utilizza invece una chiave API di Anthropic.

Inserimento manuale del token (qualsiasi provider; salva `auth-profiles.json` e aggiorna la configurazione):

```bash
openclaw models auth paste-token --provider anthropic
openclaw models auth paste-token --provider openrouter
```

Verifica automatizzabile (termina con codice `1` se scaduta/mancante, `2` se in scadenza):

```bash
openclaw models status --check
```

Gli script operativi opzionali (systemd/Termux) sono documentati qui:
[/automation/auth-monitoring](/it/automation/auth-monitoring)

> `claude setup-token` richiede un TTY interattivo.

<div id="checking-model-auth-status">
  ## Verifica dello stato di autenticazione del modello
</div>

```bash
openclaw models status
openclaw doctor
```

<div id="controlling-which-credential-is-used">
  ## Controllare quale credenziale usare
</div>

<div id="per-session-chat-command">
  ### Per sessione (comando chat)
</div>

Usa `/model <alias-or-id>@<profileId>` per bloccare una specifica credenziale di provider per la sessione corrente (ID profilo di esempio: `anthropic:default`, `anthropic:work`).

Usa `/model` (o `/model list`) per un selettore compatto; usa `/model status` per la visualizzazione completa (candidati + prossimo profilo di autenticazione, più dettagli sull&#39;endpoint del provider se configurato).

<div id="per-agent-cli-override">
  ### Per singolo agente (override tramite CLI)
</div>

Imposta una sovrascrittura esplicita dell&#39;ordine dei profili di autenticazione per un agente (salvata nel file `auth-profiles.json` di quell’agente):

```bash
openclaw models auth order get --provider anthropic
openclaw models auth order set --provider anthropic anthropic:default
openclaw models auth order clear --provider anthropic
```

Usa `--agent &lt;id&gt;` per selezionare uno specifico agente; ometti l’opzione per usare l’agente predefinito configurato.

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

<div id="no-credentials-found">
  ### “Nessuna credenziale trovata”
</div>

Se manca il profilo token Anthropic, esegui `claude setup-token` sull’**host del Gateway**, quindi controlla nuovamente:

```bash
openclaw models status
```

<div id="token-expiringexpired">
  ### Token in scadenza/scaduto
</div>

Esegui `openclaw models status` per verificare quale profilo sta per scadere. Se il profilo
non compare, riesegui `claude setup-token` e incolla nuovamente il token.

<div id="requirements">
  ## Requisiti
</div>

* Abbonamento Claude Max o Pro (per `claude setup-token`)
* Claude Code CLI installata (con il comando `claude` disponibile)