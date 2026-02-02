---
title: Nextcloud Talk
summary: "Stato del supporto, funzionalità e configurazione di Nextcloud Talk"
read_when:
  - Stai lavorando alle funzionalità del canale Nextcloud Talk
---

<div id="nextcloud-talk-plugin">
  # Nextcloud Talk (plugin)
</div>

Stato: supportato tramite plugin (bot webhook). Messaggi diretti, stanze, reazioni e messaggi in Markdown sono supportati.

<div id="plugin-required">
  ## Plugin richiesto
</div>

Nextcloud Talk viene fornito come plugin e non è incluso nell&#39;installazione principale.

Installa tramite CLI (dal registro npm):

```bash
openclaw plugins install @openclaw/nextcloud-talk
```

Checkout locale (quando esegui da un repository Git):

```bash
openclaw plugins install ./extensions/nextcloud-talk
```

Se durante la configurazione/onboarding scegli Nextcloud Talk e viene rilevato un checkout Git,
OpenClaw proporrà automaticamente il percorso locale di installazione.

Dettagli: [Plugin](/it/plugin)

<div id="quick-setup-beginner">
  ## Configurazione rapida (per principianti)
</div>

1. Installa il plugin Nextcloud Talk.
2. Sul tuo server Nextcloud, crea un bot:
   ```bash
   ./occ talk:bot:install "OpenClaw" "<shared-secret>" "<webhook-url>" --feature reaction
   ```
3. Abilita il bot nelle impostazioni della stanza di destinazione.
4. Configura OpenClaw:
   * Config: `channels.nextcloud-talk.baseUrl` + `channels.nextcloud-talk.botSecret`
   * Oppure env: `NEXTCLOUD_TALK_BOT_SECRET` (solo per l&#39;account predefinito)
5. Riavvia il Gateway (o completa l&#39;onboarding).

Configurazione minima:

```json5
{
  channels: {
    "nextcloud-talk": {
      enabled: true,
      baseUrl: "https://cloud.example.com",
      botSecret: "shared-secret",
      dmPolicy: "pairing"
    }
  }
}
```

<div id="notes">
  ## Note
</div>

* I bot non possono avviare DM. L’utente deve inviare per primo un messaggio al bot.
* L’URL del webhook deve essere raggiungibile dal Gateway; imposta `webhookPublicUrl` se sei dietro un proxy.
* Il caricamento di contenuti multimediali non è supportato dall’API dei bot; i contenuti multimediali vengono inviati come URL.
* Il payload del webhook non distingue tra DM e stanze; imposta `apiUser` + `apiPassword` per abilitare le ricerche del tipo di stanza (altrimenti i DM vengono trattati come stanze).

<div id="access-control-dms">
  ## Controllo degli accessi (DM)
</div>

* Impostazione predefinita: `channels.nextcloud-talk.dmPolicy = "pairing"`. I mittenti sconosciuti ricevono un codice di abbinamento.
* Approva tramite:
  * `openclaw pairing list nextcloud-talk`
  * `openclaw pairing approve nextcloud-talk <CODE>`
* DM pubblici: `channels.nextcloud-talk.dmPolicy="open"` più `channels.nextcloud-talk.allowFrom=["*"]` (impostazione &quot;open&quot;: consente di accettare senza restrizioni messaggi da qualsiasi utente).

<div id="rooms-groups">
  ## Stanze (gruppi)
</div>

* Impostazione predefinita: `channels.nextcloud-talk.groupPolicy = "allowlist"` (accesso basato su menzione).
* Metti le stanze nella lista di autorizzati con `channels.nextcloud-talk.rooms`:

```json5
{
  channels: {
    "nextcloud-talk": {
      rooms: {
        "room-token": { requireMention: true }
      }
    }
  }
}
```

* Per non consentire alcuna stanza, lascia vuota la lista di autorizzati oppure imposta `channels.nextcloud-talk.groupPolicy="disabled"`.

<div id="capabilities">
  ## Capacità
</div>

| Funzionalità | Stato |
|--------------|-------|
| Messaggi diretti | Supportato |
| Stanze | Supportato |
| Thread | Non supportato |
| Contenuti multimediali | Solo tramite URL |
| Reazioni | Supportato |
| Comandi nativi | Non supportato |

<div id="configuration-reference-nextcloud-talk">
  ## Riferimento di configurazione (Nextcloud Talk)
</div>

Configurazione completa: [Configuration](/it/gateway/configuration)

Opzioni del provider:

* `channels.nextcloud-talk.enabled`: abilita/disabilita l&#39;avvio del canale.
* `channels.nextcloud-talk.baseUrl`: URL dell&#39;istanza Nextcloud.
* `channels.nextcloud-talk.botSecret`: segreto condiviso del bot.
* `channels.nextcloud-talk.botSecretFile`: percorso del file del segreto.
* `channels.nextcloud-talk.apiUser`: utente API per la ricerca delle stanze (rilevamento DM).
* `channels.nextcloud-talk.apiPassword`: password API/app per la ricerca delle stanze.
* `channels.nextcloud-talk.apiPasswordFile`: percorso del file della password API.
* `channels.nextcloud-talk.webhookPort`: porta del listener del webhook (predefinita: 8788).
* `channels.nextcloud-talk.webhookHost`: host del webhook (predefinito: 0.0.0.0).
* `channels.nextcloud-talk.webhookPath`: percorso del webhook (predefinito: /nextcloud-talk-webhook).
* `channels.nextcloud-talk.webhookPublicUrl`: URL del webhook raggiungibile dall&#39;esterno.
* `channels.nextcloud-talk.dmPolicy`: `pairing | allowlist | open | disabled`.
* `channels.nextcloud-talk.allowFrom`: lista di autorizzati per DM (ID utente). `open` richiede `"*"`.
* `channels.nextcloud-talk.groupPolicy`: `allowlist | open | disabled`.
* `channels.nextcloud-talk.groupAllowFrom`: lista di autorizzati per i gruppi (ID utente).
* `channels.nextcloud-talk.rooms`: impostazioni per stanza e lista di autorizzati.
* `channels.nextcloud-talk.historyLimit`: limite della cronologia dei gruppi (0 disabilita).
* `channels.nextcloud-talk.dmHistoryLimit`: limite della cronologia DM (0 disabilita).
* `channels.nextcloud-talk.dms`: override per i DM (historyLimit).
* `channels.nextcloud-talk.textChunkLimit`: dimensione dei chunk di testo in uscita (caratteri).
* `channels.nextcloud-talk.chunkMode`: `length` (predefinito) oppure `newline` per suddividere sulle righe vuote (confini di paragrafo) prima del chunking per lunghezza.
* `channels.nextcloud-talk.blockStreaming`: disabilita il block streaming per questo canale.
* `channels.nextcloud-talk.blockStreamingCoalesce`: regolazione della coalescenza del block streaming.
* `channels.nextcloud-talk.mediaMaxMb`: limite massimo dei contenuti multimediali in ingresso (MB).