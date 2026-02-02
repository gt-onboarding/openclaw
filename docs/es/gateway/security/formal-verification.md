---
title: Verificación formal (modelos de seguridad)
summary: Modelos de seguridad verificados automáticamente para las rutas más críticas de OpenClaw.
permalink: /security/formal-verification/
---

<div id="formal-verification-security-models">
  # Verificación formal (modelos de seguridad)
</div>

Esta página recopila los **modelos formales de seguridad** de OpenClaw (TLA+/TLC hoy; más según sea necesario).

> Nota: algunos enlaces antiguos pueden hacer referencia al nombre previo del proyecto.

**Objetivo (estrella polar):** proporcionar un argumento verificado por máquina de que OpenClaw hace cumplir su
política de seguridad prevista (autorización, aislamiento de sesión, control de acceso a herramientas y
seguridad frente a errores de configuración), bajo supuestos explícitos.

**Qué es esto (hoy):** una **suite de regresión de seguridad** ejecutable y dirigida por el atacante:

- Cada afirmación tiene una verificación de modelos ejecutable sobre un espacio de estados finito.
- Muchas afirmaciones tienen un **modelo negativo** asociado que produce una traza de contraejemplo para una clase realista de errores.

**Qué no es esto (todavía):** una prueba de que «OpenClaw es seguro en todos los aspectos» o de que la implementación completa en TypeScript es correcta.

<div id="where-the-models-live">
  ## Dónde se encuentran los modelos
</div>

Los modelos se mantienen en un repositorio separado: [vignesh07/openclaw-formal-models](https://github.com/vignesh07/openclaw-formal-models).

<div id="important-caveats">
  ## Advertencias importantes
</div>

- Estos son **modelos**, no la implementación completa en TypeScript. Es posible que exista desincronización entre el modelo y el código.
- Los resultados están acotados por el espacio de estados explorado por TLC; «verde» no implica seguridad más allá de los supuestos y límites del modelo.
- Algunas afirmaciones dependen de supuestos explícitos sobre el entorno (por ejemplo, despliegue correcto, entradas de configuración correctas).

<div id="reproducing-results">
  ## Reproducción de resultados
</div>

Actualmente, los resultados se reproducen clonando localmente el repositorio de modelos y ejecutando TLC (ver más abajo). Una versión futura podría ofrecer:

* Modelos ejecutados en CI con artefactos públicos (trazas de contraejemplos, registros de ejecución)
* Un flujo de trabajo alojado de “ejecutar este modelo” para comprobaciones pequeñas y acotadas

Para empezar:

```bash
git clone https://github.com/vignesh07/openclaw-formal-models
cd openclaw-formal-models

# Se requiere Java 11+ (TLC se ejecuta en la JVM).
# El repositorio incluye un `tla2tools.jar` fijado (herramientas TLA+) y proporciona `bin/tlc` + objetivos de Make.

make <target>
```


<div id="gateway-exposure-and-open-gateway-misconfiguration">
  ### Exposición del Gateway y errores de configuración que lo dejan abierto
</div>

**Afirmación:** escuchar en interfaces más allá de loopback sin autenticación puede hacer posible una intrusión remota / aumenta la exposición; el token o la contraseña bloquea a los atacantes no autenticados (según las suposiciones del modelo).

- Ejecuciones en verde:
  - `make gateway-exposure-v2`
  - `make gateway-exposure-v2-protected`
- Rojo (esperado):
  - `make gateway-exposure-v2-negative`

Ver también: `docs/gateway-exposure-matrix.md` en el repositorio de modelos.

<div id="nodesrun-pipeline-highest-risk-capability">
  ### Pipeline de `nodes.run` (capacidad de mayor riesgo)
</div>

**Afirmación:** `nodes.run` requiere (a) una lista de permitidos de comandos del nodo más los comandos declarados y (b) aprobación en tiempo real cuando está habilitado; las aprobaciones se tokenizan para evitar ataques de repetición (en el modelo).

- Ejecuciones en verde:
  - `make nodes-pipeline`
  - `make approvals-token`
- En rojo (esperado):
  - `make nodes-pipeline-negative`
  - `make approvals-token-negative`

<div id="pairing-store-dm-gating">
  ### Almacén de emparejamiento (control de acceso por MD)
</div>

**Afirmación:** las solicitudes de emparejamiento respetan el TTL y los límites de solicitudes pendientes.

- Ejecuciones en verde:
  - `make pairing`
  - `make pairing-cap`
- En rojo (esperado):
  - `make pairing-negative`
  - `make pairing-cap-negative`

<div id="ingress-gating-mentions-control-command-bypass">
  ### Control de ingreso (menciones + bypass de comando de control)
</div>

**Afirmación:** en contextos de grupo que requieren mención, un “comando de control” no autorizado no puede eludir el control de ingreso basado en menciones.

- Verde:
  - `make ingress-gating`
- Rojo (esperado):
  - `make ingress-gating-negative`

<div id="routingsession-key-isolation">
  ### Aislamiento de enrutamiento/clave de sesión
</div>

**Afirmación:** Los mensajes directos (DM) de pares distintos no se combinan en la misma sesión a menos que se vinculen o configuren explícitamente.

- Verde:
  - `make routing-isolation`
- Rojo (esperado):
  - `make routing-isolation-negative`

<div id="v1-additional-bounded-models-concurrency-retries-trace-correctness">
  ## v1++: modelos acotados adicionales (concurrencia, reintentos, corrección de trazas)
</div>

Estos son modelos adicionales que aumentan la precisión respecto de los modos de fallo del mundo real (actualizaciones no atómicas, reintentos y difusión de mensajes).

<div id="pairing-store-concurrency-idempotency">
  ### Concurrencia / idempotencia del almacén de emparejamiento
</div>

**Afirmación:** un almacén de emparejamiento debe hacer cumplir `MaxPending` y la idempotencia incluso bajo entrecruzamientos de operaciones (es decir, “check-then-write” debe ser atómico/bloqueado; `refresh` no debería crear duplicados).

Lo que implica:

- Bajo solicitudes concurrentes, no se puede exceder `MaxPending` para un canal.
- Solicitudes/actualizaciones repetidas para el mismo `(channel, sender)` no deberían crear registros pendientes activos duplicados.

- Ejecuciones en verde:
  - `make pairing-race` (verificación de límite atómica/bloqueada)
  - `make pairing-idempotency`
  - `make pairing-refresh`
  - `make pairing-refresh-race`
- En rojo (esperado):
  - `make pairing-race-negative` (condición de carrera en el límite con begin/commit no atómicos)
  - `make pairing-idempotency-negative`
  - `make pairing-refresh-negative`
  - `make pairing-refresh-race-negative`

<div id="ingress-trace-correlation-idempotency">
  ### Correlación de trazas de ingesta / idempotencia
</div>

**Afirmación:** la ingesta debe preservar la correlación de trazas a través del fan-out y ser idempotente frente a reintentos del proveedor.

Lo que significa:

- Cuando un evento externo se convierte en múltiples mensajes internos, cada parte mantiene la misma identidad de traza/evento.
- Los reintentos no dan lugar a procesamiento duplicado.
- Si faltan los ID de eventos del proveedor, la desduplicación recurre a una clave segura (p. ej., el ID de traza) para evitar descartar eventos distintos.

- Verde:
  - `make ingress-trace`
  - `make ingress-trace2`
  - `make ingress-idempotency`
  - `make ingress-dedupe-fallback`
- Rojo (esperado):
  - `make ingress-trace-negative`
  - `make ingress-trace2-negative`
  - `make ingress-idempotency-negative`
  - `make ingress-dedupe-fallback-negative`

<div id="routing-dmscope-precedence-identitylinks">
  ### Precedencia de enrutamiento de dmScope + identityLinks
</div>

**Afirmación:** el enrutamiento debe mantener las sesiones de DM aisladas de forma predeterminada y solo fusionar sesiones cuando esté configurado explícitamente (precedencia de canal + identityLinks).

Es decir:

- Las anulaciones de dmScope específicas del canal deben prevalecer sobre los valores predeterminados globales.
- identityLinks solo debe fusionar sesiones dentro de grupos explícitamente vinculados, no entre pares no relacionados.

- Verde:
  - `make routing-precedence`
  - `make routing-identitylinks`
- Rojo (esperado):
  - `make routing-precedence-negative`
  - `make routing-identitylinks-negative`