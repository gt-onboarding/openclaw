---
title: Clawnet
summary: "Refactorización de Clawnet: unificar el protocolo de red, los roles, la autenticación, las aprobaciones y la identidad"
read_when:
  - Al planificar un protocolo de red unificado para nodos y clientes del operador
  - Al rediseñar aprobaciones, emparejamiento, TLS y presencia en todos los dispositivos
---

<div id="clawnet-refactor-protocol-auth-unification">
  # Refactorización de Clawnet (unificación de protocolo y autenticación)
</div>

<div id="hi">
  ## Hola
</div>

Hola, Peter: muy buen enfoque; esto permite una experiencia de usuario más sencilla y una seguridad más sólida.

<div id="purpose">
  ## Propósito
</div>

Documento único y riguroso que cubra:

- Estado actual: protocolos, flujos, límites de confianza.
- Puntos problemáticos: aprobaciones, enrutamiento multisalto, duplicación de UI.
- Nuevo estado propuesto: un protocolo, roles con ámbito, autenticación/emparejamiento unificados, pinning de TLS.
- Modelo de identidad: IDs estables + slugs simpáticos.
- Plan de migración, riesgos, preguntas abiertas.

<div id="goals-from-discussion">
  ## Objetivos (a partir de la discusión)
</div>

- Un único protocolo para todos los clientes (app para macOS, CLI, iOS, Android, nodo sin interfaz gráfica).
- Cada participante de la red autenticado y emparejado.
- Claridad de roles: nodos vs operadores.
- Aprobaciones centralizadas enviadas a donde esté el usuario.
- Cifrado TLS + fijación opcional de certificados para todo el tráfico remoto.
- Duplicación mínima de código.
- Cada máquina debe aparecer una sola vez (sin entradas duplicadas en la UI y el nodo).

<div id="nongoals-explicit">
  ## No objetivos (explícitos)
</div>

- Eliminar la separación de capacidades (aún se necesita el principio de mínimo privilegio).
- Exponer el plano de control completo del Gateway sin comprobaciones de scope.
- Hacer que la autenticación dependa de etiquetas humanas (los slugs siguen sin ofrecer garantías de seguridad).

---

<div id="current-state-asis">
  # Estado actual (tal y como está)
</div>

<div id="two-protocols">
  ## Dos protocolos
</div>

<div id="1-gateway-websocket-control-plane">
  ### 1) Gateway WebSocket (plano de control)
</div>

- Superficie completa de la API: configuración, canales, modelos, sesiones, ejecuciones de agentes, registros, nodos, etc.
- Enlace predeterminado: loopback. Acceso remoto mediante SSH/Tailscale.
- Autenticación: token/contraseña mediante `connect`.
- Sin TLS pinning (se basa en loopback/túnel).
- Código:
  - `src/gateway/server/ws-connection/message-handler.ts`
  - `src/gateway/client.ts`
  - `docs/gateway/protocol.md`

<div id="2-bridge-node-transport">
  ### 2) Bridge (transporte de nodo)
</div>

- Superficie de lista de permitidos reducida; identidad de nodo + emparejamiento.
- JSONL sobre TCP; TLS opcional + fijación de huella digital del certificado.
- TLS anuncia la huella digital en el TXT de descubrimiento.
- Código:
  - `src/infra/bridge/server/connection.ts`
  - `src/gateway/server-bridge.ts`
  - `src/node-host/bridge-client.ts`
  - `docs/gateway/bridge-protocol.md`

<div id="control-plane-clients-today">
  ## Clientes actuales del plano de control
</div>

- CLI → Gateway WS a través de `callGateway` (`src/gateway/call.ts`).
- UI de la app de macOS → Gateway WS (`GatewayConnection`).
- Web Control UI → Gateway WS.
- ACP → Gateway WS.
- El control en el navegador usa su propio servidor de control HTTP.

<div id="nodes-today">
  ## Nodos actualmente
</div>

- La aplicación de macOS en modo nodo se conecta al puente del Gateway (`MacNodeBridgeSession`).
- Las aplicaciones de iOS/Android se conectan al puente del Gateway.
- Emparejamiento + token por nodo almacenado en el Gateway.

<div id="current-approval-flow-exec">
  ## Flujo de aprobación actual (exec)
</div>

- El agente usa `system.run` a través del Gateway.
- El Gateway invoca al nodo a través del bridge.
- El runtime del nodo decide la aprobación.
- La app de macOS muestra un aviso en la UI (cuando node == mac app).
- El nodo devuelve `invoke-res` al Gateway.
- Multisalto; UI ligada al host del nodo.

<div id="presence-identity-today">
  ## Presencia e identidad hoy
</div>

- Entradas de presencia del Gateway desde clientes WS.
- Entradas de presencia de nodos desde el bridge.
- La app de macOS puede mostrar dos entradas para la misma máquina (UI + nodo).
- Identidad del nodo almacenada en el almacén de emparejamiento; identidad de la UI por separado.

---

<div id="problems-pain-points">
  # Problemas / puntos problemáticos
</div>

- Dos pilas de protocolos que mantener (WS + Bridge).
- Aprobaciones en nodos remotos: el mensaje aparece en el host del nodo, no donde está el usuario.
- El pinning de TLS solo existe para Bridge; WS depende de SSH/Tailscale.
- Duplicación de identidad: la misma máquina aparece como múltiples instancias.
- Roles ambiguos: las capacidades de la UI, del nodo y de la CLI no están claramente diferenciadas.

---

<div id="proposed-new-state-clawnet">
  # Nuevo estado propuesto (Clawnet)
</div>

<div id="one-protocol-two-roles">
  ## Un protocolo, dos roles
</div>

Protocolo WS único con rol + ámbito.

- **Rol: nodo** (host de capacidades)
- **Rol: operador** (plano de control)
- **scope** opcional para el operador:
  - `operator.read` (estado + visualización)
  - `operator.write` (ejecución de agentes, envío de mensajes)
  - `operator.admin` (configuración, canales, modelos)

<div id="role-behaviors">
  ### Comportamientos de roles
</div>

**Nodo**

- Puede registrar capacidades (`caps`, `commands`, permisos).
- Puede recibir comandos `invoke` (`system.run`, `camera.*`, `canvas.*`, `screen.record`, etc.).
- Puede enviar eventos: `voice.transcript`, `agent.request`, `chat.subscribe`.
- No puede llamar a las APIs del plano de control de config/models/channels/sessions/agent.

**Operador**

- Acceso completo a la API del plano de control, restringido por ámbito.
- Recibe todas las aprobaciones.
- No ejecuta directamente acciones del sistema operativo; las enruta a nodos.

<div id="key-rule">
  ### Regla clave
</div>

El rol se define por conexión, no por dispositivo. Un dispositivo puede desempeñar ambos roles por separado.

---

<div id="unified-authentication-pairing">
  # Autenticación y emparejamiento unificados
</div>

<div id="client-identity">
  ## Identidad del cliente
</div>

Cada cliente proporciona:

- `deviceId` (estable, derivado de la clave del dispositivo).
- `displayName` (nombre para mostrar).
- `role` + `scope` + `caps` + `commands`.

<div id="pairing-flow-unified">
  ## Flujo de emparejamiento (unificado)
</div>

- El cliente se conecta sin autenticarse.
- El Gateway crea una **solicitud de emparejamiento** para ese `deviceId`.
- El operador recibe un aviso y aprueba o deniega.
- El Gateway emite credenciales vinculadas a:
  - la clave pública del dispositivo
  - rol(es)
  - ámbito(s)
  - capacidades/comandos
- El cliente guarda el token y se vuelve a conectar ya autenticado.

<div id="devicebound-auth-avoid-bearer-token-replay">
  ## Autenticación vinculada al dispositivo (evitar la reutilización de tokens bearer)
</div>

Opción recomendada: pares de claves de dispositivo.

- El dispositivo genera el par de claves una sola vez.
- `deviceId = fingerprint(publicKey)`.
- Gateway envía un nonce; el dispositivo lo firma; Gateway lo verifica.
- Los tokens se emiten vinculados a una clave pública (prueba de posesión), no a una cadena de texto.

Alternativas:

- mTLS (certificados de cliente): opción más robusta, con mayor complejidad operativa.
- Tokens bearer de corta duración solo como fase temporal (rotarlos y revocarlos pronto).

<div id="silent-approval-ssh-heuristic">
  ## Aprobación silenciosa (heurística SSH)
</div>

Defínela con precisión para evitar un eslabón débil. Elige una de estas opciones:

- **Solo local**: empareja automáticamente cuando el cliente se conecta vía loopback/Unix socket.
- **Desafío vía SSH**: el Gateway genera un nonce; el cliente demuestra control de SSH recuperándolo.
- **Ventana de presencia física**: después de una aprobación local en la UI del host del Gateway, permite el auto‑emparejamiento durante una ventana breve (p. ej., 10 minutos).

Registra y deja constancia siempre de las aprobaciones automáticas.

---

<div id="tls-everywhere-dev-prod">
  # TLS en todas partes (desarrollo + producción)
</div>

<div id="reuse-existing-bridge-tls">
  ## Reutiliza el TLS del bridge existente
</div>

Usa el runtime TLS actual más fijación de huella digital:

- `src/infra/bridge/server/tls.ts`
- lógica de verificación de huella digital en `src/node-host/bridge-client.ts`

<div id="apply-to-ws">
  ## Aplicar a WS
</div>

- El servidor WS admite TLS con el mismo certificado/clave + huella digital.
- Los clientes WS pueden fijar la huella digital (opcional).
- Discovery anuncia TLS + huella digital para todos los endpoints.
  - Discovery solo proporciona indicios de localización; nunca es un ancla de confianza.

<div id="why">
  ## Por qué
</div>

- Reducir la dependencia de SSH/Tailscale para garantizar la confidencialidad.
- Hacer que las conexiones móviles remotas sean seguras por defecto.

---

<div id="approvals-redesign-centralized">
  # Rediseño del sistema de aprobaciones (centralizado)
</div>

<div id="current">
  ## Estado actual
</div>

La aprobación se realiza en el host del nodo (entorno de ejecución del nodo de la app de macOS). El diálogo aparece en el equipo donde se ejecuta el nodo.

<div id="proposed">
  ## Propuesto
</div>

La aprobación está **alojada en el Gateway** y la UI se entrega a los clientes de los operadores.

<div id="new-flow">
  ### Nuevo flujo
</div>

1) Gateway recibe el intent `system.run` (agente).
2) Gateway crea un registro de aprobación: `approval.requested`.
3) Las UI(s) de operador muestran el cuadro de diálogo.
4) La decisión de aprobación se envía a Gateway: `approval.resolve`.
5) Gateway invoca el comando del nodo si se aprueba.
6) El nodo ejecuta y devuelve `invoke-res`.

<div id="approval-semantics-hardening">
  ### Semántica de aprobación (hardening)
</div>

- Se difunde a todos los operadores; solo la UI activa muestra un modal (las demás reciben un toast).
- La primera resolución prevalece; el Gateway rechaza las resoluciones posteriores por estar ya resueltas.
- Tiempo de espera predeterminado: denegar después de N segundos (p. ej., 60s) y registrar el motivo.
- La resolución requiere el `scope` `operator.approvals`.

<div id="benefits">
  ## Beneficios
</div>

- El prompt aparece donde esté el usuario (Mac/teléfono).
- Aprobaciones coherentes para nodos remotos.
- El entorno de ejecución del nodo se mantiene sin interfaz gráfica; sin dependencia de la UI.

---

<div id="role-clarity-examples">
  # Ejemplos de claridad en los roles
</div>

<div id="iphone-app">
  ## App para iPhone
</div>

- **Rol de nodo** para micrófono, cámara, chat de voz, ubicación y función de pulsar para hablar.
- **operator.read** opcional para estado y vista de chat.
- **operator.write/admin** opcional solo cuando se habilita explícitamente.

<div id="macos-app">
  ## Aplicación de macOS
</div>

- Rol de operador por defecto (Control UI).
- Rol de nodo cuando “Mac node” está activado (system.run, screen, camera).
- El mismo deviceId para ambas conexiones → entrada combinada en la UI.

<div id="cli">
  ## CLI
</div>

- Siempre con rol de operador.
- Ámbito derivado según el subcomando:
  - `status`, `logs` → read
  - `agent`, `message` → write
  - `config`, `channels` → admin
  - aprobaciones + emparejamiento → `operator.approvals` / `operator.pairing`

---

<div id="identity-slugs">
  # Identidad y slugs
</div>

<div id="stable-id">
  ## ID estable
</div>

Requerido para la autenticación; no cambia nunca.
Recomendado:

- Huella del par de claves (hash de la clave pública).

<div id="cute-slug-lobsterthemed">
  ## Slug simpático (con temática de langosta)
</div>

Solo etiqueta para humanos.

- Ejemplo: `scarlet-claw`, `saltwave`, `mantis-pinch`.
- Almacenado en el registro del Gateway, editable.
- Manejo de colisiones: `-2`, `-3`.

<div id="ui-grouping">
  ## Agrupación en la UI
</div>

Mismo `deviceId` en todos los roles → una sola fila de “Instancia”:

- Insignia: `operator`, `nodo`.
- Muestra las capacidades y la última vez en línea.

---

<div id="migration-strategy">
  # Estrategia de migración
</div>

<div id="phase-0-document-align">
  ## Fase 0: Documentar y alinear
</div>

- Publica este documento.
- Haz un inventario de todas las llamadas de protocolo y flujos de aprobación.

<div id="phase-1-add-rolesscopes-to-ws">
  ## Fase 1: Agregar roles/ámbitos a WS
</div>

- Extender los parámetros de `connect` con `role`, `scope`, `deviceId`.
- Agregar control de acceso mediante lista de permitidos para el rol de nodo.

<div id="phase-2-bridge-compatibility">
  ## Fase 2: Compatibilidad del bridge
</div>

- Mantén el bridge en funcionamiento.
- Añade compatibilidad para nodos WS en paralelo.
- Controla las funciones mediante un flag de configuración.

<div id="phase-3-central-approvals">
  ## Fase 3: Aprobaciones centrales
</div>

- Añadir eventos de solicitud de aprobación + resolución en WS.
- Actualizar la UI de la app de macOS para solicitar + responder.
- El runtime del nodo deja de generar solicitudes en la UI.

<div id="phase-4-tls-unification">
  ## Fase 4: unificación de TLS
</div>

- Añade configuración de TLS para WS usando el runtime TLS de bridge.
- Implementa pinning en los clientes.

<div id="phase-5-deprecate-bridge">
  ## Fase 5: Marcar bridge como obsoleto
</div>

- Migrar el nodo para iOS/Android/mac a WS.
- Mantener bridge como mecanismo de respaldo; eliminarlo cuando sea estable.

<div id="phase-6-devicebound-auth">
  ## Fase 6: Autenticación vinculada al dispositivo
</div>

- Exigir identidad basada en clave para todas las conexiones remotas.
- Agregar UI de revocación y rotación.

---

<div id="security-notes">
  # Notas de seguridad
</div>

- Rol/lista de permitidos aplicados en el perímetro del Gateway.
- Ningún cliente obtiene acceso “completo” a la API sin ámbito de operador.
- Emparejamiento obligatorio para *todas* las conexiones.
- TLS + pinning reduce el riesgo de ataques MITM en dispositivos móviles.
- La aprobación silenciosa por SSH es una comodidad; sigue quedando registrada y se puede revocar.
- Discovery nunca es un ancla de confianza.
- Las declaraciones de capacidades se verifican contra las listas de permitidos del servidor según plataforma/tipo.

<div id="streaming-large-payloads-node-media">
  # Streaming y cargas útiles grandes (medios del nodo)
</div>

El plano de control WS está bien para mensajes pequeños, pero los nodos también manejan:

- clips de cámara
- grabaciones de pantalla
- transmisiones de audio

Opciones:

1) Tramas binarias de WS + fragmentación (chunking) + reglas de backpressure.
2) Endpoint de streaming separado (siempre con TLS + autenticación).
3) Mantener el puente activo más tiempo para comandos muy intensivos en contenido multimedia y migrarlos los últimos.

Elige una opción antes de la implementación para evitar desviaciones.

<div id="capability-command-policy">
  # Política de capacidades y comandos
</div>

- Las capacidades y comandos informados por el nodo se consideran **declaraciones**.
- Gateway aplica listas de permitidos por plataforma.
- Cualquier comando nuevo requiere aprobación del operador o un cambio explícito en la lista de permitidos.
- Audita los cambios con marcas de tiempo.

<div id="audit-rate-limiting">
  # Auditoría + limitación de tasa
</div>

- Registra: solicitudes de emparejamiento, aprobaciones/rechazos, emisión/rotación/revocación de tokens.
- Aplica límites de tasa al spam de emparejamiento y a los prompts de aprobación.

<div id="protocol-hygiene">
  # Higiene del protocolo
</div>

- Versión explícita de protocolo + códigos de error.
- Reglas de reconexión + política de latido.
- TTL de presencia y semántica de «última vez visto».

---

<div id="open-questions">
  # Preguntas abiertas
</div>

1) Un único dispositivo que ejecuta ambos roles: modelo de tokens
   - Se recomiendan tokens separados por rol (nodo vs operador).
   - Mismo deviceId; ámbitos diferentes; revocación más clara.

2) Granularidad del ámbito del operador
   - read/write/admin + approvals + emparejamiento (mínimo viable).
   - Considerar ámbitos por función más adelante.

3) UX de rotación y revocación de tokens
   - Rotación automática al cambiar de rol.
   - UI para revocar por deviceId + rol.

4) Descubrimiento
   - Ampliar el TXT actual de Bonjour para incluir la huella TLS de WS + indicadores de rol.
   - Tratarlo únicamente como pistas de localización.

5) Aprobación entre redes
   - Difundir a todos los clientes de operador; la UI activa muestra un modal.
   - Gana la primera respuesta; el Gateway garantiza la atomicidad.

---

<div id="summary-tldr">
  # Resumen (TL;DR)
</div>

- Hoy: plano de control WS + transporte de nodo Bridge.
- Problema: aprobaciones + duplicación + dos stacks.
- Propuesta: un protocolo WS con roles explícitos + ámbitos, emparejamiento unificado + pinning de TLS, aprobaciones alojadas en el Gateway, IDs de dispositivo estables + slugs simpáticos.
- Resultado: UX más simple, seguridad más sólida, menos duplicación, mejor enrutamiento móvil.