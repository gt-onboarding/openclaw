---
title: Signal
summary: "Supporto per Signal tramite signal-cli (JSON-RPC + SSE), configurazione e modello del numero di telefono"
read_when:
  - Configurazione del supporto per Signal
  - Debug dell'invio/ricezione su Signal
---

<div id="signal-signal-cli">
  # Signal (signal-cli)
</div>

Stato: integrazione con una CLI esterna. Il Gateway comunica con `signal-cli` tramite HTTP JSON-RPC + SSE.

<div id="quick-setup-beginner">
  ## Configurazione rapida (per principianti)
</div>

1. Usa un **numero Signal separato** per il bot (consigliato).
2. Installa `signal-cli` (è richiesto Java).
3. Collega il dispositivo del bot e avvia il demone:
   * `signal-cli link -n "OpenClaw"`
4. Configura OpenClaw e avvia il Gateway.

Configurazione minima:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"]
    }
  }
}
```

<div id="what-it-is">
  ## Che cos&#39;è
</div>

* Canale Signal tramite `signal-cli` (non usa la libsignal integrata).
* Instradamento deterministico: le risposte tornano sempre a Signal.
* I DM condividono la sessione principale dell&#39;agente; i gruppi sono isolati (`agent:<agentId>:signal:group:<groupId>`).

<div id="config-writes">
  ## Scritture della config
</div>

Per impostazione predefinita, Signal può scrivere aggiornamenti di configurazione attivati da `/config set|unset` (richiede `commands.config: true`).

Disabilita con:

```json5
{
  channels: { signal: { configWrites: false } }
}
```

<div id="the-number-model-important">
  ## Il modello dei numeri (importante)
</div>

* Il Gateway si collega a un **dispositivo Signal** (l&#39;account `signal-cli`).
* Se esegui il bot sul **tuo account Signal personale**, ignorerà i messaggi che invii tu (protezione contro i loop).
* Per &quot;scrivo un messaggio al bot e lui risponde&quot;, usa un **numero dedicato al bot**.

<div id="setup-fast-path">
  ## Configurazione (procedura rapida)
</div>

1. Installa `signal-cli` (richiede Java).
2. Collega un account bot:
   * `signal-cli link -n "OpenClaw"` poi scansiona il codice QR in Signal.
3. Configura Signal e avvia il Gateway.

Esempio:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"]
    }
  }
}
```

Supporto multi-account: usa `channels.signal.accounts` con configurazione per-account e `name` opzionale. Vedi [`gateway/configuration`](/it/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) per lo schema condiviso.

<div id="external-daemon-mode-httpurl">
  ## Modalità demone esterno (httpUrl)
</div>

Se vuoi gestire `signal-cli` autonomamente (avvii a freddo lenti della JVM, inizializzazione del container o CPU condivise), esegui il demone separatamente e configura OpenClaw in modo che punti ad esso:

```json5
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false
    }
  }
}
```

Questo evita l&#39;auto-spawn e l&#39;attesa di avvio all&#39;interno di OpenClaw. Per gestire avvii lenti durante l&#39;auto-spawn, imposta `channels.signal.startupTimeoutMs`.

<div id="access-control-dms-groups">
  ## Controllo accessi (DM + gruppi)
</div>

DM:

* Predefinito: `channels.signal.dmPolicy = "pairing"`.
* I mittenti sconosciuti ricevono un codice di abbinamento; i messaggi vengono ignorati finché non vengono approvati (i codici scadono dopo 1 ora).
* Approva tramite:
  * `openclaw pairing list signal`
  * `openclaw pairing approve signal <CODE>`
* L&#39;abbinamento è lo scambio di token predefinito per i DM Signal. Dettagli: [Pairing](/it/start/pairing)
* I mittenti con solo UUID (da `sourceUuid`) vengono memorizzati come `uuid:<id>` in `channels.signal.allowFrom`.

Gruppi:

* `channels.signal.groupPolicy = open | allowlist | disabled` (open consente l&#39;accettazione di messaggi da chiunque).
* `channels.signal.groupAllowFrom` controlla chi può attivare nei gruppi quando è impostato `allowlist`.

<div id="how-it-works-behavior">
  ## Come funziona (comportamento)
</div>

* `signal-cli` viene eseguito come demone; il Gateway legge gli eventi tramite SSE.
* I messaggi in ingresso vengono normalizzati nella busta di canale condivisa.
* Le risposte vengono sempre instradate allo stesso numero o gruppo.

<div id="media-limits">
  ## Contenuti multimediali + limiti
</div>

* Il testo in uscita viene suddiviso in chunk secondo `channels.signal.textChunkLimit` (predefinito 4000).
* Suddivisione opzionale per newline: imposta `channels.signal.chunkMode="newline"` per dividere alle righe vuote (confini di paragrafo) prima del chunking per lunghezza.
* Supporto per allegati (base64 recuperato da `signal-cli`).
* Limite multimediale predefinito: `channels.signal.mediaMaxMb` (predefinito 8).
* Usa `channels.signal.ignoreAttachments` per evitare il download dei contenuti multimediali.
* Il contesto della cronologia dei gruppi usa `channels.signal.historyLimit` (o `channels.signal.accounts.*.historyLimit`), altrimenti `messages.groupChat.historyLimit`. Imposta `0` per disabilitare (predefinito 50).

<div id="typing-read-receipts">
  ## Indicatori di digitazione + conferme di lettura
</div>

* **Indicatori di digitazione**: OpenClaw invia segnali di digitazione tramite `signal-cli sendTyping` e li aggiorna durante la generazione di una risposta.
* **Conferme di lettura**: quando `channels.signal.sendReadReceipts` è true, OpenClaw inoltra le conferme di lettura per i messaggi diretti (DM) consentiti.
* Signal-cli non fornisce conferme di lettura per i gruppi.

<div id="reactions-message-tool">
  ## Reazioni (strumento messaggi)
</div>

* Usa `message action=react` con `channel=signal`.
* Destinazioni: numero E.164 del mittente o UUID (usa `uuid:<id>` dall&#39;output di abbinamento; funziona anche l&#39;UUID senza prefisso).
* `messageId` è il timestamp di Signal del messaggio a cui stai reagendo.
* Le reazioni di gruppo richiedono `targetAuthor` o `targetAuthorUuid`.

Esempi:

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

Config:

* `channels.signal.actions.reactions`: abilita/disabilita le azioni di reazione (valore predefinito true).
* `channels.signal.reactionLevel`: `off | ack | minimal | extensive`.
  * `off`/`ack` disabilita le reazioni dell&#39;agente (lo strumento di messaggistica `react` restituirà un errore).
  * `minimal`/`extensive` abilita le reazioni dell&#39;agente e imposta il livello di guida.
* Override per account: `channels.signal.accounts.<id>.actions.reactions`, `channels.signal.accounts.<id>.reactionLevel`.

<div id="delivery-targets-clicron">
  ## Destinazioni di recapito (CLI/cron)
</div>

* DM: `signal:+15551234567` (oppure semplice E.164).
* DM UUID: `uuid:<id>` (oppure UUID senza prefisso).
* Gruppi: `signal:group:<groupId>`.
* Username: `username:<name>` (se supportato dal tuo account Signal).

<div id="configuration-reference-signal">
  ## Riferimento configurazione (Signal)
</div>

Configurazione completa: [Configuration](/it/gateway/configuration)

Opzioni del provider:

* `channels.signal.enabled`: abilita/disabilita l&#39;avvio del canale.
* `channels.signal.account`: E.164 per l&#39;account del bot.
* `channels.signal.cliPath`: percorso di `signal-cli`.
* `channels.signal.httpUrl`: URL completo del demone (sovrascrive host/port).
* `channels.signal.httpHost`, `channels.signal.httpPort`: binding del demone (predefinito 127.0.0.1:8080).
* `channels.signal.autoStart`: avvia automaticamente il demone (predefinito true se `httpUrl` non è impostato).
* `channels.signal.startupTimeoutMs`: timeout di attesa all&#39;avvio in ms (massimo 120000).
* `channels.signal.receiveMode`: `on-start | manual`.
* `channels.signal.ignoreAttachments`: ignora il download degli allegati.
* `channels.signal.ignoreStories`: ignora le storie dal demone.
* `channels.signal.sendReadReceipts`: inoltra le conferme di lettura.
* `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled` (predefinito: pairing).
* `channels.signal.allowFrom`: lista di autorizzati per i DM (E.164 o `uuid:<id>`). `open` richiede `"*"`. Signal non supporta nomi utente; usa ID telefono/UUID.
* `channels.signal.groupPolicy`: `open | allowlist | disabled` (predefinito: allowlist).
* `channels.signal.groupAllowFrom`: lista di autorizzati per i mittenti nei gruppi.
* `channels.signal.historyLimit`: numero massimo di messaggi di gruppo da includere come contesto (0 disabilita).
* `channels.signal.dmHistoryLimit`: limite di cronologia DM in turni utente. Override per utente: `channels.signal.dms["<phone_or_uuid>"].historyLimit`.
* `channels.signal.textChunkLimit`: dimensione dei chunk in uscita (caratteri).
* `channels.signal.chunkMode`: `length` (predefinito) oppure `newline` per dividere alle righe vuote (confini di paragrafo) prima del chunking per lunghezza.
* `channels.signal.mediaMaxMb`: limite massimo per i contenuti multimediali in ingresso/uscita (MB).

Opzioni globali correlate:

* `agents.list[].groupChat.mentionPatterns` (Signal non supporta le menzioni native).
* `messages.groupChat.mentionPatterns` (fallback globale).
* `messages.responsePrefix`.