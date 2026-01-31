---
title: Matrix
summary: "Estado de compatibilidad, capacidades y configuración de Matrix"
read_when:
  - Trabajando en las funciones del canal de Matrix
---

<div id="matrix-plugin">
  # Matrix (complemento)
</div>

Matrix es un protocolo de mensajería abierto y descentralizado. OpenClaw se conecta como un **usuario**
de Matrix en cualquier homeserver, por lo que necesitas una cuenta de Matrix para el bot. Una vez que el bot haya iniciado sesión, puedes enviarle mensajes directos (DM)
o invitarlo a salas (Matrix &quot;groups&quot;). Beeper también es una opción de cliente válida,
pero requiere tener E2EE habilitado.

Estado: compatible vía complemento (@vector-im/matrix-bot-sdk). Mensajes directos, salas, hilos, contenido multimedia, reacciones,
encuestas (send + poll-start como texto), ubicación y E2EE (con soporte de cifrado).

<div id="plugin-required">
  ## Complemento requerido
</div>

Matrix se distribuye como un complemento y no está incluido en la instalación principal.

Instálalo mediante la CLI (registro de npm):

```bash
openclaw plugins install @openclaw/matrix
```

Clon local (cuando se ejecuta desde un repositorio Git):

```bash
openclaw plugins install ./extensions/matrix
```

Si eliges Matrix durante la configuración inicial y se detecta un checkout de Git,
OpenClaw te sugerirá automáticamente la ruta de instalación local.

Más información: [Complementos](/es/plugin)

<div id="setup">
  ## Configuración
</div>

1. Instala el complemento de Matrix:
   * Desde npm: `openclaw plugins install @openclaw/matrix`
   * Desde un checkout local: `openclaw plugins install ./extensions/matrix`
2. Crea una cuenta de Matrix en un homeserver:
   * Revisa las opciones de hosting en [https://matrix.org/ecosystem/hosting/](https://matrix.org/ecosystem/hosting/)
   * O aloja tú mismo el servidor.
3. Obtén un token de acceso para la cuenta del bot:

   * Usa la API de inicio de sesión de Matrix con `curl` en tu homeserver:

   ```bash
   curl --request POST \
     --url https://matrix.example.org/_matrix/client/v3/login \
     --header 'Content-Type: application/json' \
     --data '{
     "type": "m.login.password",
     "identifier": {
       "type": "m.id.user",
       "user": "your-user-name"
     },
     "password": "your-password"
   }'
   ```

   * Sustituye `matrix.example.org` por la URL de tu homeserver.
   * O configura `channels.matrix.userId` + `channels.matrix.password`: OpenClaw llama al mismo
     endpoint de inicio de sesión, almacena el token de acceso en `~/.openclaw/credentials/matrix/credentials.json`,
     y lo reutiliza en el siguiente inicio.
4. Configura las credenciales:
   * Variables de entorno: `MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN` (o `MATRIX_USER_ID` + `MATRIX_PASSWORD`)
   * O config: `channels.matrix.*`
   * Si ambos están establecidos, la configuración tiene prioridad.
   * Con token de acceso: el ID de usuario se obtiene automáticamente mediante `/whoami`.
   * Cuando se configure, `channels.matrix.userId` debe ser el ID completo de Matrix (por ejemplo: `@bot:example.org`).
5. Reinicia el Gateway (o finaliza el onboarding).
6. Inicia un mensaje directo (DM) con el bot o invítalo a una sala desde cualquier cliente de Matrix
   (Element, Beeper, etc.; ver https://matrix.org/ecosystem/clients/). Beeper requiere E2EE,
   así que establece `channels.matrix.encryption: true` y verifica el dispositivo.

Configuración mínima (token de acceso, ID de usuario obtenido automáticamente):

```json5
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      dm: { policy: "emparejamiento" }
    }
  }
}
```

Configuración de E2EE (cifrado de extremo a extremo habilitado):

```json5
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      encryption: true,
      dm: { policy: "pairing" }
    }
  }
}
```

<div id="encryption-e2ee">
  ## Cifrado (E2EE)
</div>

El cifrado de extremo a extremo es **compatible** mediante el SDK de criptografía en Rust.

Activa el cifrado con `channels.matrix.encryption: true`:

* Si el módulo de criptografía se carga, las salas cifradas se descifran automáticamente.
* El contenido multimedia saliente se cifra al enviarlo a salas cifradas.
* En la primera conexión, OpenClaw solicita la verificación del dispositivo a través de tus otras sesiones.
* Verifica el dispositivo en otro cliente Matrix (Element, etc.) para habilitar el intercambio de claves.
* Si el módulo de criptografía no se puede cargar, E2EE se deshabilita y las salas cifradas no se descifrarán;
  OpenClaw registra una advertencia.
* Si ves errores de módulo de criptografía ausente (por ejemplo, `@matrix-org/matrix-sdk-crypto-nodejs-*`),
  permite los scripts de compilación para `@matrix-org/matrix-sdk-crypto-nodejs` y ejecuta
  `pnpm rebuild @matrix-org/matrix-sdk-crypto-nodejs` o descarga el binario con
  `node node_modules/@matrix-org/matrix-sdk-crypto-nodejs/download-lib.js`.

El estado de cifrado se almacena por cuenta + token de acceso en
`~/.openclaw/matrix/accounts/<account>/<homeserver>__<user>/<token-hash>/crypto/`
(base de datos SQLite). El estado de sincronización se almacena junto a este en `bot-storage.json`.
Si el token de acceso (dispositivo) cambia, se crea un nuevo almacén y el bot debe volver a
verificarse para las salas cifradas.

**Verificación de dispositivo:**
Cuando E2EE está habilitado, el bot solicitará verificación desde tus otras sesiones al arrancar.
Abre Element (u otro cliente) y aprueba la solicitud de verificación para establecer confianza.
Una vez verificado, el bot puede descifrar mensajes en salas cifradas.

<div id="routing-model">
  ## Modelo de enrutamiento
</div>

* Las respuestas siempre regresan a Matrix.
* Los mensajes directos comparten la sesión principal del agente; las salas se asignan a sesiones de grupo.

<div id="access-control-dms">
  ## Control de acceso (DMs)
</div>

* Valor predeterminado: `channels.matrix.dm.policy = "pairing"`. Los remitentes desconocidos reciben un código de emparejamiento.
* Aprobar mediante:
  * `openclaw pairing list matrix`
  * `openclaw pairing approve matrix <CODE>`
* DMs públicas: `channels.matrix.dm.policy="open"` junto con `channels.matrix.dm.allowFrom=["*"]`.
* `channels.matrix.dm.allowFrom` acepta IDs de usuario o nombres para mostrar. El asistente resuelve los nombres para mostrar a IDs de usuario cuando la búsqueda en el directorio está disponible.

<div id="rooms-groups">
  ## Rooms (grupos)
</div>

* Por defecto: `channels.matrix.groupPolicy = "allowlist"` (restringido a menciones). Usa `channels.defaults.groupPolicy` para sobrescribir el valor por defecto cuando no esté definido.
* Añade rooms a la lista de permitidos con `channels.matrix.groups` (IDs de rooms, alias o nombres):

```json5
{
  channels: {
    matrix: {
      groupPolicy: "allowlist",
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true }
      },
      groupAllowFrom: ["@owner:example.org"]
    }
  }
}
```

* `requireMention: false` habilita la respuesta automática en esa sala.
* `groups."*"` puede establecer valores predeterminados para el control mediante menciones en todas las salas.
* `groupAllowFrom` restringe qué remitentes pueden activar el bot en las salas (opcional).
* Las listas de permitidos `users` por sala pueden restringir aún más los remitentes dentro de una sala específica.
* El asistente de configuración solicita listas de permitidos de salas (ID de sala, alias o nombres) y resuelve los nombres cuando es posible.
* Al inicio, OpenClaw resuelve los nombres de salas/usuarios en las listas de permitidos a ID y registra la correspondencia en los logs; las entradas no resueltas se mantienen tal como se introdujeron.
* Las invitaciones se aceptan automáticamente de forma predeterminada; contrólalo con `channels.matrix.autoJoin` y `channels.matrix.autoJoinAllowlist`.
* Para no permitir **ninguna sala**, establece `channels.matrix.groupPolicy: "disabled"` (o deja una lista de permitidos vacía).
* Clave obsoleta: `channels.matrix.rooms` (misma estructura que `groups`).

<div id="threads">
  ## Hilos
</div>

* Se admiten las respuestas en hilos.
* `channels.matrix.threadReplies` controla si las respuestas permanecen en hilos:
  * `off`, `inbound` (predeterminado), `always`
* `channels.matrix.replyToMode` controla los metadatos de respuesta cuando no se responde en un hilo:
  * `off` (predeterminado), `first`, `all`

<div id="capabilities">
  ## Capacidades
</div>

| Función | Estado |
|---------|--------|
| Mensajes directos | ✅ Disponible |
| Salas | ✅ Disponible |
| Hilos | ✅ Disponible |
| Contenido multimedia | ✅ Disponible |
| E2EE | ✅ Disponible (requiere módulo criptográfico) |
| Reacciones | ✅ Disponible (send/read mediante herramientas) |
| Encuestas | ✅ Se admite el envío; las encuestas entrantes al iniciarse se convierten en texto (se ignoran las respuestas/finalizaciones) |
| Ubicación | ✅ Disponible (URI geo; se ignora la altitud) |
| Comandos nativos | ✅ Disponible |

<div id="configuration-reference-matrix">
  ## Referencia de configuración (Matrix)
</div>

Configuración completa: [Configuration](/es/gateway/configuration)

Opciones del proveedor:

* `channels.matrix.enabled`: habilitar/deshabilitar el inicio del canal.
* `channels.matrix.homeserver`: URL del homeserver.
* `channels.matrix.userId`: ID de usuario de Matrix (opcional con access token).
* `channels.matrix.accessToken`: access token.
* `channels.matrix.password`: contraseña para el inicio de sesión (token almacenado).
* `channels.matrix.deviceName`: nombre visible del dispositivo.
* `channels.matrix.encryption`: habilitar E2EE (predeterminado: false).
* `channels.matrix.initialSyncLimit`: límite de sincronización inicial.
* `channels.matrix.threadReplies`: `off | inbound | always` (predeterminado: inbound).
* `channels.matrix.textChunkLimit`: tamaño de fragmento de texto saliente (caracteres).
* `channels.matrix.chunkMode`: `length` (predeterminado) o `newline` para dividir en líneas en blanco (límites de párrafo) antes de fragmentar por longitud.
* `channels.matrix.dm.policy`: `pairing | allowlist | open | disabled` (predeterminado: pairing).
* `channels.matrix.dm.allowFrom`: lista de permitidos de DM (IDs de usuario o nombres visibles). `open` requiere `"*"`. El asistente resuelve nombres a IDs cuando es posible.
* `channels.matrix.groupPolicy`: `allowlist | open | disabled` (predeterminado: allowlist).
* `channels.matrix.groupAllowFrom`: remitentes en la lista de permitidos para mensajes de grupo.
* `channels.matrix.allowlistOnly`: forzar reglas de lista de permitidos para DMs y salas.
* `channels.matrix.groups`: lista de permitidos de grupos + mapa de configuración por sala.
* `channels.matrix.rooms`: configuración/lista de permitidos de grupos heredada.
* `channels.matrix.replyToMode`: modo de respuesta para hilos/etiquetas.
* `channels.matrix.mediaMaxMb`: límite de medios entrantes/salientes (MB).
* `channels.matrix.autoJoin`: manejo de invitaciones (`always | allowlist | off`, predeterminado: always).
* `channels.matrix.autoJoinAllowlist`: IDs/alias de salas permitidos para unirse automáticamente.
* `channels.matrix.actions`: control de acceso por acción de herramientas (reacciones/mensajes/pins/memberInfo/channelInfo).