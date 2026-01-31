---
title: Bloqueo del Gateway
summary: "Mecanismo de bloqueo singleton del Gateway mediante la vinculación del listener WebSocket"
read_when:
  - Al ejecutar o depurar el proceso del Gateway
  - Al investigar la aplicación de la restricción de instancia única
---

<div id="gateway-lock">
  # Bloqueo del Gateway
</div>

Última actualización: 2025-12-11

<div id="why">
  ## Por qué
</div>

- Garantizar que solo se ejecute una instancia del Gateway por puerto base en el mismo host; las instancias adicionales del Gateway deben usar perfiles aislados y puertos únicos.
- Sobrevivir a caídas/SIGKILL sin dejar archivos de bloqueo obsoletos.
- Fallar de forma inmediata con un error claro cuando el puerto de control ya está ocupado.

<div id="mechanism">
  ## Mecanismo
</div>

- El Gateway enlaza el listener WS (por defecto `ws://127.0.0.1:18789`) inmediatamente al arrancar usando un listener TCP exclusivo.
- Si el enlace falla con `EADDRINUSE`, el arranque lanza `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:<port>")`.
- El SO libera el listener automáticamente cuando cualquier proceso termina, incluidos fallos y SIGKILL—no se necesita ningún archivo de bloqueo independiente ni ningún paso de limpieza adicional.
- Al apagarse, el Gateway cierra el servidor WS y el servidor HTTP subyacente para liberar el puerto de forma inmediata.

<div id="error-surface">
  ## Superficie de errores
</div>

- Si otro proceso está ocupando el puerto, el arranque lanza `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:<port>")`.
- Otros errores de bind se exponen como `GatewayLockError("failed to bind gateway socket on ws://127.0.0.1:<port>: …")`.

<div id="operational-notes">
  ## Notas operativas
</div>

- Si el puerto está ocupado por *otro* proceso, el error será el mismo; libera el puerto o elige otro con `openclaw gateway --port <port>`.
- La aplicación de macOS sigue manteniendo su propio mecanismo ligero de protección por PID antes de lanzar el Gateway; el bloqueo en tiempo de ejecución se aplica mediante el binding de WebSocket.