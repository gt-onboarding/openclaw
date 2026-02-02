---
summary: "Instradamento multi-agente: agenti isolati, account di canale e associazioni"
title: Instradamento multi-agente
read_when: "Quando ti servono più agenti isolati (spazi di lavoro + autenticazione) in un unico processo Gateway."
status: active
---

<div id="multi-agent-routing">
  # Instradamento multi-agente
</div>

Obiettivo: più agenti *isolati* (spazio di lavoro separato + `agentDir` + sessioni), oltre a più account di canale (ad es. due account WhatsApp) in un unico Gateway in esecuzione. Il traffico in ingresso viene instradato verso un agente tramite binding.

<div id="what-is-one-agent">
  ## Che cos&#39;è “un agente”?
</div>

Un **agente** è un cervello completamente contestualizzato con il proprio:

* **Spazio di lavoro** (file, AGENTS.md/SOUL.md/USER.md, note locali, regole di persona).
* **Directory dello stato** (`agentDir`) per i profili di autenticazione, il registro dei modelli e la configurazione specifica per‑agente.
* **Archivio delle sessioni** (cronologia della chat + stato di instradamento) in `~/.openclaw/agents/<agentId>/sessions`.

I profili di autenticazione sono **per‑agente**. Ogni agente legge dal proprio:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Le credenziali dell’agente principale **non** vengono condivise automaticamente. Non riutilizzare mai `agentDir`
tra agenti diversi (può causare collisioni di autenticazione/sessione). Se vuoi condividere le credenziali,
copia `auth-profiles.json` nell’`agentDir` dell’altro agente.

Le abilità sono per singolo agente tramite la cartella `skills/` di ciascuno spazio di lavoro, con abilità condivise
disponibili in `~/.openclaw/skills`. Vedi [Abilità: per-agente vs condivise](/it/tools/skills#per-agent-vs-shared-skills).

Il Gateway può ospitare **un solo agente** (predefinito) oppure **molti agenti** affiancati.

**Nota sullo spazio di lavoro:** lo spazio di lavoro di ciascun agente è la **directory di lavoro corrente (cwd) predefinita**, non una
sandbox restrittiva. I percorsi relativi vengono risolti all’interno dello spazio di lavoro, ma i percorsi assoluti possono
raggiungere altre posizioni dell’host a meno che il sandboxing non sia abilitato. Vedi
[Sandboxing](/it/gateway/sandboxing).

<div id="paths-quick-map">
  ## Percorsi (mappa rapida)
</div>

* Configurazione: `~/.openclaw/openclaw.json` (oppure `OPENCLAW_CONFIG_PATH`)
* Directory di stato: `~/.openclaw` (oppure `OPENCLAW_STATE_DIR`)
* Spazio di lavoro: `~/.openclaw/workspace` (oppure `~/.openclaw/workspace-<agentId>`)
* Directory dell&#39;agente: `~/.openclaw/agents/<agentId>/agent` (oppure `agents.list[].agentDir`)
* Sessioni: `~/.openclaw/agents/<agentId>/sessions`

<div id="single-agent-mode-default">
  ### Modalità agente singolo (predefinita)
</div>

Se non fai nulla, OpenClaw esegue un singolo agente:

* `agentId` ha valore predefinito **`main`**.
* Le sessioni hanno chiavi del tipo `agent:main:<mainKey>`.
* Lo spazio di lavoro predefinito è `~/.openclaw/workspace` (oppure `~/.openclaw/workspace-<profile>` quando `OPENCLAW_PROFILE` è impostato).
* Lo stato predefinito è `~/.openclaw/agents/main/agent`.

<div id="agent-helper">
  ## Helper per agenti
</div>

Usa la procedura guidata per aggiungere un nuovo agente isolato:

```bash
openclaw agents add work
```

Quindi aggiungi i `bindings` (oppure lascia che sia la procedura guidata a farlo) per instradare i messaggi in ingresso.

Verifica con:

```bash
openclaw agents list --bindings
```

<div id="multiple-agents-multiple-people-multiple-personalities">
  ## Più agenti = più persone, più personalità
</div>

Con **più agenti**, ogni `agentId` diventa una **persona completamente isolata**:

* **Numeri di telefono/account diversi** (per `accountId` del canale).
* **Personalità differenti** (file dello spazio di lavoro per agente come `AGENTS.md` e `SOUL.md`).
* **Autenticazione e sessioni separate** (nessuna interazione tra loro, a meno che non sia esplicitamente abilitata).

Questo permette a **più persone** di condividere un solo server Gateway mantenendo i loro “cervelli” di IA e i relativi dati isolati.

<div id="one-whatsapp-number-multiple-people-dm-split">
  ## Un numero WhatsApp, più persone (DM separati)
</div>

Puoi instradare **DM WhatsApp diversi** verso agenti diversi usando **un solo account WhatsApp**. Esegui il matching sul mittente in formato E.164 (ad esempio `+15551234567`) con `peer.kind: "dm"`. Le risposte arrivano comunque dallo stesso numero WhatsApp (nessuna identità del mittente per agente).

Dettaglio importante: le chat dirette vengono raggruppate nella **chiave di sessione principale** dell’agente, quindi un isolamento reale richiede **un agente per persona**.

Esempio:

```json5
{
  agents: {
    list: [
      { id: "alex", workspace: "~/.openclaw/workspace-alex" },
      { id: "mia", workspace: "~/.openclaw/workspace-mia" }
    ]
  },
  bindings: [
    { agentId: "alex", match: { channel: "whatsapp", peer: { kind: "dm", id: "+15551230001" } } },
    { agentId: "mia",  match: { channel: "whatsapp", peer: { kind: "dm", id: "+15551230002" } } }
  ],
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551230001", "+15551230002"]
    }
  }
}
```

Note:

* Il controllo degli accessi ai DM è **globale per ogni account WhatsApp** (abbinamento/lista di autorizzati), non per singolo agente.
* Per i gruppi condivisi, associa il gruppo a un unico agente oppure usa [Broadcast groups](/it/broadcast-groups).

<div id="routing-rules-how-messages-pick-an-agent">
  ## Regole di instradamento (come i messaggi selezionano un agente)
</div>

I binding sono **deterministici** e **vince quello più specifico**:

1. corrispondenza `peer` (id esatto DM/gruppo/canale)
2. `guildId` (Discord)
3. `teamId` (Slack)
4. corrispondenza `accountId` per un canale
5. corrispondenza a livello di canale (`accountId: "*"`)
6. fallback sull&#39;agente predefinito (`agents.list[].default`, altrimenti primo elemento della lista, predefinito: `main`)

<div id="multiple-accounts-phone-numbers">
  ## Account / numeri di telefono multipli
</div>

I canali che supportano **account multipli** (ad esempio WhatsApp) usano `accountId` per identificare
ogni accesso. Ogni `accountId` può essere instradato verso un agente diverso, così un singolo server può ospitare
più numeri di telefono senza mescolare le sessioni.

<div id="concepts">
  ## Concetti
</div>

* `agentId`: un “cervello” (spazio di lavoro, autenticazione per agente, archivio delle sessioni per agente).
* `accountId`: un’istanza di account di canale (ad es. account WhatsApp `"personal"` vs `"biz"`).
* `binding`: instrada i messaggi in ingresso verso un `agentId` tramite `(channel, accountId, peer)` e facoltativamente ID di guild/team.
* Le chat dirette vengono ricondotte a `agent:<agentId>:<mainKey>` (“principale” per agente; `session.mainKey`).

<div id="example-two-whatsapps-two-agents">
  ## Esempio: due WhatsApp → due agenti
</div>

`~/.openclaw/openclaw.json` (JSON5):

```js
{
  agents: {
    list: [
      {
        id: "home",
        default: true,
        name: "Home",
        workspace: "~/.openclaw/workspace-home",
        agentDir: "~/.openclaw/agents/home/agent",
      },
      {
        id: "work",
        name: "Work",
        workspace: "~/.openclaw/workspace-work",
        agentDir: "~/.openclaw/agents/work/agent",
      },
    ],
  },

  // Deterministic routing: first match wins (most-specific first).
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },

    // Optional per-peer override (example: send a specific group to work agent).
    {
      agentId: "work",
      match: {
        channel: "whatsapp",
        accountId: "personal",
        peer: { kind: "group", id: "1203630...@g.us" },
      },
    },
  ],

  // Disabilitato per impostazione predefinita: la messaggistica agente-ad-agente deve essere esplicitamente abilitata e aggiunta alla lista di autorizzati.
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },

  channels: {
    whatsapp: {
      accounts: {
        personal: {
          // Optional override. Default: ~/.openclaw/credentials/whatsapp/personal
          // authDir: "~/.openclaw/credentials/whatsapp/personal",
        },
        biz: {
          // Optional override. Default: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

<div id="example-whatsapp-daily-chat-telegram-deep-work">
  ## Esempio: chat quotidiana su WhatsApp + lavoro concentrato su Telegram
</div>

Suddividi per canale: instrada WhatsApp verso un agente rapido per l&#39;uso quotidiano e Telegram verso un agente Opus.

```json5
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5"
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-5"
      }
    ]
  },
  bindings: [
    { agentId: "chat", match: { channel: "whatsapp" } },
    { agentId: "opus", match: { channel: "telegram" } }
  ]
}
```

Note:

* Se hai più account per lo stesso canale, aggiungi `accountId` al binding (per esempio `{ channel: "whatsapp", accountId: "personal" }`).
* Per instradare un singolo DM/gruppo verso Opus mantenendo il resto sulla chat, aggiungi un binding `match.peer` per quel peer specifico; i match per peer hanno sempre la precedenza sulle regole valide per l’intero canale.

<div id="example-same-channel-one-peer-to-opus">
  ## Esempio: stesso canale, un peer per Opus
</div>

Mantieni WhatsApp sull&#39;agente rapido, ma inoltra un DM a Opus:

```json5
{
  agents: {
    list: [
      { id: "chat", name: "Everyday", workspace: "~/.openclaw/workspace-chat", model: "anthropic/claude-sonnet-4-5" },
      { id: "opus", name: "Deep Work", workspace: "~/.openclaw/workspace-opus", model: "anthropic/claude-opus-4-5" }
    ]
  },
  bindings: [
    { agentId: "opus", match: { channel: "whatsapp", peer: { kind: "dm", id: "+15551234567" } } },
    { agentId: "chat", match: { channel: "whatsapp" } }
  ]
}
```

I binding dei peer hanno sempre la precedenza, quindi tienili prima della regola valida per l’intero canale.

<div id="family-agent-bound-to-a-whatsapp-group">
  ## Agente familiare vincolato a un gruppo WhatsApp
</div>

Collega un agente familiare dedicato a un singolo gruppo WhatsApp, attivabile solo tramite menzione
e con criteri per gli strumenti più restrittivi:

```json5
{
  agents: {
    list: [
      {
        id: "family",
        name: "Family",
        workspace: "~/.openclaw/workspace-family",
        identity: { name: "Family Bot" },
        groupChat: {
          mentionPatterns: ["@family", "@familybot", "@Family Bot"]
        },
        sandbox: {
          mode: "all",
          scope: "agent"
        },
        tools: {
          allow: ["exec", "read", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
          deny: ["write", "edit", "apply_patch", "browser", "canvas", "nodes", "cron"]
        }
      }
    ]
  },
  bindings: [
    {
      agentId: "family",
      match: {
        channel: "whatsapp",
        peer: { kind: "group", id: "120363999999999999@g.us" }
      }
    }
  ]
}
```

Note:

* Le liste di autorizzazione/blocco per gli strumenti riguardano i **tool**, non le abilità. Se un&#39;abilità deve eseguire un
  binario, assicurati che `exec` sia consentito e che il binario esista nella sandbox.
* Per un controllo più restrittivo, imposta `agents.list[].groupChat.mentionPatterns` e mantieni
  le liste di autorizzati per i gruppi attive per il canale.

<div id="per-agent-sandbox-and-tool-configuration">
  ## Configurazione della sandbox e degli strumenti per singolo agente
</div>

A partire dalla versione v2026.1.6, ogni agente può disporre di una propria sandbox e di restrizioni specifiche sugli strumenti:

```js
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: {
          mode: "off",  // No sandbox for personal agent
        },
        // No tool restrictions - all tools available
      },
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",     // Always sandboxed
          scope: "agent",  // One container per agent
          docker: {
            // Optional one-time setup after container creation
            setupCommand: "apt-get update && apt-get install -y git curl",
          },
        },
        tools: {
          allow: ["read"],                    // Only read tool
          deny: ["exec", "write", "edit", "apply_patch"],    // Nega gli altri
        },
      },
    ],
  },
}
```

Nota: `setupCommand` si trova sotto `sandbox.docker` e viene eseguito una volta alla creazione del container.
Gli override per agente di `sandbox.docker.*` vengono ignorati quando lo scope risolto è `"shared"`.

**Vantaggi:**

* **Isolamento della sicurezza**: limita i tool per gli agenti non attendibili
* **Controllo delle risorse**: esegui in sandbox agenti specifici mantenendo gli altri sull&#39;host
* **Policy flessibili**: permessi diversi per ogni agente

Nota: `tools.elevated` è **globale** e basata sul mittente; non è configurabile per singolo agente.
Se ti servono limitazioni per agente, usa `agents.list[].tools` per negare `exec`.
Per il targeting di gruppo, usa `agents.list[].groupChat.mentionPatterns` in modo che le @mention vengano mappate correttamente all&#39;agente previsto.

Vedi [Multi-Agent Sandbox &amp; Tools](/it/multi-agent-sandbox-tools) per esempi dettagliati.
