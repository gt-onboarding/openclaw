---
title: Dashboard
summary: "Accesso e autenticazione alla dashboard del Gateway (Control UI)"
read_when:
  - Modifica delle modalità di autenticazione o di esposizione della dashboard
---

<div id="dashboard-control-ui">
  # Dashboard (Control UI)
</div>

La dashboard del Gateway è la Control UI nel browser servita su `/` per impostazione predefinita
(puoi sovrascrivere questo comportamento con `gateway.controlUi.basePath`).

Apertura rapida (Gateway locale):

* http://127.0.0.1:18789/ (oppure http://localhost:18789/)

Riferimenti chiave:

* [Control UI](/it/web/control-ui) per l&#39;utilizzo e le funzionalità della UI.
* [Tailscale](/it/gateway/tailscale) per l&#39;automazione Serve/Funnel.
* [Superfici web](/it/web) per le modalità di bind e le note sulla sicurezza.

L&#39;autenticazione è applicata durante l&#39;handshake WebSocket tramite `connect.params.auth`
(token o password). Vedi `gateway.auth` in [Configurazione del Gateway](/it/gateway/configuration).

Nota di sicurezza: la Control UI è una **superficie di amministrazione** (chat, configurazione, approvazioni di esecuzione).
Non esporla pubblicamente. La UI memorizza il token in `localStorage` dopo il primo caricamento.
Preferisci usare localhost, Tailscale Serve o un tunnel SSH.

<div id="fast-path-recommended">
  ## Percorso rapido (consigliato)
</div>

* Dopo l&#39;onboarding, la CLI apre automaticamente la dashboard con il tuo token e mostra lo stesso link con il token incluso.
* Puoi riaprirla in qualsiasi momento con: `openclaw dashboard` (copia il link, apre il browser se possibile, mostra un suggerimento per SSH se in modalità headless).
* Il token rimane in locale (solo come parametro di query); la UI lo rimuove dopo il primo caricamento e lo salva in localStorage.

<div id="token-basics-local-vs-remote">
  ## Nozioni di base sui token (locale vs remoto)
</div>

* **Localhost**: apri `http://127.0.0.1:18789/`. Se vedi `"unauthorized"`, esegui `openclaw dashboard` e usa il link contenente il token (`?token=...`).
* **Origine del token**: `gateway.auth.token` (o `OPENCLAW_GATEWAY_TOKEN`); la UI lo salva al primo caricamento.
* **Non localhost**: usa Tailscale Serve (senza token se `gateway.auth.allowTailscale: true`), il binding del tailnet con un token oppure un tunnel SSH. Consulta [Web surfaces](/it/web).

<div id="if-you-see-unauthorized-1008">
  ## Se vedi “unauthorized” / 1008
</div>

* Esegui `openclaw dashboard` per ottenere un nuovo link tokenizzato.
* Assicurati che il Gateway sia raggiungibile (locale: `openclaw status`; remoto: tunnel SSH `ssh -N -L 18789:127.0.0.1:18789 user@host` quindi apri `http://127.0.0.1:18789/?token=...`).
* Nelle impostazioni della dashboard, incolla lo stesso token che hai configurato in `gateway.auth.token` (o `OPENCLAW_GATEWAY_TOKEN`).