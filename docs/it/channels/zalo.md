---
title: Zalo
summary: "Stato del supporto per i bot Zalo, funzionalità e configurazione"
read_when:
  - Stai lavorando su funzionalità o webhook Zalo
---

<div id="zalo-bot-api">
  # Zalo (Bot API)
</div>

Stato: sperimentale. Solo messaggi diretti; supporto ai gruppi in arrivo a breve, stando alla documentazione Zalo.

<div id="plugin-required">
  ## Plugin richiesto
</div>

Zalo viene distribuito come plugin e non è incluso nell&#39;installazione principale.

* Installa tramite CLI: `openclaw plugins install @openclaw/zalo`
* Oppure seleziona **Zalo** durante l&#39;onboarding e conferma il prompt di installazione
* Dettagli: [Plugin](/it/plugin)

<div id="quick-setup-beginner">
  ## Configurazione rapida (per principianti)
</div>

1. Installa il plugin Zalo:
   * Da una copia locale del sorgente: `openclaw plugins install ./extensions/zalo`
   * Da npm (se pubblicato): `openclaw plugins install @openclaw/zalo`
   * Oppure seleziona **Zalo** durante l&#39;onboarding e conferma il prompt di installazione
2. Imposta il token:
   * Env: `ZALO_BOT_TOKEN=...`
   * Oppure config: `channels.zalo.botToken: "..."`.
3. Riavvia il Gateway (o completa l&#39;onboarding).
4. L&#39;accesso ai DM è basato su abbinamento per impostazione predefinita; approva il codice di abbinamento al primo contatto.

Configurazione minima:

```json5
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing"
    }
  }
}
```

<div id="what-it-is">
  ## Di cosa si tratta
</div>

Zalo è un&#39;app di messaggistica incentrata sul Vietnam; la sua Bot API consente al Gateway di eseguire un bot per conversazioni 1:1.
È adatta per casi di supporto o notifiche in cui desideri un instradamento deterministico che riporti sempre a Zalo.

* Un canale Zalo Bot API gestito dal Gateway.
* Instradamento deterministico: le risposte tornano a Zalo; il modello non sceglie mai i canali.
* I DM condividono la sessione principale dell&#39;agente.
* I gruppi non sono ancora supportati (la documentazione Zalo riporta &quot;coming soon&quot;).

<div id="setup-fast-path">
  ## Configurazione rapida
</div>

<div id="1-create-a-bot-token-zalo-bot-platform">
  ### 1) Crea un token del bot (Zalo Bot Platform)
</div>

1. Vai su **https://bot.zaloplatforms.com** e accedi.
2. Crea un nuovo bot e configuralo.
3. Copia il token del bot (formato: `12345689:abc-xyz`).

<div id="2-configure-the-token-env-or-config">
  ### 2) Configura il token (env o config)
</div>

Esempio:

```json5
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing"
    }
  }
}
```

Opzione di ambiente: `ZALO_BOT_TOKEN=...` (funziona solo per l&#39;account predefinito).

Supporto multi-account: utilizza `channels.zalo.accounts` con token per account e `name` opzionale.

3. Riavvia il Gateway. Zalo viene avviato quando un token viene risolto (env o config).
4. Per l&#39;accesso ai DM l&#39;impostazione predefinita è l&#39;abbinamento. Approva il codice quando il bot viene contattato per la prima volta.

<div id="how-it-works-behavior">
  ## Come funziona (comportamento)
</div>

* I messaggi in ingresso vengono normalizzati nel formato envelope di canale condiviso, con segnaposto per i contenuti multimediali.
* Le risposte vengono sempre instradate nella stessa chat Zalo.
* Long-polling per impostazione predefinita; modalità webhook disponibile con `channels.zalo.webhookUrl`.

<div id="limits">
  ## Limiti
</div>

* Il testo in uscita viene suddiviso in blocchi da 2000 caratteri (limite API Zalo).
* Il download e l’upload di contenuti multimediali sono limitati da `channels.zalo.mediaMaxMb` (valore predefinito 5).
* Lo streaming è disabilitato per impostazione predefinita perché il limite di 2000 caratteri lo rende poco utile.

<div id="access-control-dms">
  ## Controllo degli accessi (DM)
</div>

<div id="dm-access">
  ### Accesso ai DM
</div>

* Impostazione predefinita: `channels.zalo.dmPolicy = "pairing"`. I mittenti sconosciuti ricevono un codice di abbinamento; i messaggi vengono ignorati fino all&#39;approvazione (i codici scadono dopo 1 ora).
* Approva con:
  * `openclaw pairing list zalo`
  * `openclaw pairing approve zalo <CODE>`
* L&#39;abbinamento è lo scambio di token predefinito. Dettagli: [Pairing](/it/start/pairing)
* `channels.zalo.allowFrom` accetta ID utente numerici (non è disponibile la ricerca per username).

<div id="long-polling-vs-webhook">
  ## Long-polling vs webhook
</div>

* Predefinito: long-polling (non è richiesto un URL pubblico).
* Modalità webhook: imposta `channels.zalo.webhookUrl` e `channels.zalo.webhookSecret`.
  * Il secret del webhook deve essere compreso tra 8 e 256 caratteri.
  * L&#39;URL del webhook deve usare HTTPS.
  * Zalo invia gli eventi con l&#39;header `X-Bot-Api-Secret-Token` per la verifica.
  * Il server HTTP del Gateway gestisce le richieste webhook su `channels.zalo.webhookPath` (per impostazione predefinita corrisponde al path dell&#39;URL del webhook).

**Nota:** getUpdates (polling) e webhook si escludono a vicenda, come indicato nella documentazione API di Zalo.

<div id="supported-message-types">
  ## Tipi di messaggi supportati
</div>

* **Messaggi di testo**: Supporto completo con suddivisione in blocchi da 2000 caratteri.
* **Messaggi con immagini**: Download ed elaborazione delle immagini in ingresso; invio di immagini tramite `sendPhoto`.
* **Sticker**: Registrati nei log ma non completamente elaborati (nessuna risposta dell&#39;agente).
* **Tipi non supportati**: Registrati nei log (ad esempio, messaggi da utenti protetti).

<div id="capabilities">
  ## Funzionalità
</div>

| Funzionalità | Stato |
|--------------|-------|
| Messaggi diretti | ✅ Supportati |
| Gruppi | ❌ In arrivo (secondo la documentazione di Zalo) |
| Media (immagini) | ✅ Supportati |
| Reazioni | ❌ Non supportate |
| Thread | ❌ Non supportati |
| Sondaggi | ❌ Non supportati |
| Comandi nativi | ❌ Non supportati |
| Streaming | ⚠️ Bloccato (limite di 2000 caratteri) |

<div id="delivery-targets-clicron">
  ## Destinazioni di invio (CLI/cron)
</div>

* Usa un ID chat come destinazione.
* Esempio: `openclaw message send --channel zalo --target 123456789 --message "hi"`.

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

**Il bot non risponde:**

* Verifica che il token sia valido: `openclaw channels status --probe`
* Verifica che il mittente sia approvato (abbinamento o allowFrom)
* Controlla i log del Gateway: `openclaw logs --follow`

**Il webhook non riceve eventi:**

* Assicurati che l&#39;URL del webhook utilizzi HTTPS
* Verifica che il token segreto sia lungo 8-256 caratteri
* Conferma che l&#39;endpoint HTTP del Gateway sia raggiungibile sul percorso configurato
* Controlla che il polling getUpdates non sia in esecuzione (si escludono a vicenda)

<div id="configuration-reference-zalo">
  ## Riferimento configurazione (Zalo)
</div>

Configurazione completa: [Configurazione](/it/gateway/configuration)

Opzioni provider:

* `channels.zalo.enabled`: abilita/disabilita l&#39;avvio del canale.
* `channels.zalo.botToken`: token del bot dalla Zalo Bot Platform.
* `channels.zalo.tokenFile`: legge il token dal percorso file.
* `channels.zalo.dmPolicy`: `pairing | allowlist | open | disabled` (predefinito: pairing).
* `channels.zalo.allowFrom`: lista di autorizzati per DM (ID utente). `open` richiede `"*"`. La procedura guidata richiederà ID numerici.
* `channels.zalo.mediaMaxMb`: limite per i contenuti multimediali in ingresso/uscita (MB, predefinito 5).
* `channels.zalo.webhookUrl`: abilita la modalità webhook (HTTPS richiesto).
* `channels.zalo.webhookSecret`: segreto del webhook (8-256 caratteri).
* `channels.zalo.webhookPath`: percorso del webhook sul server HTTP del Gateway.
* `channels.zalo.proxy`: URL del proxy per le richieste API.

Opzioni multiaccount:

* `channels.zalo.accounts.<id>.botToken`: token per account.
* `channels.zalo.accounts.<id>.tokenFile`: file del token per account.
* `channels.zalo.accounts.<id>.name`: nome visualizzato.
* `channels.zalo.accounts.<id>.enabled`: abilita/disabilita l&#39;account.
* `channels.zalo.accounts.<id>.dmPolicy`: policy DM per account.
* `channels.zalo.accounts.<id>.allowFrom`: lista di autorizzati per account.
* `channels.zalo.accounts.<id>.webhookUrl`: URL del webhook per account.
* `channels.zalo.accounts.<id>.webhookSecret`: segreto del webhook per account.
* `channels.zalo.accounts.<id>.webhookPath`: percorso del webhook per account.
* `channels.zalo.accounts.<id>.proxy`: URL del proxy per account.