---
title: Tlon
summary: "Stato del supporto, funzionalità e configurazione di Tlon/Urbit"
read_when:
  - Quando lavori alle funzionalità del canale Tlon/Urbit
---

<div id="tlon-plugin">
  # Tlon (plugin)
</div>

Tlon è un sistema di messaggistica decentralizzato basato su Urbit. OpenClaw si connette alla tua ship Urbit e può
rispondere ai DM e ai messaggi nelle chat di gruppo. Le risposte nei gruppi richiedono per impostazione predefinita una menzione @ e possono
essere ulteriormente ristrette tramite liste di autorizzati.

Stato: supportato come plugin. DM, menzioni nei gruppi, risposte ai thread e fallback solo testuale per i contenuti multimediali
(URL aggiunto alla didascalia). Reazioni, sondaggi e caricamenti nativi di contenuti multimediali non sono supportati.

<div id="plugin-required">
  ## Plugin necessario
</div>

Tlon viene distribuito come plugin e non è incluso nell&#39;installazione di base.

Installa tramite CLI (registry npm):

```bash
openclaw plugins install @openclaw/tlon
```

Checkout locale (se esegui da un repository Git):

```bash
openclaw plugins install ./extensions/tlon
```

Dettagli: [Plugin](/it/plugin)

<div id="setup">
  ## Configurazione
</div>

1. Installa il plugin Tlon.
2. Procurati l&#39;URL della tua ship e il codice di accesso.
3. Configura `channels.tlon`.
4. Riavvia il Gateway.
5. Invia un DM al bot o menzionalo in un canale di gruppo.

Configurazione minima (singolo account):

```json5
{
  channels: {
    tlon: {
      enabled: true,
      ship: "~sampel-palnet",
      url: "https://your-ship-host",
      code: "lidlut-tabwed-pillex-ridrup"
    }
  }
}
```

<div id="group-channels">
  ## Canali di gruppo
</div>

Il rilevamento automatico è abilitato per impostazione predefinita. Puoi anche fissare manualmente i canali:

```json5
{
  channels: {
    tlon: {
      groupChannels: [
        "chat/~host-ship/general",
        "chat/~host-ship/support"
      ]
    }
  }
}
```

Disattiva il rilevamento automatico:

```json5
{
  channels: {
    tlon: {
      autoDiscoverChannels: false
    }
  }
}
```

<div id="access-control">
  ## Controllo degli accessi
</div>

lista di autorizzati per i DM (vuota = consente tutti):

```json5
{
  channels: {
    tlon: {
      dmAllowlist: ["~zod", "~nec"]
    }
  }
}
```

Autorizzazione per gruppi (ristretta per impostazione predefinita):

```json5
{
  channels: {
    tlon: {
      defaultAuthorizedShips: ["~zod"],
      authorization: {
        channelRules: {
          "chat/~host-ship/general": {
            mode: "restricted",
            allowedShips: ["~zod", "~nec"]
          },
          "chat/~host-ship/announcements": {
            mode: "open"
          }
        }
      }
    }
  }
}
```

<div id="delivery-targets-clicron">
  ## Destinazioni di invio (CLI/cron)
</div>

Utilizza queste destinazioni con `openclaw message send` o con la consegna tramite cron:

* DM: `~sampel-palnet` o `dm/~sampel-palnet`
* Gruppo: `chat/~host-ship/channel` o `group:~host-ship/channel`

<div id="notes">
  ## Note
</div>

* Le risposte nei gruppi richiedono una menzione (ad es. `~your-bot-ship`) perché il bot possa rispondere.
* Risposte ai thread: se il messaggio in ingresso è in un thread, OpenClaw risponde nello stesso thread.
* Media: `sendMedia` esegue un fallback a testo + URL (nessun upload nativo dei file).