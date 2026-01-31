---
title: Cola
summary: "Diseño de cola de comandos que serializa las ejecuciones de respuestas automáticas entrantes"
read_when:
  - Al modificar la ejecución o la concurrencia de las respuestas automáticas
---

<div id="command-queue-2026-01-16">
  # Cola de comandos (2026-01-16)
</div>

Serializamos las ejecuciones entrantes de respuestas automáticas (en todos los canales) mediante una pequeña cola interna en proceso para evitar que múltiples ejecuciones de agentes colisionen, al tiempo que seguimos permitiendo un paralelismo seguro entre sesiones.

<div id="why">
  ## Por qué
</div>

- Las ejecuciones de respuesta automática pueden ser costosas (llamadas al LLM) y pueden entrar en conflicto cuando llegan varios mensajes entrantes casi al mismo tiempo.
- La serialización evita competir por recursos compartidos (archivos de sesión, logs, stdin de la CLI) y reduce la probabilidad de alcanzar los límites de frecuencia impuestos por servicios upstream.

<div id="how-it-works">
  ## Cómo funciona
</div>

- Una cola FIFO consciente de carriles vacía cada carril con un límite de concurrencia configurable (valor predeterminado de 1 para carriles no configurados; `main` predeterminado en 4, `subagent` en 8).
- `runEmbeddedPiAgent` encola por **session key** (carril `session:<key>`) para garantizar que solo haya una ejecución activa por sesión.
- Cada ejecución de sesión se encola luego en un **carril global** (`main` de forma predeterminada), de modo que el paralelismo total quede limitado por `agents.defaults.maxConcurrent`.
- Cuando el registro detallado está habilitado, las ejecuciones en cola emiten un aviso breve si han esperado más de ~2 s antes de comenzar.
- Los indicadores de escritura siguen activándose inmediatamente al encolarse (cuando el canal los admite), por lo que la experiencia de usuario no cambia mientras esperamos nuestro turno.

<div id="queue-modes-per-channel">
  ## Modos de cola (por canal)
</div>

Los mensajes entrantes pueden dirigir la ejecución actual, esperar a un turno posterior o hacer ambas cosas:

* `steer`: se inyecta inmediatamente en la ejecución actual (cancela las llamadas de herramientas pendientes después del siguiente límite de herramienta). Si no hay streaming, pasa a `followup`.
* `followup`: se encola para el siguiente turno del agente después de que termine la ejecución actual.
* `collect`: fusiona todos los mensajes en cola en un **único** turno posterior (predeterminado). Si los mensajes se dirigen a diferentes canales/hilos, se vacían individualmente para preservar el enrutamiento.
* `steer-backlog` (también `steer+backlog`): dirige ahora **y** conserva el mensaje para un turno posterior.
* `interrupt` (heredado): aborta la ejecución activa para esa sesión y luego ejecuta el mensaje más reciente.
* `queue` (alias heredado): igual que `steer`.

Steer-backlog significa que puedes obtener una respuesta posterior después de la ejecución dirigida, por lo que
las interfaces de streaming pueden parecer duplicadas. Es preferible usar `collect`/`steer` si quieres
una respuesta por mensaje entrante.
Envía `/queue collect` como un comando independiente (por sesión) o establece `messages.queue.byChannel.discord: "collect"`.

Valores predeterminados (cuando no se definen en la configuración):

* Todas las interfaces → `collect`

Configura globalmente o por canal mediante `messages.queue`:

```json5
{
  messages: {
    queue: {
      mode: "collect",
      debounceMs: 1000,
      cap: 20,
      drop: "summarize",
      byChannel: { discord: "collect" }
    }
  }
}
```


<div id="queue-options">
  ## Opciones de la cola
</div>

Las opciones se aplican a `followup`, `collect` y `steer-backlog` (y a `steer` cuando recurre a `followup` como alternativa):

- `debounceMs`: espera un periodo de inactividad antes de iniciar un turno de `followup` (evita “continuar, continuar”).
- `cap`: máximo de mensajes en cola por sesión.
- `drop`: política de desbordamiento (`old`, `new`, `summarize`).

`summarize` mantiene una lista breve con viñetas de los mensajes descartados y la inyecta como un prompt de `followup` sintético.  
Valores predeterminados: `debounceMs: 1000`, `cap: 20`, `drop: summarize`.

<div id="per-session-overrides">
  ## Sobrescrituras por sesión
</div>

- Envía `/queue <modo>` como un comando independiente para almacenar el modo de la sesión actual.
- Las opciones se pueden combinar: `/queue collect debounce:2s cap:25 drop:summarize`
- `/queue default` o `/queue reset` elimina la sobrescritura de la sesión.

<div id="scope-and-guarantees">
  ## Ámbito y garantías
</div>

- Se aplica a ejecuciones de agentes de respuesta automática en todos los canales entrantes que usan el pipeline de respuesta del Gateway (WhatsApp web, Telegram, Slack, Discord, Signal, iMessage, webchat, etc.).
- El carril predeterminado (`main`) es global al proceso para los latidos entrantes y principales; establece `agents.defaults.maxConcurrent` para permitir múltiples sesiones en paralelo.
- Pueden existir carriles adicionales (p. ej. `cron`, `subagent`) para que los trabajos en segundo plano se ejecuten en paralelo sin bloquear las respuestas entrantes.
- Los carriles por sesión garantizan que solo una ejecución de agente toque una sesión determinada a la vez.
- Sin dependencias externas ni hilos de trabajo en segundo plano; TypeScript puro + promesas.

<div id="troubleshooting">
  ## Solución de problemas
</div>

- Si los comandos parecen quedarse bloqueados, habilita los logs detallados y busca líneas con “queued for …ms” para confirmar que la cola se está vaciando.
- Si necesitas conocer la profundidad de la cola, habilita los logs detallados y observa las líneas con los tiempos de cola.