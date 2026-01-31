---
title: Mensajes de grupo
summary: "Comportamiento y configuración para la gestión de mensajes de grupo de WhatsApp (mentionPatterns se comparten entre interfaces)"
read_when:
  - Cambiar reglas de mensajes de grupo o menciones
---

<div id="group-messages-whatsapp-web-channel">
  # Mensajes de grupo (canal web de WhatsApp)
</div>

Objetivo: permitir que Clawd participe en grupos de WhatsApp, se active solo cuando lo mencionen y mantenga esa conversación separada de la sesión de DM personal.

Nota: `agents.list[].groupChat.mentionPatterns` ahora también lo usan Telegram/Discord/Slack/iMessage; este documento se centra en el comportamiento específico de WhatsApp. Para configuraciones con múltiples agentes, configura `agents.list[].groupChat.mentionPatterns` para cada agente (o usa `messages.groupChat.mentionPatterns` como valor global de respaldo).

<div id="whats-implemented-2025-12-03">
  ## Qué hay implementado (2025-12-03)
</div>

- Modos de activación: `mention` (predeterminado) o `always`. `mention` requiere un ping (@-menciones reales de WhatsApp vía `mentionedJids`, patrones regex o el E.164 del bot en cualquier parte del texto). `always` despierta al agente en cada mensaje, pero solo debería responder cuando pueda aportar valor significativo; en caso contrario devuelve el token silencioso `NO_REPLY`. Los valores predeterminados se pueden establecer en la configuración (`channels.whatsapp.groups`) y sobrescribirse por grupo mediante `/activation`. Cuando se establece `channels.whatsapp.groups`, también actúa como una lista de permitidos de grupos (incluye `"*"` para permitir todos los grupos).
- Política de grupo: `channels.whatsapp.groupPolicy` controla si se aceptan mensajes de grupo (`open|disabled|allowlist`). `allowlist` usa `channels.whatsapp.groupAllowFrom` (como respaldo: `channels.whatsapp.allowFrom` explícito). El valor predeterminado es `allowlist` (bloqueado hasta que agregues remitentes).
- Sesiones por grupo: las claves de sesión tienen el formato `agent:<agentId>:whatsapp:group:<jid>`, por lo que comandos como `/verbose on` o `/think high` (enviados como mensajes independientes) se limitan a ese grupo; el estado de los mensajes directos (DM) personales no se toca. Los latidos se omiten para los hilos de grupo.
- Inyección de contexto: mensajes de grupo **solo pendientes** (50 por defecto) que *no* desencadenaron una ejecución se anteponen bajo `[Chat messages since your last reply - for context]`, con la línea que dispara la ejecución bajo `[Current message - respond to this]`. Los mensajes que ya están en la sesión no se vuelven a inyectar.
- Visibilidad del remitente: ahora cada lote de grupo termina con `[from: Sender Name (+E164)]` para que Pi sepa quién está hablando.
- Efímeros/ver-una-vez: los desempaquetamos antes de extraer texto/menciones, de modo que los pings dentro de ellos sigan disparando.
- Prompt de sistema de grupo: en el primer turno de una sesión de grupo (y siempre que `/activation` cambie el modo) inyectamos un breve texto en el prompt de sistema como `You are replying inside the WhatsApp group "<subject>". Group members: Alice (+44...), Bob (+43...), … Activation: trigger-only … Address the specific sender noted in the message context.` Si los metadatos no están disponibles, de todos modos le indicamos al agente que es un chat de grupo.

<div id="config-example-whatsapp">
  ## Ejemplo de configuración (WhatsApp)
</div>

Añade un bloque `groupChat` a `~/.openclaw/openclaw.json` para que las menciones por nombre para mostrar sigan funcionando incluso cuando WhatsApp elimina el carácter `@` visual del cuerpo del mensaje:

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true }
      }
    }
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          historyLimit: 50,
          mentionPatterns: [
            "@?openclaw",
            "\\+?15555550123"
          ]
        }
      }
    ]
  }
}
```

Notas:

* Las expresiones regulares (regex) no distinguen entre mayúsculas y minúsculas; cubren una mención por nombre para mostrar como `@openclaw` y el número en bruto, con o sin `+`/espacios.
* WhatsApp sigue enviando menciones canónicas mediante `mentionedJids` cuando alguien pulsa el contacto, por lo que el mecanismo de respaldo basado en el número rara vez es necesario, pero sirve como una útil red de seguridad.


<div id="activation-command-owner-only">
  ### Comando de activación (solo propietario)
</div>

Usa el comando en el chat de grupo:

- `/activation mention`
- `/activation always`

Solo el número del propietario (desde `channels.whatsapp.allowFrom`, o el propio E.164 del bot cuando no se haya definido) puede cambiar esto. Envía `/status` como mensaje independiente en el grupo para ver el modo de activación actual.

<div id="how-to-use">
  ## Cómo usar
</div>

1) Añade tu cuenta de WhatsApp (la que ejecuta OpenClaw) al grupo.
2) Di `@openclaw …` (o incluye el número). Solo los remitentes en la lista de permitidos pueden activarlo, a menos que configures `groupPolicy: "open"` (el valor `open` permite aceptar mensajes sin restricciones de cualquier usuario).
3) El prompt del agente incluirá el contexto reciente del grupo más el marcador final `[from: …]` para que pueda dirigirse a la persona correcta.
4) Las directivas a nivel de sesión (`/verbose on`, `/think high`, `/new` o `/reset`, `/compact`) se aplican solo a la sesión de ese grupo; envíalas como mensajes independientes para que se registren. Tu sesión de mensajes directos personales permanece independiente.

<div id="testing-verification">
  ## Pruebas / verificación
</div>

- Comprobación manual básica:
  - Envía un ping `@openclaw` en el grupo y confirma una respuesta que haga referencia al nombre del remitente.
  - Envía un segundo ping y verifica que el bloque de historial se incluya y después se borre en el siguiente turno.
- Revisa los registros del Gateway (ejecútalo con `--verbose`) para ver entradas `inbound web message` que muestren `from: <groupJid>` y el sufijo `[from: …]`.

<div id="known-considerations">
  ## Consideraciones conocidas
</div>

- Los latidos se omiten intencionalmente en los grupos para evitar mensajes de difusión ruidosos.
- La supresión de eco usa la cadena del lote combinado; si envías texto idéntico dos veces sin menciones, solo el primero recibirá una respuesta.
- Las entradas del almacén de sesiones aparecerán como `agent:<agentId>:whatsapp:group:<jid>` en el almacén de sesiones (`~/.openclaw/agents/<agentId>/sessions/sessions.json` de forma predeterminada); que falte una entrada solo significa que el grupo aún no ha disparado ninguna ejecución.
- Los indicadores de escritura en grupos siguen `agents.defaults.typingMode` (predeterminado: `message` cuando no haya menciones).