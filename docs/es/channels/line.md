---
title: LINE
summary: "Instalación, configuración y uso del complemento LINE Messaging API"
read_when:
  - Quieres conectar OpenClaw con LINE
  - Necesitas configurar el webhook de LINE y sus credenciales
  - Quieres opciones de mensajes específicas de LINE
---

<div id="line-plugin">
  # LINE (complemento)
</div>

LINE se conecta a OpenClaw a través de la LINE Messaging API. El complemento se ejecuta como un receptor de webhooks
en el Gateway y usa tu channel access token + channel secret para la
autenticación.

Estado: compatible mediante complemento. Se admiten mensajes directos, chats de grupo, contenido multimedia, ubicaciones, mensajes Flex,
mensajes de plantilla y respuestas rápidas. No se admiten reacciones ni hilos.

<div id="plugin-required">
  ## Complemento necesario
</div>

Instala el complemento LINE:

```bash
openclaw plugins install @openclaw/line
```

Copia de trabajo local (al ejecutar desde un repositorio Git):

```bash
openclaw plugins install ./extensions/line
```


<div id="setup">
  ## Configuración
</div>

1. Crea una cuenta de LINE Developers y abre la consola:
   https://developers.line.biz/console/
2. Crea (o selecciona) un proveedor y añade un canal de **Messaging API**.
3. Copia el **Channel access token** y el **Channel secret** de la configuración del canal.
4. Activa **Use webhook** en la configuración de **Messaging API**.
5. Establece la URL del webhook en el endpoint de tu Gateway (HTTPS requerido):

```
https://gateway-host/line/webhook
```

El Gateway responde a la verificación del webhook de LINE (GET) y a los eventos entrantes (POST).
Si necesitas una ruta personalizada, configura `channels.line.webhookPath` o
`channels.line.accounts.<id>.webhookPath` y actualiza la URL en consecuencia.


<div id="configure">
  ## Configurar
</div>

Configuración mínima:

```json5
{
  channels: {
    line: {
      enabled: true,
      channelAccessToken: "LINE_CHANNEL_ACCESS_TOKEN",
      channelSecret: "LINE_CHANNEL_SECRET",
      dmPolicy: "pairing"
    }
  }
}
```

Variables de entorno (solo cuenta predeterminada):

* `LINE_CHANNEL_ACCESS_TOKEN`
* `LINE_CHANNEL_SECRET`

Archivos de token y secreto:

```json5
{
  channels: {
    line: {
      tokenFile: "/path/to/line-token.txt",
      secretFile: "/path/to/line-secret.txt"
    }
  }
}
```

Varias cuentas:

```json5
{
  channels: {
    line: {
      accounts: {
        marketing: {
          channelAccessToken: "...",
          channelSecret: "...",
          webhookPath: "/line/marketing"
        }
      }
    }
  }
}
```


<div id="access-control">
  ## Control de acceso
</div>

De forma predeterminada, los mensajes directos requieren emparejamiento. Los remitentes desconocidos reciben un código de emparejamiento y sus mensajes se ignoran hasta que son aprobados.

```bash
openclaw pairing list line
openclaw pairing approve line <CODE>
```

Listas de permitidos y políticas:

* `channels.line.dmPolicy`: `pairing | allowlist | open | disabled`
* `channels.line.allowFrom`: IDs de usuario de LINE permitidos para DMs
* `channels.line.groupPolicy`: `allowlist | open | disabled`
* `channels.line.groupAllowFrom`: IDs de usuario de LINE permitidos para grupos
* Anulaciones por grupo: `channels.line.groups.<groupId>.allowFrom`

Los IDs de LINE son sensibles a mayúsculas y minúsculas. Los IDs válidos tienen el siguiente formato:

* Usuario: `U` + 32 caracteres hexadecimales
* Grupo: `C` + 32 caracteres hexadecimales
* Sala: `R` + 32 caracteres hexadecimales


<div id="message-behavior">
  ## Comportamiento de los mensajes
</div>

- El texto se divide en bloques de 5000 caracteres.
- El formato Markdown se elimina; los bloques de código y las tablas se convierten en tarjetas Flex
  cuando sea posible.
- Las respuestas en streaming se almacenan en búfer; LINE recibe bloques completos con una animación
  de carga mientras el agente trabaja.
- Las descargas de medios se limitan mediante `channels.line.mediaMaxMb` (valor predeterminado: 10).

<div id="channel-data-rich-messages">
  ## Datos del canal (mensajes enriquecidos)
</div>

Usa `channelData.line` para enviar respuestas rápidas, ubicaciones, tarjetas Flex o mensajes de plantilla.

```json5
{
  text: "Here you go",
  channelData: {
    line: {
      quickReplies: ["Status", "Help"],
      location: {
        title: "Office",
        address: "123 Main St",
        latitude: 35.681236,
        longitude: 139.767125
      },
      flexMessage: {
        altText: "Status card",
        contents: { /* Flex payload */ }
      },
      templateMessage: {
        type: "confirm",
        text: "Proceed?",
        confirmLabel: "Yes",
        confirmData: "yes",
        cancelLabel: "No",
        cancelData: "no"
      }
    }
  }
}
```

El complemento LINE también incluye un comando `/card` para plantillas de mensajes Flex:

```
/card info "Welcome" "Thanks for joining!"
```


<div id="troubleshooting">
  ## Solución de problemas
</div>

- **La verificación del webhook falla:** asegúrate de que la URL del webhook sea HTTPS y de que `channelSecret` coincida con la de la consola de LINE.
- **No hay eventos entrantes:** confirma que la ruta del webhook coincida con `channels.line.webhookPath` y que el Gateway sea accesible desde LINE.
- **Errores al descargar contenido multimedia:** aumenta `channels.line.mediaMaxMb` si el contenido multimedia supera el límite predeterminado.