---
title: Dispositivi
summary: "Riferimento CLI per `openclaw devices` (abbinamento dispositivi + rotazione/revoca dei token)"
read_when:
  - Devi approvare richieste di abbinamento dei dispositivi
  - Devi ruotare o revocare i token dei dispositivi
---

<div id="openclaw-devices">
  # `openclaw devices`
</div>

Gestisce le richieste di abbinamento dei dispositivi e i token con scope a livello di dispositivo.

<div id="commands">
  ## Comandi
</div>

<div id="openclaw-devices-list">
  ### `openclaw devices list`
</div>

Elenca le richieste di abbinamento in attesa e i dispositivi abbinati.

```
openclaw devices list
openclaw devices list --json
```


<div id="openclaw-devices-approve-requestid">
  ### `openclaw devices approve <requestId>`
</div>

Approva una richiesta di abbinamento di un dispositivo in sospeso.

```
openclaw devices approve <requestId>
```


<div id="openclaw-devices-reject-requestid">
  ### `openclaw devices reject <requestId>`
</div>

Rifiuta una richiesta di abbinamento di dispositivo in attesa.

```
openclaw devices reject <requestId>
```


<div id="openclaw-devices-rotate-device-id-role-role-scope-scope">
  ### `openclaw devices rotate --device <id> --role <role> [--scope <scope...>]`
</div>

Rigenera il token del dispositivo per un ruolo specifico (eventualmente aggiornando gli scope).

```
openclaw devices rotate --device <deviceId> --role operator --scope operator.read --scope operator.write
```


<div id="openclaw-devices-revoke-device-id-role-role">
  ### `openclaw devices revoke --device <id> --role <role>`
</div>

Revoca il token del dispositivo per un ruolo specifico.

```
openclaw devices revoke --device <deviceId> --role node
```


<div id="common-options">
  ## Opzioni comuni
</div>

- `--url <url>`: URL WebSocket del Gateway (impostato per impostazione predefinita su `gateway.remote.url` quando configurato).
- `--token <token>`: token del Gateway (se richiesto).
- `--password <password>`: password del Gateway (autenticazione tramite password).
- `--timeout <ms>`: timeout RPC.
- `--json`: output JSON (consigliato per l’uso tramite script).

<div id="notes">
  ## Note
</div>

- La rotazione del token genera un nuovo token (sensibile). Trattalo come un segreto.
- Questi comandi richiedono lo scope `operator.pairing` (o `operator.admin`).