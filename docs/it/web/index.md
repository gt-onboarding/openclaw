---
title: Web
summary: "Interfacce web del Gateway: Control UI, modalità di bind e sicurezza"
read_when:
  - Vuoi accedere al Gateway tramite Tailscale
  - Vuoi usare la Control UI nel browser e modificare la configurazione
---

<div id="web-gateway">
  # Web (Gateway)
</div>

Il Gateway espone una piccola **Control UI nel browser** (Vite + Lit) sulla stessa porta del WebSocket del Gateway:

* predefinito: `http://<host>:18789/`
* prefisso opzionale: imposta `gateway.controlUi.basePath` (ad es. `/openclaw`)

Le funzionalità sono disponibili nella [Control UI](/it/web/control-ui).
Questa pagina è incentrata sulle modalità di bind, sulla sicurezza e sulle superfici web esposte.

<div id="webhooks">
  ## Webhook
</div>

Quando `hooks.enabled=true`, il Gateway espone anche un piccolo endpoint webhook sullo stesso server HTTP.
Vedi [Configurazione del Gateway](/it/gateway/configuration) → `hooks` per i dettagli su autenticazione e payload.

<div id="config-default-on">
  ## Config (attiva per impostazione predefinita)
</div>

La Control UI è **abilitata per impostazione predefinita** quando le risorse sono presenti (`dist/control-ui`).
Puoi gestirla tramite la configurazione:

```json5
{
  gateway: {
    controlUi: { enabled: true, basePath: "/openclaw" } // basePath opzionale
  }
}
```

<div id="tailscale-access">
  ## Accesso tramite Tailscale
</div>

<div id="integrated-serve-recommended">
  ### Serve integrato (consigliato)
</div>

Mantieni il Gateway su loopback e lascia che Tailscale Serve faccia da proxy:

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" }
  }
}
```

Quindi avvia il Gateway:

```bash
openclaw gateway
```

Apri:

* `https://<magicdns>/` (o il `gateway.controlUi.basePath` che hai configurato)

<div id="tailnet-bind-token">
  ### Associazione alla Tailnet + token
</div>

```json5
{
  gateway: {
    bind: "tailnet",
    controlUi: { enabled: true },
    auth: { mode: "token", token: "your-token" }
  }
}
```

Quindi avvia il Gateway (è richiesto un token per i bind su interfacce non-loopback):

```bash
openclaw gateway
```

Apri:

* `http://<tailscale-ip>:18789/` (o il `gateway.controlUi.basePath` che hai configurato)

<div id="public-internet-funnel">
  ### Internet pubblico (Funnel)
</div>

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password" } // oppure OPENCLAW_GATEWAY_PASSWORD
  }
}
```

<div id="security-notes">
  ## Note sulla sicurezza
</div>

* L&#39;autenticazione del Gateway è richiesta per impostazione predefinita (token/password o intestazioni di identità Tailscale).
* I bind non-loopback **richiedono comunque** un token/password condiviso (`gateway.auth` o variabili d&#39;ambiente).
* Il wizard genera un token del Gateway per impostazione predefinita (anche su loopback).
* La UI invia `connect.params.auth.token` o `connect.params.auth.password`.
* Con Serve, le intestazioni di identità Tailscale possono soddisfare i requisiti di autenticazione quando
  `gateway.auth.allowTailscale` è `true` (nessun token/password richiesto). Imposta
  `gateway.auth.allowTailscale: false` per richiedere credenziali esplicite. Vedi
  [Tailscale](/it/gateway/tailscale) e [Sicurezza](/it/gateway/security).
* `gateway.tailscale.mode: "funnel"` richiede `gateway.auth.mode: "password"` (password condivisa).

<div id="building-the-ui">
  ## Compilare la UI
</div>

Il Gateway serve i file statici da `dist/control-ui`. Generali con:

```bash
pnpm ui:build # installa automaticamente le dipendenze della UI alla prima esecuzione
```
