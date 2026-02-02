---
title: Whatsapp
summary: "Integrazione WhatsApp (canale web): login, inbox, risposte, media e operazioni"
read_when:
  - Quando lavori sul comportamento del canale WhatsApp/web o sull'instradamento dell'inbox
---

<div id="whatsapp-web-channel">
  # WhatsApp (canale web)
</div>

Stato: WhatsApp Web solo tramite Baileys. Il Gateway gestisce le sessioni.

<div id="quick-setup-beginner">
  ## Configurazione rapida (per principianti)
</div>

1. Usa **un numero di telefono separato**, se possibile (consigliato).
2. Configura WhatsApp in `~/.openclaw/openclaw.json`.
3. Esegui `openclaw channels login` per scansionare il codice QR (Dispositivi collegati).
4. Avvia il Gateway.

Configurazione minima:

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"]
    }
  }
}
```

<div id="goals">
  ## Obiettivi
</div>

* Più account WhatsApp (multiaccount) in un unico processo del Gateway.
* Instradamento deterministico: le risposte tornano a WhatsApp, senza instradamento da parte del modello.
* Il modello dispone di contesto sufficiente per comprendere le risposte quotate.

<div id="config-writes">
  ## Scrittura della configurazione
</div>

Per impostazione predefinita, WhatsApp è autorizzato a scrivere aggiornamenti della configurazione attivati da `/config set|unset` (richiede `commands.config: true`).

Per disabilitarlo:

```json5
{
  channels: { whatsapp: { configWrites: false } }
}
```

<div id="architecture-who-owns-what">
  ## Architettura (chi gestisce cosa)
</div>

* Il **Gateway** gestisce il socket Baileys e il loop della inbox.
* La **CLI / app macOS** comunica con il Gateway; nessun utilizzo diretto di Baileys.
* Un **listener attivo** è necessario per gli invii in uscita; in caso contrario `send` fallisce immediatamente.

<div id="getting-a-phone-number-two-modes">
  ## Ottenere un numero di telefono (due modalità)
</div>

WhatsApp richiede un vero numero di cellulare per la verifica. I numeri VoIP e virtuali sono in genere bloccati. Esistono due modalità supportate per eseguire OpenClaw con WhatsApp:

<div id="dedicated-number-recommended">
  ### Numero dedicato (consigliato)
</div>

Usa un **numero di telefono separato** per OpenClaw. Migliore esperienza utente, instradamento pulito, nessun comportamento strano con le auto‑chat. Configurazione ideale: **telefono Android di scorta/vecchio + eSIM**. Lascialo collegato al Wi‑Fi e all’alimentazione e collegalo tramite codice QR.

**WhatsApp Business:** puoi usare WhatsApp Business sullo stesso dispositivo con un numero diverso. Ottimo per mantenere separato il tuo WhatsApp personale — installa WhatsApp Business e registra lì il numero di OpenClaw.

**Configurazione di esempio (numero dedicato, lista di autorizzati per un singolo utente):**

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"]
    }
  }
}
```

**Modalità di abbinamento (facoltativa):**
Se preferisci usare l&#39;abbinamento invece della lista di autorizzati, imposta `channels.whatsapp.dmPolicy` su `pairing`. I mittenti sconosciuti ricevono un codice di abbinamento; approvalo con:
`openclaw pairing approve whatsapp <code>`

<div id="personal-number-fallback">
  ### Numero personale (opzione di riserva)
</div>

Soluzione rapida di riserva: esegui OpenClaw sul **tuo numero personale**. Scrivi a te stesso (WhatsApp “Invia un messaggio a te stesso”) per i test, così non fai spam ai contatti. Tieni presente che dovrai leggere i codici di verifica sul tuo telefono principale durante la configurazione e gli esperimenti. **Devi abilitare la modalità chat con te stesso.**
Quando la procedura guidata richiede il tuo numero WhatsApp personale, inserisci il telefono da cui invierai i messaggi (il proprietario/mittente), non il numero dell&#39;assistente.

**Configurazione di esempio (numero personale, chat con te stesso):**

```json
{
  "whatsapp": {
    "selfChatMode": true,
    "dmPolicy": "allowlist",
    "allowFrom": ["+15551234567"]
  }
}
```

Le risposte alle self-chat usano per impostazione predefinita `[{identity.name}]` quando è impostato (altrimenti `[openclaw]`)
se `messages.responsePrefix` non è definito. Impostalo esplicitamente per personalizzare o disabilitare
il prefisso (usa `""` per rimuoverlo).

<div id="number-sourcing-tips">
  ### Suggerimenti per ottenere un numero
</div>

* **eSIM locale** da un operatore mobile del tuo paese (la soluzione più affidabile)
  * Austria: [hot.at](https://www.hot.at)
  * UK: [giffgaff](https://www.giffgaff.com) — SIM gratuita, nessun contratto
* **SIM ricaricabile** — economica, deve solo ricevere un SMS per la verifica

**Evita:** TextNow, Google Voice, la maggior parte dei servizi di &quot;SMS gratuiti&quot; — WhatsApp li blocca in modo aggressivo.

**Suggerimento:** Il numero deve solo ricevere un SMS di verifica. Dopodiché, le sessioni WhatsApp Web restano attive tramite `creds.json`.

<div id="why-not-twilio">
  ## Perché non Twilio?
</div>

* Le prime versioni di OpenClaw supportavano l’integrazione WhatsApp Business di Twilio.
* I numeri WhatsApp Business non sono adatti a un assistente personale.
* Meta applica una finestra di risposta di 24 ore; se non hai risposto nelle ultime 24 ore, il numero Business non può avviare nuove conversazioni.
* Un uso ad alto volume o particolarmente “chiacchierone” fa scattare blocchi aggressivi, perché gli account Business non sono pensati per inviare decine di messaggi di assistente personale.
* Risultato: recapito inaffidabile e blocchi frequenti, quindi il supporto è stato rimosso.

<div id="login-credentials">
  ## Login + credenziali
</div>

* Comando di login: `openclaw channels login` (QR tramite Dispositivi collegati).
* Login multi-account: `openclaw channels login --account <id>` (`<id>` = `accountId`).
* Account predefinito (quando `--account` è omesso): `default` se presente, altrimenti il primo ID account configurato (in ordine crescente).
* Credenziali memorizzate in `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`.
* Copia di backup in `creds.json.bak` (ripristinata in caso di danneggiamento).
* Compatibilità con versioni precedenti: le installazioni meno recenti salvavano i file Baileys direttamente in `~/.openclaw/credentials/`.
* Logout: `openclaw channels logout` (oppure `--account <id>`) elimina lo stato di autenticazione di WhatsApp (ma mantiene il `oauth.json` condiviso).
* Socket disconnesso =&gt; viene restituito un errore che richiede di ripetere il collegamento.

<div id="inbound-flow-dm-group">
  ## Flusso in ingresso (DM + gruppo)
</div>

* Gli eventi WhatsApp provengono da `messages.upsert` (Baileys).
* I listener dell&#39;inbox vengono scollegati all&#39;arresto per evitare l&#39;accumulo di gestori di eventi in test/riavvii.
* Le chat di stato/broadcast vengono ignorate.
* Le chat dirette usano E.164; i gruppi usano il JID del gruppo.
* **DM policy**: `channels.whatsapp.dmPolicy` controlla l&#39;accesso alle chat dirette (predefinito: `pairing`).
  * Abbinamento: i mittenti sconosciuti ricevono un codice di abbinamento (approva tramite `openclaw pairing approve whatsapp <code>`; i codici scadono dopo 1 ora).
  * open: richiede che `channels.whatsapp.allowFrom` includa `"*"` e indica che l&#39;accettazione dei messaggi è senza restrizioni da parte di qualsiasi utente.
  * Il tuo numero WhatsApp collegato è implicitamente considerato attendibile, quindi i messaggi a te stesso saltano i controlli `channels.whatsapp.dmPolicy` e `channels.whatsapp.allowFrom`.

<div id="personal-number-mode-fallback">
  ### Modalità numero personale (fallback)
</div>

Se esegui OpenClaw sul **tuo numero WhatsApp personale**, abilita `channels.whatsapp.selfChatMode` (vedi esempio sopra).

Comportamento:

* I DM in uscita non generano mai risposte di abbinamento (per evitare di inviare spam ai contatti).
* I messaggi in ingresso da mittenti sconosciuti continuano a seguire `channels.whatsapp.dmPolicy`.
* La modalità chat personale (quando allowFrom include il tuo numero) evita le conferme di lettura automatiche e ignora gli JID delle menzioni.
* Le conferme di lettura vengono inviate per i DM che non sono chat personali.

<div id="read-receipts">
  ## Conferme di lettura
</div>

Per impostazione predefinita, il Gateway segna come letti (spunte blu) i messaggi WhatsApp in arrivo non appena vengono accettati.

Disabilita a livello globale:

```json5
{
  channels: { whatsapp: { sendReadReceipts: false } }
}
```

Disattiva per account:

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        personal: { sendReadReceipts: false }
      }
    }
  }
}
```

Note:

* La modalità self-chat non invia mai le conferme di lettura.

<div id="whatsapp-faq-sending-messages-pairing">
  ## FAQ WhatsApp: invio di messaggi + abbinamento
</div>

**OpenClaw invierà messaggi a contatti a caso quando collego WhatsApp?**\
No. La policy DM predefinita è **abbinamento**, quindi i mittenti sconosciuti ricevono solo un codice di abbinamento e il loro messaggio **non viene elaborato**. OpenClaw risponde solo alle chat che riceve o agli invii che attivi esplicitamente (agente/CLI).

**Come funziona l’abbinamento su WhatsApp?**\
L’abbinamento è un filtro DM per i mittenti sconosciuti:

* Il primo DM da un nuovo mittente restituisce un codice breve (il messaggio non viene elaborato).
* Approva con: `openclaw pairing approve whatsapp <code>` (elenca con `openclaw pairing list whatsapp`).
* I codici scadono dopo 1 ora; le richieste in sospeso sono limitate a 3 per canale.

**Più persone possono usare istanze diverse di OpenClaw su un unico numero WhatsApp?**\
Sì, instradando ogni mittente a un agente diverso tramite `bindings` (peer `kind: "dm"`, mittente E.164 tipo `+15551234567`). Le risposte arrivano comunque dallo **stesso account WhatsApp** e le chat dirette vengono ricondotte alla sessione principale di ciascun agente, quindi usa **un agente per persona**. Il controllo di accesso DM (`dmPolicy`/`allowFrom`) è globale per account WhatsApp. Vedi [Instradamento multi-agente](/it/concepts/multi-agent).

**Perché nel wizard chiedete il mio numero di telefono?**\
Il wizard lo usa per impostare la tua **lista di autorizzati e il proprietario (owner)** così che i tuoi DM siano consentiti. Non viene usato per l’invio automatico. Se esegui sul tuo numero WhatsApp personale, usa quello stesso numero e abilita `channels.whatsapp.selfChatMode`.

<div id="message-normalization-what-the-model-sees">
  ## Normalizzazione dei messaggi (ciò che vede il modello)
</div>

* `Body` è il corpo del messaggio corrente, incluso l&#39;envelope.
* Il contesto della risposta citata viene **sempre aggiunto in coda**:
  ```
  [Replying to +1555 id:ABC123]
  <quoted text or <media:...>>
  [/Replying]
  ```
* Vengono inoltre impostati i metadati della risposta:
  * `ReplyToId` = stanzaId
  * `ReplyToBody` = corpo citato o segnaposto per i media
  * `ReplyToSender` = E.164 quando noto
* I messaggi in ingresso composti solo da media usano dei segnaposto:
  * `<media:image|video|audio|document|sticker>`

<div id="groups">
  ## Gruppi
</div>

* I gruppi corrispondono a sessioni `agent:<agentId>:whatsapp:group:<jid>`.
* Criterio di gruppo: `channels.whatsapp.groupPolicy = open|disabled|allowlist` (predefinito `allowlist`).
* Modalità di attivazione:
  * `mention` (predefinito): richiede una @menzione o una corrispondenza regex.
  * `always`: si attiva sempre.
* `/activation mention|always` è consentito solo al proprietario e deve essere inviato come messaggio a sé stante.
* Proprietario = `channels.whatsapp.allowFrom` (o il proprio E.164 se non impostato).
* **Iniezione della cronologia** (solo elementi in sospeso):
  * I messaggi *non elaborati* recenti (predefinito 50) vengono inseriti sotto:
    `[Chat messages since your last reply - for context]` (i messaggi già presenti nella sessione non vengono reinseriti)
  * Il messaggio corrente viene inserito sotto:
    `[Current message - respond to this]`
  * Viene aggiunto il suffisso del mittente: `[from: Name (+E164)]`
* Metadati del gruppo memorizzati in cache per 5 minuti (oggetto + partecipanti).

<div id="reply-delivery-threading">
  ## Consegna delle risposte (threading)
</div>

* WhatsApp Web invia messaggi standard (nessun threading delle risposte citate nell’attuale Gateway).
* I tag di risposta vengono ignorati su questo canale.

<div id="acknowledgment-reactions-auto-react-on-receipt">
  ## Reazioni di conferma (reazioni automatiche alla ricezione)
</div>

WhatsApp può inviare automaticamente una reazione emoji ai messaggi in arrivo non appena vengono ricevuti, prima che il bot generi una risposta. In questo modo gli utenti ricevono subito un riscontro che conferma l’avvenuta ricezione del loro messaggio.

**Configurazione:**

```json
{
  "whatsapp": {
    "ackReaction": {
      "emoji": "👀",
      "direct": true,
      "group": "mentions"
    }
  }
}
```

**Opzioni:**

* `emoji` (string): Emoji da usare per la conferma (ad es. &quot;👀&quot;, &quot;✅&quot;, &quot;📨&quot;). Vuoto o omesso = funzionalità disattivata.
* `direct` (boolean, default: `true`): Invia reazioni nelle chat dirette/DM.
* `group` (string, default: `"mentions"`): Comportamento nelle chat di gruppo:
  * `"always"`: Reagisce a tutti i messaggi di gruppo (anche senza @mention)
  * `"mentions"`: Reagisce solo quando il bot è @menzionato
  * `"never"`: Non reagisce mai nei gruppi

**Override per account:**

```json
{
  "whatsapp": {
    "accounts": {
      "work": {
        "ackReaction": {
          "emoji": "✅",
          "direct": false,
          "group": "always"
        }
      }
    }
  }
}
```

**Note sul comportamento:**

* Le reazioni vengono inviate **immediatamente** alla ricezione del messaggio, prima degli indicatori di digitazione o delle risposte del bot.
* Nei gruppi con `requireMention: false` (attivazione: always), `group: "mentions"` reagirà a tutti i messaggi (non solo alle @menzioni).
* Fire-and-forget: gli errori nelle reazioni vengono registrati nei log ma non impediscono al bot di rispondere.
* Il JID del partecipante è incluso automaticamente per le reazioni nei gruppi.
* WhatsApp ignora `messages.ackReaction`; usa invece `channels.whatsapp.ackReaction`.

<div id="agent-tool-reactions">
  ## Strumento agente (reazioni)
</div>

* Strumento: `whatsapp` con azione `react` (`chatJid`, `messageId`, `emoji`, opzionale `remove`).
* Parametri opzionali: `participant` (mittente nel gruppo), `fromMe` (reazione a un tuo messaggio), `accountId` (multi-account).
* Modalità di rimozione delle reazioni: vedi [/tools/reactions](/it/tools/reactions).
* Controllo di abilitazione dello strumento: `channels.whatsapp.actions.reactions` (predefinito: abilitato).

<div id="limits">
  ## Limiti
</div>

* Il testo in uscita viene suddiviso in blocchi secondo `channels.whatsapp.textChunkLimit` (valore predefinito 4000).
* Suddivisione opzionale per righe vuote: imposta `channels.whatsapp.chunkMode="newline"` per dividere sulle righe vuote (delimitazioni di paragrafo) prima della suddivisione per lunghezza.
* I salvataggi dei contenuti multimediali in ingresso sono limitati da `channels.whatsapp.mediaMaxMb` (valore predefinito 50 MB).
* Gli elementi multimediali in uscita sono limitati da `agents.defaults.mediaMaxMb` (valore predefinito 5 MB).

<div id="outbound-send-text-media">
  ## Invio in uscita (testo + media)
</div>

* Usa il listener web attivo; errore se il Gateway non è in esecuzione.
* Suddivisione del testo: massimo 4k per messaggio (configurabile tramite `channels.whatsapp.textChunkLimit`, opzionale `channels.whatsapp.chunkMode`).
* Media:
  * Sono supportati immagini/video/audio/documenti.
  * Audio inviato come PTT; `audio/ogg` =&gt; `audio/ogg; codecs=opus`.
  * Didascalia solo sul primo elemento multimediale.
  * Il fetch dei media supporta HTTP(S) e percorsi locali.
  * GIF animate: WhatsApp si aspetta MP4 con `gifPlayback: true` per la riproduzione in loop in linea.
    * CLI: `openclaw message send --media <mp4> --gif-playback`
    * Gateway: i parametri di `send` includono `gifPlayback: true`

<div id="voice-notes-ptt-audio">
  ## Note vocali (audio PTT)
</div>

WhatsApp invia l&#39;audio come **messaggi vocali** (bolle PTT).

* Risultati migliori: OGG/Opus. OpenClaw riscrive `audio/ogg` in `audio/ogg; codecs=opus`.
* `[[audio_as_voice]]` viene ignorato per WhatsApp (l&#39;audio viene già recapitato come messaggio vocale).

<div id="media-limits-optimization">
  ## Limiti e ottimizzazione dei contenuti multimediali
</div>

* Limite predefinito in uscita: 5 MB (per elemento multimediale).
* Override: `agents.defaults.mediaMaxMb`.
* Le immagini vengono ottimizzate automaticamente in JPEG entro il limite (ridimensionamento + ottimizzazione della qualità).
* Contenuti multimediali troppo grandi =&gt; errore; la risposta multimediale viene sostituita da un avviso testuale.

<div id="heartbeats">
  ## Heartbeat
</div>

* Il **Gateway heartbeat** registra lo stato di salute della connessione (`web.heartbeatSeconds`, predefinito 60s).
* L&#39;**Agent heartbeat** può essere configurato per singolo agente (`agents.list[].heartbeat`) o globalmente
  tramite `agents.defaults.heartbeat` (usato come fallback quando non sono impostate voci per agente).
  * Usa il prompt di heartbeat configurato (predefinito: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`) + il comportamento di skip `HEARTBEAT_OK`.
  * La consegna predefinita avviene sull&#39;ultimo canale utilizzato (o sulla destinazione configurata).

<div id="reconnect-behavior">
  ## Comportamento di riconnessione
</div>

* Politica di backoff: `web.reconnect`:
  * `initialMs`, `maxMs`, `factor`, `jitter`, `maxAttempts`.
* Se viene raggiunto `maxAttempts`, il monitoraggio web si interrompe (modalità degradata).
* Logout =&gt; interrompi e richiedi nuovo collegamento.

<div id="config-quick-map">
  ## Mappa rapida della configurazione
</div>

* `channels.whatsapp.dmPolicy` (criterio DM: abbinamento/lista di autorizzati/open/disabilitato).
* `channels.whatsapp.selfChatMode` (configurazione sullo stesso telefono; il bot usa il tuo numero WhatsApp personale).
* `channels.whatsapp.allowFrom` (lista di autorizzati per i DM). WhatsApp usa numeri di telefono in formato E.164 (nessun username).
* `channels.whatsapp.mediaMaxMb` (limite massimo per il salvataggio dei contenuti multimediali in ingresso).
* `channels.whatsapp.ackReaction` (reazione automatica alla ricezione del messaggio: `{emoji, direct, group}`).
* `channels.whatsapp.accounts.<accountId>.*` (impostazioni per account + `authDir` opzionale).
* `channels.whatsapp.accounts.<accountId>.mediaMaxMb` (limite dei contenuti multimediali in ingresso per account).
* `channels.whatsapp.accounts.<accountId>.ackReaction` (override per-account della reazione di ack).
* `channels.whatsapp.groupAllowFrom` (lista di autorizzati per i mittenti nei gruppi).
* `channels.whatsapp.groupPolicy` (criterio per i gruppi).
* `channels.whatsapp.historyLimit` / `channels.whatsapp.accounts.<accountId>.historyLimit` (contesto di cronologia del gruppo; `0` disabilita).
* `channels.whatsapp.dmHistoryLimit` (limite di cronologia dei DM in turni utente). Override per utente: `channels.whatsapp.dms["<phone>"].historyLimit`.
* `channels.whatsapp.groups` (lista di autorizzati per i gruppi + impostazioni predefinite di controllo delle menzioni; usa `"*"` per consentire tutti)
* `channels.whatsapp.actions.reactions` (controlla/le reazioni degli strumenti WhatsApp).
* `agents.list[].groupChat.mentionPatterns` (o `messages.groupChat.mentionPatterns`)
* `messages.groupChat.historyLimit`
* `channels.whatsapp.messagePrefix` (prefisso in ingresso; per-account: `channels.whatsapp.accounts.<accountId>.messagePrefix`; deprecato: `messages.messagePrefix`)
* `messages.responsePrefix` (prefisso in uscita)
* `agents.defaults.mediaMaxMb`
* `agents.defaults.heartbeat.every`
* `agents.defaults.heartbeat.model` (override opzionale)
* `agents.defaults.heartbeat.target`
* `agents.defaults.heartbeat.to`
* `agents.defaults.heartbeat.session`
* `agents.list[].heartbeat.*` (override per-agente)
* `session.*` (scope, idle, store, mainKey)
* `web.enabled` (disabilita l&#39;avvio del canale quando è impostato su false)
* `web.heartbeatSeconds`
* `web.reconnect.*`

<div id="logs-troubleshooting">
  ## Log e risoluzione dei problemi
</div>

* Sottosistemi: `whatsapp/inbound`, `whatsapp/outbound`, `web-heartbeat`, `web-reconnect`.
* File di log: `/tmp/openclaw/openclaw-YYYY-MM-DD.log` (configurabile).
* Guida alla risoluzione dei problemi: [Risoluzione dei problemi del Gateway](/it/gateway/troubleshooting).

<div id="troubleshooting-quick">
  ## Risoluzione dei problemi (rapida)
</div>

**Non collegato / accesso tramite QR richiesto**

* Sintomo: `channels status` mostra `linked: false` o segnala “Not linked”.
* Correzione: esegui `openclaw channels login` sull&#39;host del Gateway e scansiona il codice QR (WhatsApp → Settings → Linked Devices).

**Collegato ma disconnesso / loop di riconnessione**

* Sintomo: `channels status` mostra `running, disconnected` o segnala “Linked but disconnected”.
* Correzione: esegui `openclaw doctor` (oppure riavvia il Gateway). Se il problema persiste, ricollega tramite `channels login` e controlla `openclaw logs --follow`.

**Runtime Bun**

* Bun è **sconsigliato**. WhatsApp (Baileys) e Telegram sono instabili su Bun.
  Esegui il Gateway con **Node**. (Vedi la nota sul runtime in Getting Started.)