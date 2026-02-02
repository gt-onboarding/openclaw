---
title: Telegram
summary: "Stato del supporto, funzionalità e configurazione dei bot Telegram"
read_when:
  - Stai lavorando su funzionalità o webhook di Telegram
---

<div id="telegram-bot-api">
  # Telegram (Bot API)
</div>

Stato: pronto per l’uso in produzione per DM al bot e gruppi tramite grammY. Long-polling per impostazione predefinita; webhook opzionale.

<div id="quick-setup-beginner">
  ## Configurazione rapida (principianti)
</div>

1. Crea un bot con **@BotFather** e copia il token.
2. Imposta il token:
   * Env: `TELEGRAM_BOT_TOKEN=...`
   * Oppure config: `channels.telegram.botToken: "..."`.
   * Se entrambi sono impostati, la configurazione ha la precedenza (la variabile d’ambiente è il fallback solo per l’account predefinito).
3. Avvia il Gateway.
4. L’accesso via DM richiede l’abbinamento per impostazione predefinita; approva il codice di abbinamento al primo contatto.

Configurazione minima:

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing"
    }
  }
}
```

<div id="what-it-is">
  ## Che cos&#39;è
</div>

* Un canale Telegram Bot API di proprietà del Gateway.
* Instradamento deterministico: le risposte vengono sempre inviate a Telegram; il modello non sceglie mai i canali.
* I DM condividono la sessione principale dell&#39;agente; i gruppi restano isolati (`agent:<agentId>:telegram:group:<chatId>`).

<div id="setup-fast-path">
  ## Configurazione rapida
</div>

<div id="1-create-a-bot-token-botfather">
  ### 1) Crea un token bot (BotFather)
</div>

1. Apri Telegram e avvia una chat con **@BotFather**.
2. Invia il comando `/newbot`, quindi segui le istruzioni (nome + username che termina con `bot`).
3. Copia il token e conservalo in un luogo sicuro.

Impostazioni opzionali di BotFather:

* `/setjoingroups` — permette/nega l&#39;aggiunta del bot ai gruppi.
* `/setprivacy` — controlla se il bot può vedere tutti i messaggi del gruppo.

<div id="2-configure-the-token-env-or-config">
  ### 2) Configura il token (env o config)
</div>

Esempio:

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } }
    }
  }
}
```

Opzione env: `TELEGRAM_BOT_TOKEN=...` (funziona per l&#39;account predefinito).
Se sia env che config sono impostati, la config ha la precedenza.

Supporto multi-account: usa `channels.telegram.accounts` con token per account e `name` opzionale. Vedi [`gateway/configuration`](/it/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) per il pattern condiviso.

3. Avvia il Gateway. Telegram si avvia quando un token viene risolto (prima config, poi fallback su env).
4. L&#39;accesso tramite DM usa l&#39;abbinamento per impostazione predefinita. Approva il codice quando il bot viene contattato per la prima volta.
5. Per i gruppi: aggiungi il bot, decidi il comportamento privacy/admin (sotto), poi imposta `channels.telegram.groups` per controllare il gating tramite menzioni e la lista di autorizzati.

<div id="token-privacy-permissions-telegram-side">
  ## Token + privacy + permessi (lato Telegram)
</div>

<div id="token-creation-botfather">
  ### Creazione del token (BotFather)
</div>

* `/newbot` crea il bot e ti fornisce il token (mantienilo segreto).
* Se un token viene compromesso, revocalo o rigeneralo tramite @BotFather e aggiorna la tua configurazione.

<div id="group-message-visibility-privacy-mode">
  ### Visibilità dei messaggi nei gruppi (Modalità privacy)
</div>

I bot Telegram usano per impostazione predefinita la **Modalità privacy**, che limita quali messaggi di gruppo possono ricevere.
Se il tuo bot deve poter vedere *tutti* i messaggi del gruppo, hai due opzioni:

* Disabilitare la modalità privacy con `/setprivacy` **oppure**
* Aggiungere il bot come **amministratore** del gruppo (i bot amministratori ricevono tutti i messaggi).

**Nota:** Quando attivi o disattivi la modalità privacy, Telegram richiede di rimuovere e aggiungere nuovamente il bot
a ogni gruppo affinché la modifica abbia effetto.

<div id="group-permissions-admin-rights">
  ### Permessi del gruppo (diritti di amministratore)
</div>

Lo stato di amministratore viene configurato all&#39;interno del gruppo (UI Telegram). I bot con privilegi di amministratore ricevono sempre tutti i messaggi del gruppo, quindi usa i privilegi di amministratore se ti serve piena visibilità.

<div id="how-it-works-behavior">
  ## Come funziona (comportamento)
</div>

* I messaggi in ingresso vengono normalizzati nella busta canale condivisa, con contesto di risposta e segnaposto per i media.
* Le risposte nei gruppi richiedono per impostazione predefinita una menzione (menzione nativa @ oppure `agents.list[].groupChat.mentionPatterns` / `messages.groupChat.mentionPatterns`).
* Override multi-agente: imposta pattern per singolo agente in `agents.list[].groupChat.mentionPatterns`.
* Le risposte vengono sempre instradate alla stessa chat Telegram.
* Il long-polling usa grammY runner con sequenziamento per chat; la concorrenza complessiva è limitata da `agents.defaults.maxConcurrent`.
* La Telegram Bot API non supporta le conferme di lettura; non esiste l&#39;opzione `sendReadReceipts`.

<div id="formatting-telegram-html">
  ## Formattazione (Telegram HTML)
</div>

* Il testo Telegram in uscita usa `parse_mode: "HTML"` (il sottoinsieme di tag supportato da Telegram).
* L’input in stile Markdown viene convertito in **HTML compatibile con Telegram** (grassetto/corsivo/barrato/codice/link); gli elementi a blocco vengono appiattiti in testo con a capo/punti elenco.
* L’HTML grezzo prodotto dai modelli viene sottoposto a escaping per evitare errori di parsing di Telegram.
* Se Telegram rifiuta il payload HTML, OpenClaw ritenta l’invio dello stesso messaggio come testo semplice.

<div id="commands-native-custom">
  ## Comandi (nativi + personalizzati)
</div>

OpenClaw registra i comandi nativi (come `/status`, `/reset`, `/model`) nel menu del bot Telegram all&#39;avvio.
Puoi aggiungere comandi personalizzati al menu tramite configurazione:

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" }
      ]
    }
  }
}
```

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

* `setMyCommands failed` nei log di solito significa che l&#39;HTTPS/DNS in uscita verso `api.telegram.org` è bloccato.
* Se vedi errori `sendMessage` o `sendChatAction`, controlla il routing IPv6 e il DNS.

Ulteriore supporto: [Risoluzione dei problemi dei canali](/it/channels/troubleshooting).

Note:

* I comandi personalizzati sono **solo voci di menu**; OpenClaw non li implementa a meno che tu non li gestisca altrove.
* I nomi dei comandi sono normalizzati (slash iniziale `/` rimosso, convertiti in minuscolo) e devono corrispondere a `a-z`, `0-9`, `_` (da 1 a 32 caratteri).
* I comandi personalizzati **non possono sostituire i comandi nativi**. I conflitti vengono ignorati e registrati nei log.
* Se `commands.native` è disabilitato, vengono registrati solo i comandi personalizzati (oppure cancellati se non ce ne sono).

<div id="limits">
  ## Limiti
</div>

* Il testo in uscita viene suddiviso in blocchi secondo `channels.telegram.textChunkLimit` (valore predefinito 4000).
* Suddivisione opzionale per righe vuote: imposta `channels.telegram.chunkMode="newline"` per dividere sulle righe vuote (confini di paragrafo) prima del frazionamento in base alla lunghezza.
* I download e gli upload di contenuti multimediali sono limitati da `channels.telegram.mediaMaxMb` (valore predefinito 5).
* Le richieste alla Telegram Bot API vanno in timeout dopo `channels.telegram.timeoutSeconds` (valore predefinito 500 tramite grammY). Imposta un valore più basso per evitare blocchi prolungati.
* Il contesto della cronologia nei gruppi usa `channels.telegram.historyLimit` (o `channels.telegram.accounts.*.historyLimit`), con fallback a `messages.groupChat.historyLimit`. Imposta `0` per disabilitare (valore predefinito 50).
* La cronologia delle DM può essere limitata con `channels.telegram.dmHistoryLimit` (turni dell&#39;utente). Override per singolo utente: `channels.telegram.dms["<user_id>"].historyLimit`.

<div id="group-activation-modes">
  ## Modalità di attivazione nei gruppi
</div>

Per impostazione predefinita, il bot risponde solo quando viene menzionato nei gruppi (`@botname` o pattern in `agents.list[].groupChat.mentionPatterns`). Per modificare questo comportamento:

<div id="via-config-recommended">
  ### Tramite configurazione (consigliato)
</div>

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": { requireMention: false }  // risponde sempre in questo gruppo
      }
    }
  }
}
```

**Importante:** Impostare `channels.telegram.groups` definisce una **lista di autorizzati**: solo i gruppi elencati (oppure `"*"`) saranno accettati.
Gli argomenti dei forum ereditano la configurazione del gruppo padre (allowFrom, requireMention, abilità, prompts), a meno che tu non aggiunga override specifici per argomento in `channels.telegram.groups.<groupId>.topics.<topicId>`.

Per consentire tutti i gruppi con always-respond:

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: false }  // tutti i gruppi, rispondi sempre
      }
    }
  }
}
```

Per mantenere la modalità solo su menzione per tutti i gruppi (comportamento predefinito):

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: true }  // oppure ometti del tutto groups
      }
    }
  }
}
```

<div id="via-command-session-level">
  ### Tramite comando (a livello di sessione)
</div>

Invia nel gruppo:

* `/activation always` - rispondi a tutti i messaggi
* `/activation mention` - richiedi una menzione (impostazione predefinita)

**Nota:** I comandi aggiornano solo lo stato della sessione. Per mantenere il comportamento attraverso i riavvii, usa la configurazione.

<div id="getting-the-group-chat-id">
  ### Ottenere l&#39;ID della chat di gruppo
</div>

Inoltra qualsiasi messaggio dal gruppo a `@userinfobot` o `@getidsbot` su Telegram per visualizzare l&#39;ID della chat (numero negativo come `-1001234567890`).

**Suggerimento:** Per ottenere il tuo ID utente, invia un DM al bot e ti risponderà con il tuo ID utente (messaggio di abbinamento), oppure usa `/whoami` una volta che i comandi sono abilitati.

**Nota sulla privacy:** `@userinfobot` è un bot di terze parti. Se preferisci, aggiungi il bot al gruppo, invia un messaggio e usa `openclaw logs --follow` per leggere `chat.id`, oppure usa le Bot API `getUpdates`.

<div id="config-writes">
  ## Scrittura della configurazione
</div>

Per impostazione predefinita, Telegram è autorizzato a scrivere aggiornamenti di configurazione attivati da eventi del canale o da `/config set|unset`.

Questo accade quando:

* Un gruppo viene aggiornato a supergruppo e Telegram emette `migrate_to_chat_id` (l&#39;ID chat cambia). OpenClaw può migrare automaticamente `channels.telegram.groups`.
* Esegui `/config set` o `/config unset` in una chat di Telegram (richiede `commands.config: true`).

Disattiva con:

```json5
{
  channels: { telegram: { configWrites: false } }
}
```

<div id="topics-forum-supergroups">
  ## Topic (supergruppi forum)
</div>

I topic dei forum Telegram includono un `message_thread_id` per ogni messaggio. OpenClaw:

* Aggiunge `:topic:<threadId>` alla chiave di sessione del gruppo Telegram in modo che ogni topic sia isolato.
* Invia indicatori di digitazione e risposte con `message_thread_id` così le risposte rimangono nel topic.
* Il topic generale (thread id `1`) è speciale: l’invio dei messaggi omette `message_thread_id` (Telegram lo rifiuta), ma gli indicatori di digitazione lo includono comunque.
* Espone `MessageThreadId` + `IsForum` nel contesto del template per instradamento/templating.
* La configurazione specifica per topic è disponibile in `channels.telegram.groups.<chatId>.topics.<threadId>` (abilità, liste di autorizzati, risposta automatica, prompt di sistema, disabilitazione).
* Le configurazioni dei topic ereditano le impostazioni del gruppo (requireMention, liste di autorizzati, abilità, prompt, enabled) a meno che non vengano sovrascritte per il singolo topic.

Le chat private possono includere `message_thread_id` in alcuni casi limite. OpenClaw mantiene invariata la chiave di sessione dei DM, ma utilizza comunque l’ID del thread per risposte/streaming delle bozze quando è presente.

<div id="inline-buttons">
  ## Pulsanti inline
</div>

Telegram supporta le tastiere inline con pulsanti di callback.

```json5
{
  "channels": {
    "telegram": {
      "capabilities": {
        "inlineButtons": "allowlist"
      }
    }
  }
}
```

Per la configurazione per singolo account:

```json5
{
  "channels": {
    "telegram": {
      "accounts": {
        "main": {
          "capabilities": {
            "inlineButtons": "allowlist"
          }
        }
      }
    }
  }
}
```

Scope:

* `off` — pulsanti inline disabilitati
* `dm` — solo DM (destinatari di gruppo bloccati)
* `group` — solo gruppi (destinatari DM bloccati)
* `all` — DM + gruppi
* `allowlist` — DM + gruppi, ma solo mittenti autorizzati da `allowFrom`/`groupAllowFrom` (stesse regole dei comandi di controllo)

Valore predefinito: `allowlist`.
Legacy: `capabilities: ["inlineButtons"]` = `inlineButtons: "all"`.

<div id="sending-buttons">
  ### Invio dei pulsanti
</div>

Usa lo strumento dei messaggi con il parametro `buttons`:

```json5
{
  "action": "send",
  "channel": "telegram",
  "to": "123456789",
  "message": "Choose an option:",
  "buttons": [
    [
      {"text": "Yes", "callback_data": "yes"},
      {"text": "No", "callback_data": "no"}
    ],
    [
      {"text": "Cancel", "callback_data": "cancel"}
    ]
  ]
}
```

Quando un utente clicca su un pulsante, i dati di callback vengono inviati all&#39;agente come messaggio nel formato:
`callback_data: value`

<div id="configuration-options">
  ### Opzioni di configurazione
</div>

Le funzionalità di Telegram possono essere configurate su due livelli (forma a oggetto mostrata sopra; i vecchi array di stringhe sono ancora supportati):

* `channels.telegram.capabilities`: configurazione globale predefinita delle funzionalità applicata a tutti gli account Telegram, a meno che non venga sovrascritta.
* `channels.telegram.accounts.<account>.capabilities`: funzionalità per singolo account che sovrascrivono i valori globali per quello specifico account.

Usa l&#39;impostazione globale quando tutti i bot/account Telegram devono comportarsi allo stesso modo. Usa la configurazione per singolo account quando bot diversi richiedono comportamenti differenti (per esempio, un account gestisce solo i DM mentre un altro è autorizzato nei gruppi).

<div id="access-control-dms-groups">
  ## Controllo accessi (DM e gruppi)
</div>

<div id="dm-access">
  ### Accesso DM
</div>

* Impostazione predefinita: `channels.telegram.dmPolicy = "pairing"`. I mittenti sconosciuti ricevono un codice di abbinamento; i messaggi vengono ignorati finché non sono approvati (i codici scadono dopo 1 ora).
* Approva tramite:
  * `openclaw pairing list telegram`
  * `openclaw pairing approve telegram <CODE>`
* L’abbinamento è lo scambio di token predefinito utilizzato per i DM di Telegram. Dettagli: [Pairing](/it/start/pairing)
* `channels.telegram.allowFrom` accetta ID utente numerici (consigliato) oppure valori `@username`. **Non** è lo username del bot; utilizza l’ID del mittente umano. La procedura guidata accetta `@username` e lo risolve nell’ID numerico quando possibile.

<div id="finding-your-telegram-user-id">
  #### Trovare il tuo ID utente Telegram
</div>

Più sicuro (nessun bot di terze parti):

1. Avvia il Gateway e manda un DM al tuo bot.
2. Esegui `openclaw logs --follow` e cerca `from.id`.

Alternativa (Bot API ufficiale):

1. Manda un DM al tuo bot.
2. Recupera gli aggiornamenti con il token del tuo bot e leggi `message.from.id`:
   ```bash
   curl "https://api.telegram.org/bot<bot_token>/getUpdates"
   ```

Terze parti (meno privato):

* Manda un DM a `@userinfobot` o `@getidsbot` e usa l&#39;ID utente restituito.

<div id="group-access">
  ### Accesso ai gruppi
</div>

Due controlli indipendenti:

**1. Quali gruppi sono consentiti** (lista di autorizzati dei gruppi tramite `channels.telegram.groups`):

* Nessuna configurazione `groups` = tutti i gruppi sono consentiti
* Con configurazione `groups` = sono consentiti solo i gruppi elencati o `"*"`
* Esempio: `"groups": { "-1001234567890": {}, "*": {} }` consente tutti i gruppi

**2. Quali mittenti sono consentiti** (filtro dei mittenti tramite `channels.telegram.groupPolicy`):

* `"open"` = tutti i mittenti nei gruppi consentiti possono inviare messaggi
* `"allowlist"` = solo i mittenti in `channels.telegram.groupAllowFrom` possono inviare messaggi
* `"disabled"` = nessun messaggio di gruppo viene accettato
  L&#39;impostazione predefinita è `groupPolicy: "allowlist"` (tutto bloccato finché non aggiungi `groupAllowFrom`).

Per la maggior parte degli utenti la configurazione consigliata è: `groupPolicy: "allowlist"` + `groupAllowFrom` + gruppi specifici elencati in `channels.telegram.groups`

<div id="long-polling-vs-webhook">
  ## Long-polling vs webhook
</div>

* Impostazione predefinita: long-polling (non è necessario un URL pubblico).
* Modalità webhook: imposta `channels.telegram.webhookUrl` (facoltativamente `channels.telegram.webhookSecret` + `channels.telegram.webhookPath`).
  * Il listener locale effettua il binding su `0.0.0.0:8787` e serve `POST /telegram-webhook` per impostazione predefinita.
  * Se il tuo URL pubblico è diverso, usa un reverse proxy e punta `channels.telegram.webhookUrl` all&#39;endpoint pubblico.

<div id="reply-threading">
  ## Thread di risposta
</div>

Telegram supporta risposte opzionali in thread tramite tag:

* `[[reply_to_current]]` -- rispondi al messaggio che ha attivato l&#39;azione.
* `[[reply_to:<id>]]` -- rispondi a un messaggio specifico tramite il suo ID.

Determinato da `channels.telegram.replyToMode`:

* `first` (predefinito), `all`, `off`.

<div id="audio-messages-voice-vs-file">
  ## Messaggi audio (nota vocale vs file)
</div>

Telegram distingue tra **note vocali** (fumetto rotondo) e **file audio** (scheda con metadati).
OpenClaw usa per impostazione predefinita i file audio per mantenere la compatibilità con le versioni precedenti.

Per forzare una bolla di nota vocale nelle risposte dell&#39;Agente, includi questo tag in qualsiasi punto della risposta:

* `[[audio_as_voice]]` — invia l&#39;audio come nota vocale invece che come file.

Il tag viene rimosso dal testo recapitato. Gli altri canali ignorano questo tag.

Per gli invii tramite lo strumento message, imposta `asVoice: true` con un URL `media` audio compatibile con le note vocali
(`message` è opzionale quando `media` è presente):

```json5
{
  "action": "send",
  "channel": "telegram",
  "to": "123456789",
  "media": "https://example.com/voice.ogg",
  "asVoice": true
}
```

<div id="stickers">
  ## Sticker
</div>

OpenClaw supporta la ricezione e l&#39;invio di sticker di Telegram con memorizzazione intelligente nella cache.

<div id="receiving-stickers">
  ### Ricezione degli sticker
</div>

Quando un utente invia uno sticker, OpenClaw lo gestisce in base al tipo di sticker:

* **Sticker statici (WEBP):** Scaricati ed elaborati tramite il modulo di visione. Lo sticker appare come segnaposto `<media:sticker>` nel contenuto del messaggio.
* **Sticker animati (TGS):** Ignorati (formato Lottie non supportato per l&#39;elaborazione).
* **Sticker video (WEBM):** Ignorati (formato video non supportato per l&#39;elaborazione).

Campo di contesto del template disponibile alla ricezione di sticker:

* `Sticker` — oggetto con:
  * `emoji` — emoji associata allo sticker
  * `setName` — nome del set di sticker
  * `fileId` — ID file di Telegram (per reinviare lo stesso sticker)
  * `fileUniqueId` — ID stabile per la ricerca nella cache
  * `cachedDescription` — descrizione generata dal modulo di visione, memorizzata in cache quando disponibile

<div id="sticker-cache">
  ### Cache degli sticker
</div>

Gli sticker vengono elaborati tramite le capacità di visione dell&#39;AI per generare descrizioni. Poiché gli stessi sticker vengono spesso inviati ripetutamente, OpenClaw mette in cache queste descrizioni per evitare chiamate API ridondanti.

**Come funziona:**

1. **Prima volta:** L&#39;immagine dello sticker viene inviata all&#39;AI per l&#39;analisi visiva. L&#39;AI genera una descrizione (ad esempio, &quot;Un gatto dei cartoni animati che saluta con entusiasmo&quot;).
2. **Memorizzazione in cache:** La descrizione viene salvata insieme all&#39;ID del file dello sticker, all&#39;emoji e al nome del set.
3. **Occorrenze successive:** Quando lo stesso sticker viene visto di nuovo, viene usata direttamente la descrizione in cache. L&#39;immagine non viene inviata all&#39;AI.

**Percorso della cache:** `~/.openclaw/telegram/sticker-cache.json`

**Formato delle voci in cache:**

```json
{
  "fileId": "CAACAgIAAxkBAAI...",
  "fileUniqueId": "AgADBAADb6cxG2Y",
  "emoji": "👋",
  "setName": "CoolCats",
  "description": "Un gatto dei cartoni animati che saluta entusiasta",
  "cachedAt": "2026-01-15T10:30:00.000Z"
}
```

**Vantaggi:**

* Riduce i costi delle API evitando chiamate di analisi visiva ripetute per lo stesso sticker
* Tempi di risposta più veloci per gli sticker presenti in cache (nessun ritardo dovuto all’elaborazione visiva)
* Abilita la funzionalità di ricerca degli sticker basata sulle descrizioni memorizzate in cache

La cache viene popolata automaticamente man mano che vengono ricevuti gli sticker. Non è necessaria alcuna gestione manuale della cache.

<div id="sending-stickers">
  ### Invio di sticker
</div>

L&#39;agente può inviare e cercare sticker usando le azioni `sticker` e `sticker-search`. Queste azioni sono disabilitate per impostazione predefinita e devono essere abilitate nella configurazione:

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true
      }
    }
  }
}
```

**Invia uno sticker:**

```json5
{
  "action": "sticker",
  "channel": "telegram",
  "to": "123456789",
  "fileId": "CAACAgIAAxkBAAI..."
}
```

Parametri:

* `fileId` (obbligatorio) — l&#39;ID file dello sticker su Telegram. Ottienilo da `Sticker.fileId` quando ricevi uno sticker, oppure da un risultato di `sticker-search`.
* `replyTo` (opzionale) — ID del messaggio a cui rispondere.
* `threadId` (opzionale) — ID del thread di messaggi per gli argomenti del forum.

**Ricerca degli sticker:**

L&#39;agente può cercare gli sticker nella cache in base a descrizione, emoji o nome del set:

```json5
{
  "action": "sticker-search",
  "channel": "telegram",
  "query": "cat waving",
  "limit": 5
}
```

Restituisce gli sticker corrispondenti presenti nella cache:

```json5
{
  "ok": true,
  "count": 2,
  "stickers": [
    {
      "fileId": "CAACAgIAAxkBAAI...",
      "emoji": "👋",
      "description": "Un gatto dei cartoni che saluta entusiasta",
      "setName": "CoolCats"
    }
  ]
}
```

La ricerca utilizza il fuzzy matching sul testo descrittivo, sui caratteri emoji e sui nomi dei set.

**Esempio con thread:**

```json5
{
  "action": "sticker",
  "channel": "telegram",
  "to": "-1001234567890",
  "fileId": "CAACAgIAAxkBAAI...",
  "replyTo": 42,
  "threadId": 123
}
```

<div id="streaming-drafts">
  ## Streaming (bozze)
</div>

Telegram può eseguire lo streaming delle **bolle di bozza** mentre l&#39;agente genera una risposta.
OpenClaw usa la Bot API `sendMessageDraft` (non sono messaggi reali) e poi invia la
risposta finale come un normale messaggio.

Requisiti (Telegram Bot API 9.3+):

* **chat private con argomenti abilitati** (modalità forum topic per il bot).
* I messaggi in ingresso devono includere `message_thread_id` (thread di argomento privato).
* Lo streaming viene ignorato per gruppi/supergruppi/canali.

Config:

* `channels.telegram.streamMode: "off" | "partial" | "block"` (predefinito: `partial`)
  * `partial`: aggiorna la bolla di bozza con il testo più recente in streaming.
  * `block`: aggiorna la bolla di bozza in blocchi più grandi (per chunk).
  * `off`: disabilita lo streaming delle bozze.
* Opzionale (solo per `streamMode: "block"`):
  * `channels.telegram.draftChunk: { minChars?, maxChars?, breakPreference? }`
    * predefiniti: `minChars: 200`, `maxChars: 800`, `breakPreference: "paragraph"` (limitato da `channels.telegram.textChunkLimit`).

Nota: lo streaming delle bozze è separato dallo **streaming a blocchi** (messaggi del canale).
Lo streaming a blocchi è disattivato per impostazione predefinita e richiede `channels.telegram.blockStreaming: true`
se desideri ricevere messaggi Telegram anticipati invece degli aggiornamenti di bozza.

Streaming del ragionamento (solo Telegram):

* `/reasoning stream` esegue lo streaming del ragionamento nella bolla di bozza mentre la risposta viene
  generata, quindi invia la risposta finale senza il ragionamento.
* Se `channels.telegram.streamMode` è `off`, lo streaming del ragionamento è disabilitato.
  Ulteriori dettagli: [Streaming + chunking](/it/concepts/streaming).

<div id="retry-policy">
  ## Criteri di retry
</div>

Le chiamate in uscita alle API di Telegram vengono ritentate automaticamente in caso di errori di rete temporanei o di risposte 429, con backoff esponenziale e jitter. Configura tramite `channels.telegram.retry`. Vedi [Criteri di retry](/it/concepts/retry).

<div id="agent-tool-messages-reactions">
  ## Strumento Agente (messaggi + reazioni)
</div>

* Strumento: `telegram` con azione `sendMessage` (`to`, `content`, opzionale `mediaUrl`, `replyToMessageId`, `messageThreadId`).
* Strumento: `telegram` con azione `react` (`chatId`, `messageId`, `emoji`).
* Strumento: `telegram` con azione `deleteMessage` (`chatId`, `messageId`).
* Semantica della rimozione delle reazioni: vedi [/tools/reactions](/it/tools/reactions).
* Abilitazione degli strumenti: `channels.telegram.actions.reactions`, `channels.telegram.actions.sendMessage`, `channels.telegram.actions.deleteMessage` (predefinito: abilitato) e `channels.telegram.actions.sticker` (predefinito: disabilitato).

<div id="reaction-notifications">
  ## Notifiche di reazione
</div>

**Come funzionano le reazioni:**
Le reazioni di Telegram arrivano come **eventi `message_reaction` separati**, non come proprietà nei payload dei messaggi. Quando un utente aggiunge una reazione, OpenClaw:

1. Riceve l&#39;aggiornamento `message_reaction` dall&#39;API di Telegram
2. Lo converte in un **evento di sistema** con formato: `"Telegram reaction added: {emoji} by {user} on msg {id}"`
3. Accoda l&#39;evento di sistema usando la **stessa chiave di sessione** dei messaggi normali
4. Quando arriva il messaggio successivo in quella conversazione, gli eventi di sistema vengono elaborati e inseriti in testa al contesto dell&#39;agente

L&#39;agente vede le reazioni come **notifiche di sistema** nella cronologia della conversazione, non come metadati del messaggio.

**Configurazione:**

* `channels.telegram.reactionNotifications`: Controlla quali reazioni generano notifiche
  * `"off"` — ignora tutte le reazioni
  * `"own"` — notifica quando gli utenti reagiscono ai messaggi del bot (best-effort; solo in memoria) (default)
  * `"all"` — notifica per tutte le reazioni

* `channels.telegram.reactionLevel`: Controlla la capacità di reazione dell&#39;agente
  * `"off"` — l&#39;agente non può reagire ai messaggi
  * `"ack"` — il bot invia reazioni di conferma (👀 durante l&#39;elaborazione) (default)
  * `"minimal"` — l&#39;agente può reagire con parsimonia (linea guida: 1 ogni 5-10 scambi)
  * `"extensive"` — l&#39;agente può reagire liberamente quando appropriato

**Forum groups:** Le reazioni nei forum groups includono `message_thread_id` e usano chiavi di sessione del tipo `agent:main:telegram:group:{chatId}:topic:{threadId}`. Questo garantisce che reazioni e messaggi nello stesso argomento restino insieme.

**Esempio di configurazione:**

```json5
{
  channels: {
    telegram: {
      reactionNotifications: "all",  // Visualizza tutte le reazioni
      reactionLevel: "minimal"        // L'agente può reagire in modo limitato
    }
  }
}
```

**Requisiti:**

* I bot di Telegram devono richiedere esplicitamente `message_reaction` nel campo `allowed_updates` (configurato automaticamente da OpenClaw)
* In modalità webhook, le reazioni sono incluse nel campo `allowed_updates` del webhook
* In modalità polling, le reazioni sono incluse nel campo `allowed_updates` di `getUpdates`

<div id="delivery-targets-clicron">
  ## Destinazioni di recapito (CLI/cron)
</div>

* Usa un chat ID (`123456789`) o uno username (`@name`) come destinatario.
* Esempio: `openclaw message send --channel telegram --target 123456789 --message "hi"`.

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

**Il bot non risponde ai messaggi senza menzione in un gruppo:**

* Se hai impostato `channels.telegram.groups.*.requireMention=false`, la **modalità privacy** della Bot API di Telegram deve essere disattivata.
  * BotFather: `/setprivacy` → **Disable** (poi rimuovi e riaggiungi il bot al gruppo)
* `openclaw channels status` mostra un avviso quando la configurazione si aspetta messaggi di gruppo senza menzione.
* `openclaw channels status --probe` può inoltre verificare l&#39;appartenenza per ID numerici espliciti dei gruppi (non può controllare le regole con wildcard `"*"`).
* Test rapido: `/activation always` (solo per la sessione; usa la configurazione per la persistenza)

**Il bot non vede per niente i messaggi del gruppo:**

* Se `channels.telegram.groups` è impostato, il gruppo deve essere elencato oppure usare `"*"`
* Controlla le impostazioni di privacy in @BotFather → &quot;Group Privacy&quot; deve essere **OFF**
* Verifica che il bot sia effettivamente membro (non solo admin senza accesso in lettura)
* Controlla i log del Gateway: `openclaw logs --follow` (cerca &quot;skipping group message&quot;)

**Il bot risponde alle menzioni ma non a `/activation always`:**

* Il comando `/activation` aggiorna lo stato della sessione ma non lo rende persistente nella configurazione
* Per un comportamento persistente, aggiungi il gruppo a `channels.telegram.groups` con `requireMention: false`

**Comandi come `/status` non funzionano:**

* Assicurati che il tuo ID utente Telegram sia autorizzato (tramite abbinamento o `channels.telegram.allowFrom`)
* I comandi richiedono autorizzazione anche nei gruppi con `groupPolicy: "open"`

**Il long-polling si interrompe subito su Node 22+ (spesso con proxy/fetch personalizzati):**

* Node 22+ è più rigoroso sugli oggetti `AbortSignal`; segnali esterni possono interrompere le chiamate `fetch` immediatamente.
* Aggiorna a una build di OpenClaw che normalizza i segnali di abort, oppure esegui il Gateway su Node 20 finché non puoi aggiornare.

**Il bot parte, poi smette di rispondere in silenzio (o nei log appare `HttpError: Network request ... failed`):**

* Alcuni host risolvono `api.telegram.org` prima su IPv6. Se il tuo server non ha un’uscita IPv6 funzionante, grammY può bloccarsi su richieste solo IPv6.
* Risolvi abilitando l’uscita IPv6 **oppure** forzando la risoluzione IPv4 per `api.telegram.org` (per esempio aggiungi una voce in `/etc/hosts` usando il record IPv4 A, oppure preferisci IPv4 nello stack DNS del tuo sistema operativo), quindi riavvia il Gateway.
* Verifica rapida: `dig +short api.telegram.org A` e `dig +short api.telegram.org AAAA` per confermare cosa restituisce il DNS.

<div id="configuration-reference-telegram">
  ## Riferimento di configurazione (Telegram)
</div>

Configurazione completa: [Configuration](/it/gateway/configuration)

Opzioni del provider:

* `channels.telegram.enabled`: abilita/disabilita l&#39;avvio del canale.
* `channels.telegram.botToken`: token del bot (BotFather).
* `channels.telegram.tokenFile`: legge il token dal percorso di un file.
* `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (predefinito: pairing).
* `channels.telegram.allowFrom`: lista di autorizzati per DM (id/username). `open` richiede `"*"`.
* `channels.telegram.groupPolicy`: `open | allowlist | disabled` (predefinito: allowlist).
* `channels.telegram.groupAllowFrom`: lista di autorizzati per i mittenti nei gruppi (id/username).
* `channels.telegram.groups`: impostazioni predefinite per gruppo + lista di autorizzati (usa `"*"` per i valori predefiniti globali).
  * `channels.telegram.groups.<id>.requireMention`: impostazione predefinita del controllo di accesso basato sulle menzioni.
  * `channels.telegram.groups.<id>.skills`: filtro delle abilità (omesso = tutte le abilità, vuoto = nessuna).
  * `channels.telegram.groups.<id>.allowFrom`: override per gruppo della lista di autorizzati dei mittenti.
  * `channels.telegram.groups.<id>.systemPrompt`: prompt di sistema aggiuntivo per il gruppo.
  * `channels.telegram.groups.<id>.enabled`: disabilita il gruppo quando è `false`.
  * `channels.telegram.groups.<id>.topics.<threadId>.*`: override per argomento (stessi campi del gruppo).
  * `channels.telegram.groups.<id>.topics.<threadId>.requireMention`: override per argomento del controllo di accesso basato sulle menzioni.
* `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (predefinito: allowlist).
* `channels.telegram.accounts.<account>.capabilities.inlineButtons`: override per account.
* `channels.telegram.replyToMode`: `off | first | all` (predefinito: `first`).
* `channels.telegram.textChunkLimit`: dimensione dei chunk in uscita (caratteri).
* `channels.telegram.chunkMode`: `length` (predefinito) oppure `newline` per dividere sulle righe vuote (limiti di paragrafo) prima del chunking per lunghezza.
* `channels.telegram.linkPreview`: attiva/disattiva le anteprime dei link per i messaggi in uscita (predefinito: true).
* `channels.telegram.streamMode`: `off | partial | block` (streaming delle bozze).
* `channels.telegram.mediaMaxMb`: limite per i media in entrata/uscita (MB).
* `channels.telegram.retry`: criteri di retry per le chiamate API Telegram in uscita (attempts, minDelayMs, maxDelayMs, jitter).
* `channels.telegram.network.autoSelectFamily`: override di Node autoSelectFamily (true=abilita, false=disabilita). Predefinito disabilitato su Node 22 per evitare timeout dovuti a Happy Eyeballs.
* `channels.telegram.proxy`: URL del proxy per le chiamate Bot API (SOCKS/HTTP).
* `channels.telegram.webhookUrl`: abilita la modalità webhook.
* `channels.telegram.webhookSecret`: secret del webhook (opzionale).
* `channels.telegram.webhookPath`: percorso locale del webhook (predefinito `/telegram-webhook`).
* `channels.telegram.actions.reactions`: controlla le reazioni degli strumenti Telegram.
* `channels.telegram.actions.sendMessage`: controlla l&#39;invio di messaggi degli strumenti Telegram.
* `channels.telegram.actions.deleteMessage`: controlla l&#39;eliminazione di messaggi degli strumenti Telegram.
* `channels.telegram.actions.sticker`: controlla le azioni sugli sticker di Telegram — invio e ricerca (predefinito: false).
* `channels.telegram.reactionNotifications`: `off | own | all` — controlla quali reazioni generano eventi di sistema (predefinito: `own` se non impostato).
* `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — controlla la capacità di reazione dell&#39;agente (predefinito: `minimal` se non impostato).

Opzioni globali correlate:

* `agents.list[].groupChat.mentionPatterns` (pattern di controllo basato sulle menzioni).
* `messages.groupChat.mentionPatterns` (fallback globale).
* `commands.native` (predefinito `"auto"` → attivo per Telegram/Discord, disattivo per Slack), `commands.text`, `commands.useAccessGroups` (comportamento dei comandi). Override con `channels.telegram.commands.native`.
* `messages.responsePrefix`, `messages.ackReaction`, `messages.ackReactionScope`, `messages.removeAckAfterReply`.