---
title: Tlon
summary: "Estado de soporte, capacidades y configuración de Tlon/Urbit"
read_when:
  - Trabajando en funciones del canal Tlon/Urbit
---

<div id="tlon-plugin">
  # Tlon (complemento)
</div>

Tlon es un servicio de mensajería descentralizado construido sobre Urbit. OpenClaw se conecta a tu nave de Urbit y puede
responder a DMs y mensajes de chat grupal. Las respuestas en grupos requieren una mención con @ de forma predeterminada y se pueden
restringir aún más mediante listas de permitidos.

Estado: compatible mediante complemento. DMs, menciones en grupos, respuestas en hilos y mecanismo de reserva en modo solo texto para contenido multimedia
(URL añadida al pie de foto). Reacciones, encuestas y subidas nativas de contenido multimedia no se admiten.

<div id="plugin-required">
  ## Complemento requerido
</div>

Tlon se distribuye como un complemento y no se incluye en la instalación base.

Instala desde la CLI (registro de npm):

```bash
openclaw plugins install @openclaw/tlon
```

Copia local (cuando se ejecuta desde un repositorio git):

```bash
openclaw plugins install ./extensions/tlon
```

Detalles: [Complementos](/es/plugin)

<div id="setup">
  ## Configuración
</div>

1. Instala el complemento Tlon.
2. Obtén la URL de tu ship y el código de inicio de sesión.
3. Configura `channels.tlon`.
4. Reinicia el Gateway.
5. Envíale un mensaje directo (DM) al bot o menciónalo en un canal grupal.

Configuración mínima (una sola cuenta):

```json5
{
  channels: {
    tlon: {
      enabled: true,
      ship: "~sampel-palnet",
      url: "https://your-ship-host",
      code: "lidlut-tabwed-pillex-ridrup"
    }
  }
}
```

<div id="group-channels">
  ## Canales de grupo
</div>

El descubrimiento automático está activado por defecto. También puedes anclar canales manualmente:

```json5
{
  channels: {
    tlon: {
      groupChannels: [
        "chat/~host-ship/general",
        "chat/~host-ship/support"
      ]
    }
  }
}
```

Desactivar el descubrimiento automático:

```json5
{
  channels: {
    tlon: {
      autoDiscoverChannels: false
    }
  }
}
```

<div id="access-control">
  ## Control de acceso
</div>

Lista de permitidos de DM (vacía = permite todo):

```json5
{
  channels: {
    tlon: {
      dmAllowlist: ["~zod", "~nec"]
    }
  }
}
```

Autorización de grupos (restringida por defecto):

```json5
{
  channels: {
    tlon: {
      defaultAuthorizedShips: ["~zod"],
      authorization: {
        channelRules: {
          "chat/~host-ship/general": {
            mode: "restricted",
            allowedShips: ["~zod", "~nec"]
          },
          "chat/~host-ship/announcements": {
            mode: "open"
          }
        }
      }
    }
  }
}
```

<div id="delivery-targets-clicron">
  ## Destinos de entrega (CLI/cron)
</div>

Utilízalos con `openclaw message send` o para la entrega con cron:

* DM: `~sampel-palnet` o `dm/~sampel-palnet`
* Grupo: `chat/~host-ship/channel` o `group:~host-ship/channel`

<div id="notes">
  ## Notas
</div>

* Las respuestas en grupos requieren una mención (p. ej., `~your-bot-ship`) para poder responder.
* Respuestas en hilos: si el mensaje entrante está dentro de un hilo, OpenClaw responde en ese mismo hilo.
* Multimedia: `sendMedia` hace un fallback a texto + URL (sin carga nativa).