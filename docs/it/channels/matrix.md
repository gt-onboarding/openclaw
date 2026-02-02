---
title: Matrix
summary: "Stato del supporto per Matrix, funzionalità e configurazione"
read_when:
  - Quando lavori sulle funzionalità del canale Matrix
---

<div id="matrix-plugin">
  # Matrix (plugin)
</div>

Matrix è un protocollo di messaggistica aperto e decentralizzato. OpenClaw si connette come **utente**
Matrix su qualsiasi homeserver, quindi ti serve un account Matrix per il bot. Una volta effettuato l&#39;accesso, puoi inviare DM
direttamente al bot oppure invitarlo nelle stanze (i &quot;gruppi&quot; di Matrix). Beeper è anche un client valido,
ma richiede che l&#39;E2EE sia abilitata.

Stato: supportato tramite plugin (@vector-im/matrix-bot-sdk). Messaggi diretti, stanze, thread, media, reazioni,
sondaggi (send + poll-start come testo), posizione ed E2EE (con supporto crittografico).

<div id="plugin-required">
  ## Plugin richiesto
</div>

Matrix è distribuito come plugin e non è incluso nell&#39;installazione principale.

Installalo tramite CLI (registry npm):

```bash
openclaw plugins install @openclaw/matrix
```

Copia locale del repository (quando esegui da un repo Git):

```bash
openclaw plugins install ./extensions/matrix
```

Se scegli Matrix durante la fase di configurazione/onboarding e viene rilevato un checkout git,
OpenClaw ti suggerirà automaticamente il percorso di installazione locale.

Dettagli: [Plugin](/it/plugin)

<div id="setup">
  ## Configurazione
</div>

1. Installa il plugin Matrix:
   * Da npm: `openclaw plugins install @openclaw/matrix`
   * Da una copia locale del repository: `openclaw plugins install ./extensions/matrix`
2. Crea un account Matrix su un homeserver:
   * Esamina le opzioni di hosting su [https://matrix.org/ecosystem/hosting/](https://matrix.org/ecosystem/hosting/)
   * Oppure ospitalo in self-hosting.
3. Ottieni un access token per l&#39;account bot:

   * Usa la Matrix login API con `curl` sul tuo homeserver:

   ```bash
   curl --request POST \
     --url https://matrix.example.org/_matrix/client/v3/login \
     --header 'Content-Type: application/json' \
     --data '{
     "type": "m.login.password",
     "identifier": {
       "type": "m.id.user",
       "user": "your-user-name"
     },
     "password": "your-password"
   }'
   ```

   * Sostituisci `matrix.example.org` con l&#39;URL del tuo homeserver.
   * Oppure imposta `channels.matrix.userId` + `channels.matrix.password`: OpenClaw chiama lo stesso
     endpoint di login, memorizza l&#39;access token in `~/.openclaw/credentials/matrix/credentials.json`
     e lo riutilizza al prossimo avvio.
4. Configura le credenziali:
   * Variabili d&#39;ambiente: `MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN` (oppure `MATRIX_USER_ID` + `MATRIX_PASSWORD`)
   * Oppure config: `channels.matrix.*`
   * Se entrambe sono impostate, la config ha la precedenza.
   * Con un access token: l&#39;user ID viene recuperato automaticamente tramite `/whoami`.
   * Quando impostato, `channels.matrix.userId` deve essere il Matrix ID completo (esempio: `@bot:example.org`).
5. Riavvia il Gateway (o termina l&#39;onboarding).
6. Avvia un DM con il bot o invitalo in una stanza da qualunque client Matrix
   (Element, Beeper, ecc.; vedi https://matrix.org/ecosystem/clients/). Beeper richiede E2EE,
   quindi imposta `channels.matrix.encryption: true` e verifica il dispositivo.

Configurazione minima (access token, user ID recuperato automaticamente):

```json5
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      dm: { policy: "abbinamento" }
    }
  }
}
```

Configurazione E2EE (crittografia end-to-end attivata):

```json5
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      encryption: true,
      dm: { policy: "pairing" }
    }
  }
}
```

<div id="encryption-e2ee">
  ## Crittografia (E2EE)
</div>

La crittografia end-to-end è **supportata** tramite il Rust crypto SDK.

Abilitala impostando `channels.matrix.encryption: true`:

* Se il modulo di crittografia viene caricato, le stanze crittografate vengono decrittate automaticamente.
* I contenuti multimediali in uscita vengono crittografati quando vengono inviati a stanze crittografate.
* Alla prima connessione, OpenClaw richiede la verifica del dispositivo dalle altre tue sessioni.
* Verifica il dispositivo in un altro client Matrix (Element, ecc.) per abilitare la condivisione delle chiavi.
* Se il modulo di crittografia non può essere caricato, l&#39;E2EE è disabilitata e le stanze crittografate non verranno decrittate;
  OpenClaw registrerà un avviso nei log.
* Se visualizzi errori relativi al modulo di crittografia mancante (per esempio, `@matrix-org/matrix-sdk-crypto-nodejs-*`),
  consenti l&#39;esecuzione degli script di build per `@matrix-org/matrix-sdk-crypto-nodejs` ed esegui
  `pnpm rebuild @matrix-org/matrix-sdk-crypto-nodejs` oppure scarica il binario con
  `node node_modules/@matrix-org/matrix-sdk-crypto-nodejs/download-lib.js`.

Lo stato della crittografia è memorizzato per account + token di accesso in
`~/.openclaw/matrix/accounts/<account>/<homeserver>__<user>/<token-hash>/crypto/`
(database SQLite). Lo stato di sincronizzazione è memorizzato nello stesso percorso, in `bot-storage.json`.
Se il token di accesso (dispositivo) cambia, viene creato un nuovo archivio e il bot deve essere
nuovamente verificato per le stanze crittografate.

**Verifica del dispositivo:**
Quando l&#39;E2EE è abilitata, all&#39;avvio il bot richiederà la verifica dalle altre tue sessioni.
Apri Element (o un altro client) e approva la richiesta di verifica per stabilire la fiducia.
Una volta verificato, il bot può decrittare i messaggi nelle stanze crittografate.

<div id="routing-model">
  ## Modello di instradamento
</div>

* Le risposte vengono sempre inviate a Matrix.
* I DM condividono la sessione principale dell’agente; le stanze sono mappate a sessioni di gruppo.

<div id="access-control-dms">
  ## Controllo accessi (DM)
</div>

* Predefinito: `channels.matrix.dm.policy = "pairing"`. I mittenti sconosciuti ricevono un codice di abbinamento.
* Approva tramite:
  * `openclaw pairing list matrix`
  * `openclaw pairing approve matrix <CODE>`
* DM pubblici: `channels.matrix.dm.policy="open"` più `channels.matrix.dm.allowFrom=["*"]`.
* `channels.matrix.dm.allowFrom` accetta ID utente o nomi visualizzati. La procedura guidata risolve i nomi visualizzati in ID utente quando è disponibile la ricerca nella directory.

<div id="rooms-groups">
  ## Stanze (gruppi)
</div>

* Predefinito: `channels.matrix.groupPolicy = "allowlist"` (limitato alle menzioni). Usa `channels.defaults.groupPolicy` per sovrascrivere il valore predefinito quando non è impostato.
* Autorizza le stanze con `channels.matrix.groups` (ID stanza, alias o nomi):

```json5
{
  channels: {
    matrix: {
      groupPolicy: "allowlist",
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true }
      },
      groupAllowFrom: ["@owner:example.org"]
    }
  }
}
```

* `requireMention: false` abilita la risposta automatica in quella stanza.
* `groups."*"` può impostare i valori predefiniti per i vincoli sulle menzioni tra le stanze.
* `groupAllowFrom` limita quali mittenti possono attivare il bot nelle stanze (opzionale).
* Le liste di autorizzati `users` per stanza possono limitare ulteriormente i mittenti all&#39;interno di una stanza specifica.
* Il wizard di configurazione chiede le liste di autorizzati delle stanze (ID stanza, alias o nomi) e risolve i nomi quando possibile.
* All&#39;avvio, OpenClaw risolve i nomi di stanze/utenti nelle liste di autorizzati in ID e registra la mappatura; le voci non risolte vengono mantenute come inserite.
* Gli inviti vengono accettati automaticamente per impostazione predefinita; controlla questo comportamento con `channels.matrix.autoJoin` e `channels.matrix.autoJoinAllowlist`.
* Per non consentire **alcuna stanza**, imposta `channels.matrix.groupPolicy: "disabled"` (o mantieni una lista di autorizzati vuota).
* Chiave legacy: `channels.matrix.rooms` (stessa struttura di `groups`).

<div id="threads">
  ## Thread
</div>

* I thread di risposta sono supportati.
* `channels.matrix.threadReplies` controlla se le risposte restano nei thread:
  * `off`, `inbound` (predefinito), `always`
* `channels.matrix.replyToMode` controlla i metadati di risposta quando non si risponde in un thread:
  * `off` (predefinito), `first`, `all`

<div id="capabilities">
  ## Funzionalità
</div>

| Funzionalità | Stato |
|--------------|--------|
| Messaggi diretti | ✅ Supportato |
| Stanze | ✅ Supportato |
| Thread | ✅ Supportato |
| Media | ✅ Supportato |
| E2EE | ✅ Supportato (modulo crittografico richiesto) |
| Reazioni | ✅ Supportato (send/read tramite tool) |
| Sondaggi | ✅ Invio supportato; gli avvii di sondaggi in ingresso vengono convertiti in testo (risposte/terminazioni ignorate) |
| Posizione | ✅ Supportato (URI geo; altitudine ignorata) |
| Comandi nativi | ✅ Supportato |

<div id="configuration-reference-matrix">
  ## Riferimento di configurazione (Matrix)
</div>

Configurazione completa: [Configuration](/it/gateway/configuration)

Opzioni del provider:

* `channels.matrix.enabled`: abilita/disabilita l&#39;avvio del canale.
* `channels.matrix.homeserver`: URL dell&#39;homeserver.
* `channels.matrix.userId`: ID utente Matrix (opzionale con access token).
* `channels.matrix.accessToken`: access token.
* `channels.matrix.password`: password per l&#39;accesso (token memorizzato).
* `channels.matrix.deviceName`: nome visualizzato del dispositivo.
* `channels.matrix.encryption`: abilita E2EE (predefinito: false).
* `channels.matrix.initialSyncLimit`: limite di sincronizzazione iniziale.
* `channels.matrix.threadReplies`: `off | inbound | always` (predefinito: inbound).
* `channels.matrix.textChunkLimit`: dimensione dei blocchi di testo in uscita (caratteri).
* `channels.matrix.chunkMode`: `length` (predefinito) o `newline` per suddividere sulle righe vuote (confini di paragrafo) prima del frazionamento per lunghezza.
* `channels.matrix.dm.policy`: `pairing | allowlist | open | disabled` (predefinito: pairing).
* `channels.matrix.dm.allowFrom`: lista di autorizzati per DM (ID utente o nomi visualizzati). `open` richiede `"*"`. Il wizard risolve i nomi in ID quando possibile.
* `channels.matrix.groupPolicy`: `allowlist | open | disabled` (predefinito: allowlist).
* `channels.matrix.groupAllowFrom`: mittenti autorizzati per i messaggi di gruppo.
* `channels.matrix.allowlistOnly`: applica forzatamente le regole della lista di autorizzati per DM + stanze.
* `channels.matrix.groups`: lista di autorizzati per gruppi + mappa delle impostazioni per stanza.
* `channels.matrix.rooms`: configurazione/lista di autorizzati legacy per gruppi.
* `channels.matrix.replyToMode`: modalità di risposta per thread/tag.
* `channels.matrix.mediaMaxMb`: limite per i media in ingresso/uscita (MB).
* `channels.matrix.autoJoin`: gestione degli inviti (`always | allowlist | off`, predefinito: always).
* `channels.matrix.autoJoinAllowlist`: ID/alias di stanza consentiti per l’auto-join.
* `channels.matrix.actions`: limitazione per azione degli strumenti (reactions/messages/pins/memberInfo/channelInfo).