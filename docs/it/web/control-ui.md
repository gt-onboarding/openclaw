---
title: UI di controllo
summary: "UI di controllo basata su browser per il Gateway (chat, nodi, configurazione)"
read_when:
  - Vuoi gestire il Gateway dal browser
  - Vuoi accedere a Tailnet senza tunnel SSH
---

<div id="control-ui-browser">
  # Control UI (browser)
</div>

La Control UI è una piccola single-page app **Vite + Lit** servita dal Gateway:

* valore predefinito: `http://&lt;host&gt;:18789/`
* prefisso opzionale: imposta `gateway.controlUi.basePath` (ad es. `/openclaw`)

Comunica **direttamente con il WebSocket del Gateway** sulla stessa porta.

<div id="quick-open-local">
  ## Apertura rapida (locale)
</div>

Se il Gateway è in esecuzione sullo stesso computer, apri:

* http://127.0.0.1:18789/ (oppure http://localhost:18789/)

Se la pagina non viene caricata, avvia prima il Gateway: `openclaw gateway`.

L&#39;autenticazione viene fornita durante l&#39;handshake WebSocket tramite:

* `connect.params.auth.token`
* `connect.params.auth.password`
  Il pannello delle impostazioni della dashboard ti permette di salvare un token; le password non vengono conservate.
  La procedura guidata di onboarding genera per impostazione predefinita un token del Gateway, quindi incollalo qui alla prima connessione.

<div id="what-it-can-do-today">
  ## Cosa può fare (oggi)
</div>

* Chattare con il modello tramite Gateway WS (`chat.history`, `chat.send`, `chat.abort`, `chat.inject`)
* Eseguire lo streaming delle chiamate agli strumenti + schede di output degli strumenti in tempo reale in Chat (eventi dell&#39;agente)
* Canali: WhatsApp/Telegram/Discord/Slack + canali dei plugin (Mattermost, ecc.) stato + login tramite QR + configurazione per canale (`channels.status`, `web.login.*`, `config.patch`)
* Istanze: elenco presenze + aggiornamento (`system-presence`)
* Sessioni: elenco + override per sessione di thinking/verbose (`sessions.list`, `sessions.patch`)
* Cron job: elenco/aggiunta/esecuzione/abilitazione/disabilitazione + cronologia delle esecuzioni (`cron.*`)
* Abilità: stato, abilita/disabilita, installa, aggiornamento delle chiavi API (`skills.*`)
* Nodi: elenco + capacità (`node.list`)
* Approvazioni di exec: modifica delle liste di autorizzati del Gateway o del nodo + richiesta della policy per `exec host=gateway/node` (`exec.approvals.*`)
* Config: visualizza/modifica `~/.openclaw/openclaw.json` (`config.get`, `config.set`)
* Config: applica + riavvia con validazione (`config.apply`) e riattiva l&#39;ultima sessione attiva
* Le scritture di configurazione includono una protezione con hash di base per evitare di sovrascrivere modifiche concorrenti
* Schema di configurazione + rendering del modulo (`config.schema`, compresi gli schemi di plugin + canali); l&#39;editor JSON grezzo rimane disponibile
* Debug: snapshot di stato/salute/modelli + log degli eventi + chiamate RPC manuali (`status`, `health`, `models.list`)
* Log: tail in tempo reale dei file di log del Gateway con filtro/esportazione (`logs.tail`)
* Update: esegue un aggiornamento del pacchetto/git + riavvia (`update.run`) con un report di riavvio

<div id="chat-behavior">
  ## Comportamento della chat
</div>

* `chat.send` è **non bloccante**: risponde immediatamente con `{ runId, status: "started" }` e la risposta viene trasmessa in streaming tramite eventi `chat`.
* Reinviare con lo stesso `idempotencyKey` restituisce `{ status: "in_flight" }` mentre è in esecuzione e `{ status: "ok" }` dopo il completamento.
* `chat.inject` aggiunge una nota dell&#39;assistente al transcript della sessione e invia un evento `chat` per aggiornamenti solo UI (nessuna esecuzione di agente, nessuna consegna al canale).
* Interruzione:
  * Fai clic su **Stop** (chiama `chat.abort`)
  * Digita `/stop` (oppure `stop|esc|abort|wait|exit|interrupt`) per interrompere out-of-band
  * `chat.abort` supporta `{ sessionKey }` (nessun `runId`) per interrompere tutte le esecuzioni attive per quella sessione

<div id="tailnet-access-recommended">
  ## Accesso tramite Tailnet (consigliato)
</div>

<div id="integrated-tailscale-serve-preferred">
  ### Tailscale Serve integrato (consigliato)
</div>

Mantieni il Gateway sull&#39;interfaccia di loopback e lascia che Tailscale Serve lo faccia da proxy HTTPS:

```bash
openclaw gateway --tailscale serve
```

Apri:

* `https://<magicdns>/` (oppure il `gateway.controlUi.basePath` che hai configurato)

Per impostazione predefinita, le richieste Serve possono essere autenticate tramite gli header di identità di Tailscale
(`tailscale-user-login`) quando `gateway.auth.allowTailscale` è `true`. OpenClaw
verifica l&#39;identità risolvendo l&#39;indirizzo `x-forwarded-for` con
`tailscale whois` e confrontandolo con l&#39;header, e accetta queste richieste solo quando
raggiungono il loopback con gli header `x-forwarded-*` di Tailscale. Imposta
`gateway.auth.allowTailscale: false` (oppure forza `gateway.auth.mode: "password"`)
se vuoi richiedere un token/password anche per il traffico Serve.

<div id="bind-to-tailnet-token">
  ### Collegare alla tailnet e al token
</div>

```bash
openclaw gateway --bind tailnet --token "$(openssl rand -hex 32)"
```

Quindi apri:

* `http://<tailscale-ip>:18789/` (oppure il `gateway.controlUi.basePath` che hai configurato)

Incolla il token nelle impostazioni dell&#39;UI (inviato come `connect.params.auth.token`).

<div id="insecure-http">
  ## HTTP non sicuro
</div>

Se apri la dashboard tramite semplice HTTP (`http://<lan-ip>` o `http://<tailscale-ip>`),
il browser viene eseguito in un **contesto non sicuro** e blocca WebCrypto. Per impostazione predefinita,
OpenClaw **blocca** le connessioni alla Control UI senza identità del dispositivo.

**Correzione consigliata:** utilizza HTTPS (Tailscale Serve) oppure apri la UI in locale:

* `https://<magicdns>/` (Serve)
* `http://127.0.0.1:18789/` (sull&#39;host del Gateway)

**Esempio di downgrade (solo token su HTTP):**

```json5
{
  gateway: {
    controlUi: { allowInsecureAuth: true },
    bind: "tailnet",
    auth: { mode: "token", token: "sostituiscimi" }
  }
}
```

Questo disabilita l&#39;identità del dispositivo e l&#39;abbinamento per la Control UI (anche su HTTPS). Utilizzalo
solo se ti fidi della rete.

Consulta [Tailscale](/it/gateway/tailscale) per indicazioni sulla configurazione di HTTPS.

<div id="building-the-ui">
  ## Build della UI
</div>

Il Gateway fornisce file statici da `dist/control-ui`. Generali con:

```bash
pnpm ui:build # installa automaticamente le dipendenze della UI alla prima esecuzione
```

Base assoluta opzionale (quando ti servono URL di asset fissi):

```bash
OPENCLAW_CONTROL_UI_BASE_PATH=/openclaw/ pnpm ui:build
```

Per lo sviluppo locale (dev server separato):

```bash
pnpm ui:dev # installa automaticamente le dipendenze dell'UI alla prima esecuzione
```

Quindi imposta la UI sull&#39;URL WS del tuo Gateway (ad esempio `ws://127.0.0.1:18789`).

<div id="debuggingtesting-dev-server-remote-gateway">
  ## Debug/testing: dev server + Gateway remoto
</div>

La Control UI è composta da file statici; il target WebSocket è configurabile e
può essere diverso dall&#39;origine HTTP. Questo è utile quando vuoi avere il dev server
Vite in locale ma il Gateway in esecuzione altrove.

1. Avvia il server di sviluppo della UI: `pnpm ui:dev`
2. Apri un URL come:

```text
http://localhost:5173/?gatewayUrl=ws://<gateway-host>:18789
```

Eventuale autenticazione una tantum (se necessaria):

```text
http://localhost:5173/?gatewayUrl=wss://<gateway-host>:18789&token=<gateway-token>
```

Note:

* `gatewayUrl` viene archiviato in localStorage dopo il caricamento della pagina e rimosso dall&#39;URL.
* `token` viene archiviato in localStorage; `password` viene mantenuta solo in memoria.
* Usa `wss://` quando il Gateway è dietro TLS (Tailscale Serve, proxy HTTPS, ecc.).

Dettagli sulla configurazione dell&#39;accesso remoto: [Accesso remoto](/it/gateway/remote).
