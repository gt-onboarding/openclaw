---
title: API HTTP per l'invocazione degli strumenti
summary: "Invoca direttamente un singolo strumento tramite l'endpoint HTTP del Gateway"
read_when:
  - Invocare strumenti senza eseguire un turno completo di un agente
  - Creare automazioni che richiedono l'applicazione delle policy degli strumenti
---

<div id="tools-invoke-http">
  # Invocazione strumenti (HTTP)
</div>

Il Gateway di OpenClaw espone un semplice endpoint HTTP per invocare direttamente un singolo strumento. È sempre attivo, ma soggetto all’autenticazione del Gateway e alle policy degli strumenti.

- `POST /tools/invoke`
- Stessa porta del Gateway (multiplex WS + HTTP): `http://<gateway-host>:<port>/tools/invoke`

La dimensione massima predefinita del payload è di 2 MB.

<div id="authentication">
  ## Autenticazione
</div>

Utilizza la configurazione di autenticazione del Gateway. Invia un token bearer:

- `Authorization: Bearer <token>`

Note:

- Quando `gateway.auth.mode="token"`, utilizza `gateway.auth.token` (oppure `OPENCLAW_GATEWAY_TOKEN`).
- Quando `gateway.auth.mode="password"`, utilizza `gateway.auth.password` (oppure `OPENCLAW_GATEWAY_PASSWORD`).

<div id="request-body">
  ## Corpo della richiesta
</div>

```json
{
  "tool": "sessions_list",
  "action": "json",
  "args": {},
  "sessionKey": "main",
  "dryRun": false
}
```

Campi:

* `tool` (string, obbligatorio): nome dello strumento da invocare.
* `action` (string, facoltativo): inserito in `args` se lo schema dello strumento supporta `action` e il payload di `args` l’ha omesso.
* `args` (object, facoltativo): argomenti specifici dello strumento.
* `sessionKey` (string, facoltativo): chiave della sessione di destinazione. Se omessa o uguale a `"main"`, il Gateway usa la chiave di sessione principale configurata (rispetta `session.mainKey` e l’agente predefinito, oppure `global` nello scope globale).
* `dryRun` (boolean, facoltativo): riservato per utilizzi futuri; attualmente ignorato.


<div id="policy-routing-behavior">
  ## Comportamento di policy e instradamento
</div>

La disponibilità degli strumenti è filtrata attraverso la stessa catena di policy utilizzata dagli agenti del Gateway:

- `tools.profile` / `tools.byProvider.profile`
- `tools.allow` / `tools.byProvider.allow`
- `agents.<id>.tools.allow` / `agents.<id>.tools.byProvider.allow`
- policy di gruppo (se la chiave di sessione è mappata a un gruppo o canale)
- policy di subagent (quando invochi con una chiave di sessione di subagent)

Se uno strumento non è autorizzato dalla policy, l'endpoint restituisce **404**.

Per facilitare la risoluzione del contesto da parte delle policy di gruppo, puoi opzionalmente impostare:

- `x-openclaw-message-channel: <channel>` (esempio: `slack`, `telegram`)
- `x-openclaw-account-id: <accountId>` (quando esistono più account)

<div id="responses">
  ## Risposte
</div>

- `200` → `{ ok: true, result }`
- `400` → `{ ok: false, error: { type, message } }` (richiesta non valida o errore dello strumento)
- `401` → non autorizzato
- `404` → strumento non disponibile (non trovato o non presente nella lista di autorizzati)
- `405` → metodo non consentito

<div id="example">
  ## Esempio
</div>

```bash
curl -sS http://127.0.0.1:18789/tools/invoke \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
    "tool": "sessions_list",
    "action": "json",
    "args": {}
  }'
```
