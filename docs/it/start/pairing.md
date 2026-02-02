---
title: Abbinamento
summary: "Panoramica dell'abbinamento: approva chi può inviarti DM e quali nodi possono connettersi"
read_when:
  - Configurazione del controllo di accesso ai DM
  - Abbinamento di un nuovo nodo iOS/Android
  - Revisione dell'assetto di sicurezza di OpenClaw
---

<div id="pairing">
  # Abbinamento
</div>

L’“abbinamento” è la fase esplicita di **approvazione da parte del proprietario** in OpenClaw.
Viene utilizzato in due contesti:

1. **Abbinamento DM** (chi è autorizzato a interagire con il bot)
2. **Abbinamento nodo** (quali dispositivi/nodi sono autorizzati a unirsi alla rete del Gateway)

Contesto di sicurezza: [Sicurezza](/it/gateway/security)

<div id="1-dm-pairing-inbound-chat-access">
  ## 1) Abbinamento DM (accesso alla chat in ingresso)
</div>

Quando un canale è configurato con la policy DM `pairing`, i mittenti sconosciuti ricevono un codice breve e il loro messaggio **non viene elaborato** finché non lo approvi.

Le policy DM predefinite sono documentate in: [Security](/it/gateway/security)

Codici di abbinamento:

* 8 caratteri, maiuscoli, senza caratteri ambigui (`0O1I`).
* **Scadono dopo 1 ora**. Il bot invia il messaggio di abbinamento solo quando viene creata una nuova richiesta (circa una volta all&#39;ora per mittente).
* Le richieste di abbinamento DM in sospeso sono limitate a **3 per canale** per impostazione predefinita; le richieste aggiuntive vengono ignorate finché una non scade o viene approvata.

<div id="approve-a-sender">
  ### Approva un mittente
</div>

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

Canali supportati: `telegram`, `whatsapp`, `signal`, `imessage`, `discord`, `slack`.

<div id="where-the-state-lives">
  ### Dove viene memorizzato lo stato
</div>

Archiviato in `~/.openclaw/credentials/`:

* Richieste in sospeso: `<channel>-pairing.json`
* Archivio della lista di autorizzati approvati: `<channel>-allowFrom.json`

Considerali dati sensibili (regolano l&#39;accesso al tuo assistente).

<div id="2-node-device-pairing-iosandroidmacosheadless-nodes">
  ## 2) Abbinamento del dispositivo nodo (nodi iOS/Android/macOS/headless)
</div>

I nodi si connettono al Gateway come **dispositivi** con ruolo `role: node`. Il Gateway
crea una richiesta di abbinamento del dispositivo che deve essere approvata.

<div id="approve-a-node-device">
  ### Approva un nodo
</div>

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

### Dove viene conservato lo stato

Salvato in `~/.openclaw/devices/`:

* `pending.json` (a breve durata; le richieste in sospeso scadono)
* `paired.json` (dispositivi associati + token)

<div id="notes">
  ### Note
</div>

* La legacy API `node.pair.*` (CLI: `openclaw nodes pending/approve`) è un
  archivio di abbinamento separato, gestito dal Gateway. I nodi WS continuano comunque a richiedere l&#39;abbinamento del dispositivo.

<div id="related-docs">
  ## Documenti correlati
</div>

* Modello di sicurezza e prompt injection: [Sicurezza](/it/gateway/security)
* Aggiornamento in sicurezza (esegui il comando doctor): [Aggiornamento](/it/install/updating)
* Configurazioni dei canali:
  * Telegram: [Telegram](/it/channels/telegram)
  * WhatsApp: [WhatsApp](/it/channels/whatsapp)
  * Signal: [Signal](/it/channels/signal)
  * iMessage: [iMessage](/it/channels/imessage)
  * Discord: [Discord](/it/channels/discord)
  * Slack: [Slack](/it/channels/slack)