---
title: Tailscale
summary: "Tailscale Serve/Funnel integrato per la dashboard del Gateway"
read_when:
  - Esporre la Control UI del Gateway al di fuori di localhost
  - Automatizzare l'accesso alla dashboard del Gateway tramite tailnet o in modo pubblico
---

<div id="tailscale-gateway-dashboard">
  # Tailscale (dashboard del Gateway)
</div>

OpenClaw può configurare automaticamente **Serve** (tailnet) o **Funnel** (pubblico) di Tailscale per la
dashboard del Gateway e la porta WebSocket. In questo modo il Gateway resta in ascolto su loopback mentre
Tailscale fornisce HTTPS, instradamento e (per Serve) intestazioni di identità.

<div id="modes">
  ## Modalità
</div>

- `serve`: Solo Tailnet Serve tramite `tailscale serve`. Il Gateway rimane su `127.0.0.1`.
- `funnel`: HTTPS pubblico tramite `tailscale funnel`. OpenClaw richiede una password condivisa.
- `off`: Modalità predefinita (nessuna automazione con Tailscale).

<div id="auth">
  ## Autenticazione
</div>

Imposta `gateway.auth.mode` per controllare l'handshake:

- `token` (predefinito quando `OPENCLAW_GATEWAY_TOKEN` è impostato)
- `password` (segreto condiviso tramite `OPENCLAW_GATEWAY_PASSWORD` o configurazione)

Quando `tailscale.mode = "serve"` e `gateway.auth.allowTailscale` è `true`,
le richieste proxy di Serve valide possono autenticarsi tramite le intestazioni di identità di Tailscale
(`tailscale-user-login`) senza fornire un token/password. OpenClaw verifica
l'identità risolvendo l'indirizzo `x-forwarded-for` tramite il demone locale di Tailscale
(`tailscale whois`) e confrontandolo con l'intestazione prima di accettarla.
OpenClaw considera una richiesta come Serve solo quando arriva dalla loopback con
le intestazioni `x-forwarded-for`, `x-forwarded-proto` e `x-forwarded-host`
di Tailscale.
Per richiedere credenziali esplicite, imposta `gateway.auth.allowTailscale: false` oppure
forza `gateway.auth.mode: "password"`.

<div id="config-examples">
  ## Esempi di configurazione
</div>

<div id="tailnet-only-serve">
  ### Solo Tailnet (Serve)
</div>

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" }
  }
}
```

Apri: `https://<magicdns>/` (oppure il `gateway.controlUi.basePath` che hai configurato)


<div id="tailnet-only-bind-to-tailnet-ip">
  ### Solo Tailnet (associa all&#39;IP Tailnet)
</div>

Usa questa opzione quando vuoi che il Gateway ascolti direttamente sull&#39;IP Tailnet (senza Serve/Funnel).

```json5
{
  gateway: {
    bind: "tailnet",
    auth: { mode: "token", token: "your-token" }
  }
}
```

Connettiti da un altro dispositivo nel Tailnet:

* Control UI: `http://<tailscale-ip>:18789/`
* WebSocket: `ws://<tailscale-ip>:18789`

Nota: l&#39;indirizzo di loopback (`http://127.0.0.1:18789`) **non** funzionerà in questa modalità.


<div id="public-internet-funnel-shared-password">
  ### Internet pubblica (Funnel + password condivisa)
</div>

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password", password: "replace-me" }
  }
}
```

Usa `OPENCLAW_GATEWAY_PASSWORD` invece di salvare una password su disco.


<div id="cli-examples">
  ## Esempi di comandi CLI
</div>

```bash
openclaw gateway --tailscale serve
openclaw gateway --tailscale funnel --auth password
```


<div id="notes">
  ## Note
</div>

- Tailscale Serve/Funnel richiede che la CLI `tailscale` sia installata e che l’accesso sia stato effettuato.
- `tailscale.mode: "funnel"` si rifiuta di avviarsi a meno che la modalità di autenticazione non sia `password`, per evitare un'esposizione pubblica.
- Imposta `gateway.tailscale.resetOnExit` se vuoi che OpenClaw annulli la configurazione di
  `tailscale serve` o `tailscale funnel` in fase di arresto.
- `gateway.bind: "tailnet"` è un binding Tailnet diretto (niente HTTPS, niente Serve/Funnel).
- `gateway.bind: "auto"` preferisce la loopback; usa `tailnet` se vuoi solo Tailnet.
- Serve/Funnel espongono solo la **Gateway Control UI + WS**. I nodi si connettono tramite
  lo stesso endpoint WS del Gateway, quindi Serve può essere utilizzato per l'accesso ai nodi.

<div id="browser-control-remote-gateway-local-browser">
  ## Controllo del browser (Gateway remoto + browser locale)
</div>

Se esegui il Gateway su una macchina ma vuoi controllare un browser su un'altra macchina,
esegui un **host del nodo** sulla macchina del browser e assicurati che entrambe siano nella stessa tailnet.
Il Gateway farà da proxy delle azioni del browser verso il nodo; non è necessario alcun server di controllo separato né un Serve URL.

Evita di usare Funnel per il controllo del browser; gestisci l'abbinamento del nodo come faresti per l'accesso dell'operatore.

<div id="tailscale-prerequisites-limits">
  ## Prerequisiti e limiti di Tailscale
</div>

- Serve richiede che HTTPS sia abilitato per il tuo tailnet; la CLI ti avvisa se manca.
- Serve inietta gli header di identità Tailscale; Funnel no.
- Funnel richiede Tailscale v1.38.3+, MagicDNS, HTTPS abilitato e un attributo di nodo funnel.
- Funnel supporta solo le porte `443`, `8443` e `10000` tramite TLS.
- Funnel su macOS richiede la variante open source dell'app Tailscale.

<div id="learn-more">
  ## Per saperne di più
</div>

- Panoramica su Tailscale Serve: https://tailscale.com/kb/1312/serve
- Comando `tailscale serve`: https://tailscale.com/kb/1242/tailscale-serve
- Panoramica su Tailscale Funnel: https://tailscale.com/kb/1223/tailscale-funnel
- Comando `tailscale funnel`: https://tailscale.com/kb/1311/tailscale-funnel