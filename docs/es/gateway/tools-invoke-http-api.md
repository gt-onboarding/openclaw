---
title: Invocar herramientas mediante la API HTTP
summary: "Invoca una única herramienta directamente a través del endpoint HTTP del Gateway"
read_when:
  - Llamar a herramientas sin ejecutar un turno completo de un agente
  - Crear automatizaciones que requieren la aplicación de políticas de herramientas
---

<div id="tools-invoke-http">
  # Invocación de herramientas (HTTP)
</div>

El Gateway de OpenClaw expone un endpoint HTTP sencillo para invocar directamente una sola herramienta. Siempre está habilitado, pero controlado por la autenticación del Gateway y la política de herramientas.

- `POST /tools/invoke`
- Mismo puerto que el Gateway (multiplexación WS + HTTP): `http://<gateway-host>:<port>/tools/invoke`

El tamaño máximo de la carga útil por defecto es de 2 MB.

<div id="authentication">
  ## Autenticación
</div>

Usa la configuración de autenticación del Gateway. Envía un token de tipo Bearer:

- `Authorization: Bearer <token>`

Notas:

- Cuando `gateway.auth.mode="token"`, usa `gateway.auth.token` (o `OPENCLAW_GATEWAY_TOKEN`).
- Cuando `gateway.auth.mode="password"`, usa `gateway.auth.password` (o `OPENCLAW_GATEWAY_PASSWORD`).

<div id="request-body">
  ## Cuerpo de la solicitud
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

Campos:

* `tool` (string, required): nombre de la herramienta que se va a invocar.
* `action` (string, optional): se mapea en `args` si el esquema de la herramienta admite `action` y la carga útil de `args` la omitió.
* `args` (object, optional): argumentos específicos de la herramienta.
* `sessionKey` (string, optional): clave de sesión de destino. Si se omite o es `"main"`, el Gateway utiliza la clave de sesión principal configurada (respeta `session.mainKey` y el agente predeterminado, o `global` en el ámbito global).
* `dryRun` (boolean, optional): reservado para uso futuro; actualmente se ignora.


<div id="policy-routing-behavior">
  ## Comportamiento de políticas y enrutamiento
</div>

La disponibilidad de herramientas se filtra a través de la misma cadena de políticas que usan los agentes del Gateway:

- `tools.profile` / `tools.byProvider.profile`
- `tools.allow` / `tools.byProvider.allow`
- `agents.<id>.tools.allow` / `agents.<id>.tools.byProvider.allow`
- políticas de grupo (si la clave de sesión se asigna a un grupo o canal)
- política de subagente (cuando se invoca con una clave de sesión de subagente)

Si una herramienta no está permitida por las políticas, el endpoint devuelve **404**.

Para ayudar a que las políticas de grupo puedan resolver el contexto, puedes configurar opcionalmente:

- `x-openclaw-message-channel: <channel>` (ejemplo: `slack`, `telegram`)
- `x-openclaw-account-id: <accountId>` (cuando existen varias cuentas)

<div id="responses">
  ## Respuestas
</div>

- `200` → `{ ok: true, result }`
- `400` → `{ ok: false, error: { type, message } }` (solicitud no válida o error de herramienta)
- `401` → no autorizado
- `404` → herramienta no disponible (no encontrada o no incluida en la lista de permitidos)
- `405` → método no permitido

<div id="example">
  ## Ejemplo
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
