---
title: Line
summary: "Installazione, configurazione e utilizzo del plugin LINE Messaging API"
read_when:
  - Vuoi connettere OpenClaw a LINE
  - Devi configurare il webhook LINE e le credenziali
  - Vuoi opzioni di messaggistica specifiche per LINE
---

<div id="line-plugin">
  # LINE (plugin)
</div>

LINE si connette a OpenClaw tramite la LINE Messaging API. Il plugin viene eseguito sul Gateway come
webhook receiver e utilizza il tuo channel access token + channel secret per
l'autenticazione.

Stato: supportato tramite plugin. Sono supportati messaggi diretti, chat di gruppo, media,
posizioni, messaggi Flex, messaggi template e quick reply. Le reazioni e i thread non
sono supportati.

<div id="plugin-required">
  ## Plugin necessario
</div>

Installa il plugin LINE:

```bash
openclaw plugins install @openclaw/line
```

Checkout locale (quando esegui da un repository Git):

```bash
openclaw plugins install ./extensions/line
```


<div id="setup">
  ## Configurazione
</div>

1. Crea un account LINE Developers e apri la Console:
   https://developers.line.biz/console/
2. Crea (o seleziona) un provider e aggiungi un canale **Messaging API**.
3. Copia il **Channel access token** e il **Channel secret** dalle impostazioni del canale.
4. Abilita **Use webhook** nelle impostazioni della Messaging API.
5. Imposta l&#39;URL del webhook sull&#39;endpoint del tuo Gateway (HTTPS richiesto):

```
https://gateway-host/line/webhook
```

Il Gateway risponde alla verifica del webhook di LINE (GET) e agli eventi in entrata (POST).
Se hai bisogno di un path personalizzato, imposta `channels.line.webhookPath` o
`channels.line.accounts.<id>.webhookPath` e aggiorna l&#39;URL di conseguenza.


<div id="setup">
  ## Configurazione
</div>

Configurazione minima:

```json5
{
  channels: {
    line: {
      enabled: true,
      channelAccessToken: "LINE_CHANNEL_ACCESS_TOKEN",
      channelSecret: "LINE_CHANNEL_SECRET",
      dmPolicy: "pairing"
    }
  }
}
```

Variabili d&#39;ambiente (solo account predefinito):

* `LINE_CHANNEL_ACCESS_TOKEN`
* `LINE_CHANNEL_SECRET`

File dei token/secret:

```json5
{
  channels: {
    line: {
      tokenFile: "/path/to/line-token.txt",
      secretFile: "/path/to/line-secret.txt"
    }
  }
}
```

Più account:

```json5
{
  channels: {
    line: {
      accounts: {
        marketing: {
          channelAccessToken: "...",
          channelSecret: "...",
          webhookPath: "/line/marketing"
        }
      }
    }
  }
}
```


<div id="access-control">
  ## Controllo accessi
</div>

I messaggi diretti, per impostazione predefinita, usano l&#39;abbinamento. I mittenti sconosciuti ricevono un codice di abbinamento e i loro messaggi vengono ignorati finché non vengono approvati.

```bash
openclaw pairing list line
openclaw pairing approve line <CODE>
```

Liste di autorizzati e policy:

* `channels.line.dmPolicy`: `pairing | allowlist | open | disabled` (`open` = impostazione che consente di accettare messaggi da qualsiasi utente, senza restrizioni)
* `channels.line.allowFrom`: ID utente LINE nella lista di autorizzati per i DM
* `channels.line.groupPolicy`: `allowlist | open | disabled`
* `channels.line.groupAllowFrom`: ID utente LINE nella lista di autorizzati per i gruppi
* Override per singolo gruppo: `channels.line.groups.&lt;groupId&gt;.allowFrom`

Gli ID LINE fanno distinzione tra maiuscole e minuscole. Gli ID validi hanno il seguente formato:

* Utente: `U` + 32 caratteri esadecimali
* Gruppo: `C` + 32 caratteri esadecimali
* Stanza: `R` + 32 caratteri esadecimali


<div id="message-behavior">
  ## Comportamento dei messaggi
</div>

- Il testo è suddiviso in blocchi da 5000 caratteri.
- La formattazione Markdown viene rimossa; i blocchi di codice e le tabelle vengono convertiti in card Flex quando possibile.
- Le risposte in streaming vengono bufferizzate; LINE riceve blocchi completi con un'animazione di caricamento mentre l'agente lavora.
- I download dei contenuti multimediali sono limitati a `channels.line.mediaMaxMb` (valore predefinito: 10).

<div id="channel-data-rich-messages">
  ## Dati del canale (messaggi ricchi)
</div>

Usa `channelData.line` per inviare risposte rapide, posizioni, schede Flex o messaggi modello.

```json5
{
  text: "Here you go",
  channelData: {
    line: {
      quickReplies: ["Status", "Help"],
      location: {
        title: "Office",
        address: "123 Main St",
        latitude: 35.681236,
        longitude: 139.767125
      },
      flexMessage: {
        altText: "Status card",
        contents: { /* Flex payload */ }
      },
      templateMessage: {
        type: "confirm",
        text: "Proceed?",
        confirmLabel: "Yes",
        confirmData: "yes",
        cancelLabel: "No",
        cancelData: "no"
      }
    }
  }
}
```

Il plugin LINE include anche un comando `/card` per i modelli predefiniti di messaggi Flex:

```
/card info "Welcome" "Thanks for joining!"
```


<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

- **Verifica del webhook non riuscita:** assicurati che l'URL del webhook utilizzi HTTPS e che
  `channelSecret` corrisponda a quello configurato nella console LINE.
- **Nessun evento in ingresso:** verifica che il percorso del webhook corrisponda a `channels.line.webhookPath`
  e che il Gateway sia raggiungibile da LINE.
- **Errori di download dei contenuti multimediali:** aumenta `channels.line.mediaMaxMb` se i contenuti multimediali superano
  il limite predefinito.