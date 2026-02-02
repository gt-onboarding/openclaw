---
title: Elevado
summary: "Modo de ejecución elevado y directivas /elevated"
read_when:
  - Al ajustar la configuración predeterminada del modo elevado, las listas de permitidos o el comportamiento de los comandos slash
---

<div id="elevated-mode-elevated-directives">
  # Modo elevado (directivas /elevated)
</div>

<div id="what-it-does">
  ## Lo que hace
</div>

- `/elevated on` se ejecuta en el host del Gateway y mantiene las aprobaciones de exec (igual que `/elevated ask`).
- `/elevated full` se ejecuta en el host del Gateway **y** aprueba automáticamente exec (omite las aprobaciones de exec).
- `/elevated ask` se ejecuta en el host del Gateway pero mantiene las aprobaciones de exec (igual que `/elevated on`).
- `on`/`ask` **no** fuerzan `exec.security=full`; la política configurada de seguridad/ask sigue vigente.
- Solo cambia el comportamiento cuando el agente está en **sandbox** (de lo contrario exec ya se ejecuta en el host).
- Formas de la directiva: `/elevated on|off|ask|full`, `/elev on|off|ask|full`.
- Solo se aceptan `on|off|ask|full`; cualquier otra cosa muestra una sugerencia y no cambia el estado.

<div id="what-it-controls-and-what-it-doesnt">
  ## Qué controla (y qué no)
</div>

- **Controles de disponibilidad**: `tools.elevated` es la referencia global. `agents.list[].tools.elevated` puede restringir aún más elevated por agente (ambos deben permitirlo).
- **Estado por sesión**: `/elevated on|off|ask|full` establece el nivel de elevated para la clave de sesión actual.
- **Directiva en línea**: `/elevated on|ask|full` dentro de un mensaje se aplica solo a ese mensaje.
- **Grupos**: en chats de grupo, las directivas elevated solo se respetan cuando se menciona al agente. Los mensajes que solo contienen comandos y que pasan por alto el requisito de mención se tratan como si hubieran mencionado al agente.
- **Ejecución en el host**: elevated fuerza `exec` en el host del Gateway; `full` también establece `security=full`.
- **Aprobaciones**: `full` omite las aprobaciones de `exec`; `on`/`ask` las respetan cuando las reglas de lista de permitidos/ask lo requieren.
- **Agentes sin sandbox**: no tiene ningún efecto sobre la ubicación; solo afecta el control de acceso, el registro y el estado.
- **La política de herramientas sigue aplicándose**: si `exec` está denegado por la política de herramientas, no se puede usar elevated.
- **Separado de `/exec`**: `/exec` ajusta los valores predeterminados por sesión para remitentes autorizados y no requiere elevated.

<div id="resolution-order">
  ## Orden de resolución
</div>

1. Directiva en línea en el mensaje (se aplica solo a ese mensaje).
2. Anulación de sesión (establecida al enviar un mensaje que solo contenga la directiva).
3. Valor predeterminado global (`agents.defaults.elevatedDefault` en la configuración).

<div id="setting-a-session-default">
  ## Configuración de un valor predeterminado de sesión
</div>

- Envía un mensaje que contenga **solo** la directiva (se permiten espacios en blanco), p. ej. `/elevated full`.
- Se envía una respuesta de confirmación (`Elevated mode set to full...` / `Elevated mode disabled.`).
- Si el acceso elevado está deshabilitado o el remitente no está en la lista de permitidos aprobada, la directiva responde con un error con instrucciones sobre cómo actuar y no cambia el estado de la sesión.
- Envía `/elevated` (o `/elevated:`) sin argumento para ver el nivel elevado actual.

<div id="availability-allowlists">
  ## Disponibilidad y listas de permitidos
</div>

- Feature gate: `tools.elevated.enabled` (de forma predeterminada puede estar desactivado mediante la configuración incluso si el código lo admite).
- Lista de permitidos de remitentes: `tools.elevated.allowFrom` con listas de permitidos por proveedor (p. ej. `discord`, `whatsapp`).
- Control por agente: `agents.list[].tools.elevated.enabled` (opcional; solo puede restringir más).
- Lista de permitidos por agente: `agents.list[].tools.elevated.allowFrom` (opcional; cuando se establece, el remitente debe coincidir con **ambas** listas de permitidos, global y por agente).
- Fallback de Discord: si se omite `tools.elevated.allowFrom.discord`, se utiliza la lista `channels.discord.dm.allowFrom` como fallback. Establece `tools.elevated.allowFrom.discord` (incluso `[]`) para anularla. Las listas de permitidos por agente **no** usan el fallback.
- Todos los controles deben superarse; de lo contrario, el modo elevado se considera no disponible.

<div id="logging-status">
  ## Registro y estado
</div>

- Las llamadas de ejecución elevada se registran al nivel de información (`info`).
- El estado de la sesión incluye el modo elevado (por ejemplo, `elevated=ask`, `elevated=full`).