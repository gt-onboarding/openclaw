---
title: Emparejamiento
summary: "Descripción general del emparejamiento: define quién puede enviarte mensajes directos (DM) y qué nodos pueden unirse"
read_when:
  - Configurar el control de acceso a mensajes directos (DM)
  - Emparejar un nuevo nodo iOS/Android
  - Revisar la postura de seguridad de OpenClaw
---

<div id="pairing">
  # Emparejamiento
</div>

El “emparejamiento” es la etapa explícita de **aprobación por parte del propietario** en OpenClaw.
Se utiliza en dos casos:

1. **Emparejamiento por DM** (quién puede hablar con el bot)
2. **Emparejamiento de nodos** (qué dispositivos/nodos pueden unirse a la red del Gateway)

Contexto de seguridad: [Seguridad](/es/gateway/security)

<div id="1-dm-pairing-inbound-chat-access">
  ## 1) Emparejamiento por DM (acceso de chat entrante)
</div>

Cuando configuras un canal con la política de DM `pairing`, los remitentes desconocidos reciben un código corto y su mensaje **no se procesa** hasta que lo apruebes.

Las políticas de DM predeterminadas están documentadas en: [Seguridad](/es/gateway/security)

Códigos de emparejamiento:

* 8 caracteres en mayúsculas, sin caracteres ambiguos (`0O1I`).
* **Expiran después de 1 hora**. El bot solo envía el mensaje de emparejamiento cuando se crea una nueva solicitud (aproximadamente una vez por hora y por remitente).
* Las solicitudes de emparejamiento de DM pendientes se limitan a **3 por canal** de forma predeterminada; las solicitudes adicionales se ignoran hasta que una expire o se apruebe.

<div id="approve-a-sender">
  ### Aprobar un remitente
</div>

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

Canales compatibles: `telegram`, `whatsapp`, `signal`, `imessage`, `discord`, `slack`.

<div id="where-the-state-lives">
  ### Dónde se almacena el estado
</div>

Se guarda en `~/.openclaw/credentials/`:

* Solicitudes pendientes: `<channel>-pairing.json`
* Almacén de la lista de permitidos aprobada: `<channel>-allowFrom.json`

Trata estos archivos como datos confidenciales (controlan el acceso a tu asistente).

<div id="2-node-device-pairing-iosandroidmacosheadless-nodes">
  ## 2) Emparejamiento de dispositivos de nodo (nodos iOS/Android/macOS/sin interfaz gráfica)
</div>

Los nodos se conectan al Gateway como **dispositivos** con `role: node`. El Gateway
crea una solicitud de emparejamiento de dispositivo que debe ser aprobada.

<div id="approve-a-node-device">
  ### Aprobar un dispositivo nodo
</div>

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

<div id="where-the-state-lives">
  ### Dónde se almacena el estado
</div>

Se almacena en `~/.openclaw/devices/`:

* `pending.json` (de corta duración; las solicitudes en espera expiran)
* `paired.json` (dispositivos emparejados + tokens)

<div id="notes">
  ### Notas
</div>

* La API antigua `node.pair.*` (CLI: `openclaw nodes pending/approve`) es un
  almacén de emparejamientos independiente gestionado por el Gateway. Los nodos WS siguen necesitando emparejamiento de dispositivo.

<div id="related-docs">
  ## Documentación relacionada
</div>

* Modelo de seguridad + inyección de prompts: [Security](/es/gateway/security)
* Actualización segura (ejecutar doctor): [Updating](/es/install/updating)
* Configuración de canales:
  * Telegram: [Telegram](/es/channels/telegram)
  * WhatsApp: [WhatsApp](/es/channels/whatsapp)
  * Signal: [Signal](/es/channels/signal)
  * iMessage: [iMessage](/es/channels/imessage)
  * Discord: [Discord](/es/channels/discord)
  * Slack: [Slack](/es/channels/slack)