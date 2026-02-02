---
title: iMessage
summary: "Supporto per iMessage tramite imsg (JSON-RPC su stdio), configurazione e instradamento di chat_id"
read_when:
  - Configurazione del supporto per iMessage
  - Debug dell'invio/ricezione di iMessage
---

<div id="imessage-imsg">
  # iMessage (imsg)
</div>

Stato: integrazione CLI esterna. Il Gateway esegue `imsg rpc` (JSON-RPC tramite stdio).

<div id="quick-setup-beginner">
  ## Configurazione rapida (principiante)
</div>

1. Assicurati che Messaggi sia attivo e che tu abbia effettuato l’accesso su questo Mac.
2. Installa `imsg`:
   * `brew install steipete/tap/imsg`
3. Configura OpenClaw con `channels.imessage.cliPath` e `channels.imessage.dbPath`.
4. Avvia il Gateway e approva tutte le richieste di macOS (Automazione + Accesso completo al disco).

Configurazione minima:

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<you>/Library/Messages/chat.db"
    }
  }
}
```

<div id="what-it-is">
  ## Che cos&#39;è
</div>

* Canale iMessage supportato da `imsg` su macOS.
* Instradamento deterministico: le risposte tornano sempre a iMessage.
* I DM condividono la sessione principale dell&#39;agente; i gruppi sono isolati (`agent:<agentId>:imessage:group:<chat_id>`).
* Se un thread con più partecipanti arriva con `is_group=false`, puoi comunque isolarlo per `chat_id` usando `channels.imessage.groups` (vedi “Thread in stile gruppo” qui sotto).

<div id="config-writes">
  ## Modifiche alla configurazione
</div>

Per impostazione predefinita, iMessage può applicare modifiche alla configurazione attivate da `/config set|unset` (richiede `commands.config: true`).

Per disabilitare:

```json5
{
  channels: { imessage: { configWrites: false } }
}
```

<div id="requirements">
  ## Requisiti
</div>

* macOS con l&#39;accesso a Messaggi effettuato.
* Accesso completo al disco per OpenClaw + `imsg` (accesso al database di Messaggi).
* Autorizzazione Automazione per l&#39;invio.
* `channels.imessage.cliPath` può puntare a qualsiasi comando che faccia da proxy per stdin/stdout (ad esempio, uno script wrapper che effettua SSH su un altro Mac ed esegue `imsg rpc`).

<div id="setup-fast-path">
  ## Configurazione rapida
</div>

1. Assicurati che Messaggi sia connesso su questo Mac.
2. Configura iMessage e avvia il Gateway.

<div id="dedicated-bot-macos-user-for-isolated-identity">
  ### Utente macOS dedicato per il bot (per identità isolata)
</div>

Se vuoi che il bot invii da un&#39;**identità iMessage separata** (e mantenere puliti i tuoi Messaggi personali), usa un Apple ID dedicato + un utente macOS dedicato.

1. Crea un Apple ID dedicato (esempio: `my-cool-bot@icloud.com`).
   * Apple potrebbe richiedere un numero di telefono per la verifica/2FA.
2. Crea un utente macOS (esempio: `openclawhome`) ed esegui l&#39;accesso.
3. Apri Messaggi in quell&#39;utente macOS ed esegui l&#39;accesso a iMessage usando l&#39;Apple ID del bot.
4. Abilita Remote Login (Impostazioni di Sistema → Generali → Condivisione → Remote Login).
5. Installa `imsg`:
   * `brew install steipete/tap/imsg`
6. Configura SSH in modo che `ssh <bot-macos-user>@localhost true` funzioni senza password.
7. Imposta `channels.imessage.accounts.bot.cliPath` su un wrapper SSH che esegua `imsg` come utente del bot.

Al primo avvio, l&#39;invio/la ricezione possono richiedere approvazioni tramite GUI (Automazione + Accesso completo al disco) nell&#39;*utente macOS del bot*. Se `imsg rpc` sembra bloccato o termina, accedi a quell&#39;utente (Condivisione schermo è utile), esegui una volta `imsg chats --limit 1` / `imsg send ...`, approva le richieste, quindi riprova.

Esempio di wrapper (`chmod +x`). Sostituisci `<bot-macos-user`&gt; con il tuo nome utente macOS effettivo:

```bash
#!/usr/bin/env bash
set -euo pipefail

# Esegui prima un SSH interattivo una volta per accettare le chiavi host:
#   ssh <bot-macos-user>@localhost true
exec /usr/bin/ssh -o BatchMode=yes -o ConnectTimeout=5 -T <bot-macos-user>@localhost \
  "/usr/local/bin/imsg" "$@"
```

Esempio di configurazione:

```json5
{
  channels: {
    imessage: {
      enabled: true,
      accounts: {
        bot: {
          name: "Bot",
          enabled: true,
          cliPath: "/path/to/imsg-bot",
          dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db"
        }
      }
    }
  }
}
```

Per configurazioni con un singolo account, usa le opzioni di primo livello (`channels.imessage.cliPath`, `channels.imessage.dbPath`) invece della mappa `accounts`.

<div id="remotessh-variant-optional">
  ### Variante remota/SSH (opzionale)
</div>

Se vuoi usare iMessage su un altro Mac, imposta `channels.imessage.cliPath` su un wrapper che esegua `imsg` sull&#39;host macOS remoto tramite SSH. OpenClaw richiede solo lo stdio.

Esempio di wrapper:

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

**Allegati remoti:** Quando `cliPath` punta a un host remoto via SSH, i percorsi degli allegati nel database di Messages fanno riferimento a file presenti sulla macchina remota. OpenClaw può recuperarli automaticamente tramite SCP configurando `channels.imessage.remoteHost`:

```json5
{
  channels: {
    imessage: {
      cliPath: "~/imsg-ssh",                     // wrapper SSH verso Mac remoto
      remoteHost: "user@gateway-host",           // for SCP file transfer
      includeAttachments: true
    }
  }
}
```

Se `remoteHost` non è configurato, OpenClaw tenta di rilevarlo automaticamente analizzando il comando SSH nel tuo script wrapper. Per una maggiore affidabilità si consiglia una configurazione esplicita.

<div id="remote-mac-via-tailscale-example">
  #### Mac remoto via Tailscale (esempio)
</div>

Se il Gateway è in esecuzione su un host/VM Linux ma iMessage deve essere eseguito su un Mac, Tailscale è il ponte più semplice: il Gateway comunica con il Mac attraverso la tailnet, esegue `imsg` via SSH e recupera gli allegati tramite SCP.

Architettura:

```
┌──────────────────────────────┐          SSH (imsg rpc)          ┌──────────────────────────┐
│ Host Gateway (Linux/VM)      │──────────────────────────────────▶│ Mac con Messages + imsg  │
│ - openclaw gateway           │          SCP (allegati)           │ - Messages autenticato   │
│ - channels.imessage.cliPath  │◀──────────────────────────────────│ - Remote Login abilitato │
└──────────────────────────────┘                                   └──────────────────────────┘
              ▲
              │ Tailscale tailnet (hostname o 100.x.y.z)
              ▼
        user@gateway-host
```

Esempio concreto di configurazione (hostname di Tailscale):

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db"
    }
  }
}
```

Esempio di wrapper (`~/.openclaw/scripts/imsg-ssh`):

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

Note:

* Assicurati che il Mac abbia effettuato l&#39;accesso a Messaggi e che Accesso remoto sia abilitato.
* Usa chiavi SSH in modo che `ssh bot@mac-mini.tailnet-1234.ts.net` funzioni senza richieste (ad esempio di password).
* `remoteHost` deve corrispondere alla destinazione SSH in modo che SCP possa recuperare gli allegati.

Supporto multi-account: usa `channels.imessage.accounts` con configurazione per account e `name` opzionale. Vedi [`gateway/configuration`](/it/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) per il pattern condiviso. Non effettuare il commit di `~/.openclaw/openclaw.json` (spesso contiene token).

<div id="access-control-dms-groups">
  ## Controllo accessi (DM + gruppi)
</div>

DM:

* Predefinito: `channels.imessage.dmPolicy = "pairing"`.
* I mittenti sconosciuti ricevono un codice di abbinamento; i messaggi vengono ignorati finché non vengono approvati (i codici scadono dopo 1 ora).
* Approva tramite:
  * `openclaw pairing list imessage`
  * `openclaw pairing approve imessage <CODE>`
* L&#39;abbinamento è lo scambio di token predefinito per i DM iMessage. Dettagli: [Pairing](/it/start/pairing)

Gruppi:

* `channels.imessage.groupPolicy = open | allowlist | disabled` (dove `open` indica che sono accettati messaggi senza restrizioni da qualsiasi utente).
* `channels.imessage.groupAllowFrom` controlla chi può attivare nei gruppi quando è impostato `allowlist`.
* Il gating basato sulle menzioni utilizza `agents.list[].groupChat.mentionPatterns` (o `messages.groupChat.mentionPatterns`) perché iMessage non fornisce metadati di menzione nativi.
* Override multi-agente: imposta pattern per singolo agente su `agents.list[].groupChat.mentionPatterns`.

<div id="how-it-works-behavior">
  ## Come funziona (comportamento)
</div>

* `imsg` effettua lo streaming degli eventi di messaggio; il Gateway li normalizza nell’envelope di canale condivisa.
* Le risposte vengono sempre instradate allo stesso ID chat o handle.

<div id="group-ish-threads-is_groupfalse">
  ## Thread di tipo gruppo (`is_group=false`)
</div>

Alcuni thread iMessage possono avere più partecipanti ma arrivare comunque con `is_group=false`, a seconda di come Messaggi memorizza l&#39;identificatore della chat.

Se configuri esplicitamente un `chat_id` sotto `channels.imessage.groups`, OpenClaw considera quel thread come un “gruppo” per:

* isolamento delle sessioni (chiave di sessione separata `agent:<agentId>:imessage:group:<chat_id>`)
* comportamento di lista di autorizzati / gating tramite menzione per il gruppo

Esempio:

```json5
{
  channels: {
    imessage: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "42": { "requireMention": false }
      }
    }
  }
}
```

Questo è utile quando desideri una personalità/modello isolati per un thread specifico (vedi [Multi-agent routing](/it/concepts/multi-agent)). Per l&#39;isolamento del filesystem, vedi [Sandboxing](/it/gateway/sandboxing).

<div id="media-limits">
  ## Media + limiti
</div>

* Acquisizione opzionale degli allegati tramite `channels.imessage.includeAttachments`.
* Limite per i contenuti multimediali tramite `channels.imessage.mediaMaxMb`.

<div id="limits">
  ## Limiti
</div>

* Il testo in uscita viene segmentato in base a `channels.imessage.textChunkLimit` (valore predefinito 4000).
* Suddivisione opzionale per righe vuote: imposta `channels.imessage.chunkMode="newline"` per dividere sulle righe vuote (confini di paragrafo) prima della suddivisione per lunghezza.
* I caricamenti di contenuti multimediali sono limitati da `channels.imessage.mediaMaxMb` (valore predefinito 16).

<div id="addressing-delivery-targets">
  ## Destinazioni di indirizzamento / consegna
</div>

Usa preferibilmente `chat_id` per un instradamento stabile:

* `chat_id:123` (preferito)
* `chat_guid:...`
* `chat_identifier:...`
* handle diretti: `imessage:+1555` / `sms:+1555` / `user@example.com`

Elenca le chat:

```
imsg chats --limit 20
```

<div id="configuration-reference-imessage">
  ## Riferimento di configurazione (iMessage)
</div>

Configurazione completa: [Configuration](/it/gateway/configuration)

Opzioni del provider:

* `channels.imessage.enabled`: abilita/disabilita l&#39;avvio del canale.
* `channels.imessage.cliPath`: percorso di `imsg`.
* `channels.imessage.dbPath`: percorso del DB di Messaggi.
* `channels.imessage.remoteHost`: host SSH per il trasferimento degli allegati via SCP quando `cliPath` punta a un Mac remoto (ad es. `user@gateway-host`). Rilevato automaticamente dal wrapper SSH se non impostato.
* `channels.imessage.service`: `imessage | sms | auto`.
* `channels.imessage.region`: regione SMS.
* `channels.imessage.dmPolicy`: `pairing | allowlist | open | disabled` (predefinito: pairing).
* `channels.imessage.allowFrom`: lista di autorizzati per DM (handle, email, numeri E.164 o `chat_id:*`). `open` richiede `"*"`. iMessage non ha username; utilizza handle o destinazioni chat.
* `channels.imessage.groupPolicy`: `open | allowlist | disabled` (predefinito: allowlist).
* `channels.imessage.groupAllowFrom`: lista di autorizzati per i mittenti nei gruppi.
* `channels.imessage.historyLimit` / `channels.imessage.accounts.*.historyLimit`: numero massimo di messaggi di gruppo da includere come contesto (0 disabilita).
* `channels.imessage.dmHistoryLimit`: limite di cronologia dei DM in turni dell’utente. Override per utente: `channels.imessage.dms["<handle>"].historyLimit`.
* `channels.imessage.groups`: valori predefiniti per gruppo + lista di autorizzati (usa `"*"` per i valori predefiniti globali).
* `channels.imessage.includeAttachments`: inserisce gli allegati nel contesto.
* `channels.imessage.mediaMaxMb`: limite per i media in ingresso/uscita (MB).
* `channels.imessage.textChunkLimit`: dimensione dei chunk in uscita (caratteri).
* `channels.imessage.chunkMode`: `length` (predefinito) o `newline` per suddividere alle righe vuote (confini di paragrafo) prima del chunking per lunghezza.

Opzioni globali correlate:

* `agents.list[].groupChat.mentionPatterns` (o `messages.groupChat.mentionPatterns`).
* `messages.responsePrefix`.