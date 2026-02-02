---
title: Ambiente
summary: "Da dove OpenClaw carica le variabili di ambiente e il relativo ordine di precedenza"
read_when:
  - Devi sapere quali variabili di ambiente vengono caricate e in quale ordine
  - Stai eseguendo il debug di chiavi API mancanti nel Gateway
  - Stai documentando l'autenticazione dei provider o gli ambienti di deployment
---

<div id="environment-variables">
  # Variabili d&#39;ambiente
</div>

OpenClaw legge le variabili d&#39;ambiente da più fonti. La regola è **non sovrascrivere mai i valori esistenti**.

<div id="precedence-highest-lowest">
  ## Precedenza (dalla più alta alla più bassa)
</div>

1. **Ambiente del processo** (ciò che il processo Gateway ha già ereditato dalla shell/daemon padre).
2. **`.env` nella directory di lavoro corrente** (comportamento predefinito di dotenv; non sovrascrive i valori esistenti).
3. **`.env` globale** in `~/.openclaw/.env` (alias `$OPENCLAW_STATE_DIR/.env`; non sovrascrive i valori esistenti).
4. **Blocco `env` di configurazione** in `~/.openclaw/openclaw.json` (applicato solo per le variabili ancora mancanti).
5. **Import opzionale dalla shell di login** (`env.shellEnv.enabled` oppure `OPENCLAW_LOAD_SHELL_ENV=1`), applicato solo per le chiavi attese ancora mancanti.

Se il file di configurazione manca del tutto, il punto 4 viene saltato; l’import dalla shell viene comunque eseguito se abilitato.

<div id="config-env-block">
  ## Blocco di configurazione `env`
</div>

Due modi equivalenti per definire variabili d&#39;ambiente inline (in entrambi i casi senza sovrascrivere quelle esistenti):

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-..."
    }
  }
}
```

<div id="shell-env-import">
  ## Importazione delle variabili d&#39;ambiente della shell
</div>

`env.shellEnv` esegue la shell di login e importa solo le variabili previste che risultano **mancanti**:

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000
    }
  }
}
```

Equivalenti come variabili di ambiente:

* `OPENCLAW_LOAD_SHELL_ENV=1`
* `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

<div id="env-var-substitution-in-config">
  ## Sostituzione delle variabili di ambiente nella configurazione
</div>

Puoi fare riferimento direttamente alle variabili di ambiente nei valori stringa della configurazione usando la sintassi `${VAR_NAME}`:

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}"
      }
    }
  }
}
```

Vedi [Configurazione: sostituzione delle variabili d&#39;ambiente](/it/gateway/configuration#env-var-substitution-in-config) per tutti i dettagli.

<div id="related">
  ## Contenuti correlati
</div>

* [Configurazione del Gateway](/it/gateway/configuration)
* [FAQ: variabili d&#39;ambiente e caricamento di .env](/it/help/faq#env-vars-and-env-loading)
* [Panoramica dei modelli](/it/concepts/models)