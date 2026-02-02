---
title: Twitch
summary: "Configurazione del bot di chat di Twitch"
read_when:
  - Configurazione dell'integrazione della chat di Twitch per OpenClaw
---

<div id="twitch-plugin">
  # Twitch (plugin)
</div>

Supporto della chat di Twitch tramite connessione IRC. OpenClaw si connette come utente Twitch (account bot) per inviare e ricevere messaggi nei canali.

<div id="plugin-required">
  ## Plugin richiesto
</div>

Twitch è distribuito come plugin e non è incluso nell&#39;installazione principale.

Installalo tramite CLI (registry npm):

```bash
openclaw plugins install @openclaw/twitch
```

Clone locale (quando esegui da un repository Git):

```bash
openclaw plugins install ./extensions/twitch
```

Dettagli: [Plugin](/it/plugin)

<div id="quick-setup-beginner">
  ## Configurazione rapida (per principianti)
</div>

1. Crea un account Twitch dedicato al bot (o usa un account esistente).
2. Genera le credenziali: [Twitch Token Generator](https://twitchtokengenerator.com/)
   * Seleziona **Bot Token**
   * Verifica che gli scope `chat:read` e `chat:write` siano selezionati
   * Copia il **Client ID** e l&#39;**Access Token**
3. Trova il tuo ID utente Twitch: https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/
4. Configura il token:
   * Env: `OPENCLAW_TWITCH_ACCESS_TOKEN=...` (solo per l&#39;account predefinito)
   * Oppure config: `channels.twitch.accessToken`
   * Se entrambi sono impostati, la config ha la precedenza (env come fallback solo per l&#39;account predefinito).
5. Avvia il Gateway.

**⚠️ Importante:** Aggiungi il controllo degli accessi (`allowFrom` o `allowedRoles`) per impedire agli utenti non autorizzati di attivare il bot. `requireMention` è `true` per impostazione predefinita.

Configurazione minima:

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",              // Bot's Twitch account
      accessToken: "oauth:abc123...",    // OAuth Access Token (or use OPENCLAW_TWITCH_ACCESS_TOKEN env var)
      clientId: "xyz789...",             // Client ID from Token Generator
      channel: "vevisk",                 // Which Twitch channel's chat to join (required)
      allowFrom: ["123456789"]           // (consigliato) Solo il tuo ID utente Twitch - ottienilo da https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/
    }
  }
}
```

<div id="what-it-is">
  ## Che cos&#39;è
</div>

* Un canale Twitch di proprietà del Gateway.
* Routing deterministico: le risposte vengono sempre recapitate a Twitch.
* Ogni account viene mappato su una chiave di sessione isolata `agent:<agentId>:twitch:<accountName>`.
* `username` è l&#39;account del bot (che effettua l&#39;autenticazione), `channel` è la chat room a cui connettersi.

<div id="setup-detailed">
  ## Configurazione (dettagliata)
</div>

<div id="generate-credentials">
  ### Genera le credenziali
</div>

Usa [Twitch Token Generator](https://twitchtokengenerator.com/):

* Seleziona **Bot Token**
* Verifica che gli scope `chat:read` e `chat:write` siano selezionati
* Copia il **Client ID** e l&#39;**Access Token**

Non è necessaria alcuna registrazione manuale dell&#39;applicazione. I token scadono dopo alcune ore.

<div id="configure-the-bot">
  ### Configura il bot
</div>

**Variabile d&#39;ambiente (solo account predefinito):**

```bash
OPENCLAW_TWITCH_ACCESS_TOKEN=oauth:abc123...
```

**Oppure tramite config:**

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk"
    }
  }
}
```

Se sono impostate sia la variabile d&#39;ambiente che la configurazione, prevale la configurazione.

<div id="access-control-recommended">
  ### Controllo accessi (consigliato)
</div>

```json5
{
  channels: {
    twitch: {
      allowFrom: ["123456789"],       // (consigliato) Solo il tuo ID utente Twitch
      allowedRoles: ["moderator"]     // Or restrict to roles
    }
  }
}
```

**Ruoli disponibili:** `"moderator"`, `"owner"`, `"vip"`, `"subscriber"`, `"all"`.

**Perché gli ID utente?** I nomi utente possono cambiare, consentendo a qualcuno di spacciarsi per un altro utente. Gli ID utente sono permanenti.

Trova il tuo ID utente Twitch: https://www.streamweasels.com/tools/convert-twitch-username-%20to-user-id/ (Converti il tuo nome utente Twitch in ID)

<div id="token-refresh-optional">
  ## Rinnovo del token (opzionale)
</div>

I token generati con [Twitch Token Generator](https://twitchtokengenerator.com/) non possono essere rinnovati automaticamente: devi rigenerarli quando scadono.

Per abilitare il rinnovo automatico dei token, crea una tua applicazione Twitch nella [Twitch Developer Console](https://dev.twitch.tv/console) e aggiungila alla configurazione:

```json5
{
  channels: {
    twitch: {
      clientSecret: "your_client_secret",
      refreshToken: "your_refresh_token"
    }
  }
}
```

Il bot esegue automaticamente il refresh dei token prima della scadenza e registra nel log gli eventi di refresh.

<div id="multi-account-support">
  ## Supporto multi-account
</div>

Usa `channels.twitch.accounts` con token specifici per ogni account. Consulta [`gateway/configuration`](/it/gateway/configuration) per lo schema condiviso.

Esempio (un account bot in due canali):

```json5
{
  channels: {
    twitch: {
      accounts: {
        channel1: {
          username: "openclaw",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "vevisk"
        },
        channel2: {
          username: "openclaw",
          accessToken: "oauth:def456...",
          clientId: "uvw012...",
          channel: "secondchannel"
        }
      }
    }
  }
}
```

**Nota:** Ogni account richiede il proprio token (uno per canale).

<div id="access-control">
  ## Controllo degli accessi
</div>

<div id="role-based-restrictions">
  ### Limitazioni in base al ruolo
</div>

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowedRoles: ["moderator", "vip"]
        }
      }
    }
  }
}
```

<div id="allowlist-by-user-id-most-secure">
  ### Lista di autorizzati per ID utente (opzione più sicura)
</div>

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowFrom: ["123456789", "987654321"]
        }
      }
    }
  }
}
```

<div id="combined-allowlist-roles">
  ### Lista di autorizzati + ruoli combinati
</div>

Gli utenti presenti in `allowFrom` ignorano i controlli sui ruoli:

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowFrom: ["123456789"],
          allowedRoles: ["moderator"]
        }
      }
    }
  }
}
```

<div id="disable-mention-requirement">
  ### Disattivare l&#39;obbligo di @mention
</div>

Per impostazione predefinita, `requireMention` è `true`. Per disattivare questa opzione e rispondere a tutti i messaggi:

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          requireMention: false
        }
      }
    }
  }
}
```

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

Per prima cosa, esegui i comandi diagnostici:

```bash
openclaw doctor
openclaw channels status --probe
```

<div id="bot-doesnt-respond-to-messages">
  ### Il bot non risponde ai messaggi
</div>

**Verifica i controlli di accesso:** Imposta temporaneamente `allowedRoles: ["all"]` per testare.

**Verifica che il bot sia nel canale:** Il bot deve essere presente nel canale specificato in `channel`.

<div id="token-issues">
  ### Problemi con i token
</div>

**Errori &quot;Failed to connect&quot; o di autenticazione:**

* Verifica che `accessToken` contenga il valore del token di accesso OAuth (in genere inizia con il prefisso `oauth:`)
* Controlla che il token abbia gli scope `chat:read` e `chat:write`
* Se utilizzi il refresh del token, verifica che `clientSecret` e `refreshToken` siano impostati

<div id="token-refresh-not-working">
  ### L&#39;aggiornamento del token non funziona
</div>

**Controlla i log per gli eventi di aggiornamento:**

```
Using env token source for mybot
Access token refreshed for user 123456 (expires in 14400s)
```

Se vedi &quot;token refresh disabled (no refresh token)&quot;:

* Assicurati che `clientSecret` sia impostato
* Assicurati che `refreshToken` sia impostato

<div id="config">
  ## Configurazione
</div>

**Configurazione dell&#39;account:**

* `username` - Nome utente del bot
* `accessToken` - Token di accesso OAuth con `chat:read` e `chat:write`
* `clientId` - Client ID di Twitch (dal Token Generator o dalla tua app)
* `channel` - Canale a cui unirsi (obbligatorio)
* `enabled` - Abilita questo account (predefinito: `true`)
* `clientSecret` - Opzionale: per il refresh automatico del token
* `refreshToken` - Opzionale: per il refresh automatico del token
* `expiresIn` - Scadenza del token in secondi
* `obtainmentTimestamp` - Timestamp di generazione del token
* `allowFrom` - Lista di autorizzati per gli ID utente
* `allowedRoles` - Controllo di accesso basato sui ruoli (`"moderator" | "owner" | "vip" | "subscriber" | "all"`)
* `requireMention` - Richiede una @mention (predefinito: `true`)

**Opzioni del provider:**

* `channels.twitch.enabled` - Abilita/disabilita l&#39;avvio del canale
* `channels.twitch.username` - Nome utente del bot (configurazione semplificata a singolo account)
* `channels.twitch.accessToken` - Token di accesso OAuth (configurazione semplificata a singolo account)
* `channels.twitch.clientId` - Client ID di Twitch (configurazione semplificata a singolo account)
* `channels.twitch.channel` - Canale a cui unirsi (configurazione semplificata a singolo account)
* `channels.twitch.accounts.<accountName>` - Configurazione multi-account (tutti i campi account indicati sopra)

Esempio completo:

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
      clientSecret: "secret123...",
      refreshToken: "refresh456...",
      allowFrom: ["123456789"],
      allowedRoles: ["moderator", "vip"],
      accounts: {
        default: {
          username: "mybot",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "your_channel",
          enabled: true,
          clientSecret: "secret123...",
          refreshToken: "refresh456...",
          expiresIn: 14400,
          obtainmentTimestamp: 1706092800000,
          allowFrom: ["123456789", "987654321"],
          allowedRoles: ["moderator"]
        }
      }
    }
  }
}
```

<div id="tool-actions">
  ## Azioni dello strumento
</div>

L&#39;agente può chiamare `twitch` con la seguente azione:

* `send` - Invia un messaggio a un canale

Esempio:

```json5
{
  "action": "twitch",
  "params": {
    "message": "Hello Twitch!",
    "to": "#mychannel"
  }
}
```

<div id="safety-ops">
  ## Sicurezza e operatività
</div>

* **Tratta i token come password** - Non eseguire mai il commit dei token su git
* **Usa il refresh automatico dei token** per bot a esecuzione prolungata
* **Usa liste di autorizzati basate sugli ID utente** invece dei nomi utente per il controllo degli accessi
* **Monitora i log** per eventi di refresh dei token e per lo stato della connessione
* **Limita al minimo l&#39;ambito dei token** - Richiedi solo `chat:read` e `chat:write`
* **Se rimani bloccato**: riavvia il Gateway dopo aver verificato che nessun altro processo detiene la sessione

<div id="limits">
  ## Limiti
</div>

* **500 caratteri** per messaggio (suddiviso automaticamente sui confini delle parole)
* Il Markdown viene rimosso prima della suddivisione in blocchi
* Nessun rate limiting esplicito (usa i rate limit integrati di Twitch)