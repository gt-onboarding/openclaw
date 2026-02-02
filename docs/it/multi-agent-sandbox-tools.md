---
summary: "Sandbox per-agente + restrizioni sugli strumenti, priorità ed esempi"
title: Sandbox multi-agente e strumenti
read_when: "Vuoi una sandbox per-agente o policy di allow/deny sugli strumenti per-agente in un Gateway multi-agente."
status: active
---

<div id="multi-agent-sandbox-tools-configuration">
  # Configurazione della sandbox e degli strumenti per ambienti multi-agente
</div>

<div id="overview">
  ## Panoramica
</div>

Ogni agente in una configurazione multi‑agente può ora avere il proprio:

* **Configurazione sandbox** (`agents.list[].sandbox` ha la precedenza su `agents.defaults.sandbox`)
* **Restrizioni sugli strumenti** (`tools.allow` / `tools.deny`, più `agents.list[].tools`)

Questo ti permette di eseguire più agenti con profili di sicurezza diversi:

* Assistente personale con accesso completo
* Agenti per famiglia/lavoro con strumenti limitati
* Agenti pubblici in sandbox

`setupCommand` va sotto `sandbox.docker` (globale o per agente) e viene eseguito una volta
quando il contenitore viene creato.

L&#39;autenticazione è per agente: ogni agente legge dal proprio archivio di autenticazione `agentDir` situato in:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Le credenziali **non** sono condivise tra gli agenti. Non riutilizzare mai `agentDir` fra agenti diversi.
Se vuoi condividere le credenziali, copia `auth-profiles.json` nell&#39;`agentDir` dell&#39;altro agente.

Per il comportamento del sandbox in fase di esecuzione, vedi [Sandboxing](/it/gateway/sandboxing).
Per eseguire il debug del “perché è bloccato?”, vedi [Sandbox vs Tool Policy vs Elevated](/it/gateway/sandbox-vs-tool-policy-vs-elevated) e `openclaw sandbox explain`.

***

<div id="configuration-examples">
  ## Esempi di configurazione
</div>

<div id="example-1-personal-restricted-family-agent">
  ### Esempio 1: Agente personale + familiare con accesso limitato
</div>

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "Personal Assistant",
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "family",
        "name": "Family Bot",
        "workspace": "~/.openclaw/workspace-family",
        "sandbox": {
          "mode": "all",
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch", "process", "browser"]
        }
      }
    ]
  },
  "bindings": [
    {
      "agentId": "family",
      "match": {
        "provider": "whatsapp",
        "accountId": "*",
        "peer": {
          "kind": "group",
          "id": "120363424282127706@g.us"
        }
      }
    }
  ]
}
```

**Risultato:**

* agente `main`: Viene eseguito sull&#39;host, accesso completo agli strumenti
* agente `family`: Viene eseguito in Docker (un container per agente), solo lo strumento `read`

***

<div id="example-2-work-agent-with-shared-sandbox">
  ### Esempio 2: Agente Work con sandbox condivisa
</div>

```json
{
  "agents": {
    "list": [
      {
        "id": "personal",
        "workspace": "~/.openclaw/workspace-personal",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "work",
        "workspace": "~/.openclaw/workspace-work",
        "sandbox": {
          "mode": "all",
          "scope": "shared",
          "workspaceRoot": "/tmp/work-sandboxes"
        },
        "tools": {
          "allow": ["read", "write", "apply_patch", "exec"],
          "deny": ["browser", "gateway", "discord"]
        }
      }
    ]
  }
}
```

***

<div id="example-2b-global-coding-profile-messaging-only-agent">
  ### Esempio 2b: Profilo di codifica globale + agente solo per la messaggistica
</div>

```json
{
  "tools": { "profile": "coding" },
  "agents": {
    "list": [
      {
        "id": "support",
        "tools": { "profile": "messaging", "allow": ["slack"] }
      }
    ]
  }
}
```

**Risultato:**

* gli agenti predefiniti dispongono degli strumenti di sviluppo
* l&#39;agente `support` gestisce solo la messaggistica (+ strumento Slack)

***

<div id="example-3-different-sandbox-modes-per-agent">
  ### Esempio 3: modalità sandbox diverse per agente
</div>

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",  // Global default
        "scope": "session"
      }
    },
    "list": [
      {
        "id": "main",
        "workspace": "~/.openclaw/workspace",
        "sandbox": {
          "mode": "off"  // Override: main never sandboxed
        }
      },
      {
        "id": "public",
        "workspace": "~/.openclaw/workspace-public",
        "sandbox": {
          "mode": "all",  // Override: public viene sempre eseguito in sandbox
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch"]
        }
      }
    ]
  }
}
```

***

<div id="configuration-precedence">
  ## Precedenza delle configurazioni
</div>

Quando esistono sia configurazioni globali (`agents.defaults.*`) sia configurazioni specifiche per l&#39;agente (`agents.list[].*`):

<div id="sandbox-config">
  ### Configurazione della sandbox
</div>

Le impostazioni specifiche dell&#39;agente hanno la precedenza su quelle globali:

```
agents.list[].sandbox.mode > agents.defaults.sandbox.mode
agents.list[].sandbox.scope > agents.defaults.sandbox.scope
agents.list[].sandbox.workspaceRoot > agents.defaults.sandbox.workspaceRoot
agents.list[].sandbox.workspaceAccess > agents.defaults.sandbox.workspaceAccess
agents.list[].sandbox.docker.* > agents.defaults.sandbox.docker.*
agents.list[].sandbox.browser.* > agents.defaults.sandbox.browser.*
agents.list[].sandbox.prune.* > agents.defaults.sandbox.prune.*
```

**Note:**

* `agents.list[].sandbox.{docker,browser,prune}.*` ha la precedenza su `agents.defaults.sandbox.{docker,browser,prune}.*` per quell&#39;agente (ignorato quando lo scope della sandbox è `"shared"`).

<div id="tool-restrictions">
  ### Restrizioni degli strumenti
</div>

L&#39;ordine di applicazione dei filtri è:

1. **Profilo degli strumenti** (`tools.profile` oppure `agents.list[].tools.profile`)
2. **Profilo degli strumenti del provider** (`tools.byProvider[provider].profile` oppure `agents.list[].tools.byProvider[provider].profile`)
3. **Policy globale degli strumenti** (`tools.allow` / `tools.deny`)
4. **Policy degli strumenti del provider** (`tools.byProvider[provider].allow/deny`)
5. **Policy degli strumenti specifica dell&#39;agente** (`agents.list[].tools.allow/deny`)
6. **Policy del provider specifica dell&#39;agente** (`agents.list[].tools.byProvider[provider].allow/deny`)
7. **Policy degli strumenti della sandbox** (`tools.sandbox.tools` oppure `agents.list[].tools.sandbox.tools`)
8. **Policy degli strumenti del sottoagente** (`tools.subagents.tools`, se applicabile)

Ogni livello può restringere ulteriormente gli strumenti, ma non può riabilitare strumenti negati dai livelli precedenti.
Se `agents.list[].tools.sandbox.tools` è impostato, sostituisce `tools.sandbox.tools` per quell&#39;agente.
Se `agents.list[].tools.profile` è impostato, sovrascrive `tools.profile` per quell&#39;agente.
Le chiavi degli strumenti del provider accettano sia `provider` (ad es. `google-antigravity`) sia `provider/model` (ad es. `openai/gpt-5.2`).

<div id="tool-groups-shorthands">
  ### Gruppi di strumenti (abbreviazioni)
</div>

Le policy degli strumenti (globali, degli agenti, della sandbox) supportano voci `group:*` che si espandono in più strumenti specifici:

* `group:runtime`: `exec`, `bash`, `process`
* `group:fs`: `read`, `write`, `edit`, `apply_patch`
* `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory`: `memory_search`, `memory_get`
* `group:ui`: `browser`, `canvas`
* `group:automation`: `cron`, `gateway`
* `group:messaging`: `message`
* `group:nodes`: `nodes`
* `group:openclaw`: tutti gli strumenti OpenClaw integrati (esclude i plugin dei provider)

<div id="elevated-mode">
  ### Modalità con privilegi elevati
</div>

`tools.elevated` è la base globale (lista di autorizzati basata sul mittente). `agents.list[].tools.elevated` può limitare ulteriormente i privilegi elevati per agenti specifici (entrambi devono consentire).

Strategie di mitigazione:

* Nega `exec` per gli agenti non attendibili (`agents.list[].tools.deny: ["exec"]`)
* Evita di autorizzare mittenti che instradano verso agenti con restrizioni
* Disabilita i privilegi elevati globalmente (`tools.elevated.enabled: false`) se vuoi esecuzione solo in sandbox
* Disabilita i privilegi elevati per agente (`agents.list[].tools.elevated.enabled: false`) per profili sensibili

***

<div id="migration-from-single-agent">
  ## Migrazione da agente singolo
</div>

**Prima (agente singolo):**

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "sandbox": {
        "mode": "non-main"
      }
    }
  },
  "tools": {
    "sandbox": {
      "tools": {
        "allow": ["read", "write", "apply_patch", "exec"],
        "deny": []
      }
    }
  }
}
```

**Dopo (multi-agent con profili diversi):**

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      }
    ]
  }
}
```

Le vecchie configurazioni `agent.*` vengono migrate da `openclaw doctor`; da ora in avanti usa `agents.defaults` + `agents.list`.

***

<div id="tool-restriction-examples">
  ## Esempi di limitazioni degli strumenti
</div>

<div id="read-only-agent">
  ### Agente in sola lettura
</div>

```json
{
  "tools": {
    "allow": ["read"],
    "deny": ["exec", "write", "edit", "apply_patch", "process"]
  }
}
```

<div id="safe-execution-agent-no-file-modifications">
  ### Agente a esecuzione sicura (senza modifiche ai file)
</div>

```json
{
  "tools": {
    "allow": ["read", "exec", "process"],
    "deny": ["write", "edit", "apply_patch", "browser", "gateway"]
  }
}
```

<div id="communication-only-agent">
  ### Agente di sola comunicazione
</div>

```json
{
  "tools": {
    "allow": ["sessions_list", "sessions_send", "sessions_history", "session_status"],
    "deny": ["exec", "write", "edit", "apply_patch", "read", "browser"]
  }
}
```

***

<div id="common-pitfall-non-main">
  ## Problema comune: &quot;non-main&quot;
</div>

`agents.defaults.sandbox.mode: "non-main"` si basa su `session.mainKey` (predefinito `"main"`),
non sull&#39;ID dell&#39;agente. Le sessioni di gruppo/canale ricevono sempre una propria chiave, quindi
sono considerate non-main e verranno eseguite in sandbox. Se vuoi che un agente non venga mai
eseguito in sandbox, imposta `agents.list[].sandbox.mode: "off"`.

***

<div id="testing">
  ## Verifica
</div>

Dopo aver configurato la sandbox multi-agente e gli strumenti:

1. **Verifica la risoluzione degli agenti:**
   ```exec
   openclaw agents list --bindings
   ```

2. **Verifica i container della sandbox:**
   ```exec
   docker ps --filter "name=openclaw-sbx-"
   ```

3. **Verifica le restrizioni sugli strumenti:**
   * Invia un messaggio che richiede strumenti soggetti a restrizioni
   * Verifica che l&#39;agente non possa usare gli strumenti non consentiti

4. **Monitora i log:**
   ```exec
   tail -f "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/logs/gateway.log" | grep -E "routing|sandbox|tools"
   ```

***

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

<div id="agent-not-sandboxed-despite-mode-all">
  ### Agente non in sandbox nonostante `mode: "all"`
</div>

* Verifica se esiste un `agents.defaults.sandbox.mode` globale che lo sovrascrive
* La configurazione specifica dell&#39;agente ha la precedenza, quindi imposta `agents.list[].sandbox.mode: "all"`

<div id="tools-still-available-despite-deny-list">
  ### Strumenti ancora disponibili anche con la deny list attiva
</div>

* Verifica l&#39;ordine di filtraggio degli strumenti: globale → agente → sandbox → sotto-agente
* Ogni livello può solo restringere ulteriormente, non può ri-concedere ciò che è stato revocato
* Verifica tramite i log: `[tools] filtering tools for agent:${agentId}`

<div id="container-not-isolated-per-agent">
  ### Container non isolato per agente
</div>

* Imposta `scope: "agent"` nella configurazione della sandbox specifica per l&#39;agente
* Il valore predefinito è `"session"`, che crea un container per sessione

***

<div id="see-also">
  ## Vedi anche
</div>

* [Instradamento multi-agente](/it/concepts/multi-agent)
* [Configurazione della sandbox](/it/gateway/configuration#agentsdefaults-sandbox)
* [Gestione delle sessioni](/it/concepts/session)