---
title: Anthropic
summary: "Usa Anthropic Claude tramite chiavi API o setup-token in OpenClaw"
read_when:
  - Vuoi usare i modelli di Anthropic in OpenClaw
  - Vuoi usare setup-token invece delle chiavi API
---

<div id="anthropic-claude">
  # Anthropic (Claude)
</div>

Anthropic sviluppa la famiglia di modelli **Claude** e fornisce l&#39;accesso tramite API.
In OpenClaw puoi autenticarti con una chiave API o con un **setup-token**.

<div id="option-a-anthropic-api-key">
  ## Opzione A: chiave API Anthropic
</div>

**Ideale per:** accesso API standard e fatturazione in base all&#39;utilizzo.
Crea la tua chiave API nella console di Anthropic.

<div id="cli-setup">
  ### Configurazione della CLI
</div>

```bash
openclaw onboard
# scegli: chiave API Anthropic

# or non-interactive
openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
```

<div id="config-snippet">
  ### Snippet di configurazione
</div>

```json5
{
  env: { ANTHROPIC_API_KEY: "sk-ant-..." },
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="prompt-caching-anthropic-api">
  ## Prompt caching (Anthropic API)
</div>

OpenClaw **non** modifica il TTL di cache predefinito di Anthropic, a meno che tu non lo imposti esplicitamente.
Questa è una funzionalità **solo API**; l’autenticazione tramite abbonamento non applica le impostazioni di TTL.

Per impostare il TTL per ogni modello, usa `cacheControlTtl` nel campo `params` del modello:

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": {
          params: { cacheControlTtl: "5m" } // oppure "1h"
        }
      }
    }
  }
}
```

OpenClaw include il flag beta `extended-cache-ttl-2025-04-11` per le richieste all’API di Anthropic; mantienilo se sovrascrivi gli header del provider (vedi [/gateway/configuration](/it/gateway/configuration)).

<div id="option-b-claude-setup-token">
  ## Opzione B: setup-token di Claude
</div>

**Ideale per:** utilizzare il tuo abbonamento a Claude.

<div id="where-to-get-a-setup-token">
  ### Dove ottenere un setup-token
</div>

I setup-token vengono creati dalla **Claude Code CLI**, non dalla console Anthropic. Puoi eseguire questo comando su **qualsiasi macchina**:

```bash
claude setup-token
```

Incolla il token in OpenClaw (procedura guidata: **Anthropic token (incolla setup-token)**), oppure esegui il comando sull&#39;host che ospita il Gateway:

```bash
openclaw models auth setup-token --provider anthropic
```

Se hai generato il token su un&#39;altra macchina, incollalo qui:

```bash
openclaw models auth paste-token --provider anthropic
```

<div id="cli-setup">
  ### Configurazione della CLI
</div>

```bash
# Incolla un setup-token durante l'onboarding
openclaw onboard --auth-choice setup-token
```

### Esempio di configurazione

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="notes">
  ## Note
</div>

* Genera il setup-token con `claude setup-token` e incollalo, oppure esegui `openclaw models auth setup-token` sull&#39;host del Gateway.
* Se vedi “OAuth token refresh failed …” su un abbonamento Claude, esegui nuovamente l&#39;autenticazione con un setup-token. Vedi [/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription](/it/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription).
* I dettagli sull&#39;autenticazione e le regole di riutilizzo sono in [/concepts/oauth](/it/concepts/oauth).

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

**Errori 401 / token improvvisamente non valido**

* L&#39;autenticazione dell&#39;abbonamento Claude può scadere o essere revocata. Esegui di nuovo `claude setup-token`
  e incolla il token nell&#39;**host del Gateway**.
* Se l&#39;accesso Claude tramite CLI avviene su un&#39;altra macchina, usa
  `openclaw models auth paste-token --provider anthropic` sull&#39;host del Gateway.

**Nessuna chiave API trovata per il provider &quot;anthropic&quot;**

* L&#39;autenticazione è **per agente**. I nuovi agenti non ereditano le chiavi dell&#39;agente principale.
* Esegui di nuovo l&#39;onboarding per quell&#39;agente oppure incolla un setup-token / chiave API
  sull&#39;host del Gateway, quindi verifica con `openclaw models status`.

**Nessuna credenziale trovata per il profilo `anthropic:default`**

* Esegui `openclaw models status` per vedere quale profilo di autenticazione è attivo.
* Esegui di nuovo l&#39;onboarding oppure incolla un setup-token / chiave API per quel profilo.

**Nessun profilo di autenticazione disponibile (tutti in cooldown/non disponibili)**

* Controlla `openclaw models status --json` per `auth.unusableProfiles`.
* Aggiungi un altro profilo Anthropic oppure attendi che il periodo di cooldown termini.

Per ulteriori informazioni: [/gateway/troubleshooting](/it/gateway/troubleshooting) e [/help/faq](/it/help/faq).