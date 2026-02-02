---
title: Gruppi
summary: "Comportamento delle chat di gruppo sulle diverse piattaforme (WhatsApp/Telegram/Discord/Slack/Signal/iMessage/Microsoft Teams)"
read_when:
  - Modificare il comportamento delle chat di gruppo o le limitazioni sulle menzioni
---

<div id="groups">
  # Gruppi
</div>

OpenClaw gestisce le chat di gruppo in modo uniforme su tutte le piattaforme: WhatsApp, Telegram, Discord, Slack, Signal, iMessage, Microsoft Teams.

<div id="beginner-intro-2-minutes">
  ## Introduzione per principianti (2 minuti)
</div>

OpenClaw “vive” sui tuoi account di messaggistica. Non esiste un utente bot WhatsApp separato.
Se **tu** sei in un gruppo, OpenClaw può vedere quel gruppo e rispondere lì.

Comportamento predefinito:

* I gruppi sono ristretti (`groupPolicy: "allowlist"`).
* Le risposte richiedono una menzione, a meno che tu non disabiliti esplicitamente il gating basato su menzione.

Traduzione: i mittenti nella lista di autorizzati possono attivare OpenClaw menzionandolo.

> TL;DR
>
> * **L’accesso ai DM** è controllato da `*.allowFrom`.
> * **L’accesso ai gruppi** è controllato da `*.groupPolicy` + liste di autorizzati (`*.groups`, `*.groupAllowFrom`).
> * **L’attivazione delle risposte** è controllata dal gating basato su menzione (`requireMention`, `/activation`).

Flusso rapido (che cosa succede a un messaggio di gruppo):

```
groupPolicy? disabled -> drop
groupPolicy? allowlist -> group allowed? no -> drop
requireMention? yes -> mentioned? no -> store for context only
otherwise -> reply
```

![Flusso dei messaggi di gruppo](/images/groups-flow.svg)

Se vuoi...

| Obiettivo                                                  | Cosa impostare                                             |
| ---------------------------------------------------------- | ---------------------------------------------------------- |
| Abilitare tutti i gruppi ma rispondere solo alle @menzioni | `groups: { "*": { requireMention: true } }`                |
| Disabilitare tutte le risposte nei gruppi                  | `groupPolicy: "disabled"`                                  |
| Solo gruppi specifici                                      | `groups: { "<group-id>": { ... } }` (nessuna chiave `"*"`) |
| Solo tu puoi attivare l&#39;assistente nei gruppi          | `groupPolicy: "allowlist"`, `groupAllowFrom: ["+1555..."]` |

<div id="session-keys">
  ## Chiavi di sessione
</div>

* Le sessioni di gruppo usano chiavi di sessione `agent:<agentId>:<channel>:group:<id>` (le stanze/canali usano `agent:<agentId>:<channel>:channel:<id>`).
* Gli argomenti dei forum Telegram aggiungono `:topic:<threadId>` all&#39;id del gruppo in modo che ogni argomento abbia la propria sessione.
* Le chat dirette usano la sessione principale (o una sessione per mittente, se configurato).
* Gli heartbeat non vengono eseguiti per le sessioni di gruppo.

<div id="pattern-personal-dms-public-groups-single-agent">
  ## Pattern: DM personali + gruppi pubblici (agente singolo)
</div>

Sì — questo funziona bene se il tuo traffico “personale” sono i **DM** e il tuo traffico “pubblico” sono i **gruppi**.

Perché: in modalità agente singolo, i DM in genere arrivano nella chiave di **sessione** principale (`agent:main:main`), mentre i gruppi usano sempre chiavi di **sessione** non principali (`agent:main:<channel>:group:<id>`). Se abiliti la sandbox con `mode: "non-main"`, quelle sessioni di gruppo girano in Docker mentre la tua sessione DM principale rimane sull’host.

Questo ti offre un unico “cervello” di agente (spazio di lavoro condiviso + memoria), ma due modalità di esecuzione:

* **DM**: strumenti completi (host)
* **Gruppi**: sandbox + strumenti con restrizioni (Docker)

> Se ti servono davvero spazi di lavoro/personas separati (“personale” e “pubblico” non devono mai mescolarsi), usa un secondo agente + binding. Vedi [Instradamento multi-agente](/it/concepts/multi-agent).

Esempio (DM su host, gruppi in sandbox + strumenti di sola messaggistica):

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // groups/channels are non-main -> sandboxed
        scope: "session", // strongest isolation (one container per group/channel)
        workspaceAccess: "none"
      }
    }
  },
  tools: {
    sandbox: {
      tools: {
        // Se allow non è vuoto, tutto il resto viene bloccato (deny ha comunque la precedenza).
        allow: ["group:messaging", "group:sessions"],
        deny: ["group:runtime", "group:fs", "group:ui", "nodes", "cron", "gateway"]
      }
    }
  }
}
```

Vuoi che “i gruppi possano vedere solo la cartella X” invece di “nessun accesso all&#39;host”? Mantieni `workspaceAccess: "none"` e monta nella sandbox solo i percorsi presenti nella lista di autorizzati:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
        docker: {
          binds: [
            // percorsoHost:percorsoContainer:mode
            "~/FriendsShared:/data:ro"
          ]
        }
      }
    }
  }
}
```

Correlati:

* Chiavi di configurazione e valori predefiniti: [Configurazione del Gateway](/it/gateway/configuration#agentsdefaultssandbox)
* Debugging dei motivi per cui uno strumento è bloccato: [Sandbox vs Tool Policy vs Elevated](/it/gateway/sandbox-vs-tool-policy-vs-elevated)
* Dettagli sui bind mount: [Sandboxing](/it/gateway/sandboxing#custom-bind-mounts)

<div id="display-labels">
  ## Etichette di visualizzazione
</div>

* Le etichette della UI usano `displayName` quando disponibile, formattate come `<channel>:<token>`.
* `#room` è riservato a stanze/canali; le chat di gruppo usano `g-<slug>` (in minuscolo, spazi -&gt; `-`, mantenere `#@+._-`).

<div id="group-policy">
  ## Criteri di gruppo
</div>

Configura come vengono gestiti, per ciascun canale, i messaggi di gruppo/stanza:

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "disabled", // "open" (permette l'accettazione di messaggi senza restrizioni) | "disabled" | "allowlist"
      groupAllowFrom: ["+15551234567"]
    },
    telegram: {
      groupPolicy: "disabled",
      groupAllowFrom: ["123456789", "@username"]
    },
    signal: {
      groupPolicy: "disabled",
      groupAllowFrom: ["+15551234567"]
    },
    imessage: {
      groupPolicy: "disabled",
      groupAllowFrom: ["chat_id:123"]
    },
    msteams: {
      groupPolicy: "disabled",
      groupAllowFrom: ["user@org.com"]
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "GUILD_ID": { channels: { help: { allow: true } } }
      }
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } }
    },
    matrix: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["@owner:example.org"],
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true }
      }
    }
  }
}
```

| Policy        | Behavior                                                                                           |
| ------------- | -------------------------------------------------------------------------------------------------- |
| `"open"`      | I gruppi ignorano la lista di autorizzati; il controllo basato sulle menzioni si applica comunque. |
| `"disabled"`  | Blocca completamente tutti i messaggi di gruppo.                                                   |
| `"allowlist"` | Consente solo gruppi/stanze che corrispondono alla lista di autorizzati configurata.               |

Note:

* `groupPolicy` è separato dal controllo basato sulle menzioni (che richiede @menzioni).
* WhatsApp/Telegram/Signal/iMessage/Microsoft Teams: usa `groupAllowFrom` (fallback: `allowFrom` esplicito).
* Discord: la lista di autorizzati usa `channels.discord.guilds.<id>.channels`.
* Slack: la lista di autorizzati usa `channels.slack.channels`.
* Matrix: la lista di autorizzati usa `channels.matrix.groups` (ID stanza, alias o nomi). Usa `channels.matrix.groupAllowFrom` per limitare i mittenti; sono supportate anche liste di autorizzati `users` per stanza.
* I DM di gruppo sono controllati separatamente (`channels.discord.dm.*`, `channels.slack.dm.*`).
* La lista di autorizzati di Telegram può corrispondere a ID utente (`"123456789"`, `"telegram:123456789"`, `"tg:123456789"`) o username (`"@alice"` o `"alice"`); i prefissi non fanno distinzione tra maiuscole e minuscole.
* Il valore predefinito è `groupPolicy: "allowlist"`; se la tua lista di autorizzati per i gruppi è vuota, i messaggi di gruppo vengono bloccati.

Modello mentale rapido (ordine di valutazione per i messaggi di gruppo):

1. `groupPolicy` (open/disabled/allowlist)
2. liste di autorizzati per i gruppi (`*.groups`, `*.groupAllowFrom`, lista di autorizzati specifica del canale)
3. controllo basato sulle menzioni (`requireMention`, `/activation`)

<div id="mention-gating-default">
  ## Limitazione tramite menzioni (predefinito)
</div>

I messaggi di gruppo richiedono una menzione, a meno che non sia stata disattivata per il singolo gruppo. I valori predefiniti sono definiti per sottosistema sotto `*.groups."*"`.

Rispondere a un messaggio del bot conta come una menzione implicita (quando il canale supporta i metadati di risposta). Questo vale per Telegram, WhatsApp, Slack, Discord e Microsoft Teams.

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
        "123@g.us": { requireMention: false }
      }
    },
    telegram: {
      groups: {
        "*": { requireMention: true },
        "123456789": { requireMention: false }
      }
    },
    imessage: {
      groups: {
        "*": { requireMention: true },
        "123": { requireMention: false }
      }
    }
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@openclaw", "openclaw", "\\+15555550123"],
          historyLimit: 50
        }
      }
    ]
  }
}
```

Note:

* `mentionPatterns` sono regex case-insensitive.
* I canali che forniscono menzioni esplicite vengono comunque accettati; i pattern fungono da fallback.
* Override per-agente: `agents.list[].groupChat.mentionPatterns` (utile quando più agenti condividono un gruppo).
* Il gating basato su menzioni è applicato solo quando il rilevamento delle menzioni è possibile (sono configurate menzioni native o `mentionPatterns`).
* I default per Discord si trovano in `channels.discord.guilds."*"` (sovrascrivibili per singola guild/canale).
* Il contesto della cronologia del gruppo è incapsulato in modo uniforme su tutti i canali ed è **solo per i messaggi in sospeso** (messaggi saltati a causa del gating per menzioni); usa `messages.groupChat.historyLimit` per il valore predefinito globale e `channels.<channel>.historyLimit` (o `channels.<channel>.accounts.*.historyLimit`) per gli override. Imposta `0` per disabilitare.

<div id="groupchannel-tool-restrictions-optional">
  ## Restrizioni degli strumenti per gruppo/canale (opzionale)
</div>

Alcune configurazioni di canale supportano la limitazione degli strumenti disponibili **all&#39;interno di uno specifico gruppo/stanza/canale**.

* `tools`: consente/nega strumenti per l&#39;intero gruppo.
* `toolsBySender`: eccezioni per singolo mittente all&#39;interno del gruppo (le chiavi sono ID/nome utente/email/numero di telefono del mittente a seconda del canale). Usa `"*"` come carattere jolly.

Ordine di applicazione (vince la regola più specifica):

1. corrispondenza `toolsBySender` del gruppo/canale
2. `tools` del gruppo/canale
3. corrispondenza `toolsBySender` predefinita (`"*"`)
4. `tools` predefiniti (`"*"`)

Esempio (Telegram):

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { tools: { deny: ["exec"] } },
        "-1001234567890": {
          tools: { deny: ["exec", "read", "write"] },
          toolsBySender: {
            "123456789": { alsoAllow: ["exec"] }
          }
        }
      }
    }
  }
}
```

Note:

* Le restrizioni sugli strumenti per gruppi/canali si applicano in aggiunta alla policy globale/di agente sugli strumenti (il deny ha comunque la precedenza).
* Alcuni canali usano strutture di annidamento diverse per stanze/canali (ad es. Discord `guilds.*.channels.*`, Slack `channels.*`, MS Teams `teams.*.channels.*`).

<div id="group-allowlists">
  ## Liste di autorizzati per i gruppi
</div>

Quando `channels.whatsapp.groups`, `channels.telegram.groups` o `channels.imessage.groups` sono configurati, le relative chiavi fungono da lista di autorizzati per i gruppi. Usa `"*"` per autorizzare tutti i gruppi mantenendo comunque il comportamento predefinito delle menzioni.

Casi d’uso comuni (copia/incolla):

1. Disabilitare tutte le risposte nei gruppi

```json5
{
  channels: { whatsapp: { groupPolicy: "disabled" } }
}
```

2. Consenti solo determinati gruppi (WhatsApp)

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "123@g.us": { requireMention: true },
        "456@g.us": { requireMention: false }
      }
    }
  }
}
```

3. Consenti tutti i gruppi ma richiedi una menzione esplicita

```json5
{
  channels: {
    whatsapp: {
      groups: { "*": { requireMention: true } }
    }
  }
}
```

4. Solo il proprietario può attivare il bot nei gruppi (WhatsApp)

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
      groups: { "*": { requireMention: true } }
    }
  }
}
```

<div id="activation-owner-only">
  ## Attivazione (solo proprietario)
</div>

I proprietari del gruppo possono attivare o disattivare la funzione per singolo gruppo:

* `/activation mention`
* `/activation always`

Il proprietario è determinato da `channels.whatsapp.allowFrom` (o dall&#39;E.164 del bot quando non impostato). Invia il comando come messaggio a sé stante. Le altre interfacce al momento ignorano `/activation`.

<div id="context-fields">
  ## Campi di contesto
</div>

Per i gruppi, i payload in ingresso impostano:

* `ChatType=group`
* `GroupSubject` (se noto)
* `GroupMembers` (se noti)
* `WasMentioned` (risultato della verifica della menzione)
* Gli argomenti dei forum di Telegram includono anche `MessageThreadId` e `IsForum`.

Il prompt di sistema dell&#39;agente include un&#39;introduzione al gruppo alla prima interazione di una nuova sessione di gruppo. Ricorda al modello di rispondere come un essere umano, evitare le tabelle in Markdown ed evitare di digitare sequenze letterali `\n`.

<div id="imessage-specifics">
  ## Specifiche iMessage
</div>

* Preferisci `chat_id:<id>` per l’instradamento o per la lista di autorizzati.
* Elenca le chat: `imsg chats --limit 20`.
* Le risposte nei gruppi vengono sempre inviate allo stesso `chat_id`.

<div id="whatsapp-specifics">
  ## Specifiche WhatsApp
</div>

Consulta [Messaggi di gruppo](/it/concepts/group-messages) per il comportamento specifico per WhatsApp (iniezione della cronologia, dettagli sulla gestione delle menzioni).