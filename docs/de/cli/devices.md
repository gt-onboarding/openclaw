---
title: Geräte
summary: "CLI-Referenz für `openclaw devices` (Gerätekopplung + Rotation/Widerruf von Gerätetoken)"
read_when:
  - Du Kopplungsanfragen für Geräte genehmigst
  - Du Gerätetoken rotieren oder widerrufen musst
---

<div id="openclaw-devices">
  # `openclaw devices`
</div>

Verwalte Geräte-Kopplungsanforderungen und gerätebezogene Token.

<div id="commands">
  ## Befehle
</div>

<div id="openclaw-devices-list">
  ### `openclaw devices list`
</div>

Listet ausstehende Kopplungsanforderungen und gekoppelte Geräte auf.

```
openclaw devices list
openclaw devices list --json
```


<div id="openclaw-devices-approve-requestid">
  ### `openclaw devices approve <requestId>`
</div>

Ausstehende Gerätekopplungsanfrage genehmigen.

```
openclaw devices approve <requestId>
```


<div id="openclaw-devices-reject-requestid">
  ### `openclaw devices reject <requestId>`
</div>

Eine ausstehende Gerätekopplungsanfrage ablehnen.

```
openclaw devices reject <requestId>
```


<div id="openclaw-devices-rotate-device-id-role-role-scope-scope">
  ### `openclaw devices rotate --device <id> --role <role> [--scope <scope...>]`
</div>

Rotiert ein Gerätetoken für eine bestimmte Rolle (optional mit aktualisierten Scopes).

```
openclaw devices rotate --device <deviceId> --role operator --scope operator.read --scope operator.write
```


<div id="openclaw-devices-revoke-device-id-role-role">
  ### `openclaw devices revoke --device <id> --role <role>`
</div>

Gerätetoken für eine bestimmte Rolle widerrufen.

```
openclaw devices revoke --device <deviceId> --role node
```


<div id="common-options">
  ## Allgemeine Optionen
</div>

- `--url <url>`: Gateway-WebSocket-URL (Standardwert ist `gateway.remote.url`, sofern konfiguriert).
- `--token <token>`: Gateway-Token (falls erforderlich).
- `--password <password>`: Gateway-Passwort (Passwort-Authentifizierung).
- `--timeout <ms>`: RPC-Timeout.
- `--json`: JSON-Ausgabe (für Skripte empfohlen).

<div id="notes">
  ## Hinweise
</div>

- Token-Rotation erzeugt ein neues (vertrauliches) Token. Behandle es wie ein Secret.
- Diese Befehle erfordern den Scope `operator.pairing` (oder `operator.admin`).