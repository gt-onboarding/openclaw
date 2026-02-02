---
title: Msteams
summary: "Stato del supporto per i bot di Microsoft Teams, funzionalità e configurazione"
read_when:
  - Quando lavori sulle funzionalità del canale MS Teams
---

<div id="microsoft-teams-plugin">
  # Microsoft Teams (plugin)
</div>

> &quot;Lasciate ogne speranza, voi ch&#39;intrate.&quot;

Aggiornato il: 2026-01-21

Stato: sono supportati testo e allegati nei DM; l&#39;invio di file in canali/gruppi richiede `sharePointSiteId` + permessi Graph (vedi [Inviare file nelle chat di gruppo](#sending-files-in-group-chats)). I sondaggi vengono inviati tramite Adaptive Cards.

<div id="plugin-required">
  ## Plugin richiesto
</div>

Microsoft Teams viene fornito come plugin e non è incluso nell&#39;installazione core.

**Breaking change (2026.1.15):** MS Teams è stato spostato fuori dal core. Se lo utilizzi, devi installare il plugin.

Spiegazione: mantiene le installazioni core più leggere e consente di aggiornare in modo indipendente le dipendenze di MS Teams.

Installa tramite CLI (registry npm):

```bash
openclaw plugins install @openclaw/msteams
```

Checkout locale (quando si esegue da un repository Git):

```bash
openclaw plugins install ./extensions/msteams
```

Se durante la configurazione/onboarding scegli Teams e viene rilevata una checkout Git locale,
OpenClaw proporrà automaticamente il percorso di installazione locale.

Dettagli: [Plugin](/it/plugin)

<div id="quick-setup-beginner">
  ## Configurazione rapida (per principianti)
</div>

1. Installa il plugin Microsoft Teams.
2. Crea un **Azure Bot** (ID dell&#39;app + client secret + tenant ID).
3. Configura OpenClaw con queste credenziali.
4. Esponi `/api/messages` (porta 3978 per impostazione predefinita) tramite un URL pubblico o un tunnel.
5. Installa il pacchetto dell&#39;app Teams e avvia il Gateway.

Configurazione minima:

```json5
{
  channels: {
    msteams: {
      enabled: true,
      appId: "<APP_ID>",
      appPassword: "<APP_PASSWORD>",
      tenantId: "<TENANT_ID>",
      webhook: { port: 3978, path: "/api/messages" }
    }
  }
}
```

Nota: le chat di gruppo sono bloccate per impostazione predefinita (`channels.msteams.groupPolicy: "allowlist"`). Per consentire le risposte nei gruppi, configura `channels.msteams.groupAllowFrom` (oppure usa `groupPolicy: "open"` per consentire i messaggi da qualsiasi membro, previa menzione).

<div id="goals">
  ## Obiettivi
</div>

* Parlare con OpenClaw tramite messaggi diretti (DM) su Teams, chat di gruppo o canali.
* Mantenere l’instradamento deterministico: le risposte tornano sempre al canale da cui sono arrivate.
* Usare per impostazione predefinita un comportamento sicuro nei canali (menzioni obbligatorie, salvo diversa configurazione).

<div id="config-writes">
  ## Modifiche alla configurazione
</div>

Per impostazione predefinita, Microsoft Teams è autorizzato a scrivere aggiornamenti di configurazione attivati da `/config set|unset` (richiede `commands.config: true`).

Per disabilitare:

```json5
{
  channels: { msteams: { configWrites: false } }
}
```

<div id="access-control-dms-groups">
  ## Controllo accessi (DM + gruppi)
</div>

**Accesso ai DM**

* Predefinito: `channels.msteams.dmPolicy = "pairing"`. I mittenti sconosciuti vengono ignorati finché non vengono approvati.
* `channels.msteams.allowFrom` accetta ID oggetto AAD, UPN o nomi visualizzati. Il wizard risolve i nomi in ID tramite Microsoft Graph quando le credenziali lo consentono.

**Accesso ai gruppi**

* Predefinito: `channels.msteams.groupPolicy = "allowlist"` (bloccato a meno che tu non aggiunga `groupAllowFrom`). Usa `channels.defaults.groupPolicy` per sovrascrivere il valore predefinito quando non è impostato.
* `channels.msteams.groupAllowFrom` controlla quali mittenti possono effettuare trigger nelle chat/canali di gruppo (se non impostato, ricade su `channels.msteams.allowFrom`).
* Imposta `groupPolicy: "open"` per consentire qualsiasi membro del gruppo (resta comunque limitato alle menzioni per impostazione predefinita; il token &quot;open&quot; indica che l’accettazione dei messaggi è senza restrizioni da parte di qualunque utente).
* Per non consentire **nessun canale**, imposta `channels.msteams.groupPolicy: "disabled"`.

Esempio:

```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"]
    }
  }
}
```

**Teams + lista di autorizzati per canali**

* Definisci l&#39;ambito delle risposte di gruppo/canale elencando team e canali sotto `channels.msteams.teams`.
* Le chiavi possono essere ID o nomi di team; le chiavi dei canali possono essere ID di conversazione o nomi.
* Quando `groupPolicy="allowlist"` ed è presente una lista di autorizzati dei team, vengono accettati solo i team/canali elencati (limitazione tramite menzione).
* La procedura guidata di configurazione accetta voci `Team/Channel` e le salva per te.
* All&#39;avvio, OpenClaw risolve i nomi dei team/canali e della lista di autorizzati degli utenti in ID (quando le autorizzazioni Graph lo consentono)
  e registra la mappatura; le voci non risolte vengono mantenute così come sono state digitate.

Esempio:

```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      teams: {
        "My Team": {
          channels: {
            "General": { requireMention: true }
          }
        }
      }
    }
  }
}
```

<div id="how-it-works">
  ## Come funziona
</div>

1. Installa il plugin Microsoft Teams.
2. Crea un **Azure Bot** (App ID + secret + tenant ID).
3. Crea un **pacchetto app di Teams** che fa riferimento al bot e include le autorizzazioni RSC riportate sotto.
4. Carica e installa l&#39;app di Teams in un team (o nell&#39;ambito personale per i DM).
5. Configura `msteams` in `~/.openclaw/openclaw.json` (o tramite variabili d&#39;ambiente) e avvia il Gateway.
6. Il Gateway, per impostazione predefinita, rimane in ascolto del traffico webhook del Bot Framework su `/api/messages`.

<div id="azure-bot-setup-prerequisites">
  ## Configurazione Bot di Azure (Prerequisiti)
</div>

Prima di configurare OpenClaw, devi creare una risorsa Bot di Azure.

<div id="step-1-create-azure-bot">
  ### Passaggio 1: Crea un bot Azure
</div>

1. Vai a [Create Azure Bot](https://portal.azure.com/#create/Microsoft.AzureBot)
2. Compila la scheda **Basics**:

   | Campo | Valore |
   |-------|--------|
   | **Bot handle** | Il nome del tuo bot, ad es. `openclaw-msteams` (deve essere univoco) |
   | **Subscription** | Seleziona la tua sottoscrizione Azure |
   | **Resource group** | Crea un nuovo gruppo o usane uno esistente |
   | **Pricing tier** | **Free** per sviluppo/test |
   | **Type of App** | **Single Tenant** (consigliato - vedi nota sotto) |
   | **Creation type** | **Create new Microsoft App ID** |

> **Avviso di deprecazione:** La creazione di nuovi bot multi-tenant è stata deprecata a partire dal 31-07-2025. Usa **Single Tenant** per i nuovi bot.

3. Fai clic su **Review + create** → **Create** (attendi ~1-2 minuti)

<div id="step-2-get-credentials">
  ### Passaggio 2: Recupera le credenziali
</div>

1. Vai alla tua risorsa Azure Bot → **Configuration**
2. Copia **Microsoft App ID** → questo è il tuo `appId`
3. Fai clic su **Manage Password** → vai a **App Registration**
4. In **Certificates &amp; secrets** → **New client secret** → copia il **Value** → questa è la tua `appPassword`
5. Vai a **Overview** → copia **Directory (tenant) ID** → questo è il tuo `tenantId`

<div id="step-3-configure-messaging-endpoint">
  ### Passaggio 3: Configura l&#39;endpoint di messaggistica
</div>

1. In Azure Bot → **Configuration**
2. Imposta **Messaging endpoint** sull&#39;URL del tuo webhook:
   * Produzione: `https://your-domain.com/api/messages`
   * Sviluppo locale: usa un tunnel (vedi [Sviluppo locale](#local-development-tunneling) più sotto)

<div id="step-4-enable-teams-channel">
  ### Passaggio 4: Abilita il canale Teams
</div>

1. In Azure Bot → **Channels**
2. Fai clic su **Microsoft Teams** → Configure → Save
3. Accetta le Condizioni di servizio

<div id="local-development-tunneling">
  ## Sviluppo locale (tunneling)
</div>

Teams non può accedere a `localhost`. Usa un tunnel per lo sviluppo locale:

**Opzione A: ngrok**

```bash
ngrok http 3978
# Copia l'URL https, ad esempio https://abc123.ngrok.io
# Imposta l'endpoint di messaggistica su: https://abc123.ngrok.io/api/messages
```

**Opzione B: Tailscale Funnel**

```bash
tailscale funnel 3978
# Usa l'URL del funnel di Tailscale come endpoint di messaggistica
```

<div id="teams-developer-portal-alternative">
  ## Teams Developer Portal (Alternativa)
</div>

Invece di creare manualmente un file ZIP del manifest, puoi usare il [Teams Developer Portal](https://dev.teams.microsoft.com/apps):

1. Fai clic su **+ New app**
2. Compila le informazioni di base (nome, descrizione, informazioni sullo sviluppatore)
3. Vai su **App features** → **Bot**
4. Seleziona **Enter a bot ID manually** e incolla il tuo Azure Bot App ID
5. Controlla gli scope: **Personal**, **Team**, **Group Chat**
6. Fai clic su **Distribute** → **Download app package**
7. In Teams: **Apps** → **Manage your apps** → **Upload a custom app** → seleziona il file ZIP

Questo è spesso più semplice che modificare manualmente i manifest JSON.

<div id="testing-the-bot">
  ## Test del bot
</div>

**Opzione A: Azure Web Chat (verifica prima il webhook)**

1. Nel portale di Azure → la tua risorsa Azure Bot → **Test in Web Chat**
2. Invia un messaggio: dovresti vedere una risposta
3. Questo conferma che il tuo endpoint webhook funziona prima della configurazione di Teams

**Opzione B: Teams (dopo l&#39;installazione dell&#39;app)**

1. Installa l&#39;app Teams (sideload o catalogo dell&#39;organizzazione)
2. Trova il bot in Teams e invia un messaggio diretto (DM)
3. Controlla i log del Gateway per l&#39;attività in ingresso

<div id="setup-minimal-text-only">
  ## Configurazione (minimale solo testo)
</div>

1. **Installa il plugin Microsoft Teams**
   * Da npm: `openclaw plugins install @openclaw/msteams`
   * Da una copia locale: `openclaw plugins install ./extensions/msteams`

2. **Registrazione del bot**
   * Crea un Azure Bot (vedi sopra) e annota:
     * App ID
     * Client secret (password dell&#39;app)
     * Tenant ID (single-tenant)

3. **Manifest dell&#39;app Teams**
   * Includi una voce `bot` con `botId = <App ID>`.
   * Scopes: `personal`, `team`, `groupChat`.
   * `supportsFiles: true` (necessario per la gestione dei file nello scope personale).
   * Aggiungi le autorizzazioni RSC (vedi sotto).
   * Crea le icone: `outline.png` (32x32) e `color.png` (192x192).
   * Comprimi tutti e tre i file insieme in uno zip: `manifest.json`, `outline.png`, `color.png`.

4. **Configura OpenClaw**

   ```json
   {
     "msteams": {
       "enabled": true,
       "appId": "<APP_ID>",
       "appPassword": "<APP_PASSWORD>",
       "tenantId": "<TENANT_ID>",
       "webhook": { "port": 3978, "path": "/api/messages" }
     }
   }
   ```

   Puoi anche usare variabili d&#39;ambiente invece delle chiavi di configurazione:

   * `MSTEAMS_APP_ID`
   * `MSTEAMS_APP_PASSWORD`
   * `MSTEAMS_TENANT_ID`

5. **Endpoint del bot**
   * Imposta l&#39;Azure Bot Messaging Endpoint su:
     * `https://<host>:3978/api/messages` (o il percorso/porta da te scelti).

6. **Esegui il Gateway**
   * Il canale Teams viene avviato automaticamente quando il plugin è installato ed esiste una configurazione `msteams` con le credenziali.

<div id="history-context">
  ## Contesto della cronologia
</div>

* `channels.msteams.historyLimit` controlla quanti messaggi recenti di canale/gruppo vengono inclusi nel prompt.
* Se non impostato, usa `messages.groupChat.historyLimit`. Imposta `0` per disabilitare (predefinito 50).
* La cronologia dei DM può essere limitata con `channels.msteams.dmHistoryLimit` (turni utente). Override per singolo utente: `channels.msteams.dms["<user_id>"].historyLimit`.

<div id="current-teams-rsc-permissions-manifest">
  ## Autorizzazioni RSC correnti in Teams (manifest)
</div>

Queste sono le **autorizzazioni resourceSpecific esistenti** nel manifest della nostra app Teams. Si applicano solo all&#39;interno del team o della chat in cui l&#39;app è installata.

**Per i canali (ambito team):**

* `ChannelMessage.Read.Group` (Application) - ricevere tutti i messaggi del canale senza @mention
* `ChannelMessage.Send.Group` (Application)
* `Member.Read.Group` (Application)
* `Owner.Read.Group` (Application)
* `ChannelSettings.Read.Group` (Application)
* `TeamMember.Read.Group` (Application)
* `TeamSettings.Read.Group` (Application)

**Per le chat di gruppo:**

* `ChatMessage.Read.Chat` (Application) - ricevere tutti i messaggi della chat di gruppo senza @mention

<div id="example-teams-manifest-redacted">
  ## Esempio di manifest di Teams (con dati omessi)
</div>

Esempio minimo valido con i campi obbligatori. Sostituisci ID e URL.

```json
{
  "$schema": "https://developer.microsoft.com/en-us/json-schemas/teams/v1.23/MicrosoftTeams.schema.json",
  "manifestVersion": "1.23",
  "version": "1.0.0",
  "id": "00000000-0000-0000-0000-000000000000",
  "name": { "short": "OpenClaw" },
  "developer": {
    "name": "Your Org",
    "websiteUrl": "https://example.com",
    "privacyUrl": "https://example.com/privacy",
    "termsOfUseUrl": "https://example.com/terms"
  },
  "description": { "short": "OpenClaw in Teams", "full": "OpenClaw in Teams" },
  "icons": { "outline": "outline.png", "color": "color.png" },
  "accentColor": "#5B6DEF",
  "bots": [
    {
      "botId": "11111111-1111-1111-1111-111111111111",
      "scopes": ["personal", "team", "groupChat"],
      "isNotificationOnly": false,
      "supportsCalling": false,
      "supportsVideo": false,
      "supportsFiles": true
    }
  ],
  "webApplicationInfo": {
    "id": "11111111-1111-1111-1111-111111111111"
  },
  "authorization": {
    "permissions": {
      "resourceSpecific": [
        { "name": "ChannelMessage.Read.Group", "type": "Application" },
        { "name": "ChannelMessage.Send.Group", "type": "Application" },
        { "name": "Member.Read.Group", "type": "Application" },
        { "name": "Owner.Read.Group", "type": "Application" },
        { "name": "ChannelSettings.Read.Group", "type": "Application" },
        { "name": "TeamMember.Read.Group", "type": "Application" },
        { "name": "TeamSettings.Read.Group", "type": "Application" },
        { "name": "ChatMessage.Read.Chat", "type": "Application" }
      ]
    }
  }
}
```

<div id="manifest-caveats-must-have-fields">
  ### Avvertenze sul manifesto (campi obbligatori)
</div>

* `bots[].botId` **deve** corrispondere all&#39;ID dell&#39;app bot di Azure.
* `webApplicationInfo.id` **deve** corrispondere all&#39;ID dell&#39;app bot di Azure.
* `bots[].scopes` deve includere gli ambiti che prevedi di usare (`personal`, `team`, `groupChat`).
* `bots[].supportsFiles: true` è necessario per la gestione dei file nello scope `personal`.
* `authorization.permissions.resourceSpecific` deve includere i permessi di channel read/send se vuoi il traffico sui canali.

<div id="updating-an-existing-app">
  ### Aggiornare un&#39;app esistente
</div>

Per aggiornare un&#39;app di Teams già installata (ad esempio per aggiungere autorizzazioni RSC):

1. Aggiorna il tuo `manifest.json` con le nuove impostazioni
2. **Incrementa il campo `version`** (ad es. da `1.0.0` a `1.1.0`)
3. **Comprimi di nuovo in uno zip** il manifest con le icone (`manifest.json`, `outline.png`, `color.png`)
4. Carica il nuovo file zip:
   * **Opzione A (Teams Admin Center):** Teams Admin Center → Teams apps → Manage apps → trova la tua app → Upload new version
   * **Opzione B (Sideload):** In Teams → Apps → Manage your apps → Upload a custom app
5. **Per i canali dei team:** reinstalla l&#39;app in ogni team affinché le nuove autorizzazioni abbiano effetto
6. **Chiudi completamente e riavvia Teams** (non limitarti a chiudere la finestra) per svuotare i metadati dell&#39;app memorizzati nella cache

<div id="capabilities-rsc-only-vs-graph">
  ## Capacità: solo RSC vs Graph
</div>

<div id="with-teams-rsc-only-app-installed-no-graph-api-permissions">
  ### Solo con **Teams RSC** (app installata, nessuna autorizzazione Graph API)
</div>

Funziona:

* Lettura del contenuto **testuale** dei messaggi del canale.
* Invio del contenuto **testuale** dei messaggi del canale.
* Ricezione di file allegati in **chat personali (DM)**.

Non funziona:

* **Contenuti di immagini o file** in canali/gruppi (il payload include solo uno stub HTML).
* Download degli allegati archiviati in SharePoint/OneDrive.
* Lettura della cronologia dei messaggi (oltre all&#39;evento webhook in tempo reale).

<div id="with-teams-rsc-microsoft-graph-application-permissions">
  ### Con **Teams RSC + autorizzazioni applicative di Microsoft Graph**
</div>

Abilita:

* Scaricamento dei contenuti ospitati (immagini incollate nei messaggi).
* Scaricamento dei file allegati archiviati in SharePoint/OneDrive.
* Lettura dello storico dei messaggi di canali/chat tramite Microsoft Graph.

<div id="rsc-vs-graph-api">
  ### RSC vs Graph API
</div>

| Funzionalità | Permessi RSC | Graph API |
|------------|-----------------|-----------|
| **Messaggi in tempo reale** | Sì (via webhook) | No (solo polling) |
| **Messaggi storici** | No | Sì (può interrogare la cronologia) |
| **Complessità di configurazione** | Solo manifest dell&#39;app | Richiede consenso dell&#39;amministratore + token flow |
| **Funziona offline** | No (deve essere in esecuzione) | Sì (interrogabile in qualsiasi momento) |

**In sintesi:** RSC è pensato per l&#39;ascolto in tempo reale; Graph API per l&#39;accesso alla cronologia. Per recuperare i messaggi persi mentre sei offline, ti serve Graph API con `ChannelMessage.Read.All` (richiede il consenso dell&#39;amministratore).

<div id="graph-enabled-media-history-required-for-channels">
  ## Media e cronologia tramite Graph (obbligatorio per i canali)
</div>

Se ti servono immagini/file nei **canali** o vuoi recuperare la **cronologia dei messaggi**, devi abilitare le autorizzazioni Microsoft Graph e concedere il consenso dell&#39;amministratore.

1. In **App Registration** di Entra ID (Azure AD), aggiungi le **Application permissions** di Microsoft Graph:
   * `ChannelMessage.Read.All` (allegati + cronologia dei canali)
   * `Chat.Read.All` o `ChatMessage.Read.All` (chat di gruppo)
2. **Concedi il consenso dell&#39;amministratore** per il tenant.
3. Aggiorna la **versione del manifest** dell&#39;app Teams, carica di nuovo il pacchetto e **reinstalla l&#39;app in Teams**.
4. **Chiudi completamente e riavvia Teams** per cancellare i metadati dell&#39;app in cache.

<div id="known-limitations">
  ## Limitazioni note
</div>

<div id="webhook-timeouts">
  ### Timeout dei webhook
</div>

Teams consegna i messaggi tramite webhook HTTP. Se l&#39;elaborazione richiede troppo tempo (ad esempio, a causa di risposte lente dell&#39;LLM), potresti notare:

* Timeout del Gateway
* Teams che ritenta l&#39;invio del messaggio (causando duplicati)
* Risposte scartate

OpenClaw gestisce la situazione rispondendo rapidamente e inviando le risposte in modo proattivo, ma risposte molto lente possono comunque causare problemi.

<div id="formatting">
  ### Formattazione
</div>

Il markdown di Teams è più limitato rispetto a Slack o Discord:

* La formattazione di base funziona: **grassetto**, *corsivo*, `codice`, collegamenti
* Il markdown complesso (tabelle, elenchi annidati) potrebbe non essere visualizzato correttamente
* Le Adaptive Cards sono supportate per sondaggi e per l&#39;invio arbitrario di schede (vedi sotto)

<div id="configuration">
  ## Configurazione
</div>

Impostazioni chiave (vedi `/gateway/configuration` per gli schemi condivisi tra canali):

* `channels.msteams.enabled`: abilita/disabilita il canale.
* `channels.msteams.appId`, `channels.msteams.appPassword`, `channels.msteams.tenantId`: credenziali del bot.
* `channels.msteams.webhook.port` (predefinito `3978`)
* `channels.msteams.webhook.path` (predefinito `/api/messages`)
* `channels.msteams.dmPolicy`: `pairing | allowlist | open | disabled` (predefinito: pairing)
* `channels.msteams.allowFrom`: lista di autorizzati per DM (ID oggetto AAD, UPN o nomi visualizzati). Il wizard risolve i nomi in ID durante la configurazione quando l’accesso a Graph è disponibile.
* `channels.msteams.textChunkLimit`: dimensione del blocco di testo in uscita.
* `channels.msteams.chunkMode`: `length` (predefinito) o `newline` per dividere sulle righe vuote (confini di paragrafo) prima del chunking per lunghezza.
* `channels.msteams.mediaAllowHosts`: lista di autorizzati per gli host degli allegati in ingresso (predefinito ai domini Microsoft/Teams).
* `channels.msteams.requireMention`: richiede @mention in canali/gruppi (predefinito: true).
* `channels.msteams.replyStyle`: `thread | top-level` (vedi [Stile di risposta](#reply-style-threads-vs-posts)).
* `channels.msteams.teams.<teamId>.replyStyle`: override per team.
* `channels.msteams.teams.<teamId>.requireMention`: override per team.
* `channels.msteams.teams.<teamId>.tools`: override predefiniti per team delle policy degli strumenti (`allow`/`deny`/`alsoAllow`) usati quando manca un override a livello di canale.
* `channels.msteams.teams.<teamId>.toolsBySender`: override predefiniti per team e per mittente delle policy degli strumenti (supporta il carattere jolly `"*"`).
* `channels.msteams.teams.<teamId>.channels.<conversationId>.replyStyle`: override per canale.
* `channels.msteams.teams.<teamId>.channels.<conversationId>.requireMention`: override per canale.
* `channels.msteams.teams.<teamId>.channels.<conversationId>.tools`: override per canale delle policy degli strumenti (`allow`/`deny`/`alsoAllow`).
* `channels.msteams.teams.<teamId>.channels.<conversationId>.toolsBySender`: override per canale e per mittente delle policy degli strumenti (supporta il carattere jolly `"*"`).
* `channels.msteams.sharePointSiteId`: ID del sito SharePoint per il caricamento di file nelle chat di gruppo/canali (vedi [Invio di file nelle chat di gruppo](#sending-files-in-group-chats)).

<div id="routing-sessions">
  ## Instradamento e sessioni
</div>

* Le chiavi di sessione seguono il formato standard dell’agente (consulta [/concepts/session](/it/concepts/session)):
  * I messaggi diretti condividono la sessione principale (`agent:<agentId>:<mainKey>`).
  * I messaggi di canale o di gruppo utilizzano l’ID della conversazione:
    * `agent:<agentId>:msteams:channel:<conversationId>`
    * `agent:<agentId>:msteams:group:<conversationId>`

<div id="reply-style-threads-vs-posts">
  ## Stile di risposta: Thread vs Post
</div>

Di recente Teams ha introdotto due stili di UI del canale basati sullo stesso modello di dati sottostante:

| Stile                       | Descrizione                                                     | `replyStyle` consigliato |
| --------------------------- | --------------------------------------------------------------- | ------------------------ |
| **Post** (classico)         | I messaggi appaiono come schede con le risposte in thread sotto | `thread` (predefinito)   |
| **Thread** (simile a Slack) | I messaggi scorrono in modo lineare, più simile a Slack         | `top-level`              |

**Il problema:** L&#39;API di Teams non indica quale stile di UI utilizza un canale. Se utilizzi il `replyStyle` sbagliato:

* `thread` in un canale con stile Thread → le risposte appaiono annidate in modo poco naturale
* `top-level` in un canale con stile Post → le risposte appaiono come post separati di primo livello invece che nel thread

**Soluzione:** Configura `replyStyle` per canale in base a come il canale è configurato:

```json
{
  "msteams": {
    "replyStyle": "thread",
    "teams": {
      "19:abc...@thread.tacv2": {
        "channels": {
          "19:xyz...@thread.tacv2": {
            "replyStyle": "top-level"
          }
        }
      }
    }
  }
}
```

<div id="attachments-images">
  ## Allegati e immagini
</div>

**Limitazioni attuali:**

* **DM:** Immagini e file allegati funzionano tramite le API per i file del bot di Teams.
* **Canali/gruppi:** Gli allegati risiedono nell’archiviazione M365 (SharePoint/OneDrive). Il payload del webhook include solo uno stub HTML, non i byte effettivi del file. **Sono necessarie le autorizzazioni alla Graph API** per scaricare gli allegati dei canali.

Senza le autorizzazioni Graph, i messaggi di canale con immagini verranno ricevuti solo come testo (il contenuto dell’immagine non è accessibile al bot).
Per impostazione predefinita, OpenClaw scarica i contenuti multimediali solo da host Microsoft/Teams. Puoi sovrascrivere questo comportamento con `channels.msteams.mediaAllowHosts` (usa `["*"]` per consentire qualsiasi host).

<div id="sending-files-in-group-chats">
  ## Invio di file nelle chat di gruppo
</div>

I bot possono inviare file nei DM usando il flusso FileConsentCard (integrato). Tuttavia, **inviare file nelle chat di gruppo/canali** richiede una configurazione aggiuntiva:

| Contesto | Come vengono inviati i file | Configurazione necessaria |
|---------|-----------------------------|---------------------------|
| **DM** | FileConsentCard → l&#39;utente accetta → il bot carica il file | Funziona immediatamente senza configurazione aggiuntiva |
| **Chat di gruppo/canali** | Caricamento su SharePoint → condivisione del link | Richiede `sharePointSiteId` + autorizzazioni Graph |
| **Immagini (qualsiasi contesto)** | Inline, codificate in Base64 | Funziona immediatamente senza configurazione aggiuntiva |

<div id="why-group-chats-need-sharepoint">
  ### Perché le chat di gruppo hanno bisogno di SharePoint
</div>

I bot non hanno un drive personale su OneDrive (l&#39;endpoint Graph API `/me/drive` non funziona per le identità applicative). Per inviare file nelle chat di gruppo e nei canali, il bot carica i file in un **sito SharePoint** e crea un link di condivisione.

<div id="setup">
  ### Configurazione
</div>

1. **Aggiungi le autorizzazioni Graph API** in Entra ID (Azure AD) → Registrazioni app:
   * `Sites.ReadWrite.All` (Application) - carica file su SharePoint
   * `Chat.Read.All` (Application) - opzionale, abilita i link di condivisione per singolo utente

2. **Concedi il consenso dell&#39;amministratore** per il tenant.

3. **Recupera l&#39;ID del tuo sito SharePoint:**
   ```bash
   # Tramite Graph Explorer o curl con un token valido:
   curl -H "Authorization: Bearer $TOKEN" \
     "https://graph.microsoft.com/v1.0/sites/{hostname}:/{site-path}"

   # Esempio: per un sito all'indirizzo "contoso.sharepoint.com/sites/BotFiles"
   curl -H "Authorization: Bearer $TOKEN" \
     "https://graph.microsoft.com/v1.0/sites/contoso.sharepoint.com:/sites/BotFiles"

   # La risposta include: "id": "contoso.sharepoint.com,guid1,guid2"
   ```

4. **Configura OpenClaw:**
   ```json5
   {
     channels: {
       msteams: {
         // ... altra configurazione ...
         sharePointSiteId: "contoso.sharepoint.com,guid1,guid2"
       }
     }
   }
   ```

<div id="sharing-behavior">
  ### Comportamento di condivisione
</div>

| Autorizzazione | Comportamento di condivisione |
|----------------|-------------------------------|
| Solo `Sites.ReadWrite.All` | Link di condivisione a livello di organizzazione (chiunque nell&#39;organizzazione può accedere) |
| `Sites.ReadWrite.All` + `Chat.Read.All` | Link di condivisione per singolo utente (solo i membri della chat possono accedere) |

La condivisione per singolo utente è più sicura perché solo i partecipanti alla chat possono accedere al file. Se manca l&#39;autorizzazione `Chat.Read.All`, il bot passa alla condivisione a livello di organizzazione.

<div id="fallback-behavior">
  ### Comportamento di fallback
</div>

| Scenario | Risultato |
|----------|--------|
| Chat di gruppo + file + `sharePointSiteId` configurato | Carica su SharePoint, invia un link di condivisione |
| Chat di gruppo + file + nessun `sharePointSiteId` | Tenta il caricamento su OneDrive (potrebbe non riuscire), invia solo il testo |
| Chat personale + file | Flusso FileConsentCard (funziona senza SharePoint) |
| Qualsiasi contesto + immagine | Inline codificata in Base64 (funziona senza SharePoint) |

<div id="files-stored-location">
  ### Percorso di archiviazione dei file
</div>

I file caricati vengono archiviati in una cartella `/OpenClawShared/` nella raccolta documenti predefinita del sito SharePoint configurato.

<div id="polls-adaptive-cards">
  ## Sondaggi (Adaptive Cards)
</div>

OpenClaw invia i sondaggi di Teams come Adaptive Cards (non esiste un&#39;API nativa per i sondaggi in Teams).

* CLI: `openclaw message poll --channel msteams --target conversation:&lt;id&gt; ...`
* I voti vengono registrati dal Gateway in `~/.openclaw/msteams-polls.json`.
* Il Gateway deve rimanere online per registrare i voti.
* I sondaggi non pubblicano ancora automaticamente un riepilogo dei risultati (controlla il file di archivio se necessario).

<div id="adaptive-cards-arbitrary">
  ## Adaptive Cards (arbitrary)
</div>

Invia qualsiasi JSON di Adaptive Card agli utenti o alle conversazioni di Teams usando lo strumento `message` o la CLI.

Il parametro `card` accetta un oggetto JSON di Adaptive Card. Quando `card` è specificato, il testo del messaggio è facoltativo.

**Strumento agente:**

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "user:<id>",
  "card": {
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": [{"type": "TextBlock", "text": "Hello!"}]
  }
}
```

**CLI:**

```bash
openclaw message send --channel msteams \
  --target "conversation:19:abc...@thread.tacv2" \
  --card '{"type":"AdaptiveCard","version":"1.5","body":[{"type":"TextBlock","text":"Hello!"}]}'
```

Consulta la [documentazione di Adaptive Cards](https://adaptivecards.io/) per lo schema delle schede e relativi esempi. Per i dettagli sui formati di destinazione, vedi la sezione [Formati di destinazione](#target-formats) più sotto.

<div id="target-formats">
  ## Formati di destinazione
</div>

I target di MSTeams usano prefissi per distinguere tra utenti e conversazioni:

| Tipo di destinazione   | Formato                          | Esempio                                             |
| ---------------------- | -------------------------------- | --------------------------------------------------- |
| Utente (per ID)        | `user:<aad-object-id>`           | `user:40a1a0ed-4ff2-4164-a219-55518990c197`         |
| Utente (per nome)      | `user:<display-name>`            | `user:John Smith` (richiede Graph API)              |
| Gruppo/canale          | `conversation:<conversation-id>` | `conversation:19:abc123...@thread.tacv2`            |
| Gruppo/canale (grezzo) | `<conversation-id>`              | `19:abc123...@thread.tacv2` (se contiene `@thread`) |

**Esempi di CLI:**

```bash
# Send to a user by ID
openclaw message send --channel msteams --target "user:40a1a0ed-..." --message "Hello"

# Invia a un utente tramite nome visualizzato (attiva la ricerca nell'API Graph)
openclaw message send --channel msteams --target "user:John Smith" --message "Hello"

# Send to a group chat or channel
openclaw message send --channel msteams --target "conversation:19:abc...@thread.tacv2" --message "Hello"

# Send an Adaptive Card to a conversation
openclaw message send --channel msteams --target "conversation:19:abc...@thread.tacv2" \
  --card '{"type":"AdaptiveCard","version":"1.5","body":[{"type":"TextBlock","text":"Hello"}]}'
```

**Esempi di strumenti per l’Agente:**

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "user:John Smith",
  "message": "Hello!"
}
```

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "conversation:19:abc...@thread.tacv2",
  "card": {"type": "AdaptiveCard", "version": "1.5", "body": [{"type": "TextBlock", "text": "Hello"}]}
}
```

Nota: senza il prefisso `user:`, i nomi vengono risolti per impostazione predefinita come gruppi/team. Usa sempre `user:` quando ti riferisci a persone tramite il nome visualizzato.

<div id="proactive-messaging">
  ## Messaggistica proattiva
</div>

* I messaggi proattivi sono possibili **solo dopo** che un utente ha interagito, perché da quel momento memorizziamo i riferimenti della conversazione.
* Consulta `/gateway/configuration` per `dmPolicy` e la gestione tramite lista di autorizzati.

<div id="team-and-channel-ids-common-gotcha">
  ## ID di team e canali (Insidia comune)
</div>

Il parametro di query `groupId` negli URL di Teams **NON** è l&#39;ID del team utilizzato per la configurazione. Estrai invece gli ID dal percorso dell&#39;URL:

**URL del team:**

```
https://teams.microsoft.com/l/team/19%3ABk4j...%40thread.tacv2/conversations?groupId=...
                                    └────────────────────────────┘
                                    Team ID (decodifica URL)
```

**URL del canale:**

```
https://teams.microsoft.com/l/channel/19%3A15bc...%40thread.tacv2/ChannelName?groupId=...
                                      └─────────────────────────┘
                                      Channel ID (decodifica URL)
```

**Per la configurazione:**

* Team ID = segmento di percorso che segue `/team/` (decodificato dall&#39;URL, ad esempio `19:Bk4j...@thread.tacv2`)
* Channel ID = segmento di percorso che segue `/channel/` (decodificato dall&#39;URL)
* **Ignora** il parametro di query `groupId`

<div id="private-channels">
  ## Canali privati
</div>

I bot hanno un supporto limitato nei canali privati:

| Funzionalità | Canali standard | Canali privati |
|--------------|-----------------|----------------|
| Installazione del bot | Sì | Limitata |
| Messaggi in tempo reale (webhook) | Sì | Potrebbe non funzionare |
| Autorizzazioni RSC | Sì | Può comportarsi in modo diverso |
| @mentions | Sì | Se il bot è accessibile |
| Cronologia Graph API | Sì | Sì (con autorizzazioni) |

**Soluzioni alternative se i canali privati non funzionano:**

1. Usa i canali standard per le interazioni con il bot
2. Usa i DM: gli utenti possono sempre inviare messaggi direttamente al bot
3. Usa Graph API per l&#39;accesso allo storico (richiede `ChannelMessage.Read.All`)

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

<div id="common-issues">
  ### Problemi comuni
</div>

* **Immagini non visibili nei canali:** autorizzazioni Graph o consenso dell&#39;amministratore mancanti. Reinstalla l&#39;app Teams e chiudi completamente/riapri Teams.
* **Nessuna risposta nel canale:** le menzioni sono richieste per impostazione predefinita; imposta `channels.msteams.requireMention=false` oppure configura per singolo team/canale.
* **Versione non aggiornata (Teams mostra ancora il vecchio manifest):** rimuovi e riaggiungi l&#39;app e chiudi completamente Teams per forzare l&#39;aggiornamento.
* **401 Unauthorized dal webhook:** comportamento previsto durante test manuali senza Azure JWT: significa che l&#39;endpoint è raggiungibile ma l&#39;autenticazione non è andata a buon fine. Usa Azure Web Chat per eseguire un test corretto.

<div id="manifest-upload-errors">
  ### Errori di caricamento del manifest
</div>

* **&quot;Icon file cannot be empty&quot;:** Il manifest fa riferimento a file icona da 0 byte. Crea icone PNG valide (32x32 per `outline.png`, 192x192 per `color.png`).
* **&quot;webApplicationInfo.Id already in use&quot;:** L&#39;app è ancora installata in un altro team/chat. Trovala e disinstallala prima, oppure attendi 5–10 minuti per la propagazione.
* **&quot;Something went wrong&quot; on upload:** Effettua il caricamento da https://admin.teams.microsoft.com, apri gli strumenti di sviluppo del browser (F12) → scheda Network e controlla il corpo della risposta per individuare l&#39;errore effettivo.
* **Sideload failing:** Prova &quot;Upload an app to your org&#39;s app catalog&quot; invece di &quot;Upload a custom app&quot;: spesso questo aggira le restrizioni di sideload.

<div id="rsc-permissions-not-working">
  ### Autorizzazioni RSC non funzionano
</div>

1. Verifica che `webApplicationInfo.id` corrisponda esattamente all&#39;App ID del tuo bot
2. Carica di nuovo l&#39;app e reinstallala nel team o nella chat
3. Controlla se l&#39;amministratore della tua organizzazione ha bloccato le autorizzazioni RSC
4. Conferma che stai usando lo scope corretto: `ChannelMessage.Read.Group` per i team, `ChatMessage.Read.Chat` per le chat di gruppo

<div id="references">
  ## Riferimenti
</div>

* [Create Azure Bot](https://learn.microsoft.com/en-us/azure/bot-service/bot-service-quickstart-registration) - guida alla configurazione di Azure Bot
* [Teams Developer Portal](https://dev.teams.microsoft.com/apps) - crea e gestisci le app di Teams
* [Teams app manifest schema](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema) - schema del manifesto dell&#39;app di Teams
* [Receive channel messages with RSC](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/channel-messages-with-rsc) - ricevere i messaggi di canale con RSC
* [RSC permissions reference](https://learn.microsoft.com/en-us/microsoftteams/platform/graph-api/rsc/resource-specific-consent) - riferimento alle autorizzazioni RSC
* [Teams bot file handling](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/bots-filesv4) (per canali/gruppi è richiesto Graph) - gestione dei file per i bot di Teams
* [Proactive messaging](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/send-proactive-messages) - messaggistica proattiva