---
title: Configurazione
summary: "Tutte le opzioni di configurazione per ~/.openclaw/openclaw.json con esempi"
read_when:
  - Per aggiungere o modificare campi di configurazione
---

<div id="configuration">
  # Configurazione 🔧
</div>

OpenClaw legge una configurazione **JSON5** opzionale da `~/.openclaw/openclaw.json` (sono consentiti commenti e virgole finali).

Se il file non esiste, OpenClaw utilizza valori predefiniti abbastanza sicuri (Agente Pi incorporato + sessioni per mittente + spazio di lavoro `~/.openclaw/workspace`). Di solito ti serve una configurazione solo per:

* limitare chi può attivare il bot (`channels.whatsapp.allowFrom`, `channels.telegram.allowFrom`, ecc.)
* gestire le liste di autorizzati per i gruppi + il comportamento delle menzioni (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.discord.guilds`, `agents.list[].groupChat`)
* personalizzare i prefissi dei messaggi (`messages`)
* impostare lo spazio di lavoro dell&#39;agente (`agents.defaults.workspace` o `agents.list[].workspace`)
* ottimizzare i valori predefiniti dell&#39;agente incorporato (`agents.defaults`) e il comportamento della sessione (`session`)
* impostare l&#39;identità per ogni singolo agente (`agents.list[].identity`)

> **Nuovo alla configurazione di OpenClaw?** Consulta la guida [Esempi di configurazione](/it/gateway/configuration-examples) per esempi completi con spiegazioni dettagliate!

<div id="strict-config-validation">
  ## Validazione rigorosa della configurazione
</div>

OpenClaw accetta solo configurazioni che corrispondono completamente allo schema.
Chiavi sconosciute, tipi non corretti o valori non validi fanno sì che il Gateway **si rifiuti di avviarsi** per motivi di sicurezza.

Quando la validazione fallisce:

* Il Gateway non si avvia.
* Sono consentiti solo i comandi diagnostici (per esempio: `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`).
* Esegui `openclaw doctor` per vedere esattamente quali sono i problemi.
* Esegui `openclaw doctor --fix` (o `--yes`) per applicare migrazioni/correzioni.

`openclaw doctor` non scrive mai modifiche a meno che tu non attivi esplicitamente `--fix`/`--yes`.

<div id="schema-ui-hints">
  ## Suggerimenti per Schema + UI
</div>

Il Gateway espone uno Schema JSON che rappresenta la configurazione tramite `config.schema`, per gli editor dell&#39;UI.
La Control UI genera un modulo a partire da questo schema, con un editor **Raw JSON** come via di uscita.

I plugin di canale e le estensioni possono registrare schema e suggerimenti per l&#39;UI relativi alla propria configurazione, così le impostazioni dei canali
restano guidate dallo schema tra le varie app senza moduli con logica hard-coded.

I suggerimenti (etichette, raggruppamenti, campi sensibili) vengono distribuiti insieme allo schema, così i client possono renderizzare
moduli migliori senza dover codificare a mano i dettagli della configurazione.

<div id="apply-restart-rpc">
  ## Apply + restart (RPC)
</div>

Usa `config.apply` per validare e scrivere l&#39;intera configurazione e riavviare il Gateway in un solo passaggio.
Scrive una sentinella di riavvio e invia un ping all&#39;ultima sessione attiva una volta che il Gateway è di nuovo attivo.

Avviso: `config.apply` sostituisce l&#39;**intera configurazione**. Se vuoi modificare solo alcune chiavi,
usa `config.patch` o `openclaw config set`. Tieni una copia di backup di `~/.openclaw/openclaw.json`.

Parametri:

* `raw` (string) — payload JSON5 per l&#39;intera configurazione
* `baseHash` (opzionale) — hash della configurazione da `config.get` (richiesto quando esiste già una configurazione)
* `sessionKey` (opzionale) — chiave dell&#39;ultima sessione attiva per il ping di riattivazione
* `note` (opzionale) — nota da includere nella sentinella di riavvio
* `restartDelayMs` (opzionale) — ritardo prima del riavvio (predefinito 2000)

Esempio (tramite `gateway call`):

```bash
openclaw gateway call config.get --params '{}' # cattura payload.hash
openclaw gateway call config.apply --params '{
  "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

<div id="partial-updates-rpc">
  ## Aggiornamenti parziali (RPC)
</div>

Usa `config.patch` per unire un aggiornamento parziale alla config esistente senza sovrascrivere
le chiavi non correlate. Applica la semantica di JSON Merge Patch:

* gli oggetti vengono uniti ricorsivamente
* `null` elimina una chiave
* gli array vengono sostituiti

Come `config.apply`, valida, scrive la config, memorizza un sentinel di riavvio e pianifica
il riavvio del Gateway (con un&#39;eventuale riattivazione quando viene fornito `sessionKey`).

Parametri:

* `raw` (stringa) — payload JSON5 contenente solo le chiavi da modificare
* `baseHash` (obbligatorio) — hash della config da `config.get`
* `sessionKey` (facoltativo) — chiave dell&#39;ultima sessione attiva per il ping di wake-up
* `note` (facoltativo) — nota da includere nel sentinel di riavvio
* `restartDelayMs` (facoltativo) — ritardo prima del riavvio (predefinito 2000)

Esempio:

```bash
openclaw gateway call config.get --params '{}' # cattura payload.hash
openclaw gateway call config.patch --params '{
  "raw": "{\\n  channels: { telegram: { groups: { \\"*\\": { requireMention: false } } } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

<div id="minimal-config-recommended-starting-point">
  ## Configurazione minima (configurazione iniziale consigliata)
</div>

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } }
}
```

Genera l&#39;immagine predefinita una sola volta con:

```bash
scripts/sandbox-setup.sh
```

<div id="self-chat-mode-recommended-for-group-control">
  ## Modalità self-chat (consigliata per la gestione dei gruppi)
</div>

Per impedire al bot di rispondere alle menzioni @ su WhatsApp nei gruppi (rispondendo solo a specifici trigger testuali):

```json5
{
  agents: {
    defaults: { workspace: "~/.openclaw/workspace" },
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["@openclaw", "reisponde"] }
      }
    ]
  },
  channels: {
    whatsapp: {
      // La lista di autorizzati è solo per i messaggi diretti; includere il proprio numero abilita la modalità di auto-chat.
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } }
    }
  }
}
```

<div id="config-includes-include">
  ## Inclusioni di configurazione (`$include`)
</div>

Suddividi la configurazione in più file usando la direttiva `$include`. Questo è utile per:

* Organizzare configurazioni di grandi dimensioni (ad esempio, definizioni di agenti per singolo client)
* Condividere impostazioni comuni tra ambienti diversi
* Mantenere separate le configurazioni sensibili

<div id="basic-usage">
  ### Utilizzo di base
</div>

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  
  // Includi un singolo file (sostituisce il valore della chiave)
  agents: { "$include": "./agents.json5" },
  
  // Include multiple files (deep-merged in order)
  broadcast: { 
    "$include": [
      "./clients/mueller.json5",
      "./clients/schmidt.json5"
    ]
  }
}
```

```json5
// ~/.openclaw/agents.json5
{
  defaults: { sandbox: { mode: "all", scope: "session" } },
  list: [
    { id: "main", workspace: "~/.openclaw/workspace" }
  ]
}
```

<div id="merge-behavior">
  ### Comportamento di merge
</div>

* **Singolo file**: Sostituisce l&#39;oggetto che contiene `$include`
* **Array di file**: Esegue un deep merge dei file in ordine (i file successivi sovrascrivono quelli precedenti)
* **Con chiavi allo stesso livello**: Le chiavi allo stesso livello vengono unite dopo gli include (sovrascrivono i valori inclusi)
* **Chiavi allo stesso livello + array/primitivi**: Non supportato (il contenuto incluso deve essere un oggetto)

```json5
// Le chiavi allo stesso livello sovrascrivono i valori inclusi
{
  "$include": "./base.json5",   // { a: 1, b: 2 }
  b: 99                          // Risultato: { a: 1, b: 99 }
}
```

<div id="nested-includes">
  ### Include annidati
</div>

I file inclusi possono a loro volta contenere direttive `$include` (fino a 10 livelli di annidamento):

```json5
// clients/mueller.json5
{
  agents: { "$include": "./mueller/agents.json5" },
  broadcast: { "$include": "./mueller/broadcast.json5" }
}
```

<div id="path-resolution">
  ### Risoluzione dei percorsi
</div>

* **Percorsi relativi**: Risolti rispetto al file che li include
* **Percorsi assoluti**: Usati così come sono
* **Directory genitori**: I riferimenti a `../` funzionano come previsto

```json5
{ "$include": "./sub/config.json5" }      // relative
{ "$include": "/etc/openclaw/base.json5" } // absolute
{ "$include": "../shared/common.json5" }   // directory padre
```

<div id="error-handling">
  ### Gestione degli errori
</div>

* **File mancante**: Errore esplicito con il percorso risolto
* **Errore di parsing**: Mostra quale file incluso ha causato l&#39;errore
* **Include circolari**: Rilevati e segnalati con la catena di include

<div id="example-multi-client-legal-setup">
  ### Esempio: configurazione legale per più clienti
</div>

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789, auth: { token: "secret" } },
  
  // Common agent defaults
  agents: {
    defaults: {
      sandbox: { mode: "all", scope: "session" }
    },
    // Unisce le liste degli agenti da tutti i clienti
    list: { "$include": [
      "./clients/mueller/agents.json5",
      "./clients/schmidt/agents.json5"
    ]}
  },
  
  // Merge broadcast configs
  broadcast: { "$include": [
    "./clients/mueller/broadcast.json5",
    "./clients/schmidt/broadcast.json5"
  ]},
  
  channels: { whatsapp: { groupPolicy: "allowlist" } }
}
```

```json5
// ~/.openclaw/clients/mueller/agents.json5
[
  { id: "mueller-transcribe", workspace: "~/clients/mueller/transcribe" },
  { id: "mueller-docs", workspace: "~/clients/mueller/docs" }
]
```

```json5
// ~/.openclaw/clients/mueller/broadcast.json5
{
  "120363403215116621@g.us": ["mueller-transcribe", "mueller-docs"]
}
```

<div id="common-options">
  ## Opzioni comuni
</div>

<div id="env-vars-env">
  ### Variabili d&#39;ambiente + `.env`
</div>

OpenClaw legge le variabili d&#39;ambiente dal processo padre (shell, launchd/systemd, CI, ecc.).

Inoltre carica:

* `.env` dalla directory di lavoro corrente (se presente)
* un `.env` globale di fallback da `~/.openclaw/.env` (alias `$OPENCLAW_STATE_DIR/.env`)

Nessuno dei file `.env` sovrascrive le variabili d&#39;ambiente esistenti.

Puoi anche fornire variabili d&#39;ambiente in linea nella configurazione. Queste vengono applicate solo se
nell&#39;ambiente di processo manca la chiave (stessa regola di non sovrascrittura):

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

Vedi [/environment](/it/environment) per l&#39;ordine di precedenza completo e le origini.

<div id="envshellenv-optional">
  ### `env.shellEnv` (opzionale)
</div>

Comodità opzionale: se abilitato e nessuna delle chiavi previste è ancora impostata, OpenClaw esegue la tua shell di login e importa solo le chiavi previste mancanti (non le sovrascrive mai).
In pratica equivale a eseguire il profilo della tua shell.

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

Variabili d&#39;ambiente equivalenti:

* `OPENCLAW_LOAD_SHELL_ENV=1`
* `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

<div id="env-var-substitution-in-config">
  ### Sostituzione delle variabili d&#39;ambiente nella configurazione
</div>

Puoi fare riferimento alle variabili d&#39;ambiente direttamente in qualunque valore di tipo stringa nella configurazione usando la sintassi
`${VAR_NAME}`. Le variabili vengono sostituite al momento del caricamento della configurazione, prima della validazione.

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}"
      }
    }
  },
  gateway: {
    auth: {
      token: "${OPENCLAW_GATEWAY_TOKEN}"
    }
  }
}
```

**Regole:**

* Vengono considerate solo variabili d&#39;ambiente con nome in maiuscolo: `[A-Z_][A-Z0-9_]*`
* Le variabili d&#39;ambiente mancanti o vuote generano un errore al caricamento della configurazione
* Usa `$${VAR}` per stampare letteralmente `${VAR}`
* Funziona con `$include` (anche i file inclusi vengono sottoposti a sostituzione)

**Sostituzione inline:**

```json5
{
  models: {
    providers: {
      custom: {
        baseUrl: "${CUSTOM_API_BASE}/v1"  // → "https://api.example.com/v1"
      }
    }
  }
}
```

<div id="auth-storage-oauth-api-keys">
  ### Archiviazione dell&#39;autenticazione (OAuth + API keys)
</div>

OpenClaw memorizza i profili di autenticazione **per-agente** (OAuth + API keys) in:

* `<agentDir>/auth-profiles.json` (predefinito: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`)

Vedi anche: [/concepts/oauth](/it/concepts/oauth)

Importazioni OAuth legacy:

* `~/.openclaw/credentials/oauth.json` (oppure `$OPENCLAW_STATE_DIR/credentials/oauth.json`)

L&#39;agente Pi incorporato mantiene una cache di runtime in:

* `<agentDir>/auth.json` (gestita automaticamente; non modificarlo manualmente)

Directory agente legacy (prima del supporto multi-agente):

* `~/.openclaw/agent/*` (migrata da `openclaw doctor` in `~/.openclaw/agents/<defaultAgentId>/agent/*`)

Override:

* Directory OAuth (solo import legacy): `OPENCLAW_OAUTH_DIR`
* Directory agente (override della root dell&#39;agente predefinito): `OPENCLAW_AGENT_DIR` (preferito), `PI_CODING_AGENT_DIR` (legacy)

Alla prima esecuzione, OpenClaw importa le voci di `oauth.json` in `auth-profiles.json`.

<div id="auth">
  ### `auth`
</div>

Metadati opzionali per i profili di autenticazione. Questo **non** memorizza segreti; associa gli ID di profilo a un provider + modalità (ed eventualmente a un indirizzo email) e definisce l’ordine di rotazione dei provider utilizzato per il failover.

```json5
{
  auth: {
    profiles: {
      "anthropic:me@example.com": { provider: "anthropic", mode: "oauth", email: "me@example.com" },
      "anthropic:work": { provider: "anthropic", mode: "api_key" }
    },
    order: {
      anthropic: ["anthropic:me@example.com", "anthropic:work"]
    }
  }
}
```

<div id="agentslistidentity">
  ### `agents.list[].identity`
</div>

Identità opzionale per singolo agente usata per i valori predefiniti e l’UX. Viene scritta dall&#39;assistente di onboarding per macOS.

Se impostata, OpenClaw deriva i valori predefiniti (solo quando non li hai impostati esplicitamente):

* `messages.ackReaction` da `identity.emoji` dell’**agente attivo** (con fallback a 👀)
* `agents.list[].groupChat.mentionPatterns` da `identity.name`/`identity.emoji` dell’agente (così “@Samantha” funziona nei gruppi su Telegram/Slack/Discord/Google Chat/iMessage/WhatsApp)
* `identity.avatar` accetta un percorso di immagine relativo allo spazio di lavoro o un URL remoto/data URL. I file locali devono risiedere all’interno dello spazio di lavoro dell’agente.

`identity.avatar` accetta:

* Percorso relativo allo spazio di lavoro (deve rimanere all’interno dello spazio di lavoro dell’agente)
* URL `http(s)`
* URI `data:`

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png"
        }
      }
    ]
  }
}
```

<div id="wizard">
  ### `wizard`
</div>

Metadati generati dai wizard della CLI (`onboard`, `configure`, `doctor`).

```json5
{
  wizard: {
    lastRunAt: "2026-01-01T00:00:00.000Z",
    lastRunVersion: "2026.1.4",
    lastRunCommit: "abc1234",
    lastRunCommand: "configure",
    lastRunMode: "local"
  }
}
```

<div id="logging">
  ### `logging`
</div>

* File di log predefinito: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`
* Se vuoi un percorso stabile, imposta `logging.file` su `/tmp/openclaw/openclaw.log`.
* L&#39;output della console può essere regolato separatamente tramite:
  * `logging.consoleLevel` (predefinito `info`, viene portato a `debug` quando usi `--verbose`)
  * `logging.consoleStyle` (`pretty` | `compact` | `json`)
* I riepiloghi degli strumenti possono essere oscurati per evitare la divulgazione di segreti:
  * `logging.redactSensitive` (`off` | `tools`, predefinito: `tools`)
  * `logging.redactPatterns` (array di stringhe regex; sostituisce i valori predefiniti)

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty",
    redactSensitive: "tools",
    redactPatterns: [
      // Esempio: sovrascrivi i valori predefiniti con le tue regole.
      "\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1",
      "/\\bsk-[A-Za-z0-9_-]{8,}\\b/gi"
    ]
  }
}
```

<div id="channelswhatsappdmpolicy">
  ### `channels.whatsapp.dmPolicy`
</div>

Controlla come vengono gestite le chat dirette WhatsApp (DM):

* `"pairing"` (predefinito): i mittenti sconosciuti ricevono un codice di abbinamento; il proprietario deve approvarlo
* `"allowlist"`: consente solo i mittenti presenti in `channels.whatsapp.allowFrom` (o nello store di abbinamenti approvati)
* `"open"`: consente tutte le DM in arrivo da qualsiasi utente senza restrizioni (**richiede** che `channels.whatsapp.allowFrom` includa `"*"`)
* `"disabled"`: ignora tutte le DM in arrivo

I codici di abbinamento scadono dopo 1 ora; il bot invia un codice di abbinamento solo quando viene creata una nuova richiesta. Le richieste di abbinamento DM in sospeso sono limitate a **3 per canale** per impostazione predefinita.

Approvazioni di abbinamento:

* `openclaw pairing list whatsapp`
* `openclaw pairing approve whatsapp <code>`

<div id="channelswhatsappallowfrom">
  ### `channels.whatsapp.allowFrom`
</div>

Lista di numeri di telefono E.164 autorizzati che possono attivare le risposte automatiche WhatsApp (**solo DM**).
Se è vuota e `channels.whatsapp.dmPolicy="pairing"`, i mittenti sconosciuti riceveranno un codice di abbinamento.
Per i gruppi, usa `channels.whatsapp.groupPolicy` + `channels.whatsapp.groupAllowFrom`.

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000, // optional outbound chunk size (chars)
      chunkMode: "length", // modalità di chunking opzionale (length | newline)
      mediaMaxMb: 50 // optional inbound media cap (MB)
    }
  }
}
```

<div id="channelswhatsappsendreadreceipts">
  ### `channels.whatsapp.sendReadReceipts`
</div>

Controlla se i messaggi WhatsApp in arrivo vengono contrassegnati come letti (spunte blu). Valore predefinito: `true`.

La modalità Self-chat ignora sempre le conferme di lettura, anche quando sono abilitate.

Override per account: `channels.whatsapp.accounts.<id>.sendReadReceipts`.

```json5
{
  channels: {
    whatsapp: { sendReadReceipts: false }
  }
}
```

<div id="channelswhatsappaccounts-multi-account">
  ### `channels.whatsapp.accounts` (multi-account)
</div>

Esegui più account WhatsApp all&#39;interno di un unico Gateway:

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {}, // optional; keeps the default id stable
        personal: {},
        biz: {
          // Override opzionale. Predefinito: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        }
      }
    }
  }
}
```

Note:

* I comandi in uscita usano per impostazione predefinita l&#39;account `default`, se presente; in caso contrario, il primo ID account configurato (in ordine di ordinamento).
* La directory di autenticazione Baileys legacy a singolo account viene migrata da `openclaw doctor` in `whatsapp/default`.

<div id="channelstelegramaccounts-channelsdiscordaccounts-channelsgooglechataccounts-channelsslackaccounts-channelsmattermostaccounts-channelssignalaccounts-channelsimessageaccounts">
  ### `channels.telegram.accounts` / `channels.discord.accounts` / `channels.googlechat.accounts` / `channels.slack.accounts` / `channels.mattermost.accounts` / `channels.signal.accounts` / `channels.imessage.accounts`
</div>

Gestisci più account per canale (ogni account ha il proprio `accountId` e un `name` facoltativo):

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC..."
        },
        alerts: {
          name: "Alerts bot",
          botToken: "987654:XYZ..."
        }
      }
    }
  }
}
```

Note:

* `default` viene usato quando `accountId` è omesso (CLI + instradamento).
* I token di ambiente (env) si applicano solo all’account **default**.
* Le impostazioni di base del canale (criteri di gruppo, gating delle menzioni, ecc.) si applicano a tutti gli account, a meno che non siano sovrascritte per singolo account.
* Usa `bindings[].match.accountId` per instradare ciascun account verso un diverso `agents.defaults`.

<div id="group-chat-mention-gating-agentslistgroupchat-messagesgroupchat">
  ### Limitazione basata su menzioni nelle chat di gruppo (`agents.list[].groupChat` + `messages.groupChat`)
</div>

Per impostazione predefinita, i messaggi nei gruppi **richiedono una menzione** (tramite metadati di menzione o pattern regex). Si applica alle chat di gruppo WhatsApp, Telegram, Discord, Google Chat e iMessage.

**Tipi di menzione:**

* **Menzioni tramite metadati**: @-menzioni native della piattaforma (ad esempio, tocca‑per‑menzionare in WhatsApp). Ignorate nella modalità di self‑chat di WhatsApp (vedi `channels.whatsapp.allowFrom`).
* **Pattern di testo**: pattern regex definiti in `agents.list[].groupChat.mentionPatterns`. Sempre verificati, indipendentemente dalla modalità di self‑chat.
* La limitazione basata su menzioni viene applicata solo quando il rilevamento della menzione è possibile (menzioni native o almeno un `mentionPattern`).

```json5
{
  messages: {
    groupChat: { historyLimit: 50 }
  },
  agents: {
    list: [
      { id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }
    ]
  }
}
```

`messages.groupChat.historyLimit` imposta il valore predefinito globale per il contesto della cronologia nei gruppi. I canali possono sovrascriverlo con `channels.<channel>.historyLimit` (oppure `channels.<channel>.accounts.*.historyLimit` per configurazioni multi-account). Imposta `0` per disabilitare la rotazione della cronologia.`

<div id="dm-history-limits">
  #### Limiti della cronologia dei DM
</div>

Le conversazioni DM usano una cronologia basata su sessioni gestita dall&#39;agente. Puoi limitare il numero di interazioni utente conservate per ogni sessione DM:

```json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30,  // limit DM sessions to 30 user turns
      dms: {
        "123456789": { historyLimit: 50 }  // override per singolo utente (ID utente)
      }
    }
  }
}
```

Ordine di applicazione:

1. Override per DM: `channels.<provider>.dms[userId].historyLimit`
2. Valore predefinito del provider: `channels.<provider>.dmHistoryLimit`
3. Nessun limite (tutta la cronologia viene conservata)

Provider supportati: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

Override per agente (ha la precedenza quando impostato, anche `[]`):

```json5
{
  agents: {
    list: [
      { id: "work", groupChat: { mentionPatterns: ["@workbot", "\\+15555550123"] } },
      { id: "personal", groupChat: { mentionPatterns: ["@homebot", "\\+15555550999"] } }
    ]
  }
}
```

I valori predefiniti per il controllo delle menzioni sono specifici per canale (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.imessage.groups`, `channels.discord.guilds`). Quando `*.groups` è impostato, funge anche da lista di autorizzati per i gruppi; includi `"*"` per autorizzare tutti i gruppi.

Per rispondere **solo** a specifici trigger testuali (ignorando le menzioni native con @):

```json5
{
  channels: {
    whatsapp: {
      // Includi il tuo numero per abilitare la modalità self-chat (ignora le @-menzioni native).
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } }
    }
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          // Only these text patterns will trigger responses
          mentionPatterns: ["reisponde", "@openclaw"]
        }
      }
    ]
  }
}
```

<div id="group-policy-per-channel">
  ### Criterio di gruppo (per canale)
</div>

Usa `channels.*.groupPolicy` per stabilire se accettare o meno i messaggi provenienti da gruppi/stanze:

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"]
    },
    telegram: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["tg:123456789", "@alice"]
    },
    signal: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"]
    },
    imessage: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["chat_id:123"]
    },
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"]
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "GUILD_ID": {
          channels: { help: { allow: true } }
        }
      }
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } }
    }
  }
}
```

Note:

* `"open"`: i gruppi ignorano le liste di autorizzati; la limitazione tramite menzione continua ad applicarsi. L&#39;impostazione `open` consente di accettare messaggi senza restrizioni da qualsiasi utente.
* `"disabled"`: blocca tutti i messaggi di gruppo/stanza.
* `"allowlist"`: consente solo gruppi/stanze che corrispondono alla lista di autorizzati configurata.
* `channels.defaults.groupPolicy` imposta il valore predefinito quando il `groupPolicy` di un provider non è impostato.
* WhatsApp/Telegram/Signal/iMessage/Microsoft Teams usano `groupAllowFrom` (fallback: `allowFrom` esplicito).
* Discord/Slack usano liste di autorizzati per i canali (`channels.discord.guilds.*.channels`, `channels.slack.channels`).
* I DM di gruppo (Discord/Slack) sono comunque controllati da `dm.groupEnabled` + `dm.groupChannels`.
* Il valore predefinito è `groupPolicy: "allowlist"` (a meno che non venga sovrascritto da `channels.defaults.groupPolicy`); se non è configurata alcuna lista di autorizzati, i messaggi di gruppo vengono bloccati.

<div id="multi-agent-routing-agentslist-bindings">
  ### Instradamento multi-agente (`agents.list` + `bindings`)
</div>

Esegui più agenti isolati (spazio di lavoro separato, `agentDir`, sessioni) all&#39;interno di un unico Gateway.
I messaggi in ingresso vengono instradati a un agente tramite i binding.

* `agents.list[]`: sovrascritture per singolo agente.
  * `id`: id agente stabile (obbligatorio).
  * `default`: opzionale; quando ne sono impostati più di uno, vince il primo e viene registrato un avviso.
    Se non ne viene impostato nessuno, la **prima voce** nell&#39;elenco è l&#39;agente predefinito.
  * `name`: nome visualizzato per l&#39;agente.
  * `workspace`: predefinito `~/.openclaw/workspace-<agentId>` (per `main`, usa come fallback `agents.defaults.workspace`).
  * `agentDir`: predefinito `~/.openclaw/agents/<agentId>/agent`.
  * `model`: modello predefinito per singolo agente, che sovrascrive `agents.defaults.model` per quell&#39;agente.
    * forma stringa: `"provider/model"`, sovrascrive solo `agents.defaults.model.primary`
    * forma oggetto: `{ primary, fallbacks }` (i fallback sovrascrivono `agents.defaults.model.fallbacks`; `[]` disabilita i fallback globali per quell&#39;agente)
  * `identity`: nome/tema/emoji per singolo agente (usati per i pattern di menzione + reazioni di ack).
  * `groupChat`: gating delle menzioni per singolo agente (`mentionPatterns`).
  * `sandbox`: configurazione della sandbox per singolo agente (sovrascrive `agents.defaults.sandbox`).
    * `mode`: `"off"` | `"non-main"` | `"all"`
    * `workspaceAccess`: `"none"` | `"ro"` | `"rw"`
    * `scope`: `"session"` | `"agent"` | `"shared"`
    * `workspaceRoot`: root sandbox personalizzata per lo spazio di lavoro
    * `docker`: sovrascritture Docker per singolo agente (ad es. `image`, `network`, `env`, `setupCommand`, limiti; ignorato quando `scope: "shared"`)
    * `browser`: sovrascritture per-agente del browser in sandbox (ignorato quando `scope: "shared"`)
    * `prune`: sovrascritture per-agente delle operazioni di pulizia della sandbox (ignorato quando `scope: "shared"`)
  * `subagents`: valori predefiniti dei sub-agenti per singolo agente.
    * `allowAgents`: lista di autorizzati di id di agenti per `sessions_spawn` da questo agente (`["*"]` = consenti qualsiasi; predefinito: solo stesso agente)
  * `tools`: restrizioni sugli strumenti per singolo agente (applicate prima dei criteri sugli strumenti della sandbox).
    * `profile`: profilo di base degli strumenti (applicato prima di allow/deny)
    * `allow`: array di nomi di strumenti consentiti
    * `deny`: array di nomi di strumenti bloccati (deny ha la precedenza)
* `agents.defaults`: valori predefiniti condivisi per gli agenti (modello, spazio di lavoro, sandbox, ecc.).
* `bindings[]`: instrada i messaggi in ingresso a un `agentId`.
  * `match.channel` (obbligatorio)
  * `match.accountId` (opzionale; `*` = qualsiasi account; omesso = account predefinito)
  * `match.peer` (opzionale; `{ kind: dm|group|channel, id }`)
  * `match.guildId` / `match.teamId` (opzionale; specifico del canale)

Ordine di corrispondenza deterministico:

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (esatto, senza peer/guild/team)
5. `match.accountId: "*"` (a livello di canale, senza peer/guild/team)
6. agente predefinito (`agents.list[].default`, altrimenti prima voce della lista, altrimenti `"main"`)

All&#39;interno di ciascun livello di corrispondenza, vince la prima voce corrispondente in `bindings`.

<div id="per-agent-access-profiles-multi-agent">
  #### Profili di accesso per agente (multi-agent)
</div>

Ogni agente può avere la propria sandbox e le proprie policy sugli strumenti. Usa questo per combinare diversi livelli di accesso in un unico Gateway:

* **Accesso completo** (agente personale)
* Strumenti **in sola lettura** + spazio di lavoro
* **Nessun accesso al filesystem** (solo strumenti di messaggistica/sessione)

Vedi [Multi-Agent Sandbox &amp; Tools](/it/multi-agent-sandbox-tools) per l&#39;ordine di precedenza e
altri esempi.

Accesso completo (nessuna sandbox):

```json5
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: { mode: "off" }
      }
    ]
  }
}
```

Tool in sola lettura + spazio di lavoro in sola lettura:

```json5
{
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "ro"
        },
        tools: {
          allow: ["read", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
          deny: ["write", "edit", "apply_patch", "exec", "process", "browser"]
        }
      }
    ]
  }
}
```

Nessun accesso al file system (strumenti di messaggistica/sessione abilitati):

```json5
{
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "none"
        },
        tools: {
          allow: ["sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status", "whatsapp", "telegram", "slack", "discord", "gateway"],
          deny: ["read", "write", "edit", "apply_patch", "exec", "process", "browser", "canvas", "nodes", "cron", "gateway", "image"]
        }
      }
    ]
  }
}
```

Esempio: due account WhatsApp → due agenti:

```json5
{
  agents: {
    list: [
      { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
      { id: "work", workspace: "~/.openclaw/workspace-work" }
    ]
  },
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } }
  ],
  channels: {
    whatsapp: {
      accounts: {
        personal: {},
        biz: {},
      }
    }
  }
}
```

<div id="toolsagenttoagent-optional">
  ### `tools.agentToAgent` (opzionale)
</div>

La messaggistica da agente ad agente è opzionale e richiede un’attivazione esplicita:

```json5
{
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"]
    }
  }
}
```

<div id="messagesqueue">
  ### `messages.queue`
</div>

Controlla il comportamento dei messaggi in ingresso quando un&#39;esecuzione di un agente è già in corso.

```json5
{
  messages: {
    queue: {
      mode: "collect", // steer | followup | collect | steer-backlog (steer+backlog ok) | interrupt (queue=steer legacy)
      debounceMs: 1000,
      cap: 20,
      drop: "summarize", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
        discord: "collect",
        imessage: "collect",
        webchat: "collect"
      }
    }
  }
}
```

<div id="messagesinbound">
  ### `messages.inbound`
</div>

Applica un debounce ai messaggi rapidi in ingresso dallo **stesso mittente**, in modo che più messaggi consecutivi vengano combinati in un singolo turno dell’agente. Il debounce è applicato per ogni coppia canale+conversazione e utilizza il messaggio più recente per il threading delle risposte e gli ID di reply.

```json5
{
  messages: {
    inbound: {
      debounceMs: 2000, // 0 per disabilitare
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500
      }
    }
  }
}
```

Note:

* Il debounce dei batch si applica solo ai messaggi di **solo testo**; i contenuti multimediali/gli allegati vengono inviati immediatamente.
* I comandi di controllo (ad es. `/queue`, `/new`) ignorano il debounce e vengono sempre inviati come messaggi separati.

<div id="commands-chat-command-handling">
  ### `commands` (gestione dei comandi chat)
</div>

Controlla come i comandi chat vengono abilitati nei vari connettori.

```json5
{
  commands: {
    native: "auto",         // register native commands when supported (auto)
    text: true,             // parse slash commands in chat messages
    bash: false,            // consenti ! (alias: /bash) (solo host; richiede liste di autorizzati tools.elevated)
    bashForegroundMs: 2000, // bash foreground window (0 backgrounds immediately)
    config: false,          // allow /config (writes to disk)
    debug: false,           // allow /debug (runtime-only overrides)
    restart: false,         // allow /restart + gateway restart tool
    useAccessGroups: true   // enforce access-group allowlists/policies for commands
  }
}
```

Notes:

* I comandi testuali devono essere inviati come messaggi **autonomi** e usare il prefisso `/` (nessun alias in testo semplice).
* `commands.text: false` disabilita l&#39;interpretazione dei messaggi di chat come comandi.
* `commands.native: "auto"` (predefinito) abilita i comandi nativi per Discord/Telegram e li lascia disabilitati per Slack; i canali non supportati rimangono solo testuali.
* Imposta `commands.native: true|false` per forzare tutti i canali, oppure esegui l&#39;override per singolo canale con `channels.discord.commands.native`, `channels.telegram.commands.native`, `channels.slack.commands.native` (bool o `"auto"`). `false` cancella i comandi precedentemente registrati su Discord/Telegram all&#39;avvio; i comandi Slack sono gestiti nell&#39;app Slack.
* `channels.telegram.customCommands` aggiunge voci extra al menu del bot Telegram. I nomi sono normalizzati; i conflitti con i comandi nativi vengono ignorati.
* `commands.bash: true` abilita `! <cmd>` per eseguire comandi shell sull&#39;host (anche `/bash <cmd>` funziona come alias). Richiede `tools.elevated.enabled` e l&#39;inserimento del mittente nella lista di autorizzati in `tools.elevated.allowFrom.<channel>`.
* `commands.bashForegroundMs` controlla per quanto tempo bash attende prima di passare in background. Finché un job bash è in esecuzione, le nuove richieste `! <cmd>` vengono rifiutate (una alla volta).
* `commands.config: true` abilita `/config` (lettura/scrittura di `openclaw.json`).
* `channels.<provider>.configWrites` controlla le modifiche di configurazione avviate da quel canale (predefinito: true). Si applica a `/config set|unset` più le migrazioni automatiche specifiche del provider (cambi di ID dei supergruppi Telegram, cambi di ID dei canali Slack).
* `commands.debug: true` abilita `/debug` (override solo a runtime).
* `commands.restart: true` abilita `/restart` e l&#39;azione di riavvio dello strumento del Gateway.
* `commands.useAccessGroups: false` consente ai comandi di ignorare le liste di autorizzati/policy dei gruppi di accesso.
* I comandi slash e le direttive vengono accettati solo per i **mittenti autorizzati**. L&#39;autorizzazione deriva dalla
  lista di autorizzati/abbinamento del canale più `commands.useAccessGroups`.

<div id="web-whatsapp-web-channel-runtime">
  ### `web` (runtime del canale WhatsApp Web)
</div>

WhatsApp viene eseguito tramite il canale web del Gateway (Baileys Web). Viene avviato automaticamente quando è presente una sessione collegata.
Imposta `web.enabled: false` per mantenerlo disattivato per impostazione predefinita.

```json5
{
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0
    }
  }
}
```

<div id="channelstelegram-bot-transport">
  ### `channels.telegram` (trasporto bot)
</div>

OpenClaw avvia Telegram solo quando esiste una sezione di configurazione `channels.telegram`. Il token del bot viene ricavato da `channels.telegram.botToken` (o `channels.telegram.tokenFile`), con `TELEGRAM_BOT_TOKEN` come fallback per l&#39;account predefinito.
Imposta `channels.telegram.enabled: false` per disabilitare l&#39;avvio automatico.
Il supporto multi-account è definito in `channels.telegram.accounts` (vedi la sezione multi-account sopra). I token dalle variabili d&#39;ambiente si applicano solo all&#39;account predefinito.
Imposta `channels.telegram.configWrites: false` per bloccare le modifiche alla configurazione avviate da Telegram (incluse le migrazioni degli ID dei supergruppi e `/config set|unset`).

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing",                 // pairing | allowlist | open | disabled
      allowFrom: ["tg:123456789"],         // optional; "open" requires ["*"]
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Keep answers brief.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Stay on topic."
            }
          }
        }
      },
      customCommands: [
        { command: "backup", description: "Backup Git" },
        { command: "generate", description: "Crea un'immagine" }
      ],
      historyLimit: 50,                     // include last N group messages as context (0 disables)
      replyToMode: "first",                 // off | first | all
      linkPreview: true,                   // toggle outbound link previews
      streamMode: "partial",               // off | partial | block (draft streaming; separate from block streaming)
      draftChunk: {                        // optional; only for streamMode=block
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph"       // paragraph | newline | sentence
      },
      actions: { reactions: true, sendMessage: true }, // tool action gates (false disables)
      reactionNotifications: "own",   // off | own | all
      mediaMaxMb: 5,
      retry: {                             // outbound retry policy
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1
      },
      network: {                           // transport overrides
        autoSelectFamily: false
      },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook"
    }
  }
}
```

Note sullo streaming delle bozze:

* Usa `sendMessageDraft` di Telegram (fumetto di bozza, non un vero messaggio).
* Richiede **topic nelle chat private** (message&#95;thread&#95;id nei DM; il bot ha i topic abilitati).
* `/reasoning stream` esegue lo streaming del ragionamento nella bozza, quindi invia la risposta finale.
  I valori predefiniti e il comportamento della retry policy sono documentati in [Retry policy](/it/concepts/retry).

<div id="channelsdiscord-bot-transport">
  ### `channels.discord` (trasporto bot)
</div>

Configura il bot Discord impostando il token del bot e, facoltativamente, le regole di gating.
Il supporto multiaccount è disponibile in `channels.discord.accounts` (vedi la sezione multiaccount sopra). I token forniti tramite variabili d&#39;ambiente si applicano solo all&#39;account predefinito.

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8,                          // clamp inbound media size
      allowBots: false,                       // allow bot-authored messages
      actions: {                              // tool action gates (false disables)
        reactions: true,
        stickers: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        voiceStatus: true,
        events: true,
        moderation: false
      },
      replyToMode: "off",                     // off | first | all
      dm: {
        enabled: true,                        // disable all DMs when false
        policy: "pairing",                    // pairing | allowlist | open | disabled
        allowFrom: ["1234567890", "steipete"], // lista di autorizzati DM opzionale ("open" richiede ["*"])
        groupEnabled: false,                 // enable group DMs
        groupChannels: ["openclaw-dm"]          // optional group DM allowlist
      },
      guilds: {
        "123456789012345678": {               // guild id (preferred) or slug
          slug: "friends-of-openclaw",
          requireMention: false,              // per-guild default
          reactionNotifications: "own",       // off | own | all | allowlist
          users: ["987654321098765432"],      // optional per-guild user allowlist
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Short answers only."
            }
          }
        }
      },
      historyLimit: 20,                       // include last N guild messages as context
      textChunkLimit: 2000,                   // optional outbound text chunk size (chars)
      chunkMode: "length",                    // optional chunking mode (length | newline)
      maxLinesPerMessage: 17,                 // soft max lines per message (Discord UI clipping)
      retry: {                                // outbound retry policy
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1
      }
    }
  }
}
```

OpenClaw avvia Discord solo quando esiste una sezione di configurazione `channels.discord`. Il token viene ottenuto da `channels.discord.token`, con `DISCORD_BOT_TOKEN` come fallback per l&#39;account predefinito (a meno che `channels.discord.enabled` sia `false`). Usa `user:<id>` (DM) o `channel:<id>` (canale di una guild) quando specifichi i destinatari per i comandi cron/CLI; gli ID numerici isolati sono ambigui e vengono rifiutati.
Gli slug delle guild sono in minuscolo con gli spazi sostituiti da `-`; le chiavi dei canali usano il nome del canale in formato slug (senza `#` iniziale). Preferisci gli ID delle guild come chiavi per evitare ambiguità in caso di rinomina.
I messaggi generati dal bot vengono ignorati per impostazione predefinita. Abilita con `channels.discord.allowBots` (i messaggi propri sono comunque filtrati per evitare loop di auto-risposta).
Modalità di notifica delle reazioni:

* `off`: nessun evento di reazione.
* `own`: reazioni ai messaggi del bot stesso (predefinito).
* `all`: tutte le reazioni su tutti i messaggi.
* `allowlist`: reazioni da `guilds.<id>.users` su tutti i messaggi (una lista vuota le disabilita).
  Il testo in uscita viene suddiviso in chunk in base a `channels.discord.textChunkLimit` (predefinito 2000). Imposta `channels.discord.chunkMode="newline"` per dividere sulle righe vuote (confini di paragrafo) prima del chunking per lunghezza. I client Discord possono troncare i messaggi molto lunghi in altezza, quindi `channels.discord.maxLinesPerMessage` (predefinito 17) suddivide le risposte lunghe su più righe anche quando sono sotto i 2000 caratteri.
  I valori predefiniti e il comportamento della politica di retry sono documentati in [Retry policy](/it/concepts/retry).

<div id="channelsgooglechat-chat-api-webhook">
  ### `channels.googlechat` (webhook dell&#39;API Chat)
</div>

Google Chat funziona tramite webhook HTTP con autenticazione a livello di applicazione (service account).
Il supporto multi-account è disponibile in `channels.googlechat.accounts` (vedi la sezione sul multi-account sopra). Le variabili d&#39;ambiente si applicano solo all&#39;account predefinito.

```json5
{
  channels: {
    "googlechat": {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url",             // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890",        // opzionale; migliora il rilevamento delle menzioni
      dm: {
        enabled: true,
        policy: "pairing",                // pairing | allowlist | open | disabled
        allowFrom: ["users/1234567890"]   // optional; "open" requires ["*"]
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true }
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20
    }
  }
}
```

Note:

* Il JSON dell’account di servizio può essere inline (`serviceAccount`) o in un file (`serviceAccountFile`).
* Variabili d’ambiente di fallback per l’account predefinito: `GOOGLE_CHAT_SERVICE_ACCOUNT` oppure `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
* `audienceType` + `audience` devono corrispondere alla configurazione di autenticazione del webhook dell’app Chat.
* Usa `spaces/<spaceId>` oppure `users/<userId|email>` quando imposti le destinazioni di recapito.

<div id="channelsslack-socket-mode">
  ### `channels.slack` (socket mode)
</div>

Slack funziona in Socket Mode e richiede sia un bot token che un token dell&#39;app:

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["U123", "U456", "*"], // optional; "open" requires ["*"]
        groupEnabled: false,
        groupChannels: ["G123"]
      },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "Short answers only."
        }
      },
      historyLimit: 50,          // include gli ultimi N messaggi del canale/gruppo come contesto (0 disabilita)
      allowBots: false,
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["U123"],
      replyToMode: "off",           // off | first | all
      thread: {
        historyScope: "thread",     // thread | channel
        inheritParent: false
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true
      },
      textChunkLimit: 4000,
      chunkMode: "length",
      mediaMaxMb: 20
    }
  }
}
```

Il supporto multi-account si trova sotto `channels.slack.accounts` (vedi la sezione multi-account sopra). I token dalle variabili d&#39;ambiente si applicano solo all&#39;account predefinito.

OpenClaw avvia Slack quando il provider è abilitato e entrambi i token sono impostati (tramite config o `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN`). Usa `user:<id>` (DM) o `channel:<id>` quando specifichi le destinazioni di recapito per i comandi cron/CLI.
Imposta `channels.slack.configWrites: false` per bloccare le scritture di configurazione avviate da Slack (incluse le migrazioni degli ID canale e `/config set|unset`).

I messaggi inviati dal bot vengono ignorati per impostazione predefinita. Abilita con `channels.slack.allowBots` o `channels.slack.channels.<id>.allowBots`.

Modalità di notifica delle reaction:

* `off`: nessun evento di reaction.
* `own`: reaction sui messaggi del bot stesso (predefinito).
* `all`: tutte le reaction su tutti i messaggi.
* `allowlist`: reaction da `channels.slack.reactionAllowlist` su tutti i messaggi (una lista vuota le disabilita).

Isolamento delle sessioni dei thread:

* `channels.slack.thread.historyScope` controlla se la cronologia del thread è per thread (`thread`, predefinito) o condivisa tra tutto il canale (`channel`).
* `channels.slack.thread.inheritParent` controlla se le nuove sessioni del thread ereditano la trascrizione del canale padre (predefinito: false).

Gruppi di azioni Slack (regolano le azioni dello strumento `slack`):

| Action group | Default | Notes                        |
| ------------ | ------- | ---------------------------- |
| reactions    | enabled | React + elenco reaction      |
| messages     | enabled | Leggi/invia/modifica/elimina |
| pins         | enabled | Fissa/sblocca/elenca         |
| memberInfo   | enabled | Info membro                  |
| emojiList    | enabled | Elenco emoji personalizzate  |

<div id="channelsmattermost-bot-token">
  ### `channels.mattermost` (bot token)
</div>

Mattermost viene fornito come plugin e non è incluso nell&#39;installazione core.
Installalo prima: `openclaw plugins install @openclaw/mattermost` (oppure `./extensions/mattermost` da una checkout Git).

Mattermost richiede un bot token e l’URL di base del tuo server:

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      textChunkLimit: 4000,
      chunkMode: "length"
    }
  }
}
```

OpenClaw avvia Mattermost quando l&#39;account è configurato (bot token + URL di base) e abilitato. Il token + URL di base sono ricavati da `channels.mattermost.botToken` + `channels.mattermost.baseUrl` oppure da `MATTERMOST_BOT_TOKEN` + `MATTERMOST_URL` per l&#39;account predefinito (a meno che `channels.mattermost.enabled` sia `false`).

Modalità di chat:

* `oncall` (predefinita): risponde ai messaggi del canale solo quando viene menzionato con @.
* `onmessage`: risponde a ogni messaggio del canale.
* `onchar`: risponde quando un messaggio inizia con un prefisso di trigger (`channels.mattermost.oncharPrefixes`, predefinito `[">", "!"]`).

Controllo di accesso:

* DM predefiniti: `channels.mattermost.dmPolicy="pairing"` (i mittenti sconosciuti ricevono un codice di abbinamento).
* DM pubblici: `channels.mattermost.dmPolicy="open"` (open consente di accettare messaggi da chiunque) più `channels.mattermost.allowFrom=["*"]`.
* Gruppi: `channels.mattermost.groupPolicy="allowlist"` per impostazione predefinita (accesso limitato tramite menzione). Usa `channels.mattermost.groupAllowFrom` per restringere i mittenti autorizzati.

Il supporto multi-account si trova sotto `channels.mattermost.accounts` (vedi la sezione multi-account sopra). Le variabili d&#39;ambiente si applicano solo all&#39;account predefinito.
Usa `channel:<id>` o `user:<id>` (o `@username`) quando specifichi le destinazioni di recapito; gli id senza prefisso vengono trattati come id di canale.

<div id="channelssignal-signal-cli">
  ### `channels.signal` (signal-cli)
</div>

Le reazioni di Signal possono emettere eventi di sistema (strumentazione condivisa per le reazioni):

```json5
{
  channels: {
    signal: {
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50 // include gli ultimi N messaggi di gruppo come contesto (0 disabilita)
    }
  }
}
```

Modalità di notifica delle reazioni:

* `off`: nessun evento di reazione.
* `own`: reazioni ai messaggi inviati dal bot stesso (predefinito).
* `all`: tutte le reazioni su tutti i messaggi.
* `allowlist`: reazioni da `channels.signal.reactionAllowlist` su tutti i messaggi (un elenco vuoto la disattiva).

<div id="channelsimessage-imsg-cli">
  ### `channels.imessage` (imsg CLI)
</div>

OpenClaw avvia `imsg rpc` (JSON-RPC su stdio). Non richiede alcun demone o porta.

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host", // SCP per allegati remoti quando si utilizza un wrapper SSH
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50,    // include last N group messages as context (0 disables)
      includeAttachments: false,
      mediaMaxMb: 16,
      service: "auto",
      region: "US"
    }
  }
}
```

Il supporto multi-account si configura in `channels.imessage.accounts` (vedi la sezione multi-account sopra).

Note:

* Richiede l’accesso completo al disco al database di Messaggi.
* Il primo invio richiederà l’autorizzazione per l’automazione di Messaggi.
* Preferisci le destinazioni `chat_id:&lt;id&gt;`. Usa `imsg chats --limit 20` per elencare le chat.
* `channels.imessage.cliPath` può puntare a uno script wrapper (ad es. `ssh` verso un altro Mac che esegue `imsg rpc`); usa chiavi SSH per evitare prompt della password.
* Per i wrapper SSH remoti, imposta `channels.imessage.remoteHost` per recuperare gli allegati tramite SCP quando `includeAttachments` è abilitato.

Esempio di wrapper:

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

<div id="agentsdefaultsworkspace">
  ### `agents.defaults.workspace`
</div>

Imposta la **singola directory globale dello spazio di lavoro** utilizzata dagli agenti per le operazioni sui file.

Valore predefinito: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } }
}
```

Se `agents.defaults.sandbox` è abilitato, le sessioni non principali possono sovrascrivere questa impostazione con propri spazi di lavoro per scope sotto `agents.defaults.sandbox.workspaceRoot`.

<div id="agentsdefaultsreporoot">
  ### `agents.defaults.repoRoot`
</div>

Radice facoltativa del repository da mostrare nella riga Runtime del prompt di sistema. Se non impostata, OpenClaw
prova a rilevare una directory `.git` risalendo dallo spazio di lavoro (e dalla directory
di lavoro corrente). Il percorso deve esistere per poter essere utilizzato.

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } }
}
```

<div id="agentsdefaultsskipbootstrap">
  ### `agents.defaults.skipBootstrap`
</div>

Disabilita la creazione automatica dei file di bootstrap dello spazio di lavoro (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md` e `BOOTSTRAP.md`).

Utilizzalo per deployment preconfigurati in cui i file dello spazio di lavoro provengono da un repository.

```json5
{
  agents: { defaults: { skipBootstrap: true } }
}
```

<div id="agentsdefaultsbootstrapmaxchars">
  ### `agents.defaults.bootstrapMaxChars`
</div>

Numero massimo di caratteri di ciascun file di bootstrap dello spazio di lavoro inserito nel prompt di sistema
prima del troncamento. Valore predefinito: `20000`.

Quando un file supera questo limite, OpenClaw registra un avviso e inserisce una versione troncata
dell&#39;inizio e della fine (head/tail) con un marcatore.

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } }
}
```

<div id="agentsdefaultsusertimezone">
  ### `agents.defaults.userTimezone`
</div>

Imposta il fuso orario dell’utente per il **contesto del system prompt** (non per i timestamp
negli envelope dei messaggi). Se non impostato, OpenClaw utilizza il fuso orario dell’host in fase di esecuzione.

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } }
}
```

<div id="agentsdefaultstimeformat">
  ### `agents.defaults.timeFormat`
</div>

Controlla il **formato dell&#39;ora** visualizzato nella sezione Data e ora correnti del prompt di sistema.
Valore predefinito: `auto` (in base alle preferenze del sistema operativo).

```json5
{
  agents: { defaults: { timeFormat: "auto" } } // auto | 12 | 24
}
```

<div id="messages">
  ### `messages`
</div>

Controlla i prefissi in ingresso/uscita e le reazioni di ack opzionali.
Vedi [Messages](/it/concepts/messages) per l&#39;accodamento, le sessioni e il contesto di streaming.

```json5
{
  messages: {
    responsePrefix: "🦞", // oppure "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions",
    removeAckAfterReply: false
  }
}
```

`responsePrefix` viene applicato a **tutte le risposte in uscita** (riepiloghi degli strumenti, streaming a blocchi, risposte finali) su tutti i canali, a meno che non sia già presente.

Se `messages.responsePrefix` non è impostato, per impostazione predefinita non viene applicato alcun prefisso. Le risposte nella chat con sé stessi su WhatsApp sono l&#39;eccezione: per impostazione predefinita usano `[{identity.name}]` quando definito, altrimenti
`[openclaw]`, così le conversazioni sullo stesso telefono restano leggibili.
Impostalo su `"auto"` per ricavare `[{identity.name}]` per l&#39;agente instradato (quando definito).

<div id="template-variables">
  #### Variabili di template
</div>

La stringa `responsePrefix` può includere variabili di template che vengono risolte dinamicamente:

| Variabile         | Descrizione                            | Esempio                         |
| ----------------- | -------------------------------------- | ------------------------------- |
| `{model}`         | Nome breve del modello                 | `claude-opus-4-5`, `gpt-4o`     |
| `{modelFull}`     | Identificatore completo del modello    | `anthropic/claude-opus-4-5`     |
| `{provider}`      | Nome del provider                      | `anthropic`, `openai`           |
| `{thinkingLevel}` | Livello di ragionamento corrente       | `high`, `low`, `off`            |
| `{identity.name}` | Nome dell&#39;identità dell&#39;agente | (uguale alla modalità `"auto"`) |

Le variabili non fanno distinzione tra maiuscole e minuscole (`{MODEL}` = `{model}`). `{think}` è un alias di `{thinkingLevel}`.
Le variabili non risolte rimangono come testo letterale.

```json5
{
  messages: {
    responsePrefix: "[{model} | think:{thinkingLevel}]"
  }
}
```

Esempio di output: `[claude-opus-4-5 | think:high] Ecco la mia risposta...`

Il prefisso dei messaggi in ingresso per WhatsApp è configurato tramite
`channels.whatsapp.messagePrefix` (deprecato: `messages.messagePrefix`). Il valore
predefinito rimane **invariato**: `"[openclaw]"` quando `channels.whatsapp.allowFrom`
è vuoto, altrimenti `""` (nessun prefisso). Quando utilizzi `"[openclaw]"`, OpenClaw
userà invece `[{identity.name}]` quando l&#39;agente instradato ha `identity.name`
impostato.

`ackReaction` invia, nel limite del possibile, una reazione con emoji per riconoscere i
messaggi in ingresso sui canali che supportano le reazioni
(Slack/Discord/Telegram/Google Chat). Il valore predefinito è `identity.emoji`
dell’agente attivo, se impostato, altrimenti `"👀"`. Impostalo su `""` per disabilitarlo.

`ackReactionScope` controlla quando vengono inviate le reazioni:

* `group-mentions` (predefinito): solo quando un gruppo/stanza richiede menzioni **e** il bot è stato menzionato
* `group-all`: tutti i messaggi di gruppo/stanza
* `direct`: solo messaggi diretti
* `all`: tutti i messaggi

`removeAckAfterReply` rimuove la reazione di ack del bot dopo l&#39;invio di una risposta
(solo Slack/Discord/Telegram/Google Chat). Valore predefinito: `false`.

<div id="messagestts">
  #### `messages.tts`
</div>

Abilita la sintesi vocale (text-to-speech) per le risposte in uscita. Quando è attiva, OpenClaw genera un audio
utilizzando ElevenLabs o OpenAI e lo allega alle risposte. Telegram utilizza note vocali in formato Opus; gli altri canali inviano audio in formato MP3.

```json5
{
  messages: {
    tts: {
      auto: "always", // off | always | inbound | tagged
      mode: "final", // final | all (include tool/block replies)
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true
      },
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0
        }
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy"
      }
    }
  }
}
```

Note:

* `messages.tts.auto` controlla l&#39;auto‑TTS (`off`, `always`, `inbound`, `tagged`).
* `/tts off|always|inbound|tagged` imposta la modalità automatica per singola sessione (sovrascrive la configurazione).
* `messages.tts.enabled` è legacy; `doctor` la migra a `messages.tts.auto`.
* `prefsPath` memorizza gli override locali (provider/limit/summarize).
* `maxTextLength` è un limite rigido per l&#39;input TTS; i riassunti vengono troncati per rientrare.
* `summaryModel` sovrascrive `agents.defaults.model.primary` per il riassunto automatico.
  * Accetta `provider/model` o un alias da `agents.defaults.models`.
* `modelOverrides` abilita override guidati dal modello come i tag `[[tts:...]]` (attivo per impostazione predefinita).
* `/tts limit` e `/tts summary` controllano le impostazioni di riassunto per utente.
* I valori `apiKey` effettuano il fallback su `ELEVENLABS_API_KEY`/`XI_API_KEY` e `OPENAI_API_KEY`.
* `elevenlabs.baseUrl` sovrascrive l&#39;URL di base dell&#39;API ElevenLabs.
* `elevenlabs.voiceSettings` supporta `stability`/`similarityBoost`/`style` (0..1),
  `useSpeakerBoost` e `speed` (0.5..2.0).

<div id="talk">
  ### `talk`
</div>

Valori predefiniti per la modalità Talk (macOS/iOS/Android). Gli ID voce usano `ELEVENLABS_VOICE_ID` o `SAG_VOICE_ID` come valore predefinito quando non sono impostati.
`apiKey` usa `ELEVENLABS_API_KEY` (o il profilo shell del Gateway) come valore predefinito quando non è impostata.
`voiceAliases` permette alle direttive Talk di usare nomi descrittivi (ad es. `"voice":"Clawd"`).

```json5
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    voiceAliases: {
      Clawd: "EXAVITQu4vr4xnSDxMaL",
      Roger: "CwhRBWXzGAHq8TQ4Fs17"
    },
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true
  }
}
```

<div id="agentsdefaults">
  ### `agents.defaults`
</div>

Controlla il runtime incorporato dell&#39;agente (modello/ragionamento/verbosità/timeouts).
`agents.defaults.models` definisce il catalogo dei modelli configurati (e funge da lista di autorizzati per `/model`).
`agents.defaults.model.primary` imposta il modello predefinito; `agents.defaults.model.fallbacks` sono i failover globali.
`agents.defaults.imageModel` è opzionale ed è **utilizzato solo se il modello primario non supporta l&#39;input di immagini**.
Ogni voce di `agents.defaults.models` può includere:

* `alias` (scorciatoia opzionale per il modello, ad es. `/opus`).
* `params` (parametri API opzionali specifici del provider, passati alla richiesta del modello).

`params` è applicato anche alle esecuzioni in streaming (agente incorporato + compattazione). Le chiavi supportate al momento sono: `temperature`, `maxTokens`. Questi vengono uniti alle opzioni specificate al momento della chiamata; i valori forniti dal chiamante hanno la precedenza. `temperature` è un controllo avanzato: lascialo non impostato a meno che tu non conosca i valori predefiniti del modello e abbia effettivamente bisogno di modificarli.

Esempio:

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-sonnet-4-5-20250929": {
          params: { temperature: 0.6 }
        },
        "openai/gpt-5.2": {
          params: { maxTokens: 8192 }
        }
      }
    }
  }
}
```

I modelli Z.AI GLM-4.x abilitano automaticamente la modalità di ragionamento, a meno che tu non:

* imposti `--thinking off`, oppure
* definisca direttamente `agents.defaults.models["zai/<model>"].params.thinking`.

OpenClaw include anche alcune abbreviazioni di alias integrate. I valori predefiniti si applicano solo quando il modello
è già presente in `agents.defaults.models`:

* `opus` -&gt; `anthropic/claude-opus-4-5`
* `sonnet` -&gt; `anthropic/claude-sonnet-4-5`
* `gpt` -&gt; `openai/gpt-5.2`
* `gpt-mini` -&gt; `openai/gpt-5-mini`
* `gemini` -&gt; `google/gemini-3-pro-preview`
* `gemini-flash` -&gt; `google/gemini-3-flash-preview`

Se configuri tu un alias con lo stesso nome (ignorando maiuscole/minuscole), il tuo valore ha la precedenza (i default non sovrascrivono mai).

Esempio: Opus 4.5 come modello primario con MiniMax M2.1 come fallback (MiniMax hosted):

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" }
      },
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: ["minimax/MiniMax-M2.1"]
      }
    }
  }
}
```

Autenticazione MiniMax: imposta la variabile d&#39;ambiente `MINIMAX_API_KEY` oppure configura `models.providers.minimax`.

<div id="agentsdefaultsclibackends-cli-fallback">
  #### `agents.defaults.cliBackends` (fallback CLI)
</div>

Backend CLI opzionali per esecuzioni di fallback solo testuali (nessuna chiamata a tool). Sono utili come
percorso di backup quando i provider API falliscono. L’inoltro delle immagini è supportato quando configuri
un `imageArg` che accetta percorsi di file.

Note:

* I backend CLI sono **text-first**; i tool sono sempre disabilitati.
* Le sessioni sono supportate quando `sessionArg` è impostato; gli ID di sessione sono mantenuti per backend.
* Per `claude-cli` sono preconfigurati dei valori predefiniti. Sovrascrivi il percorso del comando se PATH è minimale
  (launchd/systemd).

Esempio:

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude"
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          modelArg: "--model",
          sessionArg: "--session",
          sessionMode: "existing",
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat"
        }
      }
    }
  }
}
```

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "anthropic/claude-sonnet-4-1": { alias: "Sonnet" },
        "openrouter/deepseek/deepseek-r1:free": {},
        "zai/glm-4.7": {
          alias: "GLM",
          params: {
            thinking: {
              type: "enabled",
              clear_thinking: false
            }
          }
        }
      },
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: [
          "openrouter/deepseek/deepseek-r1:free",
          "openrouter/meta-llama/llama-3.3-70b-instruct:free"
        ]
      },
      imageModel: {
        primary: "openrouter/qwen/qwen-2.5-vl-72b-instruct:free",
        fallbacks: [
          "openrouter/google/gemini-2.0-flash-vision:free"
        ]
      },
      thinkingDefault: "low",
      verboseDefault: "off",
      elevatedDefault: "on",
      timeoutSeconds: 600,
      mediaMaxMb: 5,
      heartbeat: {
        every: "30m",
        target: "last"
      },
      maxConcurrent: 3,
      subagents: {
        model: "minimax/MiniMax-M2.1",
        maxConcurrent: 1,
        archiveAfterMinutes: 60
      },
      exec: {
        backgroundMs: 10000,
        timeoutSec: 1800,
        cleanupMs: 1800000
      },
      contextTokens: 200000
    }
  }
}
```

<div id="agentsdefaultscontextpruning-tool-result-pruning">
  #### `agents.defaults.contextPruning` (pruning dei risultati degli strumenti)
</div>

`agents.defaults.contextPruning` esegue il pruning dei **vecchi risultati degli strumenti** dal contesto in memoria subito prima che una richiesta venga inviata all&#39;LLM.
**Non** modifica la cronologia della sessione su disco (`*.jsonl` rimane completa).

È pensato per ridurre l’uso di token per agenti loquaci che accumulano nel tempo output di strumenti molto grandi.

A livello generale:

* Non tocca mai i messaggi utente/assistant.
* Protegge gli ultimi messaggi dell’assistant indicati da `keepLastAssistants` (nessun risultato di strumenti dopo quel punto viene eliminato).
* Protegge il prefisso di bootstrap (nulla prima del primo messaggio utente viene eliminato).
* Modalità:
  * `adaptive`: applica un soft-trim ai risultati degli strumenti sovradimensionati (mantiene testa/coda) quando il rapporto di contesto stimato supera `softTrimRatio`.
    Poi esegue una cancellazione hard dei risultati degli strumenti idonei più vecchi quando il rapporto di contesto stimato supera `hardClearRatio` **e**
    c’è abbastanza massa eliminabile di risultati degli strumenti (`minPrunableToolChars`).
  * `aggressive`: sostituisce sempre i risultati degli strumenti idonei prima della soglia con `hardClear.placeholder` (nessun controllo sul rapporto).

Soft vs hard pruning (cosa cambia nel contesto inviato all’LLM):

* **Soft-trim**: solo per risultati degli strumenti *sovradimensionati*. Mantiene l’inizio + la fine e inserisce `...` nel mezzo.
  * Prima: `toolResult("…very long output…")`
  * Dopo: `toolResult("HEAD…\n...\n…TAIL\n\n[Tool result trimmed: …]")`
* **Hard-clear**: sostituisce l’intero risultato dello strumento con il placeholder.
  * Prima: `toolResult("…very long output…")`
  * Dopo: `toolResult("[Old tool result content cleared]")`

Note / limitazioni attuali:

* I risultati degli strumenti che contengono **blocchi immagine vengono ignorati** (al momento non vengono mai ridotti/cancellati).
* Il “rapporto di contesto” stimato si basa sui **caratteri** (approssimato), non sui token esatti.
* Se la sessione non contiene ancora almeno `keepLastAssistants` messaggi dell’assistant, il pruning non viene eseguito.
* In modalità `aggressive`, `hardClear.enabled` viene ignorato (i risultati degli strumenti idonei sono sempre sostituiti con `hardClear.placeholder`).

Default (adaptive):

```json5
{
  agents: { defaults: { contextPruning: { mode: "adaptive" } } }
}
```

Per disattivare:

```json5
{
  agents: { defaults: { contextPruning: { mode: "off" } } }
}
```

Impostazioni predefinite (quando `mode` è `"adaptive"` o `"aggressive"`):

* `keepLastAssistants`: `3`
* `softTrimRatio`: `0.3` (solo in modalità adaptive)
* `hardClearRatio`: `0.5` (solo in modalità adaptive)
* `minPrunableToolChars`: `50000` (solo in modalità adaptive)
* `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }` (solo in modalità adaptive)
* `hardClear`: `{ enabled: true, placeholder: "[Contenuto precedente del risultato dello strumento eliminato]" }`

Esempio (aggressive, minimale):

```json5
{
  agents: { defaults: { contextPruning: { mode: "aggressive" } } }
}
```

Esempio (ottimizzato in modo adattivo):

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "adaptive",
        keepLastAssistants: 3,
        softTrimRatio: 0.3,
        hardClearRatio: 0.5,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { enabled: true, placeholder: "[Old tool result content cleared]" },
        // Opzionale: limita la potatura a strumenti specifici (deny ha la precedenza; supporta wildcard "*")
        tools: { deny: ["browser", "canvas"] },
      }
    }
  }
}
```

Consulta [/concepts/session-pruning](/it/concepts/session-pruning) per i dettagli sul comportamento.

<div id="agentsdefaultscompaction-reserve-headroom-memory-flush">
  #### `agents.defaults.compaction` (riserva di margine + memory flush)
</div>

`agents.defaults.compaction.mode` seleziona la strategia di compattazione/sintesi. Il valore predefinito è `default`; imposta `safeguard` per abilitare la sintesi a blocchi per cronologie molto lunghe. Vedi [/concepts/compaction](/it/concepts/compaction).

`agents.defaults.compaction.reserveTokensFloor` impone un valore minimo di `reserveTokens`
per la compattazione Pi (predefinito: `20000`). Impostalo a `0` per disabilitare questo valore minimo.

`agents.defaults.compaction.memoryFlush` esegue un turno dell’agente **silenzioso** prima
dell’auto‑compattazione, istruendo il modello a memorizzare ricordi durevoli su disco (ad es.
`memory/YYYY-MM-DD.md`). Si attiva quando la stima dei token della sessione supera una
soglia soft al di sotto del limite di compattazione.

Valori predefiniti legacy:

* `memoryFlush.enabled`: `true`
* `memoryFlush.softThresholdTokens`: `4000`
* `memoryFlush.prompt` / `memoryFlush.systemPrompt`: valori predefiniti integrati con `NO_REPLY`
* Nota: il memory flush viene saltato quando lo spazio di lavoro della sessione è in sola lettura
  (`agents.defaults.sandbox.workspaceAccess: "ro"` o `"none"`).

Esempio (ottimizzato):

```json5
{
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard",
        reserveTokensFloor: 24000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 6000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Scrivi eventuali note durature in memory/YYYY-MM-DD.md; rispondi con NO_REPLY se non c'è nulla da memorizzare."
        }
      }
    }
  }
}
```

Streaming a blocchi:

* `agents.defaults.blockStreamingDefault`: `"on"`/`"off"` (predefinito off).
* Override a livello di canale: `*.blockStreaming` (e varianti per account) per forzare lo streaming a blocchi on/off.
  I canali diversi da Telegram richiedono un esplicito `*.blockStreaming: true` per abilitare le risposte a blocchi.
* `agents.defaults.blockStreamingBreak`: `"text_end"` o `"message_end"` (predefinito: text&#95;end).
* `agents.defaults.blockStreamingChunk`: suddivisione soft in chunk per i blocchi in streaming. Valore predefinito:
  800–1200 caratteri; privilegia le interruzioni di paragrafo (`\n\n`), poi le newline, poi le frasi.
  Esempio:
  ```json5
  {
    agents: { defaults: { blockStreamingChunk: { minChars: 800, maxChars: 1200 } } }
  }
  ```
* `agents.defaults.blockStreamingCoalesce`: unisce i blocchi in streaming prima dell’invio.
  Predefinito `{ idleMs: 1000 }` ed eredita `minChars` da `blockStreamingChunk`,
  con `maxChars` limitato dal limite di testo del canale. Signal/Slack/Discord/Google Chat usano
  `minChars: 1500` come predefinito, a meno che non venga effettuato un override.
  Override di canale: `channels.whatsapp.blockStreamingCoalesce`, `channels.telegram.blockStreamingCoalesce`,
  `channels.discord.blockStreamingCoalesce`, `channels.slack.blockStreamingCoalesce`, `channels.mattermost.blockStreamingCoalesce`,
  `channels.signal.blockStreamingCoalesce`, `channels.imessage.blockStreamingCoalesce`, `channels.msteams.blockStreamingCoalesce`,
  `channels.googlechat.blockStreamingCoalesce`
  (e varianti per account).
* `agents.defaults.humanDelay`: pausa casuale tra le **risposte a blocchi** dopo la prima.
  Modalità: `off` (predefinito), `natural` (800–2500ms), `custom` (usa `minMs`/`maxMs`).
  Override per-agente: `agents.list[].humanDelay`.
  Esempio:
  ```json5
  {
    agents: { defaults: { humanDelay: { mode: "natural" } } }
  }
  ```

Vedi [/concepts/streaming](/it/concepts/streaming) per dettagli su comportamento e chunking.

Indicatori di digitazione:

* `agents.defaults.typingMode`: `"never" | "instant" | "thinking" | "message"`. Predefinito
  `instant` per chat dirette / menzioni e `message` per chat di gruppo senza menzione.
* `session.typingMode`: override per sessione della modalità.
* `agents.defaults.typingIntervalSeconds`: frequenza con cui viene aggiornato il segnale di digitazione (predefinito: 6s).
* `session.typingIntervalSeconds`: override per sessione per l&#39;intervallo di aggiornamento.
  Vedi [/concepts/typing-indicators](/it/concepts/typing-indicators) per i dettagli sul comportamento.

`agents.defaults.model.primary` dovrebbe essere impostato come `provider/model` (ad es. `anthropic/claude-opus-4-5`).
Gli alias provengono da `agents.defaults.models.*.alias` (ad es. `Opus`).
Se ometti il provider, OpenClaw al momento assume `anthropic` come fallback
temporaneo per la deprecazione.
I modelli Z.AI sono disponibili come `zai/<model>` (ad es. `zai/glm-4.7`) e richiedono
`ZAI_API_KEY` (o il legacy `Z_AI_API_KEY`) nell&#39;ambiente.

`agents.defaults.heartbeat` configura le esecuzioni periodiche di heartbeat:

* `every`: stringa di durata (`ms`, `s`, `m`, `h`); unità predefinita minuti. Valore predefinito:
  `30m`. Imposta `0m` per disabilitare.
* `model`: modello di override opzionale per le esecuzioni di heartbeat (`provider/model`).
* `includeReasoning`: quando `true`, gli heartbeat consegneranno anche il messaggio separato `Reasoning:` quando disponibile (stessa forma di `/reasoning on`). Predefinito: `false`.
* `session`: chiave di sessione opzionale per controllare in quale sessione gira l&#39;heartbeat. Predefinito: `main`.
* `to`: override opzionale del destinatario (id specifico del canale, ad es. E.164 per WhatsApp, chat id per Telegram).
* `target`: canale di consegna opzionale (`last`, `whatsapp`, `telegram`, `discord`, `slack`, `msteams`, `signal`, `imessage`, `none`). Predefinito: `last`.
* `prompt`: override opzionale per il corpo dell&#39;heartbeat (predefinito: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`). Gli override sono inviati alla lettera; includi una riga `Read HEARTBEAT.md` se vuoi ancora che il file venga letto.
* `ackMaxChars`: numero massimo di caratteri consentiti dopo `HEARTBEAT_OK` prima della consegna (predefinito: 300).

Heartbeat per singolo agente:

* Imposta `agents.list[].heartbeat` per abilitare o sovrascrivere le impostazioni di heartbeat per un agente specifico.
* Se una qualsiasi voce di agente definisce `heartbeat`, **solo quegli agenti** eseguono heartbeat; i valori predefiniti
  diventano la base condivisa per quegli agenti.

Gli heartbeat eseguono turni completi dell&#39;agente. Intervalli più brevi consumano più token; presta attenzione
a `every`, mantieni `HEARTBEAT.md` molto ridotto e/o scegli un `model` più economico.

`tools.exec` configura i valori predefiniti per l&#39;esecuzione in background:

* `backgroundMs`: tempo prima del passaggio automatico in background (ms, predefinito 10000)
* `timeoutSec`: terminazione automatica dopo questo tempo di esecuzione (secondi, predefinito 1800)
* `cleanupMs`: per quanto tempo mantenere in memoria le sessioni concluse (ms, predefinito 1800000)
* `notifyOnExit`: accoda un evento di sistema + richiede un heartbeat quando un exec in background termina (predefinito true)
* `applyPatch.enabled`: abilita l&#39;esperimentale `apply_patch` (solo OpenAI/OpenAI Codex; predefinito false)
* `applyPatch.allowModels`: lista di autorizzati opzionale di id modello (ad es. `gpt-5.2` o `openai/gpt-5.2`)
  Nota: `applyPatch` è solo sotto `tools.exec`.

`tools.web` configura gli strumenti di ricerca e recupero web:

* `tools.web.search.enabled` (predefinito: true quando la chiave è presente)
* `tools.web.search.apiKey` (consigliato: impostare tramite `openclaw configure --section web`, oppure usare la variabile d&#39;ambiente `BRAVE_API_KEY`)
* `tools.web.search.maxResults` (1–10, predefinito 5)
* `tools.web.search.timeoutSeconds` (predefinito 30)
* `tools.web.search.cacheTtlMinutes` (predefinito 15)
* `tools.web.fetch.enabled` (predefinito true)
* `tools.web.fetch.maxChars` (predefinito 50000)
* `tools.web.fetch.timeoutSeconds` (predefinito 30)
* `tools.web.fetch.cacheTtlMinutes` (predefinito 15)
* `tools.web.fetch.userAgent` (override opzionale)
* `tools.web.fetch.readability` (predefinito true; disabilita per usare solo il cleanup HTML di base)
* `tools.web.fetch.firecrawl.enabled` (predefinito true quando è impostata una API key)
* `tools.web.fetch.firecrawl.apiKey` (opzionale; valore predefinito: `FIRECRAWL_API_KEY`)
* `tools.web.fetch.firecrawl.baseUrl` (predefinito https://api.firecrawl.dev)
* `tools.web.fetch.firecrawl.onlyMainContent` (predefinito true)
* `tools.web.fetch.firecrawl.maxAgeMs` (opzionale)
* `tools.web.fetch.firecrawl.timeoutSeconds` (opzionale)

`tools.media` configura la comprensione dei media in ingresso (immagine/audio/video):

* `tools.media.models`: elenco condiviso di modelli (con tag di funzionalità; usato dopo gli elenchi specifici per capacità).
* `tools.media.concurrency`: numero massimo di esecuzioni concorrenti delle funzionalità (predefinito 2).
* `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  * `enabled`: interruttore di disattivazione (opt-out) (predefinito true quando i modelli sono configurati).
  * `prompt`: override facoltativo del prompt (image/video aggiungono automaticamente un hint `maxChars`).
  * `maxChars`: numero massimo di caratteri in output (predefinito 500 per image/video; non impostato per audio).
  * `maxBytes`: dimensione massima dei media da inviare (predefiniti: image 10MB, audio 20MB, video 50MB).
  * `timeoutSeconds`: timeout della richiesta (predefiniti: image 60s, audio 60s, video 120s).
  * `language`: suggerimento facoltativo per l&#39;audio.
  * `attachments`: policy per gli allegati (`mode`, `maxAttachments`, `prefer`).
  * `scope`: limitazione opzionale (vale la prima corrispondenza) con `match.channel`, `match.chatType` o `match.keyPrefix`.
  * `models`: elenco ordinato di voci di modello; in caso di errori o media troppo grande si passa alla voce successiva.
* Ogni voce `models[]`:
  * Voce provider (`type: "provider"` o omesso):
    * `provider`: id del provider API (`openai`, `anthropic`, `google`/`gemini`, `groq`, ecc).
    * `model`: override dell&#39;id del modello (obbligatorio per image; predefinito `gpt-4o-mini-transcribe`/`whisper-large-v3-turbo` per i provider audio e `gemini-3-flash-preview` per video).
    * `profile` / `preferredProfile`: selezione del profilo di autenticazione.
  * Voce CLI (`type: "cli"`):
    * `command`: eseguibile da eseguire.
    * `args`: argomenti con template (supporta `{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}`, ecc).
  * `capabilities`: elenco facoltativo (`image`, `audio`, `video`) per limitare una voce condivisa. Predefiniti se omesso: `openai`/`anthropic`/`minimax` → image, `google` → image+audio+video, `groq` → audio.
  * `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language` possono essere sovrascritti per singola voce.

Se non sono configurati modelli (o `enabled: false`), l&#39;analisi viene saltata; il modello riceve comunque gli allegati originali.

L&#39;autenticazione dei provider segue l&#39;ordine standard per l&#39;autenticazione dei modelli (profili di auth, variabili d&#39;ambiente come `OPENAI_API_KEY`/`GROQ_API_KEY`/`GEMINI_API_KEY`, oppure `models.providers.*.apiKey`).

Esempio:

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        scope: {
          default: "deny",
          rules: [{ action: "allow", match: { chatType: "direct" } }]
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          { type: "cli", command: "whisper", args: ["--model", "base", "{{MediaPath}}"] }
        ]
      },
      video: {
        enabled: true,
        maxBytes: 52428800,
        models: [{ provider: "google", model: "gemini-3-flash-preview" }]
      }
    }
  }
}
```

`agents.defaults.subagents` configura i valori predefiniti dei sottoagenti:

* `model`: modello predefinito per i sottoagenti generati (stringa o `{ primary, fallbacks }`). Se omesso, i sottoagenti ereditano il modello del chiamante, a meno che non venga sovrascritto per agente o per chiamata.
* `maxConcurrent`: numero massimo di esecuzioni concorrenti di sottoagenti (predefinito 1)
* `archiveAfterMinutes`: archivia automaticamente le sessioni dei sottoagenti dopo N minuti (predefinito 60; imposta `0` per disabilitare)
* Criterio degli strumenti per singolo sottoagente: `tools.subagents.tools.allow` / `tools.subagents.tools.deny` (ha la precedenza `deny`)

`tools.profile` imposta una **lista di autorizzati di base per gli strumenti** prima di `tools.allow`/`tools.deny`:

* `minimal`: solo `session_status`
* `coding`: `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
* `messaging`: `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
* `full`: nessuna restrizione (equivale a non impostato)

Override per singolo agente: `agents.list[].tools.profile`.

Esempio (solo messaggistica per impostazione predefinita, consente anche gli strumenti Slack + Discord):

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"]
  }
}
```

Esempio (profilo di coding, con exec/process negati ovunque):

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"]
  }
}
```

`tools.byProvider` ti consente di **limitare ulteriormente** gli strumenti per provider specifici (o per un singolo `provider/model`).
Override per agente: `agents.list[].tools.byProvider`.

Ordine: profilo base → profilo del provider → policy allow/deny.
Le chiavi provider accettano sia `provider` (ad es. `google-antigravity`) sia `provider/model`
(ad es. `openai/gpt-5.2`).

Esempio (mantieni il profilo di codifica globale, ma strumenti minimi per Google Antigravity):

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" }
    }
  }
}
```

Esempio (lista di autorizzati specifica per provider/modello):

```json5
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] }
    }
  }
}
```

`tools.allow` / `tools.deny` configurano una policy globale di allow/deny per i tool (deny ha la precedenza).
Il matching è case-insensitive e supporta il carattere jolly `*` (`"*"` significa tutti i tool).
Questa policy viene applicata anche quando la Docker sandbox è **off**.

Esempio (disattiva browser/canvas ovunque):

```json5
{
  tools: { deny: ["browser", "canvas"] }
}
```

I gruppi di strumenti (abbreviazioni) funzionano nelle policy degli strumenti **globali** e **per agente**:

* `group:runtime`: `exec`, `bash`, `process`
* `group:fs`: `read`, `write`, `edit`, `apply_patch`
* `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory`: `memory_search`, `memory_get`
* `group:web`: `web_search`, `web_fetch`
* `group:ui`: `browser`, `canvas`
* `group:automation`: `cron`, `gateway`
* `group:messaging`: `message`
* `group:nodes`: `nodes`
* `group:openclaw`: tutti gli strumenti OpenClaw integrati (esclude i plugin dei provider)

`tools.elevated` controlla l’accesso `exec` con privilegi elevati (host):

* `enabled`: consente la modalità con privilegi elevati (predefinito true)
* `allowFrom`: liste di autorizzati per canale (vuoto = disabilitato)
  * `whatsapp`: numeri in formato E.164
  * `telegram`: ID chat o username
  * `discord`: ID utente o username (se omesso, esegue il fallback a `channels.discord.dm.allowFrom`)
  * `signal`: numeri in formato E.164
  * `imessage`: handle/ID chat
  * `webchat`: ID sessione o username

Esempio:

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15555550123"],
        discord: ["steipete", "1234567890123"]
      }
    }
  }
}
```

Override per agente (per restringere ulteriormente):

```json5
{
  agents: {
    list: [
      {
        id: "family",
        tools: {
          elevated: { enabled: false }
        }
      }
    ]
  }
}
```

Note:

* `tools.elevated` è la base globale. `agents.list[].tools.elevated` può solo imporre ulteriori restrizioni (entrambi devono essere abilitati).
* `/elevated on|off|ask|full` memorizza lo stato per chiave di sessione; le direttive in linea si applicano a un singolo messaggio.
* L&#39;`exec` elevato viene eseguito sull&#39;host e bypassa la sandbox.
* La policy degli strumenti si applica comunque; se `exec` è negato, la modalità elevata non può essere usata.

`agents.defaults.maxConcurrent` imposta il numero massimo di esecuzioni di agenti incorporati che possono
essere eseguite in parallelo tra le sessioni. Ogni sessione è comunque serializzata (una sola esecuzione
per chiave di sessione alla volta). Predefinito: 1.

<div id="agentsdefaultssandbox">
  ### `agents.defaults.sandbox`
</div>

**Sandbox Docker** opzionale per l&#39;agente incorporato. Pensato per le sessioni
non principali, in modo che non possano accedere al tuo sistema host.

Dettagli: [Sandboxing](/it/gateway/sandboxing)

Valori predefiniti (se abilitato):

* scope: `"agent"` (un container + spazio di lavoro per agente)
* immagine basata su Debian bookworm-slim
* accesso allo spazio di lavoro dell&#39;agente: `workspaceAccess: "none"` (predefinito)
  * `"none"`: utilizza uno spazio di lavoro sandbox per scope sotto `~/.openclaw/sandboxes`
* `"ro"`: mantiene lo spazio di lavoro della sandbox in `/workspace` e monta lo spazio di lavoro dell&#39;agente in sola lettura in `/agent` (disabilita `write`/`edit`/`apply_patch`)
  * `"rw"`: monta lo spazio di lavoro dell&#39;agente in lettura/scrittura in `/workspace`
* auto-prune: inattivo &gt; 24h OPPURE anzianità &gt; 7 giorni
* policy degli strumenti: consenti solo `exec`, `process`, `read`, `write`, `edit`, `apply_patch`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` (le negazioni hanno la precedenza)
  * configura tramite `tools.sandbox.tools`, sovrascrivi per singolo agente tramite `agents.list[].tools.sandbox.tools`
  * abbreviazioni dei gruppi di strumenti supportate nella policy della sandbox: `group:runtime`, `group:fs`, `group:sessions`, `group:memory` (vedi [Sandbox vs Tool Policy vs Elevated](/it/gateway/sandbox-vs-tool-policy-vs-elevated#tool-groups-shorthands))
* browser opzionale in sandbox (Chromium + CDP, osservatore noVNC)
* opzioni di hardening: `network`, `user`, `pidsLimit`, `memory`, `cpus`, `ulimits`, `seccompProfile`, `apparmorProfile`

Avvertenza: `scope: "shared"` significa un container condiviso e uno spazio di lavoro condiviso. Nessun
isolamento tra sessioni. Usa `scope: "session"` per l&#39;isolamento per sessione.

Legacy: `perSession` è ancora supportato (`true` → `scope: "session"`,
`false` → `scope: "shared"`).

`setupCommand` viene eseguito **una sola volta** dopo che il container è stato creato (all&#39;interno del container tramite `sh -lc`).
Per l&#39;installazione di pacchetti, assicurati che sia consentito il traffico di rete in uscita, che il file system root sia scrivibile e che l&#39;utente sia root.

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent is default)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          containerPrefix: "openclaw-sbx-",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          // Override per agente (multi-agente): agents.list[].sandbox.docker.*
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
          binds: ["/var/run/docker.sock:/var/run/docker.sock", "/home/user/source:/source:rw"]
        },
        browser: {
          enabled: false,
          image: "openclaw-sandbox-browser:bookworm-slim",
          containerPrefix: "openclaw-sbx-browser-",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          allowedControlUrls: ["http://10.0.0.42:18791"],
          allowedControlHosts: ["browser.lab.local", "10.0.0.42"],
          allowedControlPorts: [18791],
          autoStart: true,
          autoStartTimeoutMs: 12000
        },
        prune: {
          idleHours: 24,  // 0 disables idle pruning
          maxAgeDays: 7   // 0 disables max-age pruning
        }
      }
    }
  },
  tools: {
    sandbox: {
      tools: {
        allow: ["exec", "process", "read", "write", "edit", "apply_patch", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"]
      }
    }
  }
}
```

Crea l&#39;immagine sandbox predefinita una sola volta con:

```bash
scripts/sandbox-setup.sh
```

Nota: i container sandbox usano come impostazione predefinita `network: "none"`; imposta `agents.defaults.sandbox.docker.network`
su `"bridge"` (o sulla tua rete personalizzata) se l&#39;agente necessita di accesso in uscita.

Nota: gli allegati in ingresso vengono posizionati nello spazio di lavoro attivo in `media/inbound/*`. Con `workspaceAccess: "rw"`, ciò significa che i file vengono scritti nello spazio di lavoro dell&#39;agente.

Nota: `docker.binds` monta directory aggiuntive dell&#39;host; i bind globali e per agente vengono combinati.

Crea l&#39;immagine browser opzionale con:

```bash
scripts/sandbox-browser-setup.sh
```

Quando `agents.defaults.sandbox.browser.enabled=true`, lo strumento browser utilizza
un&#39;istanza di Chromium in sandbox (CDP). Se noVNC è abilitato (impostazione predefinita quando `headless=false`),
l&#39;URL di noVNC viene iniettato nel prompt di sistema così che l&#39;agente possa farvi riferimento.
Questo non richiede `browser.enabled` nella config principale; l&#39;URL di controllo della sandbox
viene iniettato per ogni sessione.

`agents.defaults.sandbox.browser.allowHostControl` (predefinito: false) consente
alle sessioni in sandbox di indirizzare esplicitamente il server di controllo del browser
**host** tramite lo strumento browser (`target: "host"`). Lascia questa opzione disabilitata
se vuoi un isolamento di sandbox rigoroso.

Liste di autorizzati per il controllo remoto:

* `allowedControlUrls`: URL di controllo esatti consentiti per `target: "custom"`.
* `allowedControlHosts`: nomi host consentiti (solo hostname, senza porta).
* `allowedControlPorts`: porte consentite (predefiniti: http=80, https=443).
  Valori predefiniti: tutte le liste di autorizzati non sono impostate (nessuna restrizione). `allowHostControl` è false per impostazione predefinita.

<div id="models-custom-providers-base-urls">
  ### `models` (provider personalizzati + URL di base)
</div>

OpenClaw utilizza il catalogo di modelli **pi-coding-agent**. Puoi aggiungere provider personalizzati
(LiteLLM, server locali compatibili con OpenAI, proxy Anthropic, ecc.) scrivendo
`~/.openclaw/agents/<agentId>/agent/models.json` oppure definendo lo stesso schema all&#39;interno
della configurazione di OpenClaw sotto `models.providers`.
Panoramica per singolo provider + esempi: [/concepts/model-providers](/it/concepts/model-providers).

Quando `models.providers` è presente, OpenClaw scrive/unisce un `models.json` in
`~/.openclaw/agents/<agentId>/agent/` all&#39;avvio:

* comportamento predefinito: **merge** (mantiene i provider esistenti, li sovrascrive in base al nome)
* imposta `models.mode: "replace"` per sovrascrivere il contenuto del file

Seleziona il modello tramite `agents.defaults.model.primary` (provider/model).

```json5
{
  agents: {
    defaults: {
      model: { primary: "custom-proxy/llama-3.1-8b" },
      models: {
        "custom-proxy/llama-3.1-8b": {}
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions",
        models: [
          {
            id: "llama-3.1-8b",
            name: "Llama 3.1 8B", // Nome visualizzato del modello
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 32000
          }
        ]
      }
    }
  }
}
```

<div id="opencode-zen-multi-model-proxy">
  ### OpenCode Zen (proxy multi-modello)
</div>

OpenCode Zen è un gateway multi-modello con endpoint specifici per ogni modello. OpenClaw utilizza
il provider `opencode` integrato di pi-ai; imposta `OPENCODE_API_KEY` (o
`OPENCODE_ZEN_API_KEY`) da https://opencode.ai/auth.

Note:

* I riferimenti ai modelli usano `opencode/<modelId>` (esempio: `opencode/claude-opus-4-5`).
* Se abiliti una lista di autorizzati tramite `agents.defaults.models`, aggiungi ogni modello che intendi usare.
* Scorciatoia: `openclaw onboard --auth-choice opencode-zen`.

```json5
{
  agents: {
    defaults: {
      model: { primary: "opencode/claude-opus-4-5" },
      models: { "opencode/claude-opus-4-5": { alias: "Opus" } }
    }
  }
}
```

<div id="zai-glm-47-provider-alias-support">
  ### Z.AI (GLM-4.7) — supporto per alias di provider
</div>

I modelli Z.AI sono disponibili tramite il provider integrato `zai`. Imposta `ZAI_API_KEY`
nell’ambiente e fai riferimento al modello come provider/modello.

Scorciatoia: `openclaw onboard --auth-choice zai-api-key`.

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} }
    }
  }
}
```

Note:

* `z.ai/*` e `z-ai/*` sono alias accettati e vengono normalizzati a `zai/*`.
* Se `ZAI_API_KEY` è assente, le richieste verso `zai/*` genereranno un errore di autenticazione in fase di esecuzione.
* Esempio di errore: `No API key found for provider "zai".`
* L&#39;endpoint API generale di Z.AI è `https://api.z.ai/api/paas/v4`. Le richieste di GLM Coding
  usano l&#39;endpoint dedicato Coding `https://api.z.ai/api/coding/paas/v4`.
  Il provider `zai` integrato usa l&#39;endpoint Coding. Se hai bisogno dell&#39;endpoint
  generale, definisci un provider personalizzato in `models.providers` con l&#39;override
  della base URL (vedi la sezione sui provider personalizzati sopra).
* Usa un valore fittizio come segnaposto nella documentazione e nelle configurazioni; non effettuare mai commit di chiavi API reali.

<div id="moonshot-ai-kimi">
  ### Moonshot AI (Kimi)
</div>

Utilizza l&#39;endpoint di Moonshot compatibile con OpenAI:

```json5
{
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: { "moonshot/kimi-k2.5": { alias: "Kimi K2.5" } }
    }
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

Note:

* Imposta `MOONSHOT_API_KEY` come variabile d&#39;ambiente oppure usa `openclaw onboard --auth-choice moonshot-api-key`.
* Modello di riferimento: `moonshot/kimi-k2.5`.
* Usa `https://api.moonshot.cn/v1` se ti serve l&#39;endpoint per la Cina.

<div id="kimi-code">
  ### Kimi Code
</div>

Usa l&#39;endpoint dedicato, compatibile con OpenAI, di Kimi Code (separato da Moonshot):

```json5
{
  env: { KIMICODE_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-code/kimi-for-coding" },
      models: { "kimi-code/kimi-for-coding": { alias: "Kimi Code" } }
    }
  },
  models: {
    mode: "merge",
    providers: {
      "kimi-code": {
        baseUrl: "https://api.kimi.com/coding/v1",
        apiKey: "${KIMICODE_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-for-coding",
            name: "Kimi For Coding",
            reasoning: true,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 262144,
            maxTokens: 32768,
            headers: { "User-Agent": "KimiCLI/0.77" },
            compat: { supportsDeveloperRole: false }
          }
        ]
      }
    }
  }
}
```

Note:

* Imposta la variabile d&#39;ambiente `KIMICODE_API_KEY` oppure usa `openclaw onboard --auth-choice kimi-code-api-key`.
* Modello di riferimento: `kimi-code/kimi-for-coding`.

<div id="synthetic-anthropic-compatible">
  ### Synthetic (compatibile con Anthropic)
</div>

Usa l&#39;endpoint Anthropic-compatibile di Synthetic:

```json5
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.1": { alias: "MiniMax M2.1" } }
    }
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536
          }
        ]
      }
    }
  }
}
```

Note:

* Imposta `SYNTHETIC_API_KEY` oppure usa `openclaw onboard --auth-choice synthetic-api-key`.
* Riferimento modello: `synthetic/hf:MiniMaxAI/MiniMax-M2.1`.
* L&#39;URL di base deve omettere `/v1` perché il client Anthropic lo aggiunge automaticamente.

<div id="local-models-lm-studio-recommended-setup">
  ### Modelli locali (LM Studio) — configurazione consigliata
</div>

Consulta [/gateway/local-models](/it/gateway/local-models) per le indicazioni aggiornate sui modelli locali. In breve: esegui MiniMax M2.1 tramite LM Studio Responses API su hardware potente; mantieni i modelli hosted unificati come fallback.

<div id="minimax-m21">
  ### MiniMax M2.1
</div>

Utilizza direttamente MiniMax M2.1 senza LM Studio:

```json5
{
  agent: {
    model: { primary: "minimax/MiniMax-M2.1" },
    models: {
      "anthropic/claude-opus-4-5": { alias: "Opus" },
      "minimax/MiniMax-M2.1": { alias: "Minimax" }
    }
  },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            // Prezzi: aggiorna models.json se necessiti di un tracciamento preciso dei costi.
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

Note:

* Imposta la variabile d&#39;ambiente `MINIMAX_API_KEY` oppure usa `openclaw onboard --auth-choice minimax-api`.
* Modello disponibile: `MiniMax-M2.1` (predefinito).
* Aggiorna i prezzi in `models.json` se ti serve un monitoraggio preciso dei costi.

<div id="cerebras-glm-46-47">
  ### Cerebras (GLM 4.6 / 4.7)
</div>

Utilizza Cerebras tramite il suo endpoint compatibile con OpenAI:

```json5
{
  env: { CEREBRAS_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: {
        primary: "cerebras/zai-glm-4.7",
        fallbacks: ["cerebras/zai-glm-4.6"]
      },
      models: {
        "cerebras/zai-glm-4.7": { alias: "GLM 4.7 (Cerebras)" },
        "cerebras/zai-glm-4.6": { alias: "GLM 4.6 (Cerebras)" }
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      cerebras: {
        baseUrl: "https://api.cerebras.ai/v1",
        apiKey: "${CEREBRAS_API_KEY}",
        api: "openai-completions",
        models: [
          { id: "zai-glm-4.7", name: "GLM 4.7 (Cerebras)" },
          { id: "zai-glm-4.6", name: "GLM 4.6 (Cerebras)" }
        ]
      }
    }
  }
}
```

Note:

* Usa `cerebras/zai-glm-4.7` per Cerebras; usa `zai/glm-4.7` per Z.AI diretto.
* Imposta `CEREBRAS_API_KEY` come variabile d&#39;ambiente o nella configurazione.

Note:

* API supportate: `openai-completions`, `openai-responses`, `anthropic-messages`,
  `google-generative-ai`
* Usa `authHeader: true` + `headers` per esigenze di autenticazione personalizzate.
* Sovrascrivi la directory radice della configurazione dell&#39;agente con `OPENCLAW_AGENT_DIR` (o `PI_CODING_AGENT_DIR`)
  se vuoi che `models.json` sia archiviato altrove (predefinito: `~/.openclaw/agents/main/agent`).

<div id="session">
  ### `session`
</div>

Controlla l&#39;ambito della sessione, i criteri di reset, i trigger di reset e la posizione in cui viene scritto lo store delle sessioni.

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main",
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"]
    },
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 60
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      dm: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 }
    },
    resetTriggers: ["/new", "/reset"],
    // Il valore predefinito è già per agente in ~/.openclaw/agents/<agentId>/sessions/sessions.json
    // Puoi sovrascriverlo usando il templating {agentId}:
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    // Direct chats collapse to agent:<agentId>:<mainKey> (default: "main").
    mainKey: "main",
    agentToAgent: {
      // Max ping-pong reply turns between requester/target (0–5).
      maxPingPongTurns: 5
    },
    sendPolicy: {
      rules: [
        { action: "deny", match: { channel: "discord", chatType: "group" } }
      ],
      default: "allow"
    }
  }
}
```

Fields:

* `mainKey`: chiave del bucket di chat diretta (default: `"main"`). Utile quando vuoi “rinominare” il thread DM primario senza cambiare `agentId`.
  * Nota sandbox: `agents.defaults.sandbox.mode: "non-main"` usa questa chiave per rilevare la sessione principale. Qualsiasi chiave di sessione che non corrisponde a `mainKey` (gruppi/canali) viene eseguita in sandbox.
* `dmScope`: come vengono raggruppate le sessioni DM (default: `"main"`).
  * `main`: tutti i DM condividono la sessione principale per la continuità.
  * `per-peer`: isola i DM per id del mittente attraverso i canali.
  * `per-channel-peer`: isola i DM per canale + mittente (consigliato per caselle di posta multi‑utente).
  * `per-account-channel-peer`: isola i DM per account + canale + mittente (consigliato per caselle di posta multi‑account).
* `identityLinks`: mappa gli id canonici a peer con prefisso di provider in modo che la stessa persona condivida una sessione DM tra canali quando si usa `per-peer`, `per-channel-peer` o `per-account-channel-peer`.
  * Esempio: `alice: ["telegram:123456789", "discord:987654321012345678"]`.
* `reset`: criterio di reset principale. Per impostazione predefinita, il reset è giornaliero alle 4:00 del mattino, ora locale, sull&#39;host del Gateway.
  * `mode`: `daily` o `idle` (default: `daily` quando `reset` è presente).
  * `atHour`: ora locale (0-23) per il momento del reset giornaliero.
  * `idleMinutes`: finestra di inattività mobile in minuti. Quando daily e idle sono entrambi configurati, ha la precedenza quello che scade per primo.
* `resetByType`: override per singola sessione per `dm`, `group` e `thread`.
  * Se imposti solo il legacy `session.idleMinutes` senza alcun `reset`/`resetByType`, OpenClaw rimane in modalità solo inattività per retrocompatibilità.
* `heartbeatIdleMinutes`: override opzionale di inattività per i controlli di heartbeat (il reset giornaliero continua ad applicarsi quando è abilitato).
* `agentToAgent.maxPingPongTurns`: numero massimo di scambi di risposta avanti‑indietro tra richiedente/destinazione (0–5, default 5).
* `sendPolicy.default`: valore di fallback `allow` o `deny` quando nessuna regola corrisponde.
* `sendPolicy.rules[]`: esegue il match per `channel`, `chatType` (`direct|group|room`) o `keyPrefix` (ad es. `cron:`). Viene applicato il primo deny; in caso contrario allow.

<div id="skills-skills-config">
  ### `skills` (configurazione delle abilità)
</div>

Controlla la lista di autorizzati integrata, le preferenze di installazione, le cartelle aggiuntive delle abilità e le sostituzioni per singola abilità. Si applica alle abilità **integrate** e a `~/.openclaw/skills` (le abilità dello spazio di lavoro hanno comunque la precedenza in caso di conflitti di nome).

Campi:

* `allowBundled`: lista di autorizzati opzionale solo per le abilità **integrate**. Se impostata, solo quelle abilità integrate sono utilizzabili (le abilità gestite/dello spazio di lavoro non sono influenzate).
* `load.extraDirs`: directory aggiuntive di abilità da sottoporre a scansione (precedenza più bassa).
* `install.preferBrew`: preferisci gli installer di brew quando disponibili (predefinito: true).
* `install.nodeManager`: preferenza per il gestore di pacchetti per Node (`npm` | `pnpm` | `yarn`, predefinito: npm).
* `entries.<skillKey>`: sostituzioni di configurazione per singola abilità.

Campi per singola abilità:

* `enabled`: imposta `false` per disabilitare un&#39;abilità anche se è integrata/installata.
* `env`: variabili d&#39;ambiente iniettate per l&#39;esecuzione dell&#39;agente (solo se non già impostate).
* `apiKey`: opzione facoltativa di comodità per le abilità che dichiarano una variabile d&#39;ambiente primaria (ad es. `nano-banana-pro` → `GEMINI_API_KEY`).

Esempio:

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: [
        "~/Projects/agent-scripts/skills",
        "~/Projects/oss/some-skill-pack/skills"
      ]
    },
    install: {
      preferBrew: true,
      nodeManager: "npm"
    },
    entries: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE"
        }
      },
      peekaboo: { enabled: true },
      sag: { enabled: false }
    }
  }
}
```

<div id="plugins-extensions">
  ### `plugins` (estensioni)
</div>

Controlla l’individuazione dei plugin, le regole di allow/deny e la configurazione per singolo plugin. I plugin vengono caricati
da `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions`, oltre a tutte le
voci `plugins.load.paths`. **Le modifiche alla configurazione richiedono il riavvio del Gateway.**
Vedi [/plugin](/it/plugin) per i dettagli completi sull’utilizzo.

Campi:

* `enabled`: interruttore principale per il caricamento dei plugin (predefinito: true).
* `allow`: lista di autorizzati opzionale di ID dei plugin; se impostata, vengono caricati solo i plugin elencati.
* `deny`: lista di esclusi opzionale di ID dei plugin (deny ha la precedenza).
* `load.paths`: file o directory di plugin aggiuntivi da caricare (percorsi assoluti o con `~`).
* `entries.<pluginId>`: override per singolo plugin.
  * `enabled`: imposta `false` per disabilitare.
  * `config`: oggetto di configurazione specifico del plugin (validato dal plugin se presente).

Esempio:

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    load: {
      paths: ["~/Projects/oss/voice-call-extension"]
    },
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio"
        }
      }
    }
  }
}
```

<div id="browser-openclaw-managed-browser">
  ### `browser` (browser gestito da openclaw)
</div>

OpenClaw può avviare un&#39;istanza **dedicata e isolata** di Chrome/Brave/Edge/Chromium per openclaw ed esporre un piccolo servizio di controllo su loopback.
I profili possono puntare a un browser **remoto** basato su Chromium tramite `profiles.<name>.cdpUrl`. I profili remoti sono di tipo attach-only (start/stop/reset sono disabilitati).

`browser.cdpUrl` rimane per configurazioni legacy a singolo profilo e come schema/host di base per i profili che impostano solo `cdpPort`.

Valori predefiniti:

* enabled: `true`
* evaluateEnabled: `true` (imposta `false` per disabilitare `act:evaluate` e `wait --fn`)
* servizio di controllo: solo loopback (porta derivata da `gateway.port`, predefinita `18791`)
* URL CDP: `http://127.0.0.1:18792` (servizio di controllo + 1, singolo profilo legacy)
* colore del profilo: `#FF4500` (arancione-aragosta)
* Nota: il server di controllo viene avviato dal Gateway in esecuzione (barra dei menu di OpenClaw.app o `openclaw gateway`).
* Ordine di auto-rilevamento: browser predefinito se basato su Chromium; in caso contrario Chrome → Brave → Edge → Chromium → Chrome Canary.

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: true,
    // cdpUrl: "http://127.0.0.1:18792", // legacy single-profile override
    defaultProfile: "chrome",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" }
    },
    color: "#FF4500",
    // Advanced:
    // headless: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    // attachOnly: false, // impostare a true quando si effettua il tunneling di un CDP remoto verso localhost
  }
}
```

<div id="ui-appearance">
  ### `ui` (Aspetto)
</div>

Colore di accento opzionale utilizzato dalle app native per gli elementi della UI (ad es. la tinta della bolla in Talk Mode).

Se non impostato, i client usano come valore predefinito un azzurro tenue.

```json5
{
  ui: {
    seamColor: "#FF4500", // hex (RRGGBB or #RRGGBB)
    // Opzionale: override dell'identità dell'assistente per la Control UI.
    // Se non impostato, la Control UI utilizza l'identità dell'agente attivo (config o IDENTITY.md).
    assistant: {
      name: "OpenClaw",
      avatar: "CB" // emoji, short text, or image URL/data URI
    }
  }
}
```

<div id="gateway-gateway-server-mode-bind">
  ### `gateway` (modalità server del Gateway + bind)
</div>

Usa `gateway.mode` per indicare esplicitamente se questa macchina deve eseguire il Gateway.

Valori predefiniti:

* mode: **non impostato** (interpretato come “non avviare automaticamente”)
* bind: `loopback`
* port: `18789` (porta unica per WS + HTTP)

```json5
{
  gateway: {
    mode: "local", // or "remote"
    port: 18789, // WS + HTTP multiplex
    bind: "loopback",
    // controlUi: { enabled: true, basePath: "/openclaw" }
    // auth: { mode: "token", token: "your-token" } // il token controlla l'accesso a WS + Control UI
    // tailscale: { mode: "off" | "serve" | "funnel" }
  }
}
```

Percorso di base della Control UI:

* `gateway.controlUi.basePath` imposta il prefisso URL da cui viene servita la Control UI.
* Esempi: `"/ui"`, `"/openclaw"`, `"/apps/openclaw"`.
* Predefinito: root (`/`) (invariato).
* `gateway.controlUi.allowInsecureAuth` consente l&#39;autenticazione basata solo su token per la Control UI quando
  l&#39;identità del dispositivo è omessa (in genere su HTTP). Predefinito: `false`. Preferisci usare HTTPS
  (Tailscale Serve) o `127.0.0.1`.
* `gateway.controlUi.dangerouslyDisableDeviceAuth` disabilita i controlli di identità del dispositivo per la
  Control UI (solo token/password). Predefinito: `false`. Solo in casi di emergenza.

Documentazione correlata:

* [Control UI](/it/web/control-ui)
* [Panoramica Web](/it/web)
* [Tailscale](/it/gateway/tailscale)
* [Accesso remoto](/it/gateway/remote)

Proxy attendibili:

* `gateway.trustedProxies`: elenco di IP dei reverse proxy che terminano TLS davanti al Gateway.
* Quando una connessione proviene da uno di questi IP, OpenClaw usa `x-forwarded-for` (o `x-real-ip`) per determinare l&#39;IP client per i controlli di abbinamento locale e per i controlli HTTP di auth/local.
* Elenca solo proxy che controlli completamente e assicurati che **sovrascrivano** l&#39;header `x-forwarded-for` in ingresso.

Note:

* `openclaw gateway` si rifiuta di avviarsi a meno che `gateway.mode` non sia impostato su `local` (o non passi il flag di override).
* `gateway.port` controlla l&#39;unica porta multiplexata usata per WebSocket + HTTP (Control UI, hook, A2UI).
* Endpoint OpenAI Chat Completions: **disabilitato per impostazione predefinita**; abilitalo con `gateway.http.endpoints.chatCompletions.enabled: true`.
* Precedenza: `--port` &gt; `OPENCLAW_GATEWAY_PORT` &gt; `gateway.port` &gt; predefinito `18789`.
* L&#39;autenticazione del Gateway è richiesta per impostazione predefinita (token/password o identità Tailscale Serve). I bind non-loopback richiedono un token/password condiviso.
* La procedura guidata di onboarding genera un token del gateway per impostazione predefinita (anche su loopback).
* `gateway.remote.token` è **solo** per chiamate CLI remote; non abilita l&#39;auth locale del gateway. `gateway.token` viene ignorato.

Auth e Tailscale:

* `gateway.auth.mode` imposta i requisiti dell&#39;handshake (`token` o `password`). Se non impostato, si presume l&#39;autenticazione tramite token.
* `gateway.auth.token` memorizza il token condiviso per l&#39;autenticazione tramite token (usato dalla CLI sulla stessa macchina).
* Quando `gateway.auth.mode` è impostato, viene accettato solo quel metodo (più eventuali header Tailscale).
* `gateway.auth.password` può essere impostata qui o tramite `OPENCLAW_GATEWAY_PASSWORD` (consigliato).
* `gateway.auth.allowTailscale` consente agli header di identità Tailscale Serve
  (`tailscale-user-login`) di soddisfare l&#39;auth quando la richiesta arriva su loopback
  con `x-forwarded-for`, `x-forwarded-proto` e `x-forwarded-host`. OpenClaw
  verifica l&#39;identità risolvendo l&#39;indirizzo `x-forwarded-for` tramite
  `tailscale whois` prima di accettarla. Quando è `true`, le richieste Serve non hanno bisogno
  di token/password; imposta `false` per richiedere credenziali esplicite. Il valore predefinito è
  `true` quando `tailscale.mode = "serve"` e la modalità di auth non è `password`.
* `gateway.tailscale.mode: "serve"` usa Tailscale Serve (solo tailnet, bind su loopback).
* `gateway.tailscale.mode: "funnel"` espone la dashboard pubblicamente; richiede auth.
* `gateway.tailscale.resetOnExit` ripristina la configurazione Serve/Funnel allo shutdown.

Impostazioni predefinite del client remoto (CLI):

* `gateway.remote.url` imposta l&#39;URL WebSocket predefinito del Gateway per le chiamate CLI quando `gateway.mode = "remote"`.
* `gateway.remote.transport` seleziona il trasporto remoto su macOS (`ssh` predefinito, `direct` per WS/WSS). Quando è `direct`, `gateway.remote.url` deve iniziare con `ws://` o `wss://`. `ws://host` usa per impostazione predefinita la porta `18789`.
* `gateway.remote.token` fornisce il token per le chiamate remote (lascialo non impostato per nessuna autenticazione).
* `gateway.remote.password` fornisce la password per le chiamate remote (lascialo non impostato per nessuna autenticazione).

Comportamento dell&#39;app macOS:

* OpenClaw.app monitora `~/.openclaw/openclaw.json` e cambia modalità in tempo reale quando `gateway.mode` o `gateway.remote.url` cambiano.
* Se `gateway.mode` non è impostato ma `gateway.remote.url` è impostato, l&#39;app macOS lo considera come modalità remota.
* Quando modifichi la modalità di connessione nell&#39;app macOS, questa scrive `gateway.mode` (e `gateway.remote.url` + `gateway.remote.transport` in modalità remota) di nuovo nel file di configurazione.

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      password: "your-password"
    }
  }
}
```

Esempio di trasporto diretto (app per macOS):

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      transport: "direct",
      url: "wss://gateway.example.ts.net",
      token: "your-token"
    }
  }
}
```

<div id="gatewayreload-config-hot-reload">
  ### `gateway.reload` (Hot reload della configurazione)
</div>

Il Gateway monitora `~/.openclaw/openclaw.json` (o `OPENCLAW_CONFIG_PATH`) e applica automaticamente le modifiche.

Modalità:

* `hybrid` (predefinita): applica in hot reload le modifiche sicure; riavvia il Gateway per le modifiche critiche.
* `hot`: applica solo le modifiche sicure per l’hot reload; scrive nei log quando è necessario un riavvio.
* `restart`: riavvia il Gateway a ogni modifica della configurazione.
* `off`: disabilita l’hot reload.

```json5
{
  gateway: {
    reload: {
      mode: "hybrid",
      debounceMs: 300
    }
  }
}
```

<div id="hot-reload-matrix-files-impact">
  #### Matrice di hot reload (file + impatto)
</div>

File monitorati:

* `~/.openclaw/openclaw.json` (oppure `OPENCLAW_CONFIG_PATH`)

Applicato a caldo (senza riavvio completo del Gateway):

* `hooks` (autenticazione/percorso/associazioni dei webhook) + `hooks.gmail` (watcher di Gmail riavviato)
* `browser` (riavvio del server di controllo del browser)
* `cron` (riavvio del servizio cron + aggiornamento della concorrenza)
* `agents.defaults.heartbeat` (riavvio del runner di heartbeat)
* `web` (riavvio del canale WhatsApp Web)
* `telegram`, `discord`, `signal`, `imessage` (riavvio dei canali)
* `agent`, `models`, `routing`, `messages`, `session`, `whatsapp`, `logging`, `skills`, `ui`, `talk`, `identity`, `wizard` (letture dinamiche)

Richiede il riavvio completo del Gateway:

* `gateway` (porta/bind/auth/Control UI/Tailscale)
* `bridge` (legacy)
* `discovery`
* `canvasHost`
* `plugins`
* Qualsiasi percorso di configurazione sconosciuto/non supportato (per sicurezza si esegue comunque il riavvio predefinito)

<div id="multi-instance-isolation">
  ### Isolamento multi‑istanza
</div>

Per eseguire più gateway su un singolo host (per ridondanza o come bot di emergenza), isola lo stato e la configurazione per istanza e usa porte univoche:

* `OPENCLAW_CONFIG_PATH` (configurazione per istanza)
* `OPENCLAW_STATE_DIR` (sessioni/credenziali)
* `agents.defaults.workspace` (memorie)
* `gateway.port` (univoca per istanza)

Flag di convenienza (CLI):

* `openclaw --dev …` → usa `~/.openclaw-dev` e sposta le porte a partire dalla base `19001`
* `openclaw --profile <name> …` → usa `~/.openclaw-<name>` (porta gestita tramite config/env/flag)

Vedi il [runbook del Gateway](/it/gateway) per la mappatura delle porte derivate (gateway/browser/canvas).
Vedi [gateway multipli](/it/gateway/multiple-gateways) per i dettagli sull’isolamento delle porte del browser/CDP.

Esempio:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

<div id="hooks-gateway-webhooks">
  ### `hooks` (webhook del Gateway)
</div>

Attiva un semplice endpoint HTTP per webhook sul server HTTP del Gateway.

Valori predefiniti:

* enabled: `false`
* path: `/hooks`
* maxBodyBytes: `262144` (256 KB)

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    presets: ["gmail"],
    transformsDir: "~/.openclaw/hooks",
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate:
          "From: {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}",
        deliver: true,
        channel: "last",
        model: "openai/gpt-5.2-mini",
      },
    ],
  }
}
```

Le richieste devono includere il token dell&#39;hook:

* `Authorization: Bearer &lt;token&gt;` **oppure**
* `x-openclaw-token: &lt;token&gt;` **oppure**
* `?token=&lt;token&gt;`

Endpoint:

* `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
* `POST /hooks/agent` → `{ message, name?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }`
* `POST /hooks/&lt;name&gt;` → risolto tramite `hooks.mappings`

`/hooks/agent` invia sempre un riepilogo nella sessione principale (e può facoltativamente attivare immediatamente un heartbeat tramite `wakeMode: "now"`).

Note sul mapping:

* `match.path` corrisponde al sottopercorso dopo `/hooks` (ad es. `/hooks/gmail` → `gmail`).
* `match.source` corrisponde a un campo del payload (ad es. `{ source: "gmail" }`), in modo da poter usare un percorso generico `/hooks/ingest`.
* I template come `{{messages[0].subject}}` leggono dal payload.
* `transform` può puntare a un modulo JS/TS che restituisce un&#39;azione dell&#39;hook.
* `deliver: true` invia la risposta finale a un canale; `channel` per impostazione predefinita è `last` (con fallback su WhatsApp).
* Se non esiste un percorso di consegna precedente, imposta esplicitamente `channel` + `to` (obbligatorio per Telegram/Discord/Google Chat/Slack/Signal/iMessage/MS Teams).
* `model` sovrascrive l&#39;LLM per questa esecuzione dell&#39;hook (`provider/model` o alias; deve essere consentito se `agents.defaults.models` è impostato).

Configurazione helper Gmail (usata da `openclaw webhooks gmail setup` / `run`):

```json5
{
  hooks: {
    gmail: {
      account: "openclaw@gmail.com",
      topic: "projects/<project-id>/topics/gog-gmail-watch",
      subscription: "gog-gmail-watch-push",
      pushToken: "shared-push-token",
      hookUrl: "http://127.0.0.1:18789/hooks/gmail",
      includeBody: true,
      maxBytes: 20000,
      renewEveryMinutes: 720,
      serve: { bind: "127.0.0.1", port: 8788, path: "/" },
      tailscale: { mode: "funnel", path: "/gmail-pubsub" },

      // Opzionale: usa un modello più economico per l'elaborazione degli hook Gmail
      // Ricade su agents.defaults.model.fallbacks, poi primary, in caso di auth/rate-limit/timeout
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      // Optional: default thinking level for Gmail hooks
      thinking: "off",
    }
  }
}
```

Override del modello per gli hook Gmail:

* `hooks.gmail.model` specifica un modello da usare per l&#39;elaborazione degli hook Gmail (predefinito: primario della sessione).
* Accetta riferimenti `provider/model` o alias da `agents.defaults.models`.
* In caso di errori di autenticazione, rate limit o timeout, effettua il fallback a `agents.defaults.model.fallbacks`, quindi a `agents.defaults.model.primary`.
* Se `agents.defaults.models` è impostato, includi il modello degli hook nella lista di autorizzati.
* All&#39;avvio, viene mostrato un avviso se il modello configurato non è presente nel catalogo dei modelli o nella lista di autorizzati.
* `hooks.gmail.thinking` imposta il livello di thinking predefinito per gli hook Gmail e può essere sovrascritto dal valore `thinking` specifico per hook.

Avvio automatico del Gateway:

* Se `hooks.enabled=true` e `hooks.gmail.account` è impostato, il Gateway avvia
  `gog gmail watch serve` al boot e rinnova automaticamente il watch.
* Imposta `OPENCLAW_SKIP_GMAIL_WATCHER=1` per disabilitare l&#39;avvio automatico (per esecuzioni manuali).
* Evita di eseguire un `gog gmail watch serve` separato insieme al Gateway; si
  verificherà un errore con `listen tcp 127.0.0.1:8788: bind: address already in use`.

Nota: quando `tailscale.mode` è attivo, OpenClaw imposta per impostazione predefinita `serve.path` su `/` in modo che
Tailscale possa fare da proxy per `/gmail-pubsub` correttamente (rimuove il prefisso di percorso impostato).
Se hai bisogno che il backend riceva il percorso con il prefisso, imposta
`hooks.gmail.tailscale.target` su un URL completo (e allinea `serve.path`).

<div id="canvashost-lantailnet-canvas-file-server-live-reload">
  ### `canvasHost` (server di file Canvas LAN/Tailnet + live reload)
</div>

Il Gateway espone una directory di file HTML/CSS/JS tramite HTTP, in modo che i nodi iOS/Android possano semplicemente eseguire `canvas.navigate` verso di essa.

Root predefinita: `~/.openclaw/workspace/canvas`
Porta predefinita: `18793` (scelta per evitare la porta CDP del browser openclaw `18792`)
Il server è in ascolto sull&#39;**host di binding del Gateway** (LAN o Tailnet) così che i nodi possano raggiungerlo.

Il server:

* serve i file sotto `canvasHost.root`
* inietta un piccolo client di live-reload nell&#39;HTML servito
* monitora la directory e trasmette i reload tramite un endpoint WS (WebSocket) su `/__openclaw__/ws`
* crea automaticamente un `index.html` iniziale quando la directory è vuota (così vedi subito qualcosa)
* serve anche A2UI su `/__openclaw__/a2ui/` ed è annunciato ai nodi come `canvasHostUrl`
  (sempre usato dai nodi per Canvas/A2UI)

Disabilita il live reload (e il monitoraggio dei file) se la directory è grande o incontri `EMFILE`:

* config: `canvasHost: { liveReload: false }`

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    port: 18793,
    liveReload: true
  }
}
```

Le modifiche a `canvasHost.*` richiedono il riavvio del Gateway (il ricaricamento della configurazione causerà un riavvio).

Disabilita con:

* config: `canvasHost: { enabled: false }`
* env: `OPENCLAW_SKIP_CANVAS_HOST=1`

<div id="bridge-legacy-tcp-bridge-removed">
  ### `bridge` (bridge TCP legacy, rimosso)
</div>

Le build attuali non includono più il listener del bridge TCP; le chiavi di configurazione `bridge.*` vengono ignorate.
I nodi si connettono tramite il WebSocket del Gateway. Questa sezione è mantenuta per riferimento storico.

Comportamento legacy:

* Il Gateway poteva esporre un semplice bridge TCP per i nodi (iOS/Android), tipicamente sulla porta `18790`.

Valori predefiniti:

* enabled: `true`
* port: `18790`
* bind: `lan` (effettua il bind a `0.0.0.0`)

Modalità di bind:

* `lan`: `0.0.0.0` (raggiungibile su qualsiasi interfaccia, inclusi LAN/Wi‑Fi e Tailscale)
* `tailnet`: effettua il bind solo all’IP Tailscale della macchina (consigliato per Vienna ⇄ London)
* `loopback`: `127.0.0.1` (solo locale)
* `auto`: preferisce l’IP tailnet se presente, altrimenti `lan`

TLS:

* `bridge.tls.enabled`: abilita TLS per le connessioni del bridge (solo TLS quando abilitato).
* `bridge.tls.autoGenerate`: genera un certificato self-signed quando non sono presenti cert/key (valore predefinito: true).
* `bridge.tls.certPath` / `bridge.tls.keyPath`: percorsi PEM per il certificato del bridge + chiave privata.
* `bridge.tls.caPath`: bundle PEM CA opzionale (root personalizzate o futuro mTLS).

Quando TLS è abilitato, il Gateway annuncia `bridgeTls=1` e `bridgeTlsSha256` nei record TXT di discovery
in modo che i nodi possano effettuare il pinning del certificato. Le connessioni manuali usano il modello trust-on-first-use se non è
ancora memorizzata alcuna impronta digitale.
I certificati generati automaticamente richiedono `openssl` nel PATH; se la generazione non riesce, il bridge non verrà avviato.

```json5
{
  bridge: {
    enabled: true,
    port: 18790,
    bind: "tailnet",
    tls: {
      enabled: true,
      // Usa ~/.openclaw/bridge/tls/bridge-{cert,key}.pem se omesso.
      // certPath: "~/.openclaw/bridge/tls/bridge-cert.pem",
      // keyPath: "~/.openclaw/bridge/tls/bridge-key.pem"
    }
  }
}
```

<div id="discoverymdns-bonjour-mdns-broadcast-mode">
  ### `discovery.mdns` (modalità di broadcast Bonjour / mDNS)
</div>

Controlla i broadcast di discovery mDNS sulla LAN (`_openclaw-gw._tcp`).

* `minimal` (predefinito): omette `cliPath` + `sshPort` dai record TXT
* `full`: include `cliPath` + `sshPort` nei record TXT
* `off`: disabilita completamente i broadcast mDNS
* Hostname: per impostazione predefinita è `openclaw` (annuncia `openclaw.local`). Puoi sovrascriverlo con `OPENCLAW_MDNS_HOSTNAME`.

```json5
{
  discovery: { mdns: { mode: "minimal" } }
}
```

<div id="discoverywidearea-wide-area-bonjour-unicast-dnssd">
  ### `discovery.wideArea` (Bonjour su ampia area / DNS‑SD unicast)
</div>

Quando è abilitata, il Gateway scrive una zona DNS-SD unicast per `_openclaw-gw._tcp` sotto `~/.openclaw/dns/` usando il dominio di discovery configurato (esempio: `openclaw.internal.`).

Per fare in modo che iOS/Android individuino il Gateway anche tra reti diverse (Vienna ⇄ Londra), abbina questa impostazione a:

* un server DNS sull&#39;host del Gateway che serva il dominio scelto (si consiglia CoreDNS)
* **split DNS** di Tailscale in modo che i client risolvano quel dominio tramite il server DNS del Gateway

Helper per la configurazione una tantum (host del Gateway):

```bash
openclaw dns setup --apply
```

```json5
{
  discovery: { wideArea: { enabled: true } }
}
```

## Variabili di template

I segnaposto del template vengono espansi in `tools.media.*.models[].args` e `tools.media.models[].args` (e in qualsiasi futuro campo di argomenti basato su template).

| Variabile | Descrizione |
|----------|-------------|
| `{{Body}}` | Corpo completo del messaggio in ingresso |
| `{{RawBody}}` | Corpo grezzo del messaggio in ingresso (senza wrapper di cronologia/mittente; ideale per il parsing dei comandi) |
| `{{BodyStripped}}` | Corpo con menzioni di gruppo rimosse (miglior valore predefinito per gli agenti) |
| `{{From}}` | Identificatore del mittente (E.164 per WhatsApp; può variare in base al canale) |
| `{{To}}` | Identificatore di destinazione |
| `{{MessageSid}}` | ID del messaggio del canale (quando disponibile) |
| `{{SessionId}}` | UUID della sessione corrente |
| `{{IsNewSession}}` | `"true"` quando è stata creata una nuova sessione |
| `{{MediaUrl}}` | Pseudo-URL del contenuto multimediale in ingresso (se presente) |
| `{{MediaPath}}` | Percorso locale del contenuto multimediale (se scaricato) |
| `{{MediaType}}` | Tipo di contenuto multimediale (immagine/audio/documento/…) |
| `{{Transcript}}` | Trascrizione audio (quando abilitata) |
| `{{Prompt}}` | Prompt multimediale risolto per le voci della CLI |
| `{{MaxChars}}` | Numero massimo di caratteri di output risolto per le voci della CLI |
| `{{ChatType}}` | `"direct"` o `"group"` |
| `{{GroupSubject}}` | Oggetto del gruppo (per quanto possibile) |
| `{{GroupMembers}}` | Anteprima dei membri del gruppo (per quanto possibile) |
| `{{SenderName}}` | Nome visualizzato del mittente (per quanto possibile) |
| `{{SenderE164}}` | Numero di telefono del mittente (per quanto possibile) |
| `{{Provider}}` | Indicazione del provider (whatsapp|telegram|discord|googlechat|slack|signal|imessage|msteams|webchat|…) |

<div id="cron-gateway-scheduler">
  ## Cron (Gateway scheduler)
</div>

Cron è lo scheduler del Gateway per attivazioni e job pianificati. Consulta [Cron jobs](/it/automation/cron-jobs) per una panoramica delle funzionalità e per esempi di utilizzo della CLI.

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2
  }
}
```

***

*Avanti: [Runtime dell&#39;agente](/it/concepts/agent)* 🦞
