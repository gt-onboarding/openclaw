---
title: Discord
summary: "Stato del supporto per i bot Discord, funzionalità e configurazione"
read_when:
  - Lavori sulle funzionalità del canale Discord
---

<div id="discord-bot-api">
  # Discord (Bot API)
</div>

Stato: pronto per DM e canali di testo dei server tramite il gateway bot ufficiale di Discord.

<div id="quick-setup-beginner">
  ## Configurazione rapida (per principianti)
</div>

1. Crea un bot Discord e copia il token del bot.
2. Nelle impostazioni dell&#39;app Discord, abilita **Message Content Intent** (e **Server Members Intent** se prevedi di usare la lista di autorizzati o ricerche per nome).
3. Imposta il token per OpenClaw:
   * Env: `DISCORD_BOT_TOKEN=...`
   * Oppure config: `channels.discord.token: "..."`.
   * Se entrambi sono impostati, la config ha la precedenza (la variabile d&#39;ambiente viene usata solo come fallback per l&#39;account predefinito).
4. Invita il bot nel tuo server con i permessi per i messaggi (crea un server privato se ti interessano solo i messaggi diretti/DM).
5. Avvia il Gateway.
6. L&#39;accesso via DM è soggetto per impostazione predefinita ad abbinamento; approva il codice di abbinamento al primo contatto.

Configurazione minima:

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN"
    }
  }
}
```

<div id="goals">
  ## Obiettivi
</div>

* Comunicare con OpenClaw tramite messaggi diretti (DM) su Discord o canali di un server (guild).
* Le chat dirette vengono consolidate nella sessione principale dell&#39;agente (predefinita `agent:main:main`); i canali delle guild restano isolati come `agent:<agentId>:discord:channel:<channelId>` (i nomi visualizzati usano `discord:<guildSlug>#<channelSlug>`).
* I DM di gruppo vengono ignorati per impostazione predefinita; abilitali tramite `channels.discord.dm.groupEnabled` e, facoltativamente, limitane l’uso con `channels.discord.dm.groupChannels`.
* Mantieni l’instradamento deterministico: le risposte tornano sempre al canale da cui sono arrivate.

<div id="how-it-works">
  ## Come funziona
</div>

1. Crea un’applicazione Discord → Bot, abilita le intent necessarie (DM + messaggi nelle guild + contenuto dei messaggi) e recupera il token del bot.
2. Invita il bot al tuo server con i permessi necessari per leggere/inviare messaggi nei canali in cui vuoi usarlo.
3. Configura OpenClaw con `channels.discord.token` (oppure `DISCORD_BOT_TOKEN` come fallback).
4. Esegui il Gateway; il canale Discord viene avviato automaticamente quando è disponibile un token (prima config, poi env come fallback) e `channels.discord.enabled` non è `false`.
   * Se preferisci le variabili d’ambiente, imposta `DISCORD_BOT_TOKEN` (un blocco di config è opzionale).
5. Chat dirette: usa `user:<id>` (oppure una menzione `<@id>`) per l’invio; tutti i turni di conversazione finiscono nella sessione condivisa `main`. Gli ID numerici “nudi” sono ambigui e vengono rifiutati.
6. Canali di guild: usa `channel:<channelId>` per l’invio. Le menzioni sono richieste per impostazione predefinita e possono essere configurate per guild o per canale.
7. Chat dirette: sicure per impostazione predefinita tramite `channels.discord.dm.policy` (default: `"pairing"`). I mittenti sconosciuti ricevono un codice di abbinamento (scade dopo 1 ora); approva tramite `openclaw pairing approve discord <code>`.
   * Per mantenere il vecchio comportamento “aperto a chiunque”: imposta `channels.discord.dm.policy="open"` (modalità open: accetta messaggi senza restrizioni da qualsiasi utente) e `channels.discord.dm.allowFrom=["*"]`.
   * Per usare una lista di autorizzati rigida: imposta `channels.discord.dm.policy="allowlist"` e elenca i mittenti in `channels.discord.dm.allowFrom`.
   * Per ignorare tutti i DM: imposta `channels.discord.dm.enabled=false` oppure `channels.discord.dm.policy="disabled"`.
8. I DM di gruppo sono ignorati per impostazione predefinita; abilitali tramite `channels.discord.dm.groupEnabled` e, facoltativamente, limitane l’uso con `channels.discord.dm.groupChannels`.
9. Regole di guild opzionali: imposta `channels.discord.guilds` con chiavi per id di guild (preferito) o slug, con regole per canale.
10. Comandi nativi opzionali: `commands.native` è per impostazione predefinita `"auto"` (attivo per Discord/Telegram, disattivo per Slack). Sovrascrivi con `channels.discord.commands.native: true|false|"auto"`; `false` cancella i comandi registrati in precedenza. I comandi testuali sono controllati da `commands.text` e devono essere inviati come messaggi `/...` autonomi. Usa `commands.useAccessGroups: false` per bypassare i controlli dei gruppi di accesso per i comandi.
    * Elenco completo dei comandi + config: [Slash commands](/it/tools/slash-commands)
11. Cronologia di contesto della guild opzionale: imposta `channels.discord.historyLimit` (default 20, con fallback a `messages.groupChat.historyLimit`) per includere gli ultimi N messaggi della guild come contesto quando rispondi a una menzione. Imposta `0` per disabilitare.
12. Reazioni: l’agente può attivare reazioni tramite lo strumento `discord` (controllato da `channels.discord.actions.*`).
    * Semantica di rimozione delle reazioni: vedi [/tools/reactions](/it/tools/reactions).
    * Lo strumento `discord` è esposto solo quando il canale corrente è Discord.
13. I comandi nativi usano chiavi di sessione isolate (`agent:<agentId>:discord:slash:<userId>`) invece della sessione condivisa `main`.

Nota: La risoluzione nome → id usa la ricerca dei membri della guild e richiede la Server Members Intent; se il bot non può cercare i membri, usa id o menzioni `<@id>`.
Nota: Gli slug sono in minuscolo con gli spazi sostituiti da `-`. I nomi dei canali sono convertiti in slug senza il `#` iniziale.
Nota: Le righe di contesto della guild `[from:]` includono `author.tag` + `id` per rendere più semplice rispondere menzionando l’autore.

<div id="config-writes">
  ## Scritture della configurazione
</div>

Per impostazione predefinita, Discord può scrivere aggiornamenti della configurazione attivati da `/config set|unset` (richiede `commands.config: true`).

Per disabilitarlo:

```json5
{
  channels: { discord: { configWrites: false } }
}
```

<div id="how-to-create-your-own-bot">
  ## Come creare il tuo bot
</div>

Questa è la configurazione nel “Discord Developer Portal” per eseguire OpenClaw in un canale di un server (guild), ad esempio `#help`.

<div id="1-create-the-discord-app-bot-user">
  ### 1) Crea l&#39;app Discord + l&#39;utente bot
</div>

1. Discord Developer Portal → **Applications** → **New Application**
2. Nella tua app:
   * **Bot** → **Add Bot**
   * Copia il **Bot Token** (è quello che devi inserire in `DISCORD_BOT_TOKEN`)

<div id="2-enable-the-gateway-intents-openclaw-needs">
  ### 2) Abilita le gateway intents di cui OpenClaw ha bisogno
</div>

Discord blocca le “privileged intents” finché non le abiliti esplicitamente.

In **Bot** → **Privileged Gateway Intents**, abilita:

* **Message Content Intent** (necessaria per leggere il testo dei messaggi nella maggior parte dei server; senza, vedrai “Used disallowed intents” oppure il bot si connetterà ma non reagirà ai messaggi)
* **Server Members Intent** (consigliata; necessaria per alcune ricerche di membri/utenti e per il matching con la lista di autorizzati nei server)

Di solito **non** hai bisogno di **Presence Intent**.

<div id="3-generate-an-invite-url-oauth2-url-generator">
  ### 3) Generare un URL di invito (OAuth2 URL Generator)
</div>

Nella tua app: **OAuth2** → **URL Generator**

**Scope**

* ✅ `bot`
* ✅ `applications.commands` (richiesto per i comandi nativi)

**Permessi del bot** (set minimo consigliato)

* ✅ Visualizza canali
* ✅ Invia messaggi
* ✅ Leggi cronologia dei messaggi
* ✅ Incorpora link
* ✅ Allega file
* ✅ Aggiungi reazioni (opzionale ma consigliato)
* ✅ Usa emoji / adesivi esterni (opzionale; solo se li vuoi usare)

Evita **Administrator** a meno che tu non stia effettuando il debug e ti fidi completamente del bot.

Copia l’URL generato, aprilo, seleziona il tuo server e installa il bot.

<div id="4-get-the-ids-guilduserchannel">
  ### 4) Ottieni gli ID (guild/utente/canale)
</div>

Discord usa ID numerici ovunque; la configurazione di OpenClaw preferisce gli ID.

1. Discord (desktop/web) → **Impostazioni utente** → **Avanzate** → abilita **Modalità sviluppatore**
2. Fai clic con il tasto destro:
   * Nome del server → **Copia ID server** (guild id)
   * Canale (es. `#help`) → **Copia ID canale**
   * Il tuo profilo utente → **Copia ID utente**

<div id="5-configure-openclaw">
  ### 5) Configurare OpenClaw
</div>

<div id="token">
  #### Token
</div>

Imposta il token del bot tramite variabile di ambiente (consigliato sui server):

* `DISCORD_BOT_TOKEN=...`

Oppure tramite configurazione:

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN"
    }
  }
}
```

Supporto multi-account: usa `channels.discord.accounts` con token per-account e `name` opzionale. Vedi [`gateway/configuration`](/it/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) per lo schema condiviso.

<div id="allowlist-channel-routing">
  #### Lista di autorizzati + instradamento dei canali
</div>

Esempio: “server singolo, autorizza solo me, autorizza solo #help”:

```json5
{
  channels: {
    discord: {
      enabled: true,
      dm: { enabled: false },
      guilds: {
        "YOUR_GUILD_ID": {
          users: ["YOUR_USER_ID"],
          requireMention: true,
          channels: {
            help: { allow: true, requireMention: true }
          }
        }
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1
      }
    }
  }
}
```

Note:

* `requireMention: true` significa che il bot risponde solo quando viene menzionato (consigliato per i canali condivisi).
* `agents.list[].groupChat.mentionPatterns` (o `messages.groupChat.mentionPatterns`) contano anch&#39;essi come menzioni per i messaggi del server.
* Override multi‑agente: imposta pattern specifici per ogni agente su `agents.list[].groupChat.mentionPatterns`.
* Se `channels` è presente, qualsiasi canale non elencato è negato per impostazione predefinita.
* Usa una voce di canale `"*"` per applicare i valori predefiniti a tutti i canali; le voci di canale esplicite hanno la precedenza sulla wildcard.
* I thread ereditano la configurazione del canale padre (lista di autorizzati, `requireMention`, abilità, prompt, ecc.) a meno che tu non aggiunga esplicitamente l&#39;ID del canale del thread.
* I messaggi generati dal bot vengono ignorati per impostazione predefinita; imposta `channels.discord.allowBots=true` per consentirli (i propri messaggi restano filtrati).
* Avvertenza: se consenti le risposte ad altri bot (`channels.discord.allowBots=true`), previeni loop di risposta bot‑a‑bot usando `requireMention`, le liste di autorizzati `channels.discord.guilds.*.channels.<id>.users` e/o regole/limitazioni chiare in `AGENTS.md` e `SOUL.md`.

<div id="6-verify-it-works">
  ### 6) Verifica che funzioni
</div>

1. Avvia il Gateway.
2. Nel canale del tuo server, invia: `@Krill hello` (o qualunque sia il nome del tuo bot).
3. Se non succede nulla: consulta la sezione **Troubleshooting** qui sotto.

<div id="troubleshooting">
  ### Risoluzione dei problemi
</div>

* Per prima cosa esegui `openclaw doctor` e `openclaw channels status --probe` (avvisi con azioni suggerite + verifiche rapide).
* **&quot;Used disallowed intents&quot;**: abilita **Message Content Intent** (e probabilmente anche **Server Members Intent**) nel Developer Portal, quindi riavvia il Gateway.
* **Il bot si connette ma non risponde mai in un canale di una guild**:
  * Manca **Message Content Intent**, oppure
  * Il bot non ha le autorizzazioni sul canale (View/Send/Read History), oppure
  * La tua configurazione richiede le menzioni e non lo hai menzionato, oppure
  * La tua lista di autorizzati per guild/canale esclude quel canale/utente.
* **`requireMention: false` ma ancora nessuna risposta**:
* `channels.discord.groupPolicy` è **allowlist** per impostazione predefinita; impostalo su &quot;open&quot; (consente di accettare messaggi senza restrizioni da qualsiasi utente) oppure aggiungi una voce di guild sotto `channels.discord.guilds` (facoltativamente elenca i canali sotto `channels.discord.guilds.<id>.channels` per limitarli).
  * Se imposti solo `DISCORD_BOT_TOKEN` e non crei mai una sezione `channels.discord`, il runtime
    imposta `groupPolicy` su `open` per impostazione predefinita. Aggiungi `channels.discord.groupPolicy`,
    `channels.defaults.groupPolicy` o una lista di autorizzati per guild/canale per restringere l’accesso.
* `requireMention` deve trovarsi sotto `channels.discord.guilds` (o in un canale specifico). `channels.discord.requireMention` al livello superiore viene ignorato.
* **Verifiche delle autorizzazioni** (`channels status --probe`) eseguono controlli solo sugli ID numerici dei canali. Se usi slug/nome come chiavi di `channels.discord.guilds.*.channels`, la verifica non può controllare le autorizzazioni.
* **I DM non funzionano**: `channels.discord.dm.enabled=false`, `channels.discord.dm.policy="disabled"`, oppure non sei ancora stato approvato (`channels.discord.dm.policy="pairing"`).

<div id="capabilities-limits">
  ## Capacità e limiti
</div>

* DM e canali di testo dei server (guild) (i thread sono trattati come canali separati; canali vocali non supportati).
* Indicatori di digitazione inviati su base best-effort; la suddivisione dei messaggi usa `channels.discord.textChunkLimit` (predefinito 2000) e divide le risposte molto lunghe in base al numero di righe (`channels.discord.maxLinesPerMessage`, predefinito 17).
* Suddivisione opzionale per righe vuote: imposta `channels.discord.chunkMode="newline"` per dividere alle righe vuote (limiti di paragrafo) prima della suddivisione per lunghezza.
* Caricamento di file supportato fino al valore configurato di `channels.discord.mediaMaxMb` (predefinito 8 MB).
* Risposte nei server (guild) limitate alle menzioni per impostazione predefinita per evitare bot rumorosi.
* Il contesto della risposta viene aggiunto automaticamente quando un messaggio fa riferimento a un altro messaggio (contenuto citato + id).
* I thread di risposta nativi sono **disattivati per impostazione predefinita**; abilitali con `channels.discord.replyToMode` e i tag di risposta.

<div id="retry-policy">
  ## Policy di retry
</div>

Le chiamate API Discord in uscita vengono ritentate in caso di rate limit (429), utilizzando `retry_after` di Discord quando disponibile, con backoff esponenziale e jitter. Configurabile tramite `channels.discord.retry`. Vedi [Policy di retry](/it/concepts/retry).

<div id="config">
  ## Configurazione
</div>

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "abc.123",
      groupPolicy: "allowlist",
      guilds: {
        "*": {
          channels: {
            general: { allow: true }
          }
        }
      },
      mediaMaxMb: 8,
      actions: {
        reactions: true,
        stickers: true,
        emojiUploads: true,
        stickerUploads: true,
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
        channels: true,
        voiceStatus: true,
        events: true,
        moderation: false
      },
      replyToMode: "off",
      dm: {
        enabled: true,
        policy: "pairing", // abbinamento | lista di autorizzati | open | disabilitato
        allowFrom: ["123456789012345678", "steipete"],
        groupEnabled: false,
        groupChannels: ["openclaw-dm"]
      },
      guilds: {
        "*": { requireMention: true },
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432", "steipete"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["search", "docs"],
              systemPrompt: "Keep answers short."
            }
          }
        }
      }
    }
  }
}
```

Le reazioni di ack sono gestite globalmente tramite `messages.ackReaction` +
`messages.ackReactionScope`. Usa `messages.removeAckAfterReply` per rimuovere la
reazione di ack dopo che il bot ha risposto.

* `dm.enabled`: imposta `false` per ignorare tutti i DM (predefinito `true`).
* `dm.policy`: controllo di accesso ai DM (`pairing` consigliato). `"open"` richiede `dm.allowFrom=["*"]`.
* `dm.allowFrom`: lista di autorizzati per i DM (id o nomi utente). Usata da `dm.policy="allowlist"` e per la validazione di `dm.policy="open"`. Il wizard accetta username e li risolve in id quando il bot può cercare i membri.
* `dm.groupEnabled`: abilita i DM di gruppo (predefinito `false`).
* `dm.groupChannels`: lista di autorizzati opzionale per id o slug dei canali DM di gruppo.
* `groupPolicy`: controlla la gestione dei canali della guild (`open|disabled|allowlist`); `allowlist` richiede liste di autorizzati per i canali.
* `guilds`: regole per guild indicizzate per id della guild (preferito) o slug.
* `guilds."*"`: impostazioni predefinite per guild applicate quando non esiste una voce esplicita.
* `guilds.<id>.slug`: slug descrittivo opzionale usato per i nomi visualizzati.
* `guilds.<id>.users`: lista di utenti autorizzati opzionale per guild (id o nomi).
* `guilds.<id>.tools`: override opzionali delle policy degli strumenti per guild (`allow`/`deny`/`alsoAllow`) usati quando manca l&#39;override a livello di canale.
* `guilds.<id>.toolsBySender`: override opzionali delle policy degli strumenti per sender a livello di guild (si applicano quando manca l&#39;override a livello di canale; wildcard `"*"` supportato).
* `guilds.<id>.channels.<channel>.allow`: consenti/nega il canale quando `groupPolicy="allowlist"`.
* `guilds.<id>.channels.<channel>.requireMention`: limitazione basata su menzione per il canale.
* `guilds.<id>.channels.<channel>.tools`: override opzionali delle policy degli strumenti per canale (`allow`/`deny`/`alsoAllow`).
* `guilds.<id>.channels.<channel>.toolsBySender`: override opzionali delle policy degli strumenti per sender all&#39;interno del canale (wildcard `"*"` supportato).
* `guilds.<id>.channels.<channel>.users`: lista di utenti autorizzati opzionale per canale.
* `guilds.<id>.channels.<channel>.skills`: filtro delle abilità (omesso = tutte le abilità, vuoto = nessuna).
* `guilds.<id>.channels.<channel>.systemPrompt`: system prompt aggiuntivo per il canale (combinato con l’argomento del canale).
* `guilds.<id>.channels.<channel>.enabled`: imposta `false` per disabilitare il canale.
* `guilds.<id>.channels`: regole dei canali (le chiavi sono gli slug o gli id dei canali).
* `guilds.<id>.requireMention`: requisito di menzione per guild (sovrascrivibile per canale).
* `guilds.<id>.reactionNotifications`: modalità degli eventi del sistema di reazioni (`off`, `own`, `all`, `allowlist`).
* `textChunkLimit`: dimensione del blocco di testo in uscita (caratteri). Predefinito: 2000.
* `chunkMode`: `length` (predefinito) suddivide solo quando si supera `textChunkLimit`; `newline` suddivide sulle righe vuote (confini di paragrafo) prima del chunking per lunghezza.
* `maxLinesPerMessage`: limite massimo “morbido” di righe per messaggio. Predefinito: 17.
* `mediaMaxMb`: limita i contenuti multimediali in ingresso salvati su disco.
* `historyLimit`: numero di messaggi recenti della guild da includere come contesto quando rispondi a una menzione (predefinito 20; esegue il fallback a `messages.groupChat.historyLimit`; `0` disabilita).
* `dmHistoryLimit`: limite di cronologia DM in turni utente. Override per utente: `dms["<user_id>"].historyLimit`.
* `retry`: policy di ritentativo per le chiamate Discord API in uscita (attempts, minDelayMs, maxDelayMs, jitter).
* `actions`: gate sugli strumenti per azione; ometti per consentire tutto (imposta `false` per disabilitare).
  * `reactions` (copre reazioni + lettura delle reazioni)
  * `stickers`, `emojiUploads`, `stickerUploads`, `polls`, `permissions`, `messages`, `threads`, `pins`, `search`
  * `memberInfo`, `roleInfo`, `channelInfo`, `voiceStatus`, `events`
  * `channels` (creazione/modifica/eliminazione di canali + categorie + permessi)
  * `roles` (aggiunta/rimozione di ruoli, predefinito `false`)
  * `moderation` (timeout/kick/ban, predefinito `false`)

Le notifiche di reazione usano `guilds.<id>.reactionNotifications`:

* `off`: nessun evento di reazione.
* `own`: reazioni ai messaggi del bot stesso (predefinito).
* `all`: tutte le reazioni su tutti i messaggi.
* `allowlist`: reazioni da `guilds.<id>.users` su tutti i messaggi (lista vuota disabilita).

<div id="tool-action-defaults">
  ### Impostazioni predefinite delle azioni degli strumenti
</div>

| Gruppo di azioni | Predefinito | Note |
| --- | --- | --- |
| reactions | enabled | Reagisci + elenca reazioni + emojiList |
| stickers | enabled | Invia sticker |
| emojiUploads | enabled | Carica emoji |
| stickerUploads | enabled | Carica sticker |
| polls | enabled | Crea sondaggi |
| permissions | enabled | Snapshot delle autorizzazioni del canale |
| messages | enabled | Leggi/invia/modifica/elimina |
| threads | enabled | Crea/elenca/rispondi |
| pins | enabled | Fissa/rimuovi/elenca |
| search | enabled | Ricerca messaggi (funzionalità in anteprima) |
| memberInfo | enabled | Info membro |
| roleInfo | enabled | Elenco ruoli |
| channelInfo | enabled | Info canale + elenco |
| channels | enabled | Gestione canali/categorie |
| voiceStatus | enabled | Consultazione stato vocale |
| events | enabled | Elenca/crea eventi programmati |
| roles | disabled | Aggiungi/rimuovi ruoli |
| moderation | disabled | Timeout/kick/ban |

* `replyToMode`: `off` (predefinito), `first` o `all`. Si applica solo quando il modello include un tag di risposta.

<div id="reply-tags">
  ## Tag di risposta
</div>

Per richiedere una risposta in thread, il modello può includere un tag nel suo output:

* `[[reply_to_current]]` — risponde al messaggio Discord che ha attivato l’azione.
* `[[reply_to:<id>]]` — risponde a uno specifico id di messaggio dal contesto/dalla cronologia.
  Gli id dei messaggi correnti vengono aggiunti ai prompt come `[message_id: …]`; le voci di cronologia includono già gli id.

Il comportamento è controllato da `channels.discord.replyToMode`:

* `off`: ignora i tag.
* `first`: solo il primo chunk/allegato in uscita è una risposta.
* `all`: ogni chunk/allegato in uscita è una risposta.

Note sulla corrispondenza con la lista di autorizzati:

* `allowFrom`/`users`/`groupChannels` accettano id, nomi, tag o mention come `<@id>`.
* Sono supportati prefissi come `discord:`/`user:` (utenti) e `channel:` (DM di gruppo).
* Usa `*` per consentire qualsiasi mittente/canale.
* Quando `guilds.<id>.channels` è presente, i canali non elencati vengono negati per impostazione predefinita.
* Quando `guilds.<id>.channels` è omesso, tutti i canali nella guild in lista di autorizzati sono consentiti.
* Per consentire **nessun canale**, imposta `channels.discord.groupPolicy: "disabled"` (o mantieni una lista di autorizzati vuota).
* Il wizard di configurazione accetta nomi di `Guild/Channel` (pubblici + privati) e li risolve in ID quando possibile.
* All’avvio, OpenClaw risolve i nomi di canali/utenti nelle liste di autorizzati in ID (quando il bot può cercare i membri)
  e registra l’associazione nei log; le voci non risolte vengono mantenute come digitate.

Note sui comandi nativi:

* I comandi registrati rispecchiano i comandi di chat di OpenClaw.
* I comandi nativi rispettano le stesse liste di autorizzati dei DM/messaggi nelle guild (`channels.discord.dm.allowFrom`, `channels.discord.guilds`, regole per canale).
* Gli slash commands possono comunque essere visibili nella UI di Discord per utenti che non sono nella lista di autorizzati; OpenClaw applica le liste di autorizzati in fase di esecuzione e risponde “non autorizzato”.

<div id="tool-actions">
  ## Azioni dello strumento
</div>

L&#39;agente può chiamare `discord` con azioni come:

* `react` / `reactions` (aggiungere o visualizzare le reazioni)
* `sticker`, `poll`, `permissions`
* `readMessages`, `sendMessage`, `editMessage`, `deleteMessage`
* I payload degli strumenti di lettura/ricerca/pin includono `timestampMs` normalizzato (ms dall&#39;epoch UTC) e `timestampUtc` insieme al `timestamp` grezzo di Discord.
* `threadCreate`, `threadList`, `threadReply`
* `pinMessage`, `unpinMessage`, `listPins`
* `searchMessages`, `memberInfo`, `roleInfo`, `roleAdd`, `roleRemove`, `emojiList`
* `channelInfo`, `channelList`, `voiceStatus`, `eventList`, `eventCreate`
* `timeout`, `kick`, `ban`

Gli ID dei messaggi Discord sono esposti nel contesto iniettato (`[discord message id: …]` e righe di cronologia) in modo che l&#39;agente possa indirizzarli in modo mirato.
Le emoji possono essere Unicode (ad esempio `✅`) oppure usare la sintassi delle emoji personalizzate, come `<:party_blob:1234567890>`.

<div id="safety-ops">
  ## Sicurezza e operazioni
</div>

* Tratta il token del bot come una password; sui nodi supervisionati usa preferibilmente la variabile d&#39;ambiente `DISCORD_BOT_TOKEN` oppure limita rigorosamente i permessi del file di configurazione.
* Concedi al bot solo le autorizzazioni di cui ha effettivamente bisogno (tipicamente &quot;Read/Send Messages&quot;).
* Se il bot è bloccato o soggetto a limitazioni di frequenza, riavvia il Gateway (`openclaw gateway --force`) dopo aver verificato che nessun altro processo stia utilizzando la sessione Discord.