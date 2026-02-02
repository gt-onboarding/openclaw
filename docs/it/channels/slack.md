---
title: Slack
summary: "Configurazione di Slack per la modalità socket o webhook HTTP"
read_when: "Quando configuri Slack o esegui il debug della modalità socket/HTTP di Slack"
---

<div id="slack">
  # Slack
</div>

<div id="socket-mode-default">
  ## Modalità socket (impostazione predefinita)
</div>

<div id="quick-setup-beginner">
  ### Configurazione rapida (principianti)
</div>

1. Crea un&#39;app Slack e abilita **Socket Mode**.
2. Crea un **App Token** (`xapp-...`) e un **Bot Token** (`xoxb-...`).
3. Imposta i token in OpenClaw e avvia il Gateway.

Configurazione minima:

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-..."
    }
  }
}
```

<div id="setup">
  ### Configurazione
</div>

1. Crea un&#39;app Slack (da zero) su https://api.slack.com/apps.
2. **Socket Mode** → attivalo. Poi vai a **Basic Information** → **App-Level Tokens** → **Generate Token and Scopes** con scope `connections:write`. Copia l&#39;**App Token** (`xapp-...`).
3. **OAuth &amp; Permissions** → aggiungi gli scope del token bot (usa il manifest qui sotto). Fai clic su **Install to Workspace**. Copia il **Bot User OAuth Token** (`xoxb-...`).
4. Opzionale: **OAuth &amp; Permissions** → aggiungi gli **User Token Scopes** (vedi l&#39;elenco in sola lettura qui sotto). Reinstalla l&#39;app e copia lo **User OAuth Token** (`xoxp-...`).
5. **Event Subscriptions** → abilita gli eventi e iscriviti a:
   * `message.*` (include modifiche/eliminazioni/broadcast nei thread)
   * `app_mention`
   * `reaction_added`, `reaction_removed`
   * `member_joined_channel`, `member_left_channel`
   * `channel_rename`
   * `pin_added`, `pin_removed`
6. Invita il bot ai canali che vuoi che possa leggere.
7. Slash Commands → crea `/openclaw` se usi `channels.slack.slashCommand`. Se abiliti i comandi nativi, aggiungi uno slash command per ogni comando integrato (stessi nomi di `/help`). I comandi nativi sono disattivati per impostazione predefinita su Slack a meno che tu non imposti `channels.slack.commands.native: true` (il valore globale di `commands.native` è `"auto"`, che lascia Slack disattivato).
8. App Home → abilita la **Messages Tab** in modo che gli utenti possano inviare DM al bot.

Usa il manifest qui sotto in modo che scope ed eventi restino sincronizzati.

Supporto multi-account: usa `channels.slack.accounts` con token per account e `name` opzionale. Vedi [`gateway/configuration`](/it/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) per lo schema condiviso.

<div id="openclaw-config-minimal">
  ### Configurazione di OpenClaw (minimale)
</div>

Imposta i token tramite variabili di ambiente (consigliato):

* `SLACK_APP_TOKEN=xapp-...`
* `SLACK_BOT_TOKEN=xoxb-...`

Oppure tramite configurazione:

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-..."
    }
  }
}
```

<div id="user-token-optional">
  ### User token (opzionale)
</div>

OpenClaw può usare un token utente Slack (`xoxp-...`) per operazioni di lettura (cronologia,
pin, reazioni, emoji, info sui membri). Per impostazione predefinita resta in sola lettura: le letture
preferiscono il token utente quando è presente, e le scritture usano comunque il token bot a meno che
tu non abiliti esplicitamente il contrario. Anche con `userTokenReadOnly: false`, il token bot resta
preferito per le scritture quando è disponibile.

I token utente vengono configurati nel file di configurazione (nessun supporto per variabili d&#39;ambiente). Per
più account, imposta `channels.slack.accounts.<id>.userToken`.

Esempio con token bot + app + utente:

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-...",
      userToken: "xoxp-..."
    }
  }
}
```

Esempio con userTokenReadOnly impostato esplicitamente (consente la scrittura con il token utente):

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-...",
      userToken: "xoxp-...",
      userTokenReadOnly: false
    }
  }
}
```

<div id="token-usage">
  #### Utilizzo dei token
</div>

* Le operazioni di lettura (cronologia, elenco delle reazioni, elenco dei pin, elenco delle emoji, informazioni sui membri,
  ricerca) usano preferibilmente il token utente quando è configurato, altrimenti il token del bot.
* Le operazioni di scrittura (invio/modifica/eliminazione dei messaggi, aggiunta/rimozione delle reazioni, aggiunta/rimozione dai pin,
  caricamento di file) utilizzano il token del bot per impostazione predefinita. Se `userTokenReadOnly: false` e
  non è disponibile alcun token bot, OpenClaw ricorre al token utente.

<div id="history-context">
  ### Contesto della cronologia
</div>

* `channels.slack.historyLimit` (o `channels.slack.accounts.*.historyLimit`) controlla quanti messaggi recenti del canale/gruppo vengono inclusi nel prompt.
* Se non impostato, usa `messages.groupChat.historyLimit`. Imposta `0` per disabilitare (valore predefinito 50).

<div id="http-mode-events-api">
  ## Modalità HTTP (Events API)
</div>

Usa la modalità HTTP tramite webhook quando il tuo Gateway è raggiungibile da Slack tramite HTTPS (tipico per le installazioni su server).
La modalità HTTP utilizza Events API + Interactivity + Slash Commands con un URL di richiesta condiviso.

<div id="setup">
  ### Configurazione
</div>

1. Crea un&#39;app Slack e **disabilita Socket Mode** (opzionale se utilizzi solo la modalità HTTP).
2. Vai su **Basic Information** → copia il **Signing Secret**.
3. Vai su **OAuth &amp; Permissions** → installa l&#39;app e copia il **Bot User OAuth Token** (`xoxb-...`).
4. Vai su **Event Subscriptions** → abilita gli eventi e imposta la **Request URL** sul percorso webhook del tuo Gateway (predefinito `/slack/events`).
5. Vai su **Interactivity &amp; Shortcuts** → abilita e imposta la stessa **Request URL**.
6. Vai su **Slash Commands** → imposta la stessa **Request URL** per i tuoi comandi.

Esempio di URL di richiesta:
`https://gateway-host/slack/events`

### Configurazione di OpenClaw (minima)

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events"
    }
  }
}
```

Modalità HTTP multi-account: imposta `channels.slack.accounts.<id>.mode = "http"` e fornisci un
`webhookPath` univoco per ogni account, in modo che ciascuna app Slack possa puntare al proprio URL.

<div id="manifest-optional">
  ### Manifest (opzionale)
</div>

Usa questo manifest dell&#39;app Slack per creare rapidamente l&#39;app (modifica il nome/comando se desideri). Includi gli scope utente se prevedi di configurare un token utente.

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "groups:write",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "users:read",
        "app_mentions:read",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ],
      "user": [
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "im:history",
        "im:read",
        "mpim:history",
        "mpim:read",
        "users:read",
        "reactions:read",
        "pins:read",
        "emoji:read",
        "search:read"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

Se abiliti i comandi nativi, aggiungi una voce `slash_commands` per ciascun comando che vuoi rendere disponibile (corrispondente all&#39;elenco di `/help`). Puoi sovrascrivere questo comportamento con `channels.slack.commands.native`.

<div id="scopes-current-vs-optional">
  ## Scope (correnti vs opzionali)
</div>

L&#39;API Conversations di Slack utilizza scope distinti per tipo: ti servono solo gli scope per i
tipi di conversazione che effettivamente usi (channels, groups, im, mpim). Consulta
https://docs.slack.dev/apis/web-api/using-the-conversations-api/ per una panoramica.

<div id="bot-token-scopes-required">
  ### scopes del token bot (obbligatori)
</div>

* `chat:write` (invia/aggiorna/elimina messaggi tramite `chat.postMessage`)
  https://docs.slack.dev/reference/methods/chat.postMessage
* `im:write` (apre i DM tramite `conversations.open` per i DM degli utenti)
  https://docs.slack.dev/reference/methods/conversations.open
* `channels:history`, `groups:history`, `im:history`, `mpim:history`
  https://docs.slack.dev/reference/methods/conversations.history
* `channels:read`, `groups:read`, `im:read`, `mpim:read`
  https://docs.slack.dev/reference/methods/conversations.info
* `users:read` (ricerca utente)
  https://docs.slack.dev/reference/methods/users.info
* `reactions:read`, `reactions:write` (`reactions.get` / `reactions.add`)
  https://docs.slack.dev/reference/methods/reactions.get
  https://docs.slack.dev/reference/methods/reactions.add
* `pins:read`, `pins:write` (`pins.list` / `pins.add` / `pins.remove`)
  https://docs.slack.dev/reference/scopes/pins.read
  https://docs.slack.dev/reference/scopes/pins.write
* `emoji:read` (`emoji.list`)
  https://docs.slack.dev/reference/scopes/emoji.read
* `files:write` (caricamenti tramite `files.uploadV2`)
  https://docs.slack.dev/messaging/working-with-files/#upload

<div id="user-token-scopes-optional-read-only-by-default">
  ### Scope del token utente (opzionali, di sola lettura per impostazione predefinita)
</div>

Aggiungi quanto segue in **User Token Scopes** se configuri `channels.slack.userToken`.

* `channels:history`, `groups:history`, `im:history`, `mpim:history`
* `channels:read`, `groups:read`, `im:read`, `mpim:read`
* `users:read`
* `reactions:read`
* `pins:read`
* `emoji:read`
* `search:read`

<div id="not-needed-today-but-likely-future">
  ### Non necessario per ora (ma probabile in futuro)
</div>

* `mpim:write` (solo se aggiungiamo l&#39;apertura/avvio di DM di gruppo tramite `conversations.open`)
* `groups:write` (solo se aggiungiamo la gestione dei canali privati: creazione/rinomina/invito/archiviazione)
* `chat:write.public` (solo se vogliamo inviare messaggi ai canali in cui il bot non è presente)
  https://docs.slack.dev/reference/scopes/chat.write.public
* `users:read.email` (solo se ci servono i campi email da `users.info`)
  https://docs.slack.dev/changelog/2017-04-narrowing-email-access
* `files:read` (solo se iniziamo a elencare/leggere i metadati dei file)

<div id="config">
  ## Configurazione
</div>

Slack utilizza esclusivamente la modalità Socket (nessun server webhook HTTP). Fornisci entrambi i token:

```json
{
  "slack": {
    "enabled": true,
    "botToken": "xoxb-...",
    "appToken": "xapp-...",
    "groupPolicy": "allowlist",
    "dm": {
      "enabled": true,
      "policy": "pairing",
      "allowFrom": ["U123", "U456", "*"],
      "groupEnabled": false,
      "groupChannels": ["G123"],
      "replyToMode": "all"
    },
    "channels": {
      "C123": { "allow": true, "requireMention": true },
      "#general": {
        "allow": true,
        "requireMention": true,
        "users": ["U123"],
        "skills": ["search", "docs"],
        "systemPrompt": "Keep answers short."
      }
    },
    "reactionNotifications": "own",
    "reactionAllowlist": ["U123"],
    "replyToMode": "off",
    "actions": {
      "reactions": true,
      "messages": true,
      "pins": true,
      "memberInfo": true,
      "emojiList": true
    },
    "slashCommand": {
      "enabled": true,
      "name": "openclaw",
      "sessionPrefix": "slack:slash",
      "ephemeral": true
    },
    "textChunkLimit": 4000,
    "mediaMaxMb": 20
  }
}
```

I token possono anche essere passati tramite variabili di ambiente:

* `SLACK_BOT_TOKEN`
* `SLACK_APP_TOKEN`

Le reazioni di ack sono controllate globalmente tramite `messages.ackReaction` +
`messages.ackReactionScope`. Usa `messages.removeAckAfterReply` per rimuovere
la reazione di ack dopo che il bot ha risposto.

<div id="limits">
  ## Limiti
</div>

* Il testo in uscita viene suddiviso in chunk secondo `channels.slack.textChunkLimit` (valore predefinito 4000).
* Suddivisione opzionale per newline: imposta `channels.slack.chunkMode="newline"` per dividere sulle righe vuote (delimitazioni di paragrafo) prima della suddivisione in base alla lunghezza.
* I caricamenti di contenuti multimediali sono limitati da `channels.slack.mediaMaxMb` (valore predefinito 20).

<div id="reply-threading">
  ## Thread delle risposte
</div>

Per impostazione predefinita, OpenClaw risponde nel canale principale. Usa `channels.slack.replyToMode` per controllare il threading automatico:

| Modalità | Comportamento |
| --- | --- |
| `off` | **Predefinita.** Risponde nel canale principale. Usa il thread solo se il messaggio che ha attivato la risposta era già in un thread. |
| `first` | La prima risposta va al thread (sotto il messaggio che l&#39;ha attivata), le risposte successive vanno al canale principale. Utile per mantenere il contesto visibile evitando al contempo di affollare i thread. |
| `all` | Tutte le risposte vanno al thread. Mantiene le conversazioni contenute ma può ridurre la visibilità. |

La modalità si applica sia alle risposte automatiche che alle chiamate dei tool degli agenti (`slack sendMessage`).

<div id="per-chat-type-threading">
  ### Threading in base al tipo di chat
</div>

Puoi configurare diverse modalità di threading in base al tipo di chat impostando `channels.slack.replyToModeByChatType`:

```json5
{
  channels: {
    slack: {
      replyToMode: "off",        // default for channels
      replyToModeByChatType: {
        direct: "all",           // DMs always thread
        group: "first"           // i DM di gruppo/MPIM usano thread per la prima risposta
      },
    }
  }
}
```

Tipi di chat supportati:

* `direct`: DM 1:1 (Slack `im`)
* `group`: DM di gruppo / MPIM (Slack `mpim`)
* `channel`: canali standard (pubblici/privati)

Precedenza:

1. `replyToModeByChatType.<chatType>`
2. `replyToMode`
3. provider predefinito (`off`)

La chiave legacy `channels.slack.dm.replyToMode` è ancora accettata come fallback per `direct` quando non è impostato alcun override del tipo di chat.

Esempi:

Solo DM nei thread:

```json5
{
  channels: {
    slack: {
      replyToMode: "off",
      replyToModeByChatType: { direct: "all" }
    }
  }
}
```

Raggruppa i thread nei DM ma mantieni i canali nell&#39;elenco principale:

```json5
{
  channels: {
    slack: {
      replyToMode: "off",
      replyToModeByChatType: { group: "first" }
    }
  }
}
```

Usa i thread nei canali, mantieni i DM nella conversazione principale:

```json5
{
  channels: {
    slack: {
      replyToMode: "first",
      replyToModeByChatType: { direct: "off", group: "off" }
    }
  }
}
```

<div id="manual-threading-tags">
  ### Tag di threading manuale
</div>

Per un controllo più fine, usa questi tag nelle risposte degli agenti:

* `[[reply_to_current]]` — rispondi al messaggio che ha attivato l&#39;azione (avvia/continua il thread).
* `[[reply_to:<id>]]` — rispondi a un ID di messaggio specifico.

<div id="sessions-routing">
  ## Sessioni + instradamento
</div>

* I DM condividono la sessione `main` (come su WhatsApp/Telegram).
* I canali sono mappati a sessioni `agent:<agentId>:slack:channel:<channelId>`.
* I comandi slash usano sessioni `agent:<agentId>:slack:slash:<userId>` (prefisso configurabile tramite `channels.slack.slashCommand.sessionPrefix`).
* Se Slack non fornisce `channel_type`, OpenClaw lo deduce dal prefisso dell’ID del canale (`D`, `C`, `G`) e usa per impostazione predefinita `channel` per mantenere stabili le chiavi di sessione.
* La registrazione dei comandi nativi usa `commands.native` (valore predefinito globale `"auto"` → Slack disattivato) e può essere sovrascritta per singolo spazio di lavoro con `channels.slack.commands.native`. I comandi testuali richiedono messaggi `/...` standalone e possono essere disabilitati con `commands.text: false`. I comandi slash di Slack sono gestiti nell’app Slack e non vengono rimossi automaticamente. Usa `commands.useAccessGroups: false` per ignorare i controlli dei gruppi di accesso sui comandi.
* Elenco completo dei comandi + configurazione: [Slash commands](/it/tools/slash-commands)

<div id="dm-security-pairing">
  ## Sicurezza DM (abbinamento)
</div>

* Predefinito: `channels.slack.dm.policy="pairing"` — i mittenti DM sconosciuti ricevono un codice di abbinamento (scade dopo 1 ora).
* Approva tramite: `openclaw pairing approve slack <code>`.
* Per consentire a chiunque: imposta `channels.slack.dm.policy="open"` e `channels.slack.dm.allowFrom=["*"]` (l&#39;impostazione `open` consente di accettare messaggi senza restrizioni da qualsiasi utente).
* `channels.slack.dm.allowFrom` accetta ID utente, handle @utente o email (che vengono risolti all&#39;avvio quando i token lo consentono). La procedura guidata accetta nomi utente e li risolve in ID durante la configurazione quando i token lo consentono.

<div id="group-policy">
  ## Criteri per i gruppi
</div>

* `channels.slack.groupPolicy` controlla la gestione dei canali (`open|disabled|allowlist`).
* `allowlist` richiede che i canali siano elencati in `channels.slack.channels`.
* Se imposti solo `SLACK_BOT_TOKEN`/`SLACK_APP_TOKEN` e non crei mai una sezione `channels.slack`,
  il runtime imposta `groupPolicy` su `open` (impostazione che consente di accettare messaggi senza restrizioni da qualsiasi utente). Aggiungi `channels.slack.groupPolicy`,
  `channels.defaults.groupPolicy` o una allowlist di canali per limitarne l&#39;accesso.
* Il wizard di configurazione accetta nomi `#channel` e li risolve in ID quando possibile
  (pubblici + privati); se esistono più corrispondenze, preferisce il canale attivo.
* All&#39;avvio, OpenClaw risolve i nomi di canale/utente nelle allowlist in ID (quando i token lo consentono)
  e registra nel log la mappatura; le voci non risolte vengono mantenute come digitate.
* Per non consentire **nessun canale**, imposta `channels.slack.groupPolicy: "disabled"` (o mantieni una allowlist vuota).

Opzioni del canale (`channels.slack.channels.<id>` o `channels.slack.channels.<name>`):

* `allow`: consente/nega il canale quando `groupPolicy="allowlist"`.
* `requireMention`: controllo tramite menzione per il canale.
* `tools`: override opzionali dei criteri degli strumenti per canale (`allow`/`deny`/`alsoAllow`).
* `toolsBySender`: override opzionali dei criteri degli strumenti per mittente all&#39;interno del canale (le chiavi sono ID dei mittenti/@handle/email; supportato il jolly `"*"`).
* `allowBots`: consente i messaggi scritti da bot in questo canale (predefinito: false).
* `users`: allowlist di utenti opzionale per canale.
* `skills`: filtro delle abilità (omesso = tutte le abilità, vuoto = nessuna).
* `systemPrompt`: prompt di sistema aggiuntivo per il canale (combinato con argomento/scopo).
* `enabled`: imposta `false` per disabilitare il canale.

<div id="delivery-targets">
  ## Destinazioni di recapito
</div>

Usali con gli invii effettuati da cron/CLI:

* `user:<id>` per i DM
* `channel:<id>` per i canali

<div id="tool-actions">
  ## Azioni degli strumenti
</div>

Le azioni degli strumenti Slack possono essere controllate tramite `channels.slack.actions.*`:

| Gruppo di azioni | Predefinito | Note |
| --- | --- | --- |
| reactions | abilitato | Reagisci + elenco reazioni |
| messages | abilitato | Leggere/inviare/modificare/eliminare |
| pins | abilitato | Fissa/rimuovi/elenca |
| memberInfo | abilitato | Informazioni sui membri |
| emojiList | abilitato | Elenco emoji personalizzate |

<div id="security-notes">
  ## Note sulla sicurezza
</div>

* Le operazioni di scrittura usano per impostazione predefinita il bot token, in modo che le azioni che modificano lo stato restino limitate alle autorizzazioni e all&#39;identità del bot dell&#39;app.
* Impostare `userTokenReadOnly: false` consente di usare il token utente per le operazioni di scrittura quando un bot token non è disponibile, il che significa che le azioni vengono eseguite con l&#39;accesso dell&#39;utente che ha effettuato l&#39;installazione. Tratta il token utente come altamente privilegiato e mantieni rigorosi i gate delle azioni e le liste di autorizzati.
* Se abiliti le scritture con token utente, assicurati che il token utente includa gli scope di scrittura previsti (`chat:write`, `reactions:write`, `pins:write`, `files:write`), altrimenti tali operazioni non andranno a buon fine.

<div id="notes">
  ## Note
</div>

* Il gating tramite menzioni è controllato tramite `channels.slack.channels` (imposta `requireMention` su `true`); anche `agents.list[].groupChat.mentionPatterns` (o `messages.groupChat.mentionPatterns`) contano come menzioni.
* Override multi-agente: imposta pattern per agente in `agents.list[].groupChat.mentionPatterns`.
* Le notifiche di reazione seguono `channels.slack.reactionNotifications` (usa `reactionAllowlist` con modalità `allowlist`).
* I messaggi scritti dai bot vengono ignorati per impostazione predefinita; abilita tramite `channels.slack.allowBots` o `channels.slack.channels.<id>.allowBots`.
* Avviso: se consenti risposte ad altri bot (`channels.slack.allowBots=true` o `channels.slack.channels.<id>.allowBots=true`), previeni loop di risposte bot-verso-bot con `requireMention`, la lista di autorizzati `channels.slack.channels.<id>.users` e/o guardrail chiari in `AGENTS.md` e `SOUL.md`.
* Per lo strumento Slack, la semantica della rimozione delle reazioni è descritta in [/tools/reactions](/it/tools/reactions).
* Gli allegati vengono scaricati nell’archivio multimediale quando consentito e se rientrano nel limite di dimensione.