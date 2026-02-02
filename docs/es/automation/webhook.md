---
title: Webhook
summary: "Entrada de webhook para activación y ejecuciones de agentes aislados"
read_when:
  - Para agregar o modificar endpoints de webhook
  - Para integrar sistemas externos con OpenClaw
---

<div id="webhooks">
  # Webhooks
</div>

Gateway puede exponer un endpoint HTTP de webhook ligero para activadores externos.

<div id="enable">
  ## Activar
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

Notas:

* `hooks.token` es obligatorio cuando `hooks.enabled=true`.
* `hooks.path` tiene como valor predeterminado `/hooks`.

<div id="auth">
  ## Autenticación
</div>

Cada solicitud debe incluir el token del hook. Es preferible usar encabezados:

* `Authorization: Bearer <token>` (recomendado)
* `x-openclaw-token: <token>`
* `?token=<token>` (obsoleto; genera una advertencia en los logs y se eliminará en una próxima versión principal)

<div id="endpoints">
  ## Endpoints
</div>

<div id="post-hookswake">
  ### `POST /hooks/wake`
</div>

Cuerpo de la solicitud:

```json
{ "text": "System line", "mode": "now" }
```

* `text` **obligatorio** (string): La descripción del evento (por ejemplo, &quot;Nuevo correo recibido&quot;).
* `mode` opcional (`now` | `next-heartbeat`): Indica si se debe activar un latido inmediato (valor predeterminado `now`) o esperar a la próxima comprobación periódica.

Efecto:

* Pone en cola un evento de sistema para la sesión **principal**
* Si `mode=now`, activa un latido inmediato

<div id="post-hooksagent">
  ### `POST /hooks/agent`
</div>

Contenido de la solicitud:

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

* `message` **required** (string): El prompt o mensaje que el agente debe procesar.
* `name` opcional (string): Nombre legible para humanos del hook (p. ej., &quot;GitHub&quot;), usado como prefijo en los resúmenes de la sesión.
* `sessionKey` opcional (string): La clave usada para identificar la sesión del agente. De manera predeterminada es un `hook:<uuid>` aleatorio. Usar una clave consistente permite una conversación de varios turnos dentro del contexto del hook.
* `wakeMode` opcional (`now` | `next-heartbeat`): Indica si se debe activar un latido inmediato (valor predeterminado `now`) o esperar a la siguiente comprobación periódica.
* `deliver` opcional (boolean): Si es `true`, la respuesta del agente se enviará al canal de mensajería. De manera predeterminada es `true`. Las respuestas que son solo confirmaciones de latido se omiten automáticamente.
* `channel` opcional (string): El canal de mensajería para la entrega. Uno de: `last`, `whatsapp`, `telegram`, `discord`, `slack`, `mattermost` (complemento), `signal`, `imessage`, `msteams`. El valor predeterminado es `last`.
* `to` opcional (string): El identificador de destinatario para el canal (p. ej., número de teléfono para WhatsApp/Signal, ID de chat para Telegram, ID de canal para Discord/Slack/Mattermost (complemento), ID de conversación para MS Teams). De manera predeterminada es el último destinatario en la sesión principal.
* `model` opcional (string): Anulación del modelo (p. ej., `anthropic/claude-3-5-sonnet` o un alias). Debe estar en la lista de modelos permitidos si hay restricciones.
* `thinking` opcional (string): Anulación del nivel de razonamiento (p. ej., `low`, `medium`, `high`).
* `timeoutSeconds` opcional (number): Duración máxima de la ejecución del agente, en segundos.

Efecto:

* Ejecuta un turno de agente **aislado** (con su propia clave de sesión)
* Siempre publica un resumen en la sesión **principal**
* Si `wakeMode=now`, activa un latido inmediato

<div id="post-hooksname-mapped">
  ### `POST /hooks/<name>` (mapeado)
</div>

Los nombres de hooks personalizados se resuelven mediante `hooks.mappings` (consulta la configuración). Un mapeo puede
convertir payloads arbitrarios en acciones `wake` o `agent`, con plantillas opcionales o
transformaciones de código.

Opciones de mapeo (resumen):

* `hooks.presets: ["gmail"]` habilita el mapeo integrado de Gmail.
* `hooks.mappings` te permite definir `match`, `action` y plantillas en la configuración.
* `hooks.transformsDir` + `transform.module` carga un módulo JS/TS para lógica personalizada.
* Usa `match.source` para mantener un endpoint de ingesta genérico (enrutamiento basado en el payload).
* Las transformaciones en TS requieren un loader de TS (por ejemplo, `bun` o `tsx`) o `.js` precompilado en tiempo de ejecución.
* Configura `deliver: true` + `channel`/`to` en los mapeos para enrutar respuestas a una interfaz de chat
  (`channel` se establece de forma predeterminada en `last` y recurre a WhatsApp).
* `allowUnsafeExternalContent: true` desactiva el wrapper de seguridad de contenido externo para ese hook
  (peligroso; solo para fuentes internas de confianza).
* `openclaw webhooks gmail setup` escribe la configuración `hooks.gmail` para `openclaw webhooks gmail run`.
  Consulta [Gmail Pub/Sub](/es/automation/gmail-pubsub) para el flujo completo de supervisión (watch) de Gmail.

<div id="responses">
  ## Respuestas
</div>

* `200` para `/hooks/wake`
* `202` para `/hooks/agent` (ejecución asíncrona iniciada)
* `401` si falla la autenticación
* `400` si el payload no es válido
* `413` si el payload es demasiado grande

<div id="examples">
  ## Ejemplos
</div>

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"New email received","mode":"now"}'
```

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","wakeMode":"next-heartbeat"}'
```

<div id="use-a-different-model">
  ### Usar un modelo diferente
</div>

Añade `model` al payload del agente (o al mapa) para anular el modelo en esa ejecución:

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

Si haces cumplir `agents.defaults.models`, asegúrate de que el modelo de reemplazo esté incluido ahí.

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

<div id="security">
  ## Seguridad
</div>

* Mantén los endpoints de hook detrás de loopback, tailnet o un proxy inverso de confianza.
* Usa un token dedicado para los hooks; no reutilices los tokens de autenticación del Gateway.
* Evita incluir cargas útiles sensibles sin procesar en los registros de webhooks.
* Las cargas útiles de hooks se tratan como no confiables y se protegen mediante límites de seguridad de forma predeterminada.
  Si debes desactivar esto para un hook específico, establece `allowUnsafeExternalContent: true`
  en el mapping de ese hook (peligroso).