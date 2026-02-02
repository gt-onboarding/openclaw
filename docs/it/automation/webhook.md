---
title: Webhook
summary: "Webhook di ingresso per la riattivazione e le esecuzioni isolate degli agenti"
read_when:
  - Aggiunta o modifica degli endpoint webhook
  - Collegamento di sistemi esterni a OpenClaw
---

<div id="webhooks">
  # Webhook
</div>

Gateway può esporre un endpoint HTTP webhook minimale per attivazioni esterne.

<div id="enable">
  ## Abilitare
</div>

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks"
  }
}
```

Note:

* `hooks.token` è richiesto quando `hooks.enabled=true`.
* Il valore predefinito di `hooks.path` è `/hooks`.

<div id="auth">
  ## Autenticazione
</div>

Ogni richiesta deve includere il token dell&#39;hook. È preferibile usare gli header:

* `Authorization: Bearer <token>` (consigliato)
* `x-openclaw-token: <token>`
* `?token=<token>` (deprecato; genera un avviso nei log e verrà rimosso in una futura major release)

<div id="endpoints">
  ## Endpoint
</div>

<div id="post-hookswake">
  ### `POST /hooks/wake`
</div>

Corpo della richiesta:

```json
{ "text": "System line", "mode": "now" }
```

* `text` **required** (string): La descrizione dell&#39;evento (ad es. &quot;Nuova email ricevuta&quot;).
* `mode` facoltativo (`now` | `next-heartbeat`): Se deve attivare un heartbeat immediato (predefinito `now`) o attendere il prossimo controllo periodico.

Effetto:

* Mette in coda un evento di sistema per la sessione **principale**
* Se `mode=now`, attiva un heartbeat immediato

<div id="post-hooksagent">
  ### `POST /hooks/agent`
</div>

Corpo della richiesta:

```json
{
  "message": "Run this",
  "name": "Email",
  "sessionKey": "hook:email:msg-123",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+15551234567",
  "model": "openai/gpt-5.2-mini",
  "thinking": "low",
  "timeoutSeconds": 120
}
```

* `message` **required** (string): Il prompt o messaggio che l&#39;agente deve elaborare.
* `name` optional (string): Nome descrittivo per l&#39;hook (ad es. &quot;GitHub&quot;), usato come prefisso nei riepiloghi di sessione.
* `sessionKey` optional (string): La chiave usata per identificare la sessione dell&#39;agente. Valore predefinito: un `hook:<uuid>` casuale. L&#39;utilizzo di una chiave coerente permette una conversazione multi-turno all&#39;interno del contesto dell&#39;hook.
* `wakeMode` optional (`now` | `next-heartbeat`): Se attivare un heartbeat immediato (predefinito `now`) o attendere il successivo controllo periodico.
* `deliver` optional (boolean): Se `true`, la risposta dell&#39;agente sarà inviata al canale di messaggistica. Valore predefinito: `true`. Le risposte che sono solo conferme di heartbeat vengono automaticamente ignorate.
* `channel` optional (string): Il canale di messaggistica per la consegna. Uno tra: `last`, `whatsapp`, `telegram`, `discord`, `slack`, `mattermost` (plugin), `signal`, `imessage`, `msteams`. Valore predefinito: `last`.
* `to` optional (string): L&#39;identificatore del destinatario per il canale (ad es. numero di telefono per WhatsApp/Signal, ID chat per Telegram, ID canale per Discord/Slack/Mattermost (plugin), ID conversazione per MS Teams). Valore predefinito: l&#39;ultimo destinatario nella sessione principale.
* `model` optional (string): Override del modello (ad es. `anthropic/claude-3-5-sonnet` o un alias). Deve essere presente nell&#39;elenco dei modelli consentiti se l&#39;uso è ristretto.
* `thinking` optional (string): Override del livello di thinking (ad es. `low`, `medium`, `high`).
* `timeoutSeconds` optional (number): Durata massima, in secondi, dell&#39;esecuzione dell&#39;agente.

Effetto:

* Esegue un turno di agente **isolato** (con una propria chiave di sessione)
* Inserisce sempre un riepilogo nella sessione **principale**
* Se `wakeMode=now`, attiva un heartbeat immediato

<div id="post-hooksname-mapped">
  ### `POST /hooks/<name>` (mappato)
</div>

I nomi degli hook personalizzati vengono risolti tramite `hooks.mappings` (vedi configurazione). Una mappatura può
convertire payload arbitrari in azioni `wake` o `agent`, con template opzionali o
trasformazioni di codice.

Opzioni di mappatura (riepilogo):

* `hooks.presets: ["gmail"]` abilita la mappatura Gmail integrata.
* `hooks.mappings` ti consente di definire `match`, `action` e template nella configurazione.
* `hooks.transformsDir` + `transform.module` carica un modulo JS/TS per la logica personalizzata.
* Usa `match.source` per mantenere un endpoint di ingestione generico (instradamento basato sul payload).
* Le trasformazioni TS richiedono un loader TS (ad es. `bun` o `tsx`) o un `.js` precompilato a runtime.
* Imposta `deliver: true` + `channel`/`to` sulle mappature per instradare le risposte verso un canale/interfaccia di chat
  (`channel` predefinito è `last` e, in mancanza, viene usato WhatsApp).
* `allowUnsafeExternalContent: true` disabilita il wrapper di sicurezza per i contenuti esterni per quell’hook
  (pericoloso; solo per fonti interne attendibili).
* `openclaw webhooks gmail setup` scrive la configurazione `hooks.gmail` per `openclaw webhooks gmail run`.
  Vedi [Gmail Pub/Sub](/it/automation/gmail-pubsub) per l’intero flusso di watch per Gmail.

<div id="responses">
  ## Risposte
</div>

* `200` per `/hooks/wake`
* `202` per `/hooks/agent` (esecuzione asincrona avviata)
* `401` in caso di errore di autenticazione
* `400` in caso di payload non valido
* `413` in caso di payload di dimensioni eccessive

<div id="examples">
  ## Esempi
</div>

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"Nuova email ricevuta","mode":"now"}'
```

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","wakeMode":"next-heartbeat"}'
```

<div id="use-a-different-model">
  ### Usa un modello diverso
</div>

Aggiungi `model` al payload dell&#39;agente (o mapping) per sovrascrivere il modello in quella esecuzione:

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

Se imponi l’uso di `agents.defaults.models`, assicurati che il modello di override sia incluso in quell’elenco.

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

<div id="security">
  ## Sicurezza
</div>

* Mantieni gli endpoint degli hook dietro loopback, tailnet o un reverse proxy attendibile.
* Usa un token dedicato per gli hook; non riutilizzare i token di autenticazione del Gateway.
* Evita di includere payload non elaborati con dati sensibili nei log dei webhook.
* I payload degli hook sono considerati non attendibili e, per impostazione predefinita, vengono racchiusi entro limiti di sicurezza.
  Se devi disabilitare questo comportamento per uno specifico hook, imposta `allowUnsafeExternalContent: true`
  nel mapping di quell&#39;hook (operazione pericolosa).