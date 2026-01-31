---
title: Zalo
summary: "Estado del soporte de bots de Zalo, funcionalidades y configuración"
read_when:
  - Al trabajar en funcionalidades o webhooks de Zalo
---

<div id="zalo-bot-api">
  # Zalo (Bot API)
</div>

Estado: experimental. Mensajes directos únicamente; compatibilidad con grupos próximamente según la documentación de Zalo.

<div id="plugin-required">
  ## Complemento requerido
</div>

Zalo se distribuye como complemento independiente y no viene incluido en la instalación base.

* Instálalo mediante la CLI: `openclaw plugins install @openclaw/zalo`
* O selecciona **Zalo** durante el asistente de configuración inicial y confirma la solicitud de instalación
* Más información: [Complementos](/es/plugin)

<div id="quick-setup-beginner">
  ## Configuración rápida (principiantes)
</div>

1. Instala el complemento de Zalo:
   * Desde un checkout del código fuente: `openclaw plugins install ./extensions/zalo`
   * Desde npm (si está publicado): `openclaw plugins install @openclaw/zalo`
   * O selecciona **Zalo** durante el onboarding y confirma el aviso de instalación
2. Establece el token:
   * Variable de entorno: `ZALO_BOT_TOKEN=...`
   * O configuración: `channels.zalo.botToken: "..."`
3. Reinicia el Gateway (o completa el onboarding).
4. El acceso por DM usa emparejamiento de forma predeterminada; aprueba el código de emparejamiento en el primer contacto.

Configuración mínima:

```json5
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing"
    }
  }
}
```

<div id="what-it-is">
  ## Qué es
</div>

Zalo es una aplicación de mensajería orientada a Vietnam; su Bot API permite que el Gateway ejecute un bot para conversaciones 1:1.
Es una buena opción para soporte o notificaciones cuando necesitas enrutamiento determinista de vuelta a Zalo.

* Un canal de Zalo Bot API propiedad del Gateway.
* Enrutamiento determinista: las respuestas regresan a Zalo; el modelo nunca elige canales.
* Los mensajes directos comparten la sesión principal del agente.
* Los grupos todavía no son compatibles (la documentación de Zalo indica &quot;coming soon&quot;).

<div id="setup-fast-path">
  ## Configuración rápida
</div>

<div id="1-create-a-bot-token-zalo-bot-platform">
  ### 1) Crear un token del bot (Zalo Bot Platform)
</div>

1. Ve a **https://bot.zaloplatforms.com** e inicia sesión.
2. Crea un nuevo bot y configura su configuración.
3. Copia el token del bot (formato: `12345689:abc-xyz`).

<div id="2-configure-the-token-env-or-config">
  ### 2) Configura el token (env o config)
</div>

Ejemplo:

```json5
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing"
    }
  }
}
```

Opción de entorno: `ZALO_BOT_TOKEN=...` (solo funciona para la cuenta predeterminada).

Soporte para múltiples cuentas: usa `channels.zalo.accounts` con tokens por cuenta y un `name` opcional.

3. Reinicia el Gateway. Zalo se inicia cuando se resuelve un token (entorno o configuración).
4. El acceso por mensajes directos (DM) usa emparejamiento de forma predeterminada. Aprueba el código cuando el bot sea contactado por primera vez.

<div id="how-it-works-behavior">
  ## Cómo funciona (comportamiento)
</div>

* Los mensajes entrantes se normalizan en la envoltura de canal compartida con marcadores de posición para medios.
* Las respuestas siempre se enrutan de vuelta al mismo chat de Zalo.
* Long-polling por defecto; modo webhook disponible en `channels.zalo.webhookUrl`.

<div id="limits">
  ## Límites
</div>

* El texto saliente se fragmenta en bloques de 2000 caracteres (límite de la API de Zalo).
* Las descargas/cargas de contenido multimedia están limitadas por `channels.zalo.mediaMaxMb` (valor predeterminado 5).
* El streaming está bloqueado de forma predeterminada porque el límite de 2000 caracteres hace que el streaming sea menos útil.

<div id="access-control-dms">
  ## Control de acceso (mensajes directos)
</div>

<div id="dm-access">
  ### Acceso por DM
</div>

* Valor predeterminado: `channels.zalo.dmPolicy = "pairing"`. Los remitentes desconocidos reciben un código de emparejamiento; los mensajes se ignoran hasta que sean aprobados (los códigos caducan después de 1 hora).
* Aprobar mediante:
  * `openclaw pairing list zalo`
  * `openclaw pairing approve zalo <CODE>`
* El emparejamiento es el intercambio de tokens predeterminado. Detalles: [Emparejamiento](/es/start/pairing)
* `channels.zalo.allowFrom` acepta IDs de usuario numéricos (no hay búsqueda por nombre de usuario disponible).

<div id="long-polling-vs-webhook">
  ## Long-polling vs webhook
</div>

* Por defecto: long-polling (no se requiere URL pública).
* Modo webhook: configura `channels.zalo.webhookUrl` y `channels.zalo.webhookSecret`.
  * El secreto del webhook debe tener entre 8 y 256 caracteres.
  * La URL del webhook debe usar HTTPS.
  * Zalo envía eventos con la cabecera `X-Bot-Api-Secret-Token` para verificación.
  * El Gateway HTTP atiende las solicitudes de webhook en `channels.zalo.webhookPath` (por defecto, la ruta de la URL del webhook).

**Nota:** getUpdates (polling) y webhook son mutuamente excluyentes según la documentación de la API de Zalo.

<div id="supported-message-types">
  ## Tipos de mensajes compatibles
</div>

* **Mensajes de texto**: Soporte completo con segmentación en bloques de 2000 caracteres.
* **Mensajes de imagen**: Descarga y procesamiento de imágenes entrantes; envío de imágenes mediante `sendPhoto`.
* **Stickers**: Registrados pero no procesados completamente (sin respuesta del agente).
* **Tipos no admitidos**: Registrados (por ejemplo, mensajes de usuarios protegidos).

<div id="capabilities">
  ## Capacidades
</div>

| Funcionalidad | Estado |
|---------------|--------|
| Mensajes directos | ✅ Disponible |
| Grupos | ❌ Próximamente (según la documentación de Zalo) |
| Multimedia (imágenes) | ✅ Disponible |
| Reacciones | ❌ No disponible |
| Hilos | ❌ No disponible |
| Encuestas | ❌ No disponible |
| Comandos nativos | ❌ No disponible |
| Streaming | ⚠️ Bloqueado (límite de 2000 caracteres) |

<div id="delivery-targets-clicron">
  ## Destinos de entrega (CLI/cron)
</div>

* Usa un ID de chat como destino.
* Ejemplo: `openclaw message send --channel zalo --target 123456789 --message "hi"`.

<div id="troubleshooting">
  ## Solución de problemas
</div>

**El bot no responde:**

* Verifica que el token sea válido: `openclaw channels status --probe`
* Comprueba que el remitente esté aprobado (emparejamiento o allowFrom)
* Revisa los logs del Gateway: `openclaw logs --follow`

**El webhook no recibe eventos:**

* Asegúrate de que la URL del webhook use HTTPS
* Verifica que el token secreto tenga entre 8 y 256 caracteres
* Confirma que el endpoint HTTP del Gateway sea accesible en la ruta configurada
* Comprueba que el sondeo `getUpdates` no esté en ejecución (son mutuamente excluyentes)

<div id="configuration-reference-zalo">
  ## Referencia de configuración (Zalo)
</div>

Configuración completa: [Configuration](/es/gateway/configuration)

Opciones del proveedor:

* `channels.zalo.enabled`: activar/desactivar el inicio del canal.
* `channels.zalo.botToken`: token del bot de Zalo Bot Platform.
* `channels.zalo.tokenFile`: leer el token desde una ruta de archivo.
* `channels.zalo.dmPolicy`: `pairing | allowlist | open | disabled` (predeterminado: pairing).
* `channels.zalo.allowFrom`: lista de permitidos de DM (ID de usuario). `open` requiere `"*"`. El asistente pedirá ID numéricos.
* `channels.zalo.mediaMaxMb`: límite de medios entrantes/salientes (MB, predeterminado 5).
* `channels.zalo.webhookUrl`: habilitar modo webhook (se requiere HTTPS).
* `channels.zalo.webhookSecret`: secreto del webhook (8-256 caracteres).
* `channels.zalo.webhookPath`: ruta del webhook en el servidor HTTP del Gateway.
* `channels.zalo.proxy`: URL de proxy para solicitudes a la API.

Opciones multicuenta:

* `channels.zalo.accounts.<id>.botToken`: token por cuenta.
* `channels.zalo.accounts.<id>.tokenFile`: archivo de token por cuenta.
* `channels.zalo.accounts.<id>.name`: nombre para mostrar.
* `channels.zalo.accounts.<id>.enabled`: activar/desactivar cuenta.
* `channels.zalo.accounts.<id>.dmPolicy`: política de DM por cuenta.
* `channels.zalo.accounts.<id>.allowFrom`: lista de permitidos por cuenta.
* `channels.zalo.accounts.<id>.webhookUrl`: URL de webhook por cuenta.
* `channels.zalo.accounts.<id>.webhookSecret`: secreto de webhook por cuenta.
* `channels.zalo.accounts.<id>.webhookPath`: ruta de webhook por cuenta.
* `channels.zalo.accounts.<id>.proxy`: URL de proxy por cuenta.