---
title: Mattermost
summary: "Configurazione del bot Mattermost e di OpenClaw"
read_when:
  - Configurazione di Mattermost
  - Diagnostica del routing di Mattermost
---

<div id="mattermost-plugin">
  # Mattermost (plugin)
</div>

Stato: supportato tramite plugin (token del bot + eventi WebSocket). Sono supportati canali, gruppi e DM.
Mattermost è una piattaforma di messaggistica per team self-hosted; per dettagli sul prodotto e download, consulta il sito ufficiale
[mattermost.com](https://mattermost.com).

<div id="plugin-required">
  ## Plugin necessario
</div>

Mattermost viene fornito come plugin e non è incluso nell&#39;installazione di base.

Installalo tramite CLI (registry npm):

```bash
openclaw plugins install @openclaw/mattermost
```

Checkout locale (quando esegui da un repository Git):

```bash
openclaw plugins install ./extensions/mattermost
```

Se scegli Mattermost durante la procedura di configurazione/onboarding e viene rilevato un checkout Git,
OpenClaw proporrà automaticamente il percorso locale di installazione.

Dettagli: [Plugin](/it/plugin)

<div id="quick-setup">
  ## Configurazione rapida
</div>

1. Installa il plugin per Mattermost.
2. Crea un account bot di Mattermost e copia il **token bot**.
3. Copia l’**URL di base** di Mattermost (ad es. `https://chat.example.com`).
4. Configura OpenClaw e avvia il Gateway.

Configurazione minima:

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing"
    }
  }
}
```

<div id="environment-variables-default-account">
  ## Variabili d&#39;ambiente (account predefinito)
</div>

Impostale sull&#39;host del Gateway se preferisci usare variabili d&#39;ambiente:

* `MATTERMOST_BOT_TOKEN=...`
* `MATTERMOST_URL=https://chat.example.com`

Le variabili d&#39;ambiente si applicano solo all&#39;account **predefinito** (`default`). Gli altri account devono usare i valori di configurazione.

<div id="chat-modes">
  ## Modalità di chat
</div>

Mattermost risponde automaticamente ai messaggi diretti (DM). Il comportamento nei canali è controllato da `chatmode`:

* `oncall` (predefinito): risponde solo quando viene menzionato con @ nei canali.
* `onmessage`: risponde a tutti i messaggi del canale.
* `onchar`: risponde quando un messaggio inizia con un prefisso di attivazione.

Esempio di configurazione:

```json5
{
  channels: {
    mattermost: {
      chatmode: "onchar",
      oncharPrefixes: [">", "!"]
    }
  }
}
```

Note:

* `onchar` risponde ancora alle menzioni esplicite con @.
* `channels.mattermost.requireMention` viene ancora applicato per le configurazioni legacy, ma si preferisce `chatmode`.

<div id="access-control-dms">
  ## Controllo degli accessi (DM)
</div>

* Predefinito: `channels.mattermost.dmPolicy = "pairing"` (i mittenti sconosciuti ricevono un codice di abbinamento).
* Per approvare:
  * `openclaw pairing list mattermost`
  * `openclaw pairing approve mattermost <CODE>`
* DM pubblici: `channels.mattermost.dmPolicy="open"` (imposta la policy open, che consente di accettare messaggi da chiunque) insieme a `channels.mattermost.allowFrom=["*"]`.

<div id="channels-groups">
  ## Canali (gruppi)
</div>

* Impostazione predefinita: `channels.mattermost.groupPolicy = "allowlist"` (limitato alle menzioni).
* Autorizza i mittenti tramite `channels.mattermost.groupAllowFrom` (ID utente o `@username`).
* Canali aperti: `channels.mattermost.groupPolicy="open"` (limitato alle menzioni; l&#39;impostazione &quot;open&quot; consente l&#39;accettazione di messaggi senza restrizioni da qualsiasi utente).

<div id="targets-for-outbound-delivery">
  ## Destinazioni per la consegna in uscita
</div>

Usa questi formati di destinazione con `openclaw message send` o con cron/webhook:

* `channel:<id>` per un canale
* `user:<id>` per un messaggio diretto (DM)
* `@username` per un messaggio diretto (DM, risolto tramite la Mattermost API)

Gli ID senza prefisso vengono considerati canali.

<div id="multi-account">
  ## Multi-account
</div>

Mattermost supporta più account nella sezione `channels.mattermost.accounts`:

```json5
{
  channels: {
    mattermost: {
      accounts: {
        default: { name: "Primary", botToken: "mm-token", baseUrl: "https://chat.example.com" },
        alerts: { name: "Alerts", botToken: "mm-token-2", baseUrl: "https://alerts.example.com" }
      }
    }
  }
}
```

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

* Nessuna risposta nei canali: assicurati che il bot sia nel canale e menzionalo (oncall), usa un prefisso di attivazione (onchar) oppure imposta `chatmode: "onmessage"`.
* Errori di autenticazione: controlla il token del bot, l’URL di base e che l’account sia abilitato.
* Problemi con più account: le variabili di ambiente si applicano solo all’account `default`.