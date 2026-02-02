---
title: Emparejamiento
summary: "Emparejamiento de nodos propios del Gateway (Opción B) para iOS y otros nodos remotos"
read_when:
  - Implementar aprobaciones de emparejamiento de nodos sin UI de macOS
  - Añadir flujos en la CLI para aprobar nodos remotos
  - Ampliar el protocolo del Gateway para la gestión de nodos
---

<div id="gateway-owned-pairing-option-b">
  # Emparejamiento controlado por el Gateway (Opción B)
</div>

En el emparejamiento controlado por el Gateway, el **Gateway** es la fuente de verdad sobre qué nodos
pueden unirse. Las UI (app de macOS, clientes futuros) son solo frontends que
aprueban o rechazan solicitudes pendientes.

**Importante:** Los nodos WS usan **emparejamiento de dispositivo** (rol `node`) durante `connect`.
`node.pair.*` es un almacén de emparejamiento independiente y **no** controla el handshake WS.
Solo los clientes que llaman explícitamente a `node.pair.*` usan este flujo.

<div id="concepts">
  ## Conceptos
</div>

- **Solicitud pendiente**: un nodo ha solicitado unirse; requiere aprobación.
- **Nodo emparejado**: nodo aprobado al que se le ha emitido un token de autenticación.
- **Transporte**: el endpoint WS del Gateway reenvía solicitudes pero no decide
  sobre la pertenencia. (La compatibilidad con el puente TCP heredado está obsoleta y se ha eliminado.)

<div id="how-pairing-works">
  ## Cómo funciona el emparejamiento
</div>

1. Un nodo se conecta al WS del Gateway y solicita emparejamiento.
2. El Gateway almacena una **solicitud pendiente** y emite `node.pair.requested`.
3. Apruebas o rechazas la solicitud (CLI o UI).
4. Al aprobar, el Gateway genera un **nuevo token** (los tokens se rotan al volver a emparejar).
5. El nodo se reconecta usando el token y ahora queda “emparejado”.

Las solicitudes pendientes caducan automáticamente después de **5 minutos**.

<div id="cli-workflow-headless-friendly">
  ## Flujo de trabajo de la CLI (apto para entornos sin interfaz gráfica)
</div>

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes reject <requestId>
openclaw nodes status
openclaw nodes rename --node <id|name|ip> --name "Living Room iPad"
```

`nodes status` muestra los nodos emparejados o conectados y sus capacidades.


<div id="api-surface-gateway-protocol">
  ## Superficie de la API (protocolo del Gateway)
</div>

Eventos:

- `node.pair.requested` — se emite cuando se crea una nueva solicitud pendiente.
- `node.pair.resolved` — se emite cuando una solicitud se aprueba/rechaza/expira.

Métodos:

- `node.pair.request` — crea o reutiliza una solicitud pendiente.
- `node.pair.list` — lista los nodos pendientes + emparejados.
- `node.pair.approve` — aprueba una solicitud pendiente (emite un token).
- `node.pair.reject` — rechaza una solicitud pendiente.
- `node.pair.verify` — verifica `{ nodeId, token }`.

Notas:

- `node.pair.request` es idempotente para cada nodo: las llamadas repetidas devuelven la misma
  solicitud pendiente.
- La aprobación **siempre** genera un token nuevo; nunca se devuelve un token desde
  `node.pair.request`.
- Las solicitudes pueden incluir `silent: true` como indicador para flujos de aprobación automática.

<div id="auto-approval-macos-app">
  ## Aprobación automática (app de macOS)
</div>

La app de macOS puede, opcionalmente, intentar una **aprobación silenciosa** cuando:

- la solicitud está marcada como `silent`, y
- la app puede verificar una conexión SSH al host del Gateway usando el mismo usuario.

Si la aprobación silenciosa falla, se recurre al diálogo normal de “Aprobar/Rechazar”.

<div id="storage-local-private">
  ## Almacenamiento (local, privado)
</div>

El estado de emparejamiento se almacena en el directorio de estado del Gateway (por defecto `~/.openclaw`):

- `~/.openclaw/nodes/paired.json`
- `~/.openclaw/nodes/pending.json`

Si sobrescribes `OPENCLAW_STATE_DIR`, la carpeta `nodes/` se mueve junto con él.

Notas de seguridad:

- Los tokens son secretos; considera `paired.json` como información sensible.
- Rotar un token requiere una nueva aprobación (o eliminar la entrada del nodo).

<div id="transport-behavior">
  ## Comportamiento del transporte
</div>

- El transporte es **sin estado**; no mantiene información de membresía.
- Si el Gateway está sin conexión o el emparejamiento está deshabilitado, los nodos no pueden emparejarse.
- Si el Gateway está en modo remoto, el emparejamiento sigue realizándose en el almacén de datos del Gateway remoto.