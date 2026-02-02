---
title: Gmail Pubsub
summary: "Push Gmail Pub/Sub instradato verso i webhook di OpenClaw tramite gogcli"
read_when:
  - Collegare i trigger della posta in arrivo di Gmail a OpenClaw
  - Configurare il push Pub/Sub per l'attivazione dell'agente
---

<div id="gmail-pubsub-openclaw">
  # Gmail Pub/Sub -&gt; OpenClaw
</div>

Obiettivo: watch di Gmail -&gt; push Pub/Sub -&gt; `gog gmail watch serve` -&gt; webhook di OpenClaw.

<div id="prereqs">
  ## Prerequisiti
</div>

* `gcloud` installato e autenticato ([guida all&#39;installazione](https://docs.cloud.google.com/sdk/docs/install-sdk)).
* `gog` (gogcli) installato e autorizzato per l&#39;account Gmail ([gogcli.sh](https://gogcli.sh/)).
* Hook di OpenClaw abilitati (vedi [Webhooks](/it/automation/webhook)).
* `tailscale` autenticato ([tailscale.com](https://tailscale.com/)). La configurazione supportata utilizza Tailscale Funnel per l&#39;endpoint HTTPS pubblico.
  Altri servizi di tunneling possono funzionare, ma sono fai-da-te/non supportati e richiedono una configurazione manuale.
  Al momento supportiamo solo Tailscale.

Esempio di configurazione di hook (abilita la mappatura predefinita per Gmail):

```json5
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    path: "/hooks",
    presets: ["gmail"]
  }
}
```

Per recapitare il riepilogo Gmail a un&#39;interfaccia di chat, sovrascrivi il preset con una mappatura che imposti `deliver` e, facoltativamente, `channel`/`to`:

```json5
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    presets: ["gmail"],
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate:
          "Nuova email da {{messages[0].from}}\nOggetto: {{messages[0].subject}}\n{{messages[0].snippet}}\n{{messages[0].body}}",
        model: "openai/gpt-5.2-mini",
        deliver: true,
        channel: "last"
        // to: "+15551234567"
      }
    ]
  }
}
```

Se vuoi usare un canale fisso, imposta `channel` + `to`. Altrimenti `channel: "last"`
usa l&#39;ultimo percorso di recapito (con fallback a WhatsApp).

Per forzare un modello più economico per le esecuzioni Gmail, imposta `model` nel mapping
(`provider/model` o alias). Se configuri `agents.defaults.models`, includilo anche lì.

Per impostare un modello predefinito e un livello di thinking specifici per gli hook Gmail, aggiungi
`hooks.gmail.model` / `hooks.gmail.thinking` nella tua configurazione:

```json5
{
  hooks: {
    gmail: {
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off"
    }
  }
}
```

Note:

* Il `model`/`thinking` per singolo hook nella mappatura continua a sovrascrivere questi valori predefiniti.
* Ordine di fallback: `hooks.gmail.model` → `agents.defaults.model.fallbacks` → primario (auth/rate-limit/timeouts).
* Se `agents.defaults.models` è impostato, il modello Gmail deve essere nella lista di autorizzati.
* Il contenuto dell&#39;hook Gmail è racchiuso, per impostazione predefinita, entro barriere di sicurezza per contenuti esterni.
  Per disabilitare (operazione pericolosa), imposta `hooks.gmail.allowUnsafeExternalContent: true`.

Per personalizzare ulteriormente la gestione del payload, aggiungi `hooks.mappings` o un modulo di trasformazione JS/TS
in `hooks.transformsDir` (vedi [Webhooks](/it/automation/webhook)).

<div id="wizard-recommended">
  ## Wizard (consigliato)
</div>

Usa l&#39;assistente di OpenClaw per configurare il tutto (installa le dipendenze su macOS tramite Homebrew/brew):

```bash
openclaw webhooks gmail setup \
  --account openclaw@gmail.com
```

Impostazioni predefinite:

* Usa Tailscale Funnel per l&#39;endpoint pubblico di push.
* Scrive la configurazione `hooks.gmail` per `openclaw webhooks gmail run`.
* Abilita il preset dell&#39;hook Gmail (`hooks.presets: ["gmail"]`).

Nota sul path: quando `tailscale.mode` è abilitata, OpenClaw imposta automaticamente
`hooks.gmail.serve.path` su `/` e mantiene il path pubblico in
`hooks.gmail.tailscale.path` (predefinito `/gmail-pubsub`) perché Tailscale
rimuove il prefisso di path impostato prima di effettuare il proxy.
Se hai bisogno che il backend riceva il path con prefisso, imposta
`hooks.gmail.tailscale.target` (o `--tailscale-target`) su un URL completo come
`http://127.0.0.1:8788/gmail-pubsub` e fai corrispondere `hooks.gmail.serve.path`.

Vuoi un endpoint personalizzato? Usa `--push-endpoint <url>` oppure `--tailscale off`.

Nota sulla piattaforma: su macOS la procedura guidata installa `gcloud`, `gogcli` e `tailscale`
tramite Homebrew; su Linux installali prima manualmente.

Avvio automatico del Gateway (consigliato):

* Quando `hooks.enabled=true` e `hooks.gmail.account` è impostato, il Gateway avvia
  `gog gmail watch serve` all&#39;avvio e rinnova automaticamente la watch.
* Imposta `OPENCLAW_SKIP_GMAIL_WATCHER=1` per disattivarlo (utile se esegui tu stesso il demone).
* Non eseguire il demone manuale contemporaneamente, altrimenti otterrai
  `listen tcp 127.0.0.1:8788: bind: address already in use`.

Demone manuale (avvia `gog gmail watch serve` + rinnovo automatico):

```bash
openclaw webhooks gmail run
```

<div id="one-time-setup">
  ## Configurazione iniziale (una tantum)
</div>

1. Seleziona il progetto GCP **a cui appartiene il client OAuth** utilizzato da `gog`.

```bash
gcloud auth login
gcloud config set project <project-id>
```

Nota: la funzione di monitoraggio di Gmail richiede che il topic Pub/Sub risieda nello stesso progetto del client OAuth.

2. Abilita le API:

```bash
gcloud services enable gmail.googleapis.com pubsub.googleapis.com
```

3. Crea un topic:

```bash
gcloud pubsub topics create gog-gmail-watch
```

4. Consenti a Gmail di pubblicare in push:

```bash
gcloud pubsub topics add-iam-policy-binding gog-gmail-watch \
  --member=serviceAccount:gmail-api-push@system.gserviceaccount.com \
  --role=roles/pubsub.publisher
```

<div id="start-the-watch">
  ## Avvia il monitoraggio
</div>

```bash
gog gmail watch start \
  --account openclaw@gmail.com \
  --label INBOX \
  --topic projects/<project-id>/topics/gog-gmail-watch
```

Salva l&#39;`history_id` dall&#39;output (a fini di debug).

<div id="run-the-push-handler">
  ## Esegui l&#39;handler di push
</div>

Esempio locale (autenticazione tramite token condiviso):

```bash
gog gmail watch serve \
  --account openclaw@gmail.com \
  --bind 127.0.0.1 \
  --port 8788 \
  --path /gmail-pubsub \
  --token <shared> \
  --hook-url http://127.0.0.1:18789/hooks/gmail \
  --hook-token OPENCLAW_HOOK_TOKEN \
  --include-body \
  --max-bytes 20000
```

Note:

* `--token` protegge l&#39;endpoint di push (`x-gog-token` o `?token=`).
* `--hook-url` punta all&#39;endpoint OpenClaw `/hooks/gmail` (mappato; esecuzione isolata + riepilogo verso il flusso principale).
* `--include-body` e `--max-bytes` controllano lo snippet di body inviato a OpenClaw.

Consigliato: `openclaw webhooks gmail run` gestisce lo stesso flusso e rinnova automaticamente la watch di Gmail.

<div id="expose-the-handler-advanced-unsupported">
  ## Esporre l&#39;handler (avanzato, non supportato)
</div>

Se hai bisogno di un tunnel non Tailscale, configurane uno manualmente e usa l&#39;URL pubblico nell&#39;abbonamento push (non supportato, senza alcun meccanismo di protezione):

```bash
cloudflared tunnel --url http://127.0.0.1:8788 --no-autoupdate
```

Utilizza l&#39;URL generato come endpoint push:

```bash
gcloud pubsub subscriptions create gog-gmail-watch-push \
  --topic gog-gmail-watch \
  --push-endpoint "https://<public-url>/gmail-pubsub?token=<shared>"
```

In produzione, utilizza un endpoint HTTPS stabile e configura il JWT OIDC per Pub/Sub, quindi esegui:

```bash
gog gmail watch serve --verify-oidc --oidc-email <svc@...>
```

<div id="test">
  ## Test
</div>

Invia un messaggio alla casella di posta in arrivo monitorata:

```bash
gog gmail send \
  --account openclaw@gmail.com \
  --to openclaw@gmail.com \
  --subject "watch test" \
  --body "ping"
```

Verifica lo stato del monitoraggio e la cronologia:

```bash
gog gmail watch status --account openclaw@gmail.com
gog gmail history --account openclaw@gmail.com --since <historyId>
```

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

* `Invalid topicName`: progetto non corrispondente (topic non presente nel progetto del client OAuth).
* `User not authorized`: manca `roles/pubsub.publisher` sul topic.
* Messaggi vuoti: il push Gmail fornisce solo `historyId`; recuperali tramite `gog gmail history`.

<div id="cleanup">
  ## Pulizia
</div>

```bash
gog gmail watch stop --account openclaw@gmail.com
gcloud pubsub subscriptions delete gog-gmail-watch-push
gcloud pubsub topics delete gog-gmail-watch
```
