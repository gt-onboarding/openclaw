---
title: Sistema
summary: "Referencia de la CLI para `openclaw system` (eventos del sistema, latidos, presencia)"
read_when:
  - Quieres poner en cola un evento de sistema sin crear una tarea cron
  - Necesitas activar o desactivar los latidos del sistema
  - Quieres inspeccionar las entradas de presencia del sistema
---

<div id="openclaw-system">
  # `openclaw system`
</div>

Herramientas de sistema para el Gateway: encolar eventos de sistema, controlar los latidos
y ver el estado de presencia.

<div id="common-commands">
  ## Comandos comunes
</div>

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
openclaw system heartbeat enable
openclaw system heartbeat last
openclaw system presence
```


<div id="system-event">
  ## `system event`
</div>

Coloca en cola un evento de sistema en la sesión **principal**. El siguiente latido lo
inyectará como una línea `System:` en el prompt. Usa `--mode now` para activar el latido
inmediatamente; `next-heartbeat` espera al siguiente intervalo programado.

Opciones:

- `--text <text>`: texto obligatorio del evento de sistema.
- `--mode <mode>`: `now` o `next-heartbeat` (predeterminado).
- `--json`: salida legible por máquina.

<div id="system-heartbeat-lastenabledisable">
  ## `system heartbeat last|enable|disable`
</div>

Control de latido:

- `last`: muestra el último evento de latido.
- `enable`: vuelve a activar los latidos (usa esto si se desactivaron).
- `disable`: pausa los latidos.

Flags:

- `--json`: salida en formato JSON (legible por máquinas).

<div id="system-presence">
  ## `system presence`
</div>

Lista las entradas de presencia del sistema actuales que el Gateway conoce (nodos,
instancias y líneas de estado similares).

Flags:

- `--json`: salida legible por máquina.

<div id="notes">
  ## Notas
</div>

- Requiere un Gateway en ejecución accesible según tu configuración actual (local o remota).
- Los eventos del sistema son efímeros y no se persisten a través de los reinicios.