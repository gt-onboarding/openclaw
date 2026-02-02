---
title: Bluebubbles
summary: "iMessage tramite server BlueBubbles su macOS (invio/ricezione via REST, indicazione di digitazione, reazioni, abbinamento, azioni avanzate)."
read_when:
  - Configurazione del canale BlueBubbles
  - Risoluzione dei problemi di abbinamento del webhook
  - Configurazione di iMessage su macOS
---

<div id="bluebubbles-macos-rest">
  # BlueBubbles (macOS REST)
</div>

Stato: plugin integrato che comunica con il server BlueBubbles su macOS tramite HTTP. **Consigliato per l&#39;integrazione con iMessage** grazie alla sua API più completa e alla configurazione più semplice rispetto al canale imsg legacy.

<div id="overview">
  ## Panoramica
</div>

* Viene eseguito su macOS tramite l&#39;app di supporto BlueBubbles ([bluebubbles.app](https://bluebubbles.app)).
* Consigliato/testato: macOS Sequoia (15). macOS Tahoe (26) funziona; la modifica attualmente non funziona su Tahoe e gli aggiornamenti delle icone dei gruppi possono segnalare esito positivo ma non sincronizzarsi.
* OpenClaw comunica con esso tramite la sua REST API (`GET /api/v1/ping`, `POST /message/text`, `POST /chat/:id/*`).
* I messaggi in arrivo vengono ricevuti tramite webhook; le risposte in uscita, gli indicatori di digitazione, le conferme di lettura e i tapback sono chiamate REST.
* Allegati e sticker vengono acquisiti come media in ingresso (e resi disponibili all&#39;agente quando possibile).
* L&#39;abbinamento e la lista di autorizzati funzionano allo stesso modo degli altri canali (`/start/pairing` ecc.) con `channels.bluebubbles.allowFrom` + codici di abbinamento.
* Le reazioni vengono esposte come eventi di sistema proprio come su Slack/Telegram, così gli agenti possono &quot;menzionarle&quot; prima di rispondere.
* Funzionalità avanzate: modifica, annullamento dell&#39;invio, thread delle risposte, effetti dei messaggi, gestione dei gruppi.

<div id="quick-start">
  ## Avvio rapido
</div>

1. Installa il server BlueBubbles sul tuo Mac (segui le istruzioni su [bluebubbles.app/install](https://bluebubbles.app/install)).
2. Nella configurazione di BlueBubbles, abilita l&#39;API web e imposta una password.
3. Esegui `openclaw onboard` e seleziona BlueBubbles, oppure configura manualmente:
   ```json5
   {
     channels: {
       bluebubbles: {
         enabled: true,
         serverUrl: "http://192.168.1.100:1234",
         password: "example-password",
         webhookPath: "/bluebubbles-webhook"
       }
     }
   }
   ```
4. Configura i webhook di BlueBubbles perché puntino al tuo Gateway (esempio: `https://your-gateway-host:3000/bluebubbles-webhook?password=<password>`).
5. Avvia il Gateway; registrerà l&#39;handler del webhook e avvierà l&#39;abbinamento.

<div id="onboarding">
  ## Onboarding
</div>

BlueBubbles è disponibile tramite la procedura guidata di configurazione interattiva:

```
openclaw onboard
```

La procedura guidata richiede:

* **Server URL** (obbligatorio): indirizzo del server BlueBubbles (ad es. `http://192.168.1.100:1234`)
* **Password** (obbligatoria): password API dalle impostazioni di BlueBubbles Server
* **Webhook path** (opzionale): valore predefinito: `/bluebubbles-webhook`
* **DM policy**: pairing, allowlist, open oppure disabled
* **Allow list**: numeri di telefono, e-mail o destinatari chat autorizzati

Puoi anche aggiungere BlueBubbles tramite CLI:

```
openclaw channels add bluebubbles --http-url http://192.168.1.100:1234 --password <password>
```

<div id="access-control-dms-groups">
  ## Controllo degli accessi (DM + gruppi)
</div>

DM:

* Predefinito: `channels.bluebubbles.dmPolicy = "pairing"`.
* I mittenti sconosciuti ricevono un codice di abbinamento; i messaggi vengono ignorati finché non sono approvati (i codici scadono dopo 1 ora).
* Approva usando:
  * `openclaw pairing list bluebubbles`
  * `openclaw pairing approve bluebubbles <CODE>`
* L&#39;abbinamento è lo scambio di token predefinito. Dettagli: [Pairing](/it/start/pairing)

Gruppi:

* `channels.bluebubbles.groupPolicy = open | allowlist | disabled` (predefinito: `allowlist`).
* `channels.bluebubbles.groupAllowFrom` controlla chi può attivare l&#39;agente nei gruppi quando `allowlist` è attivato.

<div id="mention-gating-groups">
  ### Mention gating (gruppi)
</div>

BlueBubbles supporta il mention gating per le chat di gruppo, allineandosi al comportamento di iMessage/WhatsApp:

* Usa `agents.list[].groupChat.mentionPatterns` (o `messages.groupChat.mentionPatterns`) per rilevare le menzioni.
* Quando `requireMention` è abilitato per un gruppo, l&#39;agente risponde solo quando viene menzionato.
* I comandi di controllo provenienti da mittenti autorizzati bypassano il mention gating.

Configurazione per gruppo:

```json5
{
  channels: {
    bluebubbles: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true },  // default for all groups
        "iMessage;-;chat123": { requireMention: false }  // override per un gruppo specifico
      }
    }
  }
}
```

<div id="command-gating">
  ### Controllo dell&#39;accesso ai comandi
</div>

* I comandi di controllo (ad esempio `/config`, `/model`) richiedono un&#39;autorizzazione.
* Utilizza `allowFrom` e `groupAllowFrom` per determinare l&#39;autorizzazione ai comandi.
* I mittenti autorizzati possono eseguire comandi di controllo anche senza essere menzionati nei gruppi.

<div id="typing-read-receipts">
  ## Indicatori di digitazione + conferme di lettura
</div>

* **Indicatori di digitazione**: Inviati automaticamente prima e durante la generazione della risposta.
* **Conferme di lettura**: Gestite da `channels.bluebubbles.sendReadReceipts` (valore predefinito: `true`).
* **Indicatori di digitazione**: OpenClaw invia eventi di inizio digitazione; BlueBubbles cancella automaticamente l&#39;indicatore di digitazione all&#39;invio o al timeout (l&#39;arresto manuale tramite DELETE non è affidabile).

```json5
{
  channels: {
    bluebubbles: {
      sendReadReceipts: false  // disabilita le conferme di lettura
    }
  }
}
```

<div id="advanced-actions">
  ## Azioni avanzate
</div>

BlueBubbles supporta azioni avanzate sui messaggi quando sono abilitate nella configurazione:

```json5
{
  channels: {
    bluebubbles: {
      actions: {
        reactions: true,       // tapbacks (default: true)
        edit: true,            // modifica messaggi inviati (macOS 13+, non funzionante su macOS 26 Tahoe)
        unsend: true,          // unsend messages (macOS 13+)
        reply: true,           // reply threading by message GUID
        sendWithEffect: true,  // message effects (slam, loud, etc.)
        renameGroup: true,     // rename group chats
        setGroupIcon: true,    // set group chat icon/photo (flaky on macOS 26 Tahoe)
        addParticipant: true,  // add participants to groups
        removeParticipant: true, // remove participants from groups
        leaveGroup: true,      // leave group chats
        sendAttachment: true   // send attachments/media
      }
    }
  }
}
```

Azioni disponibili:

* **react**: Aggiungi/rimuovi reazioni tapback (`messageId`, `emoji`, `remove`)
* **edit**: Modifica un messaggio inviato (`messageId`, `text`)
* **unsend**: Annulla l&#39;invio di un messaggio (`messageId`)
* **reply**: Rispondi a un messaggio specifico (`messageId`, `text`, `to`)
* **sendWithEffect**: Invia con effetto iMessage (`text`, `to`, `effectId`)
* **renameGroup**: Rinomina una chat di gruppo (`chatGuid`, `displayName`)
* **setGroupIcon**: Imposta l&#39;icona/foto di una chat di gruppo (`chatGuid`, `media`) — instabile su macOS 26 Tahoe (l&#39;API può restituire esito positivo ma l&#39;icona non viene sincronizzata).
* **addParticipant**: Aggiungi qualcuno a un gruppo (`chatGuid`, `address`)
* **removeParticipant**: Rimuovi qualcuno da un gruppo (`chatGuid`, `address`)
* **leaveGroup**: Esci da una chat di gruppo (`chatGuid`)
* **sendAttachment**: Invia contenuti multimediali o file (`to`, `buffer`, `filename`, `asVoice`)
  * Memo vocali: imposta `asVoice: true` con audio **MP3** o **CAF** per inviarlo come messaggio vocale iMessage. BlueBubbles converte MP3 → CAF durante l&#39;invio dei memo vocali.

<div id="message-ids-short-vs-full">
  ### ID dei messaggi (brevi vs completi)
</div>

OpenClaw può esporre ID dei messaggi *brevi* (ad es. `1`, `2`) per risparmiare token.

* `MessageSid` / `ReplyToId` possono essere ID brevi.
* `MessageSidFull` / `ReplyToIdFull` contengono gli ID completi del provider.
* Gli ID brevi esistono solo in memoria; possono scadere al riavvio o per svuotamento della cache.
* Le azioni accettano `messageId` brevi o completi, ma gli ID brevi genereranno un errore se non sono più disponibili.

Utilizza gli ID completi per automazioni e archiviazione durevoli:

* Template: `{{MessageSidFull}}`, `{{ReplyToIdFull}}`
* Contesto: `MessageSidFull` / `ReplyToIdFull` nei payload in ingresso

Vedi [Configurazione](/it/gateway/configuration) per le variabili dei template.

<div id="block-streaming">
  ## Streaming a blocchi
</div>

Controlla se le risposte vengono inviate come un unico messaggio o trasmesse in streaming a blocchi:

```json5
{
  channels: {
    bluebubbles: {
      blockStreaming: true  // abilita lo streaming a blocchi (comportamento predefinito)
    }
  }
}
```

<div id="media-limits">
  ## Media + limiti
</div>

* Gli allegati in ingresso vengono scaricati e memorizzati nella cache multimediale.
* Limite dei contenuti multimediali impostato tramite `channels.bluebubbles.mediaMaxMb` (predefinito: 8 MB).
* Il testo in uscita viene suddiviso in blocchi secondo `channels.bluebubbles.textChunkLimit` (predefinito: 4000 caratteri).

<div id="configuration-reference">
  ## Riferimento configurazione
</div>

Configurazione completa: [Configuration](/it/gateway/configuration)

Opzioni del provider:

* `channels.bluebubbles.enabled`: Abilita/disabilita il canale.
* `channels.bluebubbles.serverUrl`: URL di base della BlueBubbles REST API.
* `channels.bluebubbles.password`: Password API.
* `channels.bluebubbles.webhookPath`: Percorso dell&#39;endpoint webhook (predefinito: `/bluebubbles-webhook`).
* `channels.bluebubbles.dmPolicy`: `pairing | allowlist | open | disabled` (predefinito: `pairing`).
* `channels.bluebubbles.allowFrom`: Lista di autorizzati per i DM (handle, indirizzi email, numeri E.164, `chat_id:*`, `chat_guid:*`).
* `channels.bluebubbles.groupPolicy`: `open | allowlist | disabled` (predefinito: `allowlist`).
* `channels.bluebubbles.groupAllowFrom`: Lista di autorizzati dei mittenti nei gruppi.
* `channels.bluebubbles.groups`: Configurazione per singolo gruppo (`requireMention`, ecc.).
* `channels.bluebubbles.sendReadReceipts`: Invia conferme di lettura (predefinito: `true`).
* `channels.bluebubbles.blockStreaming`: Abilita lo streaming a blocchi (predefinito: `true`).
* `channels.bluebubbles.textChunkLimit`: Dimensione dei blocchi in uscita in caratteri (predefinito: 4000).
* `channels.bluebubbles.chunkMode`: `length` (predefinito) suddivide solo quando si supera `textChunkLimit`; `newline` suddivide sulle righe vuote (limiti di paragrafo) prima del chunking per lunghezza.
* `channels.bluebubbles.mediaMaxMb`: Limite massimo dei media in ingresso in MB (predefinito: 8).
* `channels.bluebubbles.historyLimit`: Numero massimo di messaggi di gruppo per il contesto (0 disabilita).
* `channels.bluebubbles.dmHistoryLimit`: Limite della cronologia DM.
* `channels.bluebubbles.actions`: Abilita/disabilita azioni specifiche.
* `channels.bluebubbles.accounts`: Configurazione multi-account.

Opzioni globali correlate:

* `agents.list[].groupChat.mentionPatterns` (oppure `messages.groupChat.mentionPatterns`).
* `messages.responsePrefix`.

<div id="addressing-delivery-targets">
  ## Indirizzamento / destinazioni di recapito
</div>

Usa `chat_guid` per un instradamento stabile:

* `chat_guid:iMessage;-;+15555550123` (preferito per i gruppi)
* `chat_id:123`
* `chat_identifier:...`
* Handle diretti: `+15555550123`, `user@example.com`
  * Se un handle diretto non ha già una chat DM esistente, OpenClaw ne creerà una tramite `POST /api/v1/chat/new`. Questo richiede che la BlueBubbles Private API sia attivata.

<div id="security">
  ## Sicurezza
</div>

* Le richieste webhook sono autenticate confrontando i parametri di query o gli header `guid`/`password` con `channels.bluebubbles.password`. Le richieste provenienti da `localhost` sono accettate ugualmente.
* Mantieni segreti la password API e l&#39;endpoint webhook (trattali come credenziali).
* La fiducia accordata a localhost implica che un reverse proxy sullo stesso host può aggirare inavvertitamente la password. Se metti il Gateway dietro un proxy, richiedi l&#39;autenticazione sul proxy e configura `gateway.trustedProxies`. Vedi [Gateway security](/it/gateway/security#reverse-proxy-configuration).
* Abilita HTTPS e regole firewall sul server BlueBubbles se lo esponi all’esterno della tua LAN.

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

* Se gli eventi di digitazione/read smettono di funzionare, controlla i log del webhook di BlueBubbles e verifica che il percorso del Gateway corrisponda a `channels.bluebubbles.webhookPath`.
* I codici di abbinamento scadono dopo un&#39;ora; usa `openclaw pairing list bluebubbles` e `openclaw pairing approve bluebubbles <code>`.
* Le reazioni richiedono la private API di BlueBubbles (`POST /api/v1/message/react`); assicurati che la versione del server la esponga.
* Le azioni di edit/unsend richiedono macOS 13+ e una versione del server BlueBubbles compatibile. Su macOS 26 (Tahoe), edit è attualmente non funzionante a causa di modifiche alla private API.
* Gli aggiornamenti delle icone dei gruppi possono essere instabili su macOS 26 (Tahoe): l&#39;API può restituire successo ma la nuova icona non viene sincronizzata.
* OpenClaw nasconde automaticamente le azioni note come non funzionanti in base alla versione di macOS del server BlueBubbles. Se edit è ancora visibile su macOS 26 (Tahoe), disabilitalo manualmente con `channels.bluebubbles.actions.edit=false`.
* Per informazioni su stato/integrità: `openclaw status --all` oppure `openclaw status --deep`.

Per una panoramica generale dei workflow dei canali, consulta [Channels](/it/channels) e la guida [Plugins](/it/plugins).