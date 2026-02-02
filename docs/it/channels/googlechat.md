---
title: Googlechat
summary: "Stato del supporto, funzionalità e configurazione dell'app Google Chat"
read_when:
  - Quando lavori sulle funzionalità del canale Google Chat
---

<div id="google-chat-chat-api">
  # Google Chat (Chat API)
</div>

Stato: pronto per DM e spazi tramite webhook dell&#39;API Google Chat (solo HTTP).

<div id="quick-setup-beginner">
  ## Configurazione rapida (per principianti)
</div>

1. Crea un progetto Google Cloud e abilita la **Google Chat API**.
   * Vai a: [Google Chat API Credentials](https://console.cloud.google.com/apis/api/chat.googleapis.com/credentials)
   * Abilita l&#39;API se non è già abilitata.
2. Crea un **Service Account**:
   * Premi **Create Credentials** &gt; **Service Account**.
   * Assegnagli il nome che preferisci (ad es. `openclaw-chat`).
   * Lascia vuoti i permessi (premi **Continue**).
   * Lascia vuoti i principals con accesso (premi **Done**).
3. Crea e scarica la **JSON Key**:
   * Nell&#39;elenco dei service account, fai clic su quello che hai appena creato.
   * Vai alla scheda **Keys**.
   * Fai clic su **Add Key** &gt; **Create new key**.
   * Seleziona **JSON** e premi **Create**.
4. Salva il file JSON scaricato sull&#39;host del tuo Gateway (ad es. `~/.openclaw/googlechat-service-account.json`).
5. Crea un&#39;app Google Chat nella [Google Cloud Console Chat Configuration](https://console.cloud.google.com/apis/api/chat.googleapis.com/hangouts-chat):
   * Compila le **Application info**:
     * **App name**: (ad es. `OpenClaw`)
     * **Avatar URL**: (ad es. `https://openclaw.ai/logo.png`)
     * **Description**: (ad es. `Personal AI Assistant`)
   * Abilita le **Interactive features**.
   * In **Functionality**, seleziona **Join spaces and group conversations**.
   * In **Connection settings**, seleziona **HTTP endpoint URL**.
   * In **Triggers**, seleziona **Use a common HTTP endpoint URL for all triggers** e impostalo sull&#39;URL pubblico del tuo Gateway seguito da `/googlechat`.
     * *Suggerimento: esegui `openclaw status` per trovare l&#39;URL pubblico del tuo Gateway.*
   * In **Visibility**, seleziona **Make this Chat app available to specific people and groups in &lt;Your Domain&gt;**.
   * Inserisci il tuo indirizzo email (ad es. `user@example.com`) nella casella di testo.
   * Fai clic su **Save** in fondo.
6. **Abilita lo stato dell&#39;app**:
   * Dopo il salvataggio, **ricarica la pagina**.
   * Cerca la sezione **App status** (di solito in alto o in basso dopo il salvataggio).
   * Cambia lo stato in **Live - available to users**.
   * Fai di nuovo clic su **Save**.
7. Configura OpenClaw con il percorso del service account + webhook audience:
   * Env: `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE=/path/to/service-account.json`
   * Oppure config: `channels.googlechat.serviceAccountFile: "/path/to/service-account.json"`.
8. Imposta il tipo e il valore del webhook audience (in modo che corrispondano alla configurazione della tua app Chat).
9. Avvia il Gateway. Google Chat invierà richieste POST al tuo percorso webhook.

<div id="add-to-google-chat">
  ## Aggiungi a Google Chat
</div>

Una volta che il Gateway è in esecuzione e la tua email è stata aggiunta all&#39;elenco di visibilità:

1. Vai su [Google Chat](https://chat.google.com/).
2. Fai clic sull&#39;icona **+** (più) accanto a **Direct Messages**.
3. Nella barra di ricerca (dove di solito aggiungi le persone), digita il **nome dell&#39;app** che hai configurato nella Google Cloud Console.
   * **Nota**: il bot *non* apparirà nell&#39;elenco del &quot;Marketplace&quot; perché è un&#39;app privata. Devi cercarla per nome.
4. Seleziona il tuo bot dai risultati.
5. Fai clic su **Add** o **Chat** per avviare una conversazione 1:1.
6. Invia &quot;Hello&quot; per attivare l&#39;assistente!

<div id="public-url-webhook-only">
  ## URL pubblica (solo webhook)
</div>

I webhook di Google Chat richiedono un endpoint HTTPS pubblico. Per motivi di sicurezza, **esponi su Internet solo il percorso `/googlechat`**. Mantieni la dashboard di OpenClaw e gli altri endpoint sensibili sulla tua rete privata.

<div id="option-a-tailscale-funnel-recommended">
  ### Opzione A: Tailscale Funnel (Consigliata)
</div>

Usa Tailscale Serve per la dashboard privata e Funnel per il percorso pubblico del webhook. In questo modo mantieni `/` privato esponendo solo `/googlechat`.

1. **Verifica su quale indirizzo è in ascolto il tuo gateway:**
   ```bash
   ss -tlnp | grep 18789
   ```
   Annota l&#39;indirizzo IP (ad esempio, `127.0.0.1`, `0.0.0.0` o il tuo IP Tailscale come `100.x.x.x`).

2. **Esponi la dashboard solo alla tailnet (porta 8443):**
   ```bash
   # Se in ascolto su localhost (127.0.0.1 o 0.0.0.0):
   tailscale serve --bg --https 8443 http://127.0.0.1:18789

   # Se in ascolto solo sull'IP Tailscale (ad es., 100.106.161.80):
   tailscale serve --bg --https 8443 http://100.106.161.80:18789
   ```

3. **Esponi pubblicamente solo il percorso del webhook:**
   ```bash
   # Se in ascolto su localhost (127.0.0.1 o 0.0.0.0):
   tailscale funnel --bg --set-path /googlechat http://127.0.0.1:18789/googlechat

   # Se in ascolto solo sull'IP Tailscale (ad es., 100.106.161.80):
   tailscale funnel --bg --set-path /googlechat http://100.106.161.80:18789/googlechat
   ```

4. **Autorizza il nodo per l&#39;accesso a Funnel:**
   Se richiesto, visita l&#39;URL di autorizzazione mostrato in output per abilitare Funnel per questo nodo nella policy della tua tailnet.

5. **Verifica la configurazione:**
   ```bash
   tailscale serve status
   tailscale funnel status
   ```

Il tuo URL pubblico del webhook sarà:
`https://<node-name>.<tailnet>.ts.net/googlechat`

La tua dashboard privata rimane accessibile solo dalla tailnet:
`https://<node-name>.<tailnet>.ts.net:8443/`

Usa l&#39;URL pubblico (senza `:8443`) nella configurazione dell&#39;app Google Chat.

> Nota: questa configurazione persiste tra un riavvio e l&#39;altro. Per rimuoverla in seguito, esegui `tailscale funnel reset` e `tailscale serve reset`.

<div id="option-b-reverse-proxy-caddy">
  ### Opzione B: Reverse Proxy (Caddy)
</div>

Se utilizzi un reverse proxy come Caddy, configura il proxy solo per il percorso specifico:

```caddy
your-domain.com {
    reverse_proxy /googlechat* localhost:18789
}
```

Con questa configurazione, qualsiasi richiesta a `your-domain.com/` verrà ignorata o restituirà un errore 404, mentre `your-domain.com/googlechat` verrà instradata in modo sicuro verso OpenClaw.

<div id="option-c-cloudflare-tunnel">
  ### Opzione C: Cloudflare Tunnel
</div>

Configura le regole di ingresso del tunnel in modo che inoltrino solo il path del webhook:

* **Path**: `/googlechat` -&gt; `http://localhost:18789/googlechat`
* **Regola predefinita**: HTTP 404 (Not Found)

<div id="how-it-works">
  ## Come funziona
</div>

1. Google Chat invia richieste POST webhook al Gateway. Ogni richiesta include un’intestazione `Authorization: Bearer <token>`.
2. OpenClaw verifica il token rispetto ai valori configurati di `audienceType` + `audience`:
   * `audienceType: "app-url"` → l’audience è l’URL HTTPS del tuo webhook.
   * `audienceType: "project-number"` → l’audience è il numero del progetto Cloud.
3. I messaggi vengono instradati in base allo spazio:
   * I DM usano la chiave di sessione `agent:<agentId>:googlechat:dm:<spaceId>`.
   * Gli spazi usano la chiave di sessione `agent:<agentId>:googlechat:group:<spaceId>`.
4. L’accesso DM utilizza l’abbinamento per impostazione predefinita. I mittenti sconosciuti ricevono un codice di abbinamento; approva con:
   * `openclaw pairing approve googlechat <code>`
5. Gli spazi di gruppo richiedono per impostazione predefinita una menzione con @. Usa `botUser` se il rilevamento delle menzioni richiede il nome utente dell’app.

<div id="targets">
  ## Destinazioni
</div>

Usa questi identificatori per il recapito e la lista di autorizzati:

* Messaggi diretti: `users/<userId>` oppure `users/<email>` (gli indirizzi email sono accettati).
* Spazi: `spaces/<spaceId>`.

<div id="config-highlights">
  ## Punti chiave della configurazione
</div>

```json5
{
  channels: {
    "googlechat": {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url",
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // opzionale; aiuta il rilevamento delle menzioni
      dm: {
        policy: "pairing",
        allowFrom: ["users/1234567890", "name@example.com"]
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": {
          allow: true,
          requireMention: true,
          users: ["users/1234567890"],
          systemPrompt: "Short answers only."
        }
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20
    }
  }
}
```

Note:

* Le credenziali dell’account di servizio possono anche essere passate in linea tramite `serviceAccount` (stringa JSON).
* Il percorso predefinito del webhook è `/googlechat` se `webhookPath` non è impostato.
* Le reazioni sono disponibili tramite lo strumento `reactions` e `channels action` quando `actions.reactions` è abilitato.
* `typingIndicator` supporta `none`, `message` (valore predefinito) e `reaction` (le reazioni richiedono l’OAuth dell’utente).
* Gli allegati vengono scaricati tramite la Chat API e memorizzati nella pipeline multimediale (dimensione limitata da `mediaMaxMb`).

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

<div id="405-method-not-allowed">
  ### 405 Method Not Allowed
</div>

Se Google Cloud Logs Explorer visualizza errori del tipo:

```
status code: 405, reason phrase: HTTP error response: HTTP/1.1 405 Method Not Allowed
```

Questo significa che l&#39;handler del webhook non è stato registrato. Cause comuni:

1. **Canale non configurato**: la sezione `channels.googlechat` è assente dalla configurazione. Verifica con:
   ```bash
   openclaw config get channels.googlechat
   ```
   Se restituisce &quot;Config path not found&quot;, aggiungi la configurazione (vedi [Config highlights](#config-highlights)).

2. **Plugin non abilitato**: controlla lo stato del plugin:
   ```bash
   openclaw plugins list | grep googlechat
   ```
   Se riporta &quot;disabled&quot;, aggiungi `plugins.entries.googlechat.enabled: true` alla configurazione.

3. **Gateway non riavviato**: dopo aver aggiunto la configurazione, riavvia il Gateway:
   ```bash
   openclaw gateway restart
   ```

Verifica che il canale sia in esecuzione:

```bash
openclaw channels status
# Dovrebbe mostrare: Google Chat default: enabled, configured, ...
```

<div id="other-issues">
  ### Altri problemi
</div>

* Controlla `openclaw channels status --probe` per errori di autenticazione o per una configurazione `audience` mancante.
* Se non arrivano messaggi, verifica l&#39;URL webhook dell&#39;app Chat e le sottoscrizioni agli eventi.
* Se la limitazione basata sulle menzioni blocca le risposte, imposta `botUser` sul nome risorsa dell&#39;utente dell&#39;app e verifica `requireMention`.
* Usa `openclaw logs --follow` mentre invii un messaggio di test per vedere se le richieste raggiungono il Gateway.

Documentazione correlata:

* [Configurazione del Gateway](/it/gateway/configuration)
* [Sicurezza](/it/gateway/security)
* [Reazioni](/it/tools/reactions)