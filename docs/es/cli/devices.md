---
title: Dispositivos
summary: "Referencia de la CLI para `openclaw devices` (emparejamiento de dispositivos y rotación/revocación de tokens)"
read_when:
  - Debes aprobar solicitudes de emparejamiento de dispositivos
  - Debes rotar o revocar tokens de dispositivos
---

<div id="openclaw-devices">
  # `openclaw devices`
</div>

Gestiona las solicitudes de emparejamiento de dispositivos y los tokens con ámbito de dispositivo.

<div id="commands">
  ## Comandos
</div>

<div id="openclaw-devices-list">
  ### `openclaw devices list`
</div>

Muestra las solicitudes de emparejamiento pendientes y los dispositivos emparejados.

```
openclaw devices list
openclaw devices list --json
```


<div id="openclaw-devices-approve-requestid">
  ### `openclaw devices approve <requestId>`
</div>

Aprueba una solicitud pendiente de emparejamiento de dispositivo.

```
openclaw devices approve <requestId>
```


<div id="openclaw-devices-reject-requestid">
  ### `openclaw devices reject <requestId>`
</div>

Rechaza una solicitud pendiente de emparejamiento de dispositivo.

```
openclaw devices reject <requestId>
```


<div id="openclaw-devices-rotate-device-id-role-role-scope-scope">
  ### `openclaw devices rotate --device <id> --role <role> [--scope <scope...>]`
</div>

Rotar el token de un dispositivo para un rol específico (opcionalmente actualizando los ámbitos).

```
openclaw devices rotate --device <deviceId> --role operator --scope operator.read --scope operator.write
```


<div id="openclaw-devices-revoke-device-id-role-role">
  ### `openclaw devices revoke --device <id> --role <role>`
</div>

Revoca el token de un dispositivo para un rol específico.

```
openclaw devices revoke --device <deviceId> --role node
```


<div id="common-options">
  ## Opciones comunes
</div>

- `--url <url>`: URL WebSocket del Gateway (predeterminada: `gateway.remote.url` cuando se ha configurado).
- `--token <token>`: Token del Gateway (si es necesario).
- `--password <password>`: Contraseña del Gateway (autenticación por contraseña).
- `--timeout <ms>`: Tiempo de espera de RPC.
- `--json`: Salida en JSON (recomendado para scripts).

<div id="notes">
  ## Notas
</div>

- La rotación de tokens genera un nuevo token (confidencial). Trátalo como un secreto.
- Estos comandos requieren el ámbito `operator.pairing` (o `operator.admin`).