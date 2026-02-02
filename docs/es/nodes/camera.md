---
title: Cámara
summary: "Captura de cámara (nodo iOS + app de macOS) para uso de agentes: fotos (jpg) y clips de vídeo cortos (mp4)"
read_when:
  - Agregar o modificar la captura de cámara en nodos iOS o macOS
  - Ampliar flujos de trabajo de archivos temporales MEDIA accesibles para agentes
---

<div id="camera-capture-agent">
  # Captura de cámara (agente)
</div>

OpenClaw admite **captura con cámara** para flujos de trabajo de agentes:

- **nodo iOS** (emparejado a través del Gateway): captura una **foto** (`jpg`) o un **clip de vídeo corto** (`mp4`, con audio opcional) mediante `node.invoke`.
- **nodo Android** (emparejado a través del Gateway): captura una **foto** (`jpg`) o un **clip de vídeo corto** (`mp4`, con audio opcional) mediante `node.invoke`.
- **app de macOS** (nodo a través del Gateway): captura una **foto** (`jpg`) o un **clip de vídeo corto** (`mp4`, con audio opcional) mediante `node.invoke`.

Todo acceso a la cámara está protegido por **configuraciones controladas por el usuario**.

<div id="ios-node">
  ## Nodo de iOS
</div>

<div id="user-setting-default-on">
  ### Configuración del usuario (activada de forma predeterminada)
</div>

- Pestaña **Settings** de iOS → **Camera** → **Allow Camera** (`camera.enabled`)
  - Valor predeterminado: **activado** (si falta la clave, se considera habilitada).
  - Cuando está desactivada: los comandos `camera.*` devuelven `CAMERA_DISABLED`.

<div id="commands-via-gateway-nodeinvoke">
  ### Comandos (a través del Gateway `node.invoke`)
</div>

- `camera.list`
  - Carga útil de la respuesta:
    - `devices`: array de `{ id, name, position, deviceType }`

- `camera.snap`
  - Parámetros:
    - `facing`: `front|back` (valor predeterminado: `front`)
    - `maxWidth`: número (opcional; valor predeterminado `1600` en el nodo iOS)
    - `quality`: `0..1` (opcional; valor predeterminado `0.9`)
    - `format`: actualmente `jpg`
    - `delayMs`: número (opcional; valor predeterminado `0`)
    - `deviceId`: cadena (opcional; de `camera.list`)
  - Carga útil de la respuesta:
    - `format: "jpg"`
    - `base64: "<...>"`
    - `width`, `height`
  - Límite de carga útil: las fotos se recomprimen para mantener el payload base64 por debajo de 5 MB.

- `camera.clip`
  - Parámetros:
    - `facing`: `front|back` (valor predeterminado: `front`)
    - `durationMs`: número (valor predeterminado `3000`, limitado a un máximo de `60000`)
    - `includeAudio`: booleano (valor predeterminado `true`)
    - `format`: actualmente `mp4`
    - `deviceId`: cadena (opcional; de `camera.list`)
  - Carga útil de la respuesta:
    - `format: "mp4"`
    - `base64: "<...>"`
    - `durationMs`
    - `hasAudio`

<div id="foreground-requirement">
  ### Requisito de ejecución en primer plano
</div>

Al igual que `canvas.*`, el nodo de iOS solo permite comandos `camera.*` en **primer plano**. Las invocaciones en segundo plano devuelven `NODE_BACKGROUND_UNAVAILABLE`.

<div id="cli-helper-temp-files-media">
  ### Asistente de CLI (archivos temporales + MEDIA)
</div>

La manera más sencilla de obtener archivos adjuntos es mediante el asistente de CLI, que escribe el contenido multimedia decodificado en un archivo temporal e imprime `MEDIA:<path>`.

Ejemplos:

```bash
openclaw nodes camera snap --node <id>               # predeterminado: frontal y trasera (2 líneas MEDIA)
openclaw nodes camera snap --node <id> --facing front
openclaw nodes camera clip --node <id> --duration 3000
openclaw nodes camera clip --node <id> --no-audio
```

Notas:

* `nodes camera snap` usa por defecto **ambas** cámaras (frontal y trasera) para darle al agente ambas vistas.
* Los archivos de salida son temporales (en el directorio temporal del sistema operativo) a menos que implementes tu propio wrapper.


<div id="android-node">
  ## Nodo de Android
</div>

### Configuración de usuario (activado por defecto)

- Hoja de configuración de Android → **Camera** → **Allow Camera** (`camera.enabled`)
  - Predeterminado: **activado** (la ausencia de la clave se considera como habilitado).
  - Cuando está desactivado: los comandos `camera.*` devuelven `CAMERA_DISABLED`.

<div id="permissions">
  ### Permisos
</div>

- Android requiere permisos en tiempo de ejecución:
  - `CAMERA` tanto para `camera.snap` como para `camera.clip`.
  - `RECORD_AUDIO` para `camera.clip` cuando `includeAudio=true`.

Si faltan permisos, la aplicación mostrará un aviso cuando sea posible; si se deniegan, las solicitudes `camera.*` fallarán con un error
`*_PERMISSION_REQUIRED`.

### Requisito de primer plano

Al igual que `canvas.*`, el nodo de Android solo permite comandos `camera.*` en **primer plano**. Las invocaciones en segundo plano devuelven `NODE_BACKGROUND_UNAVAILABLE`.

<div id="payload-guard">
  ### Protección del payload
</div>

Las fotos se recomprimen para mantener el payload base64 por debajo de 5 MB.

<div id="macos-app">
  ## Aplicación para macOS
</div>

<div id="user-setting-default-off">
  ### Configuración de usuario (desactivada de forma predeterminada)
</div>

La app complementaria de macOS muestra una casilla de verificación:

- **Settings → General → Allow Camera** (`openclaw.cameraEnabled`)
  - Predeterminado: **off**
  - Cuando está desactivada, las solicitudes de acceso a la cámara devuelven “Camera disabled by user”.

<div id="cli-helper-node-invoke">
  ### Utilidad de CLI (invocar nodo)
</div>

Usa la CLI principal de `openclaw` para ejecutar comandos de cámara en el nodo macOS.

Ejemplos:

```bash
openclaw nodes camera list --node <id>            # list camera ids
openclaw nodes camera snap --node <id>            # prints MEDIA:<path>
openclaw nodes camera snap --node <id> --max-width 1280
openclaw nodes camera snap --node <id> --delay-ms 2000
openclaw nodes camera snap --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --duration 10s          # prints MEDIA:<path>
openclaw nodes camera clip --node <id> --duration-ms 3000      # imprime MEDIA:<ruta> (flag heredado)
openclaw nodes camera clip --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --no-audio
```

Notas:

* `openclaw nodes camera snap` tiene como valor predeterminado `maxWidth=1600`, a menos que se anule.
* En macOS, `camera.snap` espera `delayMs` (valor predeterminado 2000 ms) tras el calentamiento/estabilización de la exposición antes de capturar.
* Los datos de la foto se recomprimen para mantener el base64 por debajo de 5 MB.


<div id="safety-practical-limits">
  ## Límites de seguridad y prácticos
</div>

- El acceso a la cámara y al micrófono genera las solicitudes de permisos habituales del sistema operativo (y requiere cadenas de uso en Info.plist).
- Los clips de vídeo tienen un límite de duración (actualmente `<= 60s`) para evitar cargas útiles del nodo demasiado grandes (sobrecarga de base64 + límites de mensajes).

<div id="macos-screen-video-os-level">
  ## Grabación de pantalla en macOS (a nivel del sistema operativo)
</div>

Para grabación de *pantalla* (no de cámara), usa la aplicación complementaria de macOS:

```bash
openclaw nodes screen record --node <id> --duration 10s --fps 15   # imprime MEDIA:<ruta>
```

Notas:

* Requiere el permiso de **Grabación de pantalla** de macOS (TCC).
