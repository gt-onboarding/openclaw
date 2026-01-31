---
title: Activación por voz
summary: "Palabras de activación por voz globales (propiedad del Gateway) y cómo se sincronizan entre nodos"
read_when:
  - Cambiar el comportamiento o la configuración predeterminada de las palabras de activación por voz
  - Incorporar nuevas plataformas de nodo que requieran sincronización de palabras de activación por voz
---

<div id="voice-wake-global-wake-words">
  # Activación por voz (palabras de activación globales)
</div>

OpenClaw trata **las palabras de activación como una única lista global** propiedad del **Gateway**.

- **No hay palabras de activación personalizadas por nodo**.
- **Cualquier UI de nodo/app puede modificar** la lista; el Gateway guarda los cambios y los difunde a todos.
- Cada dispositivo sigue manteniendo su propio interruptor de **Activación por voz habilitada/deshabilitada** (la UX local y los permisos difieren).

<div id="storage-gateway-host">
  ## Almacenamiento (host del Gateway)
</div>

Las palabras de activación se almacenan en el host donde se ejecuta el Gateway:

* `~/.openclaw/settings/voicewake.json`

Estructura:

```json
{ "triggers": ["openclaw", "claude", "computer"], "updatedAtMs": 1730000000000 }
```


<div id="protocol">
  ## Protocolo
</div>

<div id="methods">
  ### Métodos
</div>

- `voicewake.get` → `{ triggers: string[] }`
- `voicewake.set` con parámetros `{ triggers: string[] }` → `{ triggers: string[] }`

Notas:

- Los triggers se normalizan (se recortan y se eliminan los vacíos). Las listas vacías vuelven a los valores predeterminados.
- Se aplican límites por seguridad (límites máximos de cantidad y de longitud).

<div id="events">
  ### Eventos
</div>

- `voicewake.changed` payload `{ triggers: string[] }`

Quién lo recibe:

- Todos los clientes WebSocket (app de macOS, WebChat, etc.)
- Todos los nodos conectados (iOS/Android) y también al conectarse el nodo, como envío inicial del estado actual.

<div id="client-behavior">
  ## Comportamiento del cliente
</div>

<div id="macos-app">
  ### aplicación de macOS
</div>

- Usa la lista global para controlar los disparadores de `VoiceWakeRuntime`.
- Al editar “Trigger words” en la configuración de Voice Wake se llama a `voicewake.set` y luego se depende de la difusión para mantener sincronizados a los demás clientes.

<div id="ios-node">
  ### nodo de iOS
</div>

- Utiliza la lista global para la detección de activadores de `VoiceWakeManager`.
- Editar las palabras de activación en Configuración llama a `voicewake.set` (a través del WS del Gateway) y también mantiene receptiva la detección local de palabras de activación.

<div id="android-node">
  ### Nodo Android
</div>

- Expone un editor de Wake Words en Configuración.
- Invoca `voicewake.set` a través del WS del Gateway para que los cambios se sincronicen en todos los dispositivos.